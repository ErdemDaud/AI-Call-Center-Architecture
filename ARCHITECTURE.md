# System Architecture

This architecture documentation contains no proprietary code and is shared for educational/portfolio purposes only.
---

## System Overview

```
┌────────────────────────────────────────────────────────────────┐
│                         USER LAYER                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Phone System │  │  Web Client  │  │Admin Dashboard│         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
└─────────┼──────────────────┼──────────────────┼────────────────┘
          │                  │                  │
          │  WebSocket       │  WebSocket       │  HTTP/REST
          │                  │                  │
┌─────────▼──────────────────▼──────────────────▼────────────────┐
│                    FLASK APPLICATION SERVER                     │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Flask-SocketIO WebSocket Server             │  │
│  │  - Connection Management                                 │  │
│  │  - Session Routing                                       │  │
│  │  - Event Broadcasting                                    │  │
│  └────────────────────┬─────────────────────────────────────┘  │
│                       │                                         │
│  ┌────────────────────▼─────────────────────────────────────┐  │
│  │            RealtimeCallHandler (Session Manager)         │  │
│  │                                                           │  │
│  │  State Machine:                                          │  │
│  │  GREETING → LISTENING → PROCESSING → RESPONDING          │  │
│  │                  ↑_____________________________↓         │  │
│  │                                                           │  │
│  │  - Audio Buffer Management                               │  │
│  │  - Turn-based Conversation Control                       │  │
│  │  - Context Maintenance                                   │  │
│  └──┬────────┬─────────┬──────────┬──────────┬─────────────┘  │
└─────┼────────┼─────────┼──────────┼──────────┼────────────────┘
      │        │         │          │          │
      │        │         │          │          │
┌─────▼────┐ ┌▼──────┐ ┌▼───────┐ ┌▼────────┐ ┌▼───────────┐
│   STT    │ │  LLM  │ │  RAG   │ │   TTS   │ │  Database  │
│          │ │       │ │        │ │         │ │            │
│ Faster-  │ │Ollama │ │ FAISS  │ │  gTTS   │ │   SQLite   │
│ Whisper  │ │       │ │   +    │ │    +    │ │            │
│          │ │LLaMA  │ │Sentence│ │ pygame  │ │  Sessions  │
│   GPU    │ │  3.1  │ │ Trans. │ │         │ │   Turns    │
│   CUDA   │ │       │ │        │ │ Speed   │ │  Analytics │
│          │ │       │ │ Vector │ │  1.2x   │ │            │
└──────────┘ └───────┘ └────────┘ └─────────┘ └────────────┘
```

---

## Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      CONVERSATION TURN FLOW                      │
└─────────────────────────────────────────────────────────────────┘

1. Audio Input Phase:
   ┌──────────────┐
   │ User speaks  │
   └──────┬───────┘
          │ Audio Stream (chunks)
          ▼
   ┌─────────────────────┐
   │  Audio Buffer       │
   │  - Accumulate chunks│
   │  - VAD monitoring   │
   └──────┬──────────────┘
          │ Silence Detected (2s)
          ▼

2. Transcription Phase:
   ┌─────────────────────┐
   │  Speech-to-Text     │
   │  - faster-whisper   │
   │  - GPU acceleration │
   │  - Turkish language │
   └──────┬──────────────┘
          │ Transcribed Text
          ▼
   ┌─────────────────────┐
   │ Topic Classifier    │
   │ - LLM classification│
   └──────┬──────────────┘
          │ Topic Label
          ▼

3. Context Assembly Phase:
   ┌─────────────────────┐     ┌──────────────────┐
   │ Conversation History│     │  RAG Retrieval   │
   │ - Last 10 turns     │     │  - Query embed   │
   │ - Session context   │     │  - FAISS search  │
   └──────┬──────────────┘     │  - Top-K chunks  │
          │                     └────────┬─────────┘
          │                              │
          └───────────┬──────────────────┘
                      │ Combined Context
                      ▼

