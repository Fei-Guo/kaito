# KAITO Presets Directory - AI Agent Navigation Guide

## Overview
The `/presets` directory contains all model preset configurations, runtime implementations, and RAG (Retrieval-Augmented Generation) engine components for the KAITO (Kubernetes AI Toolchain Operator) project. This directory serves as the core implementation layer for:
- **Model inference** using HuggingFace Transformers and vLLM
- **Model fine-tuning** with LoRA and quantization support
- **RAG engine** for document indexing and vector-based retrieval
- **Model metadata** and GPU SKU calculations

---

## Directory Structure

### 📁 `/presets/workspace/` - Model Workspace Implementation
Core implementation for LLM model inference and fine-tuning operations.

#### **Key Subdirectories:**

##### `/workspace/models/` - Model Definitions (Go)
- **Purpose**: Go-based model metadata, configurations, and SKU requirements
- **Key Files**:
  - `supported_models.yaml` - Registry of all supported models with versions and tags
  - `metadata.go` - Core Go package for model metadata management
  - Model-specific directories: `phi4/`, `phi3/`, `llama3/`, `mistral/`, `falcon/`, `deepseek/`, `qwen/`, `gemma3/`, `gpt/`
  - Each model directory contains:
    - `model.go` - Model configuration (GPU requirements, memory, runtime params)
    - `README.md` - Model documentation and usage

##### `/workspace/inference/` - Inference Engines (Python)
- **Purpose**: Python implementations for model inference APIs
- **Subdirectories**:
  - `text-generation/` - Transformers-based inference API
    - `inference_api.py` - FastAPI server for text generation
    - `api_spec.json` - OpenAPI specification
    - `tests/` - Unit tests
  - `vllm/` - vLLM-based high-performance inference
    - `inference_api.py` - vLLM FastAPI integration
    - `multi-node-health-check.py` - Multi-node distributed health monitoring
    - `api_spec.json` - OpenAPI specification
  - `chat_templates/` - Jinja2 templates for chat formatting
    - Multiple model-specific templates (`.jinja` files)
    - `chat_template_guide.md` - Documentation for creating chat templates

##### `/workspace/tuning/` - Fine-Tuning Implementation (Python)
- **Purpose**: Model fine-tuning with LoRA and quantization
- **Key Files in `text-generation/`**:
  - `fine_tuning.py` - Main fine-tuning logic with LoRA and quantization
  - `cli.py` - Command-line configuration dataclasses
  - `parser.py` - YAML config parser for training parameters
  - `dataset.py` - Dataset loading and preprocessing
  - `metrics/metrics_server.py` - Prometheus metrics server for training

##### `/workspace/generator/` - SKU Calculation Tools (Python)
- **Purpose**: GPU resource calculation and preset generation
- **Key Files**:
  - `preset_generator.py` - Generates model presets from HuggingFace models
  - `model-sku-calculation.md` - VRAM calculation formulas and SKU estimation guide
  - `README.md` - Usage guide for preset generation
  - **Use Cases**:
    - Calculate GPU memory requirements for models
    - Estimate optimal node counts
    - Generate YAML configs for new model presets

##### `/workspace/dependencies/` - Python Dependencies
- **Purpose**: Shared Python requirements
- **Files**:
  - `requirements.txt` - Core dependencies for inference and tuning
  - `requirements-test.txt` - Testing dependencies

##### `/workspace/test/` - Testing and Benchmarking
- **Purpose**: Test manifests and benchmarking scripts
- **Subdirectories**:
  - `manifests/` - Kubernetes YAML manifests for testing
  - `llama-test/`, `falcon-benchmark/` - Model-specific benchmarks
  - `scripts/` - Manifest generation utilities

---

### 📁 `/presets/ragengine/` - RAG Engine Implementation (Python)
Complete RAG (Retrieval-Augmented Generation) engine for document indexing and vector search.

#### **Key Subdirectories:**

##### `/ragengine/` (Root)
- **Main Files**:
  - `main.py` - FastAPI application entry point
  - `models.py` - Pydantic models for API requests/responses
  - `config.py` - Configuration constants and environment variables
  - `requirements.txt` - RAG-specific dependencies (llama-index, faiss, etc.)

##### `/ragengine/vector_store/` - Vector Database Implementations
- **Purpose**: Vector storage and retrieval backends
- **Key Files**:
  - `base.py` - Abstract base class for vector stores
  - `faiss_store.py` - FAISS vector store implementation
  - `node_processors/` - Custom node processing (context selection)
  - `transformers/` - Custom document transformers

##### `/ragengine/embedding/` - Embedding Models
- **Purpose**: Text embedding generation
- **Files**:
  - `base.py` - Base embedding interface
  - `huggingface_local_embedding.py` - Local HuggingFace embeddings
  - `remote_embedding.py` - Remote embedding service integration

