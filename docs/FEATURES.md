# GHOST Features

> Complete feature list with implementation details

---

## Core Features

### 1. Invisible Overlay Technology

**What it does:**  
Creates a desktop overlay that is completely invisible to screen recording and screenshot software while remaining fully visible to the user.

**How it works:**
- Uses Windows `WDA_EXCLUDEFROMCAPTURE` display affinity flag
- Window rendered as transparent + always-on-top
- GPU-accelerated rendering for smooth performance
- No impact on video conferencing or screen sharing

**Use cases:**
- Technical interviews with live coding
- Online presentations and demos
- Video tutorials and recordings
- Screen sharing in meetings

**Status:** ✅ Fully implemented

---

### 2. Multi-Provider LLM Support

**What it does:**  
Connect to different AI language models based on your needs and preferences.

**Supported providers:**

#### Anthropic Claude
- **Models:** Claude 3.5 Sonnet, Claude 3 Opus, Claude 3 Haiku
- **Best for:** General assistance, coding, complex reasoning
- **Latency:** Low (~500ms first token)
- **Cost:** Pay-per-token

#### OpenAI GPT
- **Models:** GPT-4o, GPT-4 Turbo, GPT-3.5 Turbo
- **Best for:** Alternative to Claude, specific use cases
- **Latency:** Medium (~800ms first token)
- **Cost:** Pay-per-token

#### Ollama (Local)
- **Models:** llama2, mistral, codellama, phi-2, many more
- **Best for:** Complete privacy, offline use
- **Latency:** Variable (depends on hardware)
- **Cost:** Free (runs on your GPU/CPU)

#### LM Studio (Local)
- **Models:** Any GGUF format model
- **Best for:** Custom model hosting, fine-tuned models
- **Latency:** Variable (depends on hardware)
- **Cost:** Free (runs on your hardware)

**Provider switching:**
- Change provider on-the-fly in settings
- No conversation loss when switching
- Automatic fallback if primary provider fails

**Status:** ✅ Fully implemented

---

### 3. Specialized Modes

**What it does:**  
Pre-configured assistant behaviors optimized for specific scenarios.

#### Interview Mode

**Optimized for:** Technical interviews, coding challenges

**System prompt highlights:**
- Hints without giving away answers
- Complexity analysis reminders
- Edge case suggestions
- Best practice nudges

**Behavior:**
- Concise responses (no walls of text)
- Focus on algorithms and data structures
- Time/space complexity analysis
- Code pattern suggestions

**Example interaction:**
```
You: "How should I solve this linked list problem?"
GHOST: "Consider the two-pointer technique. Fast and slow pointers can help detect cycles or find the middle. What's your initial approach?"
```

#### Meeting Mode

**Optimized for:** Professional meetings, presentations, demos

**System prompt highlights:**
- Quick fact lookups
- Talking point reminders
- Context management
- Professional tone

**Behavior:**
- Bullet-point responses
- Focus on clarity
- No technical jargon unless requested
- Time-aware (keeps responses short)

**Example interaction:**
```
You: "Client asking about API latency improvements"
GHOST: "Key points:
• 40% latency reduction in Q4
• New caching layer (Redis)
• CDN expansion to APAC
• P95 now under 200ms"
```

#### Coding Mode

**Optimized for:** Active development, debugging

**System prompt highlights:**
- Code suggestions
- Bug analysis
- Documentation lookup
- Architecture advice

**Behavior:**
- Code-first responses
- Language-specific best practices
- Link to official docs
- Alternative approaches

**Example interaction:**
```
You: "Best way to handle async errors in Python?"
GHOST: "Use try-except with asyncio.gather:

```python
results = await asyncio.gather(
    task1(), task2(), task3(),
    return_exceptions=True
)

for result in results:
    if isinstance(result, Exception):
        logging.error(f"Task failed: {result}")
```

Or use asyncio.TaskGroup (Python 3.11+)"
```

#### Learning Mode

**Optimized for:** Tutorials, courses, research

**System prompt highlights:**
- Concept explanations
- Related resources
- Q&A style responses
- Encourages exploration

**Behavior:**
- Teaching tone
- Analogies and examples
- Follow-up questions
- Links to learning materials

**Example interaction:**
```
You: "What is dependency injection?"
GHOST: "Dependency injection is like ordering at a restaurant:

WITHOUT DI: You walk into the kitchen and make your own food (tight coupling)

WITH DI: You order from a menu, and the kitchen brings you food (loose coupling)

The class receives its dependencies from outside rather than creating them internally. This makes testing easier (you can inject mocks) and code more flexible.

Want to see a Python example?"
```

#### General Mode

**Optimized for:** Flexible, general-purpose assistance

