# 📘 Lecture Notes: MCP Lifecycle

## 1. Introduction

* **MCP (Model Context Protocol)**

  * A protocol that defines how a **host (client)** and a **server** communicate.
  * Used in systems like *Claude Desktop*, *Cursor IDE*, etc.
  * Communication happens over **JSON-RPC 2.0** messages.

* **Why MCP Lifecycle matters?**

  * It explains the **step-by-step sequence** of how a client and server:

    1. Establish a connection
    2. Exchange information during a session
    3. Terminate the connection safely

* **Definition**

  * *MCP Lifecycle describes the complete sequence of steps that govern how a host and a server establish, use, and end a connection during a session.*

* **Session = Continuous connection between client & server**
  Example:

  * Host = Claude Desktop
  * Server = GitHub server
  * Once Claude Desktop starts, it stays connected to GitHub server until the desktop is closed.

---

## 2. Stages of MCP Lifecycle

1. **Initialization Phase** – The handshake (client ↔ server setup)
2. **Operation Phase** – Normal communication (tool/resource usage)
3. **Shutdown Phase** – Connection termination

---

## 3. Stage 1: Initialization Phase

* **Purpose**:

  * First interaction between client & server.
  * Ensure compatibility & exchange capabilities.

* **Two main tasks**:

  1. **Protocol version compatibility check**
  2. **Capabilities negotiation (what each side can do)**

* **Analogy**: Like a handshake agreement before working together.

---

### 🔹 Step 1: Client → Server (Initialize Request)

Client sends `initialize` request with:

* **Protocol version** (e.g. `2023-10-23`)
* **Capabilities** (e.g. `roots`, `sampling`)
* **Implementation info** (client name & version)

**Example JSON-RPC request**:

```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "method": "initialize",
  "params": {
    "protocolVersion": "2023-10-23",
    "capabilities": {
      "roots": {},
      "sampling": {}
    },
    "implementation": {
      "name": "Claude Desktop",
      "version": "1.0.0"
    }
  }
}
```

---

### 🔹 Step 2: Server → Client (Initialize Response)

Server replies with:

* Its **protocol version**
* Its **capabilities** (e.g. `tools`, `resources`)
* **Implementation info** (server name & version)

**Example JSON-RPC response**:

```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "protocolVersion": "2023-09-30",
    "capabilities": {
      "tools": {},
      "resources": {}
    },
    "implementation": {
      "name": "Filesystem Server",
      "version": "2.0.0"
    }
  }
}
```

---

### 🔹 Step 3: Client → Server (Initialized Notification)

* Client confirms initialization is complete.
* This is a **notification** (no `id`, no response expected).

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized",
  "params": {}
}
```

✅ After this, client & server are fully connected.

---

### 🔑 Rules during Initialization

1. **Client cannot send other requests** (except `ping`) before server responds.
2. **Server cannot serve requests** (except `ping`, `logging`) before receiving `initialized` notification.

---

### 🛠️ Special Topics in Initialization

* **Version Negotiation**

  * If protocol versions mismatch, client checks its config for supported versions.
  * If supported → continue.
  * If not → disconnect immediately.

* **Capability Negotiation**

  * Both sides declare what they can do.
  * Examples:

    * Client capabilities: `roots`, `sampling`, `elicitation`
    * Server capabilities: `tools`, `resources`, `prompts`, `logging`
  * Expectations set for the session.

---

## 4. Stage 2: Operation Phase

* **Purpose**: Real work happens here.
* **Rules**:

  1. Respect negotiated **protocol version**.
  2. Use only **declared capabilities**.

---

### Part 1: Capability Discovery

* Client asks server what exactly is available.

**Example requests:**

```json
{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}
{"jsonrpc": "2.0", "id": 2, "method": "resources/list"}
{"jsonrpc": "2.0", "id": 3, "method": "prompts/list"}
```

**Example response (tools):**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {"name": "read_file", "description": "Reads file content"},
      {"name": "write_file", "description": "Writes content to file"},
      {"name": "list_directory", "description": "Lists files in a directory"}
    ]
  }
}
```

---

### Part 2: Tool/Resource Usage

* Client calls specific tools/resources.

**Example: User asks → “Show me contents of hello.py”**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": {"path": "Desktop/hello.py"}
  }
}
```

**Server responds:**

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "content": "print('Hello, MCP!')"
  }
}
```

---

## 5. Stage 3: Shutdown Phase

* **Purpose**: End the session cleanly.

* **Triggered by**:

  * Usually the client (closing the app).
  * Rarely the server.

* **Key point**:

  * **No JSON-RPC messages are exchanged.**
  * Shutdown is handled by **transport layer**.

---

### Local Servers (STDIO transport)

* **Client-initiated shutdown**:

  1. Client closes server’s input stream.
  2. Waits for server to exit.
  3. If server doesn’t exit → send `SIGTERM`.
  4. If still alive → send `SIGKILL`.

* **Server-initiated shutdown (rare)**:

  * Server closes its output stream & exits.

---

### Remote Servers (HTTP transport)

* **Client-initiated**: Close HTTP connection.
* **Server-initiated**: Server drops HTTP connection → client must handle gracefully.

---

## 6. Special Cases in MCP Lifecycle

### (a) Pings

* Lightweight request/response to check liveness.
* Either side can send.

**Ping request:**

```json
{"jsonrpc": "2.0", "id": 10, "method": "ping"}
```

**Response:**

```json
{"jsonrpc": "2.0", "id": 10, "result": {}}
```

---

### (b) Error Handling

* MCP inherits JSON-RPC error structure.

**Error response format:**

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "error": {
    "code": -32601,
    "message": "Method not found"
  }
}
```

**Common Error Codes**:

* `-32601`: Method not found
* `-32602`: Invalid parameters
* `-32600`: Invalid request
* `-32700`: Parse error (invalid JSON)
* `32000+`: Authentication/Rate limit/Internal issues

---

### (c) Timeouts

* Client sets a threshold (e.g. 30s).
* If exceeded → client cancels request.

**Cancellation notification:**

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/cancelled",
  "params": {
    "requestId": 7,
    "reason": "timeout exceeded"
  }
}
```

---

### (d) Progress Notifications

* For **long-running tasks**, server periodically updates client.

**Example progress update:**

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "progressToken": "7",
    "progress": 60,
    "message": "Scanned 600/1000 files"
  }
}
```

---

## 7. Summary

* MCP Lifecycle = **Initialization → Operation → Shutdown**.
* Initialization = version + capability handshake.
* Operation = discovery + tool/resource usage.
* Shutdown = graceful termination by transport layer.
* Special cases: **ping, error handling, timeouts, progress notifications**.

---

✅ With this understanding, you’re ready to build your **own MCP servers & clients** in upcoming coding exercises.
