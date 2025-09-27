# 🌌 Deep Agents

These notes combine **conceptual foundations, design patterns, prompting strategies, and code-level practices** into one cohesive guide to building **deep, long-horizon agents** with LangGraph.

---

## 🧭 I. What Are Deep Agents?

Deep agents are **long-horizon, context-rich, tool-intensive** AI systems designed to execute multi-step, complex tasks.

They overcome the limitations of **shallow agents** (turn-by-turn, memoryless, prone to drift) through **architecture + discipline**, not just bigger models.

**Core capabilities:**

1. **Plan workflows** → explicit TODO lists with status tracking.
2. **Persist context** → virtual file systems instead of bloated prompts.
3. **Isolate subtasks** → sub-agents with clean contexts.
4. **Reflect deliberately** → checkpoints to prevent runaway loops.

👉 Think of them as **project managers + researchers** rather than just chatbots.

---

## 🏛️ II. The Three Pillars of Context Engineering

Deep agents rest on **three golden patterns**:

1. **Task Planning (TODO Lists)**

   * Externalized executive function.
   * Forces explicit tracking of goals: `pending`, `in_progress`, `completed`.
   * Best practice: only **one in_progress** at a time.
   * Agents must **recite** their TODOs often → anti-drift technique.

2. **Context Offloading (Virtual File System)**

   * Ephemeral filesystem inside `DeepAgentState`.
   * Agents store raw research, drafts, summaries as files.
   * Benefits:

     * Avoids token bloat.
     * Makes memory persistent across turns.
     * Enables orientation (`ls`) and re-use (`read_file`).
   * Workflow: **Orient → Save → Research → Read → Answer.**

3. **Context Isolation (Sub-agents)**

   * Parent delegates tasks via `task(description, subagent_type)`.
   * Sub-agent starts with a **sterile context** containing only the description.
   * Prevents contamination of parent history.
   * Scaling patterns:

     * Fact-finding → 1 sub-agent.
     * Comparisons → 1 per entity.
     * Multi-faceted tasks → parallel sub-agents.

**Synergy:**
TODOs define tasks → Sub-agents execute them → Results offloaded to files → Parent integrates final synthesis.

---

## ⚙️ III. Architecture in LangGraph

Deep agents are implemented as **stateful cognitive systems**.

### Central State (`DeepAgentState`)

* Extends LangGraph’s `AgentState`.
* Fields:

  * `messages`: conversation history.
  * `todos`: structured task list.
  * `files`: virtual filesystem.
* `file_reducer`: merges file updates incrementally → supports parallel writes.
* **Principle:** All cognition lives in state, not hidden in prompts.

### The `Command` Pattern

* Tools return `Command` objects that mutate state.
* Example: `write_file` updates `files` + appends a `ToolMessage`.
* Makes tools **first-class drivers** of reasoning.

### Dependency Injection

* `InjectedState` → tools can read state.
* `InjectedToolCallId` → tie tool output to triggering message.
* Ensures clean continuity across tool calls.

---

## 🔧 IV. Tools as Cognitive Primitives

Tools are the “organs” of a deep agent.

### 1. TODO Tools

* `write_todos`: updates todo list.
* `read_todos`: recites list with emoji statuses.
* Rules:

  * Always send full updated list.
  * Keep it pruned and focused.
  * Batch into single TODO when possible.

### 2. File Tools

* `ls`: list files.
* `read_file`: safe read with offset/limit, per-line truncation.
* `write_file`: overwrite-only → avoids corruption.
* Usage cycle: **Orient → Save → Research → Read → Answer.**

### 3. Research Tools

* `tavily_search`:

  * Calls Tavily API.
  * Pulls HTML → converts to Markdown → summarizes (<150 words).
  * Generates descriptive filename with UUID suffix (no collisions).
  * Saves raw + summary to VFS.
  * Returns only **minimal summary** to context.
* `think_tool`: reflection checkpoint:

  * What did I find?
  * What’s missing?
  * Should I stop?

### 4. Sub-Agent Tool