**System prompt highlights:**
- Adaptive responses
- Wide knowledge base
- Context-aware
- Balanced tone

**Behavior:**
- Adjusts to query type
- No specific constraints
- Can handle any topic
- Full Claude/GPT capabilities

**Status:** ✅ All 5 modes implemented

---

### 4. Real-Time Speech-to-Text

**What it does:**  
Convert your speech to text in real-time for hands-free interaction.

#### Deepgram (Cloud)

**Features:**
- Ultra-low latency (< 300ms)
- High accuracy (>90% WER)
- Real-time streaming
- Multi-language support

**Best for:**
- Real-time conversations
- Interviews (fastest response)
- When internet is available

**Configuration:**
```json
{
  "stt": {
    "provider": "deepgram",
    "language": "en-US",
    "model": "nova-2",
    "interim_results": true
  }
}
```

#### faster-whisper (Local)

**Features:**
- Completely offline
- Multiple model sizes (tiny, base, small, medium, large)
- GPU acceleration (CUDA)
- No API costs

**Best for:**
- Privacy-sensitive use cases
- Offline scenarios
- When latency is acceptable (~2-5s)

**Configuration:**
```json
{
  "stt": {
    "provider": "whisper",
    "model_size": "base",
    "device": "cuda",
    "language": "en"
  }
}
```

#### Input Modes

**Push-to-talk:**
- Hold hotkey (default: Ctrl+Space) to record
- Release to process
- Visual feedback while recording

**Continuous:**
- Always listening
- Voice activity detection (VAD)
- Auto-submit on silence

**Manual:**
- Type instead of speak
- Fallback when audio not available

**Status:** ✅ Both providers implemented, VAD in progress

---

### 5. RAG (Retrieval-Augmented Generation)

**What it does:**  
Upload documents and get context-aware answers based on their content.

**Supported formats:**
- PDF (.pdf)
- Word documents (.docx)
- Text files (.txt)
- Markdown (.md)

**How it works:**

1. **Upload:** Drag and drop or select files
2. **Processing:** Text extraction + chunking (500 tokens)
3. **Embedding:** Convert to vectors (OpenAI embeddings)
4. **Storage:** Persist in ChromaDB
5. **Query:** Automatic retrieval when answering

**Example use case: Interview Prep**

```
1. Upload: company_info.pdf, product_docs.pdf
2. Ask: "What are our Q4 priorities?"
3. GHOST: "Based on your documents:
   • Launch new API v2 (Q4 target)
   • Expand EMEA customer base
   • Improve onboarding flow
   
   [Sources: company_info.pdf, page 12]"
```

**Configuration:**
```json
{
  "rag": {
    "enabled": true,
    "chunk_size": 500,
    "chunk_overlap": 50,
    "top_k": 3,
    "similarity_threshold": 0.7
  }
}
```

**Status:** ✅ Basic implementation, advanced features in progress

---

## UI Features

### 1. Customizable Appearance

**Theme:**
- Dark mode (default)
- Light mode
- Auto (system preference)

**Opacity:**
- Adjustable 50-100%
- Persistent across sessions

**Position:**
- Drag and drop
- Snap to screen edges
- Multi-monitor support
- Remember position per mode

**Size:**
- Resize from corners
- Min/max constraints
- Responsive layout

---

### 2. Hotkeys

**Global hotkeys (work anywhere):**
- `Ctrl+Shift+G` — Toggle overlay visibility
- `Ctrl+Space` — Push-to-talk (hold to record)
- `Ctrl+Shift+M` — Cycle through modes
- `Ctrl+Shift+C` — Clear conversation

**Local hotkeys (when focused):**
- `Esc` — Hide overlay
- `Ctrl+Enter` — Send message
- `Ctrl+K` — Focus input
- `Ctrl+/` — Show help

**Customization:**
```json
{
  "hotkeys": {
    "toggle_overlay": "Ctrl+Shift+G",
    "push_to_talk": "Ctrl+Space",
    "cycle_mode": "Ctrl+Shift+M"
  }
}
```

---

### 3. Conversation Management

**History:**
- Last 20 messages kept in memory
- Optional persistence to disk
- Export as JSON or Markdown

**Search:**
- Full-text search in history
- Filter by mode
- Date range filtering

**Context control:**
- Clear conversation (start fresh)
- Delete specific messages
- Edit user messages

---

### 4. System Tray Integration

**Features:**
- Quick access to modes
- Show/hide overlay
- Settings shortcut
- Quit application

**Indicators:**
- Active mode icon
- Recording status
- Connection status (cloud STT/LLM)

---

## Advanced Features

### 1. Prompt Templates

**What it does:**  
Save and reuse custom prompts for repeated tasks.

**Example templates:**

