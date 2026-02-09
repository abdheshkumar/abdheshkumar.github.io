# Claude Code Explained - A Beginner's Guide

This guide explains Claude Code features from the ground up. No prior knowledge needed!

## What is Claude Code?

Claude Code is an AI coding assistant that lives in your terminal. Think of it like having an expert programmer sitting next to you, but instead of typing messages in a chat, it can:
- Read and write files in your project
- Search your codebase
- Run terminal commands
- Browse the web for information
- And much more!

```mermaid
graph LR
    YOU[👤 You] -->|Ask questions<br/>Request code changes| CC[🤖 Claude Code]
    CC -->|Reads files<br/>Writes code<br/>Runs commands| PROJECT[📁 Your Project]
    CC -->|Responds| YOU

    style YOU fill:#e1f5ff
    style CC fill:#c8e6c9
    style PROJECT fill:#fff9c4
```

## The Big Picture: How to Extend Claude Code

Out of the box, Claude Code can read files, write code, search, and run commands. But you can **extend** it with additional capabilities:

```mermaid
graph TB
    CC[🤖 Claude Code<br/>Core Capabilities]

    subgraph EXTENSIONS[" "]
        CM[📄 CLAUDE.md<br/>Project Memory]
        SK[📚 Skills<br/>Reusable Knowledge]
        MCP[🔌 MCP<br/>External Services]
        SA[🤖 Subagents<br/>Helper Workers]
        HK[⚡ Hooks<br/>Automation Scripts]
    end

    CC -.->|You can add| EXTENSIONS

    style CC fill:#c8e6c9
    style CM fill:#bbdefb
    style SK fill:#ce93d8
    style MCP fill:#ffab91
    style SA fill:#80cbc4
    style HK fill:#fff59d
```

Let's understand each extension one by one.

---

## 1. CLAUDE.md - Project Memory

### What is it?
A special file in your project that Claude reads **every single time** you start a conversation.

### Real-World Analogy
Think of it like a sticky note on your monitor that says:
- "Always use spaces, not tabs"
- "Our database is PostgreSQL"
- "Run `npm test` before committing"

### How it Works

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant CLAUDE.md
    participant Project

    You->>Claude: Start Claude Code
    Claude->>CLAUDE.md: Read instructions
    CLAUDE.md->>Claude: "Always use pnpm, not npm"
    Note over Claude: Now Claude remembers<br/>this for entire session

    You->>Claude: "Install a package"
    Claude->>Project: Runs "pnpm install lodash"
    Note over Claude: Used pnpm because<br/>CLAUDE.md told it to
```

### Example

**File: `/your-project/CLAUDE.md`**
```markdown
# Project Guidelines

## Package Manager
Always use `pnpm` instead of npm.

## Testing
Run `npm test` before committing any changes.

## Code Style
- Use TypeScript for all new files
- Follow ESLint rules
- Maximum line length: 100 characters
```

### When to Use
- Project-wide rules that **always** apply
- Build commands
- "Never do X" rules
- Code conventions

### When NOT to Use
- Large reference documentation (use Skills instead)
- Things that only matter sometimes
- Keep it under ~500 lines

---

## 2. Skills - Reusable Knowledge & Workflows

### What is it?
A package of instructions, knowledge, or a workflow that Claude can use when needed.

### Real-World Analogy
Like a recipe book. You don't need to read every recipe when cooking dinner - you only open the book when you need a specific recipe. Similarly, Skills load only when needed.

### Two Types of Skills

```mermaid
graph TB
    subgraph SKILL_TYPES[" "]
        REF[📖 Reference Skills<br/>Knowledge/Documentation]
        ACT[🎬 Action Skills<br/>Workflows you trigger]
    end

    REF -->|Example| R1["API Documentation<br/>Design Patterns<br/>Best Practices"]
    ACT -->|Example| A1["/deploy - Deploy app<br/>/review - Code review<br/>/test - Run tests"]

    style REF fill:#bbdefb
    style ACT fill:#ce93d8
