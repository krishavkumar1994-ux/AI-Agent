# Personal Agent — Development Principles & Agent Guidelines

Personal Agent is a Windows desktop AI assistant built from scratch to learn and master agentic AI engineering. This document defines the engineering standards, architecture, security policies, and collaboration workflow for all development.

---

## 1. Core Engineering Principles

1. **Incremental Implementation**  
   Implement the project incrementally, file by file and function by function. Avoid monolithic or premature multi-file additions.

2. **Inspect Before Modifying**  
   Before changing or creating any file, inspect the existing project structure, existing patterns, and relevant dependencies to maintain consistency.

3. **Pre-Implementation File Specifications**  
   Before writing or updating code, clearly explain:
   - The file's single responsibility and purpose.
   - Its inputs and outputs (data contracts, types, interfaces).
   - Its dependencies (internal modules and external packages).

4. **Transparent Design & Trade-Off Analysis**  
   For any non-trivial design decision, explicitly explain:
   - The architectural rationale and pattern chosen.
   - Viable alternatives considered.
   - Pros, cons, and performance trade-offs.
   - Security and maintainability implications.

5. **Targeted, Scoped Modifications & Reviewable Diffs**  
   - Modify only the files explicitly required for the current task. Never overwrite or regenerate the entire project unless explicitly instructed.
   - Never silently change dependencies, architecture, or unrelated files.
   - All AI-generated edits must be inspectable as discrete diffs and backed by Git checkpoints to allow easy review and revert capabilities.

6. **Change & Command Authorizations**  
   Do not install new packages, execute destructive shell commands, alter system settings, or access external user accounts without explicit prior approval.

7. **Strict Credential & Secret Hygiene**  
   Never hardcode, log, screenshot, or commit API keys, access tokens, passwords, or secrets. All secrets must be managed via secure environment variables or OS-level credential stores and excluded from version control.

---

## 2. Target Architecture & Technology Stack

8. **Architecture Stack**  
   - **Frontend / Desktop Container**: Electron + React + TypeScript.
   - **Backend / Agent Engine**: Python + FastAPI.
   - **Persistence**: SQLite for local structured state, session history, and metadata.
   - **Model Layer**: Cloud LLM APIs (e.g., Gemini, OpenAI, Claude) accessed via controlled backend adapters.

9. **Desktop Security & Electron Isolation**  
   - Electron must run with `contextIsolation: true` and `nodeIntegration: false`.
   - Communication between the renderer and main process must use a minimal, strictly validated IPC bridge defined in `preload.ts`.
   - The renderer must never have direct filesystem, shell, or raw network access to local system resources.
   - **Local API Boundaries**: The Electron renderer must not have unrestricted network access to local resources. Restrict the backend to local connections initially, validate all requests strictly, and configure CORS deliberately. Localhost alone is not treated as an implicit trust boundary.

10. **Backend Orchestration & Tool Guardrails**  
    - The Python backend owns agent orchestration, tool schema definitions, validation, permission checks, and execution logging.
    - **Never give the LLM unrestricted shell or OS access.** All tools must be specific, sandboxed, and schema-bounded.

---

## 3. Safety, Autonomy & Security Policies

11. **Human-in-the-Loop & Permission Boundaries**  
    - Predefined safe, idempotent actions (e.g., local state lookups, read-only formatting, schema validation) may execute automatically.
    - Consequential operations—such as sending messages, deleting or overwriting files, modifying system resources, or initiating transactions—require explicit user confirmation unless a narrowly scoped, user-approved policy is in effect.
    - **Backend-Enforced Approvals**: Approval must be enforced directly by the backend, not merely delegated to the model or UI. An action requiring approval must never execute until the backend independently verifies that the user approved that specific action and its exact arguments.

12. **Defense Against Untrusted Input & Prompt Injection**  
    - Treat all retrieved content—including emails, web pages, downloaded documents, and external API responses—as untrusted data.
    - Untrusted content must be segregated from system instructions and sanitized to mitigate direct and indirect prompt injection attacks.

---

## 4. Verification & Collaboration Cadence

13. **Focused, Test-Driven Verification**  
    - Write and run focused, automated tests (unit and integration) for each meaningful implementation step.
    - Verify functional correctness before moving to subsequent tasks.

14. **Standardized Post-Task Reporting**  
    After completing every task, provide a concise summary containing:
    - **Files Changed**: List of created or modified files.
    - **Key Decisions**: Rationale, trade-offs, and security choices made.
    - **Commands Executed**: Any environment or test commands executed.
    - **Tests & Verification**: Test results and observed outcomes.
    - **Remaining Work & Open Issues**: Next steps and potential blockers.

15. **Strict Cadence & Review Gates**  
    Stop immediately after completing the requested task. Do not jump ahead into future steps. Wait for review and approval before proceeding.
