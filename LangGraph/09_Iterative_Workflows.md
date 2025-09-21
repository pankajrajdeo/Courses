# 📘 Lecture Notes: Iterative (Looping) Workflows in LangGraph

## 1. Recap of Previous Lectures

Before diving into today’s concept, here’s what has been covered in the playlist so far:

1. **Sequential Workflows**

   * Tasks are executed **one after another** in a linear fashion.
   * Example: Task A → Task B → Task C.

2. **Parallel Workflows**

   * Multiple tasks run **simultaneously**.
   * Example: Task A and Task B run together, then results combine.

3. **Conditional Workflows**

   * Multiple possible tasks exist, but **only one task is chosen** based on a condition.
   * Example: If condition X → Task A else → Task B.

---

## 2. Today’s Topic: Iterative (Looping) Workflows

### 🔑 Definition

An **iterative (or looping) workflow** is a flow where tasks **repeat between nodes** until a desired result is achieved.

* Useful for **refinement, optimization, or continuous improvement**.
* Common in real-world scenarios where the **first attempt is rarely perfect**.

### Real-World Analogy

* **Use Case:** Automating social media posts for multiple platforms (e.g., Twitter/X, LinkedIn, Instagram).
* Challenge: The generated post may be **mediocre or repetitive**.
* Solution: Build a **looped workflow** that generates → evaluates → improves → re-evaluates posts until they meet quality standards.

---

## 3. Workflow Design: Generator–Evaluator–Optimizer

The iterative workflow involves **three core components**:

1. **Generator (LLM 1)**

   * Creates a post (tweet in this case) from a given topic.

2. **Evaluator (LLM 2)**

   * Evaluates the post against strict criteria.
   * Returns:

     * `Approved` → End workflow.
     * `Needs Improvement` → Passes feedback + post to optimizer.

3. **Optimizer (LLM 3)**

   * Uses evaluator’s feedback to improve the post.
   * Returns the new version to evaluator.

This creates a **loop**:
`Generate → Evaluate → (Approved ? End : Optimize → Evaluate again)`

---

## 4. LangGraph Implementation

### Step 1: Define State

Every LangGraph workflow needs a **state definition** to track data flow.

```python
from typing import TypedDict, Literal, Annotated
from operator import add

class TweetState(TypedDict):
    topic: str                      # User-provided topic
    tweet: str                      # Latest generated tweet
    evaluation: Literal["approved", "needs_improvement"]
    feedback: str                   # Evaluator’s feedback
    iteration: int                  # Current loop count
    max_iterations: int             # Safety limit
    tweet_history: Annotated[list[str], add]     # Stores all tweets
    feedback_history: Annotated[list[str], add]  # Stores feedback history
```

---

### Step 2: Build Graph & Nodes

We need **three nodes** (`generate`, `evaluate`, `optimize`).

```python
from langgraph.graph import StateGraph

graph = StateGraph(TweetState)

# Add nodes
graph.add_node("generate", generate_tweet)
graph.add_node("evaluate", evaluate_tweet)
graph.add_node("optimize", optimize_tweet)
```

---

### Step 3: Define Functions

#### (a) Generator Function

Creates an initial funny/original tweet.

```python
def generate_tweet(state: TweetState) -> dict:
    messages = [
        {"role": "system", "content": "You are a funny and clever Twitter influencer."},
        {"role": "user", "content": f"Write a short, original, hilarious tweet about {state['topic']}.\n"
                                    "Rules: ≤280 characters, avoid Q&A format, use observational humor, irony, sarcasm, cultural references."}
    ]
    response = generator_llm.invoke(messages)
    return {
        "tweet": response.content,
        "tweet_history": [response.content]
    }
```

---

#### (b) Evaluator Function

Strictly evaluates tweet quality.

```python
from pydantic import BaseModel
from typing import Literal

class TweetEvaluation(BaseModel):
    evaluation: Literal["approved", "needs_improvement"]
    feedback: str

def evaluate_tweet(state: TweetState) -> dict:
    messages = [
        {"role": "system", "content": "You are a ruthless Twitter critic. Evaluate tweets on humor, originality, virality, and format."},
        {"role": "user", "content": f"Evaluate this tweet: {state['tweet']}\n"
                                    "Reject if: Q&A style, >280 chars, or cliché jokes.\n"
                                    "Return structured output: {evaluation, feedback}"}
    ]

    structured_eval = evaluator_llm.with_structured_output(TweetEvaluation)
    result = structured_eval.invoke(messages)

    return {
        "evaluation": result.evaluation,
        "feedback": result.feedback,
        "feedback_history": [result.feedback]
    }
```

---

#### (c) Optimizer Function

Improves tweet based on evaluator’s feedback.

```python
def optimize_tweet(state: TweetState) -> dict:
    messages = [
        {"role": "system", "content": "You improve tweets for humor & virality based on given feedback."},
        {"role": "user", "content": f"Topic: {state['topic']}\n"
                                    f"Original Tweet: {state['tweet']}\n"
                                    f"Feedback: {state['feedback']}\n"
                                    "Rewrite into a short, viral-worthy tweet under 280 characters."}
    ]

    response = optimizer_llm.invoke(messages)
    return {
        "tweet": response.content,
        "tweet_history": [response.content],
        "iteration": state["iteration"] + 1
    }
```

---

### Step 4: Control Flow (Edges & Loop)

Add conditional routing for looping.

```python
# Edges
graph.add_edge("START", "generate")
graph.add_edge("generate", "evaluate")

# Conditional routing
def route_evaluation(state: TweetState) -> str:
    if state["evaluation"] == "approved" or state["iteration"] >= state["max_iterations"]:
        return "approved"
    return "needs_improvement"

graph.add_conditional_edges(
    "evaluate",
    route_evaluation,
    {
        "approved": "END",
        "needs_improvement": "optimize"
    }
)

# Loop edge
graph.add_edge("optimize", "evaluate")
```

---

### Step 5: Running the Workflow

```python
workflow = graph.compile()

initial_state = {
    "topic": "Indian Railways",
    "iteration": 1,
    "max_iterations": 5
}

result = workflow.invoke(initial_state)

print("Final Tweet:", result["tweet"])
print("All Iterations:", result["tweet_history"])
print("Feedback History:", result["feedback_history"])
```

---

## 5. Key Learnings

* **Iterative workflows** let you refine outputs through looping until they meet quality.
* Three components: **Generation → Evaluation → Optimization**.
* Always define a **max iteration** to prevent infinite loops.
* Use **history tracking** (tweets + feedback) for transparency.
* Evaluator quality directly affects the **improvement cycle**.

---

## 6. Extensions for Future Work

* Add **Human-in-the-Loop (HITL)** approval before publishing.
* Connect APIs to **post directly** on Twitter/X.
* Experiment with different **LLMs per role** (some are better at writing, others at evaluation).
* Enhance evaluation with **numerical scoring** instead of binary feedback.

---

✅ With this, you should have a **complete understanding** of iterative workflows in LangGraph, both conceptually and practically.