```

### How Skills Work

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant Skill
    participant Project

    Note over Claude: At startup, Claude sees<br/>skill names & descriptions

    You->>Claude: "Deploy my app"
    Claude->>Skill: Loads /deploy skill
    Skill->>Claude: Instructions: "1. Run tests<br/>2. Build production<br/>3. Push to server"
    Claude->>Project: Executes deployment steps
    Claude->>You: "Deployed successfully!"
```

### Example: Action Skill

**File: `/skills/deploy/SKILL.md`**
```markdown
---
name: deploy
description: Deploy the application to production
---

# Deployment Workflow

When the user asks to deploy:

1. Run `npm test` to ensure tests pass
2. Run `npm run build` to create production build
3. Run `./deploy.sh` to push to server
4. Verify deployment at https://myapp.com
```

**Usage:**
```bash
You: /deploy
Claude: Running tests... ✓
        Building... ✓
        Deploying... ✓
        Verified at https://myapp.com
```

### Example: Reference Skill

**File: `/skills/api-guide/SKILL.md`**
```markdown
---
name: api-guide
description: API design patterns and conventions for our services
---

# API Guidelines

## Endpoint Naming
- Use plural nouns: `/users` not `/user`
- Use kebab-case: `/user-profiles` not `/userProfiles`

## Response Format
Always return JSON:
{
  "data": {...},
  "error": null,
  "timestamp": "2024-01-01T00:00:00Z"
}
```

**Usage:** Claude loads this automatically when you ask about APIs.

### When to Use
- Repeatable workflows you trigger with `/command-name`
- Reference documentation (API docs, style guides)
- Domain-specific knowledge
- Things that don't need to be in every conversation

---

## 3. Subagents - Helper Workers

### What is it?
A **separate copy of Claude** that works on a specific task in isolation, then reports back.

### Real-World Analogy
Like asking a coworker to research something. They go off, do the research, and come back with a summary. They don't need to know about your entire day - just their specific task.

### Why Use Subagents?

**Problem:** If Claude reads 50 files to research something, your conversation gets cluttered.

**Solution:** Send a subagent to do the research. The subagent reads all 50 files in isolation, then returns just the summary.

### How Subagents Work

```mermaid
graph TB
    YOU[👤 You]
    MAIN[🤖 Main Claude<br/>Your conversation]

    subgraph ISOLATION[Isolated Context]
        SUB1[🤖 Subagent 1<br/>Researching X]
        SUB2[🤖 Subagent 2<br/>Reviewing code]
    end

    YOU <--> MAIN
    MAIN -->|Spawns| SUB1
    MAIN -->|Spawns| SUB2
    SUB1 -.->|Summary only| MAIN
    SUB2 -.->|Summary only| MAIN

    style YOU fill:#e1f5ff
    style MAIN fill:#c8e6c9
    style SUB1 fill:#80cbc4
    style SUB2 fill:#80cbc4
```

### Example Flow

```mermaid
sequenceDiagram
    participant You
    participant Main Claude
    participant Subagent
    participant Files[50 Files]

    You->>Main Claude: "How does authentication work?"
    Main Claude->>Subagent: "Research authentication.<br/>Read all related files."

    Subagent->>Files: Reads 50 authentication files
    Note over Subagent: Analyzes patterns,<br/>compiles findings

    Subagent->>Main Claude: "Summary: Uses JWT tokens,<br/>OAuth2 flow, stored in Redis"

    Main Claude->>You: "Your auth uses JWT tokens with OAuth2..."

    Note over Main Claude: Your conversation only has<br/>the summary, not 50 files!
```

### Built-in Subagent Types

```mermaid
graph LR
    subgraph SUBAGENTS[Available Subagents]
        EXP[🔍 Explore<br/>Search codebase]
        PLN[📋 Plan<br/>Design approach]
        BSH[⚙️ Bash<br/>Git operations]
        REV[✅ Code Review<br/>Quality checks]
        GEN[🎯 General<br/>Multi-step tasks]
    end

    style EXP fill:#80cbc4
    style PLN fill:#80cbc4
    style BSH fill:#80cbc4
    style REV fill:#80cbc4
    style GEN fill:#80cbc4
```

