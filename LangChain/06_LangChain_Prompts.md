# Lecture Notes: **LangChain Prompts**

## 1. Introduction

* Previous lecture covered **LangChain models** (first key component).
* Today’s lecture: **Prompts** (second key component).
* Prompts are central to how LLMs work.
* Lecture goal: full **end-to-end understanding** of prompts, static vs. dynamic, templates, chat prompts, messages, and placeholders.

---

## 2. Quick Recap of Past Videos

1. **Intro to LangChain** – Why it’s needed as a framework.
2. **6 Core Components of LangChain** – With real-life examples.
3. **Deep Dive into Models** – How LLMs integrate with LangChain.

This video: Focus entirely on **Prompts**.

---

## 3. Error Correction from Last Lecture (Temperature Misconception)

* **Mistake in previous explanation**: Said temperature ranges **0–2**, with 0 = deterministic and 2 = creative.
* **Correct Explanation**:

  * **Temperature = 0** → deterministic.

    * Same input = same output every time.
  * **Temperature > 0 (e.g., 0.5–1.5)** → more randomness.

    * Same input may produce **different outputs** (more creative).
* **Code Example**:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0) 
prompt = "Write a 5-line poem on cricket."
print(llm.invoke(prompt))

# Running again with temperature=0 → same output every time.
# With temperature=1.5 → varied creative outputs each run.
```

* **Key takeaway**:

  * **Use low temperature** for consistency (e.g., summarization tools).
  * **Use higher temperature** for creativity (e.g., story generation).

---

## 4. What Are Prompts?

* **Definition**: The message you send to the LLM (text, image, audio, or video).
* **Types**:

  * Text prompts (most common).
  * Multimodal prompts (images, sound, video – becoming more prominent).
* **Why important?**

  * LLM output is **highly sensitive** to prompt phrasing.
  * Even small changes → drastically different outputs.
* **Emerging field**: **Prompt Engineering** – designing effective prompts for LLMs.

---

## 5. Static vs. Dynamic Prompts

### Static Prompts

* **Definition**: Hardcoded prompts written by programmer.
* **Example**:

```python
result = llm.invoke("Summarize the paper 'Attention is All You Need'")
print(result)
```

* **Problems**:

  * Users must write the whole prompt themselves.
  * Output consistency depends entirely on user phrasing.
  * Risk of **typos** or poor instructions leading to poor results.

---

### Dynamic Prompts

* **Definition**: Predefined **prompt templates** where only **specific fields** are filled by the user.

* **Benefits**:

  * **Consistent style & quality**.
  * Control over output format.
  * Avoids user errors (spelling, vague instructions).

* **Code Example (with Streamlit UI):**

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
import streamlit as st

llm = ChatOpenAI(model="gpt-4")

# UI
st.header("Research Assistant Tool")
paper = st.selectbox("Select Paper", ["Attention is All You Need", "Word2Vec"])
style = st.selectbox("Explanation Style", ["Simple", "Math-heavy", "Code-heavy"])
length = st.selectbox("Summary Length", ["Short", "Medium", "Long"])

# Prompt Template
template = """
Please summarize the research paper titled '{paper}'
in a {style} style and in {length} length.
Include equations if present and use analogies.
"""
prompt_template = PromptTemplate.from_template(template)

# Fill template
prompt = prompt_template.invoke({"paper": paper, "style": style, "length": length})

# Call model
result = llm.invoke(prompt)
st.write(result.content)
```

* **Result**:

  * Users just select dropdowns.
  * Application maintains **consistent formatting**.

---

## 6. PromptTemplate vs. f-Strings

* **Why not just use f-strings?**

  * Both allow variable insertion, but `PromptTemplate` has advantages:

### Benefits of `PromptTemplate`:

1. **Validation** – Ensures placeholders match inputs.

   * Errors detected **before runtime**.
2. **Reusability** – Templates can be stored in JSON and reloaded.

   ```python
   prompt_template.save("template.json")
   loaded_template = load_prompt("template.json")
   ```
3. **Integration** – Works seamlessly with **LangChain chains**.

---

## 7. Messages in LangChain

* LLM conversation = **list of messages**.

* **3 Types of Messages**:

  1. **SystemMessage** – Defines role/context.

     * Example: `"You are a helpful assistant."`
  2. **HumanMessage** – User input.

     * Example: `"Tell me about LangChain."`
  3. **AIMessage** – LLM response.

* **Code Example**:

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage

llm = ChatOpenAI(model="gpt-4")

messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="Tell me about LangChain")
]

response = llm.invoke(messages)
print(response.content)
```

* **Why useful?**

  * Keeps **roles clear**.
  * Enables **multi-turn conversations** with context.

---

## 8. Chat Prompt Templates

* Used when working with **multiple messages** instead of one.
* Allows **dynamic message creation**.

### Example:

```python
from langchain_core.prompts import ChatPromptTemplate

chat_template = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful {domain} expert."),
    ("human", "Explain {topic} in simple terms.")
])

prompt = chat_template.invoke({"domain": "cricket", "topic": "Duckworth-Lewis method"})
print(prompt)
```

* **Output**: A structured prompt with placeholders replaced.

---

## 9. MessagePlaceholders

* **Definition**: A placeholder inside `ChatPromptTemplate` used to dynamically insert an **entire chat history**.
* **Use Case**: Chatbots that must maintain context across sessions.

### Example:

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

chat_template = ChatPromptTemplate.from_messages([
    ("system", "You are a customer support agent."),
    MessagesPlaceholder("chat_history"),
    ("human", "{query}")
])

chat_history = [
    ("human", "I want a refund for order 12345"),
    ("ai", "Your refund request is being processed."),
]

final_prompt = chat_template.invoke({
    "chat_history": chat_history,
    "query": "Where is my refund?"
})

print(final_prompt)
```

* **Result**:

  * Old history gets injected automatically.
  * LLM has full **conversation context**.

---

## 10. Conclusion

* **Prompts are the foundation** of LLM interaction.
* We learned:

  * Static vs. Dynamic Prompts.
  * Why PromptTemplate is better than f-strings.
  * Messages (System, Human, AI).
  * ChatPromptTemplate for multi-turn convos.
  * MessagePlaceholders for chat history.
* Next steps: **Prompt Engineering techniques** (e.g., Chain of Thought, Few-shot learning).

---

✅ With these notes, you should be able to **reconstruct the entire lecture**, revise key concepts, and even implement working examples in Python.

Would you like me to also **draw a diagram/flowchart** showing how prompts flow through LangChain (from user → template → LLM → response) for quick visual revision?
