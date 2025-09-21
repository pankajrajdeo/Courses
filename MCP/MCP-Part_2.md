# MCP (Model Context Protocol) – Architecture

## 1. Introduction to the "What" Aspect

* This section focuses on **what MCP is**, specifically its **architecture**.
* The full “What” series is divided into three parts:

  1. **Architecture** (covered here).
  2. **Lifecycle** (how client & server interact step-by-step).
  3. **Advanced concepts**.

This part covers the **architecture of MCP from first principles**.

---

## 2. Simplest Version of MCP Architecture

At the most basic level, there are **two entities**:

1. **Host**

   * The **AI chatbot** that interacts with the user.
   * Can be a ready-made chatbot (Claude desktop, Cursor IDE) or custom-built.
   * Behind the scenes, connected to an **LLM** (OpenAI, Anthropic, Gemini, etc.).

2. **Server**

   * Represents a **tool or service** that performs specific tasks.
   * Examples:

     * GitHub server → manage repositories.
     * Slack server → read/write messages.
     * Google Drive server → manipulate files.

👉 All communication flows **between host and server**.

**Example**:

* User asks: *“Are there any new commits on my GitHub repo?”*

  1. Host passes this to the LLM.
  2. LLM realizes it needs external data → GitHub server.
  3. Host queries GitHub server.
  4. GitHub server checks repo and returns commits.
  5. LLM forms the final answer → Host shows it to user.

---

## 3. Adding Refinement: The Role of the Client

* In reality, **hosts never directly talk to servers**.
* They need a helper called the **MCP Client**.

### MCP Client

* Acts as the **bridge** between host and server.
* Speaks the same MCP “language” as the server.
* Responsibilities:

  * Converts **high-level requests** from host into **MCP-compatible requests**.
  * Sends requests to server.
  * Translates **structured MCP responses** into a format host can understand.

👉 **No direct host ↔ server communication. All passes via the client.**

---

## 4. One-to-One Relationship (Client ↔ Server)

* MCP defines a **strict one-to-one relationship**:

  * **One client can only connect to one server at a time.**
* If host needs multiple servers (GitHub, Slack, Drive):

  * It must use **separate clients**, one per server.

**Analogy**:

* Phone = Host
* SIM card = Client
* Airtel/Jio/Vodafone = Servers
* To connect to multiple networks, you need **multiple SIMs**.

---

## 5. Benefits of this Architecture

1. **Decoupling / Separation of Concerns**

   * Each server communication is isolated.
   * If GitHub line breaks, Slack/Drive still work.
   * Safer and more resilient.

2. **Scalability**

   * Add as many servers as needed.
   * Just install a new client per server.
   * Enables **parallel execution** (GitHub + Slack simultaneously).

---

## 6. Primitives: What Servers Offer to Hosts

**Primitives = offerings that servers provide to hosts.**
There are **three types**:

1. **Tools**

   * Actions that the host can ask the server to perform.
   * Examples:

     * GitHub: list commits, create issue, push code.
     * Drive: search files, create new file.

2. **Resources**

   * Static data sources (documents, schemas).
   * Examples:

     * GitHub: fetch README file.
     * Database: fetch schema.

3. **Prompts**

   * Predefined prompt templates + instructions that **guide the AI’s behavior**.
   * Example: *Issue Report Prompt* that enforces a clear format: title, steps to reproduce, expected vs. actual behavior, environment.
   * Helps LLM produce **better structured requests** to the server.

---

## 7. Standard Operations for Each Primitive

MCP defines **standard functions** to interact with primitives:

1. **Tools**

   * `tools/list` → Get available tools.
   * `tools/call` → Execute a specific tool with arguments.

2. **Resources**

   * `resources/list` → List available resources.
   * `resources/read` → Read a specific resource.
   * `resources/subscribe` / `unsubscribe` → Get notified on changes.

3. **Prompts**

   * `prompts/list` → List available prompt templates.
   * `prompts/get` → Fetch details of a specific prompt.

---

## 8. Data Layer

* Defines the **language & grammar** for MCP communication.
* Built on **JSON-RPC 2.0**.

### JSON-RPC 2.0

* Combines **Remote Procedure Call (RPC)** with **JSON**.
* Allows executing functions on remote machines “as if local”.
* Requests/responses structured in JSON.

**Example JSON-RPC Request**:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {}
}
```

**Response**:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": ["listIssues", "listPulls"]
}
```

### Why JSON-RPC over REST?

1. Lightweight (no extra HTTP headers).
2. Supports **bi-directional communication** (server → client requests).
3. Transport agnostic (works with stdio, HTTP, WebSockets, etc.).
4. Supports **batching** (multiple requests at once).
5. Supports **notifications** (fire-and-forget, no response needed).

---

## 9. Transport Layer

* Defines **how JSON-RPC messages travel** between client and server.

### Two Types of Servers

1. **Local Servers**

   * Run on the same machine as the host.
   * Use **STDIO (Standard Input/Output)** for transport.
   * Benefits: **fast, secure, simple**.

2. **Remote Servers**

   * Run on other machines (cloud, network).
   * Use **HTTP + SSE (Server-Sent Events)** for transport.
   * HTTP POST → send JSON-RPC requests.
   * SSE → stream results chunk-by-chunk (better for long tasks).

### Key Point

* **Data layer (JSON-RPC) is transport agnostic**.
* Same JSON-RPC message can travel over STDIO or HTTP.
* This flexibility is why Anthropic chose JSON-RPC.

---

## 10. Final Summary of MCP Architecture

* **Host** = AI chatbot.
* **Client** = Bridge between host and server (1:1 relationship).
* **Server** = External tool/service.
* **Server Primitives** = Tools, Resources, Prompts.
* **Standard Operations** = `/list`, `/call`, `/read`, etc.
* **Data Layer** = JSON-RPC 2.0 (request/response grammar).
* **Transport Layer** =

  * Local servers → STDIO.
  * Remote servers → HTTP + SSE.

**Benefits**:

* Decoupled, safe, scalable, parallel.
* Transport agnostic.
* Moves us closer to a **unified AI work partner**.

Would you like me to also **turn these notes into a diagram (visual architecture map)** so you can see host, client, server, primitives, data & transport layers all in one picture?
