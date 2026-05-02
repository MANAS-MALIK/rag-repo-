# RAG-Based Text-to-SQL Generator

Upload a `.docx` file containing existing SQL queries → RAG retrieves the best match → LLM adapts it for a new client.

## How it works

```
.docx (SQL queries)
    ↓ Parse
Documents (page_content + metadata)
    ↓ OpenAI Embeddings
FAISS Vector Store
    ↓ User prompt (new client)
Metadata filter + Semantic search
    ↓ Retrieved SQL
LLM modifies it → New SQL ✅
```

## Run on Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

1. Upload `rag_sql.ipynb` to Colab
2. Upload `sample_queries.docx` when prompted
3. Add your OpenAI API key in Step 2
4. Run all cells!

## Project Structure

```
├── rag_sql.ipynb          # Main notebook (run this)
├── sample_queries.docx    # Sample input file
└── README.md
```

## Key Concepts

| Concept | Role |
|---|---|
| `page_content` | SQL text → gets embedded |
| `metadata` | client, query_name → used for filtering |
| FAISS | Vector DB for similarity search |
| RAG | Retrieve real SQL, then modify (no hallucination) |

## Requirements

- Python 3.8+
- OpenAI API key
- Libraries: `openai`, `faiss-cpu`, `python-docx`, `numpy`

## .docx Format

Your document should have headings like:
```
Client: Kate | Query: SKU Status

SELECT sku, CASE WHEN ...
```
