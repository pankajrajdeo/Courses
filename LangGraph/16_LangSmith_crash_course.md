# 📚 Lecture Notes: LangSmith – Observability for LLM Applications

---

## 1. Introduction

* **Instructor**: Nitesh (YouTube channel host)

* **Topic**: LangSmith – an observability and evaluation platform for LLM applications.

* **Prerequisites**:

  * Understanding of **LangChain**.
  * Basic knowledge of **LangGraph**.

* **Why this lecture?**

  * While studying LangGraph, the need arose to learn **observability** for debugging and monitoring LLM apps.
  * LangSmith helps **debug, test, and monitor** LLM applications.

---

## 2. Why Observability in LLM Apps?

### ⚠️ Key Problem

LLM applications behave like **black boxes**.

* Inputs go in, outputs come out.
* No visibility into **intermediate steps**.
* LLMs are **non-deterministic** → same input may produce different outputs.
* Debugging latency, cost, or hallucination issues is very difficult.

### 🔑 Solution

* Introduce **observability** → ability to see **inside the system**.
* Track inputs, outputs, intermediate steps, errors, token usage, latency, and costs.

---

## 3. Real-World Scenarios

### **Scenario 1: Startup Job Application Tool**

* App flow:

  1. Student uploads JD or job link.
  2. App analyzes JD with LLM.
  3. Fetches student resume & projects from Google Drive.
  4. Matches relevant skills.
  5. Generates and proofreads a tailored cover letter.
* Issue:

  * Initially: \~2 minutes per request.
  * Suddenly: 7–10 minutes.
  * No way to identify which **component** (JD analysis, resume fetch, cover letter, proofread) caused the slowdown.
* **Need**: observability to find the bottleneck.

---

### **Scenario 2: Research Assistant Agent**

* Agent fetches research papers (Google Scholar/Arxiv).
* Extracts key points, summarizes, generates report.
* **Cost issue**:

  * Usually: ₹0.50 per report.
  * Suddenly: some reports cost ₹2.00.
* Root cause:

  * A **prompt change** → agent kept re-running until “best” report created.
* Hard to debug since:

  * Not every report fails.
  * No clear error traces.
* **Need**: observability to trace where excess tokens/costs occur.

---

### **Scenario 3: RAG-based HR Chatbot**

* TCS internal chatbot for employees (leave policy, insurance, notice period).
* Uses RAG:

  * Query → Retriever → Relevant docs → LLM generates response.
* Problem:

  * Chatbot started **hallucinating** answers.
  * Employees received **wrong policies**.
* Possible failure points:

  1. **Retriever issue**: fetching irrelevant documents.
  2. **Generator issue**: hallucinating or ignoring context.
* Without observability:

  * Can’t see which component failed.
* **Need**: see retriever docs + final LLM input/output.

---

## 4. Observability: Formal Definition

> **Observability** = the ability to understand a system’s internal state by analyzing external outputs (logs, metrics, traces).

* Helps:

  * Diagnose issues.
  * Understand performance.
  * Improve reliability.
* Especially important in **non-deterministic LLM apps**.

---

## 5. What is LangSmith?

* **Unified Observability & Evaluation Platform**.
* Teams can:

  * Debug LLM apps.
  * Test workflows.
  * Monitor AI system performance.

### LangSmith Tracks:

* Input/output of every execution.
* Intermediate steps (retriever docs, prompts, parser outputs).
* Latency (per component + total).
* Token usage and cost.
* Errors at component level.
* Tags and metadata (custom or system-generated).
* User feedback (optional).

---

## 6. Core Concepts in LangSmith

* **Project**: Entire LLM application (e.g., a chatbot).
* **Trace**: One execution run of the app (input → output).
* **Run**: Execution of one **component** inside a trace.

Example:

```plaintext
User Input → Prompt → LLM → Parser → Output
```

* Project: the app.
* Trace: one complete input/output cycle.
* Runs: Prompt, LLM, Parser.

---

## 7. Practical Setup

### Steps

1. Clone repo:

   ```bash
   git clone <repo-url>
   ```
2. Open in VS Code.
3. Create & activate virtual environment:

   ```bash
   python -m venv myenv
   source myenv/bin/activate   # Mac/Linux
   myenv\Scripts\activate      # Windows
   ```
4. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```
5. Create `.env` file:

   ```env
   OPENAI_API_KEY=your_api_key
   LANGCHAIN_TRACING_V2=true
   LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
   LANGCHAIN_API_KEY=your_langsmith_api_key
   LANGCHAIN_PROJECT="LangSmith Demo"
   ```
6. Signup/login to LangSmith → generate API key.

---

## 8. Example 1: Simple LLM Call

### Code

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema import StrOutputParser

prompt = ChatPromptTemplate.from_template("What is the capital of {country}?")
model = ChatOpenAI()
parser = StrOutputParser()

chain = prompt | model | parser

print(chain.invoke({"country": "Peru"}))
```

### Observability Output

* Project: `LangSmith Demo`
* Trace:

  * Input: `"What is the capital of Peru?"`
  * Output: `"Lima"`
  * Latency: \~1.11s
  * Tokens + Cost
* Runs:

  1. Prompt template (input/output).
  2. LLM (input/output).
  3. Parser.

---

## 9. Example 2: Sequential Chain (Report + Summary)

### Code Highlights

* Generate report → Summarize in 5 points.
* Two different models:

  * GPT-4o-mini (temperature=0.7) for report.
  * GPT-4o (temperature=0.5) for summary.
* Add **tags, metadata, run name**.

```python
import os
os.environ["LANGCHAIN_PROJECT"] = "Sequential LLM App"

config = {
    "tags": ["LLM app", "report generation", "summarization"],
    "metadata": {
        "model1": "gpt-4o-mini",
        "temperature1": 0.7,
        "parser": "StrOutputParser"
    },
    "run_name": "Sequential Chain"
}
```

---

## 10. Example 3: RAG Application

### Problem with Naive RAG

* Only traced retriever + LLM.
* No tracing of:

  * PDF loading.
  * Chunking.
  * Embeddings.

### Fix with `@traceable`

* Wrap normal Python functions in **traceable decorators**.

```python
from langsmith import traceable

@traceable(name="Load PDF", tags=["pdf", "loader"], metadata={"loader": "PyPDFLoader"})
def load_pdf(path): ...

@traceable(name="Split Docs")
def split_docs(docs, chunk_size, chunk_overlap): ...

@traceable(name="Build Vector Store", metadata={"embedding_model": "text-embedding-3-small"})
def build_vector_store(docs): ...
```

* Output now shows:

  * PDF loading time (e.g., 15s).
  * Splitting details (chunk size, overlap).
  * Embedding model & cost.

---

## 11. Example 4: Improving Latency with Index Caching

* Store embeddings in **FAISS index**.
* First run: build index (slow).
* Subsequent runs: load index (fast).

### Key Code Idea

```python
if os.path.exists("index.faiss"):
    load_index()
else:
    build_index()
```

* First run: \~30s.
* Subsequent runs: \~1–2s.

---

## 12. Example 5: Agent with Tools (ReAct Framework)

* **Tools**:

  1. DuckDuckGo search.
  2. Weather API.
* Agent reasoning cycle:

  1. Prompt → Decide tool.
  2. Execute tool → Get observation.
  3. Add to scratchpad.
  4. Repeat until final answer.

### Observability

* Trace shows:

  * Scratchpad updates.
  * Prompts before each LLM call.
  * Tool inputs & outputs.
  * Final answer.
* Example Queries:

  * `"What is the release date of Dhadak 2?"`
  * `"What is the temperature in Gurgaon?"`
  * `"Find Kalpana Chawla’s birthplace and current temperature there."`

---

## 13. Benefits of LangSmith

* ✅ Debug latency issues (component-level timing).
* ✅ Debug cost issues (token-level tracking).
* ✅ Debug hallucinations (see retriever docs + prompts).
* ✅ Add searchable metadata & tags.
* ✅ Trace normal Python functions (not just LangChain runnables).
* ✅ Make black-box LLM apps transparent.

---

# 📝 Summary

* **LLM apps** are complex and non-deterministic → debugging is hard.
* **Observability** makes systems debuggable.
* **LangSmith**:

  * Projects → Traces → Runs.
  * Tracks inputs, outputs, tokens, cost, latency, errors.
* **Integration**:

  * Works automatically with LangChain/LangGraph.
  * For normal functions → use `@traceable`.
* **Use Cases**:

  * Sequential LLM chains.
  * RAG applications.
  * Agents with tools.

---

👉 These notes now cover **theory, practical examples, and code snippets** so you can revise the entire lecture in one go.
