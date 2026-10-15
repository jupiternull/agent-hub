# Agent Hub

A local agent orchestration prototype with a dark React workspace, streaming chat, connector setup, and a multi-agent task board.

## Run

Requires Node.js 20.19+ and npm. From this directory:

```sh
npm install && npm run dev
```

Open http://localhost:5173. Vite proxies `/api` to the Express server on `127.0.0.1:8787`. Both processes start together with concurrently. Stop with Ctrl+C.

```sh
npm run build
```

Runs strict TypeScript checks for frontend and backend and produces the frontend in `dist/`. `npm start` runs only the API; this prototype uses Vite for the web UI.

## Connect agents

The four seeded agents are labeled DEMO and always produce `[mock]` output. Use **Add agent** to create a real connector. Incomplete HTTP/webhook connectors also return labeled mock replies.

- **OpenAI-compatible HTTP:** enter the API root (including `/v1` if required), model, and key. The server appends `/chat/completions` and submits the current prompt. No real keys ship with this project.
- **Local CLI:** provide a shell command such as `codex exec {prompt}`. Install and authenticate the CLI separately. Each `{prompt}` is replaced by a POSIX single-quoted, escaped prompt. If the placeholder is missing, the quoted prompt is appended. Commands execute from this project directory, with stdin closed, stdout streamed through SSE, and a 120-second timeout. Diagnostic output is hidden to avoid exposing credentials. CLI templates are trusted local shell code.
- **Webhook:** receives a JSON POST `{ "prompt": "…", "agentId": "…" }`. Return JSON with `reply` or `text`, or plain text. Other JSON is displayed as formatted output.

HTTP/webhook calls have a 120-second timeout. Their activity is streamed while awaiting the response; CLI stdout streams as it arrives. Connector failures appear in the conversation. A disconnected client terminates its CLI process group. Chat submits one prompt per request; earlier messages are visible in the UI but are not sent as model history.

## Workspace

Select an agent for chat. The right pane shows connector information, session counts, on-demand routines that fill the composer, and live activity. **Orchestrate** lets you select agents and broadcast one task concurrently. The task board moves through queued, running, and done; unsuccessful tasks are marked failed in the final lane. Responses appear in side-by-side columns. Conversations, activity, and tasks live in browser memory and reset on refresh. Agent connections persist across restarts.

## Local data and security

Connections are saved in `data/agents.json` on first startup. This file is gitignored and created with owner-only permissions. API responses mask keys; keys and connector responses are never written to server logs. Keys are stored as local plaintext, not encrypted. Do not commit or share this file. Remove it while the API is stopped to reset the demo agents.

There is no authentication or database. The API binds to loopback. Vite listens on all interfaces for development previews; keep the development server on a trusted machine/network because the proxy provides access to local connector execution. Do not expose it publicly. Agent output renders as plain text, never HTML. The UI uses optional Google Fonts with local font fallbacks; no other external assets are needed.
