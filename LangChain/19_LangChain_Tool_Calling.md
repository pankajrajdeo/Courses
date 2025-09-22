# Lecture Notes: LangChain Tool Binding & Tool Calling

## 1. Introduction

* Continuing the **LangChain playlist**: focus on building **AI agents**.
* Previous video introduced the concept of **Tools**.
* Today’s topic:

  * **Tool Binding** → connecting LLMs with tools.
  * **Tool Calling** → letting LLMs decide when/how to use tools.
  * **Tool Execution** → actually running the tool.
* Final application: **Real-time Currency Converter** using LLM + external API.

---

## 2. Quick Revision: Powers & Limitations of LLMs

### LLM Capabilities:

1. **Reasoning** → Understand & break down questions.
2. **Output Generation** → Produce answers from parametric knowledge.

Analogy: An intelligent human who can think and speak.

### Limitation:

* LLMs **cannot perform tasks** (e.g., querying a DB, posting on Twitter, fetching real-time weather).
* They lack “hands & legs.”

### Solution:

* Use **Tools** = functions (often Python functions) that perform tasks.
* Tools can be:

  * Built-in (e.g., DuckDuckGo search, shell command runner).
  * Custom (user-defined functions).
  * Toolkits (bundled sets of tools).

---

## 3. Tool Binding

### Definition:

* The process of **registering tools** with an LLM.
* Allows the LLM to know:

  1. Which tools are available.
  2. What each tool does (via description).
  3. Input format/schema required.

### Example: Multiplication Tool

```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Given two numbers A and B, returns their product."""
    return a * b

# Testing the tool
print(multiply.invoke({"a": 3, "b": 4}))  # Output: 12
```

### Binding the Tool to an LLM:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(api_key="your_api_key")
llm_with_tools = llm.bind_tools([multiply])
```

Now, `llm_with_tools` can **suggest calling `multiply`** whenever multiplication is needed.

---

## 4. Tool Calling

### Definition:

* Process where the **LLM decides** during a conversation that it needs a tool.
* LLM generates **structured output**:

  * Tool name.
  * Arguments.

### Example:

```python
response = llm_with_tools.invoke("Can you multiply 3 with 10?")
print(response.tool_calls)
```

Output:

```json
[{
  "id": "xyz123",
  "type": "tool_call",
  "name": "multiply",
  "args": {"a": 3, "b": 10}
}]
```

⚠️ **Important**:

* The LLM **does not execute** the tool.
* It only suggests → “Use tool X with these arguments.”
* Execution is programmer’s responsibility.

---

## 5. Tool Execution

### Definition:

* The step where the **programmer actually invokes** the tool using the arguments provided by LLM.
* Returns the result.

### Example:

```python
tool_call = response.tool_calls[0]   # Extract tool call
args = tool_call["args"]

result = multiply.invoke(args)
print(result)  # Output: 30
```

### Using Tool Messages:

* Executing with the entire tool call generates a **Tool Message**.
* Tool Message can be appended to conversation history → keeps context for LLM.

```python
messages = []
query = "Multiply 3 with 10"
messages.append(HumanMessage(query))

ai_message = llm_with_tools.invoke(messages)
messages.append(ai_message)

tool_result = multiply.invoke(ai_message.tool_calls[0])
messages.append(tool_result)

final = llm_with_tools.invoke(messages)
print(final.content)  # "The product of 3 and 10 is 30"
```

---

## 6. Example Application: Currency Conversion Tool

### Problem:

* LLMs can’t access **real-time exchange rates**.
* Solution: Build two tools:

  1. **Get Conversion Factor** → fetches rate from an API.
  2. **Convert** → multiplies base currency value with rate.

---

### Tool 1: Get Conversion Factor

```python
import requests
from langchain_core.tools import tool

API_KEY = "your_api_key"

@tool
def get_conversion_factor(base_currency: str, target_currency: str) -> float:
    """Fetch the currency conversion rate between base and target currencies."""
    url = f"https://api.exchangerate-api.com/v4/latest/{base_currency}"
    response = requests.get(url).json()
    return response["rates"][target_currency]
```

---

### Tool 2: Convert

```python
@tool
def convert(base_value: int, conversion_rate: float) -> float:
    """Given a conversion rate, calculates target currency value."""
    return base_value * conversion_rate
```

---

### Binding Tools

```python
llm = ChatOpenAI(api_key="your_api_key")
llm_with_tools = llm.bind_tools([get_conversion_factor, convert])
```

---

### Handling Injected Tool Arguments

Problem: LLM may try to guess conversion rates from training data instead of waiting for API response.
Solution: Use **InjectedToolArgument**.

```python
from langchain_core.tools import InjectedToolArg
from typing import Annotated

@tool
def convert(
    base_value: int,
    conversion_rate: Annotated[float, InjectedToolArg()]
) -> float:
    return base_value * conversion_rate
```

Now LLM won’t guess; programmer injects the real value.

---

### Execution Flow

1. User: *“Convert 10 USD to INR”*.
2. LLM suggests tool calls:

   * `get_conversion_factor("USD", "INR")`
   * `convert(10, conversion_rate)`
3. Programmer executes in sequence:

   ```python
   tool_msg1 = get_conversion_factor.invoke({"base_currency": "USD", "target_currency": "INR"})
   conversion_rate = tool_msg1["rate"]

   tool_msg2 = convert.invoke({"base_value": 10, "conversion_rate": conversion_rate})
   ```
4. Final LLM response:

   ```
   The conversion factor between USD and INR is 85.34.
   Based on this, converting 10 USD gives 853.4 INR.
   ```

---

## 7. Is This an AI Agent?

* **Answer: No.**
* Reason:

  * True agents are **autonomous**.
  * They break down tasks, call tools, and sequence execution **without programmer intervention**.
  * Our example: programmer controlled the logic & execution.
* Next step (future video): building a **real autonomous agent** with LangChain.

---

# ✅ Summary

* **LLMs have reasoning + output powers but no action capability.**
* **Tools** = functions enabling LLMs to act.
* **Tool Binding** = registering tools with LLM.
* **Tool Calling** = LLM suggests tool + args.
* **Tool Execution** = programmer runs tool & feeds result back.
* **Application**: Currency conversion app using real-time API + two tools.
* This was **not an AI Agent** (no autonomy yet), but a step toward building one.
