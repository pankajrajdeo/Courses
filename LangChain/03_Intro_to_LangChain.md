# Lecture Notes: Introduction to LangChain

---

## 1. Introduction

* LangChain is an **open-source framework** for building applications powered by **Large Language Models (LLMs)**.
* It helps connect all the moving parts needed to make an LLM-powered application work.
* Before diving into “What is LangChain,” we first ask **why do we need it**?

---

## 2. Why Do We Need LangChain?

### Problem Context

* Imagine building a **PDF reading + chat application** (like “Chat with your book”).
* A user uploads a PDF (say, a machine learning textbook).
* The user should be able to:

  * Ask: “Explain page 5 as if I’m 5 years old.”
  * Ask: “Generate true/false questions from the linear regression chapter.”
  * Ask: “Summarize the decision tree section.”

This requires **more than just reading text**. It requires:

* **Understanding queries** (NLU).
* **Searching relevant parts** (semantic search).
* **Generating contextual answers** (NLG).

---

## 3. High-Level System Design

### Workflow

1. **User uploads PDF** → stored in database/cloud.
2. **User asks a query** → e.g., “What are the assumptions of linear regression?”
3. **Semantic Search**:

   * Find relevant pages/paragraphs in the PDF.
   * Better than keyword search (retrieves meaningful context).
4. **Send query + retrieved pages to Brain (LLM)**.
5. **Brain responsibilities**:

   * **Understand query (NLU)**.
   * **Generate context-aware response (NLG)**.
6. **Return answer to user**.

---

## 4. Why Not Give Whole Book to the LLM?

* Giving 1000 pages is inefficient and costly.
* Like asking a teacher:

  * *I have a doubt in this book* (inefficient).
  * vs. *I have a doubt on page 155* (efficient).
* Solution: **semantic search narrows down content first**, then LLM processes it.

---

## 5. How Semantic Search Works

1. Convert text into **embeddings (vectors)**.

   * E.g., use Word2Vec, Doc2Vec, or Transformer embeddings.
   * Each document/page = a vector in N-dimensional space.
2. Convert user query into a vector.
3. Compare query vector with document vectors using **similarity** (cosine similarity).
4. Retrieve top-N closest vectors → relevant text chunks.

### Visual Analogy

* Imagine 3 paragraphs about Virat Kohli, Jasprit Bumrah, Rohit Sharma.
* Query: *“How many runs has Virat scored?”*
* Convert all text + query into vectors → Virat’s paragraph will be closest.

---

## 6. Low-Level Implementation Flow

### Steps

1. **Store PDF** (e.g., AWS S3).
2. **Load document** using a **Document Loader**.
3. **Split text into chunks** (per page, paragraph, or section) using **Text Splitter**.
4. **Generate embeddings** for each chunk.
5. **Store embeddings in a Vector Database** (e.g., Pinecone, FAISS, Weaviate).
6. When user asks a query:

   * Convert query → embedding.
   * Search vector DB for similar embeddings.
   * Retrieve top-k chunks.
7. Combine: **user query + retrieved chunks → prompt for LLM**.
8. LLM generates **final contextual answer**.

### Pseudocode Example

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.llms import OpenAI
from langchain.chains import RetrievalQA

# 1. Load PDF
loader = PyPDFLoader("ml_book.pdf")
documents = loader.load()

# 2. Split text into chunks
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(documents)

# 3. Generate embeddings
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(chunks, embeddings)

# 4. Setup LLM with Retrieval
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    retriever=vectorstore.as_retriever()
)

# 5. Ask questions
query = "What are the assumptions of linear regression?"
answer = qa.run(query)
print(answer)
```

---

## 7. Challenges in Building This System

1. **Brain (LLM) challenge**

   * Requires NLU + context-aware text generation.
   * Solved today by **LLMs (e.g., GPT, Claude, LLaMA, etc.)**.

2. **Computation challenge**

   * Hosting large LLMs is expensive.
   * Solution: Use **LLM APIs** (OpenAI, Anthropic, etc.).
   * Pay-per-use model saves cost.

3. **Orchestration challenge**

   * Multiple components:

     * Cloud storage
     * Text splitter
     * Embedding model
     * Vector DB
     * LLM API
   * Coordinating them manually is hard.
   * **This is where LangChain helps.**

---

## 8. What LangChain Provides

### Benefits

1. **Chains**:

   * Define pipelines of tasks (loading → splitting → embedding → retrieval → LLM).
   * Support sequential, parallel, and conditional chains.

2. **Model Agnostic**:

   * Swap models easily (OpenAI ↔ Anthropic ↔ Local LLM).

3. **Rich Ecosystem**:

   * Document loaders (PDF, Word, web pages, cloud).
   * Many text splitters.
   * Support for multiple embedding models + vector DBs.

4. **Memory & State Handling**:

   * Stores conversation context.
   * Enables follow-up questions without repeating the whole query.

---

## 9. What Can You Build with LangChain?

1. **Conversational Chatbots**

   * Customer support bots to reduce call center load.
   * First-line communication before human agents.

2. **Knowledge Assistants**

   * Domain-specific Q\&A bots (e.g., course material, internal docs).

3. **AI Agents**

   * Bots that **take actions**, not just chat.
   * Example: Booking flights, scheduling meetings.

4. **Workflow Automation**

   * Automate repetitive tasks in business processes.

5. **Summarization & Research Helpers**

   * Summarize large documents or research papers.
   * Solve **context-length limitation** of tools like ChatGPT.

---

## 10. Alternatives to LangChain

* **LlamaIndex** (formerly GPT Index):

  * Focused on indexing and retrieval pipelines.

* **Haystack**:

  * Another framework for building LLM-based apps.

* Choice depends on pricing, ecosystem fit, and company preference.

---

## 11. Conclusion

* **LangChain = Orchestrator** for LLM-powered apps.
* It simplifies integrating multiple components (storage, embeddings, retrieval, LLMs).
* Key strengths:

  * Easy orchestration via **chains**.
  * Rich ecosystem of plug-and-play tools.
  * Memory & state handling.
* Use cases range from chatbots → agents → summarization tools.
* Alternatives exist (LlamaIndex, Haystack), but LangChain is most popular.

---

✅ You now have both **conceptual clarity** and **practical code** to understand how LangChain works and why it’s important.
