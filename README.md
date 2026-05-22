# Lab 6 — Hermann's Project Portfolio

Two live projects: a biblical AI chatbot and a Raspberry Pi phone control system.

---

## Projects

| Project | Stack | Live |
|---------|-------|------|
| [JesusGPT](./jesusgpt/) | React + Express + Groq + Gemini | [jesus-gpt.vercel.app](https://jesus-gpt.vercel.app) |
| [Pi Cockpit](./pi-cockpit/) | Flask + uinput + SSE | `http://<pi-ip>:5000` |

---

## JesusGPT

Biblical AI chatbot. Responds as Jesus Christ drawing from Gospel RAG (retrieval-augmented generation). Full conversation memory, multi-language, direct and parable modes.

**Features**
- Speaks in first person as Jesus — not "in the style of"
- Groq (llama-3.3-70b) primary, Gemini 2.0 Flash fallback
- RAG over 4 Gospels in EN / ES / LA / GR
- Conversation history (multi-turn dialogue)
- Verse references link to Bible Gateway
- TTS read-aloud via browser speech API
- ChatGPT-style chat layout, auto-resize input

**Deploy → Vercel**
https://jesus-gpt.vercel.app/
See [jesusgpt/DEPLOYMENT.md](./jesusgpt/DEPLOYMENT.md)

---

## Pi Cockpit

Unified phone control server for Raspberry Pi 3. One URL on your phone controls everything.

**Features**
- Boot mode selector: CLAUDE DEV / GAMING / DESKTOP / HEADLESS
- Phone keyboard + mouse via `python-uinput` (kernel injection)
- Claude Code terminal streamed live to phone (SSE)
- Pegasus gamepad (D-pad + ABXY) for RetroArch / Pegasus frontend
- Git dashboard — view commits, diffs, pull
- CLAUDE.md editor from phone
- System health: temp, RAM, disk, throttle, uptime
- Power controls: reboot / shutdown (hold 0.9s)
- Runs as systemd service on boot

**Deploy → Raspberry Pi**

See [pi-cockpit/DEPLOYMENT.md](./pi-cockpit/DEPLOYMENT.md)