##### `/ragengine/vector_store_manager/` - Vector Store Lifecycle
- **Purpose**: Manage vector store creation, loading, and persistence
- **Files**:
  - `manager.py` - Vector store manager class

##### `/ragengine/inference/` - RAG Inference
- **Purpose**: LLM inference integration for RAG
- **Files**:
  - `inference.py` - Inference abstraction layer

##### `/ragengine/lifecycle/` - System Lifecycle Management
- **Purpose**: Startup/shutdown hooks and lifecycle management
- **Files**:
  - `manager.py` - Lifecycle manager
  - `hooks.py` - Lifecycle hooks (startup, shutdown)

##### `/ragengine/metrics/` - Prometheus Metrics
- **Purpose**: Observability and monitoring
- **Files**:
  - `prometheus_metrics.py` - Metrics definitions and collectors
  - `helpers.py` - Metric helper utilities

##### `/ragengine/tests/` - RAG Engine Tests
- **Purpose**: Comprehensive test suite
- **Subdirectories**:
  - `api/` - API endpoint tests
  - `vector_store/` - Vector store tests

---

## Common Tasks & File Locations

### 🔍 **Adding a New Model**
1. **Update model registry**: `workspace/models/supported_models.yaml`
2. **Generate preset config**: Use `workspace/generator/preset_generator.py`
3. **Create model directory**: `workspace/models/<model-name>/`
4. **Implement model.go**: Define GPU requirements, runtime params in `workspace/models/<model-name>/model.go`
5. **Add chat template** (if needed): `workspace/inference/chat_templates/<model-name>.jinja`

### 🚀 **Inference-Related Tasks**
- **Modify Transformers inference**: `workspace/inference/text-generation/inference_api.py`
- **Modify vLLM inference**: `workspace/inference/vllm/inference_api.py`
- **Add chat template**: `workspace/inference/chat_templates/`
- **Multi-node health checks**: `workspace/inference/vllm/multi-node-health-check.py`

### 🎯 **Fine-Tuning Related Tasks**
- **Modify fine-tuning logic**: `workspace/tuning/text-generation/fine_tuning.py`
- **Update training config parsing**: `workspace/tuning/text-generation/parser.py`
- **Add dataset processing**: `workspace/tuning/text-generation/dataset.py`
- **Metrics collection**: `workspace/tuning/text-generation/metrics/metrics_server.py`

### 📊 **SKU & Resource Calculation**
- **Generate new model preset**: Run `workspace/generator/preset_generator.py`
- **Calculate VRAM requirements**: Reference `workspace/generator/model-sku-calculation.md`
- **Update model metadata**: Modify corresponding `workspace/models/<model>/model.go`

### 🔎 **RAG Engine Tasks**
- **API endpoints**: `ragengine/main.py`
- **Add vector store backend**: Implement in `ragengine/vector_store/` following `base.py`
- **Add embedding model**: Implement in `ragengine/embedding/` following `base.py`
- **Document processing**: `ragengine/vector_store/node_processors/`
- **Lifecycle hooks**: `ragengine/lifecycle/hooks.py`
- **Metrics**: `ragengine/metrics/prometheus_metrics.py`

### 🧪 **Testing**
- **Inference tests**: `workspace/inference/*/tests/`
- **RAG engine tests**: `ragengine/tests/`
- **Benchmark scripts**: `workspace/test/<model>-benchmark/`
- **Test manifests**: `workspace/test/manifests/`

---

## Technology Stack Summary

### **Languages**
- **Go**: Model metadata, Kubernetes integration, SKU definitions
- **Python**: Inference APIs, fine-tuning, RAG engine

### **Key Python Libraries**
#### Inference & Training:
- `transformers` - HuggingFace Transformers
- `vllm` - High-performance LLM inference
- `peft` - Parameter-Efficient Fine-Tuning (LoRA)
- `bitsandbytes` - Quantization
- `trl` - Transformer Reinforcement Learning
- `accelerate` - Distributed training
- `torch` - PyTorch backend

#### RAG Engine:
- `llama-index` - RAG framework
- `faiss-cpu` - Vector similarity search
- `fastapi` - API framework
- `uvicorn` - ASGI server
- `openai` - OpenAI API compatibility
- `prometheus-client` - Metrics

### **Inference Runtimes**
- **TFS (Transformers)**: HuggingFace Transformers-based
- **vLLM**: High-throughput LLM serving

---

## Quick Reference: File Type Guide

| File Pattern | Purpose | Language |
|--------------|---------|----------|
| `*.go` | Model metadata, SKU configs, Kubernetes integration | Go |
| `*_api.py` | FastAPI inference/RAG endpoints | Python |
| `fine_tuning.py` | Training loop implementation | Python |
| `*.yaml` | Model registry, Kubernetes manifests | YAML |
| `*.jinja` | Chat templates for model formatting | Jinja2 |
| `requirements*.txt` | Python dependencies | Text |
| `*_test.py`, `test_*.py` | Unit/integration tests | Python |
| `README.md` | Documentation | Markdown |