### When to Use
- Tasks that need to read many files
- Research questions
- Parallel work (run multiple subagents at once)
- Keep your main conversation clean

---

## 4. MCP (Model Context Protocol) - External Services

### What is it?
A way to connect Claude to external services like databases, Slack, Jira, etc.

### Real-World Analogy
Like giving Claude a phone with apps installed. Without MCP, Claude can't call anyone or use any apps. With MCP, Claude can "call" your database, "message" Slack, or "open" Jira.

### How MCP Works

```mermaid
graph TB
    CLAUDE[🤖 Claude Code]

    subgraph MCP_SERVERS[MCP Servers]
        DB[🗄️ Database<br/>MCP Server]
        SLACK[💬 Slack<br/>MCP Server]
        JIRA[📋 Jira<br/>MCP Server]
        BROWSER[🌐 Browser<br/>MCP Server]
    end

    CLAUDE <-->|Query data| DB
    CLAUDE <-->|Send messages| SLACK
    CLAUDE <-->|Create tickets| JIRA
    CLAUDE <-->|Control browser| BROWSER

    style CLAUDE fill:#c8e6c9
    style DB fill:#ffab91
    style SLACK fill:#ffab91
    style JIRA fill:#ffab91
    style BROWSER fill:#ffab91
```

### Example: Database Connection

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant MCP Server
    participant Database

    You->>Claude: "How many users signed up today?"
    Claude->>MCP Server: query_database("SELECT COUNT(*) FROM users...")
    MCP Server->>Database: Execute SQL query
    Database->>MCP Server: 42 users
    MCP Server->>Claude: 42
    Claude->>You: "42 users signed up today"
```

### Common MCP Servers
- **Database**: Query PostgreSQL, MySQL, etc.
- **Slack**: Send messages, read channels
- **GitHub**: Create issues, PRs
- **Jira**: Create/update tickets
- **Browser**: Control Chrome for testing
- **File System**: Extended file operations

### When to Use
- Need to access external data (databases, APIs)
- Need to perform external actions (post to Slack, create tickets)
- Integrate with your existing tools

---

## 5. Hooks - Automation Scripts

### What is it?
Scripts that run automatically when specific events happen.

### Real-World Analogy
Like "If This Then That" (IFTTT).
- **IF** Claude edits a file, **THEN** run ESLint
- **IF** conversation ends, **THEN** save a summary

### How Hooks Work

```mermaid
graph LR
    EVENT[📅 Event Happens]
    HOOK[⚡ Hook Script Runs]
    ACTION[🎯 Action Executes]

    EVENT -->|Triggers| HOOK
    HOOK -->|Performs| ACTION

    E1[File edited] -.-> EVENT
    E2[Prompt submitted] -.-> EVENT
    E3[Session starts] -.-> EVENT

    A1[Run linter] -.-> ACTION
    A2[Post to Slack] -.-> ACTION
    A3[Log to file] -.-> ACTION

    style EVENT fill:#fff59d
    style HOOK fill:#fff59d
    style ACTION fill:#fff59d
```

### Example Hook

**Scenario:** Automatically run ESLint whenever Claude edits a JavaScript file.

**File: `.claude/hooks/post-edit.sh`**
```bash
#!/bin/bash
# Runs after every file edit

if [[ "$FILE" == *.js ]]; then
  echo "Running ESLint on $FILE..."
  eslint "$FILE" --fix
fi
```

**What Happens:**
```mermaid
sequenceDiagram
    participant Claude
    participant File
    participant Hook
    participant ESLint

    Claude->>File: Edits app.js
    Note over File: File saved
    File->>Hook: Triggers post-edit hook
    Hook->>ESLint: eslint app.js --fix
    ESLint->>File: Applies fixes
    Note over Claude: Claude sees ESLint output
