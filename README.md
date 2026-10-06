# 🧠 Synapse CLI

> **An NLP-powered Command Line AI Agent and Developer Assistant with OAuth 2.0 Device Flow, Multi-Tool Execution, and Autonomous Application Generation.**

Synapse CLI bridges the gap between terminal workflows and modern Large Language Models (LLMs). Built with Google's Generative AI (`gemini-2.5-flash`), Prisma ORM, and Better Auth, it provides developers with seamless, context-aware AI assistance directly within their native shell environment.

---

## 🌟 Key Features

1. **🔐 OAuth 2.0 Device Authorization Grant (Device Flow)**
   - Enterprise-grade terminal authentication mirroring AWS and GitHub CLI.
   - Zero-password terminal login: simply run `synapse login`, approve on the browser, and session tokens are securely handled and cached.

2. **💬 Interactive Context-Aware Chat**
   - Natural language programming assistance with real-time streaming output (`streamText`).
   - Terminal-optimized syntax highlighting and Markdown formatting via `marked-terminal`.
   - Continuous multi-turn conversation persistence stored in PostgreSQL.

3. **🛠️ Tool Calling & Autonomous Execution**
   - Seamlessly leverages external Google tools directly from the prompt:
     - **Google Search**: Live real-time documentation retrieval and web search.
     - **Python Code Execution**: Sandboxed Python execution for calculations, algorithm verification, and data processing.
     - **URL Context**: Deep inspection and summarization of arbitrary URLs.

4. **🤖 Agentic Mode (Autonomous Code Generation)**
   - Scaffolds end-to-end, runnable applications from natural language prompts.
   - Uses strict schema constraints (`generateObject` via Zod) to output full directory trees, write production-ready code files directly to disk, and output bootstrap shell commands.

