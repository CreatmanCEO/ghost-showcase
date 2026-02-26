# GHOST Architecture

> Technical deep dive into GHOST's system design and implementation

---

## System Overview

GHOST is built as a **two-process architecture** with clear separation between UI and backend logic:

1. **Electron Process** (TypeScript + React) — UI, rendering, and Windows integration
2. **Python Backend** (FastAPI) — AI logic, LLM routing, STT processing, RAG system

This separation provides:
- **Clean architecture** — UI concerns separated from business logic
- **Technology flexibility** — Best tool for each job (Electron for desktop, Python for AI)
- **Performance isolation** — Heavy AI processing doesn't block UI
- **Independent scaling** — Backend can be offloaded to remote server

---

## Component Architecture

### 1. Frontend Layer (Electron + React)

```
┌────────────────────────────────────────┐
│         Main Process (Electron)        │
│  • Window management                   │
│  • System tray integration             │
│  • Global hotkeys                      │
│  • IPC bridge to backend               │
│  • WDA_EXCLUDEFROMCAPTURE flag         │
└────────────┬───────────────────────────┘
             │
             ▼
┌────────────────────────────────────────┐
│      Renderer Process (React)          │
│  • UI components                       │
│  • State management (Redux/Zustand)    │
│  • WebSocket client                    │
│  • Audio capture                       │
└────────────────────────────────────────┘
```

#### Key Technologies
- **Electron 32+** — Desktop framework with native Windows API access
- **React 18** — Component-based UI
- **TypeScript** — Type safety
- **Tailwind CSS** — Styling
- **WebSocket** — Real-time backend communication

#### Invisible Overlay Implementation

**The Core Innovation:**

```typescript
// Main process (Electron)
const mainWindow = new BrowserWindow({
  frame: false,
  transparent: true,
  alwaysOnTop: true,
  skipTaskbar: true,
  webPreferences: {
    nodeIntegration: false,
    contextIsolation: true
  }
});

// Windows-specific: Exclude from capture
if (process.platform === 'win32') {
  const { windowHandle } = mainWindow.getNativeWindowHandle();
  const WDA_EXCLUDEFROMCAPTURE = 0x00000011;
  
  // Set window attribute to exclude from screen capture
  SetWindowDisplayAffinity(windowHandle, WDA_EXCLUDEFROMCAPTURE);
}
```

**How it works:**
1. Window created as transparent + always-on-top
2. Windows display affinity set to `WDA_EXCLUDEFROMCAPTURE`
3. Screen recording software skips this window
4. User still sees the overlay normally

**Challenges solved:**
- **Performance** — Transparent windows can be heavy; optimized with GPU acceleration
- **Input handling** — Click-through for background, but interactive for controls
- **Multi-monitor** — Position persistence across different setups

---

### 2. Backend Layer (Python + FastAPI)

```
┌─────────────────────────────────────────────┐
│            FastAPI Application              │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │   WebSocket Server                  │   │
│  │   • Handles frontend connections    │   │
│  │   • Bidirectional real-time comms   │   │
│  └─────────────┬───────────────────────┘   │
│                │                            │
│    ┌───────────┴───────────┐               │
│    │                       │               │
│    ▼                       ▼               │
│  ┌────────────┐     ┌────────────┐         │
│  │ LLM Router │     │ STT Engine │         │
│  └────────────┘     └────────────┘         │
│         │                  │                │
│         ▼                  ▼                │
│  ┌─────────────────────────────────────┐   │
│  │         RAG System                  │   │
│  │  • Document processor               │   │
│  │  • Vector DB (ChromaDB)             │   │
│  │  • Context retrieval                │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

#### LLM Router

**Responsibility:** Abstract LLM provider differences

```python
class LLMRouter:
    def __init__(self, config: LLMConfig):
        self.provider = config.provider
        self.clients = {
            'claude': ClaudeClient(config.claude_api_key),
            'openai': OpenAIClient(config.openai_api_key),
            'ollama': OllamaClient(config.ollama_url),
            'lmstudio': LMStudioClient(config.lmstudio_url)
        }
    
    async def stream_completion(
        self, 
        messages: List[Message], 
        mode: str
    ) -> AsyncIterator[str]:
        """Stream LLM response with provider-specific handling"""
        client = self.clients[self.provider]
        
        # Add mode-specific system prompt
        system_prompt = self.get_mode_prompt(mode)
        messages = [{'role': 'system', 'content': system_prompt}] + messages
        
        # Stream tokens
        async for chunk in client.stream(messages):
            yield chunk
```

**Supported providers:**
- **Claude** (via Anthropic API) — Best for general assistance
- **GPT** (via OpenAI API) — Alternative for specific use cases
- **Ollama** — Local models (llama2, mistral, etc.)
- **LM Studio** — Custom local model hosting

---

#### STT Engine

**Responsibility:** Convert speech to text in real-time

**Architecture:**

```
Audio Input → Preprocessing → STT Provider → Text Output
                                   ↓
                           ┌───────┴────────┐
                           │                │
                       Deepgram        faster-whisper
                      (cloud, fast)    (local, private)
