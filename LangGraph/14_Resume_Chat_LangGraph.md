# Lecture Notes – Building a “Resume Chat” Feature in a Chatbot (LangGraph + Streamlit)

---

## 1. Background & Progress So Far

* We have been building a chatbot step by step:

  1. **Basic console chatbot** (no UI).
  2. **UI added** using Streamlit.
  3. **Streaming responses** added → improves UX because users see partial responses instantly.

* Current status before this lecture:

  * Chatbot works with streaming.
  * But lacks "resume chat" → i.e., the ability to revisit old conversations.

---

## 2. New Feature: Resume Chat

* Goal: Add a **resume chat functionality** similar to ChatGPT.
* Features to implement:

  1. **New Chat** → Start a fresh conversation with its own thread ID.
  2. **Resume Previous Chat** → Access older threads and continue where left off.
  3. Sidebar UI with:

     * "New Chat" button.
     * "My Conversations" list (all previous threads, clickable).

---

## 3. Architecture

* **Backend (LangGraph):**

  * Already handles threads & messages.
  * No changes required.
* **Frontend (Streamlit):**

  * All changes will be made here.
  * Will create a new file (e.g., `streamlit_frontend_resume.py`).

---

## 4. Development Strategy

* **Break the big feature into smaller tasks**:

  1. Add Sidebar UI.
  2. Generate dynamic thread IDs and store in session.
  3. Show current thread ID in sidebar.
  4. Implement “New Chat” functionality (reset messages & new thread).
  5. Maintain all thread IDs (list).
  6. Display all threads as clickable buttons.
  7. On click → load conversation messages from backend into UI.

---

## 5. Task 1: Sidebar UI

* Add sidebar with:

  * Title
  * New Chat button
  * Header “My Conversations”

### Code:

```python
import streamlit as st

# Sidebar UI
st.sidebar.title("LangGraph Chatbot")
new_chat_button = st.sidebar.button("New Chat")
st.sidebar.header("My Conversations")
```

✅ Sidebar UI now displays, though button has no functionality yet.

---

## 6. Task 2: Dynamic Thread IDs

* Problem: Currently hardcoded (`thread_1`).
* Solution: Use Python’s `uuid` to auto-generate.

### Code – Utility function:

```python
import uuid

def generate_thread_id():
    return str(uuid.uuid4())
```

* Save thread ID in session:

```python
if "thread_id" not in st.session_state:
    st.session_state.thread_id = generate_thread_id()
```

* Use it instead of manual ID:

```python
thread_id = st.session_state.thread_id
```

✅ Now every session has a unique thread ID.

---

## 7. Task 3: Display Current Thread ID

* Show current ID under sidebar.

### Code:

```python
st.sidebar.text(f"Current Thread: {st.session_state.thread_id}")
```

✅ Displays current active conversation’s ID.

---

## 8. Task 4: New Chat Functionality

* Requirements:

  1. Generate a new thread ID.
  2. Replace old ID in session.
  3. Reset `message_history` (empty list).

### Code:

```python
def reset_chat():
    new_thread_id = generate_thread_id()
    st.session_state.thread_id = new_thread_id
    st.session_state.message_history = []

# Trigger on button click
if new_chat_button:
    reset_chat()
```

✅ Clicking "New Chat" clears old messages and assigns a new thread ID.

⚠️ Problem: Old thread IDs disappear (lost).
➡️ Solution in next step.

---

## 9. Task 5: Keep All Thread IDs

* Maintain list of all thread IDs in session (`chat_threads`).

### Code:

```python
# Initialize list
if "chat_threads" not in st.session_state:
    st.session_state.chat_threads = []

# Utility: Add thread to list
def add_thread(thread_id):
    if thread_id not in st.session_state.chat_threads:
        st.session_state.chat_threads.append(thread_id)

# On load
add_thread(st.session_state.thread_id)

# On new chat
def reset_chat():
    new_thread_id = generate_thread_id()
    st.session_state.thread_id = new_thread_id
    st.session_state.message_history = []
    add_thread(new_thread_id)
```

✅ All threads are now preserved in session.

---

## 10. Task 6: Display All Threads in Sidebar

* Instead of just one, show **all stored thread IDs**.

### Code:

```python
st.sidebar.header("My Conversations")
for tid in reversed(st.session_state.chat_threads):
    if st.sidebar.button(str(tid)):
        # Action will be added in next step
        pass
```

✅ Sidebar now lists all thread IDs as buttons (newest on top).

---

## 11. Task 7: Resume Old Conversations

* Requirement: On button click, load conversation associated with thread.

### Backend Function:

```python
def load_conversation(thread_id):
    config = {"configurable": {"thread_id": thread_id}}
    state = chatbot.get_state(config)
    return state.values['messages']  # Extract messages
```

* Convert messages into required frontend format:

```python
def format_messages(messages):
    formatted = []
    for msg in messages:
        role = "user" if msg.__class__.__name__ == "HumanMessage" else "assistant"
        formatted.append({"role": role, "content": msg.content})
    return formatted
```

* Update session when thread is clicked:

```python
for tid in reversed(st.session_state.chat_threads):
    if st.sidebar.button(str(tid)):
        st.session_state.thread_id = tid
        msgs = load_conversation(tid)
        st.session_state.message_history = format_messages(msgs)
```

✅ Old conversation fully loads into UI.

---

## 12. Current Limitation

* **Persistence issue**:

  * Currently uses **in-memory storage**.
  * When you refresh the app, all past conversations are lost.
* Fix in next lecture:

  * Connect LangGraph backend to a **database**.
  * Persistent conversations even after refresh.

---

## 13. Homework

* Replace thread IDs in sidebar with **meaningful conversation names** (like ChatGPT).
* Hint: Could use **first user message** as the conversation title.

---

## 14. Summary of Features Achieved

* Sidebar with "New Chat" + list of conversations.
* Dynamic thread IDs using `uuid`.
* Session-based storage for:

  * Current thread
  * All threads
  * Message history
* Ability to:

  * Start new conversations
  * Resume old conversations
* Messages load in correct format (user vs assistant).

---

## 15. Example Workflow in Action

1. User asks: *“Write factorial code in Python”*.
   → New thread created (e.g., `thread_abc`).

2. User clicks **New Chat**.
   → New thread created (`thread_xyz`). Previous messages cleared.

3. Sidebar shows:

   * `thread_xyz`
   * `thread_abc`

4. User clicks `thread_abc`.
   → Factorial conversation restored in UI.
