# kilroy.starter.agent

Multi-class Kilroy app starter template with conversation UI (chat agent).

Use this as the starting point when your app needs per-session conversation interfaces,
push messaging, swarm integration, or AI backend connectivity.

---

## App Pattern

This app uses the **agent pattern**: a `manager` pdClass that orchestrates the app
lifecycle, plus a `chat_agent` pdClass that can be instantiated multiple times — each
instance runs its own process diagram and renders a configurable chat UI in an iframe.

**Choose this pattern when your app needs to:**
- Present a conversation-style UI (message history, user/bot bubbles) to users
- Support multiple independent chat sessions with isolated state per session
- Route user messages through backend workflows (AI processing, tool calls, swarms)
- Allow the backend to push messages to a specific agent's chat window
- Integrate with AI services, pipelines, or swarm-orchestrated agents

**Choose a different pattern if you need:**
- No iframe UI \u2192 use `kilroy.starter.simple`
- Per-instance panels without a chat interface \u2192 use `kilroy.starter.widget`

---

## Structure

```
kilroy.starter.agent/
  manifest.json                     — app identity, pdClass definitions
  process_diagrams/
    manager/
      diagram.json                  — manager: lifecycle + create/delete agent
    chat_agent/
      diagram.json                  — agent: lifecycle, config, resize, swarm, chat
  workflows/
    manager/
      init_hook/payload/
      resume_hook/payload/
      pause_hook/payload/
      stop_hook/payload/
      new_chat_agent/payload/       — interactive: collect config, launch agent
      create_chat_agent/payload/    — server-side: start_app_diagram
      delete_chat_agent/payload/    — server-side: stop_app_diagram
      new_wf_agent/payload/         — create a wf_agent from the AI palette
    chat_agent/
      init_hook/payload/
      resume_hook/payload/
      pause_hook/payload/
      stop_hook/payload/
      set_config/payload/           — applies full config to agent context vars
      set_style/payload/            — applies theme style
      ui_resize_hook/payload/       — handles iframe resize from platform chrome
      ui_settings/payload/          — interactive: edit agent config inline
      swarm_hook/payload/           — receives swarm messages
      publish_swarm_hook/payload/   — publishes to a send swarm
      chat_hook/payload/            — receives chat messages from the UI
  public/
    chat_agent.html                 — iframe UI entry point
    js/chat_agent.js                — frontend logic (PubSub, markdown, bubbles)
    css/markdown.css                — full dark/light design system
    ai/js/palette_defs.js           — AI palette block defaults
    info.html
    ABOUT.md  HELP.md  RELEASE_NOTES.md  images/logo.png
```

Each `payload/` directory contains three files: `json.json`, `xml.xml`, `script.js`.

---

## Chat UI Features

- Dark mode (default) and light mode, controlled by `wf_webhook_args.darkmode`
- Markdown rendering with syntax-highlighted code blocks (highlight.js)
- Bubble-style messages — user messages in blue, bot messages in neutral gray
- Resize support — the iframe resizes when the platform window is dragged
- Swarm subscriptions — the agent can join swarms to receive/send messages

---

## Customizing This Template

1. **Replace the app id** — change every instance of `kilroy.starter.agent` to
   your new app id. This must match:
   - `manifest.json` → `id`, every `pdAlias`
   - Every `action_workflow` path in both `diagram.json` files
   - Every `name` field in all `workflows/.../payload/json.json` files
   - The directory name under `platform_data/demo/apps/`

2. **Update manifest metadata** — set `displayName`, `desc`, `author`.

3. **Rename the agent pdClass** if desired (e.g. `assistant`, `support_agent`) —
   update `manifest.json`, both `diagram.json` files, all workflow paths, and the
   `pdAlias` references in `create_chat_agent` / `delete_chat_agent` workflows.

4. **Build your chat backend** — edit `chat_hook` to process user messages and
   publish responses back via swarm or direct PubSub publish.

5. **Customize the UI** — edit `public/chat_agent.html`, `public/js/chat_agent.js`,
   and `public/css/markdown.css`. The iframe receives config via URL params injected
   by the `set_config` workflow.

6. **Replace assets** — update `public/images/logo.png`, `info.html`, `ABOUT.md`.

---

## Client/Server Communication

The iframe UI communicates with the backend via:
- **Webhooks** — HTTP POST to `/api/v1/pd/webhook/API_TOKEN/${alias}/<hook_name>`
- **PubSub** — the Faye-based `PubSubClient.js` library receives server-pushed messages

The alias is passed to the iframe as a URL search param (`?alias=...`) by the platform.

---

## Reference

- Pattern guide: `kilroy.sandbox/docs/02-apps/pattern-agent-app.md`
- App structure reference: `kilroy.sandbox/docs/02-apps/app-structure-reference.md`
- Workflow authoring: `kilroy.sandbox/copilot-notes/kilroy-workflow-reference.md`
