# SAP Incident Triage Engine (POC)

**Current Status:** Prototype (Migrating from Local RTX 3060 to Containerized Service)

This is a Proof-of-Concept agentic workflow designed to automate L1/L2 SAP Support ticket resolution. It ingests unstructured ST22 (ABAP Dumps) and SM21 Logs, matches them against a vector knowledge base of SAP Notes, and proposes resolution playbooks.

## 🏗 Architecture
- **Ingestion:** Asynchronous parsing of raw text logs (see `data/raw_dumps/`).
- **Inference:** Mistral-7B (4-bit GGUF quantization) running on `llama.cpp` python bindings.
- **RAG:** FAISS vector store containing embedded resolution strategies (simulated SAP Notes).
- **Guardrails:** - **Depth-3 Recursion Limit:** Prevents the agent from entering infinite reasoning loops on ambiguous errors (e.g., generic `SYSTEM_CORE_DUMP`).
    - **Confidence Threshold:** Answers with <0.8 confidence are routed to human review (simulated via Console output).

## 🚀 Performance
- **Hardware:** Tested on NVIDIA RTX 3060 (12GB VRAM).
- **Latency:** Average Time-to-Triage: **1.2s** (down from manual avg of ~15 mins).
- **Throughput:** Capable of processing ~40 logs/min in batch mode.

## 🛠 How to Run
```bash
# Install dependencies (requires llama-cpp-python compiled with cuBLAS)
pip install -r requirements.txt

# Run the triage agent on the sample dumps
python main.py --input data/raw_dumps/st22_dump_01_conversion_error.txt