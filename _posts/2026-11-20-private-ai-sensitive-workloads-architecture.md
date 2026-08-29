---
title: "Private AI — Architecture for Sensitive Workloads That Can't Leave the Org"
date: 2026-11-20
categories: [ai, agentic-ai]
tags: [agentic-ai, enterprise, ai-in-sdlc]
description: "Healthcare, financial services, and defense have workloads that legally cannot use external AI APIs — this is the architecture for building AI capabilities that stay within your network boundary."
mermaid: true
---

The conversation around private AI usually starts with data governance. A legal or compliance team flags that the data you want to run through an LLM cannot go to an external API. The engineering team then needs to build AI capabilities that work entirely within the organization's network boundary — no external API calls, no telemetry, no model weight downloads from the internet at runtime.

This is harder than self-hosting with a DPA. It requires thinking through every component of the AI stack for potential data egress, including ones that aren't obvious: telemetry from Python libraries, automatic update checks in serving frameworks, external embedding model downloads, and logging that flows to SaaS observability tools.

```mermaid
graph TB
    subgraph "Internal Network Boundary — No Egress Permitted"
        direction TB
        UQ[User Query] --> GW[API Gateway\nRate limiting + Auth]
        GW --> VLLM[vLLM Inference Server\nAir-gapped model weights]
        GW --> EMB[Embedding Model\nnomic-embed-text local]
        EMB --> VS[Vector Store\nQdrant self-hosted]
        VS --> RAG[RAG Pipeline]
        RAG --> VLLM
        VLLM --> RESP[Response]
        RESP --> LOG[Audit Log\nPostgres on-prem]
    end

    subgraph "External — Blocked"
        EXT1[HuggingFace Hub]
        EXT2[Anthropic API]
        EXT3[External Telemetry]
        EXT4[Package Registries]
    end

    GW -.->|BLOCKED| EXT1
    VLLM -.->|BLOCKED| EXT2
    LOG -.->|BLOCKED| EXT3
    style "External — Blocked" fill:#742a2a,color:#fff
    style "Internal Network Boundary — No Egress Permitted" fill:#1a365d,color:#fff
```

---

## When Private AI Is Required vs. Preferred

There's a difference between legally required and operationally preferred. Getting this right determines how restrictive your architecture needs to be.

**Legally required (no external APIs regardless of DPA):**
- HIPAA PHI in US healthcare — some healthcare organizations accept a BAA from major cloud providers; others mandate on-premises for all PHI processing
- Government classified workloads — classified data cannot touch commercial cloud under any circumstances
- Export-controlled technical data (ITAR/EAR in the US) — certain defense and dual-use technical data cannot go to foreign cloud providers
- Attorney-client privileged communications — privilege can be waived by disclosure to third parties

**Operationally preferred (external APIs possible with contractual controls):**
- PII in most jurisdictions — can use external APIs with appropriate DPA and data processing agreements
- Proprietary source code — engineers are cautious, but most vendor DPAs explicitly exclude training on customer data
- Internal business data without special regulatory classification

If you're in the "legally required" category, the architecture in this post applies. If you're in the "operationally preferred" category, a well-governed external API with a strong DPA is likely simpler and worth evaluating before committing to the private AI infrastructure burden.

---

## Hardware for Constrained Environments

Air-gapped or on-premises environments limit your hardware options to what you can physically procure and install.

**NVIDIA A100 SXM 80GB** — still available, well-understood, mature driver stack. For most private AI deployments, 2-4x A100 nodes running Llama 3.1 70B in AWQ INT4 is the practical starting point.

**NVIDIA H100 SXM 80GB** — better performance and FP8 support. Supply chain availability has improved but is not guaranteed. Verify lead times before designing around H100 for a planned deployment.

**AMD MI300X 192GB** — 192GB HBM3 means a 70B model fits in one chip in FP16. The software ecosystem (ROCm) is improving but is not at CUDA parity. Validate your specific model and serving stack before committing.

**Physical security:** NVIDIA A/H100 SXM cards require SXM-form-factor servers (DGX, HGX, or compatible OEM). These are large, power-hungry machines requiring proper data center infrastructure. Plan for 6-10kW per GPU node, specialized cooling, and physical access controls appropriate for your security classification.

---

## Software Stack Without External Dependencies

**Model serving: vLLM with no telemetry**

```bash
# vLLM telemetry opt-out — must be set before starting the server
export VLLM_NO_USAGE_STATS=1
export DO_NOT_TRACK=1

# Start vLLM pointing at locally stored model weights
vllm serve /models/llama-3.1-70b-awq \
  --trust-remote-code \
  --max-model-len 8192 \
  --gpu-memory-utilization 0.90 \
  --disable-log-requests \  # Don't log request content to stdout
  --no-deprecation-warnings
```

**HuggingFace Hub mirror:** Model weights need to be downloaded once and stored internally. Use `huggingface-cli` on a machine with internet access (outside the air-gapped environment), then transfer via secure media or an internal file server:

