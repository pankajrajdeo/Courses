# Lecture Notes: MCP (Model Context Protocol) – *How to Implement MCP*

---

## 1. Introduction to the Playlist Structure

* **MCP Playlist is divided into 3 parts:**

  1. **Why** MCP? (Motivation, purpose)
  2. **What** is MCP? (Architecture + Lifecycle)
  3. **How** to use MCP in projects (practical hands-on)

* Up to now, we have completed:

  * ✅ Why MCP is needed
  * ✅ What MCP is (Architecture + Lifecycle)

* **Now starting: "How" part**

  * Will be covered in 3 videos:

    1. **Experience MCP with ready-made clients & servers** (this lecture)
    2. Connect Claude Desktop with a **custom MCP server**
    3. Build your **own MCP client + server** and connect them

---

## 2. Key Idea: MCP Communication

* MCP always involves **communication between a Client and a Server**.
* In this lecture:

  * **Client**: Ready-made → *Claude Desktop*
  * **Servers**: Pre-existing → Google Drive, Twitter (X), File System, Manim, Weather API, etc.

---

## 3. Plan of Action for This Lecture

1. Install **Claude Desktop** on local machine.

2. Connect Claude Desktop to **4 servers**:

   * **Local Servers:**

     * File System MCP Server
     * Manim MCP Server
   * **Remote Servers:**

     * Google Drive MCP Server
     * Twitter/X MCP Server
       *(later an actual remote Weather Server also shown)*

3. Understand **Connectors vs JSON Config** for adding servers.

---

## 4. MCP Connectors – Key Concept

### Two ways to connect MCP servers:

1. **JSON Config File (Manual)**

   * Edit Claude’s `config.json` file
   * Add MCP server details manually
   * Example (pseudo):

     ```json
     {
       "mcpServers": {
         "manim": {
           "command": "python3",
           "args": ["/path/to/manim_server.py"],
           "env": {}
         }
       }
     }
     ```

2. **Connectors (Automatic)**

   * Built-in feature in Claude Desktop

   * Works like an **App Store** for MCP servers

   * One-click connection to common SaaS tools like:

     * Google Drive
     * Notion
     * Slack
     * Gmail, etc.

   * **Advantages of Connectors:**

     * Easier (non-technical users can use it)
     * Safer (Anthropic team writes/maintains code)
     * More consistent (less chance of misconfigurations)

   * **Limitations of Connectors:**

     * Only available for popular SaaS tools
     * Every MCP server can’t have a connector (too many servers + open standard principle)

---

## 5. Installing Claude Desktop

* Download from official site:

  * macOS → `.dmg`
  * Windows → `.exe`
* Install & Sign in.
* Interface looks similar to ChatGPT with an added **“Connect Tools” (+)** option.

---

## 6. Server Integrations

### 6.1 Local Server #1 – File System MCP Server

* **Purpose:** Allow Claude to manipulate local directories (read/write files).

* **Method:** Connector (direct integration).

* **Steps:**

  1. Open Claude Desktop → *Add Connectors* → Desktop Extensions → *File System*.
  2. Grant access to specific folders (e.g., Desktop).
  3. Restart Claude Desktop to activate.

* **Example Queries:**

  * **Read:**

    > “Can you list PDF files on my Desktop?”
  * **Write:**

    ```python
    # Fibonacci code example
    def fibonacci(n):
        a, b = 0, 1
        for _ in range(n):
            print(a, end=" ")
            a, b = b, a+b

    fibonacci(10)
    ```

    * Save this code to a `.py` file:

      > “Write the above code into `fibonacci.py` on my Desktop.”

* **Use Cases:**

  * Auto-organize messy downloads folder
  * Summarize a project folder
  * Create/edit code files automatically

---

### 6.2 Local Server #2 – Manim MCP Server

* **Purpose:** Generate mathematical visualizations (like *3Blue1Brown* videos).

* **Setup:** No connector available → must use JSON config.

* **Steps:**

  1. Install dependencies:

     ```bash
     pip install manim
     pip install mcp
     ```
  2. Clone Manim MCP repo:

     ```bash
     git clone https://github.com/.../manim-mcp-server.git
     ```
  3. Edit Claude’s config JSON:

     ```json
     {
       "mcpServers": {
         "manim": {
           "command": "/usr/bin/python3",
           "args": ["/Users/username/Desktop/manim-mcp-server/src/server.py"]
         }
       }
     }
     ```
  4. Restart Claude Desktop.

