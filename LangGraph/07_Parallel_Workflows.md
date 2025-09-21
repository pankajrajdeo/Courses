# Lecture Notes: Building Parallel Workflows in LangGraph (Agentic AI Series)

## 1. Recap of Previous Videos

* Playlist so far: 5 videos.
* Mostly covered **concepts** around:

  * Agentic AI.
  * LangGraph fundamentals.
* In the last video:

  * Started **practical implementation**.
  * Built **sequential (linear) workflows** using LangGraph.

👉 Today’s focus: **Parallel workflows** in LangGraph.

---

## 2. Agenda for This Lecture

1. Learn **parallel workflows** through **two examples**:

   * **Example 1**: A simple **non-LLM workflow** (Cricket stats).
   * **Example 2**: A more **realistic LLM-based workflow** (UPSC essay evaluation).
2. Key concepts:

   * Parallel execution in LangGraph.
   * **Partial state updates**.
   * **Structured output** (Pydantic schemas).
   * **Reducer functions** for merging parallel results.

---

## 3. Example 1: Cricket Stats Workflow (Non-LLM)

### 🎯 Goal

* Input: Batsman stats from one inning.

  * Runs scored.
  * Balls faced.
  * Number of fours.
  * Number of sixes.
* Output:

  1. **Strike Rate**
  2. **Boundary Percentage (Runs in boundaries %)**
  3. **Balls per Boundary**
  4. A summary combining all of the above.

### ⚙️ Observation

* These three calculations are **independent** of each other.
* Can be computed **in parallel**.
* Then merged into a summary node.

---

### 🔑 State Definition

```python
from typing import TypedDict

class BatsmanState(TypedDict):
    runs: int
    balls: int
    fours: int
    sixes: int
    strike_rate: float
    boundary_percent: float
    balls_per_boundary: float
    summary: str
```

---

### 🖇 Graph Structure

* **Start Node → Parallel Nodes → Summary Node → End Node**

Diagram:

```
Start
   ├──> Calculate Strike Rate
   ├──> Calculate Boundary %
   └──> Calculate Balls/Boundary
           ↓
        Summary
           ↓
          End
```

---

### 🧩 Functions for Each Node

#### 1. Strike Rate

```python
def calculate_strike_rate(state: BatsmanState):
    strike_rate = (state["runs"] / state["balls"]) * 100
    return {"strike_rate": strike_rate}
```

#### 2. Balls per Boundary

```python
def calculate_bpb(state: BatsmanState):
    bpb = state["balls"] / (state["fours"] + state["sixes"])
    return {"balls_per_boundary": bpb}
```

#### 3. Boundary Percentage

```python
def calculate_boundary_percent(state: BatsmanState):
    boundary_runs = (state["fours"] * 4) + (state["sixes"] * 6)
    boundary_percent = (boundary_runs / state["runs"]) * 100
    return {"boundary_percent": boundary_percent}
```

#### 4. Summary Node

```python
def summary(state: BatsmanState):
    text = f"""
    Strike Rate: {state['strike_rate']}
    Balls/Boundary: {state['balls_per_boundary']}
    Boundary%: {state['boundary_percent']}
    """
    return {"summary": text}
```

---

### 🕸 Defining the Graph

```python
from langgraph.graph import StateGraph, START, END

graph = StateGraph(BatsmanState)
graph.add_node("Calculate Strike Rate", calculate_strike_rate)
graph.add_node("Calculate BPB", calculate_bpb)
graph.add_node("Calculate Boundary%", calculate_boundary_percent)
graph.add_node("Summary", summary)

# Edges
graph.add_edge(START, "Calculate Strike Rate")
graph.add_edge(START, "Calculate BPB")
graph.add_edge(START, "Calculate Boundary%")

graph.add_edge("Calculate Strike Rate", "Summary")
graph.add_edge("Calculate BPB", "Summary")
graph.add_edge("Calculate Boundary%", "Summary")

graph.add_edge("Summary", END)

workflow = graph.compile()
```

---

### 🚨 Problem: InvalidUpdateError

* Error occurs because **all nodes return full state** → LangGraph sees conflicting updates to same attributes.
* **Solution**:
  Return **only partial updates** (dicts with changed keys).
  → Called **Partial State Updates**.

---

### ✅ Fixed Output

```python
initial_state = {"runs": 100, "balls": 50, "fours": 6, "sixes": 4}
result = workflow.invoke(initial_state)

print(result["summary"])
```

Output:

```
Strike Rate: 200.0
Balls/Boundary: 5.0
Boundary%: 48.0
```

👉 Lesson: Always prefer **partial state updates** → works for both sequential & parallel.

---

## 4. Example 2: UPSC Essay Evaluation Workflow (LLM-based)

### 🎯 Goal

