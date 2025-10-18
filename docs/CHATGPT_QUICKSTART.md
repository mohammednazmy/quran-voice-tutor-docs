# ChatGPT Guide - Quran Voice Tutor Project Documentation

This guide helps ChatGPT (or any AI assistant) quickly understand and access all documentation for the Quran Voice Tutor project.

---

## 🎯 Quick Access URLs

### PRIMARY - FastAPI Passthrough Endpoints (Use These First!)

**Backend Documentation**:
- **API Documentation**: https://quran.asimo.io/public/api_doc
  - Complete API specifications for all 12 endpoints
  - Request/response formats
  - Data models and examples
  - 31KB of detailed documentation

- **Backend Changelog**: https://quran.asimo.io/public/changelog
  - Chronological record of all backend changes
  - Frontend impact notes
  - Breaking changes marked with ⚠️
  - Recent updates on tajweed tokenization and multi-ayah evaluation

- **Backend Project Context**: https://quran.asimo.io/public/context
  - Backend architecture and design decisions
  - Technology stack details
  - Development environment info

**Frontend Documentation**:
- **Frontend Repository**: https://github.com/mohammednazmy/quran-voice-tutor
- **Frontend Context**: Available in repository at `PROJECT_CONTEXT.md`
- **Frontend Changelog**: Available in repository at `FRONTEND_CHANGELOG.md`
- **README**: Available in repository at `README.md`

### Why These URLs Are Best

1. **Reliability**: Served directly by FastAPI backend
2. **Speed**: Faster than GitHub raw URLs
3. **Fresh**: Auto-synced every 5 minutes
4. **CORS**: Proper headers for browser access
5. **Content-Type**: Correct `text/markdown` headers

---

## 📚 Project Overview

### What Is This Project?

An AI-powered Quranic recitation training app with:
- Real-time voice coaching using OpenAI Realtime API
- Automatic pronunciation evaluation
- Tajweed rule detection and color-coding
- Multi-platform support (iOS, Android, macOS, Web)

### Technology Stack

**Backend**:
- FastAPI (Python 3.12)
- OpenAI Realtime API / Whisper
- WebRTC for real-time audio
- Hosted at: https://quran.asimo.io

**Frontend**:
- Flutter (cross-platform)
- Material Design 3
- WebRTC support
- Repository: https://github.com/mohammednazmy/quran-voice-tutor

---

## 🔧 How Documentation Works

### Auto-Sync System

1. **Backend Updates**: Backend Claude updates documentation files
2. **5-Minute Sync**: Auto-sync copies files to multiple locations
3. **FastAPI Serves**: Documentation available via `/public/*` endpoints
4. **Frontend Access**: Frontend Claude fetches updates
5. **GitHub Backup**: All documentation also pushed to GitHub

### Documentation Flow

```
Backend Changes
    ↓
Update Docs Locally
    ↓
5-Min Auto-Sync → FastAPI Endpoints (PRIMARY)
    ↓              ↓
GitHub Repos    Apache Alias URLs (backup)
    ↓
Frontend Claude Reads & Updates
    ↓
Frontend Changes Pushed to GitHub
```

---

## 📖 How to Use This Documentation

### For Understanding the API

1. **Start here**: https://quran.asimo.io/public/api_doc
2. Check the "Table of Contents" section
3. Look for endpoint categories:
   - Core Data (surahs, verses, canonical text)
   - Evaluation (text and audio evaluation)
   - Audio Management (canonical audio)
   - Real-Time Communication (WebRTC/Voice)

### For Understanding Recent Changes

1. **Check changelog**: https://quran.asimo.io/public/changelog
2. Look at top entries (most recent first)
3. Pay attention to:
   - "Added" sections (new features)
   - "Changed" sections (modifications)
   - ⚠️ symbols (breaking changes)

### For Understanding Architecture

1. **Backend**: https://quran.asimo.io/public/context
2. **Frontend**: https://github.com/mohammednazmy/quran-voice-tutor/blob/main/PROJECT_CONTEXT.md

---

## 🎨 Key Features to Know About

### Voice Tab (Real-Time Coaching)
- WebRTC connection to OpenAI Realtime API
- Live conversation with AI coach
- Auto-evaluate on silence detection
- Manual ayah evaluation with instant feedback

### Read Tab (Continuous Reading)
- Full surah display with color-coded tajweed
- Silence-based auto-segmentation
- Auto-scroll and auto-advance
- Per-ayah evaluation

### Practice Tab (Single Ayah Practice)
- Record specific ayah
- Get WER (Word Error Rate) statistics
- See color-coded word diff
- View tajweed rules and coach tips
- Multi-reciter support
- Playback speed control

---

## 🔍 Common Questions

### Q: How do I find endpoint details?
**A**: Go to https://quran.asimo.io/public/api_doc and search for the endpoint name.

### Q: What changed in the backend recently?
**A**: Check https://quran.asimo.io/public/changelog - look at the top entries.

### Q: How do I see the Flutter app code?
**A**: Clone https://github.com/mohammednazmy/quran-voice-tutor and look in `lib/main.dart`.

### Q: Are there any breaking changes I should know about?
**A**: Search for "⚠️" in https://quran.asimo.io/public/changelog.

### Q: What's the base URL for API calls?
**A**: `https://quran.asimo.io` (production) or `http://127.0.0.1:5056` (local dev).

---

## 🚀 Quick Start for AI Assistants

1. **Understand the API**: Read https://quran.asimo.io/public/api_doc
2. **Check Recent Changes**: Scan https://quran.asimo.io/public/changelog
3. **See Frontend Code**: Browse https://github.com/mohammednazmy/quran-voice-tutor
4. **Ask Questions**: All the context you need is in these three documents

---

## 🔗 All Available URLs (For Reference)

### Primary (Recommended)
- https://quran.asimo.io/public/api_doc
- https://quran.asimo.io/public/changelog
- https://quran.asimo.io/public/context

### GitHub (Backup)
- https://github.com/mohammednazmy/quran-voice-tutor (frontend code)
- https://github.com/mohammednazmy/quran-voice-tutor-docs (backend docs)
- https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/BACKEND_API_DOCUMENTATION.md
- https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/BACKEND_CHANGELOG.md

### Legacy (Deprecated)
- https://public2.asimo.io/QURAN_API_DOCUMENTATION.md
- https://public3.asimo.io/CHANGELOG.md

---

## 📞 Need Help?

- **Developer**: mohammed.nazmy@gmail.com
- **Backend**: https://quran.asimo.io
- **Issues**: https://github.com/mohammednazmy/quran-voice-tutor/issues

---

**Last Updated**: 2025-10-18
**Documentation Version**: 1.0.0
**Auto-Sync Status**: ✅ Active (5-minute interval)
