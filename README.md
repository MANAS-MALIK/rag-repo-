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


RAG-Based Text-to-SQL Project
Complete Interview Preparation Guide

Covers: RAG, Embeddings, FAISS, Gemini, Groq, LangChain vs No LangChain



 
SECTION 1: What is RAG?
RAG stands for Retrieval-Augmented Generation. It is a technique where instead of asking an LLM to answer from its training data alone, you first RETRIEVE relevant documents from a database, then pass them to the LLM to GENERATE a grounded answer.

The Problem RAG Solves
•	LLMs hallucinate — they make up answers when they don't know
•	LLMs have a knowledge cutoff — they don't know recent data
•	LLMs don't know YOUR company's private data

RAG Flow (Simple)
•	Step 1: Store your documents as vectors in a database
•	Step 2: User asks a question
•	Step 3: Convert question to vector, find similar documents
•	Step 4: Send retrieved documents + question to LLM
•	Step 5: LLM answers based on REAL data, not guesswork

Our Project Use Case
We have 20+ SQL queries written for existing clients (Kate, Nike). When a new client (Coach) comes in, instead of writing SQL from scratch, RAG retrieves the most similar existing query and the LLM modifies it. This prevents hallucination and reuses proven logic.

Without RAG	With RAG
LLM guesses SQL from scratch	LLM uses proven existing query
High chance of hallucination	Grounded in real SQL logic
Inconsistent across clients	Consistent, reusable patterns
No reference to past work	Learns from all past queries


 
SECTION 2: Key Concepts Explained
2.1 Embeddings
An embedding is a way to convert text into a list of numbers (a vector) that captures the MEANING of the text. Similar sentences produce similar vectors, even if the words are different.

Example:
"decline in foot traffic"    → [0.12, -0.45, 0.88, ...]
"performance drop"           → [0.10, -0.40, 0.91, ...]
These two phrases mean the same thing. Their vectors are close to each other. This is called semantic similarity. Keywords like 'decline' and 'drop' are different words, but embeddings capture that they mean the same thing.

Why we used Gemini Embeddings:
•	Model: models/gemini-embedding-001
•	Free to use with a Google account
•	Produces 768-dimensional vectors
•	Good quality semantic understanding

2.2 Vector Database (FAISS)
FAISS (Facebook AI Similarity Search) is a library that stores vectors and lets you search for the most similar ones very fast. Think of it as a database, but instead of searching by ID or text, you search by meaning.

How FAISS works in our project:
•	We store each SQL query as a vector in FAISS
•	When a user asks for new SQL, we convert their request to a vector
•	FAISS finds the closest matching SQL vector
•	We retrieve that SQL and pass it to the LLM

We used IndexFlatL2 — this does exact search using L2 (Euclidean) distance. It checks every vector and finds the closest match. Good for small datasets like ours.

2.3 page_content vs metadata
In our project (and in LangChain), every document has two parts:

Part	What it contains	How it is used
page_content	The actual SQL query text	Gets embedded → semantic search
metadata	client name, query name	Used for filtering → exact match

Example: When Coach asks for SKU Status SQL, we first filter by query_name = 'SKU Status' (metadata filter), then among those results we find the most semantically similar one (embedding search). Metadata narrows down, embeddings rank by relevance.

2.4 Why NOT LangChain?
LangChain is a popular framework that provides ready-made tools for building RAG systems. However, we built our RAG pipeline from scratch without LangChain. Here is why:

Aspect	LangChain	Our Approach (No LangChain)
Control	Abstracted, hard to debug	Full control over every step
Flexibility	Limited by their API	Customise anything
Learning value	Hides the internals	You understand every line
Production	Can be slow/complex	Optimised for your needs
Interview value	Hard to explain internals	Easy to explain every step

Key interview answer: LangChain is just an orchestration wrapper. It does not give new capabilities — it just reduces boilerplate code. Our project shows we understand the internals, which is more impressive.

 
SECTION 3: Tools & Libraries Used
3.1 Google Gemini (Embeddings)
We used Gemini for converting text to vectors (embeddings). Originally we tried OpenAI, but it is paid. Gemini offers a free tier via Google AI Studio.

