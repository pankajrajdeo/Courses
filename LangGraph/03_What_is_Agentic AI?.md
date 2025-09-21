# Lecture Notes: Introduction to Agentic AI

## 1. Introduction

* **Instructor**: Nitesh
* **Context**: Second video in the playlist *Agentic AI using LangGraph*.
* **Goal of the lecture**:

  * Provide a **formal study** of Agentic AI.
  * Cover:

    1. What is Agentic AI?
    2. Characteristics of Agentic AI.
    3. Components of Agentic AI.

---

## 2. What is Agentic AI?

* **Definition**:
  Agentic AI is a type of AI that can take a task or goal from a user and work toward completing it **with minimal human guidance**.

  * It plans, takes actions, adapts to changes, and seeks help only when necessary.

* **Key Idea**:
  Unlike *Generative AI* (reactive, only responds when asked), *Agentic AI* is **autonomous and proactive**.

* **Example (Travel to Goa)**:

  * *Generative AI*: User must ask step by step (transport, hotels, places to visit, etc.). AI just reacts with answers.
  * *Agentic AI*: User states the goal (“I want to go to Goa between these dates”). The system autonomously:

    * Finds best transport,
    * Suggests hotels,
    * Creates itinerary,
    * Plans meals and activities.

---

## 3. Real-World Example: AI Recruiter

**Scenario**: Recruiter needs to hire a backend engineer using an Agentic AI chatbot.
Steps the Agent performs autonomously:

1. **Understand the goal**: Hire a backend engineer with 2–4 years’ experience, remote.
2. **Plan**: Draft JD → Post on platforms → Monitor applications → Screen candidates → Schedule interviews → Draft offer → Manage onboarding.
3. **Execution**:

   * Draft JD from company docs.
   * Post JD using LinkedIn/Naukri APIs.
   * Monitor applications and adapt (e.g., tweak JD, run ads).
   * Screen resumes using parsing tools.
   * Schedule interviews by checking recruiter’s calendar.
   * Draft & send offer letter.
   * Manage acceptance, onboarding, IT setup.

**Key Insight**:
The system is **autonomous, proactive, adaptive, and goal-driven**, with minimal human involvement except checkpoints/permissions.

---

## 4. Characteristics of Agentic AI

Agentic AI systems share six defining traits:

### 4.1 Autonomy

* Ability to make decisions and act independently without step-by-step instructions.
* **Types of autonomy**:

  * Execution autonomy (carry out plans).
  * Decision autonomy (e.g., shortlist candidates).
  * Tool autonomy (deciding which tools to use).
* **Control mechanisms**:

  * Limit permissions (e.g., can screen candidates but not reject).
  * Human-in-the-loop (e.g., approval required before posting JD).
  * Override controls (pause/stop/change behavior anytime).
  * Guardrails/policies (e.g., no weekend interviews, no informal email tone).
* **Risks of uncontrolled autonomy**: Wrong offers, bias in shortlisting, overspending on ads.

### 4.2 Goal-Oriented

* Operates with a persistent objective.
* All actions/planning tied to achieving the goal.
* Goals can be:

  * Independent (e.g., hire a backend engineer).
  * Constrained (e.g., remote-only, within budget, specific country).
* Stored in memory (structured, often JSON-like format).
* **Flexibility**: Goals can change mid-process (e.g., switch from hiring full-time to freelancer).

### 4.3 Planning

* Central to Agentic AI’s operation.
* **Two-step cycle**:

  1. Planning: Break high-level goal into structured subgoals/actions.
  2. Execution: Carry out the plan.
* **Iterative**: If execution fails, return to planning.
* **Three stages of planning**:

  1. Generate multiple candidate plans.
  2. Evaluate (efficiency, tool availability, cost, risk, constraint alignment).
  3. Select best plan (via policy or human input).

### 4.4 Reasoning

* Cognitive process for interpreting input, drawing conclusions, and making decisions.
* Needed in both planning and execution:

  * **Planning stage**: Goal decomposition, tool selection, resource estimation.
  * **Execution stage**: Decision-making, error handling, human-in-the-loop choice.
* Example: If LinkedIn API is down → Reason to retry, notify human, or use alternative platform.

### 4.5 Adaptability

* Ability to modify strategies when unexpected situations arise.
* Triggers for adaptation:

  * Tool failures (API down).
  * External feedback (low applications).
  * Goal changes (switch from full-time hire to freelancer).

### 4.6 Context Awareness

* Retains and uses context across time to function coherently.
* **Types of context maintained**:

  * Original goal.
  * Progress so far.
  * Human-agent interaction history.
  * Environmental state (e.g., job posting stats).
  * Tool responses (e.g., resume parser results).
  * User preferences (e.g., remote candidates preferred).
  * Policies/guardrails (e.g., no ads without approval).
* **Implemented via memory**:

  * Short-term memory: Session-specific info (tool responses, current tasks).
  * Long-term memory: High-level goals, past interactions, policies, user preferences.

---

## 5. Components of Agentic AI Systems

Five key high-level components:

1. **Brain** (LLM in LLM-based agents):

   * Interprets goals.
   * Handles planning, reasoning, tool selection.
   * Manages communication in natural language.

2. **Orchestrator**:

   * Executes plans step by step.
   * Handles sequencing, conditional routing, retries, loops, delegation.
   * Comparable to a project manager or nervous system.

3. **Tools**:

   * Interfaces to interact with external world (APIs, databases, email, job platforms, RAG knowledge bases).
   * Represent the agent’s “hands and legs.”

4. **Memory**:

   * Short-term: Session-level actions, tool calls, immediate decisions.
   * Long-term: Goals, preferences, policies, cross-session history.
   * Enables context awareness and continuity.

5. **Supervisor**:

   * Enforces human-in-the-loop.
   * Handles approvals, guardrails, escalations, exceptions.
   * Ensures human control in risky/high-stakes actions.

---

## 6. Conclusion

* **We studied**:

  1. What Agentic AI is.
  2. Real-world example (AI recruiter).
  3. Six key characteristics (Autonomy, Goal-oriented, Planning, Reasoning, Adaptability, Context-awareness).
  4. Five main components (Brain, Orchestrator, Tools, Memory, Supervisor).

* **Takeaway**:
  Agentic AI is **autonomous, proactive, adaptive, and context-aware**, enabling it to execute complex, multi-step goals with minimal human guidance.
