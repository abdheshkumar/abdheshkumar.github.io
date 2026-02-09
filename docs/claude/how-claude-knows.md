# How Does Claude Code Know Everything?

**Short answer:** I don't! But let me explain how my knowledge actually works.

## The Big Picture

```mermaid
graph TB
    subgraph TRAINING["1. Training Knowledge"]
        T1[📚 Books, articles, documentation<br/>up to January 2025]
        T2[💻 Code from open source<br/>programming patterns]
        T3[🌐 Public internet content<br/>General knowledge]
    end

    subgraph CONTEXT["2. Session Context"]
        C1[💬 Your conversation<br/>What you tell me]
        C2[📄 Files I read<br/>Your project code]
        C3[📋 CLAUDE.md<br/>Project instructions]
        C4[📚 Loaded skills<br/>Domain knowledge]
    end

    subgraph TOOLS["3. Real-Time Tools"]
        TO1[🔍 Search your codebase<br/>Grep, Glob]
        TO2[📖 Read any file<br/>Read tool]
        TO3[🌐 Browse the web<br/>WebFetch]
        TO4[🔌 Query external services<br/>MCP servers]
    end

    CLAUDE[🤖 Claude Code]

    TRAINING -->|Built into my brain| CLAUDE
    CONTEXT -->|Current session| CLAUDE
    TOOLS -->|I can use to find info| CLAUDE

    style CLAUDE fill:#c8e6c9
    style TRAINING fill:#bbdefb
    style CONTEXT fill:#fff9c4
    style TOOLS fill:#ffab91
```

Let me break down each source of "knowledge"...

---

## 1. Training Knowledge - What I Learned Before

### How I Was Trained

I'm an AI model (Claude Sonnet 4.5) trained on a massive dataset. Think of it like going to school for years.

```mermaid
graph LR
    subgraph TRAINING_DATA["Training Data"]
        D1[📚 Books & Articles]
        D2[💻 Open Source Code<br/>GitHub, Stack Overflow]
        D3[🌐 Websites & Docs]
        D4[📰 News & Information]
    end

    TRAINING_DATA -->|Trained on| NEURAL[🧠 Neural Network<br/>175B+ parameters]
    NEURAL -->|Becomes| CLAUDE[🤖 Claude Model]

    NOTE[⚠️ Knowledge cutoff:<br/>January 2025]

    style TRAINING_DATA fill:#bbdefb
    style NEURAL fill:#ce93d8
    style CLAUDE fill:#c8e6c9
    style NOTE fill:#ffab91
```

### What I Know From Training

**Programming Languages:**
- Syntax, patterns, best practices
- Common libraries and frameworks
- Design patterns and algorithms

**Example:**
```
You: "How do I sort an array in JavaScript?"
Me: *From training* "You can use .sort() method..."
```

**General Knowledge:**
- History, science, math
- Technology concepts
- Common facts and information

**Example:**
```
You: "What is REST API?"
Me: *From training* "REST stands for Representational State Transfer..."
```

### What I DON'T Know From Training

```mermaid
graph TB
    DONT[❌ Things I DON'T Know]

    DONT --> N1[🔴 Your project's code<br/>I haven't seen it yet]
    DONT --> N2[🔴 Internal company systems<br/>Not in public training data]
    DONT --> N3[🔴 Recent events after Jan 2025<br/>Knowledge cutoff]
    DONT --> N4[🔴 Your specific requirements<br/>You haven't told me]
    DONT --> N5[🔴 Private APIs/services<br/>Not publicly documented]

    style DONT fill:#ffcdd2
    style N1 fill:#ffcdd2
    style N2 fill:#ffcdd2
    style N3 fill:#ffcdd2
    style N4 fill:#ffcdd2
    style N5 fill:#ffcdd2
```

**Important:** My training knowledge is **frozen at January 2025**. I don't know about:
- Events after January 2025
- New libraries released after that
- Your company's internal systems
- Your project's specific code (until I read it)

---

## 2. Session Context - What I See Right Now

Every time you talk to me, I have a "context window" - like short-term memory for our conversation.

### What Goes Into Context

