# Git Repository Setup

This file contains instructions for setting up this project as a git repository.

## Initial Setup

If git is not already initialized, run:

```bash
cd /Users/carbonjo/Documents/UB_CSE_mirror
git init
```

## First Commit

After initializing, you can make your first commit:

```bash
# Add all files (respecting .gitignore)
git add .

# Make initial commit
git commit -m "Initial commit: UB CSE website mirror and query system"
```

## Recommended: Add Remote Repository

If you want to push to GitHub, GitLab, or another remote:

```bash
# Add remote (replace with your repository URL)
git remote add origin <your-repo-url>

# Push to remote
git branch -M main
git push -u origin main
```

## What's Included

The `.gitignore` file is configured to exclude:
- Python cache files (`__pycache__/`, `*.pyc`)
- Virtual environments (`venv/`, `env/`)
- Jupyter checkpoints (`.ipynb_checkpoints/`)
- Vector database (`chroma_db/`)
- Log files (`*.log`)
- OS files (`.DS_Store`, `Thumbs.db`)
- IDE files (`.vscode/`, `.idea/`)

**Note**: The `engineering.buffalo.edu/` folder (mirrored website) is currently tracked. If you don't want to commit 4,000+ HTML files, uncomment the line in `.gitignore`:

```
# engineering.buffalo.edu/
```

## Repository Structure

```
UB_CSE_mirror/
├── .gitignore                          # Git ignore rules
├── README.md                           # Main documentation
├── COMPARISON.md                       # Comparison of query approaches
├── SETUP.md                            # This file
├── ub_cse_mirror_wget_workflow.ipynb  # Website mirroring
├── query_with_ollama.ipynb            # Simple query system
└── rag_query_website.ipynb            # RAG query system
```
