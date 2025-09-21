# Lecture Notes: Sequential Workflows in LangGraph

## 1. Introduction

* This lecture is part of a playlist on **Agentic AI using LangGraph**.
* Previous videos covered:

  * Differences between **Generative AI** and **Agentic AI**.
  * Detailed discussion of **Agentic AI with a use case**.
  * Why **LangGraph** was created when LangChain already exists.
  * Core **LangGraph concepts** necessary before coding.

👉 Today’s goal: Start **practical coding** with LangGraph, focusing on **Sequential Workflows**.

---

## 2. What is a Sequential Workflow?

* A **sequential workflow** connects tasks in a straight line.
* Flow: Task 1 → Task 2 → Task 3 → … → End.
* **No branching or parallel paths.**
* Example: Input → Process → Output.

---

## 3. Setup & Installation

### Steps:

1. Create a project folder: `langraph_tutorials` (opened in VS Code).

2. Create a **virtual environment**:

   ```bash
   python -m venv myenv
   myenv\Scripts\activate   # Windows
   source myenv/bin/activate  # Linux/Mac
   ```

3. Install dependencies:

   ```bash
   pip install langgraph
   pip install langchain
   pip install langchain-openai
   pip install python-dotenv
   ```

   * **Why LangChain?** LangGraph is for workflows; LangChain provides LLM components (models, prompt templates, loaders, etc.).
   * **Why dotenv?** For securely reading environment variables (like API keys).

4. Test imports in Jupyter Notebook (`0_test_installation.ipynb`):

   ```python
   from langgraph.graph import StateGraph
   from langchain_openai import ChatOpenAI
   ```

   ✅ If no errors, setup is correct.

---

## 4. First Workflow: **BMI Calculator** (Non-LLM Workflow)

### 4.1 Define State

The state is a **typed dictionary** holding data through the workflow.

```python
from typing import TypedDict

class BMIState(TypedDict):
    weight: float   # in kg
    height: float   # in meters
    bmi: float
```

### 4.2 Define the Graph

```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(BMIState)
```

### 4.3 Define Node Function

Each node is backed by a Python function.

```python
def calculate_bmi(state: BMIState) -> BMIState:
    weight = state["weight"]
    height = state["height"]
    bmi = weight / (height ** 2)
    state["bmi"] = round(bmi, 2)
    return state
```

### 4.4 Add Node and Edges

```python
graph.add_node("calculate_bmi", calculate_bmi)
graph.add_edge(START, "calculate_bmi")
graph.add_edge("calculate_bmi", END)
```

### 4.5 Compile and Run

```python
workflow = graph.compile()

initial_state = {"weight": 80, "height": 1.73, "bmi": 0}
final_state = workflow.invoke(initial_state)

print(final_state)
# Output: {'weight': 80, 'height': 1.73, 'bmi': 26.73}
```

---

## 5. Enhancing Workflow: **BMI Category**

### Update State

```python
class BMIState(TypedDict):
    weight: float
    height: float
    bmi: float
    category: str
```

### Add New Node

```python
def label_bmi(state: BMIState) -> BMIState:
    bmi = state["bmi"]
    if bmi < 18.5:
        state["category"] = "Underweight"
    elif bmi < 25:
        state["category"] = "Normal"
    elif bmi < 30:
        state["category"] = "Overweight"
    else:
        state["category"] = "Obese"
    return state
```

### Add to Graph

```python
graph.add_node("label_bmi", label_bmi)
graph.add_edge("calculate_bmi", "label_bmi")
graph.add_edge("label_bmi", END)
```

✅ Now workflow computes BMI **and classifies it**.

---

## 6. First **LLM-Based Workflow** (Simple Q\&A)

### Define State

```python
class LLMState(TypedDict):
    question: str
    answer: str
```

### Setup Model

```python
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()
model = ChatOpenAI(model="gpt-3.5-turbo")
```

### Define Node

```python
def llm_qa(state: LLMState) -> LLMState:
    question = state["question"]
    prompt = f"Answer the following question: {question}"
    response = model.invoke(prompt)
    state["answer"] = response.content
    return state
```

### Build Graph

```python
graph = StateGraph(LLMState)
graph.add_node("llm_qa", llm_qa)
graph.add_edge(START, "llm_qa")
graph.add_edge("llm_qa", END)

workflow = graph.compile()
```

### Run

```python
initial_state = {"question": "How far is the moon from Earth?", "answer": ""}
final_state = workflow.invoke(initial_state)

print(final_state["answer"])
```

---

## 7. Prompt Chaining Workflow (Multi-LLM Calls)

### State

```python
class BlogState(TypedDict):
    title: str
    outline: str
    content: str
```

### Node 1: Generate Outline

```python
def create_outline(state: BlogState) -> BlogState:
    title = state["title"]
    prompt = f"Generate a detailed outline for a blog on the topic: {title}"
    response = model.invoke(prompt)
    state["outline"] = response.content
    return state
```

### Node 2: Generate Blog

```python
def create_blog(state: BlogState) -> BlogState:
    title = state["title"]
    outline = state["outline"]
    prompt = f"Write a detailed blog on '{title}' using this outline:\n{outline}"
    response = model.invoke(prompt)
    state["content"] = response.content
    return state
```

### Graph

```python
graph = StateGraph(BlogState)
graph.add_node("create_outline", create_outline)
graph.add_node("create_blog", create_blog)

graph.add_edge(START, "create_outline")
graph.add_edge("create_outline", "create_blog")
graph.add_edge("create_blog", END)

workflow = graph.compile()
```

### Run

```python
initial_state = {"title": "Rise of AI in India", "outline": "", "content": ""}
final_state = workflow.invoke(initial_state)

print("Outline:\n", final_state["outline"])
print("\nBlog Content:\n", final_state["content"])
```

---

## 8. Homework (Extension)

* Add an **Evaluate Node** to the blog workflow.
* The node should:

  * Take blog content + outline.
  * Prompt LLM: *“Based on this outline, rate my blog out of 10.”*
  * Save score into state.

---

# Key Takeaways

1. **LangGraph workflows** are built on four steps:

   * Define State → Add Nodes → Add Edges → Compile & Run.
2. Each **Node = Python function**.
3. **Sequential workflows** are easy but demonstrate syntax.
4. **LangGraph + LangChain** = workflows + LLM components.
5. **Prompt chaining** lets you break complex tasks into multiple LLM calls while preserving state.