* Factory pattern (`_create_task_tool`).
* `task(description, subagent_type)`:

  * Resets state to description.
  * Sub-agent executes independently.
  * Returns results as `ToolMessage` + merges files.

---

## 📜 V. Prompt Engineering & Context Strategy

Prompts act as **operating manuals** for the agent.

### Core Practices

1. **Structured formatting**: XML-like tags (`<Task>`, `<Hard Limits>`) + Markdown headers.
2. **Hyper-detailed tool descriptions**: not just API signatures, but usage rules, guardrails, and examples.
3. **Explicit workflows**: codify algorithms (Plan → Do → Reflect → Mark Complete).
4. **Forced reflection**: `think_tool` after every major step.
5. **Constraints**: `<Hard Limits>` define budgets (1–2 searches simple, max 5 complex).
6. **Structured outputs**: JSON schemas for summaries ensure reliability.

### Examples

* **TODO prompts**: emphasize recitation + one in-progress rule.
* **File prompts**: enforce orientation + persistence.
* **Research prompts**: encourage broad-to-narrow + stop conditions.
* **Sub-agent prompts**: remind that sub-agents see nothing but the description.

---

## 💻 VI. Code-Level Design Choices

* **Graceful degradation**: Summarization falls back to truncation if structured output fails.
* **Defensive design**:

  * `read_file` truncates long lines.
  * `write_file` overwrites whole file → no partial edits.
* **Performance**: Summarization with `gpt-4o-mini` (cheap, fast).
* **Collision prevention**: UUID suffix for filenames.
* **Extensibility**: Sub-agent factory allows domain-specific agents.

---

## 🛠️ VII. Developer Practices

* **Reproducibility**: `uv sync` ensures consistent environments.
* **Observability**: LangSmith tracing captures 50+ tool-call workflows.
* **Code quality**: `ruff` + `mypy`.
* **Visualization**: `utils.py` provides Rich panels for Human, Assistant, Tool messages.
* **Meta-docs**: `CLAUDE.md` guides both humans and AI coding assistants.

---

## 🎬 VIII. End-to-End Example

**User query:** *“Compare OpenAI vs Anthropic approaches to AI safety.”*

1. **Plan** → TODO list with 3 tasks: research Anthropic, research OpenAI, synthesize.
2. **Delegate** → Two sub-agents spun in parallel.
3. **Isolated research** → Each sub-agent uses Tavily search, saves results in VFS.
4. **Merge** → Files + summaries returned, reducer merges them.
5. **Synthesize** → Parent reads both files, writes comparative answer, marks TODOs completed.

---

## ✅ IX. Best Practices to Apply

### Prompt Hygiene

* Constrain everything (rules, stop conditions, examples).
* Require recitation + reflection.

### Context Engineering

* Offload aggressively into files.
* Isolate via sub-agents.
* Recite plans often.

### Tool Design

* APIs small + predictable.
* Tools enforce structure (status enums, overwrite-only).
* Guardrails (budgets, one-in-progress).

### Workflow Engineering

* Start every query with TODO planning.
* Alternate **Act → Reflect**.
* Summarize for steering, not archival.

### Scalability

* Use sub-agents only when tasks are independent.
* Allow parallelization but cap it.

### Debugging

* Visualize state (todos + files).
* Use LangSmith + Rich formatting.

---

## 🔑 X. Meta-Learnings

Deep agents succeed not by **more tokens** but by **better architecture**:

* **Planning** keeps them goal-driven.
* **Offloading** gives them memory.
* **Isolation** keeps them focused.
* **Reflection** makes them deliberate.
* **Constraints** keep them efficient.

💡 **Deep ≠ infinite** — the key is **discipline, not excess**.

---

# 🎯 Final Takeaway

To build deep agents:
Combine **structured planning (TODOs)**, **external memory (files)**, **context isolation (sub-agents)**, **strategic reflection (think_tool)**, and **tight prompting discipline with hard stop rules**.

This transforms LLMs into **robust, multi-step reasoning systems** capable of 50+ tool calls without drift, collapse, or confusion.
