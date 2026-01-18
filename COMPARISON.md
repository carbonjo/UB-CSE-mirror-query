# Comparison: Query Without RAG vs. Query With RAG

This document explains the differences between the two query approaches for the UB CSE mirrored website.

## Overview

We have two notebooks for querying the mirrored website:

1. **`query_with_ollama.ipynb`** - Direct querying without RAG
2. **`rag_query_website.ipynb`** - Querying with RAG (Retrieval Augmented Generation)

## The Challenge

The mirrored website contains:
- **4,042 HTML files**
- **~403MB of data**
- Far exceeds any LLM's context window (even large models typically have 32K-128K token limits)

## Approach 1: Query Without RAG (`query_with_ollama.ipynb`)

### How It Works

1. **Direct File Search**: Uses keyword-based search on filenames/paths
2. **Limited File Selection**: Selects a small number of files (typically 3-5) based on filename matches
3. **Direct LLM Query**: Passes selected file contents directly to the LLM
4. **No Preprocessing**: No indexing or embedding step

### Code Flow

```python
# 1. Find files by keyword matching
relevant_files = [f for f in html_files if keyword in f.lower()]

# 2. Extract text from selected files
file_text = extract_text_from_html(file_path)

# 3. Query LLM directly with file content
response = ollama.generate(model=model, prompt=f"Context: {file_text}\nQuestion: {query}")
```

### Limitations

❌ **Limited Coverage**: Only searches 3-5 files at a time  
❌ **Keyword-Based**: Relies on filename/path matching, not semantic understanding  
❌ **No Context Ranking**: Can't determine which files are most relevant  
❌ **Inefficient**: Processes entire files even if only small portions are relevant  
❌ **Misses Relevant Content**: If keywords don't match filenames, relevant content is missed  
❌ **No Persistence**: Must search files every time  

### Example Scenario

**Query**: "What are the research areas in computer science?"

**Process**:
1. Searches for files with "research" or "computer" in filename
2. Finds: `research.html`, `research-areas.html`
3. Reads entire content of these 2-3 files
4. Passes all content to LLM
5. Gets answer based only on those files

**Problem**: Might miss relevant information in other files like `faculty.html` or `about-us.html` that discuss research but don't have "research" in the filename.

### When to Use

✅ Good for:
- Quick, exploratory queries
- When you know which specific files contain the answer
- Testing the setup
- Small datasets (< 100 files)

❌ Not ideal for:
- Large datasets (like our 4,042 files)
- Complex queries requiring information from multiple sources
- When you need comprehensive answers
- Production use cases

---

## Approach 2: Query With RAG (`rag_query_website.ipynb`)

### How It Works

1. **Indexing Phase** (one-time setup):
   - Extracts text from all HTML files
   - Chunks text into smaller pieces (1000 chars with 200 overlap)
   - Creates embeddings (vector representations) for each chunk
   - Stores embeddings in ChromaDB vector database

2. **Query Phase** (fast, ~5-10 seconds):
   - Converts query to embedding
   - Performs semantic search to find most relevant chunks
   - Retrieves top-k most relevant chunks
   - Passes only relevant chunks to LLM
   - Returns answer with source citations

### Code Flow

```python
# INDEXING (one-time)
# 1. Create chunks from all files
chunks = create_chunks_from_files(html_files)

# 2. Create embeddings and store in vector DB
for chunk in chunks:
    embedding = get_embedding(chunk['text'])
    collection.add(embeddings=[embedding], documents=[chunk['text']])

# QUERYING
# 1. Convert query to embedding
query_embedding = get_embedding(query)

# 2. Semantic search in vector DB
results = collection.query(query_embeddings=[query_embedding], n_results=5)

# 3. Get top-k relevant chunks
context = results['documents'][0]

# 4. Query LLM with relevant context
response = ollama.generate(model=model, prompt=f"Context: {context}\nQuestion: {query}")
```

### Advantages

✅ **Semantic Understanding**: Finds relevant content based on meaning, not just keywords  
✅ **Comprehensive Coverage**: Searches across all 4,042 files  
✅ **Efficient**: Only retrieves and processes relevant chunks  
✅ **Ranked Results**: Returns most relevant content first  
✅ **Source Citations**: Shows which files/chunks were used  
✅ **Persistent Index**: Build once, query many times  
✅ **Scalable**: Works with datasets of any size  

### Example Scenario

**Query**: "What are the research areas in computer science?"