```bash
# On internet-connected machine (pre-staging environment)
pip install huggingface_hub
huggingface-cli download Qwen/Qwen2.5-72B-Instruct-AWQ \
  --local-dir /staging/models/qwen2.5-72b-awq \
  --token $HF_TOKEN

# Verify model file integrity
python -c "
from huggingface_hub import scan_cache_dir
print(scan_cache_dir())
"

# Transfer to air-gapped environment via approved secure transfer mechanism
# rsync, encrypted USB, secure file transfer — per your security policy
```

Set `HF_HUB_OFFLINE=1` in the air-gapped environment to prevent any attempt to reach the HuggingFace Hub:

```bash
export HF_HUB_OFFLINE=1
export TRANSFORMERS_OFFLINE=1
```

---

## Private RAG: Everything On-Premises

RAG in a private AI deployment requires every component to run within the network boundary.

**Embedding model — nomic-embed-text or e5-mistral-7b:**

```python
# Using nomic-embed-text locally via Ollama (no external calls once pulled)
# Or via sentence-transformers directly:
from sentence_transformers import SentenceTransformer

# Model weights pre-downloaded and stored locally
model = SentenceTransformer("/models/nomic-embed-text-v1.5", local_files_only=True)

texts = ["Document chunk to embed", "Another chunk"]
embeddings = model.encode(texts, show_progress_bar=False)
print(embeddings.shape)  # (2, 768)
```

**Vector store — Qdrant self-hosted:**

```yaml
# docker-compose.yml — Qdrant with persistent storage
version: "3.8"
services:
  qdrant:
    image: qdrant/qdrant:v1.9.1  # Pin version — no auto-updates
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - /data/qdrant:/qdrant/storage
    environment:
      QDRANT__TELEMETRY_DISABLED: "true"  # Disable Qdrant telemetry
    restart: unless-stopped
```

**Qdrant Python client:**

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

client = QdrantClient(host="localhost", port=6333)

# Create collection
client.create_collection(
    collection_name="enterprise_docs",
    vectors_config=VectorParams(size=768, distance=Distance.COSINE),
)

# Upsert documents
client.upsert(
    collection_name="enterprise_docs",
    points=[
        PointStruct(id=1, vector=embeddings[0].tolist(), payload={"text": texts[0]}),
        PointStruct(id=2, vector=embeddings[1].tolist(), payload={"text": texts[1]}),
    ],
)

# Search
results = client.search(
    collection_name="enterprise_docs",
    query_vector=model.encode("What is the document about?").tolist(),
    limit=5,
)
```

---

## Operational Security: The Egress You Missed

The most common failure mode in "private" AI deployments is residual data egress that wasn't designed but happens by default.

**Python package telemetry:** Many ML libraries (PyTorch, HuggingFace Transformers, others) make network calls for telemetry, update checks, and feature flags. Audit and disable:

```bash
# Set these in your container environment
DO_NOT_TRACK=1
HF_HUB_DISABLE_TELEMETRY=1
TRANSFORMERS_OFFLINE=1
HF_HUB_OFFLINE=1
PYTORCH_NO_CUDA_MEMORY_CACHING=0  # Not related, but unset confusable env vars

# Test: strace your inference server at startup to catch unexpected network calls
strace -f -e trace=network -p $(pgrep vllm) 2>&1 | grep -v "127.0.0.1\|::1"
```

**Container image pulls:** Pin container image versions with digest hashes. Never use `latest` in a private AI deployment. A `docker pull` on restart could reach external registries.

```bash
# Use digest pinning
docker pull vllm/vllm-openai@sha256:abc123...

# Or mirror the image to your internal registry
docker pull vllm/vllm-openai:0.5.4
docker tag vllm/vllm-openai:0.5.4 internal-registry.company.com/vllm:0.5.4
docker push internal-registry.company.com/vllm:0.5.4
```

**Logging pipelines:** If your logs flow to Datadog, Splunk Cloud, or any other SaaS observability tool, model inputs/outputs flowing through those logs become an external data transfer. Route private AI logs to on-premises Elasticsearch or a database with data classification filters.

---

## Compliance Documentation

Documenting a private AI system for regulatory review typically requires:

1. **Data flow diagram** — every path data takes from input to output, explicitly showing no egress. The Mermaid diagram at the top of this post is the kind of artifact regulators ask for.
2. **Model provenance** — where the model weights came from, how they were verified (checksum), and what fine-tuning (if any) was applied to them.
3. **Access controls** — who can query the model, who can access logs, how authentication is enforced.
4. **Audit logging** — every inference request logged with timestamp, user identity, and response metadata (not necessarily content, depending on the use case).
5. **Incident response procedure** — what happens if the model serves harmful content or if a data breach occurs.

Most regulated-industry compliance frameworks are not LLM-specific. They apply existing data processing and information security frameworks to AI systems. The documentation is the standard security control documentation your org already produces — applied to new components.

The architecture is achievable. The operational discipline is harder than the engineering.