Detail	Value
Model used	models/gemini-embedding-001
Output	768-dimensional vector
Cost	Free (with Google account)
Get key at	aistudio.google.com/app/apikey

Why not text-embedding-004?
We first tried text-embedding-004 and gemini-embedding-2, but got 404 errors. We ran list_models() to find available models and found gemini-embedding-001 works. Always verify model availability before assuming.

3.2 Groq (LLM for SQL Generation)
We used Groq to run the LLaMA model for generating modified SQL. Originally the plan used OpenAI GPT-4, but that is paid. We then tried Gemini Flash for generation, but hit quota limits (limit: 0) due to regional restrictions in India.

Detail	Value
Model used	llama-3.1-8b-instant
Provider	Groq (runs LLaMA on fast hardware)
Cost	Free tier — 1500 requests/day
Speed	Very fast (Groq uses custom chips)
Get key at	console.groq.com

Why Groq specifically?
Groq is not a model company — they make inference hardware (LPUs) that run open-source models like LLaMA extremely fast. The API is compatible with OpenAI's format, so it was easy to swap in. No region restrictions for free tier unlike Gemini.

3.3 FAISS
Facebook AI Similarity Search — an open source library for fast vector similarity search. We used it as our local vector database. No server needed, runs entirely in memory. Perfect for demo/interview projects.

3.4 python-docx
Used to read the .docx file uploaded by the user. It lets us loop through paragraphs and extract SQL queries along with their client/query metadata from headings.

 
SECTION 4: Code Walkthrough — Line by Line
Step 1: Install Libraries
!pip install python-docx google-generativeai faiss-cpu numpy groq -q
•	python-docx: reads .docx Word files
•	google-generativeai: Gemini SDK for embeddings
•	faiss-cpu: vector database, CPU version (no GPU needed)
•	numpy: array math for vectors
•	groq: client for calling LLaMA via Groq API

Step 2: Configure APIs
import google.generativeai as genai
genai.configure(api_key=GEMINI_API_KEY)
This sets your Gemini API key globally. All subsequent Gemini calls use this key automatically.

from groq import Groq
groq_client = Groq(api_key=GROQ_API_KEY)
Creates a Groq client object. We use this later to call LLaMA for SQL generation.

Step 3: Parse the .docx File
doc = Document(filename)
Loads the Word document into memory using python-docx.

for para in doc.paragraphs:
    if 'Client:' in text and 'Query:' in text:
We loop through every paragraph. If it looks like a heading (contains 'Client:' and 'Query:'), we treat it as metadata for the next SQL block. Otherwise we collect the text as SQL content.

parts = text.split('|')
current_meta = { 'client': parts[0].replace('Client:', '').strip(), ... }
We split the heading on the pipe character to extract client name and query name separately.

Step 4: Create Documents (page_content + metadata)
doc_entry = {
    'page_content': f"Query: {q['query_name']}\n{q['sql']}",
    'metadata': { 'client': q['client'], 'query_name': q['query_name'] }
}
We combine query name + SQL into page_content. This is what gets embedded. The metadata stays separate for filtering. We include query_name in page_content so the embedding captures the intent, not just raw SQL syntax.

Step 5: Generate Embeddings
def get_embedding(text):
    result = genai.embed_content(
        model='models/gemini-embedding-001',
        content=text
    )
    return result['embedding']
This function takes any text and returns a 768-number vector. We call this for every document's page_content. The vector captures the semantic meaning of the SQL query.

Step 6: Store in FAISS
vectors = np.array([d['embedding'] for d in documents], dtype='float32')
Stack all embedding vectors into a 2D numpy array. Shape = (number of documents, 768).

index = faiss.IndexFlatL2(dim)
index.add(vectors)
Create a FAISS index with L2 distance metric. Add all vectors. Now FAISS can search across all of them instantly.

Step 7: Retrieval Function
def retrieve(user_prompt, filter_query_name=None, top_k=3):
This is the core RAG function. It does two things:
•	Metadata filter: keeps only documents where query_name matches
•	Embedding search: converts the user prompt to a vector, finds closest match in FAISS

query_vector = np.array([get_embedding(user_prompt)], dtype='float32')
Convert the user's prompt to a vector so we can compare it against stored SQL vectors.