```mermaid
graph TB
    CONTEXT[📦 Context Window<br/>My Current Memory]

    CONTEXT --> C1[💬 Conversation History<br/>Everything you and I said]
    CONTEXT --> C2[📄 Files I've Read<br/>Code you asked me to read]
    CONTEXT --> C3[📋 CLAUDE.md<br/>Project instructions]
    CONTEXT --> C4[📚 Loaded Skills<br/>Domain-specific knowledge]
    CONTEXT --> C5[⚙️ System Instructions<br/>How Claude Code works]
    CONTEXT --> C6[🔧 Available Tools<br/>What I can do]
    CONTEXT --> C7[🗄️ Git Status<br/>Current branch, changes]

    style CONTEXT fill:#fff9c4
    style C1 fill:#e1f5ff
    style C2 fill:#e1f5ff
    style C3 fill:#e1f5ff
    style C4 fill:#e1f5ff
    style C5 fill:#e1f5ff
    style C6 fill:#e1f5ff
    style C7 fill:#e1f5ff
```

### How Context Works

```mermaid
sequenceDiagram
    participant You
    participant Context
    participant Claude

    Note over Context: Empty at start

    You->>Context: "I'm building a React app"
    Context->>Claude: Processes with training knowledge

    Claude->>You: "Great! What can I help with?"

    You->>Context: "Read src/App.js"
    Note over Context: Claude uses Read tool
    Context->>Context: Adds file content

    You->>Context: "Fix the bug in the handleClick"
    Context->>Claude: Processes with:<br/>• Training (React knowledge)<br/>• Context (App.js code)<br/>• Context (previous messages)

    Claude->>You: "I see the issue on line 42..."
    Note over Context: My response also added to context
```

### Example: How I Learn About Your Project

**Turn 1:**
```
You: "I have a Node.js project"
Context: "User has Node.js project"
My Knowledge: General Node.js (from training) + Your statement (from context)
```

**Turn 2:**
```
You: "Read package.json"
*I use Read tool*
Context: "User has Node.js project" + "package.json contents"
My Knowledge: Now I know your dependencies, scripts, etc.
```

**Turn 3:**
```
You: "Fix the authentication bug"
Context: All of the above + "User wants auth bug fixed"
My Knowledge: General auth patterns (training) + Your specific code (context)
```

### Context Limitations

```mermaid
graph TB
    LIMIT[⚠️ Context Window Limits]

    LIMIT --> L1[📏 Size Limit<br/>~200K tokens<br/>≈ 150K words]
    LIMIT --> L2[⏱️ Time Limit<br/>Only this session<br/>Resets on restart]
    LIMIT --> L3[🧹 Auto-Summarization<br/>Old messages summarized<br/>to save space]

    style LIMIT fill:#fff9c4
    style L1 fill:#ffab91
    style L2 fill:#ffab91
    style L3 fill:#ffab91
```

**What this means:**
- If we talk a LOT, old messages get summarized
- If I read MANY files, context fills up
- Each session starts fresh (but CLAUDE.md loads again)

---

## 3. Real-Time Tools - How I Access Information

I can actively look up information using tools!

### My Toolbox

```mermaid
graph TB
    TOOLS[🛠️ My Tools]

    subgraph SEARCH[Search Tools]
        S1[🔍 Grep<br/>Search file contents]
        S2[📁 Glob<br/>Find files by pattern]
    end

    subgraph FILE[File Tools]
        F1[📖 Read<br/>Read any file]
        F2[✏️ Edit<br/>Modify files]
        F3[📝 Write<br/>Create files]
    end

    subgraph EXECUTE[Execution Tools]
        E1[⚡ Bash<br/>Run commands]
        E2[🐙 Git<br/>Version control]
    end

    subgraph EXTERNAL[External Tools]
        EX1[🌐 WebFetch<br/>Browse websites]
        EX2[🔌 MCP<br/>External services]
    end

    subgraph META[Meta Tools]
        M1[🤖 Task<br/>Spawn subagents]
        M2[❓ AskUserQuestion<br/>Ask you questions]
    end

    TOOLS --> SEARCH
    TOOLS --> FILE
    TOOLS --> EXECUTE
    TOOLS --> EXTERNAL
    TOOLS --> META

    style TOOLS fill:#c8e6c9
```

