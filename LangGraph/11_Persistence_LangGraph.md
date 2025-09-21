# Lecture Notes: Persistence in LangGraph

---

## 1. Introduction

* **Topic**: Persistence in LangGraph
* **Why important?**

  * It is a *foundational concept* in LangGraph.
  * Many advanced features (e.g., fault tolerance, resumable chats, human-in-the-loop, time travel) are built on persistence.

---

## 2. Recap: Core Principles of LangGraph

### (a) Graphs

* Any high-level goal can be broken into **tasks**.
* Tasks are represented as **nodes** in a graph.
* Edges define **execution order** between tasks.

Example:

* Task 1 → Task 2 → Task 3
* Nodes = tasks, Edges = order of execution.

---

### (b) State

* A **state** stores the important data needed during workflow execution.
* Represented as a **Python dictionary** (`dict`).
* Each node can:

  * **Read** values from state.
  * **Write/modify** values in state.

Example (Chatbot workflow):

```python
state = {
    "messages": []  # stores conversation history
}
```

* Every node can add or read from `messages`.

---

## 3. Problem Without Persistence

* Execution flow:

  1. Workflow starts with input.
  2. Nodes execute sequentially, updating state.
  3. Final state is produced.
* **Issue**:

  * When workflow finishes, the state (intermediate + final values) is erased from memory (RAM).
  * Once lost, it cannot be accessed in the future.
* This limits:

  * Resuming workflows.
  * Building chatbots with memory.
  * Fault tolerance.

---

## 4. What is Persistence?

**Definition**:

> *Persistence in LangGraph refers to the ability to save and restore the state of a workflow over time.*

### Key Features:

1. Saves **all state values**:

   * Not just the final values.
   * Also **intermediate values** at every step.
2. Allows:

   * Future access.
   * Workflow resumption from where it crashed.
   * Chat history storage.
   * Debugging via replay.

---

## 5. How Persistence Works

### (a) CheckPointers

* **CheckPointer**: The mechanism that implements persistence.
* Splits workflow execution into **checkpoints**.
* At each checkpoint:

  * Current state values are saved (to memory or DB).

### (b) SuperSteps

* A **superstep** = a set of parallel tasks executed together.
* Each superstep → becomes a checkpoint.
* Example graph:

  * Start → Node1 (Superstep 1)
  * Node1 → Node2, Node3, Node4 (parallel) (Superstep 2)
  * Node2/3/4 → End (Superstep 3)

---

### Example: Numbers Reducer with Checkpoints

```python
state = {"numbers": []}

# Superstep 1
state["numbers"].append(1)  # checkpoint 1 → [1]

# Superstep 2
state["numbers"].append(2)  # checkpoint 2 → [1,2]

# Superstep 3
state["numbers"].extend([3,4,5])  # checkpoint 3 → [1,2,3,4,5]

# Superstep 4 (end)
final = state["numbers"]  # checkpoint 4 → [1,2,3,4,5]
```

* Each intermediate state is saved.

---

### (c) Threads

* Each workflow execution = a **thread**.
* **Thread ID** uniquely identifies one run of a workflow.
* Purpose:

  * Multiple runs may exist in DB.
  * Thread ID helps retrieve *only the relevant state values*.

Example:

```python
# Thread 1 execution → DB stores under "thread_id=1"
# Thread 2 execution → DB stores under "thread_id=2"

workflow.invoke(state={"topic": "pizza"}, config={"thread_id": "1"})
workflow.invoke(state={"topic": "pasta"}, config={"thread_id": "2"})
```

---

## 6. Practical Example: Joke Generator Workflow

### Workflow:

1. Input topic (e.g., `"pizza"`).
2. Generate a **joke** about the topic.
3. Generate an **explanation** for the joke.

### Code Outline

```python
from langgraph.checkpoint.memory import InMemorySaver

# Create LLM (mock)
def llm(prompt):
    return f"Mocked response for: {prompt}"

# State
state = {
    "topic": "",
    "joke": "",
    "explanation": ""
}

# Nodes
def generate_joke(state):
    prompt = f"Generate a joke on {state['topic']}"
    state["joke"] = llm(prompt)
    return state

def generate_explanation(state):
    prompt = f"Explain the joke: {state['joke']}"
    state["explanation"] = llm(prompt)
    return state

# Graph + CheckPointer
checkpoint = InMemorySaver()
graph = Graph(checkpointer=checkpoint)

graph.add_node("generate_joke", generate_joke)
graph.add_node("generate_explanation", generate_explanation)

graph.add_edge("START", "generate_joke")
graph.add_edge("generate_joke", "generate_explanation")
graph.add_edge("generate_explanation", "END")

# Run
config = {"thread_id": "1"}
graph.invoke({"topic": "pizza"}, config=config)

# Retrieve state
print(graph.get_state(config))            # final state
print(graph.get_state_history(config))    # all checkpoints
```

---

## 7. Benefits of Persistence

1. **Short-Term Memory**

   * Chatbots can resume past conversations.
   * Example: ChatGPT showing old conversations.

2. **Fault Tolerance**

   * If workflow crashes mid-execution, it can **resume from last checkpoint**, not restart from beginning.

   Example:

   ```python
   # If crash at Node 2
   graph.invoke(None, config={"thread_id": "1"})  # resumes instead of restarting
   ```

3. **Human in the Loop (HITL)**

   * Workflows can pause, wait for human approval/input, then resume.
   * Example: Approve/reject a LinkedIn post before publishing.

4. **Time Travel**

   * Replay workflow execution from any past checkpoint.
   * Useful for debugging or experimentation.
   * You can also **update state** at a past checkpoint and replay from there.

---

## 8. Time Travel Example

### Replaying from a Checkpoint

```python
# Get checkpoint history
history = graph.get_state_history({"thread_id": "1"})

# Pick checkpoint ID
checkpoint_id = history[1].id

# Replay from there
graph.invoke(None, config={"thread_id": "1", "checkpoint_id": checkpoint_id})
```

### Updating State at a Checkpoint

```python
graph.update_state(
    {"topic": "samosa"}, 
    config={"thread_id": "1", "checkpoint_id": checkpoint_id}
)

# Now replay → generates joke/explanation about samosa
```

---

## 9. Summary

* **Persistence** = save & restore workflow state.
* Implemented using:

  * **CheckPointers** (save state at checkpoints).
  * **Threads** (distinguish between runs).
* **Key Benefits**:

  1. Short-term memory for chatbots.
  2. Fault tolerance (resume on crash).
  3. Human in the loop (pause → wait → resume).
  4. Time travel (replay/debug workflows).

**Final Thought**:
Persistence makes LangGraph workflows powerful, reliable, and production-ready.

---

✅ These notes cover **theory, implementation, examples, and code snippets**.