```

**Deepgram Implementation (Cloud):**

```python
class DeepgramSTT:
    async def stream_transcribe(self, audio_stream: AsyncIterator[bytes]):
        """Real-time streaming transcription"""
        async with websockets.connect(
            f"wss://api.deepgram.com/v1/listen",
            extra_headers={"Authorization": f"Token {self.api_key}"}
        ) as ws:
            # Send audio chunks
            async for audio_chunk in audio_stream:
                await ws.send(audio_chunk)
            
            # Receive transcriptions
            async for message in ws:
                result = json.loads(message)
                if result.get('is_final'):
                    yield result['channel']['alternatives'][0]['transcript']
```

**faster-whisper Implementation (Local):**

```python
class WhisperSTT:
    def __init__(self, model_size: str = "base"):
        self.model = WhisperModel(model_size, device="cuda", compute_type="float16")
    
    def transcribe(self, audio_file: str) -> str:
        """Transcribe audio file"""
        segments, info = self.model.transcribe(audio_file, beam_size=5)
        return " ".join([segment.text for segment in segments])
```

---

#### RAG System

**Responsibility:** Provide context from user documents

**Flow:**

```
1. Document Upload
   └→ Text Extraction (PyPDF2, python-docx)
      └→ Chunking (semantic + size-based)
         └→ Embedding (OpenAI text-embedding-3-small)
            └→ Storage (ChromaDB)

2. Query Time
   └→ User Question
      └→ Embedding Generation
         └→ Vector Search (top-k similar chunks)
            └→ Context Injection into LLM
```

**Implementation:**

```python
class RAGSystem:
    def __init__(self):
        self.chroma_client = chromadb.PersistentClient(path="./chroma_db")
        self.collection = self.chroma_client.get_or_create_collection(
            name="documents",
            embedding_function=embedding_functions.OpenAIEmbeddingFunction(
                api_key=os.getenv("OPENAI_API_KEY")
            )
        )
    
    async def add_document(self, file_path: str):
        """Process and store document"""
        # Extract text
        text = await self.extract_text(file_path)
        
        # Chunk text (500 tokens with 50 overlap)
        chunks = self.chunk_text(text, chunk_size=500, overlap=50)
        
        # Store with metadata
        self.collection.add(
            documents=chunks,
            metadatas=[{"source": file_path, "chunk": i} for i in range(len(chunks))],
            ids=[f"{file_path}_{i}" for i in range(len(chunks))]
        )
    
    async def retrieve_context(self, query: str, n_results: int = 3) -> List[str]:
        """Get relevant document chunks"""
        results = self.collection.query(
            query_texts=[query],
            n_results=n_results
        )
        return results['documents'][0]
```

---

## Communication Flow

### Typical Interaction Sequence

```
User → Frontend → Backend → LLM → Backend → Frontend → User
```

**Detailed flow:**

1. **User input** (text or speech)
2. **Frontend capture** → Audio or text input
3. **WebSocket send** → JSON message to backend
4. **STT processing** (if audio) → Convert speech to text
5. **RAG retrieval** (if enabled) → Get relevant context
6. **LLM request** → Send to selected provider
7. **Stream response** → Real-time token streaming
8. **Frontend render** → Display in overlay

**WebSocket message format:**

```typescript
// Frontend → Backend
{
  type: 'chat_message',
  payload: {
    content: 'What is the time complexity of quicksort?',
    mode: 'interview',
    use_rag: true
  }
}

// Backend → Frontend (streaming)
{
  type: 'llm_chunk',
  payload: {
    chunk: 'The time complexity of quicksort is...',
    done: false
  }
}
```

---

## Data Flow

### Configuration Management

```
┌─────────────────────┐
│  config.json        │
│  (user editable)    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  ConfigManager      │
│  • Validation       │
│  • Encryption       │
│  • Hot reload       │
└──────────┬──────────┘
           │
           ├─────────────┬────────────┐
           ▼             ▼            ▼
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ LLM      │  │ STT      │  │ UI       │
    │ Router   │  │ Engine   │  │ Settings │
    └──────────┘  └──────────┘  └──────────┘
```

### State Management

**Frontend state** (React + Zustand):
- UI state (theme, position, size)
- Conversation history
- Active mode
- Connection status

**Backend state** (in-memory):
- Active sessions (WebSocket connections)
- RAG document index
- LLM provider status
- Configuration cache

---

## Performance Optimizations

### 1. Lazy Loading

- **Documents:** Only load embeddings when RAG is actively used
- **Models:** Initialize LLM clients on first use
- **UI:** Code-split React components

### 2. Caching

- **LLM responses:** Cache common queries (optional)
- **RAG embeddings:** Persist ChromaDB to disk
- **Configuration:** In-memory cache with file watching

### 3. Streaming

- **LLM tokens:** Display as they arrive (reduces perceived latency)
- **STT:** Real-time transcription (no wait for complete audio)
- **Audio:** Stream from mic without buffering entire recording

### 4. Resource Management

- **GPU acceleration:** CUDA for Whisper (if available)
- **Connection pooling:** Reuse HTTP connections to APIs
- **Memory limits:** Bounded conversation history (last 20 messages)

---

## Security Considerations

### API Key Storage

```python
# Encrypted configuration
from cryptography.fernet import Fernet