### How I Use Tools to "Know" Things

#### Example 1: Finding Information

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant Grep Tool
    participant Files

    You->>Claude: "Where is the authentication code?"

    Note over Claude: I DON'T know this from training<br/>or context - I need to search!

    Claude->>Grep Tool: Search for "authentication"
    Grep Tool->>Files: Searches all files
    Files->>Grep Tool: Found in: src/auth.js, utils/auth.ts
    Grep Tool->>Claude: Returns file paths

    Claude->>You: "Authentication code is in:<br/>• src/auth.js<br/>• utils/auth.ts"

    Note over Claude: Now I "know" where it is!
```

#### Example 2: Learning Your Code

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant Read Tool
    participant File

    You->>Claude: "How does user login work?"

    Note over Claude: Training: General auth patterns<br/>Context: Nothing specific yet<br/>Need to read code!

    Claude->>Read Tool: Read src/auth.js
    Read Tool->>File: Reads file
    File->>Read Tool: Returns content
    Read Tool->>Claude: Adds to context

    Note over Claude: Now I have:<br/>• General auth knowledge (training)<br/>• Your specific implementation (context)

    Claude->>You: "Your login uses JWT tokens,<br/>stored in localStorage,<br/>validated on line 45..."
```

#### Example 3: Getting Latest Info

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant WebFetch
    participant Internet

    You->>Claude: "What's the latest React version?"

    Note over Claude: Training cutoff: Jan 2025<br/>I might not know if newer!

    Claude->>WebFetch: Fetch reactjs.org
    WebFetch->>Internet: HTTP request
    Internet->>WebFetch: HTML content
    WebFetch->>Claude: Page content

    Claude->>You: "The latest React version is 19.1.2"

    Note over Claude: Used web to get current info!
```

---

## 4. How Extensions Add Knowledge

Extensions like CLAUDE.md, Skills, and MCP augment what I know.

### Knowledge Layers

```mermaid
graph TB
    subgraph LAYER1[Base Layer - Always Present]
        BASE[🧠 Training Knowledge<br/>General programming, concepts]
    end

    subgraph LAYER2[Project Layer - Session Start]
        PROJ[📋 CLAUDE.md<br/>Project-specific rules]
    end

    subgraph LAYER3[Domain Layer - On Demand]
        DOM[📚 Skills<br/>Specialized knowledge]
    end

    subgraph LAYER4[External Layer - Live Access]
        EXT[🔌 MCP Servers<br/>External data sources]
    end

    BASE --> PROJ
    PROJ --> DOM
    DOM --> EXT

    EXT --> FINAL[🤖 Complete Knowledge<br/>for answering your question]

    style BASE fill:#bbdefb
    style PROJ fill:#fff9c4
    style DOM fill:#ce93d8
    style EXT fill:#ffab91
    style FINAL fill:#c8e6c9
```

### Example: Answering a Complex Question

**Question:** "How do I deploy using egctl to the production environment?"

```mermaid
graph TB
    Q[❓ Question:<br/>Deploy with egctl to prod]

    Q --> K1{Do I know<br/>what egctl is?}
    K1 -->|No, not in training| SKILL[📚 Load egctl skill]
    SKILL --> K1

    K1 -->|Now yes!| K2{Do I know<br/>project specifics?}
    K2 -->|Check CLAUDE.md| CM[📋 CLAUDE.md says:<br/>Use pnpm, test first]

    K2 --> K3{Do I know<br/>current state?}
    K3 --> READ[📖 Read deployment config]

    READ --> K4{Do I need<br/>external data?}
    K4 -->|Yes| MCP[🔌 MCP: Check cluster status]

    MCP --> ANSWER[✅ Complete Answer:<br/>• Training: General deployment<br/>• Skill: egctl specifics<br/>• CLAUDE.md: Project rules<br/>• Files: Your config<br/>• MCP: Current cluster state]

    style Q fill:#e1f5ff
    style SKILL fill:#ce93d8
    style CM fill:#fff9c4
    style READ fill:#ffab91
    style MCP fill:#ffab91
    style ANSWER fill:#c8e6c9
