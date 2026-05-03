# 👻 GHOST — General Helper & Overlay Smart Tool

[![License: Commercial](https://img.shields.io/badge/license-Commercial-9d6cff)](LICENSE)
[![Stars](https://img.shields.io/github/stars/CreatmanCEO/ghost-showcase?style=flat&color=yellow)](https://github.com/CreatmanCEO/ghost-showcase/stargazers)
[![Validate](https://github.com/CreatmanCEO/ghost-showcase/actions/workflows/validate.yml/badge.svg)](https://github.com/CreatmanCEO/ghost-showcase/actions/workflows/validate.yml)
[![Status](https://img.shields.io/badge/%D1%81%D1%82%D0%B0%D1%82%D1%83%D1%81-%D0%A4%D0%B0%D0%B7%D0%B0%201%20%C2%B7%20%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%B0%D1%8F%20%D1%80%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0-22c55e)](#статус-проекта)
[![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4?logo=windows&logoColor=white)](#ограничения)
[![Open to discussion](https://img.shields.io/badge/%D0%BE%D1%82%D0%BA%D1%80%D1%8B%D1%82%D1%8B%20%D0%BA-%D0%B8%D0%BD%D0%B2%D0%B5%D1%81%D1%82%D0%B8%D1%86%D0%B8%D0%B8%20%C2%B7%20%D0%BF%D0%B0%D1%80%D1%82%D0%BD%D1%91%D1%80%D1%81%D1%82%D0%B2%D1%83%20%C2%B7%20%D0%BF%D1%80%D0%BE%D0%B4%D0%B0%D0%B6%D0%B5-cc785c)](#открыты-к-обсуждению)

🇷🇺 Русский · [🇬🇧 English](README.md)

> **AI-ассистент в реальном времени, невидимый при записи экрана и шеринге — для собеседований, встреч, кодинга и обучения.** Phase 1 в активной разработке. Автор открыт к инвестициям / партнёрству / продаже.

---

## Траектория проекта

GHOST начался как личная альтернатива дорогим платным инструментам — построен через анализ что каждый конкурент делает хорошо и где промахивается, и комбинацию лучших частей в одном продукте. После двух месяцев фокусной разработки на столе два пути, и автор открыт к обоим:

- **Путь A — owner-led рост** — с инвестициями, ко-фаундером / техническим партнёром или небольшой командой для коммерческой стороны (сильная сторона автора — инжиниринг и продукт, не go-to-market)
- **Путь B — стратегическая продажа** — реепаблишеру или крупному игроку у которого уже есть дистрибуция и коммерческий пайплайн

Проект **пока не готов** ни к одному из путей. Этот репозиторий — публичная витрина дизайна, архитектуры и инженерного прогресса на сегодня — и честная карта того, что нужно сделать до V1.0. См. [Что нужно для релиза](#что-нужно-для-релиза).

Если любой путь интересен — секция [Открыты к обсуждению](#открыты-к-обсуждению) содержит контакты.

---

## Зачем GHOST?

Поле AI-overlay-ассистентов плотное. Автор проанализировал каждого значимого конкурента и построил GHOST вокруг пробелов:

| Конкурент | Что взяли | Чего избежали |
|---|---|---|
| **Cluely** ($75/мес) | Multi-mode (собеседования / встречи / экзамены) — чёткое позиционирование use-case'ов | Только stealth, нет offline-режима, дорого за то что предлагают |
| **Screenpipe** (16K stars) | Plugin-архитектура, MCP-сервер, 24/7 захват, full-text поиск | Нет invisible-overlay, по умолчанию captures всё (privacy-вопрос) |
| **Natively** (open-source) | RAG с локальной vector DB, multi-provider LLM, WASAPI | UX менее polished, меньше режимов, нет streaming-first pipeline |
| **Interview Hunter** | Бесплатный tier, низкий порог входа, загрузка резюме для персонализации | Single-purpose, только собеседования |
| **Interview Coder** | Пошаговое решение задач, распознавание кода с экрана | Узкий, нет general assistant |
| **Final Round AI** | Mock-interview / preparation mode | Закрытая экосистема, нет general assistant |

**GHOST = stealth + multi-mode + multi-provider LLM + RAG + offline-capable + streaming-first pipeline.** Сегодня ни один конкурент не объединяет все шесть.

### Критические проблемы индустрии, которые мы решаем

1. **Задержка ответа 10–30 секунд** — большинство ждут полного вопроса, потом полного ответа LLM. Убийца usability на live-собесе.
2. **Шаблонные ответы в стиле ChatGPT** — без RAG-контекста (резюме, описание вакансии, заметки о компании) ассистент звучит совершенно не как пользователь.
3. **Detection несмотря на «stealth»-обещания** — стелс многих конкурентов тихо ломается на свежих апдейтах Zoom / Teams.
4. **Зависимость от облачных LLM** — privacy-conscious пользователи хотят local-LLM опцию для чувствительных контекстов.

Решения GHOST на каждое — в разделах [Архитектура](#архитектура) и [Latency target](#latency-target).

---

## Архитектура

![Two-process архитектура: невидимый Electron-оверлей (renderer + main с WDA_EXCLUDEFROMCAPTURE через koffi) ↔ WebSocket JSON-RPC порт 9876 ↔ Python asyncio sidecar (Silero VAD, Deepgram или faster-whisper STT, litellm router для Claude/GPT/Ollama/LM Studio, ChromaDB RAG)](docs/architecture.svg)

Two-process design с чистой WebSocket-границей:

| Слой | Стек | Ответственность |
|---|---|---|
| Electron renderer | React 19 · Vite · Tailwind · Zustand | Невидимый UI оверлея · streaming-токены · session view · 5 режимов |
| Electron main | Node 20+ · `koffi` · Win32 API | `SetWindowDisplayAffinity` → `WDA_EXCLUDEFROMCAPTURE` · global hotkeys · system tray · process supervisor |
| Транспорт | WebSocket JSON-RPC (порт 9876) | Развязывает UI и inference. Reconnection-resilient, schema-versioned. |
| Python sidecar | Python 3.11+ · `asyncio` · `websockets` | EventBus · CaptureManager · Silero VAD · STT · LLM router · RAG |
| Захват аудио | `pyaudiowpatch` (WASAPI loopback) + микрофон | Dual-source — захват системного аудио (голос собеседника) и микрофона одновременно |
| VAD | `silero-vad` | Детект начала речи за ~200 мс — драйвит streaming pipeline |
| STT | `deepgram-sdk` (cloud) → `faster-whisper` (local fallback) | Streaming partial-транскрипты; cloud быстрее, local приватнее |
| LLM | `litellm` Router → Anthropic Claude · OpenAI GPT · Ollama · LM Studio | Генерация начинается на partial-транскрипте — первые слова ответа появляются до окончания вопроса |
| RAG | `chromadb` (embedded, local) | Per-session загрузка документов (CV, JD, заметки) → контекстуальные ответы |
| Хранилище | SQLite (sessions / transcripts / settings) + encrypted `.env` (API ключи) | Локально по умолчанию, без телеметрии |

Глубже — [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

### Невидимый оверлей — ключевая техническая инновация

```typescript
// Electron main process — Windows-specific
import koffi from 'koffi';

const user32 = koffi.load('user32.dll');
const SetWindowDisplayAffinity = user32.func('SetWindowDisplayAffinity', 'bool', ['void *', 'uint32']);

const WDA_EXCLUDEFROMCAPTURE = 0x00000011;  // Windows 10 v2004+
SetWindowDisplayAffinity(mainWindow.getNativeWindowHandle(), WDA_EXCLUDEFROMCAPTURE);
```

Результат: оверлей виден глазам пользователя, но **OS исключает его из screen-recording API**, включая используемые Zoom / Teams / Google Meet / Discord / OBS / нативными скриншотерами. Протестировано на Windows 10 v2004+ и Windows 11.

---

## Latency target

> **Первые токены ответа в течение 1–2 секунд после окончания вопроса пользователем.** Это target, ещё не измеренный benchmark. Phase 1 строит streaming pipeline, который делает это достижимым.

Pipeline:

```
Пользователь начинает говорить
    ↓
Silero VAD детектит начало речи (~200 мс)
    ↓
Deepgram (или faster-whisper) стримит ЧАСТИЧНЫЕ транскрипты по мере распознавания слов
    ↓
LLM начинает генерацию на ЧАСТИЧНОМ транскрипте — НЕ ждёт окончания фразы
    ↓
Токены LLM стримятся в оверлей по символу
    ↓
Пользователь видит первые слова ответа ~1–2 с после того как замолчал
```

Почему конкуренты это не делают — большинство ждут полного окончания вопроса, потом одного non-streaming LLM-вызова, потом полного ответа, потом рендерят. Кумулятивная задержка 10–30 секунд.

---

## Статус проекта

**Phase 1 — Core (активная разработка).** Конкретный прогресс:

| Модуль | Статус |
|---|---|
| Невидимый оверлей через `koffi` + Win32 `WDA_EXCLUDEFROMCAPTURE` | ✅ |
| Каркас Electron 33 + React 19 + Vite + Tailwind | ✅ |
| Глобальные хоткеи (`Ctrl+Shift+G/V/A/M/Esc`) | ✅ |
| WASAPI loopback (системное аудио) + микрофон — dual-source | ✅ |
| Auto-detection параметров аудиоустройств (sample rate, channels) | ✅ |
| Python sidecar с `asyncio` WebSocket-сервером на порту 9876 | ✅ |
| EventBus с thread-safe locking | ✅ |
| CaptureManager интегрирован в start / stop capture | ✅ |
| Capture-mode selector в UI | ✅ |
| Silero VAD интеграция | 🔨 |
| Streaming Deepgram → streaming LLM → streaming overlay-токены | 🔨 |
| Multi-document RAG с ChromaDB | 🔨 |
| UI кастомных хоткеев | 🔨 |
| Performance оптимизация (target < 10% CPU в idle) | 🔨 |

**Phase 2 — V1.0 ship requirements.** См. [Что нужно для релиза](#что-нужно-для-релиза).

**Phase 3 — после V1.0.** macOS · Linux · мобильный компаньон · team / enterprise.

---

## Что нужно для релиза

Честный gap-analysis для V1.0. Это то, что инвестору / партнёру / acquirer'у нужно оценить.

### Engineering пробелы

- **macOS / Linux порты** — `WDA_EXCLUDEFROMCAPTURE` Windows-only. Эквиваленты на macOS (`NSWindowSharingNone`) и X11 / Wayland существуют, но требуют platform-specific работы.
- **Code signing & notarization** — Windows code signing (Authenticode), macOS notarization (когда придёт Phase 3). Без этого — SmartScreen / Gatekeeper warning при каждой установке.
- **Auto-update инфраструктура** — `electron-updater` + release-сервер (S3 + signed manifests).
- **Distributable installer** — Inno Setup или NSIS для одного `.exe` install path с нормальным uninstaller'ом.
- **Crash reporting** — Sentry или self-hosted альтернатива для production-диагностики. Opt-in.
- **Telemetry** — usage analytics (только opt-in) для product-решений. Строго анонимизировано.

### Product пробелы

- **Onboarding flow polish** — first-launch UX, API-key wizard, voice-test, выбор режима.
- **Pricing page и маркетинг-сайт** — публичного сайта пока нет.
- **Payment integration** — Stripe / Paddle / Lemon Squeezy. License-key валидация при старте.
- **Trial / freemium логика** — N-day trial vs free local-LLM-only tier.
- **Support documentation** — user-facing docs, FAQ, troubleshooting.

### Business пробелы

- **Legal review** — Terms of Service, Privacy Policy, GDPR / CCPA аудит, EULA для бинарника.
- **Trademark filing** — защита имени «GHOST» в ключевых юрисдикциях.
- **Customer support ops** — минимум triaged inbox + Discord / Telegram community.
- **Marketing strategy** — позиционирование vs Cluely / Screenpipe в целевых каналах (HN, Reddit, dev YouTube).

### Investment thesis (3-line summary)

- **Рынок** — AI assistant tools, быстро растёт; ценники конкурентов $20–$150/мес сигналят о реальной готовности платить.
- **Gap** — ни один продукт сейчас не комбинирует stealth + multi-mode + multi-provider + RAG + offline; ближайший Cluely $75/мес только за stealth.
- **Why now** — снижение стоимости LLM + жизнеспособные local LLM (Ollama / LM Studio) + зрелость Win32 stealth API делают продукт достижимым на indie-масштабе.

---

## Возможности

### 🎯 Пять специализированных режимов

Каждый режим — это opinionated промпт + UI affordance, не отдельная модель:

1. **Interview** — подсказки алгоритмов, complexity-анализ, синтаксис. Тюнен под live-coding собесы.
2. **Meeting** — talking points, action-item capture, контекст по темам.
3. **Coding** — лукап документации, code-pattern suggestions, debug-помощь.
4. **Learning** — объяснения концептов, related resources, интерактивный Q&A.
5. **General** — flexible default для всего остального.

### 🤖 Multi-provider LLM

Выбирается per-mode или глобально. Всё через `litellm`:

- **Anthropic Claude** — primary рекомендация (Sonnet / Opus / Haiku)
- **OpenAI GPT** — альтернатива (GPT-4o, GPT-4 Turbo, GPT-3.5 Turbo)
- **Ollama** (локальный) — `llama3`, `mistral`, `codellama`, `phi-3` и др.
- **LM Studio** (локальный) — любая GGUF-модель с OpenAI-совместимым API

### 🎤 Real-time STT

- **Deepgram** (cloud) — ультра-низкая latency, 95%+ word accuracy
- **faster-whisper** (локально) — offline, медленнее, полностью приватно
- Push-to-talk и continuous (VAD-driven) режимы
- Multi-language

### 📚 RAG-документы

Загружаешь PDF / DOCX / TXT / MD per-session — обычно резюме, описание вакансии, заметки о компании для собеса, или повестка встречи. Документы чанкуются, эмбеддятся, лежат в локальной ChromaDB-коллекции. Retrieval автоматический во время генерации.

### 🔒 Privacy & security

- **Offline-режим** — Ollama + faster-whisper = 100% локально, 0 сетевых запросов
- **No data storage по умолчанию** — сессии можно сохранять по запросу
- **Encrypted config** — API-ключи в `.env`, опционально OS-keychain-protected
- **Open architecture** — каждый компонент задокументирован, никакой непрозрачной cloud-зависимости

Полный feature catalogue — [`docs/FEATURES.md`](docs/FEATURES.md).

---

## Как это построено

GHOST построен соло автором через структурированный **multi-agent dev workflow** поверх [Claude Code](https://code.claude.com). Паттерн:

| Роль агента | Ответственность |
|---|---|
| **Orchestrator** | Планирует таски, распределяет coderам, отслеживает прогресс, делает интеграцию |
| **Coder × N** | Реализует независимые модули параллельно — TDD, маленькие коммиты, feature branches |
| **Tester** | Запускает полный test-suite после каждого coderа — ловит регрессии рано |
| **Documentation** | Обновляет design-доки и changelog в той же сессии что и код |

Это тот же паттерн, который автор поддерживает как standalone-инструмент в [`claude-code-antiregression-setup`](https://github.com/CreatmanCEO/claude-code-antiregression-setup) — featured на Хабре (топ-5 дня, 20K чтений, заявка на «Технотекст 8»). Для инвестора / acquirer'а это ответ на «как соло-фаундер шипит сложный софт»: документированным повторяемым процессом, не по принципу «через силу».

Полный Claude Code-toolset того же автора — в разделе [Связанные проекты](#связанные-проекты).

---

## Tech stack

### Frontend
- **Electron 33+** — desktop framework, native Windows API через `koffi`
- **React 19** — UI и state
- **TypeScript** — type-safe development
- **Vite** — быстрый dev server и сборка
- **Tailwind CSS** — стилизация
- **Zustand** — state management

### Backend (Python sidecar)
- **Python 3.11+** — core логика
- **`asyncio` + `websockets`** — sidecar server
- **`pyaudiowpatch`** — WASAPI loopback capture
- **`silero-vad`** — voice activity detection
- **`litellm`** — multi-provider LLM router
- **`deepgram-sdk`** — cloud STT
- **`faster-whisper`** — local STT
- **`chromadb`** — embedded vector DB

### AI providers
- **Anthropic Claude** — primary LLM
- **OpenAI GPT** — alternative
- **Ollama / LM Studio** — local LLM hosting
- **Deepgram** — real-time cloud STT

### Инфраструктура
- **SQLite** — local data
- **ChromaDB** — vector DB для RAG
- **Inno Setup / electron-builder** — Windows-пакетирование (V1.0)

---

## Use cases

### 1. Технические собеседования

**Сценарий:** software-engineering собес с live coding.

GHOST подсказывает алгоритмы и структуры данных, complexity-hints, напоминает синтаксис. Невидим интервьюерам при шеринге экрана.

**Режим:** Interview.

### 2. Онлайн-встречи

**Сценарий:** product demo или client presentation.

Быстрый доступ к talking points, real-time fact lookup, контекст по темам. Профессиональный вид — без видимого ассистента.

**Режим:** Meeting.

### 3. Кодинг

**Сценарий:** делаешь сложную фичу.

Documentation lookup, code-pattern suggestions, bug investigation, обсуждение архитектуры.

**Режим:** Coding.

### 4. Обучение и research

**Сценарий:** проходишь технический туториал или читаешь paper.

Объяснения концептов, related resources, troubleshooting, интерактивный Q&A.

**Режим:** Learning.

---

## Configuration пример

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

Полная схема — [`examples/config.example.json`](examples/config.example.json).

---

## Ограничения

- **Только Windows 10 v2004+.** `WDA_EXCLUDEFROMCAPTURE` Windows-specific. macOS (`NSWindowSharingNone`) и Linux (Wayland / X11) — на roadmap'е, ещё не сделаны.
- **Phase 1 — pre-V1.0.** Streaming pipeline, multi-document RAG, onboarding polish, installer — всё в процессе.
- **API-ключи нужны для cloud-провайдеров** (Deepgram / Claude / GPT). 100% offline режим требует Ollama или LM Studio локально — пути «без банковской карты» нет до V1.0 (где будет free-tier опция).
- **Screen-capture exclusion — OS-level, не магия.** Работает против стандартных Windows Desktop Duplication API и DirectX-capture путей, используемых mainstream meeting / recording софтом. Целеустремлённый противник с kernel-level или внешним hardware capture — out of scope.
- **Мобильного приложения-компаньона пока нет.** Telegram-bot bridge — возможное направление Phase 3.

---

## Открыты к обсуждению

Проект в активной разработке. Автор открыт к:

- **Инвестициям** для owner-led роста — соло-фаундер ищет business / commercial ко-фаундера или seed funding для V1.0
- **Стратегической продаже или republishing** — если у вас есть существующий канал дистрибуции и хотите acquire / license технологию
- **Техническому ко-фаундеру или партнёрству** — для go-to-market, ops или business-стороны

**Контакты:**

- 📧 Email: **creatmanick@gmail.com**
- 💬 Telegram: **[@Creatman_it](https://t.me/Creatman_it)**
- 🌐 Сайт: **[creatman.site](https://creatman.site)**
- 🐙 GitHub: **[@CreatmanCEO](https://github.com/CreatmanCEO)**

---

## Связанные проекты — Claude Code-экосистема того же автора

GHOST построен в Claude Code-centric workflow. Автор поддерживает публичный toolset для этой дисциплины:

- [`claude-code-antiregression-setup`](https://github.com/CreatmanCEO/claude-code-antiregression-setup) — `CLAUDE.md` + субагенты + хуки. [Habr](https://habr.com/ru/articles/1013330/) (топ-5 дня, 20K чтений).
- [`ai-context-hierarchy`](https://github.com/CreatmanCEO/ai-context-hierarchy) — трёхуровневая система контекста. Featured в [Graphify v5.0 roadmap](https://github.com/safishamsi/graphify/issues/425).
- [`claude-statusline`](https://github.com/CreatmanCEO/claude-statusline) — statusline для Claude Code с VPS-мониторингом. [Habr](https://habr.com/ru/articles/1013414/).
- [`notebooklm-claude-workflows`](https://github.com/CreatmanCEO/notebooklm-claude-workflows) — 7 slash-команд для Google NotebookLM research-пайплайнов.
- [`lingua-companion`](https://github.com/CreatmanCEO/lingua-companion) — voice-first English-tutor для русскоговорящих IT-специалистов. Другой домен, та же инженерная дисциплина.
- [`diabot`](https://github.com/CreatmanCEO/diabot) — non-commercial Telegram-бот для диабета 1 типа (фото еды → carb counting). Тот же автор.

---

## Автор

**Николай Подоляк (Nick Podolyak)** — Python-разработчик и цифровой архитектор в [CREATMAN](https://creatman.site)

- GitHub: [@CreatmanCEO](https://github.com/CreatmanCEO)
- Habr: [creatman](https://habr.com/ru/users/creatman/)
- dev.to: [@creatman](https://dev.to/creatman)

---

## Лицензия

**Commercial — All rights reserved.** Этот showcase-репозиторий для демонстрации, портфолио и обсуждения. Само GHOST-приложение и его исходный код — proprietary и в этом репозитории не лежат. См. [`LICENSE`](LICENSE).

По вопросам лицензирования — инвестиции, партнёрство, acquisition — секция [Открыты к обсуждению](#открыты-к-обсуждению).

---

<sub>Built with Electron · React · TypeScript · Python · Claude · Deepgram · ChromaDB</sub>