* **Prompt Example:**

  > "Use the Manim server to create an animation showing vector transformation in linear algebra."

  * Output: Video with 2D grid, basis vectors `i` and `j`, matrix transformation applied, new vectors shown.

* **Notes:**

  * Requires **LaTeX installed** for proper math symbols.
  * Without LaTeX, fallback rendering may look different.

---

### 6.3 Remote Server #1 – Google Drive MCP Server

* **Purpose:** Access Google Drive documents.

* **Method:** Connector available.

* **Steps:**

  1. Open Claude Desktop → Add Connector → Google Drive.
  2. Sign in with Google account.
  3. Restart Claude Desktop.

* **Capabilities:**

  * ✅ Read documents
  * ❌ Cannot write/create files

* **Example:**

  > “Summarize the document titled ‘AI Newsletter Content Ideas’ from my Google Drive.”

---

### 6.4 Remote Server #2 – Twitter (X) MCP Server

* **Purpose:** Read & post tweets.

* **Setup:** JSON Config + Twitter Developer Keys.

* **Steps:**

  1. Create Developer Account on Twitter.
  2. Generate API keys:

     * API Key + Secret
     * Access Token + Secret
  3. Edit config:

     ```json
     {
       "mcpServers": {
         "twitter": {
           "command": "npx",
           "args": ["twitter-mcp-server"],
           "env": {
             "API_KEY": "...",
             "API_SECRET": "...",
             "ACCESS_TOKEN": "...",
             "ACCESS_SECRET": "..."
           }
         }
       }
     }
     ```

* **Tools Provided:**

  * `search_tweets`
  * `post_tweet`

* **Examples:**

  * Search:

    > “What are the top tweets about AI this week?”
  * Post:

    > “Post a tweet saying: ‘Hello from CampusX!’”

  *(May require enabling read + write permissions in developer account.)*

---

### 6.5 Remote Server #3 – Weather MCP Server

* **Purpose:** Fetch real-time weather info.

* **Setup:** JSON Config with AccuWeather API key.

* **Steps:**

  1. Install dependency:

     ```bash
     pip install uv
     ```
  2. Get free AccuWeather API key.
  3. Add to config:

     ```json
     {
       "mcpServers": {
         "weather": {
           "command": "/path/to/uvx",
           "args": ["accuweather-mcp-server"],
           "env": {
             "ACCUWEATHER_API_KEY": "your_api_key_here"
           }
         }
       }
     }
     ```
  4. Restart Claude Desktop.

* **Example:**

  > “Tell me the current weather in Gurgaon.”

* **Note:** Sometimes fails due to API quota/technical issues.

---

## 7. Discovery of New MCP Servers

* Best resource: **Awesome MCP Servers (GitHub)**

  * Maintains updated list of servers by categories
  * Example: Found Manim MCP Server here
* Other marketplaces are emerging (like app stores for MCP).

---

## 8. Summary of Key Learnings

* **Two connection methods:**

  * Connectors → Easy, safe, for popular tools
  * JSON Config → Flexible, for custom/local servers
* **In this lecture we saw 5 MCP servers:**

  * File System (local, via Connector)
  * Manim (local, via JSON Config)
  * Google Drive (remote, via Connector)
  * Twitter/X (local installed via NPM, via JSON Config)
  * Weather API (remote, via JSON Config)
* **Best Practices:**

  * Restart Claude Desktop after adding a server
  * Grant minimal permissions (safety)
  * Explore new servers via GitHub/marketplaces

---

## 9. Code & Config Reference Snippets

### Install Dependencies

```bash
pip install manim mcp uv
npm install -g twitter-mcp-server
```

### Example Claude Config JSON (combined)

```json
{
  "mcpServers": {
    "manim": {
      "command": "/usr/bin/python3",
      "args": ["/Users/nitish/Desktop/manim-mcp-server/src/server.py"]
    },
    "twitter": {
      "command": "npx",
      "args": ["twitter-mcp-server"],
      "env": {
        "API_KEY": "xxxx",
        "API_SECRET": "xxxx",
        "ACCESS_TOKEN": "xxxx",
        "ACCESS_SECRET": "xxxx"
      }
    },
    "weather": {
      "command": "/usr/local/bin/uvx",
      "args": ["accuweather-mcp-server"],
      "env": {
        "ACCUWEATHER_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

---

✅ By the end of this lecture, you should be able to:

* Install Claude Desktop
* Add servers via **both methods** (Connectors & JSON Config)
* Use multiple local and remote MCP servers
* Discover and experiment with new MCP integrations