class SecureConfig:
    def __init__(self):
        self.key = self.load_or_generate_key()
        self.cipher = Fernet(self.key)
    
    def encrypt_api_key(self, api_key: str) -> str:
        return self.cipher.encrypt(api_key.encode()).decode()
    
    def decrypt_api_key(self, encrypted: str) -> str:
        return self.cipher.decrypt(encrypted.encode()).decode()
```

### Network Security

- **HTTPS only** for API calls
- **WebSocket WSS** for frontend-backend (if remote)
- **Certificate validation** for all external connections

### Data Privacy

- **No logging** of conversation content by default
- **Local storage** for RAG documents
- **No telemetry** unless explicitly enabled

---

## Deployment Architecture

### Standalone Application

```
GHOST.exe
├── electron/          (frontend)
├── python_backend/    (bundled with PyInstaller)
│   ├── main.exe
│   ├── dependencies/
│   └── models/       (for local STT)
└── config.json
```

**Distribution:**
- Single installer (Inno Setup or similar)
- ~500MB (includes Whisper base model)
- Windows 10/11 x64

### Remote Backend (Optional)

For team/enterprise use:

```
┌─────────────┐         ┌─────────────┐
│  Client 1   │────────▶│             │
└─────────────┘         │   Backend   │
                        │   Server    │
┌─────────────┐         │  (FastAPI)  │
│  Client 2   │────────▶│             │
└─────────────┘         └─────────────┘
```

**Benefits:**
- Centralized LLM API key management
- Shared RAG knowledge base
- Usage analytics
- Cost optimization (shared API quota)

---

## Future Architecture Improvements

### Planned Enhancements

1. **Plugin system** — Allow community extensions
2. **Multi-window support** — Different modes in separate windows
3. **Voice output** — TTS for responses
4. **Mobile companion** — Control from phone
5. **Cloud sync** — Conversation history across devices

---

## Technical Decisions & Rationale

### Why Electron?

**Pros:**
- Cross-platform (Windows, macOS, Linux)
- Native API access (WDA_EXCLUDEFROMCAPTURE)
- Rich ecosystem (React, TypeScript)
- Mature desktop app framework

**Cons:**
- Large bundle size (~150MB base)
- Memory overhead
- Performance vs native

**Decision:** Pros outweigh cons for MVP. Native rewrite possible later.

---

### Why Python Backend?

**Pros:**
- Best-in-class AI/ML libraries (transformers, langchain)
- Fast iteration (prototyping)
- Rich ecosystem for RAG, embeddings
- Easy integration with ML models

**Cons:**
- Distribution complexity (PyInstaller)
- Startup time
- Not as performant as Go/Rust

**Decision:** Python's AI ecosystem is unmatched. Performance acceptable for use case.

---

### Why FastAPI?

**Pros:**
- Async/await support (crucial for streaming)
- WebSocket support
- Type hints (similar to TypeScript)
- Automatic API documentation

**Alternatives considered:**
- Flask (no native async)
- Django (too heavy)
- Node.js (wanted Python for AI libs)

---

### Why ChromaDB for RAG?

**Pros:**
- Lightweight (embedded mode)
- Good Python integration
- Fast enough for document retrieval
- Active development

**Alternatives considered:**
- Pinecone (cloud only, cost)
- Weaviate (too heavy for local)
- FAISS (lower-level, more work)

---

## Lessons Learned

### 1. Invisible Overlay Challenges

**Problem:** Initial attempts with window transparency had capture issues

**Solution:** Research led to `WDA_EXCLUDEFROMCAPTURE` Windows flag — game changer

### 2. WebSocket Stability

**Problem:** Connections dropping randomly

**Solution:** Implemented reconnection logic with exponential backoff + heartbeat pings

### 3. RAG Context Length

**Problem:** Too much context made LLM responses slow and unfocused

**Solution:** Limited to top-3 chunks (500 tokens each) — sweet spot for relevance

### 4. STT Latency

**Problem:** faster-whisper too slow for real-time

**Solution:** Hybrid approach — Deepgram for real-time, Whisper for offline

---

## Development Setup

For developers interested in similar architectures:

```bash
# Frontend
cd frontend
npm install
npm run dev

# Backend
cd backend
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
uvicorn main:app --reload

# Full app
npm run start  # Launches both processes
```

---

**Architecture Version:** 1.0  
**Last Updated:** 2026-02-26  
**Author:** CREATMAN

---

<sub>See also: [FEATURES.md](FEATURES.md) | [TECH_STACK.md](TECH_STACK.md) | [USE_CASES.md](USE_CASES.md)</sub>