4. LLM Generation Phase:
   ┌─────────────────────────────────┐
   │  LLM Query (Ollama/LLaMA 3.1)   │
   │                                  │
   │  System Prompt +                 │
   │  Conversation History +          │
   │  RAG Context +                   │
   │  User Question                   │
   └──────┬──────────────────────────┘
          │ LLM Response
          ▼

5. Response Delivery Phase:
   ┌─────────────────────┐
   │  Text-to-Speech     │
   │  - gTTS generation  │
   │  - Speed: 1.2x      │
   │  - MP3 format       │
   └──────┬──────────────┘
          │ Audio Output
          ▼
   ┌─────────────────────┐
   │  Play to User       │
   │  - pygame mixer     │
   └─────────────────────┘

6. Persistence Phase:
   ┌─────────────────────┐
   │  Database Save      │
   │  - Turn record      │
   │  - Topic label      │
   │  - Timestamps       │
   │  - Audio path       │
   └─────────────────────┘
          │
          ▼
   Return to LISTENING state
```

---

## State Machine Diagram

```
┌─────────────────────────────────────────────────────────────┐
│              SESSION STATE MACHINE                           │
└─────────────────────────────────────────────────────────────┘

                    ┌──────────────┐
                    │  start_call  │
                    │    event     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
           ┌───────►│   GREETING   │
           │        └──────┬───────┘
           │               │ Play welcome message
           │               ▼
           │        ┌──────────────┐
           │   ┌───►│  LISTENING   │◄───┐
           │   │    └──────┬───────┘    │
           │   │           │             │
           │   │           │ Audio       │
           │   │           │ chunks      │
           │   │           │             │
           │   │           │ Silence     │
           │   │           │ detected    │
           │   │           ▼             │
           │   │    ┌──────────────┐    │
           │   │    │ PROCESSING   │    │
           │   │    │              │    │
           │   │    │ - STT        │    │
           │   │    │ - LLM query  │    │
           │   │    │ - TTS gen    │    │
           │   │    └──────┬───────┘    │
           │   │           │             │
           │   │           ▼             │
           │   │    ┌──────────────┐    │
           │   │    │ RESPONDING   │    │
           │   │    │              │    │
           │   │    │ - Play audio │    │
           │   │    │ - Save DB    │    │
           │   │    └──────┬───────┘    │
           │   │           │             │
           │   │           └─────────────┘
           │   │        Resume listening
           │   │
           │   │    ┌──────────────┐
           │   └────│  end_call    │
           │        │    event     │
           │        └──────┬───────┘
           │               │
           │               ▼
           │        ┌──────────────┐
           └────────│  COMPLETED   │
                    │              │
                    │ - Stop audio │
                    │ - Save final │
                    │ - Close sess │
                    └──────────────┘

States:
  GREETING    : Playing initial welcome message
  LISTENING   : Accepting audio input, buffering
  PROCESSING  : Audio locked, transcribing + generating response
  RESPONDING  : Playing TTS output to user
  COMPLETED   : Session ended, cleanup

Key Rules:
  - Only accept audio in LISTENING state
  - PROCESSING locks audio buffer (no new input)
  - Auto-transition back to LISTENING after response
  - Can end_call from any state
