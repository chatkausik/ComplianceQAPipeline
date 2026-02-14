# Compliance QA Pipeline

<p align="center">
  <img src="assets/logo.png" alt="Compliance QA Pipeline Logo" width="180" />
</p>

<p align="center">
  <strong>Production-grade Retrieval-Augmented Generation (RAG) for compliance documentation</strong>
</p>

<p align="center">
  A specialized question-answering system designed for regulated environments, providing auditable, citation-backed answers from internal compliance documents.
</p>

---

## Overview

The Compliance QA Pipeline is a production-oriented RAG system built specifically for compliance-focused organizations. It ingests internal documents (policies, SOPs, contracts, audit evidence), indexes them for semantic search, and generates grounded answers with full source citations—essential for regulatory compliance and audit trails.

### Key Capabilities

- **Document Ingestion**: Process multiple formats (PDF, DOCX, TXT, MD) with robust text extraction
- **Intelligent Chunking**: Context-aware segmentation with configurable overlap for optimal retrieval
- **Vector Indexing**: Scalable embedding storage with metadata preservation (document ID, section, page, timestamps)
- **Semantic Retrieval**: Top-k similarity search with optional reranking for precision
- **Grounded Generation**: LLM-powered answers constrained to retrieved context
- **Full Citations**: Source attribution with document names, sections, and text snippets for auditability
- **Quality Evaluation**: Built-in metrics for answer relevance, faithfulness, and retrieval performance

### Compliance Standards Supported

Designed to support documentation requirements for:
- SOC 2 (Type I & II)
- ISO 27001 / 27002
- HIPAA
- PCI DSS
- GDPR
- NIST frameworks

---

## Architecture

### High-Level Pipeline

```
┌─────────────┐    ┌──────────┐    ┌───────────┐    ┌──────────┐
│  Documents  │───▶│ Ingestion│───▶│ Chunking  │───▶│Embedding │
│ (PDF/DOCX)  │    │& Extract │    │& Metadata │    │  Model   │
└─────────────┘    └──────────┘    └───────────┘    └──────────┘
                                                            │
                                                            ▼
┌─────────────┐    ┌──────────┐    ┌───────────┐    ┌──────────┐
│   Answer    │◀───│   LLM    │◀───│ Retrieval │◀───│  Vector  │
│+ Citations  │    │Generator │    │  Engine   │    │  Index   │
└─────────────┘    └──────────┘    └───────────┘    └──────────┘
```

### LangGraph Workflow

<p align="center">
  <img src="assets/Project2_Langgraph_Architecture.png" alt="LangGraph Architecture Diagram" width="900" />
</p>

The system implements a directed graph architecture with the following nodes:

#### Offline Path (Indexing)
1. **Document Loading**: Ingest files with metadata extraction
2. **Text Normalization**: Clean and structure raw content
3. **Chunking**: Segment into retrievable units with overlap
4. **Embedding**: Generate vector representations
5. **Index Storage**: Persist to vector database with metadata

#### Online Path (Query Processing)
1. **Request Entry**: Accept user query with session context
2. **Retrieval**: Semantic search → Top-k chunks → Optional reranking
3. **Answer Generation**: LLM invocation with grounded context and citation constraints
4. **Citation Formatting**: Transform chunk metadata into human-readable sources
5. **Quality Guardrails** (optional): Confidence checks, missing evidence detection, policy validation
6. **Response Assembly**: Package answer + citations + diagnostics

---

## Repository Structure

