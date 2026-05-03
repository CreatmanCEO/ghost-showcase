# 👻 GHOST — General Helper & Overlay Smart Tool

[![License: Commercial](https://img.shields.io/badge/license-Commercial-9d6cff)](LICENSE)
[![Stars](https://img.shields.io/github/stars/CreatmanCEO/ghost-showcase?style=flat&color=yellow)](https://github.com/CreatmanCEO/ghost-showcase/stargazers)
[![Validate](https://github.com/CreatmanCEO/ghost-showcase/actions/workflows/validate.yml/badge.svg)](https://github.com/CreatmanCEO/ghost-showcase/actions/workflows/validate.yml)
[![Status](https://img.shields.io/badge/status-Phase%201%20%C2%B7%20active%20development-22c55e)](#project-status)
[![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4?logo=windows&logoColor=white)](#limitations)
[![Open to discussion](https://img.shields.io/badge/open%20to-investment%20%C2%B7%20partnership%20%C2%B7%20acquisition-cc785c)](#open-to-discussion)

🇬🇧 English · [🇷🇺 Русский](README.ru.md)

> **Real-time AI assistance that stays invisible during screen recordings and screen sharing — for interviews, meetings, coding sessions, and learning.** Phase 1 in active development. Author is open to investment / partnership / acquisition.

---

## Project trajectory

GHOST started as a personal alternative to expensive paid tools — built by analysing what each competitor does well and what each gets wrong, then combining the best parts in one product. After two months of focused development, two paths are now on the table, and the author is open to either:

- **Path A — owner-led growth** with investment, a co-founder / technical partner, or a small team to handle commercial side (the author's strength is engineering and product, not go-to-market)
- **Path B — strategic acquisition** by a republisher or a larger player who already has the distribution and commercial pipeline

The project is **not yet ready** for either path. This repository is the public showcase of the design, architecture, and engineering progress to date — and an honest map of what is needed to ship V1.0. See [What's needed to ship](#whats-needed-to-ship) below.

If either path is interesting to you, the [Open to discussion](#open-to-discussion) section has the contact channels.

---

## Why GHOST?

The AI-overlay assistant space is crowded. The author analysed every meaningful competitor and built GHOST around the gaps:

| Competitor | Strength taken | Weakness avoided |
|---|---|---|
| **Cluely** ($75/mo) | Multi-mode (interviews / meetings / exams) — clear use-case framing | Stealth-only, no offline option, expensive for what it offers |
| **Screenpipe** (16K stars) | Plugin architecture, MCP server, 24/7 capture, full-text search | No invisible-overlay capability, captures continuously by default (privacy concern) |
| **Natively** (open-source) | RAG with local vector DB, multi-provider LLM, WASAPI capture | Less polished UX, fewer modes, no streaming-first pipeline |
| **Interview Hunter** | Free tier, low entry barrier, CV upload for personalised answers | Single-purpose, limited to interviews |
| **Interview Coder** | Step-by-step problem solving, code-from-screen recognition | Niche, no general assistant capability |
| **Final Round AI** | Mock-interview / preparation mode | Closed ecosystem, no general assistant capability |

**GHOST combines: stealth + multi-mode + multi-provider LLM + RAG + offline-capable + streaming-first response pipeline.** No competitor today combines all six.

### Critical industry problems we are solving

1. **10–30 second response delay** — most tools wait for the full question, then for the full LLM response. Killer of usability during a live interview.
2. **Generic, ChatGPT-style answers** — without RAG context (CV, job description, company notes) the assistant sounds nothing like the user.
3. **Detection despite "stealth" claims** — many competitors' stealth fails silently in modern Zoom / Teams updates.
4. **Cloud-only LLM dependency** — privacy-conscious users want a local-LLM option for sensitive contexts.

GHOST's answers to each are described in [Architecture](#architecture) and the [Latency target](#latency-target) section.

---

## Architecture

![Two-process architecture: invisible Electron overlay (renderer + main with WDA_EXCLUDEFROMCAPTURE via koffi) ↔ WebSocket JSON-RPC port 9876 ↔ Python asyncio sidecar (Silero VAD, Deepgram or faster-whisper STT, litellm router for Claude/GPT/Ollama/LM Studio, ChromaDB RAG)](docs/architecture.svg)

Two-process design with a clean WebSocket boundary:

| Layer | Stack | Responsibility |
|---|---|---|
| Electron renderer | React 19 · Vite · Tailwind · Zustand | Invisible overlay UI · streaming token rendering · session view · 5 modes |
| Electron main | Node 20+ · `koffi` · Win32 API | `SetWindowDisplayAffinity` → `WDA_EXCLUDEFROMCAPTURE` · global hotkeys · system tray · process supervisor |
| Transport | WebSocket JSON-RPC (port 9876) | Decouples UI from inference. Reconnection-resilient, schema-versioned. |
| Python sidecar | Python 3.11+ · `asyncio` · `websockets` | EventBus · CaptureManager · Silero VAD · STT · LLM router · RAG |
| Audio capture | `pyaudiowpatch` (WASAPI loopback) + microphone | Dual-source — captures system audio (the other party in a call) and microphone simultaneously |
| Speech detection | `silero-vad` | Detects speech onset within ~200 ms — drives streaming pipeline |
| STT | `deepgram-sdk` (cloud) → `faster-whisper` (local fallback) | Streaming partial transcripts; cloud is faster, local is private |
| LLM | `litellm` Router → Anthropic Claude · OpenAI GPT · Ollama · LM Studio | Streaming generation begins on partial transcripts — answer first words appear before user finishes asking |
| RAG | `chromadb` (embedded, local) | Per-session document upload (CV, JD, notes) → contextual answers |
| Storage | SQLite (sessions / transcripts / settings) + encrypted `.env` (API keys) | Local-only by default, no telemetry |

For deeper detail see [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

### The invisible overlay — core technical innovation

```typescript
// Electron main process — Windows-specific
import koffi from 'koffi';

const user32 = koffi.load('user32.dll');
const SetWindowDisplayAffinity = user32.func('SetWindowDisplayAffinity', 'bool', ['void *', 'uint32']);

const WDA_EXCLUDEFROMCAPTURE = 0x00000011;  // Windows 10 v2004+
SetWindowDisplayAffinity(mainWindow.getNativeWindowHandle(), WDA_EXCLUDEFROMCAPTURE);
```

The result: the overlay is rendered to the user's eyes but is **excluded by the OS** from screen-recording APIs, including those used by Zoom / Teams / Google Meet / Discord / OBS / native screenshot tools. Tested on Windows 10 v2004+ and Windows 11.

---

## Latency target

> **First-reply-tokens within 1–2 seconds of the user finishing their question.** This is a target, not yet a measured benchmark. Phase 1 builds the streaming pipeline that makes it achievable.

Pipeline:

```
User starts speaking
    ↓
Silero VAD detects speech onset (~200 ms)
    ↓
Deepgram (or faster-whisper) streams PARTIAL transcripts as words are recognised
    ↓
LLM begins generation on the PARTIAL transcript — does not wait for end-of-utterance
    ↓
LLM tokens stream into the overlay character-by-character
    ↓
User sees the first words of the answer ~1–2 s after they stop talking
```

Why competitors fail this — most wait for the full question end, then send a single non-streaming LLM call, then await the full response, then render. The cumulative delay is 10–30 seconds.

---

## Project status

**Phase 1 — Core (active development).** Concrete progress:

| Module | Status |
|---|---|
| Invisible overlay via `koffi` + Win32 `WDA_EXCLUDEFROMCAPTURE` | ✅ |
| Electron 33 + React 19 + Vite + Tailwind shell | ✅ |
| Global hotkeys (`Ctrl+Shift+G/V/A/M/Esc`) | ✅ |
| WASAPI loopback (system audio) + microphone — dual-source | ✅ |
| Audio device auto-detection (sample rate, channels) | ✅ |
| Python sidecar with `asyncio` WebSocket on port 9876 | ✅ |
| EventBus with thread-safe locking | ✅ |
| CaptureManager integrated with start / stop capture | ✅ |
| Capture-mode selector in UI | ✅ |
| Silero VAD integration | 🔨 |
| Streaming Deepgram → streaming LLM → streaming overlay tokens | 🔨 |
| Multi-document RAG with ChromaDB | 🔨 |
| Custom hotkey configuration UI | 🔨 |
| Performance optimisation (target < 10% CPU at idle) | 🔨 |

**Phase 2 — V1.0 ship requirements.** See [What's needed to ship](#whats-needed-to-ship).

**Phase 3 — Beyond V1.0.** macOS support · Linux support · mobile companion · team / enterprise features.

---

## What's needed to ship

Honest gap analysis for V1.0. This is what an investor / partner / acquirer needs to evaluate.

### Engineering gaps

- **macOS / Linux ports** — `WDA_EXCLUDEFROMCAPTURE` is Windows-only. Equivalents on macOS (`NSWindowSharingNone`) and X11 / Wayland exist but require platform-specific work.
- **Code signing & notarisation** — Windows code signing (Authenticode), macOS notarisation (when Phase 3 lands). Without this, SmartScreen / Gatekeeper warnings on every install.
- **Auto-update infrastructure** — `electron-updater` wired to a release server (S3 + signed manifests).
- **Distributable installer** — Inno Setup or NSIS for a single `.exe` install path with proper uninstaller.
- **Crash reporting** — Sentry or self-hosted alternative for production diagnostics. Opt-in by default.
- **Telemetry** — usage analytics (opt-in only) for product decisions. Strictly anonymised.

### Product gaps

- **Onboarding flow polish** — first-launch UX, API key wizard, voice-test, mode selection.
- **Pricing page & marketing site** — currently no public-facing site exists.
- **Payment integration** — Stripe / Paddle / Lemon Squeezy. License key validation on startup.
- **Trial / freemium logic** — N-day trial vs free local-LLM-only tier.
- **Support documentation** — user-facing docs, FAQ, troubleshooting guide.

### Business gaps

- **Legal review** — Terms of Service, Privacy Policy, GDPR / CCPA compliance audit, EULA for the binary.
- **Trademark filing** — "GHOST" name protection in core jurisdictions.
- **Customer support ops** — at minimum a triaged inbox + Discord / Telegram community.
- **Marketing strategy** — positioning vs Cluely / Screenpipe in target channels (HN, Reddit, dev YouTube).

### Investment thesis (3-line summary)

- **Market** — AI assistant tools, growing fast; competitor pricing $20–$150/mo signals real willingness to pay.
- **Gap** — no current product combines stealth + multi-mode + multi-provider + RAG + offline-capable; closest is Cluely at $75/mo for stealth alone.
- **Why now** — LLM cost decline + viable local LLMs (Ollama / LM Studio) + Win32 stealth API maturity make this product feasible at indie scale.

---

## Features

### 🎯 Five specialised modes

Each mode is an opinionated prompt + UI affordance, not a separate model:

1. **Interview** — algorithm hints, complexity analysis, syntax reminders. Tuned for live coding interviews.
2. **Meeting** — talking points, action-item capture, context across topics.
3. **Coding** — documentation lookup, code-pattern suggestions, debugging help.
4. **Learning** — concept explanations, related resources, interactive Q&A.
5. **General** — flexible default for anything else.

### 🤖 Multi-provider LLM

Choose per-mode or globally. All routed through `litellm`:

- **Anthropic Claude** — primary recommendation (Sonnet / Opus / Haiku)
- **OpenAI GPT** — alternative (GPT-4o, GPT-4 Turbo, GPT-3.5 Turbo)
- **Ollama** (local) — `llama3`, `mistral`, `codellama`, `phi-3` and others
- **LM Studio** (local) — any GGUF model with OpenAI-compatible API

### 🎤 Real-time STT

- **Deepgram** (cloud) — ultra-low latency streaming, 95%+ word accuracy
- **faster-whisper** (local) — offline-capable, slower, fully private
- Push-to-talk and continuous (VAD-driven) modes
- Multi-language support

### 📚 RAG document support

Upload PDFs / DOCX / TXT / MD per session — typically a CV, job description, company notes for an interview, or a meeting agenda. Documents are chunked, embedded, and stored in a local ChromaDB collection. Retrieval is automatic during answer generation.

### 🔒 Privacy & security

- **Offline mode** — Ollama + faster-whisper = 100% local, no network calls
- **No data storage by default** — sessions can be saved on demand
- **Encrypted config** — API keys live in `.env`, optionally OS-keychain-protected
- **Open architecture** — every component documented, no opaque cloud dependency

For the full feature catalogue see [`docs/FEATURES.md`](docs/FEATURES.md).

---

## How it's built

GHOST is built solo by the author using a structured **multi-agent development workflow** on top of [Claude Code](https://code.claude.com). The pattern:

| Agent role | Responsibility |
|---|---|
| **Orchestrator** | Plans tasks, distributes to coders, tracks progress, handles integration |
| **Coder × N** | Implements independent modules in parallel — TDD, small commits, feature branches |
| **Tester** | Runs the full test suite after every coder completes — catches regressions early |
| **Documentation** | Updates design docs and changelog in the same session as the code change |

This pattern is the same one the author maintains as a standalone tool in [`claude-code-antiregression-setup`](https://github.com/CreatmanCEO/claude-code-antiregression-setup) — featured on Habr (top-5 of the day, 20K reads, Технотекст 8 entry). For an investor / acquirer, this is the answer to "how does a solo founder ship complex software": with documented, repeatable process, not by force.

The full Claude Code-related toolset by the same author is listed in [Related](#related).

---

## Tech stack

### Frontend
- **Electron 33+** — desktop framework, native Windows API access via `koffi`
- **React 19** — UI components and state
- **TypeScript** — type-safe development
- **Vite** — fast dev server and build
- **Tailwind CSS** — styling
- **Zustand** — state management

### Backend (Python sidecar)
- **Python 3.11+** — core logic
- **`asyncio` + `websockets`** — sidecar server
- **`pyaudiowpatch`** — WASAPI loopback capture
- **`silero-vad`** — voice activity detection
- **`litellm`** — multi-provider LLM router
- **`deepgram-sdk`** — cloud STT
- **`faster-whisper`** — local STT
- **`chromadb`** — embedded vector DB

### AI providers
- **Anthropic Claude** — primary LLM
- **OpenAI GPT** — alternative LLM
- **Ollama / LM Studio** — local LLM hosting
- **Deepgram** — real-time cloud STT

### Infrastructure
- **SQLite** — local data
- **ChromaDB** — vector DB for RAG
- **Inno Setup / electron-builder** — Windows packaging (planned for V1.0)

---

## Use cases

### 1. Technical interviews

**Scenario:** software-engineering interview with live coding.

GHOST suggests algorithms and data structures, provides complexity-analysis hints, reminds you of syntax. Invisible to interviewers during screen sharing.

**Mode:** Interview.

### 2. Online meetings

**Scenario:** product demo or client presentation.

Quick access to talking points, real-time fact lookup, context management across topics. Professional appearance — no visible assistant.

**Mode:** Meeting.

### 3. Coding sessions

**Scenario:** building a complex feature.

Documentation lookup, code-pattern suggestions, bug investigation, architecture discussion.

**Mode:** Coding.

### 4. Learning & research

**Scenario:** following a technical tutorial or reading a paper.

Concept explanations, related resources, troubleshooting help, interactive Q&A.

**Mode:** Learning.

---

## Configuration example

```jsonc
{
  "llm": {
    "provider": "claude",
    "model": "claude-3-5-sonnet-20241022",
    "temperature": 0.7,
    "max_tokens": 2000
  },
  "stt": {
    "provider": "deepgram",
    "language": "en",
    "mode": "vad-driven"
  },
  "ui": {
    "opacity": 0.95,
    "position": "top-right",
    "theme": "dark"
  },
  "modes": {
    "default": "general",
    "interview": {
      "system_prompt": "You are a technical interview assistant..."
    }
  }
}
```

Full schema: [`examples/config.example.json`](examples/config.example.json).

---

## Limitations

- **Windows 10 v2004+ only.** `WDA_EXCLUDEFROMCAPTURE` is Windows-specific. macOS (`NSWindowSharingNone`) and Linux (Wayland / X11) ports are on the roadmap, not yet built.
- **Phase 1 — pre-V1.0.** Streaming pipeline, multi-document RAG, onboarding polish, installer — all in progress.
- **API keys required for cloud providers** (Deepgram / Claude / GPT). 100% offline mode requires Ollama or LM Studio installed locally — no credit-card-required path until V1.0 includes a free-tier option.
- **Screen-capture exclusion is OS-level, not magical.** It works against the standard Windows Desktop Duplication API and DirectX capture paths used by mainstream meeting / recording software. A determined adversary using kernel-level or external hardware capture is out of scope.
- **No mobile companion app yet.** Telegram-bot bridge exploration is a possible Phase 3 direction.

---

## Open to discussion

This project is in active development. The author is open to:

- **Investment** for owner-led growth — sole founder, looking for a business / commercial co-founder or seed funding to bring V1.0 to market
- **Strategic acquisition or republishing** — if you have an existing distribution channel and want to acquire / license the technology
- **Technical co-founder or partnership** — for go-to-market, ops, or business-side execution

**Contact:**

- 📧 Email: **creatmanick@gmail.com**
- 💬 Telegram: **[@Creatman_it](https://t.me/Creatman_it)**
- 🌐 Website: **[creatman.site](https://creatman.site)**
- 🐙 GitHub: **[@CreatmanCEO](https://github.com/CreatmanCEO)**

---

## Related — Claude Code ecosystem by the same author

GHOST is built using a Claude Code-centric workflow. The author maintains a public toolset that supports this discipline:

- [`claude-code-antiregression-setup`](https://github.com/CreatmanCEO/claude-code-antiregression-setup) — `CLAUDE.md` + subagents + hooks pattern that keeps refactors from breaking working code. Featured on [Habr](https://habr.com/ru/articles/1013330/) (top-5 of the day, 20K reads).
- [`ai-context-hierarchy`](https://github.com/CreatmanCEO/ai-context-hierarchy) — three-level context system. Featured in the [Graphify v5.0 roadmap](https://github.com/safishamsi/graphify/issues/425).
- [`claude-statusline`](https://github.com/CreatmanCEO/claude-statusline) — statusline for Claude Code with VPS health monitoring. Featured on [Habr](https://habr.com/ru/articles/1013414/).
- [`notebooklm-claude-workflows`](https://github.com/CreatmanCEO/notebooklm-claude-workflows) — seven slash-commands for Google NotebookLM research pipelines.
- [`lingua-companion`](https://github.com/CreatmanCEO/lingua-companion) — voice-first English tutor for Russian-speaking IT professionals. Different domain, same engineering discipline.
- [`diabot`](https://github.com/CreatmanCEO/diabot) — non-commercial Telegram bot for type 1 diabetes (food photo → carb counting). Same author.

---

## Author

**Nick Podolyak** — Python developer and digital architect at [CREATMAN](https://creatman.site)

- GitHub: [@CreatmanCEO](https://github.com/CreatmanCEO)
- Habr: [creatman](https://habr.com/ru/users/creatman/)
- dev.to: [@creatman](https://dev.to/creatman)

---

## License

**Commercial — All rights reserved.** This showcase repository is for demonstration, portfolio, and discussion purposes. The actual GHOST application and its source code are proprietary and not included in this repository. See [`LICENSE`](LICENSE).

For licensing inquiries — investment, partnership, or acquisition — see [Open to discussion](#open-to-discussion).

---

<sub>Built with Electron · React · TypeScript · Python · Claude · Deepgram · ChromaDB</sub>
