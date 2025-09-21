# Lecture Notes: Implementing Streaming in an LLM Chatbot (LangGraph + Streamlit)

---

## 1. Introduction

* In the last few videos of the playlist, we’ve been **building a chatbot step by step**:

  1. **Basic chatbot** → connected to an LLM for simple Q\&A.
  2. **Short-term memory** → chatbot remembers context of conversation.
  3. **UI integration** → used Streamlit to make it interactive.

* **Today’s Problem:**
  When chatbot generates a **large output** (e.g., 500-word blog), the **UI stays blank** until the *entire response* is ready. Then suddenly, the full text appears.

* **Problems with this behavior:**

  1. **User waiting time** → feels like the app is frozen.
  2. **Poor readability** → entire long text dumps at once.

* **Better UX (like ChatGPT):**

  * Text appears **token by token**, giving a “typewriter effect.”
  * This feature is called **Streaming**.

---

## 2. What is Streaming?

**Definition:**
In LLMs, *streaming* means the model starts sending tokens **as soon as they are generated**, instead of waiting for the entire response.

### Example

* **Non-streaming (traditional):**

  1. LLM generates full answer internally.
  2. After finishing, sends the whole text back at once.

* **Streaming:**

  * Tokens are sent to the user interface **as they are generated**, in real time.

---

## 3. Why Streaming is Important?

### Benefits of Streaming

1. **Faster perceived response time**

   * User sees text immediately (no long waiting).
   * Prevents users from thinking app is frozen.

2. **Human-like conversation**

   * Mimics natural dialogue.
   * Feels alive and engaging.

3. **Multi-modal UI support**

   * Devices like Alexa need streaming for natural flow.
   * Without streaming, long pauses break conversational experience.

4. **Better readability for long outputs (e.g., code)**

   * Code appears step by step, easier to follow.
   * Prevents overwhelming users with large blocks of text.

5. **Ability to interrupt responses**

   * Stop mid-response if irrelevant.
   * Saves **tokens → cost savings** (since providers charge per token).

6. **Progress updates in agents**

   * E.g., booking a movie ticket:

     * “Opened BookMyShow…”
     * “Selected movie…”
     * “Selected seats…”
     * “Payment processing…”
   * Keeps user informed of intermediate steps.

👉 **Takeaway:** Streaming is a **small change technically**, but provides **10x better user experience**.

---

## 4. Technical Foundation: Python Generators

* **Key concept:** LangGraph returns a **generator** when using `.stream()`.

* **Generator definition:**

  * A special type of iterator in Python.
  * Yields values one at a time using `yield` instead of returning everything at once.
  * Example:

```python
def my_generator():
    yield "Hello"
    yield "World"

for word in my_generator():
    print(word)
```

**Output:**

```
Hello
World
```

---

## 5. Implementing Streaming in LangGraph

### Old way (non-streaming):

```python
response = chatbot.invoke(initial_state)
print(response)
```

### New way (streaming):

```python
stream = chatbot.stream(
    {"messages": [HumanMessage(content="What is the recipe to make pasta?")]},
    config={"configurable": {"thread_id": "1"}},
    stream_mode="messages"
)

for message_chunk, metadata in stream:
    if message_chunk.content:
        print(message_chunk.content, end=" ")
```

### Key Changes:

* **`.invoke()` → `.stream()`**

* **Arguments needed:**

  1. **Initial state** (user query).
  2. **Config** (like thread\_id).
  3. **Stream mode** (`messages` for chatbot).

* **Stream object** returns a generator → loop through chunks → print tokens.

---

## 6. Backend Implementation

### Step 1: Replace `.invoke()` with `.stream()`

```python
from langchain.schema import HumanMessage

stream = chatbot.stream(
    {"messages": [HumanMessage(content="What is the recipe to make pasta?")]},
    config={"configurable": {"thread_id": "1"}},
    stream_mode="messages"
)

for message_chunk, metadata in stream:
    if message_chunk.content:
        print(message_chunk.content, end=" ")
```

* Confirm that `stream` is a generator:

```python
print(type(stream))  # <class 'generator'>
```

---

## 7. Frontend Implementation (Streamlit)

### Old code (non-streaming):

* Assistant response was collected fully and then displayed in one go.

### New code (streaming):

* Use Streamlit’s `st.write_stream()` function.
* Automatically creates a **typewriter effect** in UI.

#### Code:

```python
import streamlit as st

with st.chat_message("assistant"):
    ai_message = st.write_stream(
        message_chunk.content
        for message_chunk, metadata in chatbot.stream(
            {"messages": [HumanMessage(content=user_input)]},
            config={"configurable": {"thread_id": "1"}},
            stream_mode="messages"
        )
    )

# Save assistant’s message in session state
st.session_state.messages.append({"role": "assistant", "content": ai_message})
```

### Notes:

* `st.write_stream()` handles the display of streaming content.
* Generator yields chunks → displayed immediately.
* At the end, full response is stored in `ai_message` and saved in session.

---

## 8. Debugging Example

* Initial bug: hardcoded `"What is the recipe to make pasta?"` in backend.
* Fix: replace with `user_input`.

Corrected:

```python
{"messages": [HumanMessage(content=user_input)]}
```

---

## 9. Final Flow

1. Backend (`.stream()`): streams tokens from LLM.
2. Frontend (`st.write_stream()`): displays them token by token.
3. Session state updated with final response.

---

## 10. Key Takeaways

* **Streaming = Better UX** for chatbots and AI agents.
* Only **one major code change**: use `.stream()` instead of `.invoke()`.
* In **Streamlit**, `st.write_stream()` makes it very easy to integrate streaming.
* Benefits: faster responses, human-like interaction, improved readability, ability to interrupt, cost saving, real-time progress updates.

---

✅ **You should now be able to:**

* Explain what streaming is and why it matters.
* Modify an LLM-based chatbot to stream responses.
* Use Python generators in this context.
* Implement streaming in both backend and frontend using LangGraph + Streamlit.