```text
ComplianceQAPipeline/
│
├── assets/                     # Visual assets
│   ├── logo.png               # Project logo
│   ├── Project2_Langgraph_Architecture.png
│   └── langsmith_tracing.png
│
├── backend/                    # Backend application code
│   └── src/
│       ├── api/               # FastAPI server and endpoints
│       │   └── server.py      # Main API application
│       ├── ingestion/         # Document loaders and text extraction
│       ├── embeddings/        # Embedding model wrappers
│       ├── vectorstore/       # Vector database adapters (FAISS/Chroma/Pinecone)
│       ├── retrieval/         # Retriever logic and reranking
│       ├── llm/               # LLM client wrappers, prompts, guardrails
│       ├── qa/                # RAG orchestration and citation formatting
│       └── utils/             # Shared utilities (logging, config, I/O)
│
├── data/                       # Data storage (excluded from version control)
│   ├── raw/                   # Original compliance documents
│   ├── processed/             # Normalized/extracted text
│   └── indexes/               # Vector index artifacts
│
├── configs/                    # Configuration files
│   └── default.yaml           # Default pipeline configuration
│
├── scripts/                    # CLI entrypoints
│   ├── build_index.py         # Index generation script
│   ├── ask.py                 # Interactive query CLI
│   └── evaluate.py            # Evaluation runner
│
├── tests/                      # Test suite
│   ├── unit/                  # Unit tests
│   └── integration/           # Integration tests
│
├── notebooks/                  # Jupyter notebooks (optional)
│   └── exploration.ipynb      # Data exploration and demos
│
├── docs/                       # Extended documentation
│   ├── architecture.md        # Detailed architecture decisions
│   └── ADRs/                  # Architecture Decision Records
│
├── .env.example               # Environment variable template
├── pyproject.toml             # Project dependencies and metadata
└── README.md                  # This file
```

---

## Getting Started

### Prerequisites

- **Python**: 3.10 or higher
- **Package Manager**: `uv` (recommended) or `pip`
- **API Keys**: For your chosen LLM and embedding providers (unless using local models)
- **Docker** (optional): For containerized deployment

### Installation

#### Using uv (Recommended)

```bash
# Install uv if not already installed
pip install uv
# Alternative: pipx install uv
# Alternative: brew install uv

# Clone the repository
git clone https://github.com/your-org/ComplianceQAPipeline.git
cd ComplianceQAPipeline

# Sync dependencies
uv sync

# Activate the virtual environment
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

#### Using pip

```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -U pip
pip install -e .
```

### Configuration

1. **Copy environment template**:
   ```bash
   cp .env.example .env
   ```

2. **Configure environment variables** in `.env`:
   ```bash
   # LLM Configuration
   LLM_PROVIDER=openai          # Options: openai, azure, anthropic, local
   LLM_MODEL=gpt-4
   LLM_API_KEY=your_api_key_here
   
   # Embedding Configuration
   EMBEDDING_MODEL=text-embedding-ada-002
   EMBEDDING_PROVIDER=openai
   
   # Vector Store
   VECTORSTORE_PROVIDER=faiss   # Options: faiss, chroma, pinecone
   
   # Paths
   DATA_DIR=./data
   INDEX_DIR=./data/indexes
   
   # Optional: Observability
   LANGSMITH_API_KEY=your_langsmith_key
   LANGSMITH_PROJECT=compliance-qa
   ```

3. **Adjust pipeline configuration** in `configs/default.yaml`:
   ```yaml
   chunking:
     chunk_size: 512
     chunk_overlap: 128
   
   retrieval:
     top_k: 5
     similarity_threshold: 0.7
     use_reranker: true
   
   generation:
     temperature: 0.1
     max_tokens: 1000
   ```

---

## Usage

### 1. Index Your Documents

Place your compliance documents in `data/raw/`, then build the index:

```bash
python -m scripts.build_index \
  --input data/raw \
  --output data/indexes \
  --config configs/default.yaml
```

**Output**: Vector index stored in `data/indexes/` with document metadata.

### 2. Ask Questions (CLI)

```bash
python -m scripts.ask \
  --query "What are the retention requirements for audit logs?" \
  --index data/indexes \
  --config configs/default.yaml
```

**Example Output**:
```json
{
  "answer": "Audit logs must be retained for a minimum of 90 days in active storage and 7 years in archive storage according to our retention policy.",
  "citations": [
    {
      "source": "Data_Retention_Policy_v2.1.pdf",
      "section": "3.2 Audit Log Retention",
      "page": 5,
      "snippet": "All audit logs shall be retained for no less than 90 days..."
    }
  ],
  "confidence": 0.92
}
```

### 3. Run the API Server

Start the FastAPI development server:

```bash
uv run uvicorn backend.src.api.server:app --reload
```

The server starts at `http://localhost:8000`

**Available Endpoints**:
- **API Documentation**: `http://localhost:8000/docs` (Swagger UI)
- **Health Check**: `GET http://localhost:8000/health`
- **Query Endpoint**: `POST http://localhost:8000/audit`

**Example API Request**:
```bash
curl -X POST "http://localhost:8000/audit" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://youtu.be/abc123"
  }'
```

#### API Request Flow

