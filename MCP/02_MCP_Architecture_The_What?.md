# 📘 Lecture Notes: MCP Architecture

## 1. Introduction

* **MCP (Model Context Protocol)**: A framework that defines how AI hosts (chatbots) interact with external servers/tools.
* This lecture covers:

  1. **MCP Architecture**
  2. **MCP Lifecycle** (covered in the next lecture)
  3. **Advanced Concepts** (later lecture)

👉 Today’s focus = **MCP Architecture**.

---

## 2. Simplest Version of MCP Architecture

### Entities

1. **Host**

   * The AI chatbot interface the user interacts with.
   * Could be:

     * Ready-made chatbot (Claude Desktop, Cursor IDE).
     * Custom-built chatbot (e.g., with LangGraph).
   * Host connects to an LLM (OpenAI, Anthropic, Gemini, etc.).

2. **Server**

   * A tool/service capable of executing tasks.
   * Examples:

     * GitHub Server → Manage repositories, commits, issues.
     * Slack Server → Read/write channel messages.
     * Google Drive Server → Manipulate files/folders.

### Example Flow

> User asks: *“Are there any new commits on the GitHub repo?”*

1. User sends query → Host.
2. Host forwards → LLM.
3. LLM realizes it needs external data → asks Host to contact GitHub server.
4. Host queries GitHub server → gets commit list.
5. Host returns data → LLM.
6. LLM generates answer → Host displays to User.

👉 **Key takeaway**: Communication is between **Host** and **Server**.

---

## 3. Refined Architecture: Role of the Client

* In reality, **Host never directly talks to Server**.
* Communication happens through a **Client**.

### Why Client?

* Client speaks **MCP language**, which both server and host can’t natively handle.
* Client:

  * Converts host’s high-level request → MCP request.
  * Sends MCP request → Server.
  * Receives structured MCP response → Converts → host-readable format.

### One-to-One Relationship

* Each **Client ↔ Server** link is **1-to-1**.
* If Host wants to connect to multiple servers:

  * GitHub → Client A
  * Slack → Client B
  * Google Drive → Client C

👉 Host manages **multiple clients**, one per server.

### Analogy

* **Phone = Host**
* **SIM = MCP Client**
* **Network (Airtel/Jio) = Server**
* Multiple networks → multiple SIMs → multiple Clients.

---

## 4. Benefits of MCP Architecture

1. **Decoupling (Separation of Concerns)**

   * GitHub communication independent of Slack/Drive.
   * Failure in one doesn’t affect others.

2. **Scalability**

   * Add more servers by simply adding more clients.
   * Parallel execution possible (GitHub + Slack tasks simultaneously).

---

## 5. MCP Primitives

Primitives = Offerings from a **Server** to a **Host**.
There are 3:

1. **Tools** (Dynamic actions)

   * Executable operations.
   * Examples:

     * GitHub: `create_issue`, `list_commits`, `count_repos`.
     * Google Drive: `search_file`, `create_file`.

   ```json
   {
     "tool": "create_issue",
     "params": {
       "title": "Login button not working",
       "description": "Clicking login does nothing."
     }
   }
   ```

2. **Resources** (Static data)

   * Structured, mostly read-only information.
   * Examples:

     * GitHub repo’s `README.md`.
     * Database schema.

   ```json
   {
     "resource": "repo_readme",
     "url": "https://github.com/user/project/readme.md"
   }
   ```

3. **Prompts** (Guidelines/templates)

   * Predefined instructions to help LLM generate structured outputs.
   * Example: GitHub Issue Template

   ```json
   {
     "prompt": "issue_report",
     "description": "Write detailed GitHub issues",
     "structure": {
       "title": "Bug in Login Button",
       "steps_to_reproduce": ["Open login page", "Enter credentials", "Click login"],
       "expected": "User should be logged in",
       "actual": "Nothing happens",
       "environment": "Chrome, macOS"
     }
   }
   ```

👉 Prompts ensure LLM produces **high-quality, structured responses**.

---

## 6. Standard Operations in MCP

Servers provide **standard methods** to interact with primitives.

### Tools

* `tools/list` → list all tools server supports.
* `tools/call` → execute a specific tool with parameters.

### Resources

* `resources/list` → list available resources.
* `resources/read` → fetch a document.
* `resources/subscribe` → get updates when resource changes.
* `resources/unsubscribe` → stop updates.

### Prompts

* `prompts/list` → get all available prompt templates.
* `prompts/get` → fetch a specific template.

---

## 7. Data Layer

* **Definition**: Language + grammar for Host ↔ Client ↔ Server communication.
* MCP uses **JSON-RPC 2.0** as its foundation.

### JSON-RPC Basics

* Remote Procedure Call protocol using JSON.
* Lightweight, transport-agnostic.

**Request Example**:

```json
{
  "jsonrpc": "2.0",
  "method": "add",
  "params": [2, 3],
  "id": 1
}
```

**Response Example**:

```json
{
  "jsonrpc": "2.0",
  "result": 5,
  "id": 1
}
```

**Error Example**:

```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,
    "message": "Method not found"
  },
  "id": 1
}
```

### Why JSON-RPC (vs REST)?

1. Lightweight (no HTTP headers).
2. Bi-directional communication possible.
3. Transport-agnostic (works with STDIO, HTTP, WebSockets).
4. Supports batching (multiple requests at once).
5. Supports notifications (fire-and-forget messages).

---

## 8. Transport Layer

* **Definition**: Mechanism to move JSON-RPC messages between Client & Server.
* Depends on server type:

  1. **Local Servers** → STDIO (Standard Input/Output)
  2. **Remote Servers** → HTTP + SSE (Server-Sent Events)

### Local Server with STDIO

* Workflow:

  1. Host launches Server as subprocess.
  2. Host writes JSON-RPC request → Server’s STDIN.
  3. Server responds → STDOUT → Host.

**Example (Python subprocess)**:

```python
import subprocess

process = subprocess.Popen(
    ["python3", "server.py"],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    text=True
)

# Send request
process.stdin.write('{"jsonrpc":"2.0","method":"ping","id":1}\n')
process.stdin.flush()

# Read response
response = process.stdout.readline()
print("Response:", response)
```

✅ Advantages:

* Very fast (same machine).
* Secure (no open ports).
* Easy to implement.

---

### Remote Server with HTTP + SSE

* **HTTP POST** used for sending JSON-RPC requests.
* **SSE (Server-Sent Events)** used for streaming incremental responses.

👉 Useful for:

* Long-running tasks.
* Streaming outputs (like AI text generation).

---

## 9. Final MCP Architecture Summary

* **Host** = AI chatbot.
* **Client** = Middleman that converts host requests into MCP language.
* **Server** = External tool/service (GitHub, Slack, Drive).
* **One Client ↔ One Server** relationship.
* **Server offers primitives** = Tools, Resources, Prompts.
* **Communication** = JSON-RPC 2.0.
* **Transport**:

  * Local → STDIO.
  * Remote → HTTP + SSE.

👉 Benefits:

* Decoupling = safer, modular.
* Scalability = multiple clients/servers possible.
* Flexibility = transport-agnostic, future-proof.

---

## 10. Key Interview-Style Questions

1. What are the 3 entities in MCP architecture?
2. Why does MCP need a Client between Host and Server?
3. What are MCP Primitives? Give examples.
4. How does JSON-RPC differ from REST APIs?
5. Explain STDIO vs HTTP+SSE in MCP Transport Layer.

---

✅ With this, you now have a **full picture of MCP Architecture**.