---

## Environment Variables Reference

### **Inference (Transformers & vLLM)**
- `DEBUG_MODE` - Enable debug logging (`true`/`false`)
- `MODEL_NAME` - HuggingFace model identifier
- `ADAPTERS_DIR` - LoRA adapter directory (default: `/mnt/adapter`)

### **Fine-Tuning**
- `YAML_FILE_PATH` - Training config YAML path (default: `/mnt/config/training_config.yaml`)
- `DEBUG_MODE` - Debug logging

### **RAG Engine**
- `EMBEDDING_SOURCE_TYPE` - Embedding source (`local`/`remote`)
- `LOCAL_EMBEDDING_MODEL_ID` - Local HF embedding model
- `REMOTE_EMBEDDING_URL` - Remote embedding API endpoint
- `REMOTE_EMBEDDING_ACCESS_SECRET` - API secret for remote embeddings
- `DEFAULT_VECTOR_DB_PERSIST_DIR` - Vector DB persistence path
- `RAG_SIMILARITY_THRESHOLD` - Vector search similarity threshold
- `RAG_DEFAULT_CONTEXT_TOKEN_FILL_RATIO` - Context window fill ratio

---

## Important Conventions

### **Model Naming**
- Model names use lowercase with hyphens: `llama-3.1-8b-instruct`
- Go constants use PascalCase: `PresetPhi4Model`
- File paths use snake_case: `inference_api.py`

### **Configuration Files**
- Model configs: YAML format (`supported_models.yaml`)
- Training configs: YAML with specific schema (see `parser.py`)
- API specs: JSON OpenAPI format (`api_spec.json`)

### **Testing**
- Test files prefix: `test_*.py`
- Use pytest framework
- Separate test requirements: `requirements-test.txt`

---

## Code Navigation Tips for AI Agents

1. **For model configuration queries**: Start with `workspace/models/supported_models.yaml` then navigate to specific model's `model.go`

2. **For inference issues**: 
   - Transformers → `workspace/inference/text-generation/inference_api.py`
   - vLLM → `workspace/inference/vllm/inference_api.py`

3. **For fine-tuning issues**: `workspace/tuning/text-generation/fine_tuning.py`

4. **For RAG functionality**: `ragengine/main.py` (entry point) → specific module

5. **For resource calculations**: `workspace/generator/preset_generator.py` or `model-sku-calculation.md`

6. **For dependencies**: 
   - Workspace: `workspace/dependencies/requirements.txt`
   - RAG: `ragengine/requirements.txt`

7. **For testing**: Look for matching `tests/` directory or `test_*.py` files

---

## API Endpoints Quick Reference

### **Workspace Inference API** (text-generation & vLLM)
- `POST /chat/completions` - Chat completion
- `GET /health` - Health check
- `GET /` - API info

### **RAG Engine API**
- `POST /index` - Index documents
- `POST /chat/completions` - RAG-augmented chat
- `GET /documents` - List indexed documents
- `PUT /documents/{doc_id}` - Update document
- `DELETE /documents/{doc_id}` - Delete document
- `GET /health` - Health status
- `GET /metrics` - Prometheus metrics

---

## Dependencies & Build Process

### **Python Environment Setup**
```bash
# Workspace dependencies
pip install -r presets/workspace/dependencies/requirements.txt

# RAG engine dependencies
pip install -r presets/ragengine/requirements.txt

# Testing
pip install -r presets/workspace/dependencies/requirements-test.txt
pip install -r presets/ragengine/requirements-test.txt
```

### **Go Model Building**
- Models are registered via Go plugins in each model's `model.go`
- Build process managed by parent Makefile

---

## Related Documentation

- **Model SKU Calculation**: [workspace/generator/model-sku-calculation.md](workspace/generator/model-sku-calculation.md)
- **Preset Generator**: [workspace/generator/README.md](workspace/generator/README.md)
- **Chat Templates**: [workspace/inference/chat_templates/chat_template_guide.md](workspace/inference/chat_templates/chat_template_guide.md)
- **TFS Docker Build**: [../docker/presets/models/tfs/README.md](../docker/presets/models/tfs/README.md)

---

## Version & Update History

Models maintain version history in `supported_models.yaml` with tag comments documenting changes.

Example:
```yaml
- name: base
  tag: 0.1.1
  # Tag history:
  # 0.1.1 - bump ray to 2.52.1
  # 0.1.0 - bump vLLM to 0.12.0
```

---

*This guide is optimized for AI agent navigation. For human-readable documentation, refer to the main KAITO docs.*
