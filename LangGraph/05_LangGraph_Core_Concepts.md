# Lecture Notes: Core Concepts of LangGraph

---

## 1. Quick Recap: What is LangGraph?

* **LangGraph** is an **orchestration framework** for building **intelligent, stateful, multi-step LLM workflows**.
* It represents workflows as **graphs**:

  * **Nodes** = individual tasks (e.g., calling an LLM, tool, or decision logic).
  * **Edges** = connections that define execution flow (which task follows which).
* Key Features:

  * Sequential, parallel, branching, and looping flows.
  * Memory integration (records interactions).
  * Resumability (restart from failure point).
  * Ideal for **agentic, production-grade AI applications**.

🔑 **Analogy**: Think of LangGraph as a **flowchart executor for LLM workflows**.

---

## 2. Core Concept: LLM Workflows

### What is a Workflow?

* A **workflow** = sequence of tasks executed in order to achieve a goal.
* Example: **Automated hiring workflow**:

  1. Create Job Description.
  2. Post JD.
  3. Shortlist candidates.
  4. Conduct interviews.
  5. Onboard selected candidate.

### What is an LLM Workflow?

* A workflow where **some/many tasks rely on LLMs**.
* Example: Automated hiring:

  * JD writing → LLM required.
  * Candidate shortlisting → LLM may help.
  * Interview Q\&A generation → LLM helps.

🔑 **Definition**: *LLM workflows are step-by-step processes to build complex LLM-powered applications.*

---

## 3. Common LLM Workflows (Patterns)

### 3.1 Prompt Chaining

* **Multiple sequential calls** to an LLM, breaking down a complex task.
* Example: Topic → Outline → Full Report.

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

# Step 1: Generate outline
outline = llm.invoke("Create an outline for a report on Climate Change.")

# Step 2: Generate full report
report = llm.invoke(f"Write a detailed report based on this outline: {outline}")
```

* Allows adding **checks** (e.g., word limit, structure validation).

---

### 3.2 Routing

* LLM decides **where to send input next**.
* Example: **Customer Support Bot**

  * Refund query → Refund LLM.
  * Technical issue → Tech LLM.
  * Sales inquiry → Sales LLM.

```python
def router(query: str):
    if "refund" in query.lower():
        return refund_llm
    elif "error" in query.lower():
        return tech_llm
    else:
        return sales_llm
```

---

### 3.3 Parallelization

* Break task into **independent subtasks**, run **simultaneously**, then aggregate results.
* Example: **Content Moderation**

  * Check community guidelines.
  * Detect misinformation.
  * Detect sexual content.

```python
from concurrent.futures import ThreadPoolExecutor

tasks = [check_guidelines, check_misinformation, check_sexual_content]

with ThreadPoolExecutor() as executor:
    results = list(executor.map(lambda f: f(video_text), tasks))

decision = all(results)  # publish if all pass
```

---

### 3.4 Orchestrator–Worker

* Similar to parallelization, but **tasks are not predefined**.
* **Orchestrator** LLM assigns tasks dynamically to **workers**.
* Example: Research Assistant

  * Orchestrator decides:

    * Scientific query → Google Scholar worker.
    * Political query → Google News worker.

```python
def orchestrator(query: str):
    if is_scientific(query):
        return scholar_worker(query)
    elif is_political(query):
        return news_worker(query)
```

---

### 3.5 Evaluator–Optimizer

* Used when tasks need **iterations & refinement**.
* Two LLMs:

  * **Generator**: Creates draft (blog, email, story).
  * **Evaluator**: Checks against criteria, provides feedback.
* Loop continues until evaluator accepts.

```python
while True:
    draft = generator.invoke("Write blog on AI in Education")
    feedback = evaluator.invoke(f"Evaluate this blog:\n{draft}")
    if "Approved" in feedback:
        break
```

---

## 4. Core Concept: Graphs, Nodes, and Edges

* Workflows are represented as **graphs**:

  * **Nodes** = Python functions (tasks).
  * **Edges** = execution order.

🔑 **Nodes = What to do**, **Edges = When to do it**.

### Edge Types:

* Sequential: One after another.
* Parallel: Multiple at once.
* Conditional: Branch based on condition.
* Loop: Return to a previous node.

---

## 5. Core Concept: State

* **State = shared memory** across workflow.
* Stores evolving data during execution.
* Example (Essay Evaluation Workflow):

  * Fields in state:

    * `essay_text`
    * `clarity_score`
    * `depth_score`
    * `language_score`
    * `overall_score`

```python
from typing import TypedDict

class EssayState(TypedDict):
    essay_text: str
    clarity_score: int
    depth_score: int
    language_score: int
    overall_score: int
```

* **State Characteristics**:

  * Accessible to all nodes.
  * Mutable → changes over time.
  * Passed automatically between nodes.

---

## 6. Core Concept: Reducers

* Define **how state updates happen**.
* Options:

  * Replace (default).
  * Append (for conversations/history).
  * Merge (combine structures).

### Example Problem

* Chatbot with state:

  * If message replaces previous → old context lost.
  * Solution: **Append messages** instead.

```python
def message_reducer(old, new):
    return old + [new]  # append instead of replace
```

---

## 7. Core Concept: Execution Model

* Inspired by **Google Pregel** (graph processing).

* Execution Phases:

  1. **Graph Definition** → Define nodes, edges, state.
  2. **Compilation** → Validate structure (no orphan nodes).
  3. **Invocation** → Trigger first node with initial state.

* **Message Passing**:

  * State updates flow through edges to next node.

* **Supersteps**:

  * Each "round" of execution = **superstep**.
  * A superstep can include multiple parallel executions.

🔑 Execution ends when:

* No active nodes remain.
* No messages in transit.

---

# ✅ Key Takeaways

1. **LangGraph = orchestration framework** for LLM workflows.
2. Workflows = **series of tasks** (LLM/tool/logic).
3. **Common patterns**:

   * Prompt Chaining
   * Routing
   * Parallelization
   * Orchestrator–Worker
   * Evaluator–Optimizer
4. **Graphs** represent workflows:

   * Nodes = tasks, Edges = flow.
5. **State** = evolving shared memory (TypedDict).
6. **Reducers** define update policy (replace/append/merge).
7. **Execution Model**:

   * Graph → Compile → Invoke → Message Passing → Supersteps.

---

📌 Next Steps for Practice:

* Try coding a **simple prompt chaining workflow**.
* Experiment with **state updates & reducers**.
* Implement a **parallel content moderation example**.