distances, indices = sub_index.search(query_vector, k)
FAISS returns the k closest vectors. distances = how far, indices = which documents. Lower distance = more similar.

Step 8: Generate SQL with LLM
prompt = f"""You are a SQL expert.
Reference SQL: {retrieved_doc['page_content']}
Generate modified SQL for: {new_client}
Changes: {modifications}"""
We build a prompt that includes the retrieved SQL as context. The LLM does NOT generate from scratch — it modifies the real, proven SQL. This is the core RAG advantage.

response = groq_client.chat.completions.create(
    model='llama-3.1-8b-instant',
    messages=[{'role': 'user', 'content': prompt}]
)
Call Groq's API. It runs LLaMA 3.1 8B model and returns the modified SQL. The response format is identical to OpenAI's API, just with a different client object.

 
SECTION 5: Expected Interview Questions & Answers
Q1: What is RAG and why did you use it?
RAG is Retrieval-Augmented Generation. Instead of asking an LLM to generate SQL from scratch (which causes hallucination and inconsistency), we first retrieve the most relevant existing SQL query from our database, then ask the LLM to modify it. This gives grounded, consistent, and correct SQL every time.

Q2: What is an embedding?
An embedding is a numerical representation of text as a vector (list of numbers). Similar meaning = similar vectors. We use embeddings to compare a new user request against stored SQL queries by measuring vector distance. This enables semantic search — finding matches by meaning, not just keywords.

Q3: What is FAISS?
FAISS stands for Facebook AI Similarity Search. It is an open-source library that stores vectors and retrieves the most similar ones very efficiently. We use it as our vector database. IndexFlatL2 does exact L2 (Euclidean) distance search across all stored vectors.

Q4: Why not use LangChain?
LangChain is an orchestration framework — it provides ready-made components but hides the internals. By building from scratch, I have full control, better debuggability, and I understand every step. For production systems, this matters. LangChain is useful for quick prototypes but not always ideal for custom production logic.

Q5: What is the difference between metadata filtering and embedding search?
Metadata filtering is exact matching — like a SQL WHERE clause. It narrows down candidates by structured fields like client name or query type. Embedding search is semantic matching — it finds documents with similar meaning. In our pipeline, we first filter by query_name (metadata) to narrow the search space, then rank by embedding similarity to find the best match. Both are needed.

Q6: Why Gemini for embeddings and Groq for generation?
Both are free. OpenAI requires payment. We tried Gemini Flash for generation but hit regional quota limits (India). Groq has no such restrictions on the free tier and is actually faster due to their custom LPU hardware. Gemini embeddings work well and are stable. Using the right free tool for each job is a practical production decision.

Q7: Is page_content or metadata embedded?
By default, only page_content is embedded. Metadata is not converted to vectors — it is used separately for filtering through exact match logic. We include important metadata fields like query_name inside page_content so the embedding captures that information semantically as well.

Q8: Can this scale to 1000+ queries?
Yes. FAISS scales well to millions of vectors. For larger datasets, we would use FAISS with IVF (Inverted File Index) for approximate search which is faster at scale. We would also move to a persistent vector database like Pinecone or Weaviate instead of in-memory FAISS. The metadata filtering reduces the search space significantly, which helps with scale.

 
SECTION 6: Complete Flow Summary

Step	What happens	Tool used
1. Load file	Read .docx, extract SQL + client info	python-docx
2. Create documents	Structure as page_content + metadata	Plain Python
3. Embed	Convert SQL text to 768-dim vectors	Gemini Embedding
4. Store	Save vectors in searchable index	FAISS
5. User request	New client asks for modified SQL	—
6. Filter	Keep only matching query_name docs	Metadata filter
7. Embed query	Convert user request to vector	Gemini Embedding
8. Search	Find closest SQL vector in FAISS	FAISS similarity
9. Build prompt	Combine retrieved SQL + instructions	Plain Python
10. Generate	LLM modifies SQL for new client	Groq / LLaMA 3.1


Key interview quote: We retrieve proven SQL first, then generate modifications — RAG eliminates hallucination.

<img width="504" height="181" alt="image" src="https://github.com/user-attachments/assets/ba34ceeb-74fe-4db6-a74f-8a33767183fe" />


