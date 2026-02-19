<div align="center">
  <img src="https://github.com/bgauryy/octocode-mcp/raw/main/packages/octocode-mcp/assets/logo_white.png" width="400px" alt="Octocode Logo">

  <h1> Octocode Research Skill</h1>

  <p><strong>Turn your agent into a code research powerhouse</strong></p>
  <p>Zero-config MCP tools • Intent-driven workflows • Smart discovery</p>

  [![Skill](https://img.shields.io/badge/skill-agentskills.io-purple)](https://agentskills.io/what-are-skills)
  [![License](https://img.shields.io/badge/license-MIT-blue)](../../LICENSE)
  [![Port](https://img.shields.io/badge/port-1987-green)](http://localhost:1987)

</div>

---

## The Problem

agents struggle with code research because:
- **Too many tools** — Which one do I use? In what order?
- **No context** — Tools work in isolation, losing research continuity
- **Manual orchestration** — You have to chain tools yourself
- **Different APIs** — Local, GitHub, LSP tools all work differently

## The Solution

**Octocode Research** gives your agent:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🎯 Intent Detection  →  📋 Auto-Select Prompt  →  🔧 Smart Tool Chaining  │
│                                                                             │
│     "How does React                  research              GitHub + LSP    │
│      useState work?"         ──────────────────►          tool chain       │
│                                                                             │
│     "Review PR #123"                 reviewPR              Diff analysis   │
│                              ──────────────────►          + code review    │
│                                                                             │
│     "Trace auth flow                 research_local        LSP semantic    │
│      in our app"             ──────────────────►          analysis         │
└─────────────────────────────────────────────────────────────────────────────┘
```



https://github.com/user-attachments/assets/d1260dbc-e7b6-4bec-909f-232ebee91ce9


---

## 🚀 Quick Start

### Installation

```bash
npx add-skill https://github.com/bgauryy/octocode-mcp/tree/main/skills/octocode-research
```

> **Important**: Make sure you are authenticated with GitHub!
> See [Authentication Setup](../../packages/octocode-mcp/docs/AUTHENTICATION_SETUP.md) in the main README.

### Option 2: Manual Setup

```bash
cd skills/octocode-research
npm install && npm start

# Verify it's running (defaults to localhost:1987)
curl http://localhost:1987/health
```

> By default the server listens on `localhost:1987`. You can override this with:
> - `OCTOCODE_RESEARCH_HOST` (default: `localhost`)
> - `OCTOCODE_RESEARCH_PORT` (default: `1987`)

---

## ✨ Key Features

### 1. All MCP Tools Work Out of the Box

No configuration. No token setup. Just ask your question.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         13 TOOLS • ONE INTERFACE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   LOCAL TOOLS              LSP TOOLS              EXTERNAL TOOLS            │
│   ────────────             ─────────              ──────────────            │
│   localSearchCode          lspGotoDefinition      githubSearchCode          │
│   localGetFileContent      lspFindReferences      githubGetFileContent      │
│   localFindFiles           lspCallHierarchy       githubViewRepoStructure   │
│   localViewStructure                              githubSearchRepositories  │
│                                                   githubSearchPullRequests  │
│                                                   packageSearch             │
│                                                                             │
│   ──────────────────────────────────────────────────────────────────────    │
│   All tools: POST /tools/call/:toolName  { "queries": [...] }               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2. Smart Tool & Prompt Discovery

Your agent discovers what's available at runtime:

```bash
# List available tools
GET /tools/list

# Get tool schema BEFORE calling (required!)
GET /tools/info/localSearchCode

# List available prompts
GET /prompts/list

# Get prompt content by intent
GET /prompts/info/research
```

### 3. Intent-Driven Prompt Selection

The skill auto-selects the right workflow based on your question:

| Your Intent | Auto-Selected Prompt | What Happens |
|-------------|---------------------|--------------|
| *"How does React useState work?"* | `research` | GitHub repo exploration, source code analysis |
| *"Trace the auth flow in our app"* | `research_local` | LSP-powered semantic analysis, call hierarchies |
| *"Review PR #123"* | `reviewPR` | Diff analysis, code review, change impact |
| *"Plan adding caching to the API"* | `plan` | Architecture design, implementation steps |

### 4. Production-Ready Resilience

Built-in protection against failures:

```
┌──────────────────────────────────────────────────────────────────┐
│                    RESILIENCE PATTERNS                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CIRCUIT BREAKER          RETRY + BACKOFF         THROTTLING     │
│  ───────────────          ───────────────         ──────────     │
│                                                                  │
│  CLOSED ──► OPEN          Attempt 1 ────►         50 req/min     │
│    │         │            wait 1s                 then gradual   │
│    │    [timeout]         Attempt 2 ────►         slowdown       │
│    │         │            wait 3s                                │
│    ▼         ▼            Attempt 3 ────►                        │
│  HALF-OPEN ◄─┘            success/fail                           │
│                                                                  │
│  Per-service configs:     GitHub: 3 attempts, 30s max            │
│  GitHub: 60s timeout      LSP: 3 attempts, 5s max                │
│  LSP: 10s timeout         Local: 2 attempts, 1s max              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔄 The Research Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          OCTOCODE RESEARCH FLOW                             │
└─────────────────────────────────────────────────────────────────────────────┘

  USER QUESTION                                                    ANSWER
       │                                                              ▲
       ▼                                                              │
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│              │    │              │    │              │    │              │
│  1. INIT     │───►│  2. CONTEXT  │───►│  3. PLAN     │───►│  4. RESEARCH │
│              │    │              │    │              │    │              │
│  Health      │    │  System      │    │  Create      │    │  Execute     │
│  check       │    │  prompt      │    │  research    │    │  tool        │
│  /health     │    │  /tools/     │    │  plan with   │    │  chains      │
│              │    │  system      │    │  tasks       │    │              │
│              │    │              │    │              │    │  Follow      │
│              │    │  Select      │    │              │    │  hints       │
│              │    │  prompt by   │    │              │    │              │
│              │    │  intent      │    │              │    │  Iterate     │
│              │    │  /prompts/   │    │              │    │              │
│              │    │  info/:name  │    │              │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘

```

### Phase 1: INIT
```bash
# Check server is ready
curl http://localhost:1987/health
# → {"status":"ok","port":1987,"uptime":123,...}
```

### Phase 2: CONTEXT
```bash
# Load system prompt (ALWAYS FIRST)
curl http://localhost:1987/tools/system

# Select prompt by user intent
curl http://localhost:1987/prompts/info/research
```

### Phase 3: PLAN
- Analyze user's question
- Create research plan with tasks
- Determine which tools to use

### Phase 4: RESEARCH
```bash
# Execute tools with research context
curl -X POST http://localhost:1987/tools/call/localSearchCode \
  -H "Content-Type: application/json" \
  -d '{
    "queries": [{
      "mainResearchGoal": "Understand authentication flow",
      "researchGoal": "Find auth middleware location",
      "reasoning": "Need to locate entry point before tracing",
      "pattern": "authenticate",
      "path": "/Users/me/project/src"
    }]
  }'
```

---

## 📡 API Reference

### Discovery Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Server health, memory, circuit states |
| `GET` | `/tools/list` | List all tools (concise) |
| `GET` | `/tools/info/:toolName` | **Get tool schema (call BEFORE using!)** |
| `GET` | `/tools/system` | **System prompt (load FIRST!)** |
| `GET` | `/prompts/list` | List all prompts |
| `GET` | `/prompts/info/:promptName` | Get prompt content |

### Execution Endpoint

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/tools/call/:toolName` | **Execute any tool** |

### Request Format

All tool calls use the same structure:

```json
{
  "queries": [{
    "mainResearchGoal": "Overall research objective",
    "researchGoal": "This specific query's goal",
    "reasoning": "Why this approach helps",
    // ... tool-specific parameters
  }]
}
```

### Response Format

```json
{
  "tool": "localSearchCode",
  "success": true,
  "data": { /* tool results */ },
  "hints": [
    "Use lineHint for LSP tools",
    "Consider narrowing search with path filter"
  ],
  "research": {
    "mainResearchGoal": "...",
    "researchGoal": "...",
    "reasoning": "..."
  }
}
```

> **Critical**: Every response includes `hints` — **always read and follow them** for optimal research flow!

---

## 🛠 Available Tools

### LSP Tools (Semantic Analysis)

| Tool | Purpose | Use When |
|------|---------|----------|
| `lspGotoDefinition` | Jump to symbol definition | "Where is X defined?" |
| `lspFindReferences` | Find all usages | "Who uses X?" |
| `lspCallHierarchy` | Trace call flow | "Who calls X? What does X call?" |

### Local Tools (Filesystem)

| Tool | Purpose | Use When |
|------|---------|----------|
| `localSearchCode` | Ripgrep search | "Find pattern X in codebase" |
| `localGetFileContent` | Read file content | "Show me file X" |
| `localFindFiles` | Find by pattern/metadata | "Find all *.ts files" |
| `localViewStructure` | Directory tree | "What's in this folder?" |

### GitHub Tools (External)

| Tool | Purpose | Use When |
|------|---------|----------|
| `githubSearchCode` | Search code in repos | "Find X in React repo" |
| `githubGetFileContent` | Read file from repo | "Show React's useState" |
| `githubViewRepoStructure` | View repo tree | "What's in facebook/react?" |
| `githubSearchRepositories` | Search repos | "Find auth libraries" |
| `githubSearchPullRequests` | Search PRs | "Find PR that added X" |

### Package Tools

| Tool | Purpose | Use When |
|------|---------|----------|
| `packageSearch` | Search npm/PyPI | "Find the lodash package" |

---

## 🎯 Example: How React Implements useState

```bash
# 1. Search for useState in React repo
curl -X POST http://localhost:1987/tools/call/githubSearchCode \
  -H "Content-Type: application/json" \
  -d '{
    "queries": [{
      "mainResearchGoal": "Understand React useState implementation",
      "researchGoal": "Find useState source code location",
      "reasoning": "Need to locate the hook definition first",
      "owner": "facebook",
      "repo": "react",
      "keywordsToSearch": ["useState", "function"],
      "match": "file"
    }]
  }'

# 2. View repository structure
curl -X POST http://localhost:1987/tools/call/githubViewRepoStructure \
  -H "Content-Type: application/json" \
  -d '{
    "queries": [{
      "mainResearchGoal": "Understand React useState implementation",
      "researchGoal": "Find packages directory structure",
      "reasoning": "Hooks likely in react-reconciler or react package",
      "owner": "facebook",
      "repo": "react",
      "branch": "main",
      "path": "packages/react/src",
      "depth": 2
    }]
  }'

# 3. Read the implementation file
curl -X POST http://localhost:1987/tools/call/githubGetFileContent \
  -H "Content-Type: application/json" \
  -d '{
    "queries": [{
      "mainResearchGoal": "Understand React useState implementation",
      "researchGoal": "Read useState hook source code",
      "reasoning": "Found location, now reading implementation",
      "owner": "facebook",
      "repo": "react",
      "path": "packages/react/src/ReactHooks.js",
      "matchString": "useState"
    }]
  }'
```

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          AGENT (Claude, GPT, etc.)                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      OCTOCODE RESEARCH SERVER (:1987)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────────────────────┐  │
│  │   PROMPTS     │   │    TOOLS      │   │      SYSTEM CONTEXT           │  │
│  │               │   │               │   │                               │  │
│  │  research     │   │  GitHub API   │   │  Decision guides              │  │
│  │  research_    │   │  Local FS     │   │  Tool chaining rules          │  │
│  │   local       │   │  LSP Server   │   │  Error recovery               │  │
│  │  reviewPR     │   │  Package APIs │   │  Best practices               │  │
│  │  plan         │   │               │   │                               │  │
│  └───────────────┘   └───────────────┘   └───────────────────────────────┘  │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     RESILIENCE LAYER                                  │  │
│  │  Circuit Breaker  •  Retry + Backoff  •  Throttling  •  Error Queue   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
        ┌──────────┐           ┌──────────┐           ┌──────────┐
        │ Your     │           │ GitHub   │           │ npm/PyPI │
        │ Codebase │           │ Repos    │           │ Registry │
        └──────────┘           └──────────┘           └──────────┘
```

---

## 📊 Bulk Queries

Execute up to 3 queries in parallel:

```bash
curl -X POST http://localhost:1987/tools/call/localSearchCode \
  -H "Content-Type: application/json" \
  -d '{
    "queries": [
      {
        "mainResearchGoal": "Understand state management",
        "researchGoal": "Find useState usages",
        "reasoning": "Compare with useReducer",
        "pattern": "useState",
        "path": "/project/src"
      },
      {
        "mainResearchGoal": "Understand state management",
        "researchGoal": "Find useReducer usages",
        "reasoning": "Compare with useState",
        "pattern": "useReducer",
        "path": "/project/src"
      }
    ]
  }'
```

Response:
```json
{
  "tool": "localSearchCode",
  "bulk": true,
  "success": true,
  "results": [
    { "id": 1, "status": "hasResults", "data": {...} },
    { "id": 2, "status": "hasResults", "data": {...} }
  ],
  "hints": {
    "hasResults": ["Use lineHint for LSP follow-up"],
    "empty": ["Try broader pattern"]
  },
  "counts": { "total": 2, "hasResults": 2, "empty": 0, "error": 0 }
}
```

---

## 🔒 Privacy & Telemetry

We collect **de-identified** telemetry to improve the tool:
- Command usage counts
- Error rates
- **Never** source code, env vars, or PII

Opt-out anytime:
```bash
export LOG=false
```

Local logs stored at `~/.octocode/logs/` for your debugging — **never uploaded**.

See [Privacy Policy](../../PRIVACY.md) and [Terms](../../TERMS.md).

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [SKILL.md](./SKILL.md) | Agent workflow guide |
| [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) | Full architecture details |
| [docs/API_REFERENCE.md](./docs/API_REFERENCE.md) | Complete API reference |
| [docs/FLOWS.md](./docs/FLOWS.md) | Request flow diagrams |
| [../../docs/TROUBLESHOOTING.md](../../docs/TROUBLESHOOTING.md) | Common issues and solutions |

---

## License

MIT License © 2026 Octocode

See [LICENSE](../../LICENSE) for details.
