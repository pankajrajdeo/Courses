# Lecture Notes: Building a Chatbot with LangGraph (Part 1)

---

## 1. Introduction

* We are continuing the **Agentic AI using LangGraph** playlist.
* Until now, we covered:

  1. **LangGraph fundamentals** (theoretical + basics of Agentic AI).
  2. **Workflows in LangGraph**:

     * Sequential workflow
     * Parallel workflow
     * Conditional workflow
     * Looping workflow

👉 With these basics, we are ready to **build real applications**.

---

## 2. What We’re Building

We will build a **feature-rich chatbot** step by step.
Today, we focus on **a simple chatbot that remembers conversation history**.

### Planned features (to be added in future videos):

1. Normal chatting with LLM.
2. RAG (Retrieve answers from documents).
3. Tool integration (let the bot take actions).
4. UI integration.
5. LangSmith integration (monitoring & tracing).
6. Advanced concepts:

   * Memory
   * Persistence & Checkpointers
   * Human-in-the-loop (HITL)
   * Retry logic & fault tolerance

---

## 3. Chatbot Design

* A chatbot is essentially a **workflow**.
* For the simplest version:

  * Workflow = **sequential graph**
  * Contains only **one node** (chat node).
  * Process:

    1. User sends message → enters the workflow.
    2. Chat node (LLM) processes the message.
    3. LLM generates reply → returned to user.
    4. Repeat while chatting.

---

## 4. State Management

In LangGraph, every workflow has a **state** that stores data during execution.
For a chatbot, the **state is the conversation history**.

* We’ll store conversation history as a **list of messages**.
* Types of messages:

  * `HumanMessage` → User input
  * `AIMessage` → LLM response
  * `SystemMessage` → Instructions to LLM
  * `ToolMessage` → Tool interactions

All of them inherit from `BaseMessage`.
So, we’ll store a **list of `BaseMessage`** in state.

---

### 4.1 Reducer Functions

* By default, when you add new data to state, **old values get replaced**.
* But we want to **append messages instead of replacing**.
* Solution: use a **reducer function**.

LangGraph provides:

```python
from langgraph.graph.message import add_messages
```

This appends new messages to the state efficiently.

---

## 5. Code: Defining State

```python
from typing import List
from typing_extensions import TypedDict
from langchain.schema import BaseMessage
from langgraph.graph.message import add_messages

class ChatState(TypedDict):
    messages: List[BaseMessage]
```

* State has only one key: `messages`.
* It will store a list of all conversation messages.

---

## 6. Code: Creating Graph

```python
from langgraph.graph import StateGraph
from langchain_openai import ChatOpenAI

# Define graph with state
graph = StateGraph(ChatState)
```

---

## 7. Chat Node

### Logic:

1. Take user’s query from state.
2. Send to LLM.
3. Get response.
4. Append response to state.

```python
llm = ChatOpenAI(model="gpt-3.5-turbo")  # default LLM

def chat_node(state: ChatState):
    # Get conversation so far
    messages = state["messages"]

    # Send conversation to LLM
    response = llm.invoke(messages)

    # Return new AI message appended to state
    return {"messages": [response]}
```

---

## 8. Adding Node & Edges

```python
# Add node
graph.add_node("chat_node", chat_node)

# Connect workflow
graph.add_edge("start", "chat_node")
graph.add_edge("chat_node", "end")

# Compile graph
chatbot = graph.compile()
```

👉 Workflow = `start → chat_node → end`

---

## 9. Running the Chatbot (Single Interaction)

```python
from langchain.schema import HumanMessage

# Initial state
initial_state = {
    "messages": [HumanMessage(content="What is the capital of India?")]
}

# Run chatbot
result = chatbot.invoke(initial_state)

# Check messages
for msg in result["messages"]:
    print(msg.type, ":", msg.content)
```

**Output:**

```
human : What is the capital of India?
ai    : The capital of India is New Delhi.
```

---

## 10. Making It Interactive (Loop)

To simulate a real chatbot, wrap in a **while loop**:

```python
while True:
    user_input = input("You: ")

    # Exit conditions
    if user_input.strip().lower() in ["exit", "quit", "bye"]:
        print("Chatbot: Goodbye!")
        break

    # Run chatbot
    state = {"messages": [HumanMessage(content=user_input)]}
    response = chatbot.invoke(state)

    # Extract latest AI message
    ai_message = response["messages"][-1]
    print("Chatbot:", ai_message.content)
```

---

## 11. Problem: Forgetfulness

* Issue: Chatbot **forgets past conversations**.
* Why?

  * Each loop iteration creates a **fresh state**.
  * Only the current message is passed to the LLM.
  * Previous messages are not remembered.

👉 Result = Chatbot cannot recall user’s name, past answers, or ongoing context.

---

## 12. Solution: Persistence

### 12.1 What is Persistence?

* By default, LangGraph **erases state when workflow ends**.
* Persistence allows saving state so it can be restored in future runs.
* Options:

  1. Save to **RAM** (temporary, lost on restart).
  2. Save to **database** (permanent, survives restarts).

---

### 12.2 Code: Adding Memory

```python
from langgraph.checkpoint.memory import MemorySaver

# Create checkpointer (RAM-based persistence)
checkpointer = MemorySaver()

# Compile graph with checkpointer
chatbot = graph.compile(checkpointer=checkpointer)
```

---

### 12.3 Using Threads

* Each conversation = **thread**.
* Thread ID = identifier for one user’s chat session.
* Allows multiple users (Nitesh, Rahul, Amit) to chat simultaneously.

```python
thread_id = "nitesh-session"

config = {
    "configurable": {"thread_id": thread_id}
}
```

---

### 12.4 Invoking with Persistence

```python
user_message = input("You: ")

response = chatbot.invoke(
    {"messages": [HumanMessage(content=user_message)]},
    config=config
)

ai_message = response["messages"][-1]
print("Chatbot:", ai_message.content)
```

---

## 13. Demo Example

```
You: Hi my name is Nitesh
Chatbot: Hello Nitesh! Nice to meet you.

You: What is my name?
Chatbot: Your name is Nitesh.

You: Add 10 to 100
Chatbot: The result is 110.

You: Multiply the result by 2
Chatbot: Sure, 110 * 2 = 220.
```

👉 Works because **entire conversation history is remembered** using persistence.

---

## 14. Limitations of RAM Memory

* If program restarts, all memory is erased.
* For production:

  * Use **database-backed persistence** (e.g., SQLite, Postgres).
  * Ensures chatbot remembers conversations even across multiple sessions.

---

## 15. Key Takeaways

1. **State** = conversation history (messages).
2. Use **`add_messages` reducer** to append instead of replace.
3. **Chatbot = workflow** with one node.
4. Without persistence, chatbot forgets past interactions.
5. **Persistence (with MemorySaver or DB)** solves this issue.
6. Threads allow multi-user conversations.

---

## 16. What’s Next?

* Next lecture: **Persistence in depth**

  * Checkpointers
  * Fault tolerance
  * Human-in-the-loop (HITL)
  * Retry logic
  * Database persistence

---

✅ At this point, you can build a **basic chatbot with memory** using LangGraph.
