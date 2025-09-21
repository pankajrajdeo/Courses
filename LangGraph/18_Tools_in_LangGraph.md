# Lecture Notes: Adding Tools to a Chatbot with LangGraph

## 1. Context & Motivation

* We have already built a chatbot with:

  * GUI (Streamlit front-end)
  * Short-term memory
  * Database persistence
  * Streaming responses

* **Current limitation**:
  The chatbot only talks using an LLM backend (OpenAI).
  It cannot *perform actions* (like calculations, searches, or fetching real-time data).

* **Goal of this lecture**:
  Add **tools** so that the chatbot can:

  1. Perform **numerical calculations** (calculator tool)
  2. Perform **internet searches** (DuckDuckGo search tool)
  3. Fetch **real-time stock prices** (Stock API tool)

---

## 2. Key New Concepts

### 2.1 Tool Node

* A **prebuilt node** in LangGraph.
* Acts as a **bridge** between your workflow graph and external tools.
* Responsibilities:

  * Receives tool calls from the LLM
  * Routes them to the correct tool (calculator, search, stock API, etc.)
  * Returns results

👉 Think of it as a **Tool Executor**.

---

### 2.2 Tools Condition

* A **conditional function** in LangGraph.
* Helps the graph decide:

  * Should execution go back to the LLM (normal chat)?
  * Or should it go to the Tool Node (perform an action)?

---

### 2.3 Types of Tools

1. **Prebuilt tools** (provided by LangChain)

   * e.g., DuckDuckGo search tool
2. **Custom tools** (created with `@tool` decorator)

   * e.g., Calculator tool
   * e.g., Stock price fetcher using AlphaVantage API

---

## 3. Workflow Evolution

### 3.1 Simple Workflow (before tools)

```
User → Chat Node (LLM) → End
```

### 3.2 With Tools (initial attempt)

```
User → Chat Node → [Decision]
        ↳ Normal chat → End
        ↳ Action needed → Tool Node → End
```

**Problems:**

1. Tool outputs are raw/technical (e.g., JSON from stock API).
2. Cannot perform multi-step reasoning (e.g., stock price + calculate total cost).

---

### 3.3 Improved Workflow (with loop-back)

```
User → Chat Node → [Decision]
        ↳ Normal chat → End
        ↳ Action needed → Tool Node → back to Chat Node → (loop until done) → End
```

✅ Advantages:

* LLM refines tool outputs into user-friendly responses
* Supports **multi-step reasoning** (e.g., search → stock price → calculator)

---

## 4. Coding in LangGraph

### 4.1 Imports

```python
from langgraph.prebuilt import ToolNode, tools_condition
from langchain_core.tools import tool
from langchain_community.tools import DuckDuckGoSearchRun
from langchain_openai import ChatOpenAI
import os
import requests
```

---

### 4.2 Defining Tools

#### (a) Prebuilt: DuckDuckGo Search

```python
search = DuckDuckGoSearchRun()
```

#### (b) Custom: Calculator

```python
@tool
def calculator(first_num: float, second_num: float, operation: str) -> float:
    """Perform basic arithmetic: add, subtract, multiply, divide"""
    if operation == "add":
        return first_num + second_num
    elif operation == "subtract":
        return first_num - second_num
    elif operation == "multiply":
        return first_num * second_num
    elif operation == "divide":
        return first_num / second_num
    else:
        return "Invalid operation"
```

#### (c) Custom: Stock Price

```python
@tool
def get_stock_price(symbol: str) -> str:
    """Fetch the current stock price for a company symbol using AlphaVantage API"""
    api_key = os.getenv("ALPHAVANTAGE_API_KEY")
    url = f"https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol={symbol}&apikey={api_key}"
    response = requests.get(url).json()
    return response
```

---

### 4.3 Binding Tools to LLM

```python
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
tools = [search, calculator, get_stock_price]

llm_with_tools = llm.bind_tools(tools)
```

---

### 4.4 Graph Setup

```python
from langgraph.graph import StateGraph, END

# Define state
class State(dict):
    messages: list

graph = StateGraph(State)

# Chat Node
def chat_node(state: State):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": messages + [response]}

graph.add_node("chat", chat_node)

# Tool Node
tool_node = ToolNode(tools)
graph.add_node("tools", tool_node)

# Edges
graph.add_edge("start", "chat")
graph.add_conditional_edges("chat", tools_condition)  # decides: End or Tools
graph.add_edge("tools", "chat")  # loop-back
graph.add_edge("chat", END)

# Compile
chatbot = graph.compile()
```

---

### 4.5 Example Usage

```python
# Normal chat
print(chatbot.invoke({"messages": ["Hello!"]}))

# Calculation
print(chatbot.invoke({"messages": ["What is 25 * 32?"]}))

# Stock price
print(chatbot.invoke({"messages": ["What is the stock price of Tesla?"]}))

# Multi-step reasoning
print(chatbot.invoke({"messages": ["What is the stock price of Apple? How much would 15 shares cost?"]}))
```

---

## 5. Integration with Frontend (Streamlit)

* **Problem**: Both AI responses and raw tool outputs were being streamed.
* **Fix**: Only stream AI messages, not tool messages.

```python
from langchain_core.messages import AIMessage

# In streaming loop:
if isinstance(msg, AIMessage):
    st.write(msg.content)
```

---

## 6. Adding Status Updates (UX Improvement)

* Use Streamlit’s **status container** to show:

  * Which tool is being used
  * Whether chatbot is in normal chat or tool mode

👉 This improves user experience by making chatbot behavior transparent.

---

## 7. Key Takeaways

* **Tools extend LLMs** beyond text → enable actions.
* **Tool Node** in LangGraph: handles all tools.
* **Tools Condition**: routes flow between LLM & tools.
* Workflow should **loop back tool outputs into LLM** → ensures:

  * Refined, human-friendly responses
  * Multi-step reasoning

---

## 8. Homework

* Try adding your own tool!
  Examples:

  * Weather API tool
  * Wikipedia summarizer
  * Currency converter

---

✅ After this lecture, you should be able to:

* Understand what **tools** are in LangGraph
* Differentiate between **prebuilt** and **custom tools**
* Build a chatbot workflow that **talks + acts**
* Handle **multi-step reasoning** with looped execution
