# Lecture Notes: Tools in LangChain (Part 1 of Agents Series)

## 1. Introduction & Context

* This lecture marks the **start of Part 3** of the LangChain playlist.
* Previous parts:

  * **Part 1**: Fundamentals (components like Models, Prompts, Chains, etc.)
  * **Part 2**: Retrieval-Augmented Generation (RAG) systems (document loaders, text splitters, vector stores, retrievers, building retrieval-based systems).
* **Part 3**: Focuses on **Agents**.

  * First topic: **Tools**.
  * Next: Tool Calling, then Agents (using LLMs, tools, and tool calling together).

---

## 2. What is a Tool?

* Tools extend LLMs by giving them the ability to **take actions**, not just think and speak.
* **LLMs’ core abilities**:

  1. **Reasoning** (can think, break down a problem).
  2. **Language generation** (can speak/answer).
* **Limitations of LLMs**:

  * Cannot fetch live data (e.g., weather).
  * Cannot reliably solve complex math.
  * Cannot interact with APIs, databases, or run code on their own.
* **Analogy**: LLM is like a human body with a brain but without hands/legs.
* **Tools = Hands & Legs for LLMs**.

  * A tool is just a **Python function** packaged so that the LLM can understand and call it.
  * Example: a train booking function → once connected, the LLM can execute bookings.

---

## 3. Tools in LangChain

Two types:

1. **Built-in Tools**

   * Prebuilt, production-ready tools provided by LangChain (e.g., search, Wikipedia, code execution, API requests).
   * Require minimal or no setup.
2. **Custom Tools**

   * Created when you need functionality specific to your app (e.g., internal APIs, company-specific logic).

---

## 4. Connection Between Tools & Agents

* **Agent** = LLM + Tools.
* Agent = autonomous system that:

  1. Thinks/reasons (LLM).
  2. Takes actions (via Tools).
* Thus, tools are essential for building agents.

---

## 5. Built-in Tools in LangChain

### Examples:

* **DuckDuckGoSearchRun** → Web search.
* **WikipediaQueryRun** → Summarized Wikipedia queries.
* **PythonREPLTool** → Execute raw Python code.
* **ShellTool** → Run shell/console commands.
* **RequestsGetTool** → Send HTTP requests.
* **GmailSendMessageTool** → Send Gmail messages.
* **Slack tool** → Interact with Slack.
* **SQL Database Query Tool** → Run SQL queries.

### Example: DuckDuckGo Search

```python
from langchain_community.tools import DuckDuckGoSearchRun

search_tool = DuckDuckGoSearchRun()
result = search_tool.invoke("IPL news")
print(result)
```

* The LLM fetches real-time web data → useful when knowledge cutoff is outdated.

### Example: Shell Tool

```python
from langchain_experimental.tools import ShellTool

shell_tool = ShellTool()
print(shell_tool.invoke("whoami"))  # shows current user
print(shell_tool.invoke("ls"))      # lists files in working directory
```

⚠️ Warning: Shell tool is **powerful but risky** (can delete files).

---

## 6. Custom Tools

When to create custom tools:

* No built-in tool exists for your use case.
* Need to call your own APIs.
* Encapsulate business logic unique to your app.
* Interact with your own databases/products.

### Method 1: Using `@tool` Decorator

Steps:

1. Write a Python function.
2. Add type hints & docstring (recommended).
3. Decorate with `@tool`.

Example:

```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

# Invoke the tool
result = multiply.invoke({"a": 3, "b": 5})
print(result)  # 15
```

Tool attributes:

```python
print(multiply.name)          # "multiply"
print(multiply.description)   # "Multiply two numbers."
print(multiply.args)          # schema of required arguments
```

---

### Method 2: Using `StructuredTool` + Pydantic

* More **strict schema enforcement** via Pydantic models.
* Good for **production-grade tools**.

Example:

```python
from langchain.tools import StructuredTool
from pydantic import BaseModel, Field

class MultiplyInput(BaseModel):
    a: int = Field(..., description="First number")
    b: int = Field(..., description="Second number")

def multiply_func(a: int, b: int) -> int:
    return a * b

multiply_tool = StructuredTool.from_function(
    func=multiply_func,
    name="multiply",
    description="Multiply two integers",
    args_schema=MultiplyInput
)

print(multiply_tool.invoke({"a": 3, "b": 5}))  # 15
```

---

### Method 3: Using `BaseTool` Class

* The **abstract base class** for all tools in LangChain.
* Useful when you need **deep customization** or async support.

Example:

```python
from langchain.tools import BaseTool
from pydantic import BaseModel, Field

class MultiplyInput(BaseModel):
    a: int = Field(..., description="First number")
    b: int = Field(..., description="Second number")

class MultiplyTool(BaseTool):
    name = "multiply"
    description = "Multiply two integers"
    args_schema = MultiplyInput

    def _run(self, a: int, b: int) -> int:
        return a * b

multiply_tool = MultiplyTool()
print(multiply_tool.invoke({"a": 3, "b": 5}))  # 15
```

Advantages:

* Full control (e.g., async methods).
* Flexible for advanced/prod scenarios.

---

## 7. Toolkits

* **Definition**: A collection of related tools grouped together for convenience and reusability.
* Example: **Google Drive Toolkit** may include:

  * Upload tool
  * Search tool
  * Read file tool
* **Benefit**: Reusability across multiple applications.

### Example: Math Toolkit

```python
from langchain_core.tools import tool

@tool
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

class MathToolkit:
    def get_tools(self):
        return [add, multiply]

toolkit = MathToolkit()
for tool in toolkit.get_tools():
    print(tool.name, tool.description, tool.args)
```

---

## 8. Summary

* **Tools** = give LLMs the ability to act.
* **Built-in Tools**: Quick, pre-made solutions (e.g., search, SQL, email).
* **Custom Tools**: When your use case is unique (via `@tool`, `StructuredTool`, or `BaseTool`).
* **Toolkits**: Group of related tools for reusability.
* Tools are the **foundation of agents**, as they allow LLMs to reason *and* act.

**Next lecture** → Tool Calling (connecting tools with LLMs).

---

✅ With these notes, you now have a structured summary, full explanations, and runnable Python examples.