```

---

## RAG Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 RAG (Retrieval-Augmented Generation)         │
└─────────────────────────────────────────────────────────────┘

INDEXING PHASE (One-time setup):

┌──────────────────┐
│  Knowledge Base  │
│  (Bilgi.pdf)     │
│  - 50 pages      │
│  - Turkish text  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  PDF Extraction  │
│  - PyPDF2        │
│  - Page by page  │
└────────┬─────────┘
         │ Raw text
         ▼
┌──────────────────┐
│  Text Chunking   │
│  - 500 words     │
│  - 50 overlap    │
│  - ~1500 chunks  │
└────────┬─────────┘
         │ Text chunks
         ▼
┌──────────────────┐
│  Embedding Gen   │
│  - Sentence-     │
│    Transformers  │
│  - Multilingual  │
│  - 384 dims      │
└────────┬─────────┘
         │ Vector embeddings
         ▼
┌──────────────────┐
│  FAISS Index     │
│  - IndexFlatL2   │
│  - L2 distance   │
│  - Save to disk  │
└──────────────────┘


RETRIEVAL PHASE (Per query):

┌──────────────────┐
│  User Question   │
│  "Ders kaydı     │
│   nasıl yapılır?"│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Query Embedding │
│  - Same model    │
│  - 384 dims      │
└────────┬─────────┘
         │ Query vector
         ▼
┌──────────────────┐
│  FAISS Search    │
│  - Top-K = 5     │
│  - L2 distance   │
│  - <50ms         │
└────────┬─────────┘
         │ Top-K chunks with scores
         ▼
┌──────────────────┐
│  Rank & Filter   │
│  - Score thresh  │
│  - Deduplication │
│  - Max tokens    │
└────────┬─────────┘
         │ Relevant context
         ▼
┌──────────────────┐
│  Context Inject  │
│  "Knowledge:     │
│   [chunk1]       │
│   [chunk2]       │
│   Question: ..." │
└────────┬─────────┘
         │
         ▼
    To LLM Query
```

---

## Database Schema

```
┌──────────────────────────────────────────────────────────────┐
│                    DATABASE SCHEMA (SQLite)                   │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│         call_sessions                    │
├─────────────────────────────────────────┤
│ PK │ id              INTEGER             │
│ UK │ session_id      TEXT (UUID)         │
│  I │ phone_number    TEXT                │
│  I │ tc_number       TEXT (nullable)     │
│  I │ call_start_time TIMESTAMP           │
│    │ call_end_time   TIMESTAMP (null)    │
│    │ status          TEXT                │
│    │                 (active/completed)  │
│    │ audio_path      TEXT (nullable)     │
│    │ created_at      TIMESTAMP           │
└─────┬───────────────────────────────────┘
      │
      │ 1:N relationship
      │
      ▼
┌─────────────────────────────────────────┐
│       conversation_turns                 │
├─────────────────────────────────────────┤
│ PK │ id              INTEGER             │
│ FK │ session_id      TEXT                │
│    │ turn_number     INTEGER             │
│  I │ topic_label     TEXT                │
│ FT │ user_question   TEXT                │
│ FT │ llm_response    TEXT                │
│  I │ timestamp       TIMESTAMP           │
└─────────────────────────────────────────┘

Indexes:
  - idx_session_id (call_sessions.session_id)
  - idx_phone_number (call_sessions.phone_number)
  - idx_tc_number (call_sessions.tc_number)
  - idx_call_start_time (call_sessions.call_start_time)
  - idx_conv_session_id (conversation_turns.session_id)
  - idx_conv_timestamp (conversation_turns.timestamp)

Full-Text Search:
  - user_question (FTS5)
  - llm_response (FTS5)

Query Patterns:
  1. Find sessions by phone: O(log n) via index
  2. Get session with turns: JOIN + sort by turn_number
  3. Search conversations: FTS5 query
  4. Topic analytics: GROUP BY topic_label
  5. Time-range queries: INDEX on timestamp
```

---

