# Designing Effective AI Agent Tools

---

## 1. Introduction: Why Tools Matter for AI Agents

* The effectiveness of an AI agent depends on:

  * **The intelligence of the underlying model** (e.g., Claude, GPT-4/5).
  * **The quality of tools available** to it.
* Unlike traditional deterministic software:

  * Same input → always same output.
  * In **agentic systems**, same input → different strategies, tool calls, or follow-ups.
* Tools are not just APIs; they’re designed for **agents**, not humans.
  → Must be ergonomic for LLMs.

---

## 2. The Tool-Building Process (Anthropic’s 4-Step Workflow)

1. **Prototype** – Build a quick initial version of the tool.
2. **Evaluation (Evals)** – Run real-world test cases to measure effectiveness.
3. **Collaboration** – Use coding agents (e.g., Claude Code) to refine descriptions & specs.
4. **Iteration** – Improve tools with agent feedback and repeated evals.

   > *“If you can’t measure it, you can’t improve it.”*

---

## 3. Key Principles for Writing Better Tools

### 3.1 Choose the Right Tools

* **More tools ≠ better outcomes.**
* Avoid wrapping every existing API blindly.
* Focus on **human-like usage patterns**:

  * **Search > Browse**
  * **Filter > Scan**
  * **Navigate > Iterate**

#### Example: Address Book Search

* **Bad design:**

  * Tool A: `listContacts()` → returns 500 entries (25k tokens wasted).
  * Tool B: `getContactByIndex()`.
* **Better design:**

  ```python
  def searchContacts(query: str) -> list[dict]:
      """
      Search contacts by name, email, or phone number.
      Returns only relevant matches with name, phone, email.
      """
  ```

  * Returns **only the subset needed**.
  * Token-efficient and more useful to the agent.

---

### 3.2 Group Tools with Namespaces

* Helps agents disambiguate similar tool names.
* Use **prefixes or suffixes** consistently.

#### Example

* Instead of:

  * `searchMessages`, `searchTickets`, `searchTasks`
* Use:

  * `slack.searchMessages`
  * `jira.searchTickets`
  * `asana.searchTasks`

⚠️ Agents sometimes perform differently if namespace is a **prefix** vs. **suffix**. Test both.

---

### 3.3 Return Structured, High-Signal Outputs

* Don’t flood the agent with low-value identifiers.
* Prefer **natural language + structured schema**.

#### Example: Bad Response

```json
{
  "user_id": "a87f9-23kd-99sd",
  "img_token": "x84h...ff3"
}
```

#### Example: Good Response

```json
{
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "profile_image": "https://example.com/jane.jpg",
  "user_id": "a87f9-23kd-99sd"  // keep technical IDs only if needed
}
```

* Use formats LLMs are **trained on**:

  * Claude/OpenAI → strong with **XML** and **JSON**.
  * Markdown works but is less structured.

---

### 3.4 Optimize for Token Efficiency

* Large tool responses → context window bloat → worse reasoning.
* Strategies:

  * **Pagination** (`page=1, size=50`).
  * **Filtering** (return top-k matches).
  * **Summarization** (only high-signal content).

#### Example: Paginated Logs

```python
def searchLogs(query: str, page: int = 1, page_size: int = 100) -> dict:
    """
    Return paginated logs matching query.
    Includes timestamp, message, and surrounding context.
    """
```

* Default Claude limit: ~25k tokens per tool response.
* Many tool calls in one task → adds up quickly.

---

### 3.5 Prompt-Engineering Tool Descriptions

* Tool descriptions are like **instructions to interns**:

  * Be **clear, explicit, unambiguous**.
  * Define **inputs, outputs, and formats** precisely.
* Avoid vague descriptions (agents may hallucinate usage).

#### Example: Bad Tool Spec

```json
{
  "name": "search",
  "description": "Search for something",
  "parameters": { "q": "string" }
}
```

#### Example: Good Tool Spec

```json
{
  "name": "searchLogs",
  "description": "Search error logs for a given keyword or customer ID. 
                 Returns only relevant entries with surrounding context (±3 lines).",
  "parameters": { 
      "query": "string (e.g., 'customer_id=9182' or 'payment_failed')" 
  }
}
```

---

## 4. Evaluation: Testing Tools in the Real World

### 4.1 Why Evaluations Matter

* Synthetic examples → misleading results.
* Realistic, multi-step tasks → better robustness.

### 4.2 Example: Weak vs Strong Tasks

* **Weak Task:**

  * “Schedule a meeting with Jane next week.”
* **Strong Task:**

  * “Schedule a meeting with Jane next week to discuss the Acme project.
    Attach notes from last planning meeting and reserve a conference room.”

👉 Strong tasks force the agent to use **multiple tools in sequence**.

### 4.3 Evaluation Metrics

* **Accuracy** of task completion.
* **Number of tool calls** per task.
* **Token usage** per call.
* **Runtime**.
* **Tool errors** (invalid params, etc.).

---

## 5. Collaborating with Agents to Improve Tools

* Agents can:

  * Spot confusing descriptions.
  * Suggest refactoring.
  * Identify redundant tools.
* Workflow:

  1. Run evals.
  2. Concatenate transcripts.
  3. Feed back into Claude Code.
  4. Let the agent rewrite/refine tools automatically.

---

## 6. Browser Automation Example (Browserbase + Stagehand)

* Real-world agent tool for **web browsing**.
* Features:

  * Automates Puppeteer/Playwright via natural language.
  * Includes “stagehand” open-source SDK.
  * Supports:

    * Visiting pages
    * Clicking elements
    * Scraping structured data
* **Natural Language Example:**

  ```python
  agent.execute("Go to example.com and extract all product names and prices")
  ```

---

## 7. Final Takeaways

1. **Fewer but smarter tools** work better than many simple wrappers.
2. **Namespace clearly** to prevent confusion.
3. **Return structured, high-signal outputs** (avoid dumping raw data).
4. **Token efficiency is critical**—optimize context usage.
5. **Well-engineered tool descriptions** = higher reliability.
6. **Real-world evaluation > synthetic tests**.
7. **Iterate with agents themselves** to improve tools.
