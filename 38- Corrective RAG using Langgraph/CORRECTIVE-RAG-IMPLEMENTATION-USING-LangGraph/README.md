# This Repo Explains about Implementing Corrective RAG

## What Is Corrective RAG?

**Corrective RAG** is a smarter version of normal RAG.
Instead of always trusting the retrieved documents, it:

1. **Checks if the retrieved docs are relevant.**
2. **If not**, it:

   * Rewrites the user’s question.
   * Performs a web search.
   * Tries again with better info.

This way, it **corrects itself** before generating the final answer.

## Implementation Breakdown

### 1. **Document Indexing (Setup Stage)**

Before answering anything, the system:

* Loads a set of webpages (like blog articles).
* Splits the content into small chunks.
* Creates a searchable index (FAISS) using embeddings.

> This lets the system **quickly retrieve relevant chunks** later based on any question.

### 2. **Retrieval**

* When the user asks a question, the system **retrieves matching documents** from the FAISS index.

### 3. **Grading the Retrieved Docs**

* Each retrieved document is **graded by an LLM**.
* The LLM checks: *“Is this document relevant to the user’s question?”*
* Documents are marked either **“yes” (relevant)** or **“no” (not relevant)**.

> This is the **core of the correction**. It prevents bad or unrelated documents from being used.

### 4. **Decision Logic: Generate or Fix**

* If **some documents are relevant**, the system proceeds to **generate an answer**.
* If **all documents are irrelevant**, it **corrects** itself:

  * It **rewrites the original question** to make it more suitable for web search.

### 5. **Query Rewriting & Web Search**

* The question rewriter LLM improves the question.
* Then, a **web search tool** (like Tavily) is used to get fresh and relevant documents from the internet.

> Now the system has **better context** to work with, even if the original data didn’t help.

### 6. **Final Answer Generation**

* Whether the docs came from the original index or web search, the best documents are passed to an LLM.
* The LLM **generates the final answer** using those documents.


### 7. **All Controlled by LangGraph**

* Each of the above steps is implemented as a **node** in a LangGraph workflow.
* The graph decides the **next step** based on conditions (like “are docs relevant?”).


## Summary Flow

```
User Question
   ↓
Retrieve from Vector DB
   ↓
Grade Documents for Relevance
   ↓
Relevant? ── Yes ──► Generate Answer
     │
     └─ No ──► Rewrite Question
                  ↓
            Web Search with Rewritten Question
                  ↓
            Generate Answer
```

## Benefits of This Corrective RAG Design:

* Avoids hallucination by filtering irrelevant content.
* Dynamically improves queries if needed.
* Uses both offline (pre-indexed) and online (web) sources.
* Modular and scalable using LangGraph.