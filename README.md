# UB CSE Website Mirror and Query System

This repository contains tools to mirror and query the University at Buffalo Computer Science and Engineering department website using local LLM models via Ollama.

## Overview

The project consists of three main components:

1. **Website Mirroring** (`ub_cse_mirror_wget_workflow.ipynb`) - Downloads and mirrors the UB CSE website
2. **Simple Query System** (`query_with_ollama.ipynb`) - Direct querying without RAG
3. **RAG Query System** (`rag_query_website.ipynb`) - Advanced querying with Retrieval Augmented Generation

## Prerequisites

- **macOS** (or Linux with appropriate package manager)
- **Homebrew** (for macOS)
- **Ollama** - [Install from ollama.com](https://ollama.com) or `brew install ollama`
- **Python 3.8+**
- **Jupyter Notebook** or JupyterLab

## Quick Start

### 1. Install Ollama

```bash
# macOS
brew install ollama

# Or download from https://ollama.com
```

Start Ollama:
```bash
ollama serve
```

### 2. Pull Required Models

For the RAG system, you'll need an embedding model:
```bash
ollama pull nomic-embed-text
```

For querying, pull any LLM model:
```bash
ollama pull llama3.2:latest
# or
ollama pull mistral:latest
```

### 3. Mirror the Website

1. Open `ub_cse_mirror_wget_workflow.ipynb`
2. Run the cells to download the website
3. This will create the `engineering.buffalo.edu/` folder

### 4. Query the Website

Choose one of two approaches:

#### Option A: Simple Query (Without RAG)
- Open `query_with_ollama.ipynb`
- Select a model by setting `MODEL_NAME = "model_1"` (or model_2, etc.)
- Run queries directly

#### Option B: RAG Query (Recommended for Large Datasets)
- Open `rag_query_website.ipynb`
- Build the vector database index (one-time, ~30-60 min)
- Query with semantic search across all files

## Project Structure

```
UB_CSE_mirror/
├── ub_cse_mirror_wget_workflow.ipynb  # Website mirroring workflow
├── query_with_ollama.ipynb            # Simple query system (no RAG)
├── rag_query_website.ipynb            # RAG-based query system
├── COMPARISON.md                       # Comparison of the two query approaches
├── README.md                           # This file
├── .gitignore                          # Git ignore rules
├── engineering.buffalo.edu/            # Mirrored website (4,042 HTML files)
└── chroma_db/                          # Vector database (created by RAG system)
```

## Notebooks Explained

### `ub_cse_mirror_wget_workflow.ipynb`

**Purpose**: Download and mirror the UB CSE website locally

**Features**:
- Uses `wget` to download website
- Configurable depth and scope
- Options to include/exclude specific content
- Creates offline-browsable mirror

**Usage**: Run cells sequentially to mirror the website.

### `query_with_ollama.ipynb`

**Purpose**: Simple query system without RAG

**Features**:
- Keyword-based file search
- Direct LLM querying
- Model selection via numbered labels (model_1, model_2, etc.)
- Quick setup, no indexing required

**Best for**: 
- Quick exploratory queries
- Small datasets
- When you know which files contain answers

**Limitations**:
- Only searches 3-5 files at a time
- Keyword-based (not semantic)
- Limited coverage

### `rag_query_website.ipynb`

**Purpose**: Advanced query system with RAG

**Features**:
- Semantic search across all files
- Vector database for fast retrieval
- Comprehensive coverage (all 4,042 files)
- Source citations
- One-time indexing, fast queries

**Best for**:
- Large datasets (100+ files)
- Production use
- Complex queries requiring multiple sources
- When you need comprehensive answers

**Setup**: Requires one-time indexing (~30-60 min for full dataset)

## Comparison

See `COMPARISON.md` for a detailed comparison of the two query approaches.

## Model Selection

Both query notebooks support model selection:

1. Available models are listed with labels: `model_1`, `model_2`, etc.
2. Set `MODEL_NAME = "model_1"` to select a model
3. The system automatically uses the selected model for queries

Example:
```python
MODEL_NAME = "model_2"  # Use the second model in the list
```

## Data Statistics

- **Total HTML files**: 4,042
- **Total size**: ~403 MB
- **Main sections**:
  - People: 1,885 files
  - News and Events: 1,505 files
  - Research: 87 files
  - Graduate: 87 files
  - And more...

## Troubleshooting

### Ollama Connection Issues

If you get connection errors:
```bash
# Check if Ollama is running
ollama list

# Start Ollama if not running
ollama serve
```

### Model Not Found

If a model isn't available:
```bash
# Pull the model
ollama pull <model-name>

# List available models
ollama list
```

### Vector Database Issues (RAG)

If the vector database isn't found:
1. Make sure you've run the indexing cells in `rag_query_website.ipynb`
2. Check that `chroma_db/` folder exists
3. Re-run the indexing if needed

### Memory Issues

Large models (like llama4:latest at 62GB) may cause memory issues:
- Use smaller models for testing
- Close other applications
- Consider using smaller models like `llama3.2:latest` (1.88 GB)

## Contributing

This is a personal project, but suggestions and improvements are welcome!

## License

This project is for educational and research purposes. The mirrored website content belongs to the University at Buffalo.

## Acknowledgments

- **Ollama** - For providing local LLM capabilities
- **ChromaDB** - For vector database functionality
- **University at Buffalo** - For the publicly available website content

## Resources

- [Ollama Documentation](https://github.com/ollama/ollama)
- [ChromaDB Documentation](https://www.trychroma.com/)
- [RAG (Retrieval Augmented Generation) Overview](https://www.pinecone.io/learn/retrieval-augmented-generation/)
