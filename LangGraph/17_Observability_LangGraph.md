# Lecture Notes: Adding **Observability** to a LangGraph Chatbot using **LangSmith**

---

## 1. Recap of the Playlist So Far

* Started with **theory**:

  * What is *Agentic AI*?
  * What is *LangGraph* and why is it needed?
* Then moved to **practical learning**:

  * Fundamentals of LangGraph.
  * Building different types of **workflows**.
* Built a **Chatbot Project** step by step:

  1. Basic chatbot functionality.
  2. Added a **GUI** for user interaction.
  3. Added **streaming** for real-time LLM responses.
  4. Added **database persistence** (chat history is saved even after shutdown).

At this stage:
👉 Chatbot supports GUI + Streaming + Persistence.
Today’s goal: **Add Observability with LangSmith**.

---

## 2. What is Observability?

* **Definition in LLM context**:
  Ability to **trace execution end-to-end** and monitor internal behaviors.

  * Capture all **user inputs** and **model outputs**.
  * Track **token usage**, **latency**, **system execution flow**.
* **Tool used**: [LangSmith](https://smith.langchain.com/).
* Benefit:

  * Debugging and monitoring chatbot behavior.
  * Essential for **future features** like Tools, RAG, MCP, etc.
  * Makes system ready for **production deployment**.

---

## 3. Setting Up LangSmith Integration

### Step 1: Create LangSmith Account

* Go to: **smith.langchain.com**.
* Create/Login into your account.

### Step 2: Generate API Key

* Navigate to **Settings → API Keys**.
* Click **Create API Key**, give description, copy the key.

### Step 3: Add Environment Variables

* Create or update your **`.env` file** inside project folder.

```bash
LANGCHAIN_TRACING_V2="true"
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY="your_api_key_here"
LANGCHAIN_PROJECT="chatbot_project"
```

* Explanation:

  * `LANGCHAIN_TRACING_V2` → Enables tracing.
  * `LANGCHAIN_ENDPOINT` → Where logs are sent.
  * `LANGCHAIN_API_KEY` → Authentication with LangSmith.
  * `LANGCHAIN_PROJECT` → Organizes traces into a specific project.

✅ Once set, LangSmith **automatically starts tracing** your project, no code changes needed.

---

## 4. Running the Chatbot with LangSmith

* Re-run the **same chatbot code** (no modifications).
* LangSmith automatically captures:

  * **Project** = "chatbot\_project".
  * **Trace** = Each user → chatbot turn.
  * Details: input, output, execution time, tokens used, latency, etc.

Example:

* User: `"Give me a roadmap to study AI Engineering."`
* Chatbot: *(response)*
* In LangSmith:

  * A new **trace** is logged.
  * Shows node name (`chat_node`), model used (`ChatOpenAI`), timestamps, tokens, latency, etc.

Each **conversation turn = one trace**.

---

## 5. Problem: Mismanaged Conversations

* By default:
  **All turns (even from different conversations/threads)** are stored in the same project → hard to separate.
* Example:

  * Thread 1: AI syllabus Q\&A.
  * Thread 2: Recipe of Biryani.
  * Both appear in **same trace list** without separation.
* Need: **Organize by conversation threads**.

---

## 6. Solution: Add **Thread Support**

* LangSmith allows traces to be grouped by **Threads** (or Sessions / Conversation IDs).

### Step 1: Add Thread Metadata

Modify the `config` dictionary in chatbot code:

**Old config:**

```python
config = {
    "configurable": {"thread_id": session["thread_id"]}
}
```

**New config with metadata:**

```python
config = {
    "configurable": {"thread_id": session["thread_id"]},
    "metadata": {
        "thread_id": session["thread_id"],   # Group traces into threads
        "run_name": "chat_turn"              # Rename trace for readability
    }
}
```

* `thread_id`: ensures each conversation is stored inside its **own thread**.
* `run_name`: replaces default `"LangGraph"` with `"chat_turn"` (clearer).

### Step 2: Run Again

1. Delete old project in LangSmith dashboard (optional, for clarity).
2. Restart chatbot → start chatting.

---

## 7. Observing Organized Threads in LangSmith

* In **Traces View**: see all turn-by-turn logs.
* In **Threads View**:

  * Each **conversation = one thread**.
  * Inside thread: each **turn = one trace**.

Example:

* Thread 1:

  * User: "Hi" → Bot: "Hello, how can I help you?"
  * User: "My name is Nitesh" → Bot: "Nice to meet you, Nitesh."
* Thread 2:

  * User: "Hi, my name is Rahul" → Bot: "Hello Rahul."
  * User: "What is the roadmap to study AI?" → Bot: *(response)*

✅ Each conversation is **separated and well-organized**.

---

## 8. Benefits of Observability with Threads

* Clear separation of **multiple conversations**.
* Easy debugging:

  * Check what question was asked.
  * Verify model response.
  * Analyze latency + token usage.
* Useful for:

  * Research & development.
  * Production monitoring.
  * Scaling chatbot with advanced features.

---

## 9. What’s Next

* Current lecture covered:

  * Observability basics.
  * LangSmith integration.
  * Threads for structured conversation logging.
* Future videos will cover:

  * Advanced **LangSmith features**:

    * Monitoring dashboards.
    * Datasets & experiments.
    * Prompt management.
    * Playground testing.
  * Adding **complex features** in chatbot:

    * Tools.
    * Retrieval-Augmented Generation (RAG).
    * MCP integration.

---

## 10. Key Code Snippets

### `.env` setup

```bash
LANGCHAIN_TRACING_V2="true"
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY="your_api_key_here"
LANGCHAIN_PROJECT="chatbot_project"
```

### Config with Threads + Metadata

```python
config = {
    "configurable": {"thread_id": session["thread_id"]},
    "metadata": {
        "thread_id": session["thread_id"],
        "run_name": "chat_turn"
    }
}
```

---

# 🔑 Takeaways

* **Observability** = tracing and monitoring LLM systems.
* **LangSmith** provides end-to-end monitoring.
* Setup requires only environment variables → **minimal code changes**.
* To properly separate conversations, use **thread\_id**.
* Each **conversation → thread**, each **turn → trace**.
* Helps in debugging, scaling, and deploying AI chatbots in production.