```

---

## 5. What I Actually "Know" vs. Can "Find Out"

Let me be honest about the difference:

### Things I Actually Know (In My Training)

```mermaid
graph LR
    subgraph KNOW[✅ I Actually Know These]
        K1[Programming languages<br/>syntax & patterns]
        K2[Algorithms &<br/>data structures]
        K3[Common frameworks<br/>React, Node.js, etc.]
        K4[General concepts<br/>REST, databases, etc.]
        K5[Public documentation<br/>that existed before Jan 2025]
    end

    style KNOW fill:#c8e6c9
```

**Example:**
```
You: "How do I reverse an array in JavaScript?"
Me: *Instantly from training* "Use .reverse() or [...arr].reverse()"
```

### Things I Can Find Out (Using Tools)

```mermaid
graph LR
    subgraph FIND[🔍 I Can Find These Out]
        F1[Your project's code<br/>*use Read tool*]
        F2[Where something is<br/>*use Grep/Glob*]
        F3[Current state<br/>*run commands*]
        F4[Latest documentation<br/>*use WebFetch*]
        F5[External data<br/>*use MCP*]
    end

    style FIND fill:#bbdefb
```

**Example:**
```
You: "Where is the user authentication logic?"
Me: *Needs to search* "Let me search your codebase..."
    *Uses Grep tool*
    "Found in src/auth/UserAuth.ts"
```

### Things I Don't Know and Can't Find

```mermaid
graph LR
    subgraph CANT[❌ I Cannot Know These]
        C1[What you're thinking<br/>*You need to tell me*]
        C2[Internal company secrets<br/>*Not in training or accessible*]
        C3[Passwords/credentials<br/>*Never stored or accessible*]
        C4[Your future plans<br/>*You need to tell me*]
        C5[Proprietary systems<br/>*Without MCP server*]
    end

    style CANT fill:#ffcdd2
```

---

## 6. The Complete Information Flow

Here's how I answer any question:

```mermaid
graph TB
    START[You ask a question]

    START --> CHECK1{Is it general<br/>programming?}
    CHECK1 -->|Yes| TRAINING[✅ Answer from training<br/>Quick response]

    CHECK1 -->|No| CHECK2{Is it in<br/>CLAUDE.md?}
    CHECK2 -->|Yes| CLAUDEMD[✅ Use CLAUDE.md rules]

    CHECK2 -->|No| CHECK3{Is there a<br/>relevant skill?}
    CHECK3 -->|Yes| SKILL[✅ Load skill]

    CHECK3 -->|No| CHECK4{Is it about<br/>your code?}
    CHECK4 -->|Yes| READ[🔍 Search/Read files]

    CHECK4 -->|No| CHECK5{Is it external<br/>information?}
    CHECK5 -->|Yes - web| WEB[🌐 Use WebFetch]
    CHECK5 -->|Yes - service| MCP[🔌 Use MCP]

    CHECK5 -->|No| ASK[❓ Ask you for clarification]

    TRAINING --> ANSWER
    CLAUDEMD --> ANSWER
    SKILL --> ANSWER
    READ --> ANSWER
    WEB --> ANSWER
    MCP --> ANSWER
    ASK --> ANSWER[📝 Provide answer]

    style START fill:#e1f5ff
    style ANSWER fill:#c8e6c9
    style TRAINING fill:#bbdefb
    style CLAUDEMD fill:#fff9c4
    style SKILL fill:#ce93d8
    style READ fill:#ffab91
    style WEB fill:#ffab91
    style MCP fill:#ffab91
    style ASK fill:#fff59d
```

---

## 7. Real-World Examples

### Example 1: Simple Question (Pure Training)

```mermaid
sequenceDiagram
    participant You
    participant Training
    participant Claude

    You->>Claude: "How do I sort an array in Python?"

    Note over Claude: This is general Python knowledge

    Claude->>Training: Recall from training
    Training->>Claude: "Use .sort() or sorted()"

    Claude->>You: "Use arr.sort() for in-place<br/>or sorted(arr) for a new list"

    Note over Claude: No tools needed!<br/>Instant answer from training
```

### Example 2: Project-Specific Question (Need to Search)

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant Grep
    participant Read
    participant Files

    You->>Claude: "Where do we validate user emails?"

    Note over Claude: Not in training or context<br/>Need to search!

    Claude->>Grep: Search for "email" and "validation"
    Grep->>Files: Searches codebase
    Files->>Grep: Found in: validators/email.js
    Grep->>Claude: Results

    Claude->>Read: Read validators/email.js
    Read->>Files: Reads file
    Files->>Read: File content
    Read->>Claude: Content in context

    Claude->>You: "Email validation is in validators/email.js<br/>Using regex: /^[^@]+@[^@]+\\.[^@]+$/"

    Note over Claude: Combined:<br/>• Training (validation concepts)<br/>• Tools (found and read your code)
```

### Example 3: Company-Specific Question (Need Skill + MCP)

```mermaid
sequenceDiagram
    participant You
    participant Claude
    participant Skill
    participant MCP
    participant Service

    You->>Claude: "Deploy to Expedia prod cluster"

    Note over Claude: "Expedia" and "cluster" not in<br/>general training!

    Claude->>Skill: Load "egctl" skill
    Skill->>Claude: Instructions for egctl deployment

    Note over Claude: Now I know egctl commands

    Claude->>MCP: Check cluster status
    MCP->>Service: Query Expedia service
    Service->>MCP: Cluster ready, version 3.2
    MCP->>Claude: Status info

    Claude->>You: "Cluster is ready (v3.2).<br/>Running: egctl deploy --env prod..."

    Note over Claude: Combined:<br/>• Training (general deployment)<br/>• Skill (egctl specifics)<br/>• MCP (live cluster status)
```

---

## 8. My Limitations (Being Honest!)

### What I'm Good At

```mermaid
graph LR
    GOOD[✅ I'm Good At]

    GOOD --> G1[General programming<br/>Common patterns]
    GOOD --> G2[Reading & understanding<br/>your code]
    GOOD --> G3[Searching & finding<br/>information]
    GOOD --> G4[Following instructions<br/>from CLAUDE.md & skills]
    GOOD --> G5[Learning from context<br/>within session]

    style GOOD fill:#c8e6c9
```

### What I'm Not Good At

```mermaid
graph LR
    BAD[⚠️ I'm Not Good At]

    BAD --> B1[Remembering across sessions<br/>*Context resets*]
    BAD --> B2[Knowing internal systems<br/>*Without documentation*]
    BAD --> B3[Guessing requirements<br/>*Need you to tell me*]
    BAD --> B4[Knowing latest events<br/>*After Jan 2025 cutoff*]
    BAD --> B5[Reading minds<br/>*You must be explicit*]

    style BAD fill:#ffab91
```

### Common Misconceptions

| ❌ Wrong Belief | ✅ Reality |
|----------------|-----------|
| "Claude knows my entire codebase" | I only know files I've read or searched |
| "Claude remembers from last session" | Each session starts fresh (but CLAUDE.md loads) |
| "Claude knows all Expedia systems" | I only know what's documented or accessible via MCP |
| "Claude knows the latest news" | Training cutoff is January 2025, but I can search web |
| "Claude can see my screen" | I can only see files you ask me to read |

---

## 9. How to Help Me Know What You Need

### Be Explicit

```mermaid
graph TB
    subgraph GOOD[✅ Good: Be Specific]
        G1["Fix the authentication bug<br/>in src/auth/login.js"]
        G2["Use PostgreSQL instead of MySQL<br/>for the user database"]
        G3["Follow our company's API guidelines<br/>in docs/api-standards.md"]
    end

    subgraph BAD[❌ Vague: Hard to Help]
        B1["Fix the bug"]
        B2["Use a database"]
        B3["Follow best practices"]
    end

    style GOOD fill:#c8e6c9
    style BAD fill:#ffab91
```

### Provide Context

```mermaid
sequenceDiagram
    participant You
    participant Claude

    Note over You,Claude: ❌ Vague Request

    You->>Claude: "The login isn't working"
    Claude->>You: "Can you show me the code?<br/>What's the error?<br/>Which file?"

    Note over You,Claude: Many back-and-forth messages

    Note over You,Claude: ✅ Clear Request

    You->>Claude: "Login fails with 401 error.<br/>See src/auth/login.ts<br/>Error on line 42"
    Claude->>You: "I see the issue. The token<br/>validation is missing..."

    Note over You,Claude: Immediate, focused help
```

### Use Project Documentation

```mermaid
graph TB
    DOC[📋 Help Me Know by Documenting]

    DOC --> D1[📄 CLAUDE.md<br/>Always use pnpm<br/>Test before commit]
    DOC --> D2[📚 Skills<br/>egctl deployment process<br/>API guidelines]
    DOC --> D3[🔌 MCP Servers<br/>Connect to internal services<br/>Access company data]
    DOC --> D4[💬 Tell Me Directly<br/>We use microservices<br/>This is Node.js 20]

    style DOC fill:#c8e6c9
    style D1 fill:#fff9c4
    style D2 fill:#ce93d8
    style D3 fill:#ffab91
    style D4 fill:#e1f5ff
```

---

## 10. Summary

### The Truth About My Knowledge

```mermaid
graph TB
    CLAUDE[🤖 Claude Code's Knowledge]

    subgraph STATIC[Static Knowledge - Baked In]
        S1[🧠 Training<br/>General knowledge<br/>up to Jan 2025]
    end

    subgraph DYNAMIC[Dynamic Knowledge - Per Session]
        D1[📦 Context<br/>Current conversation<br/>Files I've read]
        D2[📋 CLAUDE.md<br/>Project rules]
        D3[📚 Skills<br/>Domain expertise]
    end

    subgraph ACTIVE[Active Knowledge - I Look It Up]
        A1[🔍 Search<br/>Find in codebase]
        A2[📖 Read<br/>Read files]
        A3[🌐 Web<br/>Current info]
        A4[🔌 MCP<br/>External services]
    end

    CLAUDE --> STATIC
    CLAUDE --> DYNAMIC
    CLAUDE --> ACTIVE

    NOTE[I combine all sources<br/>to answer your questions]

    STATIC --> NOTE
    DYNAMIC --> NOTE
    ACTIVE --> NOTE

    style CLAUDE fill:#c8e6c9
    style STATIC fill:#bbdefb
    style DYNAMIC fill:#fff9c4
    style ACTIVE fill:#ffab91
    style NOTE fill:#e1f5ff
```

### Key Takeaways

1. **Training** = General knowledge frozen at January 2025
2. **Context** = What I see in current session (your messages + files I read)
3. **Tools** = How I actively find information (search, read, web, MCP)
4. **Extensions** = How you teach me about your project (CLAUDE.md, Skills, MCP)

### I'm Most Effective When:

✅ You tell me what you're working on
✅ You point me to relevant files
✅ You use CLAUDE.md for project rules
✅ You create skills for company-specific knowledge
✅ You connect MCP servers to internal systems
✅ You're specific about what you need

### I Struggle When:

❌ You assume I know your codebase without reading it
❌ You refer to internal systems without documentation
❌ You're vague about requirements
❌ You expect me to remember from previous sessions
❌ You ask about very recent events (after Jan 2025)

---

## Final Thought

**I'm not all-knowing, but I'm good at learning!**

Think of me as a really smart assistant who:
- Has read a LOT of programming books (training)
- Can quickly read and understand your code (tools)
- Remembers our conversation (context)
- Can look things up instantly (search, web, MCP)
- Follows your instructions precisely (CLAUDE.md, Skills)

The better you teach me about your project (through documentation, skills, and clear communication), the better I can help! 🚀

---

## Questions to Reflect On

```mermaid
graph LR
    Q1{Does Claude know<br/>about my internal<br/>company systems?}
    Q1 -->|Only if| A1[Documented in Skills<br/>or accessible via MCP]

    Q2{Will Claude remember<br/>from last session?}
    Q2 -->|Only| A2[CLAUDE.md loads again<br/>But not conversation]

    Q3{How does Claude know<br/>where code is?}
    Q3 -->|By| A3[Searching with Grep/Glob<br/>or you telling me]

    style Q1 fill:#e1f5ff
    style Q2 fill:#e1f5ff
    style Q3 fill:#e1f5ff
    style A1 fill:#c8e6c9
    style A2 fill:#c8e6c9
    style A3 fill:#c8e6c9
```

Now you understand how I "know" things - it's a combination of training, context, tools, and the extensions you add! 🎓
