# Lecture Notes: Generative AI vs. Agentic AI

## 1. Introduction

* Speaker: Nitesh, introducing a new YouTube playlist: **“Agentic AI using LangGraph”**.
* Goal: Teach the evolution from **Generative AI (GenAI)** to **Agentic AI**.
* Strategy:

  * Start with **differences between Generative AI and Agentic AI**.
  * Next video: formal definition of Agentic AI.
* Motivation: Understanding *why* Agentic AI emerged by contrasting it with GenAI.

---

## 2. What is Generative AI?

### Definition

* A **branch of AI** that creates new content in different modalities:

  * Text, images, audio, video, and code.
* Outputs **human-like quality** data.
* Formal definition: *“Generative AI refers to AI models that can create new content (text, images, audio, code, video) resembling human-created data.”*

### History & Rise

* Popularized in the last **3 years**.
* Started with **ChatGPT**, followed by:

  * Other LLMs: Google Gemini, Anthropic Claude, Grok.
  * Image models: DALL·E, MidJourney.
  * Code models: Code LLaMA, Copilot.
  * TTS models: Eleven Labs.
  * Video models: Sora, Runway.

### Comparison with Traditional AI

* **Traditional AI**:

  * Works on *input-output mapping*.
  * Solves **classification** (spam vs non-spam, cancer detection) and **regression** (predicting prices, temperatures).
  * Learns patterns & relationships.
* **Generative AI**:

  * Learns **distribution of data** (the "nature" of data).
  * Can generate **new samples** (e.g., new cat images after learning cat image distribution).
* Key strength: Output looks authentically human-made.

---

## 3. Applications of Generative AI

1. **Creative & Business Writing**

   * Blogs, emails, summaries, drafts.
   * Integrated in tools like Gmail for smart replies.

2. **Software Development**

   * Autocomplete, debugging help, error resolution.
   * Tools like GitHub Copilot.

3. **Customer Support**

   * AI chatbots handling large user queries at scale (e.g., Ola, Uber, Zomato).

4. **Education**

   * Personalized learning paths, summarization, doubt-solving.
   * Students can clarify concepts interactively.

5. **Designing**

   * Graphics, thumbnails, infographics.
   * AI-generated video ads (Runway, Sora).

### Key Insight

* GenAI is **constantly evolving**:
  Early models produced low-quality outputs (e.g., misspelled text in images), but newer models are highly refined.

---

## 4. Practical Scenario: Hiring a Backend Engineer

### HR Recruiter Workflow

1. Draft Job Description (JD).
2. Post on platforms (LinkedIn, Naukri).
3. Shortlist resumes.
4. Conduct interviews.
5. Send offer letter.
6. Onboard candidate.

---

## 5. Generative AI as Solution (Step 1)

* Company provides a **chatbot (LLM-based)**.
* Recruiter uses it to:

  * Draft JD.
  * Get suggestions on posting platforms.
  * Ask for resume screening guidance.
  * Draft interview invite emails.
  * Generate interview questions.
  * Draft offer letters.

**Limitations:**

1. **Reactive** – responds only when prompted.
2. **No memory** – forgets past interactions.
3. **Generic advice** – not company-specific.
4. **No action-taking** – cannot itself post jobs, send emails, etc.

---

## 6. Improvement 1: RAG-based Chatbot

* **Retrieval Augmented Generation (RAG)**:

  * Connect chatbot to **company’s knowledge base**:

    * Past JDs, hiring playbooks, salary bands, interview questions, onboarding documents.
* Benefits:

  * Provides **tailored advice** specific to company culture and history.
  * Example: Auto-includes correct salary range, tech stack, and processes.
* Still limited:

  * Reactive, no memory, cannot take independent actions.

---

## 7. Improvement 2: Tool-Augmented Chatbot

* **Integrates with APIs and tools**:

  * LinkedIn API → post jobs automatically.
  * Resume parser → auto-analyze candidates.
  * Calendar API → schedule interviews.
  * Mail API → send/receive emails.
  * HRMS → automate onboarding.
* Outcome:

  * Chatbot executes tasks, not just advises.
  * Recruiter workload reduced significantly.

**Remaining Problems:**

1. Still reactive (human initiates).
2. Still lacks memory/context awareness.
3. Still not adaptive (doesn’t detect problems unless told).

---

## 8. Final Stage: Agentic AI

* **Agentic AI = Proactive, Context-Aware, Adaptive**.
* Behavior:

  * Understands **goal** (e.g., “Hire backend engineer”).
  * Plans entire process (JD → posting → monitoring → shortlisting → interviewing → offer → onboarding).
  * Executes steps autonomously.
  * Monitors for issues, adapts strategy.
  * Keeps human in the loop for approvals.

### Example:

* Notices job ad has low applicants → autonomously suggests solutions (broaden role, boost ad).
* Screens resumes, schedules interviews, reminds recruiter.
* Drafts and sends offer letters, triggers onboarding automatically.
* Tracks candidate’s acceptance and follow-ups.

---

## 9. Key Differences: Generative AI vs. Agentic AI

1. **Goal**

   * GenAI: Create **content** (text, image, etc.).
   * Agentic AI: Achieve **end goals** through planning + execution.

2. **Nature**

   * GenAI: **Reactive** → requires human guidance each step.
   * Agentic AI: **Proactive & autonomous** → plans and adapts.

3. **Relationship**

   * GenAI is a **capability**.
   * Agentic AI is a **behavior** built on multiple capabilities, including GenAI.

---

## 10. Conclusion

* Journey shown: **Generative AI → RAG Chatbot → Tool-Augmented Chatbot → Agentic AI**.
* Core insight:

  * **GenAI focuses on creation**.
  * **Agentic AI focuses on completion of goals** with autonomy.
* Next lecture: Formal definition and deep dive into **Agentic AI**.

---

✅ **Revision Tip**: Remember the analogy —

* **GenAI = Talent (capability to create content).**
* **Agentic AI = Worker (uses talents + tools + planning to achieve goals).**