```

### Available Hook Events
- **post-edit**: After file edits
- **pre-bash**: Before running commands
- **post-bash**: After commands finish
- **session-start**: When Claude Code starts
- **session-end**: When you exit
- **prompt-submit**: When you send a message

### When to Use
- Automated checks (linting, formatting)
- Logging and monitoring
- Notifications (post to Slack when changes happen)
- Predictable automation that doesn't need AI

---

## 6. Plugins - Packaging It All Together

### What is it?
A bundle that packages skills, hooks, MCP servers, and configurations into one installable unit.

### Real-World Analogy
Like a browser extension. Instead of manually installing 5 different things, you install one plugin that includes everything.

### What Plugins Can Include

```mermaid
graph TB
    PLUGIN[📦 Plugin]

    PLUGIN --> SKILLS[📚 Multiple Skills]
    PLUGIN --> HOOKS[⚡ Hooks]
    PLUGIN --> MCP[🔌 MCP Servers]
    PLUGIN --> CONFIG[⚙️ Configuration]

    SKILLS --> S1["/deploy skill"]
    SKILLS --> S2["/review skill"]
    SKILLS --> S3["/test skill"]

    style PLUGIN fill:#ce93d8
    style SKILLS fill:#bbdefb
    style HOOKS fill:#fff59d
    style MCP fill:#ffab91
    style CONFIG fill:#c8e6c9
```

### When to Use
- Share capabilities across multiple projects
- Distribute to your team
- Package related features together

---

## How Features Work Together

Features often combine to create powerful workflows. Here are common patterns:

### Pattern 1: CLAUDE.md + Skills

```mermaid
graph LR
    CM[📄 CLAUDE.md<br/>Always follow API rules]
    SK[📚 Skill<br/>Full API documentation]
    CLAUDE[🤖 Claude]

    CM -->|Always loaded| CLAUDE
    SK -->|Loaded when needed| CLAUDE

    style CM fill:#bbdefb
    style SK fill:#ce93d8
    style CLAUDE fill:#c8e6c9
```

**Use Case:**
- CLAUDE.md: "Follow our API conventions"
- Skill: Contains 50-page API style guide
- Result: Claude always knows to follow API rules, loads full guide when building APIs

### Pattern 2: Skill + Subagent

```mermaid
sequenceDiagram
    participant You
    participant Main Claude
    participant Review Skill
    participant Subagent 1
    participant Subagent 2
    participant Subagent 3

    You->>Main Claude: /review
    Main Claude->>Review Skill: Load review skill
    Review Skill->>Main Claude: "Spawn 3 subagents"

    Main Claude->>Subagent 1: Check security
    Main Claude->>Subagent 2: Check performance
    Main Claude->>Subagent 3: Check tests

    Subagent 1->>Main Claude: Security OK ✓
    Subagent 2->>Main Claude: Found perf issue ⚠️
    Subagent 3->>Main Claude: Tests passing ✓

    Main Claude->>You: Review complete (1 issue found)
```

**Use Case:**
- `/review` skill triggers parallel code reviews
- Each subagent checks different aspect
- Results merge in main conversation

### Pattern 3: MCP + Skill

```mermaid
graph TB
    MCP[🔌 MCP Server<br/>Database connection]
    SKILL[📚 Skill<br/>Database schema docs]
    CLAUDE[🤖 Claude]

    MCP -->|Provides tools| CLAUDE
    SKILL -->|Provides knowledge| CLAUDE

    CLAUDE -->|Knows how to query<br/>from skill| Q[Smart Queries]
    CLAUDE -->|Can execute queries<br/>from MCP| Q

    style MCP fill:#ffab91
    style SKILL fill:#ce93d8
    style CLAUDE fill:#c8e6c9
    style Q fill:#c8e6c9
