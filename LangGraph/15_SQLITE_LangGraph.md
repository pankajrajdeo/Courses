# Lecture Notes: Persistent Storage in LangGraph Chatbot (SQLite Checkpointer)

---

## 1. Background: Evolution of the Chatbot

Over the past videos in the playlist, the chatbot was gradually improved:

1. **Basic chatbot**:

   * Built with **LangGraph workflow**.
   * Interaction through **VS Code terminal**.

2. **GUI Integration**:

   * Used **Streamlit** to create a front-end for the chatbot.

3. **Streaming Responses**:

   * Instead of showing the entire AI response at once, implemented **streaming token-by-token output** for better UX.

4. **Thread Feature**:

   * Allowed users to **resume past conversations** (thread-based memory).
   * Users could switch between different threads (e.g., “introductions” vs “pasta recipe”).

---

## 2. The Problem: Memory Loss After Refresh

* Current implementation uses **InMemorySaver** (RAM-based storage).
* Limitations:

  * Once the program is stopped or the page is refreshed, all conversations are **lost**.
  * No **persistent storage** available.

---

## 3. The Goal of This Lecture

* Replace **InMemorySaver** with a **database-backed checkpointer**.
* Use **SQLite** for persistence:

  * Messages stored in a database file (`chatbot.db`).
  * Chats will survive program restarts and page refreshes.
  * Users can resume conversations days later from the same point.

---

## 4. Types of Checkpointers in LangGraph

1. **InMemorySaver**

   * Stores in RAM.
   * Data lost after shutdown.

2. **SQLiteSaver**

   * Lightweight database.
   * Good for **prototyping**.

3. **PostgresSaver**

   * Production-grade database.
   * Scales better for real-world applications.

👉 For learning, we will use **SQLiteSaver**.

---

## 5. Backend Implementation

### Step 1: Install Required Library

`langgraph-checkpoint-sqlite` is not built-in, install it:

```bash
pip install langgraph-checkpoint-sqlite
```

---

### Step 2: Create a New Backend File

Create file: **`langgraph_database_backend.py`**

Copy the existing backend code into it, then modify.

---

### Step 3: Import SQLite Saver

Replace InMemorySaver with SQLiteSaver.

```python
import sqlite3
from langgraph.checkpoint.sqlite import SqliteSaver
```

---

### Step 4: Create SQLite Database Connection

```python
# Create or connect to chatbot.db in project folder
conn = sqlite3.connect("chatbot.db", check_same_thread=False)

# Pass the connection to SqliteSaver
checkpointer = SqliteSaver(conn)
```

#### Why `check_same_thread=False`?

* By default, SQLite enforces **single-thread usage**.
* Our chatbot uses multiple threads for conversations.
* Setting `check_same_thread=False` allows database access across threads.

---

### Step 5: Using the Checkpointer in LangGraph

Replace the previous `InMemorySaver` checkpointer with the new one:

```python
workflow = workflow.compile(checkpointer=checkpointer)
```

---

### Step 6: Testing the Database Storage

#### Example Test Code:

```python
from langgraph_database_backend import workflow

# Example: send a user message
config = {"configurable": {"thread_id": "thread1"}}
response = workflow.invoke(
    {"role": "user", "content": "Hi, my name is Nitish"},
    config=config
)

print(response)
```

* Running this creates `chatbot.db` in your project directory.
* Both **user** and **AI responses** get stored automatically.

---

### Step 7: Verifying Persistence

1. Run once:

   ```
   User: Hi, my name is Nitish
   AI: Hello Nitish, how can I assist you today?
   ```

   → Database file created: `chatbot.db`.

2. Run again with:

   ```python
   {"role": "user", "content": "What is my name?"}
   ```

   → AI replies: `"Your name is Nitish"`
   → Even though program restarted, the bot **remembered**.

---

### Step 8: Multiple Threads

```python
# Thread 1
config1 = {"configurable": {"thread_id": "thread1"}}

# Thread 2
config2 = {"configurable": {"thread_id": "thread2"}}
```

