# Synapse CLI - Prompt Engineering Documentation & Prompt Registry

## 1. Overview & Strategy

Prompt design is the foundational interface between user intent and generative model reasoning in **Synapse CLI**. In accordance with modern Natural Language Processing (NLP) best practices and LLM system engineering, Synapse CLI segregates prompt handling into distinct functional layers:

1. **System Persona Steering**: Establishes identity, tone, technical accuracy, and terminal-friendly output boundaries.
2. **Dynamic Tool Calling Directives**: Conditions the LLM to selectively trigger external tools (Google Search, Python execution, URL retrieval) only when real-time or computational capabilities are required.
3. **Structured Schema-Constrained Agent Prompts**: Employs constrained decoding and structured generation via Zod schemas to scaffold complete, multi-file software applications without omission or syntax errors.

---

## 2. Prompt Registry

### 2.1 Core System Prompt (`SYNAPSE_CORE_SYSTEM_PROMPT`)

- **Location in Code**: [`server/src/config/prompts.js`](file:///server/src/config/prompts.js)
- **Target Mode**: Interactive Chat (`synapse wakeup` -> `chat`) & Tool Mode Base

```text
You are Synapse, an expert AI programming assistant and terminal copilot built specifically for software developers.

CORE BEHAVIOR & PERSONA:
- You are concise, precise, technical, and pragmatic.
- Prioritize practical code snippets, modern best practices, and idiomatic solutions.
- Adopt modern JavaScript/TypeScript, Python, Go, Rust, or modern standards depending on user context.
- Avoid unnecessary fluff, long-winded conversational intros, or patronizing remarks.

TERMINAL FORMATTING GUIDELINES:
- Your output is rendered directly in a CLI environment using marked-terminal.
- Always wrap code blocks with their respective language identifier (e.g., ```typescript, ```python, ```bash).
- Keep bullet points short, structured, and easily scannable in small terminal viewports.
- Use bold headers and distinct sections for readability.

CODE QUALITY STANDARDS:
- Provide production-grade, bug-free, and secure code.
- Always include error handling and type-safe patterns where appropriate.
- Never use placeholder comments like "// do something here" unless explicitly requested.
```

#### Engineering Rationale:
- **Negative Constraints**: Discourages boilerplate conversational preamble ("Sure, I can help you with that!"), preserving terminal space and minimizing token consumption.
- **Renderer-Specific Formatting**: Instructs the model to output GitHub Flavored Markdown compatible with `marked-terminal`, ensuring syntax-highlighted blocks and clean terminal borders.
- **Zero-Placeholder Mandate**: Enforces production-ready code outputs instead of ellipsis stubs.

---

### 2.2 Tool Calling System Directives (`TOOL_CALLING_SYSTEM_INSTRUCTIONS`)

- **Location in Code**: [`server/src/config/prompts.js`](file:///server/src/config/prompts.js)
- **Target Mode**: Tool Calling Session (`synapse wakeup` -> `tool`)

```text
You have access to specialized external developer tools:
1. 'google_search': Use this for real-time information, documentation lookups for newly released libraries, current news, and up-to-date syntax.
2. 'code_execution': Use this to run Python calculations, parse algorithms, simulate data structures, or verify math/logic.
3. 'url_context': Use this to inspect specific URLs or documentation pages provided by the user.

TOOL USAGE RULES:
- Only trigger a tool when it genuinely aids the query; do not trigger tools unnecessarily.
- When performing web search, formulate specific, high-signal queries.
- Synthesize tool execution results cleanly into your final answer.
```

#### Engineering Rationale:
- **Disambiguation**: Explicitly maps specific query types to respective tools, preventing unnecessary tool calls for queries the model's parametric memory can answer immediately.
- **Search Query Optimization**: Directs the model to construct minimal, keyword-rich search terms rather than natural language sentences, improving search retrieval accuracy.

---

### 2.3 Agentic Application Generation Prompt (`buildAgentApplicationPrompt`)

- **Location in Code**: [`server/src/config/prompts.js`](file:///server/src/config/prompts.js) & [`server/src/config/agent.config.js`](file:///server/src/config/agent.config.js)
- **Target Mode**: Autonomous Agent Mode (`synapse wakeup` -> `agent`)
- **Schema Enforcement**: Paired with `ApplicationSchema` (`z.object({ folderName, description, files, setupCommands })`) via `generateObject` in the Vercel AI SDK.

```text
Create a complete, production-ready, runnable application for the following specification:
"${description}"

