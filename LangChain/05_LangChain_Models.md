# Lecture Notes: LangChain's Model Component

Welcome to a detailed exploration of LangChain's **Model Component**. These notes are designed to provide a deep understanding of how to work with various AI models within the LangChain framework.

-----

## 1\. Introduction and Recap

This lecture focuses on the **Model component**, one of the core building blocks of LangChain. We've previously covered:

  * **LangChain Introduction:** What it is, why we need it, and what kind of applications we can build.
  * **LangChain Components:** An overview of key components like **Models, Prompts, Agents, and Chains**.

Today, we'll dive deep into **Models**, the very first component we discussed. The goal is to not only understand the theory but also to build and interact with these models through code, which makes learning more engaging.

-----

## 2\. What are Models in LangChain?

The model component in LangChain acts as a crucial **interface** that facilitates interactions with various language and embedding models.

### The Problem LangChain Solves

Different AI models from different companies (like OpenAI, Anthropic, Google) have their own unique APIs and ways of interacting with them. This creates a fragmented development experience. LangChain's model component provides a **common interface** so you can easily connect to any of these models without rewriting your code every time you switch.

### Two Types of Models in LangChain

LangChain primarily deals with two types of models:

| **Model Type** | **Input** | **Output** | **Purpose / Use Case** |
| :--- | :--- | :--- | :--- |
| **Language Models** | Text (string) | Text (string) | Used for **conversational tasks** like building chatbots and agents. They understand a query and generate a textual response. |
| **Embedding Models** | Text (string) | A series of numbers (embeddings) | Used for **semantic search** and understanding the context of text. The numbers (**vectors**) represent the meaning of the text, enabling applications like RAG (Retrieval-Augmented Generation). |

-----

## 3\. Language Models: LLMs vs. Chat Models

Within the category of Language Models, there are two distinct types: **LLMs (Large Language Models)** and **Chat Models**. While both deal with text, their purpose and how they handle input and output are different.

| **Feature** | **LLMs** (e.g., GPT-3.5-turbo-instruct) | **Chat Models** (e.g., GPT-4) |
| :--- | :--- | :--- |
| **Purpose** | Free-form text generation, summarization, translation. They are **general-purpose**. | Multi-turn conversations, chatbots, virtual assistants. They are **specialized for conversational tasks**. |
| **Input/Output** | Takes a **single string** as input and returns a **single string** as output. | Takes a **sequence of messages** (a conversation history) as input and returns a **sequence of messages** as output. |
| **Training** | Trained on a broad range of text data (books, articles). | Initially trained on broad data, then **fine-tuned on chat datasets** to better handle conversations. |
| **Memory** | Do **not** have a built-in concept of memory. | Designed to remember and understand the **conversation history**. |
| **Role Awareness** | Do **not** support assigning roles (e.g., "You are a doctor"). | Can understand and adopt specific **roles** defined by system messages (e.g., `role="system"`). |

**Note:** LangChain is increasingly deprecating support for LLMs in favor of Chat Models, as modern AI applications are predominantly conversational. It is recommended to use Chat Models for new projects.

-----

## 4\. Setting Up Your Development Environment ⚙️

Before we start coding, we need to set up our project.

### Step-by-Step Setup

1.  **Create a Project Folder:** Create a new folder (e.g., `langchain_models`).
2.  **Open in VS Code:** Open the folder in VS Code.
3.  **Create a Virtual Environment:**
      * Open a new terminal in VS Code.
      * Run `python -m venv venv`.
4.  **Activate the Virtual Environment:**
      * Run `venv\Scripts\activate`.
5.  **Create a `requirements.txt` file:**
      * Create a file named `requirements.txt` in your project folder.
      * Add the following libraries:
        ```
        langchain
        langchain-openai
        langchain-anthropic
        langchain-google-genai
        langchain-huggingface
        python-dotenv
        ```
6.  **Install Libraries:**
      * Run `pip install -r requirements.txt`.
      * This will install all the necessary packages for the lecture.
7.  **Verify Installation:**
      * Create a file `test.py`.
      * Add `import langchain; print(langchain.__version__)`.
      * Run `python test.py`. The version number should print, confirming LangChain is installed.
