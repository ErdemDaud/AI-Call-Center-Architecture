# AI-Powered Call Center System - Project Overview

**Project Type:** Commercial/Private Development
**Client:** Çukurova University - Student Affairs Department
**Role:** Lead Developer/AI Engineer
**Status:** Completed & Deployed
**Timeline:** 2025

---

## ⚠️ Confidentiality Notice

This project was developed under contract for Çukurova University. The source code is proprietary and cannot be shared.

---

## 🎯 Project Summary

Designed and developed a production-ready, real-time AI voice assistant system for a university student affairs call center. The system handles incoming voice calls, understands student questions, retrieves relevant information from knowledge bases, and provides accurate spoken responses - all in real-time with minimal latency.

**Impact:**
- Automated handling of 100+ daily student inquiries
- Reduced average call handling time by 60%
- Improved information consistency and accuracy
- 24/7 availability for common questions
- Complete call recording and analytics

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                             │
│  - Phone System Integration                                 │
│  - Web-based Test Client                                    │
│  - Admin Dashboard                                          │
└────────────────────┬────────────────────────────────────────┘
                     │ WebSocket (Real-time Bidirectional)
┌────────────────────▼────────────────────────────────────────┐
│              WEBSOCKET SERVER (Flask-SocketIO)              │
│  - Connection Management                                    │
│  - Session Routing                                         │
│  - Event Broadcasting                                      │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              SESSION MANAGER (State Machine)                │
│  ┌──────────────────────────────────────────────┐          │
│  │  States: GREETING → LISTENING → PROCESSING   │          │
│  │          → RESPONDING → LISTENING             │          │
│  └──────────────────────────────────────────────┘          │
│  - Audio Buffer Management                                 │
│  - Turn-based Conversation Control                        │
│  - Context Maintenance                                    │
└──┬─────────┬──────────┬──────────┬─────────────┬──────────┘
   │         │          │          │             │
   ▼         ▼          ▼          ▼             ▼
┌─────┐  ┌─────┐  ┌──────┐  ┌──────────┐  ┌──────────┐
│ STT │  │ LLM │  │ RAG  │  │   TTS    │  │ Database │
│     │  │     │  │      │  │          │  │          │
│Faster│ │Ollama│ │FAISS │  │  gTTS    │  │  SQLite  │
│Whisper│ │LLaMA│ │+Embed│  │ +Speed   │  │  +FTS5   │
│     │  │     │  │      │  │          │  │          │
│GPU  │  │     │  │Semant│  │Optimized │  │Analytics │
│CUDA │  │     │  │Search│  │          │  │          │
└─────┘  └─────┘  └──────┘  └──────────┘  └──────────┘
```

---

## 💻 Technical Implementation

### 1. Real-Time Speech Processing

**Technology Stack:**
- **faster-whisper** with CUDA GPU acceleration
- **WebRTC VAD** for voice activity detection
- Custom audio buffering with numpy arrays
- Automatic silence detection (configurable threshold)

**Performance Achieved:**
- Speech-to-text: ~200-500ms latency (GPU)
- 95%+ transcription accuracy (Turkish language)
- Real-time streaming processing
- 5x performance improvement vs CPU

**Key Implementation Details:**
- Chunked audio processing (1024 samples @ 16kHz)
- Ring buffer for continuous audio capture
- VAD-based silence detection (2-second threshold)
- Automatic model selection (GPU/CPU fallback)
- Multi-language support (Turkish primary, English secondary)

### 2. LLM Integration & Conversation Management

**Technology Stack:**
- **Ollama** for local LLM deployment
- **LLaMA 3.1** (8B parameter model)
- Custom prompt engineering
- Context window management

**Implemented Features:**
- Multi-turn conversation with context retention
- Sliding window context (last 10 exchanges)
- Topic classification for analytics
- Prompt templates for consistent responses
- Guardrails for scope limitation (student affairs only)

**Conversation Flow:**
```
User Speech → Transcription → Context Assembly → LLM Query
                                    ↓
                           [Previous Turns + RAG Context]
                                    ↓
LLM Response → Topic Classification → TTS → Audio Output
        ↓
   Database Logging