* Input: Essay text.
* Parallel evaluation by **3 LLM nodes**:

  1. Clarity of Thought.
  2. Depth of Analysis.
  3. Language Quality.
* Each node returns:

  * **Feedback** (string).
  * **Score** (0–10 integer).
* Final node:

  * Summarizes feedbacks.
  * Computes average score.

---

### 🔑 State Definition

```python
from typing import TypedDict, List
from typing_extensions import Annotated
import operator

class UPSCState(TypedDict):
    essay: str
    language_feedback: str
    analysis_feedback: str
    clarity_feedback: str
    overall_feedback: str
    individual_scores: Annotated[List[int], operator.add]  # Reducer function
    average_score: float
```

👉 Why Reducer?

* Parallel nodes append scores → must **merge lists** instead of overwrite.

---

### 🛠 Structured Output with Pydantic

```python
from pydantic import BaseModel, Field

class EvaluationSchema(BaseModel):
    feedback: str = Field(description="Detailed feedback for the essay")
    score: int = Field(description="Score out of 10", ge=0, le=10)
```

* Use **structured model** (e.g., GPT-4o-mini) with schema enforcement.
* Ensures reliable JSON output → prevents `"SEVEN"` type errors.

---

### 🧩 Node Functions

#### 1. Evaluate Language

```python
def evaluate_language(state: UPSCState):
    prompt = f"""
    Evaluate the language quality of the following essay.
    Provide detailed feedback and assign a score out of 10.
    Essay: {state['essay']}
    """
    output = structured_model.invoke(prompt)  # returns EvaluationSchema
    return {
        "language_feedback": output.feedback,
        "individual_scores": [output.score]
    }
```

#### 2. Evaluate Depth of Analysis

```python
def evaluate_analysis(state: UPSCState):
    prompt = f"""
    Evaluate the depth of analysis of the following essay.
    Provide detailed feedback and assign a score out of 10.
    Essay: {state['essay']}
    """
    output = structured_model.invoke(prompt)
    return {
        "analysis_feedback": output.feedback,
        "individual_scores": [output.score]
    }
```

#### 3. Evaluate Clarity of Thought

```python
def evaluate_clarity(state: UPSCState):
    prompt = f"""
    Evaluate the clarity of thought in the following essay.
    Provide detailed feedback and assign a score out of 10.
    Essay: {state['essay']}
    """
    output = structured_model.invoke(prompt)
    return {
        "clarity_feedback": output.feedback,
        "individual_scores": [output.score]
    }
```

#### 4. Final Evaluation

```python
def final_evaluation(state: UPSCState):
    prompt = f"""
    Based on the following feedbacks, create a summarized feedback:
    Language: {state['language_feedback']}
    Analysis: {state['analysis_feedback']}
    Clarity: {state['clarity_feedback']}
    """
    final_fb = llm.invoke(prompt)["content"]

    avg_score = sum(state["individual_scores"]) / len(state["individual_scores"])

    return {
        "overall_feedback": final_fb,
        "average_score": avg_score
    }
```

---

### 🕸 Graph Definition

```python
graph = StateGraph(UPSCState)

graph.add_node("Evaluate Language", evaluate_language)
graph.add_node("Evaluate Analysis", evaluate_analysis)
graph.add_node("Evaluate Clarity", evaluate_clarity)
graph.add_node("Final Evaluation", final_evaluation)

# Parallel edges
graph.add_edge(START, "Evaluate Language")
graph.add_edge(START, "Evaluate Analysis")
graph.add_edge(START, "Evaluate Clarity")

# Merge into final evaluation
graph.add_edge("Evaluate Language", "Final Evaluation")
graph.add_edge("Evaluate Analysis", "Final Evaluation")
graph.add_edge("Evaluate Clarity", "Final Evaluation")

# End
graph.add_edge("Final Evaluation", END)

workflow = graph.compile()
```

---

### ✅ Execution

```python
essay_text = "India has many good engineers and smart students..."
state = {"essay": essay_text}
result = workflow.invoke(state)

print(result["overall_feedback"])
print("Average Score:", result["average_score"])
```

Sample Output:

```
Overall Feedback:
The essay lacks grammatical correctness, analysis is weak, and clarity is missing...
Average Score: 4.5
```

---

## 5. Key Learnings

1. **Parallel Workflows**: Independent tasks can run simultaneously.
2. **Partial State Updates**: Return only the updated fields → avoid conflicts.
3. **Structured Output**: Enforce schema with Pydantic + LLM → ensures reliable results.
4. **Reducer Functions**: Merge parallel results into single state attributes (e.g., list of scores).
5. **Scalability**: Combining LangGraph with LangChain enables building complex, real-world agentic AI workflows.

---

## 6. Next Steps

* Experiment by:

  * Adding new cricket metrics.
  * Evaluating essays with more criteria.
* Upcoming videos: more **complex & powerful workflows** combining multiple advanced concepts.