```

**Use Case:**
- MCP: Gives Claude ability to query database
- Skill: Teaches Claude your database schema
- Result: Claude can write smart queries using correct tables

---

## Decision Tree: Which Feature Should I Use?

```mermaid
graph TD
    START{What do you<br/>need?}

    START -->|Always-on rules| Q1{Does it apply<br/>to every session?}
    START -->|Reference docs| Q2{Large or small?}
    START -->|Workflow| Q3{Who triggers it?}
    START -->|External data| MCP[🔌 Use MCP]
    START -->|Automation| HOOKS[⚡ Use Hooks]
    START -->|Research task| SUB[🤖 Use Subagent]

    Q1 -->|Yes| CLAUDE[📄 Use CLAUDE.md]
    Q1 -->|Sometimes| SKILL1[📚 Use Skill]

    Q2 -->|Small<br/>Under 500 lines| CLAUDE2[📄 Use CLAUDE.md]
    Q2 -->|Large| SKILL2[📚 Use Skill]

    Q3 -->|I trigger it| SKILL3[📚 Use Action Skill<br/>/command-name]
    Q3 -->|Claude decides| SKILL4[📚 Use Reference Skill]

    style START fill:#e1f5ff
    style CLAUDE fill:#bbdefb
    style CLAUDE2 fill:#bbdefb
    style SKILL1 fill:#ce93d8
    style SKILL2 fill:#ce93d8
    style SKILL3 fill:#ce93d8
    style SKILL4 fill:#ce93d8
    style MCP fill:#ffab91
    style HOOKS fill:#fff59d
    style SUB fill:#80cbc4