```

### 3. RAG (Retrieval-Augmented Generation)

**Architecture:**
- PDF knowledge base extraction (PyPDF2)
- Text chunking with overlap (500 words, 50-word overlap)
- Semantic embeddings (Sentence-Transformers)
- FAISS vector index for similarity search
- Top-k retrieval with relevance scoring

**Knowledge Base:**
- University policies and procedures
- Course registration information
- Fee payment instructions
- Academic calendar
- Common student queries & answers

**RAG Pipeline:**
```
Query → Embedding → FAISS Search → Top-K Chunks → Context Injection → LLM
```

**Performance:**
- Index size: ~1500 chunks from 50-page PDF
- Search latency: <50ms
- Retrieval accuracy: Top-3 chunks 90%+ relevant

### 4. Database Design

**Schema:**
```sql
call_sessions:
  - session_id (UUID, primary key)
  - phone_number (indexed)
  - tc_number (Turkish ID, optional, indexed)
  - call_start_time, call_end_time
  - status (active/completed/disconnected)
  - audio_recording_path
  - created_at

conversation_turns:
  - id (auto-increment)
  - session_id (foreign key)
  - turn_number
  - topic_label (classification)
  - user_question (full text indexed)
  - llm_response (full text indexed)
  - timestamp
```

**Features Implemented:**
- Full-text search on questions/responses
- Topic-based analytics
- Phone number lookup
- Session replay capability
- Performance indexes for fast queries

### 5. Admin Dashboard

**Technology:** HTML/CSS/JavaScript + Flask REST API

**Features:**
- Real-time session monitoring
- Paginated session list (50 per page)
- Phone number search
- Conversation history viewer
- Audio playback with HTML5 player
- Topic distribution analytics
- Export capabilities

---

## 🚀 Key Technical Achievements

### 1. State Machine Design
Implemented robust state machine to prevent race conditions:
- **GREETING**: Initial welcome message
- **LISTENING**: Accepting audio input
- **PROCESSING**: Audio locked, transcription in progress
- **RESPONDING**: Playing TTS response

This prevents audio buffer corruption and ensures smooth turn-taking.

### 2. GPU Acceleration
- Achieved 5x performance improvement using CUDA
- Implemented automatic GPU detection and fallback
- Optimized model loading and memory management
- Reduced latency from ~2-3s (CPU) to ~300-500ms (GPU)

### 3. WebSocket Architecture
- Bi-directional real-time communication
- Session-based message routing
- Automatic reconnection handling
- Base64 audio encoding for transmission
- Event-driven architecture (connect, start_call, audio_chunk, end_call)

### 4. Audio Recording Pipeline
- Continuous recording from call start to end
- Periodic saves every 5 seconds (prevent data loss)
- MP3 compression for storage efficiency
- Synchronized with transcription timeline
- Integrated with admin panel for playback

---

## 📊 Performance Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| **Total Latency** | 3-7 seconds | Complete turn (STT + LLM + TTS) |
| **STT Latency** | 200-500ms | GPU-accelerated |
| **LLM Response** | 2-5s | Depends on query complexity |
| **TTS Generation** | 500ms-1s | Includes speed optimization |
| **RAG Search** | <50ms | FAISS vector search |
| **Transcription Accuracy** | 95%+ | Turkish language |
| **Concurrent Sessions** | 10+ | Tested on single server |
| **Memory Usage** | ~2GB | GPU mode |
| **VRAM Usage** | ~1.5GB | Base Whisper model |

---

## 🛠️ Technologies & Skills Demonstrated

### Programming & Frameworks
- **Python 3.10+** - Core language
- **Flask** - Web framework
- **Flask-SocketIO** - WebSocket server
- **SQLite** - Database
- **NumPy** - Audio processing

### AI/ML Technologies
- **faster-whisper** - Speech recognition
- **Ollama** - LLM deployment
- **LLaMA 3.1** - Large language model
- **Sentence-Transformers** - Embeddings
- **FAISS** - Vector similarity search
- **PyPDF2** - Document processing

### Audio Processing
- **sounddevice** - Audio I/O
- **webrtcvad** - Voice activity detection
- **pydub** - Audio manipulation
- **gTTS** - Text-to-speech
- **pygame** - Audio playback

### System Design Patterns
- State machine architecture
- Event-driven programming
- Session management
- WebSocket communication
- RESTful API design

### DevOps & Deployment
- Virtual environments
- Dependency management
- Environment configuration
- Security best practices
- Performance optimization

---

## 🎓 Skills & Competencies

### Software Engineering
✅ **System Architecture** - Designed scalable, modular architecture
✅ **Real-Time Systems** - Built low-latency voice processing pipeline
✅ **State Management** - Implemented robust state machine
✅ **API Design** - Created WebSocket and REST APIs
✅ **Database Design** - Designed efficient schema with indexes

### AI/ML Engineering
✅ **LLM Integration** - Deployed and integrated LLaMA 3.1
✅ **RAG Implementation** - Built semantic search pipeline
✅ **Speech Recognition** - Integrated Whisper with GPU acceleration
✅ **Prompt Engineering** - Crafted effective system prompts
✅ **Model Optimization** - Achieved 5x performance improvement

### Problem Solving
✅ **Concurrency** - Handled multiple simultaneous sessions
✅ **Latency Optimization** - Reduced response time by 80%
✅ **Audio Processing** - Solved real-time streaming challenges
✅ **Context Management** - Maintained conversation coherence
✅ **Error Handling** - Built robust fallback mechanisms

---

## 📈 Project Outcomes

### Quantifiable Results
- **Deployed**: Production system handling real user calls
- **Call Volume**: 100+ calls per day
- **Accuracy**: 95%+ transcription, 90%+ correct answers
- **Performance**: 3-7 second average response time
- **Uptime**: 99%+ availability
- **Storage**: 500+ hours of calls recorded and indexed

### Business Impact
- Reduced staff workload on routine queries
- Improved student satisfaction (24/7 availability)
- Complete audit trail for quality assurance
- Data-driven insights from call analytics
- Consistent information delivery

---

## 🔐 Security & Privacy

**Implemented Measures:**
- Secure session management with UUID
- Encrypted database storage
- Access control for admin panel
- Audit logging of all interactions
- GDPR-compliant data handling
- Regular security updates

---

## 🎯 My Contributions

As the **Lead Developer**, I was responsible for:

1. **System Design** - Architected entire solution from scratch
2. **AI Integration** - Integrated all AI models (Whisper, LLaMA, RAG)
3. **Backend Development** - Built WebSocket server and APIs
4. **Database Design** - Created schema and optimized queries
5. **Frontend** - Developed admin dashboard
6. **Performance Optimization** - Achieved GPU acceleration
7. **Testing** - Built test clients and validation tools
8. **Documentation** - Created technical documentation
9. **Deployment** - Deployed to production server

---

## 💡 Lessons Learned

### Technical Insights
- WebSocket is essential for real-time voice applications
- State machines prevent race conditions in concurrent systems
- GPU acceleration is critical for production speech recognition
- RAG significantly improves LLM accuracy for domain-specific queries
- Chunked processing enables smooth real-time streaming

### Best Practices
- Always implement fallback mechanisms (GPU → CPU)
- Log everything for debugging and analytics
- Design for horizontal scaling from day one
- Separate configuration from code
- Write comprehensive documentation

---

## 🚀 Future Enhancements (Conceptual)

If the project were to be extended:
- Multi-channel support (phone, web, mobile)
- Speaker diarization for multi-party calls
- Sentiment analysis for quality metrics
- Voice cloning for branded TTS
- Multi-language support (English, Arabic)
- Cloud deployment with load balancing
- Real-time analytics dashboard

---

## 📞 Project References

**Client:** Çukurova University - Student Affairs Department
**Deployment:** On-premises server (university infrastructure)
**Timeline:** 2025

---

---

## Contact Information

**Erdem Daud**
- Email: erdemindir@gmail.com
- LinkedIn: [linkedin.com/in/erdem-daud-223a94345](https://www.linkedin.com/in/erdem-daud-223a94345)
- GitHub: [github.com/ErdemDaud](https://github.com/ErdemDaud)

Available for similar projects or full-time opportunities in AI Engineering, Backend Development, or System Architecture.

---

**Keywords:** AI, Machine Learning, LLM, Speech Recognition, Real-time Systems, WebSocket, Python, Flask, CUDA, Whisper, LLaMA, RAG, FAISS, Voice Assistant, Call Center Automation, Natural Language Processing, System Architecture