8.  **Organize your project:** Create three subfolders: `llms`, `chat_models`, and `embedding_models`. We will place our code demos in these folders.

-----

## 5\. Coding Demos

### 5.1. Working with LLMs (using OpenAI)

We will use OpenAI's API to demonstrate an LLM.

#### Pre-requisites: OpenAI API Key

  * Go to **platform.openai.com**.
  * Create an account and **add at least $5 of credit**. As of now, OpenAI does not provide free credits for API usage.
  * Once you have credit, go to `Settings` -\> `API Keys`.
  * Click `Create new secret key`. Copy the key and save it.
  * **Important:** Never hardcode your API key in your code. Store it as an environment variable.

#### Storing the API Key in a `.env` file

1.  Create a file named `.env` in the root of your project folder.
2.  Add your API key in the following format:
    ```
    OPENAI_API_KEY="your_api_key_here"
    ```

#### Code Demo: LLM

  * Create a new file `llm_demo.py` inside the `llms` folder.
  * This code will use the `langchain-openai` integration package and the `LLM` component.

<!-- end list -->

```python
# llm_demo.py

# 1. Import necessary libraries
from langchain_openai import OpenAI
from dotenv import load_dotenv

# 2. Load environment variables from .env file
load_dotenv()

# 3. Create an LLM object
# We specify the model we want to use.
llm = OpenAI(model="gpt-3.5-turbo-instruct")

# 4. Invoke the LLM with a prompt
# The `invoke` method is a key function in LangChain for interacting with models.
prompt = "What is the capital of India?"
result = llm.invoke(prompt)

# 5. Print the result
# The output will be a simple string.
print(result)
```

**Output:**

```
The capital of India is New Delhi.
```

-----

### 5.2. Working with Chat Models (using OpenAI)

Now, let's explore the recommended way to interact with models—using the Chat Model interface.

#### Code Demo: Chat Model

  * Create a new file `chat_model_openai.py` inside the `chat_models` folder.
  * Notice the subtle but crucial change: we import `ChatOpenAI` instead of `OpenAI`. `ChatOpenAI` inherits from `BaseChatModel`, whereas `OpenAI` inherits from `BaseLLM`. This distinction is key to how LangChain handles these models.

<!-- end list -->

```python
# chat_model_openai.py

# 1. Import necessary libraries
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

# 2. Load environment variables
load_dotenv()

# 3. Create a Chat Model object
# We'll use a more advanced model like GPT-4.
model = ChatOpenAI(model="gpt-4")

# 4. Invoke the model with a prompt
prompt = "What is the capital of India?"
result = model.invoke(prompt)

# 5. Print the result
# The output is not a simple string; it's an object containing metadata.
print(result)

# To get the clean answer, we access the 'content' attribute.
print(result.content)
```

**Output:**
The first `print` statement will show a structured object with details like `content`, `usage_metadata`, etc. The second `print` statement will give you the clean answer:

```
The capital of India is New Delhi.
```

-----

### 5.3. Additional Parameters: The Temperature Parameter

Chat models (and some LLMs) allow you to set additional parameters to control their behavior. One of the most important is **`temperature`**.

  * **What it is:** A parameter that controls the **randomness** of the model's output. A higher temperature results in more random and creative responses, while a lower temperature makes the output more deterministic and predictable.
  * **Values:** Typically ranges from `0` to `2`.
  * **Recommendation:**
      * **Low Temperature (0.0 - 0.3):** Use for factual, deterministic tasks (e.g., code generation, mathematical problems).
      * **Medium Temperature (0.5 - 0.7):** Good for general question-answering and explanations.
      * **High Temperature (0.9 - 1.2):** Best for creative writing, storytelling, and generating diverse ideas.

#### Code Demo: Using Temperature

  * Modify the `chat_model_openai.py` file to include the `temperature` parameter.

<!-- end list -->

```python
# chat_model_openai.py (modified)

from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

# Set a low temperature for predictable answers
model = ChatOpenAI(model="gpt-4", temperature=0)

# Change the prompt to something that can have multiple answers
prompt = "Suggest me five Indian male names."
result = model.invoke(prompt)

print(result.content)
```

By changing the `temperature` and the prompt, you can observe how the model's creativity and diversity of answers change.