**Process**:
1. Converts query to embedding vector
2. Searches vector database for semantically similar chunks
3. Finds relevant chunks from:
   - `research-areas.html`
   - `research.html`
   - `faculty.html` (mentions research)
   - `about-us.html` (discusses research focus)
   - `sitemap.html` (links to research pages)
4. Retrieves top 5 most relevant chunks
5. Passes only these chunks to LLM
6. Gets comprehensive answer with sources

**Result**: More complete answer drawing from multiple relevant sources across the entire website.

### When to Use

✅ Ideal for:
- Large datasets (100+ files)
- Production applications
- Complex queries requiring multiple sources
- When you need comprehensive, accurate answers
- When you want source citations

❌ Overkill for:
- Very small datasets (< 50 files)
- Simple, single-file queries
- One-off exploratory queries

---

## Side-by-Side Comparison

| Feature | Without RAG | With RAG |
|---------|-------------|----------|
| **Setup Time** | Immediate | 30-60 min (one-time) |
| **Query Speed** | Fast (~2-5 sec) | Fast (~5-10 sec) |
| **File Coverage** | 3-5 files | All files (4,042) |
| **Search Method** | Keyword matching | Semantic search |
| **Relevance Ranking** | No | Yes |
| **Context Efficiency** | Low (entire files) | High (relevant chunks only) |
| **Source Citations** | Basic | Detailed |
| **Scalability** | Poor (linear slowdown) | Excellent (constant time) |
| **Accuracy** | Variable | High |
| **Storage** | None | Vector DB (~100-200MB) |

## Technical Differences

### Search Strategy

**Without RAG**:
```python
# Simple keyword matching
if "research" in file_path.lower():
    relevant_files.append(file_path)
```

**With RAG**:
```python
# Semantic similarity search
query_embedding = get_embedding("research areas")
results = vector_db.query(query_embeddings=[query_embedding])
# Returns chunks semantically similar to query
```

### Context Management

**Without RAG**:
- Reads entire files (could be 10K+ characters)
- No chunking or optimization
- May exceed context window for large files

**With RAG**:
- Chunks files into 1000-character pieces
- Only retrieves relevant chunks
- Stays within context window limits

### Data Processing

**Without RAG**:
- Processes files on-demand
- No preprocessing
- No persistence

**With RAG**:
- Preprocesses all files once
- Creates searchable index
- Persists in vector database

## Performance Comparison

### Query: "What undergraduate programs are offered?"

**Without RAG**:
- Files searched: 3-5 (by filename match)
- Time: ~3 seconds
- Result: May miss programs mentioned in other files

**With RAG**:
- Files searched: All 4,042 (via semantic search)
- Time: ~8 seconds
- Result: Comprehensive answer from multiple relevant sources

## Memory and Storage

### Without RAG
- **Memory**: Minimal (only loads files during query)
- **Storage**: None
- **Disk I/O**: High (reads files every query)

### With RAG
- **Memory**: Moderate (vector DB in memory)
- **Storage**: ~100-200MB (vector database)
- **Disk I/O**: Low (indexed data, fast lookups)

## Recommendation

### Use **Without RAG** (`query_with_ollama.ipynb`) when:
- You're exploring the dataset
- You know exactly which files contain answers
- You have < 100 files
- You need quick, one-off queries
- You don't want to set up indexing

### Use **With RAG** (`rag_query_website.ipynb`) when:
- You have a large dataset (100+ files)
- You need comprehensive, accurate answers
- You want source citations
- You'll query multiple times
- You need production-quality results
- You want semantic understanding

## For This Project (4,042 files)

**Recommendation: Use RAG** (`rag_query_website.ipynb`)

The dataset is too large for effective keyword-based search. RAG provides:
- Better coverage across all files
- More accurate, comprehensive answers
- Source citations for verification
- Efficient querying despite dataset size

The one-time indexing cost (30-60 minutes) is worth it for the improved query quality and efficiency.

## Summary

| Aspect | Winner |
|--------|--------|
| **Setup Complexity** | Without RAG (simpler) |
| **Query Quality** | With RAG (better) |
| **Coverage** | With RAG (comprehensive) |
| **Speed** | Tie (both fast) |
| **Scalability** | With RAG (much better) |
| **Production Ready** | With RAG (yes) |

**Bottom Line**: For large datasets like this (4,042 files), RAG is the clear winner. For small, exploratory queries, the simpler approach works fine.