5. **📝 Dedicated Prompt Architecture**
   - Centralized prompt registry ([`server/src/config/prompts.js`](file:///server/src/config/prompts.js)) and comprehensive engineering documentation ([`PROMPTS.md`](file:///PROMPTS.md)).

---

## 🏗️ Architecture & Tech Stack

```
┌────────────────────────────────────────────────────────┐
│                   Terminal (CLI)                       │
│  - Commander.js        - @clack/prompts (TUI)          │
│  - Boxen / Chalk       - Marked Terminal (Markdown)    │
└───────────────┬────────────────────────▲───────────────┘
                │                        │
         OAuth Device Flow        API Responses
                │                        │
┌───────────────▼──────────────┐  ┌──────┴────────────────┐
│      Next.js Web Portal      │  │    Express Backend    │
│  - Next.js 16 / React 19     │  │  - Better-Auth Core   │
│  - Tailwind CSS v4           │  │  - Prisma ORM         │
│  - /device & /approve routes │  │  - PostgreSQL / Neon  │
└──────────────────────────────┘  └──────┬────────────────┘
                                         │
                                   Vercel AI SDK
                                 (@ai-sdk/google)
                                         │
                                  ┌──────▼──────┐
                                  │Google Gemini│
                                  │2.5 Flash LLM│
                                  └─────────────┘
```

- **Frontend / Auth Portal**: Next.js 16.2, React 19, Tailwind CSS v4, Lucide Icons, Shadcn UI.
- **Backend Server**: Node.js, Express 5, Better-Auth, Prisma ORM.
- **Database**: PostgreSQL (Neon Serverless PostgreSQL).
- **AI Engine**: Vercel AI SDK (`ai`), `@ai-sdk/google`, Google Gemini (`gemini-2.5-flash`).
- **CLI Core**: `commander`, `@clack/prompts`, `chalk`, `boxen`, `marked`, `marked-terminal`, `yocto-spinner`, `zod`.

---

## 📋 Prerequisites

Before running the project, ensure you have:
1. **Node.js**: v18.0.0 or higher
2. **PostgreSQL Database**: A running PostgreSQL instance or a free [Neon](https://neon.tech) database.
3. **Google Gemini API Key**: Obtainable from [Google AI Studio](https://aistudio.google.com/).
4. **GitHub OAuth App**: Create one at [GitHub Developer Settings](https://github.com/settings/developers) with:
   - Homepage URL: `http://localhost:3000`
   - Authorization callback URL: `http://localhost:3005/api/auth/callback/github`

---

## 🚀 Quick Start Guide

### 1. Clone the Repository
```bash
git clone https://github.com/Garvit-2/Synapse-Cli.git
cd Synapse-Cli
```

### 2. Configure Backend Environment
Navigate to the `server` directory and copy the environment template:
```bash
cd server
cp .env.example .env
```
Open `.env` and fill in your database credentials and API keys:
```env
PORT=3005
DATABASE_URL="postgresql://user:password@localhost:5432/synapse?sslmode=require"
BETTER_AUTH_SECRET="your-32-char-random-secret"
BETTER_AUTH_URL="http://localhost:3005"
GITHUB_CLIENT_ID="your_github_client_id"
GITHUB_CLIENT_SECRET="your_github_client_secret"
GOOGLE_GENERATIVE_AI_API_KEY="your_gemini_api_key"
SYNAPSE_MODEL="gemini-2.5-flash"
```

### 3. Install Dependencies & Initialize Database
Inside the `server` directory:
```bash
npm install
npx prisma generate
npx prisma db push
```

### 4. Start the Backend Server (Terminal 1)
```bash
npm start
# Server will start on http://localhost:3005
```

### 5. Start the Next.js Client (Terminal 2)
In a new terminal window:
```bash
cd client
npm install
npm run dev
# Web authentication dashboard will start on http://localhost:3000
```

### 6. Link & Run Synapse CLI (Terminal 3)
In a third terminal window:
```bash
cd server
npm link
```
Now `synapse` is globally available! Alternatively, run with `node src/cli/main.js`.

---

## 💻 CLI Usage Guide

### 1. Authentication
Initiate the device flow login:
```bash
synapse login
```
The CLI will generate a user code and open your browser to `http://localhost:3000/device`. Sign in with GitHub and authorize the device. The CLI will automatically receive the session token and store it in `~/.better-auth/token.json`.

Check your authenticated status anytime:
```bash
synapse whoami
```

### 2. Launching the AI Assistant
Wake up Synapse:
```bash
synapse wakeup
```
Select from three modes:
- **Chat**: Interactive conversational streaming assistant.
- **Tool Calling**: Multi-tool assistant with Google Search, Code Execution, and URL context.
- **Agentic Mode**: Autonomous software generator that scaffolds complete project folders and code files.

### 3. Ending Session
```bash
synapse logout
```

---

## 📁 Repository Structure

```
Synapse-Cli/
├── .env.example                # Global environment configuration template
├── PROMPTS.md                  # Comprehensive prompt engineering & prompt registry
├── README.md                   # Complete project documentation
├── project_report.md           # Formal project report & architectural paper
├── client/                     # Next.js 16 Web Authentication Portal
│   ├── app/
│   │   ├── (auth)/             # Sign-in & authentication routes
│   │   ├── approve/            # Device flow approval dashboard
│   │   └── device/             # Device code input interface
│   ├── lib/auth-client.ts      # Better Auth client configuration
│   └── package.json
└── server/                     # Express Backend, Prisma ORM & Synapse CLI
    ├── .env.example            # Server-specific environment template
    ├── prisma/
    │   └── schema.prisma       # Database schema (User, Session, Conversation, Message)
    ├── src/
    │   ├── cli/
    │   │   ├── ai/             # AI service layer (streamText, tools integration)
    │   │   ├── chat/           # Chat, tool, and agent loop handlers
    │   │   ├── commands/       # CLI commands (login, wakeup, logout, whoami)
    │   │   └── main.js         # CLI entry point
    │   ├── config/
    │   │   ├── agent.config.js # Application generator schema & file creator
    │   │   ├── google.config.js# AI SDK & model configuration
    │   │   ├── prompts.js      # Central prompt engineering registry
    │   │   └── tool.config.js  # External Google tools configuration
    │   ├── index.js            # Express server entry point
    │   ├── lib/                # Database & authentication helpers
    │   └── services/           # Conversation & message database services
    └── package.json
```

---

## 📑 Project Requirements Compliance

| Professor Requirement | How It Is Satisfied in Synapse CLI |
| :--- | :--- |
| **Solo Project** | Individual development adhering to modular architecture. |
| **NLP-based with LLM API Calls** | Uses Google Gemini (`gemini-2.5-flash`) via `@ai-sdk/google` and Vercel AI SDK for natural language parsing, code synthesis, and tool calling. |
| **Meaningful Implementation** | 3 distinct operational modes (Chat, Tool Calling, Agentic Generation) backed by PostgreSQL multi-turn persistence and OAuth 2.0 Device Flow. |
| **Prompt File Included** | Dedicated [`PROMPTS.md`](file:///PROMPTS.md) and [`server/src/config/prompts.js`](file:///server/src/config/prompts.js) showcasing persona engineering, zero-placeholder enforcement, and token optimization. |
| **Configuration File Included** | `.env.example` in root and `server/.env.example`, plus configuration modules in `server/src/config/`. |
| **All Other Necessary Files** | Full Prisma schemas, build scripts, Next.js portal, CLI binary scripts, and setup guides. |