```
1. Client → POST /audit with request body
2. FastAPI validates request (AuditRequest model)
3. audit_video() generates session_id
4. LangGraph workflow executes:
   START → Indexer → Auditor → END
5. Response validated (AuditResponse model)
6. JSON returned to client
7. Telemetry captured (LangSmith + Azure Monitor)
```

### 4. Evaluate System Performance

Run evaluation on a test question set:

```bash
python -m scripts.evaluate \
  --questions data/eval/questions.jsonl \
  --index data/indexes \
  --out data/eval/results.json \
  --config configs/default.yaml
```

**Metrics Tracked**:
- Answer relevance
- Faithfulness (hallucination detection)
- Retrieval precision/recall
- Citation accuracy
- Latency (p50, p95, p99)

---

## Observability

### LangSmith Tracing

<p align="center">
  <img src="assets/langsmith_tracing.png" alt="LangSmith Trace View" width="900" />
</p>

The pipeline integrates with LangSmith for comprehensive observability:

- **End-to-end traces**: Track requests from ingestion through retrieval to generation
- **Node-level timing**: Identify bottlenecks in the LangGraph workflow
- **Token usage**: Monitor LLM consumption and costs
- **Error tracking**: Capture and diagnose failures with full context
- **A/B testing**: Compare prompt variations and model configurations

**Setup**:
```bash
export LANGSMITH_API_KEY="your_key"
export LANGSMITH_PROJECT="compliance-qa"
export LANGCHAIN_TRACING_V2=true
```

### Azure Monitor (Optional)

For production deployments, integrate with Azure Application Insights:
- Request duration and status codes
- Dependency calls (vector DB, LLM API)
- Custom events and metrics
- Distributed tracing correlation

---

## Development

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=backend/src --cov-report=html

# Run specific test suite
pytest tests/unit/test_retrieval.py
```

### Code Quality

```bash
# Format code
black backend/

# Lint
ruff check backend/

# Type checking
mypy backend/
```

### Pre-commit Hooks

```bash
# Install pre-commit hooks
pre-commit install

# Run manually
pre-commit run --all-files
```

---

## Deployment

### Docker Deployment

```bash
# Build image
docker build -t compliance-qa-pipeline .

# Run container
docker run -p 8000:8000 \
  -e LLM_API_KEY=$LLM_API_KEY \
  -v $(pwd)/data:/app/data \
  compliance-qa-pipeline
```

### Production Considerations

- **Secrets Management**: Use Azure Key Vault, AWS Secrets Manager, or similar
- **Vector Store**: Migrate to managed service (Pinecone, Weaviate, Qdrant)
- **Scaling**: Deploy behind load balancer with horizontal pod autoscaling
- **Monitoring**: Configure alerts for latency, error rates, and token usage
- **Caching**: Implement Redis for frequently-asked questions
- **Rate Limiting**: Protect endpoints from abuse

---

## Roadmap

- [ ] Multi-modal support (images, tables in PDFs)
- [ ] Advanced reranking (cross-encoder models)
- [ ] Query expansion and reformulation
- [ ] Multi-document comparison queries
- [ ] Fine-tuned embedding models for compliance domain
- [ ] Human-in-the-loop feedback integration
- [ ] Automated document version tracking
- [ ] Compliance-specific evaluation benchmarks

---

## Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Setup

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes with tests
4. Run quality checks (`pytest`, `black`, `ruff`)
5. Commit changes (`git commit -m 'Add amazing feature'`)
6. Push to branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- Built with [LangChain](https://github.com/langchain-ai/langchain) and [LangGraph](https://github.com/langchain-ai/langgraph)
- Observability powered by [LangSmith](https://smith.langchain.com/)
- Vector search using [FAISS](https://github.com/facebookresearch/faiss) / [Chroma](https://www.trychroma.com/)

---

## Support

- **Documentation**: [docs/](docs/)
- **Issues**: [GitHub Issues](https://github.com/your-org/ComplianceQAPipeline/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-org/ComplianceQAPipeline/discussions)
- **Email**: compliance-qa@your-org.com

---

## Citation

If you use this project in your research or production systems, please cite:

```bibtex
@software{compliance_qa_pipeline,
  title = {Compliance QA Pipeline: RAG for Regulated Environments},
  author = {Your Organization},
  year = {2024},
  url = {https://github.com/your-org/ComplianceQAPipeline}
}
```