* Thread 1 conversation = about Nitish.
* Thread 2 conversation = about Rahul.
* Switching threads recalls correct context.

---

## 6. Exploring the Database

### Step 1: VS Code Extension

Install **SQLite Viewer** (by Florian Klämpfl).

* Open `chatbot.db`.
* See all stored messages, threads, and checkpoints.

### Step 2: Checkpoint Structure

* Each execution creates \~3 checkpoints:

  1. Start of workflow
  2. Chat node
  3. End node

* Multiple runs = multiple checkpoints.

* But threads remain **distinct** (`thread1`, `thread2`, etc.).

---

## 7. Retrieving Threads from Database

Problem: When the chatbot loads, the frontend currently **starts with empty threads**.
Now, since we have persistence, we should **load existing threads**.

---

### Step 1: Use Checkpointer `.list()` Function

```python
threads = set()
for checkpoint in checkpointer.list():
    thread_id = checkpoint.config.configurable.get("thread_id")
    threads.add(thread_id)

print(list(threads))
# Example Output: ['thread1', 'thread2']
```

---

### Step 2: Create a Function

```python
def retrieve_all_threads():
    threads = set()
    for checkpoint in checkpointer.list():
        thread_id = checkpoint.config.configurable.get("thread_id")
        threads.add(thread_id)
    return list(threads)
```

---

## 8. Frontend Implementation (Streamlit)

### Step 1: Create New File

**`streamlit_frontend_database.py`**

Copy previous frontend code (thread-based version).

---

### Step 2: Modify Session Initialization

Before:

```python
st.session_state["chat_threads"] = []
```

After:

```python
from langgraph_database_backend import chatbot, retrieve_all_threads

st.session_state["chat_threads"] = retrieve_all_threads()
```

---

### Step 3: Behavior

* When chatbot UI loads:

  * All past threads (`thread1`, `thread2`, etc.) appear.
  * Their **conversations** are also retrieved and displayed.
* New conversations automatically stored in `chatbot.db`.
* Restarting the app does not lose past chats.

---

## 9. Demonstration

* **Thread 1**: About Nitish → remembers his name.
* **Thread 2**: About Rahul → responds with Rahul.
* **Thread 3**: Recipe for Biryani → persists after restart.

✅ After closing and restarting the app:

* All threads still available.
* Conversations intact.
* True persistence achieved.

---

## 10. Key Takeaways

* **Problem solved**: Conversations no longer disappear after restart.
* **SQLite checkpointer** integrates LangGraph workflows with persistent storage.
* **Threads** remain independent.
* **Frontend** updated to fetch stored threads on load.
* Conversations can be **resumed anytime**, even days later.

---

## 11. Code Summary

### Backend Setup

```python
import sqlite3
from langgraph.checkpoint.sqlite import SqliteSaver

# Connect to SQLite DB
conn = sqlite3.connect("chatbot.db", check_same_thread=False)
checkpointer = SqliteSaver(conn)

# Workflow compilation
workflow = workflow.compile(checkpointer=checkpointer)

# Function to retrieve threads
def retrieve_all_threads():
    threads = set()
    for checkpoint in checkpointer.list():
        thread_id = checkpoint.config.configurable.get("thread_id")
        threads.add(thread_id)
    return list(threads)
```

---

### Frontend Setup (Streamlit)

```python
import streamlit as st
from langgraph_database_backend import chatbot, retrieve_all_threads

# Initialize session
if "chat_threads" not in st.session_state:
    st.session_state["chat_threads"] = retrieve_all_threads()

# UI to interact with chatbot
selected_thread = st.selectbox("Choose a thread:", st.session_state["chat_threads"])

user_input = st.text_input("Your message:")
if st.button("Send"):
    response = chatbot.invoke(
        {"role": "user", "content": user_input},
        config={"configurable": {"thread_id": selected_thread}}
    )
    st.write(response)
```

---

✅ At this point, your chatbot has:

* GUI
* Streaming responses
* Threading
* **Persistent storage with SQLite**
