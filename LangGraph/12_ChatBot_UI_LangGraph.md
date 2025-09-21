# Lecture Notes: Building a Chatbot UI with **Streamlit** + **LangGraph**

## 1. Introduction

* In the last sessions, we built a chatbot using **LangGraph**.
* Features already implemented:

  * Working chatbot logic
  * Short-term memory (stores past messages)
* **Problem**: No UI (User Interface).

  * Interaction was only through Jupyter Notebook.
* **Today’s goal**:

  * Build a clean, simple **web-based UI** for the chatbot.
  * Use **Streamlit** (Python library for quick web apps).

---

## 2. The Plan of Action

1. **Split chatbot into two components:**

   * **Backend**:

     * Contains LangGraph workflow and memory logic.
   * **Frontend**:

     * Built using **Streamlit**.
     * Provides the chat interface (UI).
2. **Workflow**:

   * User interacts via frontend (website).
   * Frontend sends messages → backend (LangGraph).
   * Backend generates response → sent back to frontend.
   * Response is displayed in chat window.

---

## 3. Project Setup

* Project folder contains:

  1. **`langgraph_backend.py`**

     * Holds backend logic (LangGraph chatbot).
  2. **`streamlit_frontend.py`**

     * Holds frontend (UI with Streamlit).
  3. **`.env` file**

     * Stores **OpenAI API Key**.
  4. **Virtual environment** with dependencies installed:

     * `langchain`
     * `langgraph`
     * `streamlit`

---

## 4. Backend (`langgraph_backend.py`)

* Almost identical to the chatbot built in the earlier video.
* Key elements:

  * Import libraries
  * Define state
  * Build workflow graph
  * Add **checkpoint** with **in-memory saver** for short-term memory

### Example (simplified):

```python
from langgraph.graph import StateGraph
from langgraph.checkpoint.memory import MemorySaver
from langchain_openai import ChatOpenAI

# State definition
class State(dict):
    messages: list

# Create graph
graph = StateGraph(State)
graph.add_node("chat", ChatOpenAI())
graph.set_entry_point("chat")
graph.add_edge("chat", "chat")  # Loop for continuous conversation

# Memory saver (checkpoint)
memory = MemorySaver()
chatbot = graph.compile(checkpointer=memory)
```

This object `chatbot` will be imported by the frontend.

---

## 5. Frontend with Streamlit (`streamlit_frontend.py`)

### 5.1 Components of Chat UI

1. **Chat Messages**:

   * Displayed boxes showing:

     * User’s messages
     * Assistant’s responses
   * Built with:

     ```python
     st.chat_message("user").write("Hi")
     st.chat_message("assistant").write("Hello! How can I help you?")
     ```

2. **Chat Input**:

   * Input box at the bottom where user types.
   * Built with:

     ```python
     user_input = st.chat_input("Type here...")
     ```

---

### 5.2 First Example – Static Messages

```python
import streamlit as st

# User message
with st.chat_message("user"):
    st.text("Hi")

# Assistant message
with st.chat_message("assistant"):
    st.text("How can I help you?")
```

* Running command:

  ```bash
  streamlit run streamlit_frontend.py
  ```

---

### 5.3 Adding Input Box

```python
import streamlit as st

user_input = st.chat_input("Type your message here...")

if user_input:
    with st.chat_message("user"):
        st.text(user_input)
```

---

### 5.4 Problem: Old messages disappear

* Reason: **Streamlit reruns script** on every input.
* Solution: **Store history**.

---

### 5.5 Conversation History Management

* Use **Session State** (persistent dictionary across reruns).
* Initialize once:

```python
if "message_history" not in st.session_state:
    st.session_state["message_history"] = []
```

* Append new messages:

```python
st.session_state["message_history"].append({"role": "user", "content": user_input})
```

* Display all messages:

```python
for msg in st.session_state["message_history"]:
    with st.chat_message(msg["role"]):
        st.text(msg["content"])
```

---

### 5.6 Copycat Chatbot (Echo Bot)

* Assistant simply repeats user input.

```python
import streamlit as st

if "message_history" not in st.session_state:
    st.session_state["message_history"] = []

# Display past conversation
for msg in st.session_state["message_history"]:
    with st.chat_message(msg["role"]):
        st.text(msg["content"])

# Take new input
user_input = st.chat_input("Type your message here...")

if user_input:
    # User message
    st.session_state["message_history"].append({"role": "user", "content": user_input})
    with st.chat_message("user"):
        st.text(user_input)

    # Assistant echo message
    st.session_state["message_history"].append({"role": "assistant", "content": user_input})
    with st.chat_message("assistant"):
        st.text(user_input)
```

---

## 6. Integrating LangGraph Backend

* Replace the dummy assistant reply with **LangGraph response**.

### Steps:

1. **Import chatbot object** from backend:

   ```python
   from langgraph_backend import chatbot
   from langchain_core.messages import HumanMessage
   ```

2. **Invoke LangGraph**:

   ```python
   response = chatbot.invoke(
       {"messages": [HumanMessage(content=user_input)]},
       config={"configurable": {"thread_id": "thread1"}}
   )
   ```

3. **Extract assistant reply**:

   ```python
   ai_message = response["messages"][-1].content
   ```

4. **Store and display**:

   ```python
   st.session_state["message_history"].append({"role": "assistant", "content": ai_message})
   with st.chat_message("assistant"):
       st.text(ai_message)
   ```

---

### Final Integrated Frontend (simplified)

```python
import streamlit as st
from langgraph_backend import chatbot
from langchain_core.messages import HumanMessage

if "message_history" not in st.session_state:
    st.session_state["message_history"] = []

# Display past history
for msg in st.session_state["message_history"]:
    with st.chat_message(msg["role"]):
        st.text(msg["content"])

# Input
user_input = st.chat_input("Type your message here...")

if user_input:
    # User message
    st.session_state["message_history"].append({"role": "user", "content": user_input})
    with st.chat_message("user"):
        st.text(user_input)

    # AI response
    response = chatbot.invoke(
        {"messages": [HumanMessage(content=user_input)]},
        config={"configurable": {"thread_id": "thread1"}}
    )
    ai_message = response["messages"][-1].content

    st.session_state["message_history"].append({"role": "assistant", "content": ai_message})
    with st.chat_message("assistant"):
        st.text(ai_message)
```

---

## 7. Key Concepts Learned

1. **Splitting system into Backend + Frontend** is good design.
2. **Streamlit Components**:

   * `st.chat_message(role)` → Display messages.
   * `st.chat_input()` → Input box.
   * `st.session_state` → Persistent memory across reruns.
3. **LangGraph Integration**:

   * Requires **thread\_id** when using checkpoints.
   * `chatbot.invoke()` handles LLM interaction.
   * Responses must be extracted properly.

---

## 8. Summary

* We built a **full chatbot with UI**:

  * Backend: LangGraph (conversation + memory).
  * Frontend: Streamlit (UI with persistent chat history).
* Core challenges solved:

  * Persistent history with `session_state`.
  * Linking UI with backend responses.
* End result:

  * Functional, interactive chatbot on a **web UI**.
  * Supports memory, user identity, knowledge queries.

---

✅ Now you have a **complete roadmap + code** to revise and reproduce the chatbot.