```

---

## Quick Reference Table

| Feature | Always On? | Context Cost | Best For | Example |
|---------|-----------|-------------|----------|---------|
| 📄 **CLAUDE.md** | ✅ Yes | High (always loaded) | Project rules, conventions | "Use pnpm, not npm" |
| 📚 **Skills** | ❌ No (on-demand) | Low until used | Workflows, reference docs | `/deploy` command, API guide |
| 🤖 **Subagents** | ❌ No (spawned) | Isolated context | Research, parallel work | "Research authentication across 50 files" |
| 🔌 **MCP** | ✅ Yes (tools) | Medium | External services | Query database, post to Slack |
| ⚡ **Hooks** | ✅ Yes (triggers) | Zero | Automation | Run ESLint after edits |
| 📦 **Plugins** | Depends | Varies | Bundle & distribute | Package skills+hooks together |

---

## Context and Memory Explained

One important concept: **context** is Claude's short-term memory for the current session.

### How Context Works

```mermaid
graph TB
    subgraph CONTEXT[Claude's Context - What Claude Remembers]
        SYSTEM[System Instructions]
        CLAUDE_MD[CLAUDE.md content]
        SKILLS_DESC[Skill descriptions]
        HISTORY[Your conversation]
        FILES[Files Claude read]
        TOOLS[Available tools from MCP]
    end

    MODEL[🧠 Claude's Brain] -->|Processes| CONTEXT
    CONTEXT -->|Informs| DECISIONS[Claude's Decisions]

    style CONTEXT fill:#fff9c4
    style MODEL fill:#bbdefb
    style DECISIONS fill:#c8e6c9
```

### Context Costs by Feature

```mermaid
graph TB
    subgraph LOW[Low Context Cost]
        S1[Skills - only descriptions<br/>until used]
    end

    subgraph MEDIUM[Medium Context Cost]
        M1[MCP - tool definitions]
    end

    subgraph HIGH[High Context Cost]
        H1[CLAUDE.md - full content<br/>every request]
        H2[Loaded skills - full content]
    end

    subgraph ZERO[Zero Context Cost]
        Z1[Hooks - run externally]
        Z2[Subagents - isolated context]
    end

    style LOW fill:#c8e6c9
    style MEDIUM fill:#fff9c4
    style HIGH fill:#ffab91
    style ZERO fill:#e1f5ff
```

**Key Insight:** Use CLAUDE.md for essentials only. Move large docs to skills that load on-demand.

---

## Real-World Example: Complete Setup

Let's see how a real project might use all these features together.

### Scenario: E-commerce Platform Development

```mermaid
graph TB
    subgraph ALWAYS_ON[Always Active]
        CM[📄 CLAUDE.md<br/>Use TypeScript<br/>Run tests before commit<br/>Follow REST conventions]
        MCP_DB[🔌 MCP Database]
        MCP_SLACK[🔌 MCP Slack]
    end

    subgraph ON_DEMAND[On-Demand]
        SK_DEPLOY["📚 /deploy skill<br/>Deployment workflow"]
        SK_API[📚 API docs skill<br/>Endpoint reference]
        SK_REVIEW["📚 /review skill<br/>Code review checklist"]
    end

    subgraph BACKGROUND[Background Automation]
        HK_LINT[⚡ Hook ESLint<br/>After file edits]
        HK_NOTIFY[⚡ Hook Notify<br/>On deploy]
    end

    subgraph WORKERS[Workers]
        SUB[🤖 Subagents<br/>Parallel code reviews]
    end

    CLAUDE[🤖 Claude Code]

    ALWAYS_ON --> CLAUDE
    ON_DEMAND -.->|When needed| CLAUDE
    CLAUDE -->|Spawns| WORKERS

    style CLAUDE fill:#c8e6c9
    style CM fill:#bbdefb
    style MCP_DB fill:#ffab91
    style MCP_SLACK fill:#ffab91
    style SK_DEPLOY fill:#ce93d8
    style SK_API fill:#ce93d8
    style SK_REVIEW fill:#ce93d8
    style HK_LINT fill:#fff59d
    style HK_NOTIFY fill:#fff59d
    style SUB fill:#80cbc4
```

### What Each Piece Does

**CLAUDE.md** (Always active)
```markdown
# Project Rules
- Use TypeScript for all new files
- Run `npm test` before committing
- Follow REST API conventions in api-docs skill
```

**Skills** (Load when needed)
- `/deploy` - Full deployment workflow (you trigger)
- `api-docs` - 100-page API reference (Claude loads when needed)
- `/review` - Code review checklist (you trigger)

**MCP Servers** (Always connected)
- Database MCP - Query customer/order data
- Slack MCP - Send notifications

**Hooks** (Run on events)
- Post-edit hook - Run ESLint
- Post-deploy hook - Send Slack notification

**Subagents** (Spawned for big tasks)
- When you run `/review`, spawns 3 subagents:
  - Security checker
  - Performance analyzer
  - Test coverage checker

### Example Interaction

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant CLAUDE.md
    participant API Skill
    participant Database MCP
    participant Subagent

    Note over Claude: Session starts
    Claude->>CLAUDE.md: Reads rules

    You->>Claude: "Add an endpoint to get user orders"
    Claude->>API Skill: Loads API documentation
    Claude->>Database MCP: Queries schema

    Note over Claude: Writes code following<br/>TypeScript rules from CLAUDE.md<br/>API patterns from skill<br/>Database structure from MCP

    Claude->>You: Created /api/users/{id}/orders endpoint

    You->>Claude: /review
    Claude->>Subagent: Spawn security review
    Note over Subagent: Checks for SQL injection,<br/>auth issues, etc.
    Subagent->>Claude: Security: OK ✓

    Claude->>You: Review complete. All checks passed!
```

---

## Getting Started Checklist

Here's the recommended order to add features:

```mermaid
graph TD
    START[🎯 Starting Out]

    START --> STEP1[1️⃣ Create CLAUDE.md<br/>Add basic project rules]
    STEP1 --> STEP2[2️⃣ Add skills as needed<br/>Start with workflows you repeat]
    STEP2 --> STEP3[3️⃣ Connect MCP servers<br/>If you need external services]
    STEP3 --> STEP4[4️⃣ Try subagents<br/>For research and parallel work]
    STEP4 --> STEP5[5️⃣ Add hooks<br/>Automate repetitive tasks]
    STEP5 --> STEP6[6️⃣ Package as plugin<br/>Share with your team]

    style START fill:#e1f5ff
    style STEP1 fill:#bbdefb
    style STEP2 fill:#ce93d8
    style STEP3 fill:#ffab91
    style STEP4 fill:#80cbc4
    style STEP5 fill:#fff59d
    style STEP6 fill:#ce93d8
```

### Step 1: Start with CLAUDE.md

Create `/your-project/CLAUDE.md`:
```markdown
# Project Guidelines

## Language
Use TypeScript for all new files.

## Package Manager
Use pnpm instead of npm.

## Testing
Run tests before committing: `npm test`
```

### Step 2: Add Your First Skill

Create `/skills/deploy/SKILL.md`:
```markdown
---
name: deploy
description: Deploy the application
---

# Deployment Steps

1. Run tests: `npm test`
2. Build: `npm run build`
3. Deploy: `./deploy.sh`
```

Use it: `/deploy`

### Step 3: Try the Rest

As you need more capabilities, add:
- MCP servers for external services
- Subagents for research tasks
- Hooks for automation
- Plugins to share with team

---

## Summary: The Complete Picture

```mermaid
graph TB
    YOU[👤 You]

    subgraph CLAUDE_CODE[🤖 Claude Code System]
        MAIN[Main Claude<br/>Your Conversation]

        subgraph ALWAYS[Always Available]
            CM[📄 CLAUDE.md<br/>Project Memory]
            MCP[🔌 MCP<br/>External Services]
        end

        subgraph ONDEMAND[On-Demand]
            SK[📚 Skills<br/>Knowledge & Workflows]
            SA[🤖 Subagents<br/>Helper Workers]
        end

        subgraph AUTO[Automatic]
            HK[⚡ Hooks<br/>Event Scripts]
        end
    end

    PLUGIN[📦 Plugins<br/>Package Everything]

    YOU <--> MAIN
    ALWAYS --> MAIN
    ONDEMAND -.-> MAIN
    AUTO -.-> MAIN

    PLUGIN -.->|Bundles| CM
    PLUGIN -.->|Bundles| SK
    PLUGIN -.->|Bundles| MCP
    PLUGIN -.->|Bundles| HK

    style YOU fill:#e1f5ff
    style MAIN fill:#c8e6c9
    style CM fill:#bbdefb
    style SK fill:#ce93d8
    style MCP fill:#ffab91
    style SA fill:#80cbc4
    style HK fill:#fff59d
    style PLUGIN fill:#ce93d8
```

### Remember

1. **Start simple** - Begin with CLAUDE.md for basic rules
2. **Add as needed** - Don't over-engineer. Add skills when you have repeated workflows
3. **Keep it clean** - Put only essentials in CLAUDE.md, use skills for the rest
4. **Isolate when needed** - Use subagents to keep your main conversation focused
5. **Automate the boring** - Use hooks for predictable automation
6. **Share with team** - Package as plugins once your setup is solid

---

## Where to Learn More

- **CLAUDE.md**: [Documentation](https://code.claude.com/docs/en/memory)
- **Skills**: [Documentation](https://code.claude.com/docs/en/skills)
- **Subagents**: [Documentation](https://code.claude.com/docs/en/sub-agents)
- **MCP**: [Documentation](https://code.claude.com/docs/en/mcp)
- **Hooks**: [Documentation](https://code.claude.com/docs/en/hooks)
- **Plugins**: [Documentation](https://code.claude.com/docs/en/plugins)

---

## Quick Glossary

| Term | Simple Definition |
|------|------------------|
| **Context** | Claude's short-term memory for the current session |
| **Skill** | A package of instructions or workflows Claude can use |
| **Subagent** | A helper Claude that works on tasks in isolation |
| **MCP** | Connection to external services (database, Slack, etc.) |
| **Hook** | Script that runs automatically on events |
| **Plugin** | Bundle of skills, hooks, and configurations |
| **CLAUDE.md** | File Claude reads every session for project rules |
| **Marketplace** | Place to discover and install plugins |

---

That's it! You now understand all the core concepts of extending Claude Code. Start with CLAUDE.md and add features as you need them. Happy coding! 🚀