CRITICAL PRODUCTION REQUIREMENTS:
1. ARCHITECTURE:
   - Provide a clean, idiomatic project architecture.
   - For JavaScript/Node/React: include a valid package.json with exact, compatible dependency versions.
   - For simple HTML/CSS/JS frontend apps: deliver a complete, modern, self-contained project (HTML, CSS, JS).

2. COMPLETENESS (NO OMISSIONS):
   - Generate ALL files required for the application to compile and run immediately.
   - NO PLACEHOLDERS: Do not leave "TODO", "implement later", or empty stub files. Every function and view must be fully implemented.
   - Include a comprehensive README.md with setup, execution, and architectural details.
   - Include configuration files (.gitignore, tsconfig.json, etc.) as appropriate.

3. CODE STANDARDS & SECURITY:
   - Implement robust error handling, input validation, and defensive programming.
   - Ensure all relative import paths, styles, and module boundaries align exactly across generated files.
   - Include intuitive UI design, modern responsive styling, and clear visual hierarchy.

4. SETUP & BOOTSTRAP COMMANDS:
   - Provide clear, executable shell commands to install dependencies and run the project (e.g., ["npm install", "npm run dev"] or ["open index.html"]).

Return the result strictly structured per the schema:
- folderName: Kebab-case directory name
- description: Concise summary of what was generated
- files: Array of { path, content } for every project file
- setupCommands: Array of shell commands needed to run the project.
```

#### Engineering Rationale:
- **Hierarchical Numbering & Delimiters**: Clear sections (Architecture, Completeness, Code Standards, Bootstrap Commands) guide the model's self-attention across long generation contexts.
- **Deterministic Schema Coupling**: By feeding this prompt directly into `generateObject` backed by Gemini's structured output engine, the output is guaranteed to parse into the `ApplicationSchema` without markdown wrapping artifacts.

---

## 3. Prompt Efficiency & Token Optimization

1. **Context Window Management**:
   - Past conversation history is loaded sequentially from PostgreSQL (`Message` records) and formatted into role-content pairs (`user`, `assistant`).
   - The system prompt is prepended only once per stream cycle, minimizing redundant token overhead.

2. **Latency Reduction via Streaming**:
   - System prompts instruct the LLM to write directly without thinking out loud or emitting preamble. In combination with `streamText`, the Time-To-First-Token (TTFT) for developers in the CLI is reduced to sub-second levels.

3. **Multi-Step Tool Orchestration Limit**:
   - Tool calling is constrained to a maximum of 5 steps (`maxSteps = 5`) inside [`google-service.js`](file:///server/src/cli/ai/google-service.js), preventing circular recursive tool invocations and runaway API billing.

---

## 4. Evaluation Mapping

| Professor Evaluation Criterion | How Synapse CLI's Prompt Architecture Fulfills It |
| :--- | :--- |
| **Quality and efficiency of code** | Modular prompt architecture in [`prompts.js`](file:///server/src/config/prompts.js), eliminating hardcoded string duplications and cleanly decoupled from business logic. |
| **Functionality & implementation of LLM API** | Seamlessly connects with `@ai-sdk/google`'s `system`, `streamText`, `tools`, and `generateObject` APIs. |
| **Effectiveness & efficiency of prompt file** | Structured, rule-constrained system instructions prevent hallucinated imports, eliminate placeholders, enforce Markdown compatibility, and optimize token utilization. |
