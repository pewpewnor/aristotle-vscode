# Aristotle for VS Code

![Aristotle logo](media/aristotle.png)

Aristotle for VS Code is the editor client for the [Aristotle backend](../aristotle). It adds an `@aristotle` chat participant that can answer questions about indexed codebases and send relevant editor context to the backend.

## What it does

- Adds the `@aristotle` participant to VS Code Chat.
- Sends the current prompt and the participant's conversation history to the backend's `POST /chat` endpoint.
- Includes files attached as chat references, except `copilot-instructions.md`, as additional context.
- Loads Python and Markdown files from the current workspace through the backend's `POST /load` endpoint.
- Displays model answers, references, progress messages, and API errors in the chat response.
- Uses `media/aristotle.png` as the chat participant icon.

## Use case and benefit

The extension keeps codebase questions inside the editor. Instead of switching to a separate chat window and manually copying files, a developer can ask about an indexed repository, attach the relevant files to a prompt, or load the current workspace into Aristotle and continue the conversation in VS Code.

The extension is intentionally thin: parsing, indexing, retrieval, model orchestration, and answer generation remain in the backend. This keeps the editor integration small and lets the same backend serve other clients later.

## How it works

```
VS Code Chat (@aristotle)
          |
          +-- prompt + history + referenced files -- POST /chat --> Aristotle backend
          |
          +-- workspace .py/.md files ------------ POST /load --> Aristotle backend
          |
          v
   streamed progress and final Markdown answer
```

When a user asks a normal question, the chat handler converts VS Code history into the backend's `HUMAN_MESSAGE`/`AI_MESSAGE` format, reads eligible file references, and sends a JSON request. The backend performs retrieval and returns a response plus references; the extension renders both in the Chat view.

For workspace loading, the handler scans the first open workspace for `.py` and `.md` files, skips `node_modules` and `.venv`, and uploads files concurrently. The backend is responsible for parsing and indexing them.

## Quick start

### Requirements

- VS Code `^1.105.0`
- Node.js and [pnpm](https://pnpm.io/)
- A running Aristotle backend, normally at `http://localhost:8000`

### Install dependencies and build

From this repository:

```
pnpm install
pnpm run compile
```

To develop the extension, open this folder in VS Code and press `F5` (or use **Run Extension**) to launch an Extension Development Host. Open the Chat view and address a prompt to `@aristotle`.

The extension defaults to `http://localhost:8000`. If the backend is elsewhere, set this value in VS Code settings:

```json
{
  "aristotle.api_base_url": "http://localhost:8000"
}
```

To load the current workspace, use the workspace-loading command exposed by the current handler:

```text
@aristotle /load_cwd my-project
```

The package metadata currently shows `/loadWorkspace` as the sample command name; `/load_cwd` is the token recognized by the implementation.

## Project layout

```
src/extension.ts       Activation, command registration, and chat participant setup
src/handlers/chat.ts   Chat requests, history conversion, and attached-file context
src/handlers/workspace.ts
                       Workspace discovery and file uploads
src/services/api.ts    HTTP client for /chat and /load
src/types/api.ts       Backend request/response types
media/                 Extension artwork
esbuild.js             Development and production bundling
```

## Available scripts

```text
pnpm run compile        Type-check, lint, and build the extension
pnpm run package        Production build
pnpm run check-types    TypeScript checks only
pnpm run lint           ESLint
pnpm run test           VS Code extension tests
```

## Development notes

This repository contains the VS Code client only; it does not include the model, graph database, vector index, or backend retrieval pipeline. See the backend README for Ollama, Neo4j, FAISS, and model configuration.
