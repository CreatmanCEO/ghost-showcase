# GHOST — AI Assistant with Invisible Overlay

> **Real-time AI assistance that stays invisible during screen recordings and screenshots**

![Status](https://img.shields.io/badge/Status-In%20Development-yellow?style=flat-square)
![Commercial](https://img.shields.io/badge/Product-Commercial-blue?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-Windows-0078D4?style=flat-square&logo=windows)

---

## Overview

**GHOST** is an intelligent AI assistant designed for professionals who need real-time support during interviews, meetings, coding sessions, and learning. Unlike traditional assistant tools, GHOST features an **invisible overlay** that won't appear in screen recordings or screenshots, making it perfect for online interviews and presentations.

### Key Innovation: Invisible Technology

Built with Windows' `WDA_EXCLUDEFROMCAPTURE` flag, GHOST's overlay is completely invisible to screen capture software while remaining fully visible to you. This breakthrough makes it the ideal companion for:
- **Technical interviews** — Get coding hints without revealing the assistant
- **Online meetings** — Access notes and context while presenting
- **Coding sessions** — Real-time code suggestions and documentation
- **Learning** — Interactive assistance while following tutorials

---

## Features

### 🎯 5 Specialized Modes

Each mode is optimized for specific use cases with tailored prompts and behaviors:

1. **Interview Mode** — Technical interview support with coding hints and algorithm suggestions
2. **Meeting Mode** — Context management and talking points for professional meetings
3. **Coding Mode** — Real-time code analysis, suggestions, and documentation lookup
4. **Learning Mode** — Interactive explanations and concept clarification
5. **General Mode** — Flexible assistance for any task

### 🤖 Multi-Provider LLM Support

Choose the best model for your needs:
- **Claude** (Anthropic) — Primary recommendation for best results
- **GPT** (OpenAI) — Alternative for specific use cases
- **Ollama** — Run models locally for complete privacy
- **LM Studio** — Local model hosting with custom configurations

### 🎤 Real-Time Speech-to-Text

- **Cloud**: Deepgram (ultra-low latency streaming)
- **Local**: faster-whisper (offline capability)
- Push-to-talk and continuous modes
- Multi-language support

### 📚 RAG Document Support

- Upload PDFs, DOCX, TXT files
- Context-aware responses based on your documents
- Perfect for interview prep with company materials
- Automatic document chunking and embedding

### 🔒 Privacy & Security

- **Offline Mode** — Works completely offline with local LLMs
- **No Data Storage** — Conversations aren't saved by default
- **Encrypted Config** — API keys stored securely
- **Open Architecture** — Transparent operation

---

## Architecture

```
┌─────────────────────────────────────────────┐
│         Electron (TypeScript + React)       │
│  ┌─────────────────────────────────────┐   │
│  │   UI Layer (Invisible Overlay)      │   │
│  │   • WDA_EXCLUDEFROMCAPTURE          │   │
│  │   • Transparent + Always-on-top     │   │
│  │   • Draggable, resizable            │   │
│  └─────────────────────────────────────┘   │
│                    │                        │
│                    ▼                        │
│  ┌─────────────────────────────────────┐   │
│  │   IPC Bridge (WebSocket-like)       │   │
│  └─────────────────────────────────────┘   │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│          Python Backend (FastAPI)           │
│  ┌─────────────────────────────────────┐   │
│  │   LLM Router                        │   │
│  │   • Claude API                      │   │
│  │   • OpenAI API                      │   │
│  │   • Ollama (local)                  │   │
│  │   • LM Studio (local)               │   │
│  └─────────────────────────────────────┘   │
│  ┌─────────────────────────────────────┐   │
│  │   STT Engine                        │   │
│  │   • Deepgram (cloud)                │   │
│  │   • faster-whisper (local)          │   │
│  └─────────────────────────────────────┘   │
│  ┌─────────────────────────────────────┐   │
│  │   RAG System                        │   │
│  │   • Document processing             │   │
│  │   • Vector embeddings               │   │
│  │   • Context retrieval               │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed technical documentation.

---

## Tech Stack

### Frontend
- **Electron** — Desktop application framework
- **React** — UI components and state management
- **TypeScript** — Type-safe development
- **Tailwind CSS** — Styling and responsive design

### Backend
- **Python 3.11+** — Core logic and integrations
- **FastAPI** — High-performance async API
- **WebSocket** — Real-time bidirectional communication

### AI & ML
- **Anthropic Claude** — Primary LLM
- **OpenAI GPT** — Alternative LLM
- **Ollama** — Local model hosting
- **Deepgram** — Real-time STT
- **faster-whisper** — Local STT
- **LangChain** — RAG implementation

### Infrastructure
- **SQLite** — Local data storage
- **ChromaDB** — Vector database for RAG
- **PyInstaller** — Python backend packaging

---

## Use Cases

### 1. Technical Interviews

**Scenario:** Software engineering interview with live coding

**How GHOST helps:**
- Suggests algorithms and data structures
- Provides complexity analysis hints
- Reminds you of syntax and best practices
- Invisible to interviewers during screen sharing

**Mode:** Interview Mode

---

### 2. Online Meetings

**Scenario:** Product demo or client presentation

**How GHOST helps:**
- Quick access to talking points
- Real-time information lookup
- Context management across topics
- Professional appearance (no visible assistant)

**Mode:** Meeting Mode

---

### 3. Coding Sessions

**Scenario:** Building a complex feature

**How GHOST helps:**
- Documentation lookup
- Code pattern suggestions
- Bug investigation assistance
- Architecture design discussions

**Mode:** Coding Mode

---

### 4. Learning & Research

**Scenario:** Following a technical tutorial

**How GHOST helps:**
- Concept explanations
- Related resources
- Troubleshooting help
- Interactive Q&A

**Mode:** Learning Mode

---

## Project Status

**Development Stage:** Active Development  
**Target Release:** Q2 2026  
**Platform:** Windows 10/11 (macOS planned)

### What's Working
- ✅ Invisible overlay technology
- ✅ Multi-provider LLM integration
- ✅ Real-time STT (Deepgram + faster-whisper)
- ✅ 5 specialized modes
- ✅ Basic RAG implementation

### In Progress
- 🔨 Advanced RAG with multi-document support
- 🔨 Voice activity detection (VAD)
- 🔨 Custom hotkey configuration
- 🔨 Performance optimizations

### Planned
- 📋 macOS support
- 📋 Linux support
- 📋 Mobile companion app
- 📋 Team/Enterprise features

---

## Documentation

- [Architecture Details](docs/ARCHITECTURE.md) — System design and technical decisions
- [Features Overview](docs/FEATURES.md) — Complete feature list with details
- [Tech Stack Deep Dive](docs/TECH_STACK.md) — Technology choices and rationale
- [Use Cases](docs/USE_CASES.md) — Real-world scenarios and workflows

---

## Configuration Example

```json
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
    "mode": "push-to-talk"
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

See [examples/config.example.json](examples/config.example.json) for full configuration.

---

## Why Commercial?

GHOST represents significant R&D investment in:
- **Novel UI technology** (invisible overlay implementation)
- **Multi-modal integration** (LLM + STT + RAG)
- **User experience design** for high-stakes scenarios
- **Performance optimization** for real-time operation

This showcase repository demonstrates:
- Technical capabilities and architectural decisions
- Real-world problem-solving approach
- Code quality and design thinking
- Production-ready engineering practices

**Note:** This is a documentation showcase. The actual product code is proprietary.

---

## About the Creator

Built by [CREATMAN](https://github.com/CreatmanCEO) — Full-Stack Developer specializing in automation, AI integration, and production-grade tools.

**Other Projects:**
- [ACCU](https://github.com/CreatmanCEO/accu) — AI-Curated Code Universe
- [smart-link-collector](https://github.com/CreatmanCEO/smart-link-collector) — AI Link Organizer Bot
- [AviaWallet](https://apps.apple.com/ru/app/aviawallet/id6754718339) — Crypto Wallet (App Store)

---

## License

**Commercial Product** — All rights reserved.

This showcase repository is for demonstration and portfolio purposes. The actual GHOST application and its source code are proprietary.

For business inquiries: creatmanick@gmail.com

---

## Connect

- **Website:** [creatman.site](https://creatman.site)
- **Telegram:** [@Creatman_it](https://t.me/Creatman_it)
- **Email:** creatmanick@gmail.com
- **GitHub:** [@CreatmanCEO](https://github.com/CreatmanCEO)

---

<sub>Built with Electron · React · Python · Claude · Deepgram</sub>
