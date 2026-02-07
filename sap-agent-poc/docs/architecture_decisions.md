# Architecture Decision Record (ADR-001)

## Context
We needed to deploy an automated triage agent for SAP L1 Support tickets. 
**Constraint:** Customer data (ST22 Dumps) cannot leave the on-premise network (Data Sovereignty). Cloud APIs (OpenAI/Anthropic) are strictly forbidden.

## Decision: Local Quantized Inference (Mistral-7B)
We chose to run **Mistral-7B-Instruct-v0.2** quantized to 4-bit (GGUF format) running on a local NVIDIA RTX 3060.

### Alternatives Considered
1.  **Bert-Base-Uncased (Fine-Tuned):** * *Pros:* Extremely fast (<100ms).
    * *Cons:* Poor reasoning capabilities on "ambiguous" errors. Cannot generate "suggested actions," only classification labels.
2.  **Llama-2-70B (GPTQ):**
    * *Pros:* High reasoning capability.
    * *Cons:* Requires 2x A100s (48GB+ VRAM). Too expensive for an edge node.

### Why Mistral-7B + vLLM?
* **Performance:** Fits entirely in 12GB VRAM (approx 5.8GB memory footprint).
* **Throughput:** Token generation speed on RTX 3060 is ~35 t/s, which meets the sub-2s latency SLA for triage.
* **Recursion:** The model context window (8k) allows for roughly 3 "reasoning steps" (Recursive RAG) before context overflow.

## Future Roadmap
* Migrate vector store from FAISS (In-Memory) to **Qdrant** for persistence.
* Implement **docker-compose** orchestration to separate the API service from the Vector DB.