```markdown
# Code Review
Review this code for:
- Security issues
- Performance problems
- Best practices violations
- Suggested improvements

{code}
```

```markdown
# Meeting Notes
Summarize the key points from this meeting:
- Main decisions
- Action items
- Open questions
- Next steps

{transcript}
```

**Usage:**
1. Create template in settings
2. Use `/template-name` command
3. Fill in placeholders
4. Get structured response

**Status:** 📋 Planned for v1.1

---

### 2. Multi-Language Support

**UI languages:**
- English (default)
- Russian
- Spanish
- French
- German

**STT languages:**
- All languages supported by Deepgram/Whisper
- Auto-detection option
- Language switching on-the-fly

**Status:** 📋 Planned for v1.2

---

### 3. Team Features

**For organizations:**

**Shared knowledge base:**
- Company documents in shared RAG
- Team templates
- Common prompts

**Usage analytics:**
- API cost tracking
- Usage reports
- User activity

**Admin controls:**
- Approved LLM providers
- Usage limits
- Security policies

**Status:** 📋 Planned for v2.0

---

## Configuration

### Full Configuration Schema

```json
{
  "llm": {
    "provider": "claude",
    "claude": {
      "api_key": "encrypted:...",
      "model": "claude-3-5-sonnet-20241022",
      "temperature": 0.7,
      "max_tokens": 2000
    },
    "openai": {
      "api_key": "encrypted:...",
      "model": "gpt-4o",
      "temperature": 0.7,
      "max_tokens": 2000
    },
    "ollama": {
      "url": "http://localhost:11434",
      "model": "llama2"
    },
    "lmstudio": {
      "url": "http://localhost:1234",
      "model": "custom-model"
    }
  },
  "stt": {
    "provider": "deepgram",
    "deepgram": {
      "api_key": "encrypted:...",
      "model": "nova-2",
      "language": "en-US"
    },
    "whisper": {
      "model_size": "base",
      "device": "cuda",
      "language": "en"
    },
    "mode": "push-to-talk",
    "vad_threshold": 0.5
  },
  "rag": {
    "enabled": true,
    "chunk_size": 500,
    "chunk_overlap": 50,
    "top_k": 3,
    "embedding_provider": "openai"
  },
  "ui": {
    "theme": "dark",
    "opacity": 0.95,
    "position": {
      "x": 100,
      "y": 100
    },
    "size": {
      "width": 400,
      "height": 600
    }
  },
  "hotkeys": {
    "toggle_overlay": "Ctrl+Shift+G",
    "push_to_talk": "Ctrl+Space",
    "cycle_mode": "Ctrl+Shift+M",
    "clear_conversation": "Ctrl+Shift+C"
  },
  "modes": {
    "default": "general",
    "interview": {
      "system_prompt": "You are a technical interview assistant...",
      "temperature": 0.5,
      "max_tokens": 500
    },
    "meeting": {
      "system_prompt": "You are a professional meeting assistant...",
      "temperature": 0.6,
      "max_tokens": 300
    },
    "coding": {
      "system_prompt": "You are a coding assistant...",
      "temperature": 0.3,
      "max_tokens": 1000
    },
    "learning": {
      "system_prompt": "You are a learning assistant...",
      "temperature": 0.7,
      "max_tokens": 800
    },
    "general": {
      "system_prompt": "You are a helpful AI assistant...",
      "temperature": 0.7,
      "max_tokens": 2000
    }
  },
  "privacy": {
    "save_conversations": false,
    "telemetry_enabled": false
  }
}
```

---

## Feature Roadmap

### v1.0 (Current)
- ✅ Invisible overlay
- ✅ Multi-provider LLM
- ✅ 5 specialized modes
- ✅ Real-time STT (Deepgram + Whisper)
- ✅ Basic RAG

### v1.1 (Q2 2026)
- 🔨 Advanced RAG (multi-document, citations)
- 🔨 Voice activity detection (VAD)
- 🔨 Custom hotkeys
- 🔨 Prompt templates
- 🔨 Conversation export

### v1.2 (Q3 2026)
- 📋 macOS support
- 📋 Multi-language UI
- 📋 TTS (text-to-speech)
- 📋 Plugin system
- 📋 Mobile companion app

### v2.0 (Q4 2026)
- 📋 Linux support
- 📋 Team features
- 📋 Cloud sync
- 📋 Analytics dashboard
- 📋 Enterprise features

---

**Legend:**  
✅ Implemented | 🔨 In Progress | 📋 Planned

---

<sub>See also: [ARCHITECTURE.md](ARCHITECTURE.md) | [TECH_STACK.md](TECH_STACK.md) | [USE_CASES.md](USE_CASES.md)</sub>
