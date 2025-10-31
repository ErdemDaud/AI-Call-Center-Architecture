# AI Call Center System

> ⚠️ **Confidentiality Notice**: Source code is proprietary to Çukurova University and cannot be shared. This repository contains architecture documentation only.

---

## 🎯 Project Overview

**Role:** Lead AI Engineer
**Client:** Çukurova University - Student Affairs Department
**Status:** Deployed to Production

Real-time AI voice assistant that automates student inquiry calls using speech recognition, LLM conversation, and knowledge base retrieval.

### Impact
- **100+ daily calls** automated
- **60% reduction** in call handling time
- **95%+ accuracy** in Turkish transcription
- **3-7 seconds** response time per turn
- **99%+ uptime** in production

---

## 🏗️ Architecture

```
Voice Input → Speech-to-Text → LLM + Knowledge Base → Text-to-Speech → Response
             (GPU Whisper)     (LLaMA 3.1 + RAG)      (gTTS)
```

**Full diagrams:** [ARCHITECTURE.md](ARCHITECTURE.md)

---

## 💻 Technology Stack

**AI/ML:**
- faster-whisper (GPU-accelerated speech recognition)
- LLaMA 3.1 via Ollama (language model)
- FAISS + Sentence-Transformers (RAG semantic search)
- WebRTC VAD (voice activity detection)
- Google TTS (text-to-speech)

**Backend:**
- Python, Flask, Flask-SocketIO
- WebSocket (real-time communication)
- SQLite (database with full-text search)
- CUDA (GPU acceleration)

---

## 🚀 Key Achievements

### 1. Performance Optimization (5x Improvement)
- Migrated to GPU-accelerated Whisper
- Reduced latency from 2-3s to 400-500ms
- **Result:** 5x faster speech recognition

### 2. RAG Implementation (90% Accuracy)
- Built semantic search with FAISS
- Indexed 50-page knowledge base
- **Result:** Answer accuracy improved from 70% to 90%

### 3. Real-Time Architecture
- WebSocket for bidirectional communication
- State machine for session management
- **Result:** Handles 10+ concurrent sessions

---

## 🛠️ Skills Demonstrated

**Software Engineering:**
- System architecture design
- Real-time systems development
- WebSocket protocol
- State machine implementation
- Database design

**AI/ML Engineering:**
- Speech recognition integration
- LLM deployment
- RAG pipeline implementation
- GPU optimization (CUDA)
- Prompt engineering

**Problem Solving:**
- Latency optimization
- Concurrent session management
- Audio streaming challenges
- Production deployment

---

## 📚 Documentation

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - System architecture diagrams
- **[TECHNICAL_DETAILS.md](TECHNICAL_DETAILS.md)** - Implementation details
- **[INTERVIEW_GUIDE.md](INTERVIEW_GUIDE.md)** - Interview preparation guide

---

## 💼 Project Capabilities

This project demonstrates:
- ✅ Complex AI system architecture
- ✅ Cutting-edge technology integration
- ✅ Production-level performance optimization
- ✅ Measurable business impact delivery
- ✅ Enterprise-scale system deployment

---

## 📞 Contact

**Erdem Daud**
AI Engineer | Backend Developer

- 📧 erdemindir@gmail.com
- 💼 [LinkedIn](www.linkedin.com/in/erdem-daud)
- 🐙 [GitHub](https://github.com/ErdemDaud)

---

**Keywords:** AI, Machine Learning, LLM, Speech Recognition, Real-time Systems, WebSocket, Python, CUDA, Voice Assistant, RAG, FAISS