## Deployment Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    PRODUCTION DEPLOYMENT                      │
└──────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               University Network (On-Premises)               │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              Application Server (Ubuntu)               │ │
│  │                                                        │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  Python 3.10 Virtual Environment                 │ │ │
│  │  │                                                   │ │ │
│  │  │  ┌────────────────────────────────────────────┐  │ │ │
│  │  │  │  Flask-SocketIO Server (Port 5000)        │  │ │ │
│  │  │  │  - Gunicorn (4 workers)                   │  │ │ │
│  │  │  │  - eventlet async                         │  │ │ │
│  │  │  └────────────────────────────────────────────┘  │ │ │
│  │  │                                                   │ │ │
│  │  │  ┌────────────────────────────────────────────┐  │ │ │
│  │  │  │  Ollama Service (Port 11434)              │  │ │ │
│  │  │  │  - LLaMA 3.1 (8B)                         │  │ │ │
│  │  │  │  - GPU acceleration                       │  │ │ │
│  │  │  └────────────────────────────────────────────┘  │ │ │
│  │  │                                                   │ │ │
│  │  │  ┌────────────────────────────────────────────┐  │ │ │
│  │  │  │  SQLite Database                          │  │ │ │
│  │  │  │  - /var/lib/call-center/call_center.db   │  │ │ │
│  │  │  │  - WAL mode enabled                       │  │ │ │
│  │  │  └────────────────────────────────────────────┘  │ │ │
│  │  │                                                   │ │ │
│  │  │  ┌────────────────────────────────────────────┐  │ │ │
│  │  │  │  File Storage                             │  │ │ │
│  │  │  │  - /var/lib/call-center/recordings/      │  │ │ │
│  │  │  │  - MP3 audio files                        │  │ │ │
│  │  │  │  - Automatic rotation (30 days)           │  │ │ │
│  │  │  └────────────────────────────────────────────┘  │ │ │
│  │  │                                                   │ │ │
│  │  └───────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  │  GPU: NVIDIA GPU with CUDA support (12GB+ VRAM)       │ │
│  │  RAM: 32GB+ recommended                               │ │
│  │  CPU: Multi-core processor (8+ cores)                │ │
│  │  Storage: SSD recommended for performance             │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │               Nginx Reverse Proxy                      │ │
│  │  - SSL/TLS termination                                │ │
│  │  - WebSocket upgrade handling                         │ │
│  │  - Static file serving                                │ │
│  │  - Request logging                                    │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────┬───────────────────────────────────────┘
                       │
                       │ HTTPS/WSS
                       │
                   ┌───▼────┐
                   │ Users  │
                   └────────┘

Security Measures:
  - Firewall (ufw): Only ports 80, 443 open
  - SSL certificate (Let's Encrypt)
  - Environment variables for secrets
  - Regular security updates
  - Database encryption at rest
  - Audit logging
```

---

## Performance Optimization Strategies

```
┌──────────────────────────────────────────────────────────────┐
│                  PERFORMANCE OPTIMIZATIONS                    │
└──────────────────────────────────────────────────────────────┘

1. GPU Acceleration (5x improvement):
   ┌────────────────────────────────────────┐
   │  CPU Mode          GPU Mode (CUDA)     │
   ├────────────────────────────────────────┤
   │  2-3 seconds       400-500ms           │
   │  (slower-whisper)  (faster-whisper)    │
   │  int8 quantize     float16 precision   │
   └────────────────────────────────────────┘

2. Audio Buffering Strategy:
   ┌────────────────────────────────────────┐
   │  Chunked Processing (1024 samples)     │
   │  - Continuous capture                  │
   │  - Ring buffer                         │
   │  - VAD per chunk                       │
   │  - Batch transcription                 │
   └────────────────────────────────────────┘

3. Database Indexing:
   ┌────────────────────────────────────────┐
   │  Without Index     With Index          │
   ├────────────────────────────────────────┤
   │  O(n) scan         O(log n) lookup     │
   │  500ms (1000 rows) 5ms (1000 rows)     │
   └────────────────────────────────────────┘

4. RAG Caching:
   ┌────────────────────────────────────────┐
   │  Build index once at startup           │
   │  - Persist to disk (FAISS)             │
   │  - Load in memory                      │
   │  - No rebuild per query                │
   │  - 50ms search vs 5s rebuild           │
   └────────────────────────────────────────┘

5. TTS Speed Optimization:
   ┌────────────────────────────────────────┐
   │  Original Speed    1.2x Speed          │
   ├────────────────────────────────────────┤
   │  3 seconds         2.5 seconds         │
   │  (normal speech)   (faster, clear)     │
   └────────────────────────────────────────┘

6. Context Window Management:
   ┌────────────────────────────────────────┐
   │  Keep only last 10 turns               │
   │  - Reduce LLM token count              │
   │  - Faster inference                    │
   │  - Maintain relevance                  │
   └────────────────────────────────────────┘
```

---
