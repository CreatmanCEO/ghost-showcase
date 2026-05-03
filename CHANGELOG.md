# Changelog

Public-facing milestones for the GHOST project. Internal commit-level history is private.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) · [SemVer](https://semver.org/spec/v2.0.0.html).

## [Unreleased] — Phase 1, in progress

### Working on
- Silero VAD integration into the streaming pipeline
- Streaming Deepgram → streaming LLM → streaming overlay-token rendering
- Multi-document RAG with ChromaDB
- Custom hotkey configuration UI
- Performance optimisation (target < 10% CPU at idle)

## [Showcase 0.2.0] — 2026-05-03

### Changed (showcase repository only — product code is private)
- README repositioned from generic "commercial product showcase" to **fundraising / exit-prep document**: explicit Project trajectory section spelling out the two paths (owner-led growth with investment / partnership, or strategic acquisition / republishing)
- Added `Why GHOST?` section with concrete competitor analysis — Cluely, Screenpipe, Natively, Interview Hunter, Interview Coder, Final Round AI
- Added `Latency target` section explaining the streaming pipeline rationale (VAD onset → streaming partial transcripts → LLM begins on partials → token-by-token overlay output → first reply tokens within 1–2 s of question end)
- Added `What's needed to ship` section — honest engineering / product / business gap analysis to bring the project to V1.0
- Added `Investment thesis` summary
- Added `How it's built` section — multi-agent dev workflow (Orchestrator + Coders + Tester + Documentation Agent) with cross-link to the same author's `claude-code-antiregression-setup`
- Added `Open to discussion` section — explicit dual CTA for investment / partnership / acquisition
- Added `Limitations` section — Windows-only, beta-stage, OS-level capture exclusion is not magic
- Added `Related — Claude Code ecosystem` section cross-linking to all six sister repos by the same author
- `Project status` rewrites the previous abstract "In Progress / Planned" into a concrete Phase-1 checklist with what is implemented vs in progress
- Author signature expanded: full name, Habr / dev.to profile links

### Added
- `docs/architecture.svg` — hand-rendered system diagram showing the actual stack (Electron 33 + React 19 + Vite + koffi for Win32 / WebSocket JSON-RPC port 9876 / Python 3.11 asyncio sidecar with Silero VAD + pyaudiowpatch + deepgram-sdk + faster-whisper + litellm Router + ChromaDB)
- `README.ru.md` — full Russian mirror with the same structure, badges, and CTAs
- `CHANGELOG.md` (this file)
- `.github/workflows/validate.yml` — CI checking LICENSE and CHANGELOG exist, every `docs/*` asset referenced from README exists, internal Markdown links resolve, JSON in `examples/config.example.json` is parseable, SVG is well-formed XML
- "Stars" and "Validate CI" badges; Status / Platform / "Open to discussion" badges added; old static badges retired

### Removed / fixed
- Removed broken README references to `docs/TECH_STACK.md` and `docs/USE_CASES.md` (those files never existed in this repository) — content surfaced inline in the new README structure
- Stale architecture description (called the backend "FastAPI") replaced with the actual stack: `asyncio` + `websockets` JSON-RPC sidecar
- Stale Electron / React versions (claimed 32 / 18) updated to actual versions used (33 / 19)

### Notes
- This release affects only the **showcase repository**. The actual GHOST application source code remains private. The license stays Commercial — All rights reserved.
- Topics on GitHub applied separately via `gh api` after merge.

## [Showcase 0.1.0] — 2026-02-26

### Added (initial showcase publication)
- `README.md` with overview, features, architecture (ASCII), tech stack, use cases, project status, configuration example, "Why Commercial?" rationale
- `LICENSE` — Commercial, all rights reserved
- `docs/ARCHITECTURE.md` — system design deep dive
- `docs/FEATURES.md` — full feature list with implementation details
- `examples/config.example.json` — configuration schema example
