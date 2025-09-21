## Introduction to the LangChain & LangGraph Ecosystem

This lecture is the third in a series on agentic AI using LangGraph. The previous lectures covered the key differences between generative AI and agentic AI, and a detailed overview of what agentic AI is, including its key characteristics, components, and a practical example of automated hiring. Now, we'll shift our focus to the practical development of agentic AI applications.

Building agentic AI applications from scratch is complex. Therefore, using frameworks is essential. We will explore several existing frameworks like **CrewAI**, **Microsoft's Autogen**, and **LangGraph**. This series will focus on **LangGraph**, which was developed by the **LangChain** team. LangGraph is a very popular framework and is considered one of the top choices for building agentic AI applications.

---

## The Core Goals of This Lecture

The main goals of this lecture are:

* **To provide a deep intuition for why LangGraph exists:** We'll explore the problems that LangChain couldn't solve, which led to the creation of LangGraph.
* **To give a technical overview of LangGraph:** We'll dive into what LangGraph is and how it works.
* **To cover the key differences between LangChain and LangGraph:** This will help you understand when to use each framework.

By the end of this lecture, you'll be able to look at any application and intuitively understand whether you should use LangChain or LangGraph to build it.

---

### Prerequisites

To get the most out of this lecture, you should have a basic understanding of **LangChain**. This includes knowing what LangChain is, what you can do with it, and how to write basic code using its components. If you're not familiar with LangChain, it's highly recommended to watch the introductory videos on LangChain before continuing.

---

## A Quick Recap of LangChain

**LangChain** is an open-source library designed to simplify the process of building **LLM-based applications**. The rise of LLMs has led to a new wave of software that integrates these models, such as chatbots and browser plugins. LangChain simplifies the development of these applications by providing modular **building blocks** that can be combined to create complex workflows.

The key modular building blocks of LangChain are:

* **Models:** Provides a unified interface to interact with various LLM providers, like OpenAI, Anthropic, or Hugging Face. This makes it easy to switch between different models without major code changes.
* **Prompts:** Helps in engineering prompts for LLMs effectively.
* **Retrievers:** Fetches relevant documents from a vector store or knowledge base, which is crucial for building **RAG (Retrieval-Augmented Generation)**-based applications.
* **Chains:** The most powerful feature of LangChain. Chains allow you to link different components together, where the output of one component automatically becomes the input for the next. This abstraction makes it easy to build multi-step workflows.

Using these building blocks, LangChain enables you to create various applications:

* **Simple Conversational Workflows:** Build chatbots or text summarizers by chaining a user prompt to an LLM.
* **Multi-step Workflows:** Construct a sequence of operations, like generating a detailed report from a topic and then creating a summary from that report.
* **RAG-based Applications:** Create systems that can chat with your company's documents by retrieving relevant information and feeding it to an LLM.
* **Simple Agents:** Build basic agents by connecting LLMs to **tools** (e.g., APIs, Python functions), allowing the LLM to decide which tool to use based on the user's query.

---

## The Automated Hiring Workflow: A Case Study

Now, we'll consider a more complex workflow: the **automated hiring process**. This flow is not a true agentic AI application but rather a **predefined workflow** (or a complex chain) created by a developer. 

---

### Workflows vs. Agentic Applications

It's crucial to understand the difference between a workflow and an agentic application.

* **Workflows** are systems where LLMs and tools are orchestrated through **predefined, static code paths**. The developer designs the entire flow chart in advance, and the system follows the exact same steps every time.
* **Agentic Applications**, on the other hand, are systems where the **LLM dynamically directs its own processes** and tool usage. The agent decides what steps to take, in what order, to accomplish a task. The "flow chart" is not static; it's generated on the fly by the agent itself.

Our automated hiring example is a workflow because the entire process is pre-planned and static.

---

### Challenges of Building This Workflow in LangChain

Attempting to build this complex, non-linear workflow using LangChain presents significant challenges. We will discuss eight key challenges.

### 1. Control Flow Complexity

LangChain is excellent for building **linear chains** (where A leads to B, which leads to C). However, our hiring workflow is highly non-linear due to:

* **Conditional branches:** The flow changes based on conditions (e.g., "if enough applications are received").
* **Loops:** The process repeats until a condition is met (e.g., "keep modifying the job description until it's approved").
* **Jumps:** The flow can jump backward or forward in the process (e.g., "after waiting 48 hours, go back to monitor applications").

LangChain lacks built-in constructs for conditional branching, loops, and jumps. To implement these in LangChain, you must use regular **Python code**.

**Example:** To implement the loop for JD approval, you would need to write a `while` loop in Python. The LangChain components (like the chains for creating and approving the JD) would be nested inside this loop.

**The problem with this approach is the "glue code."** This is the custom Python code you write to connect LangChain's components. In a complex, non-linear application, you end up writing a large amount of glue code. This makes the system hard to maintain, debug, and work on in a team. This is the biggest flaw of LangChain for complex projects: it excels at linear workflows but falters when non-linearity is introduced.

---

### How LangGraph Solves This Problem

LangGraph represents the entire workflow as a **graph**.

* Each task (e.g., "create JD," "check approval") is a **node** in the graph.
* The connections between these nodes are called **edges**, which define the control flow.

Because a graph is inherently a non-linear data structure, LangGraph can easily represent and execute highly complex, non-linear workflows.

**Example:**
In LangGraph, you create a `Graph` object and add nodes to it. Each node is a Python function. Then, you define the edges that connect these nodes. The edges can be **conditional**, allowing you to specify different next steps based on the output of a node. For example, the "check approval" node can return "approved" or "not approved," and the edges from that node will route the workflow accordingly.

This graphical representation and built-in support for conditional logic and loops within the framework itself **eliminates the need for extensive glue code**, making complex applications much easier to build, maintain, and debug.
