# Quran Voice Tutor - Project Context & Coordination Guide

**Last Updated**: 2025-10-18 (GitHub Integration Complete)

## Quick Start for Claude

When the user asks to work on the "quran app project", reference this file first, then:

1. **Check Backend Documentation**:
   - API Specs: https://public2.asimo.io/QURAN_API_DOCUMENTATION.md
   - Changelog: https://public3.asimo.io/CHANGELOG.md

2. **Review Current State** (see sections below)

3. **Ask user**: "What would you like to work on today?"

---

## Project Overview

**Application**: Quran recitation training app with AI-powered voice coaching and evaluation
**Platform**: Flutter (iOS, Android, macOS, Web)
**Backend**: FastAPI on Ubuntu server at https://quran.asimo.io

### Key Features
- **Voice Tab**: Real-time WebRTC conversation with AI Quran coach (OpenAI Realtime API)
- **Practice Tab**: Record recitation → STT transcription → Evaluation with WER metrics → Tajweed rules detection

---

## Backend-Frontend Coordination System

### Documentation URLs (Always Check These First!)

**PRIMARY - FastAPI Passthrough Endpoints (Most Reliable)**:
- **Backend API Documentation**: https://quran.asimo.io/public/api_doc
  - Complete current API specifications
  - All endpoints, request/response formats
  - Updated every 5 minutes via auto-sync
  - Proper content-type headers and CORS support

- **Backend Changelog**: https://quran.asimo.io/public/changelog
  - Chronological record of all backend changes
  - Frontend impact notes for each change
  - Breaking changes clearly marked with ⚠️

- **Backend Project Context**: https://quran.asimo.io/public/context
  - Backend development context and architecture

**GitHub Documentation Repository**: https://github.com/mohammednazmy/quran-voice-tutor-docs

**Alternative Access Methods**:

*Raw GitHub URLs* (For version control/history):
- https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/BACKEND_API_DOCUMENTATION.md
- https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/BACKEND_CHANGELOG.md
- https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/PROJECT_CONTEXT.md

*Legacy URLs* (Deprecated but still active):
- https://public2.asimo.io/QURAN_API_DOCUMENTATION.md
- https://public3.asimo.io/CHANGELOG.md

### Coordination Protocol
1. Backend Claude on Ubuntu makes changes
2. Backend Claude updates documentation files and pushes to GitHub
3. User notifies Mac Claude (or Mac Claude periodically checks)
4. Mac Claude fetches updates via raw GitHub URLs and adapts frontend
5. **Mac Claude automatically syncs documentation after each session using `./scripts/sync_frontend_docs.sh`**
6. Both Claude instances can access each other's latest documentation via GitHub

**Backend Contact**: Claude Console on Ubuntu server (user can relay messages)

### Automatic Documentation Sync
**Script**: `scripts/sync_frontend_docs.sh`

**Usage**: `./scripts/sync_frontend_docs.sh "commit message"`

**What it does**:
- Automatically commits documentation changes (PROJECT_CONTEXT.md, FRONTEND_CHANGELOG.md, README.md, docs/)
- Pulls latest changes from remote (in case backend pushed updates)
- Pushes to GitHub so backend Claude can immediately access updates
- Handles merge conflicts gracefully

**When to run**: At the end of every work session (see session checklist below)

---

## Development Environment

### Frontend (This Machine - Mac)
- **Project Path**: `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter`
- **Flutter SDK**: Installed via Homebrew
- **IDE**: Works with VS Code or Xcode
- **Git Repository**: https://github.com/mohammednazmy/quran-voice-tutor
- **Branch**: main

### Backend (Ubuntu Server)
- **URL**: https://quran.asimo.io
- **Stack**: FastAPI (Python) on port 5056, Apache2 reverse proxy
- **Data Path**: `/opt/quran-rtc/data/quran.json`
- **Audio Path**: `/opt/quran-rtc/audio/{surah}/{ayah}.{ext}`

---

## iOS Deployment Setup

### Bundle Configuration
- **Bundle ID**: `com.nazmy.quranvoicetutor`
- **Team ID**: `2F2XVQPM5R`
- **Developer**: mohammed.nazmy@gmail.com (B469KZC2JT)

### Physical Device Testing
- **Device**: iPhone XR (Ausimov-iOS)
- **UDID**: `00008020-0014188A3EA3002E`
- **iOS Version**: 18.5 beta (22F76)
- **Connection**: USB (wireless has pairing issues with iOS 18.5 beta + Xcode 16.4 beta)

### Known Deployment Issue & Workaround
**Problem**: Xcode CoreDevice framework shows device as "unavailable" with iOS 18.5 beta
**Root Cause**: iOS 18.5 beta + Xcode 16.4 beta pairing incompatibility
**Solution**: Use `ios-deploy` with legacy pairing mechanism

### Deployment Commands

#### Debug Build (crashes due to missing debugger support):
```bash
flutter build ios --debug
ios-deploy --id 00008020-0014188A3EA3002E --bundle build/ios/iphoneos/Runner.app --debug --no-wifi
```

#### Release Build (WORKING METHOD):
```bash
flutter build ios --release
ios-deploy --id 00008020-0014188A3EA3002E --bundle build/ios/iphoneos/Runner.app --justlaunch --no-wifi
```

**Note**: Release builds work because they don't require debugger connection. Use this for physical device testing.

---

## Current Architecture

### API Endpoints in Use

#### Core Data
- `GET /health` - Health check
- `GET /surahs` - List all surahs (Practice tab)
- `GET /verses?surah={id}` - Get verses for surah (Practice tab)
- `GET /canon?verse_id={surah}:{ayah}` - Get canonical text

#### Evaluation
- `POST /evaluate` - Text-based evaluation
- `POST /evaluate_audio` - Audio transcription + evaluation (Practice tab)
  - Multipart form: `file` (audio), `verse_id` (surah:ayah), `user_id`
  - Returns: WER stats, word-level diff, tajweed rules, tips, timed transcript

#### Audio Management
- `POST /audio` - Upload canonical audio
- `GET /audio?verse_id={surah}:{ayah}` - Get reference audio (Practice tab)

#### Real-Time WebRTC
- `POST /rtc/offer` - Proxy mode (Voice tab)
- `POST /rtc/offer_ephemeral` - Ephemeral proxy mode (Voice tab)
- `GET /realtime/ephemeral` - Get short-lived OpenAI client secret (Voice tab)
- `GET /persona` - Get AI coach instructions (Voice tab)

### Data Models

**Verse ID Format**: `"{surah}:{ayah}"` (e.g., "1:1" for Al-Fatiha verse 1)

**Evaluation Response**:
```json
{
  "stats": {
    "wer": 0.15,
    "sub": 1,
    "ins": 0,
    "del": 2
  },
  "ops": [
    {"type": "ok", "ref": "word", "hyp": "word"},
    {"type": "sub", "ref": "wrong", "hyp": "substitute"},
    {"type": "del", "ref": "missing", "hyp": ""},
    {"type": "ins", "ref": "", "hyp": "extra"}
  ],
  "tips": ["Tip 1", "Tip 2"],
  "rules": [
    {"rule": "Idgham", "kind": "with ghunnah", "context": "example", "char": "ن"}
  ],
  "timings": {
    "words": [
      {"text": "word", "start": 0.5, "end": 0.8}
    ]
  }
}
```

---

## Flutter App Structure

### Main File
`/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/lib/main.dart`

### Configuration
```dart
const backendBase = 'https://quran.asimo.io';
const userId = 'nazmy-dev';
```

### App Structure
```
HomeShell (StatefulWidget)
├── AppBar with ephemeral toggle switch
└── TabBarView
    ├── VoiceTab (WebRTC realtime conversation)
    │   ├── Mic permission request
    │   ├── PeerConnection setup
    │   ├── Data channel ('oai-events')
    │   ├── SDP offer/answer exchange
    │   ├── Two modes:
    │   │   ├── Direct: OpenAI ephemeral → direct to api.openai.com
    │   │   └── Proxy: Regular → through quran.asimo.io
    │   └── Audio playback via RTCVideoRenderer
    │
    └── PracticeTab (Record & evaluate)
        ├── Surah/Verse selection dropdowns
        ├── Duration slider (3-10s)
        ├── Record button → AudioRecorder (WAV, 16kHz)
        ├── Upload to /evaluate_audio
        ├── Display:
        │   ├── WER metrics
        │   ├── Color-coded diff (green/orange/red/blue)
        │   ├── Tajweed rules list
        │   ├── Coach tips
        │   └── Timed transcript with playback sync
        └── A/B playback: User take vs canonical audio
```

### Key Dependencies
```yaml
dependencies:
  flutter_webrtc: ^latest  # WebRTC for Voice tab
  http: ^latest           # API calls
  record: ^latest         # Audio recording (Practice tab)
  just_audio: ^latest     # Audio playback (Practice tab)
  path_provider: ^latest  # Temp file storage
```

### iOS Permissions (Info.plist)
```xml
<key>NSMicrophoneUsageDescription</key>
<string>We need mic access for Qur'an recitation practice.</string>
```

---

## Current Implementation Status

### ✅ Completed Features

**Voice Tab**:
- Real-time WebRTC connection to OpenAI Realtime API
- Ephemeral token support for direct OpenAI connection
- Proxy mode fallback through backend
- Automatic SDP optimization (OPUS-only, max-bundle, unified-plan)
- ICE candidate gathering with timeout
- Microphone mute/unmute
- Speaker audio output
- Arabic greeting and coaching instructions
- Multiple fallback endpoints for resilience

**Practice Tab**:
- Surah and verse selection
- Configurable recording duration (3-10s)
- WAV audio recording at 16kHz
- Multipart file upload to backend
- WER statistics display
- Color-coded word-level diff visualization
- Tajweed rules display with Arabic characters
- Coach tips display
- Timed transcript with word-level timestamps
- Synchronized playback highlighting
- A/B comparison: user recording vs canonical audio
- Automatic canonical audio fetching

**iOS Deployment**:
- Code signing configured
- Bundle identifier changed from default
- Release build deployment via ios-deploy
- App successfully running on physical iPhone

### 🚧 Known Issues

1. **iOS 18.5 Beta + Xcode 16.4 Beta Pairing**:
   - CoreDevice framework fails to pair
   - Device shows as "unavailable" in Xcode
   - **Workaround**: Use ios-deploy with release builds

2. **Debug Builds on Physical Device**:
   - Debug builds crash immediately (missing debugger support)
   - **Workaround**: Always use release builds for testing

### 📋 Potential Enhancements (Not Yet Implemented)

- User authentication & profiles
- Progress tracking across sessions
- Historical performance charts
- Offline mode / caching
- Push notifications for practice reminders
- Social features (leaderboards, sharing)
- Custom recitation goals
- Multiple reciter audio sources
- Playback speed control
- Loop practice mode
- Dark mode theme
- Accessibility improvements
- Internationalization (currently Arabic + English)

---

## Testing & Development Workflow

### Local Development
```bash
# Test on macOS
flutter run -d macos

# Test in Chrome (web)
flutter run -d chrome

# Test in iOS Simulator
flutter run -d 'iPhone 16 Plus'
```

### Physical iPhone Testing
```bash
# 1. Build release version
flutter build ios --release

# 2. Deploy to device
ios-deploy --id 00008020-0014188A3EA3002E \
  --bundle build/ios/iphoneos/Runner.app \
  --justlaunch --no-wifi

# 3. If "Untrusted Developer" error on iPhone:
#    Settings → General → VPN & Device Management → Trust developer
```

### Checking Backend Status
```bash
# Health check
curl https://quran.asimo.io/health

# List surahs
curl https://quran.asimo.io/surahs

# Get verses for Al-Fatiha (surah 1)
curl 'https://quran.asimo.io/verses?surah=1'
```

---

## Troubleshooting Guide

### App Won't Deploy to iPhone
1. Check device is unlocked and connected via USB
2. Verify Developer Mode is enabled (Settings → Privacy & Security → Developer Mode)
3. Trust computer on iPhone if prompted
4. Use release build, not debug build
5. Kill any existing ios-deploy processes: `killall -9 ios-deploy`

### App Crashes on Launch
- If debug build: rebuild as release (`flutter build ios --release`)
- Check iPhone logs via Xcode → Devices & Simulators

### WebRTC Connection Fails (Voice Tab)
1. Check mic permissions granted
2. Verify backend is responding: `curl https://quran.asimo.io/health`
3. Try toggling between ephemeral and proxy modes
4. Check browser console (if testing web) or Xcode logs (if iOS)

### Audio Recording Fails (Practice Tab)
1. Verify mic permissions granted on device
2. Check app is not in web mode (web doesn't support native recording)
3. Ensure sufficient storage space on device

### Backend Returns Errors
1. Check backend documentation for API changes: https://quran.asimo.io/public/api_doc
2. Review changelog for breaking changes: https://quran.asimo.io/public/changelog
3. Contact backend Claude via user for clarification

---

## Communication Protocols

### When Backend Changes
1. User will notify you: "The backend has been updated"
2. You should:
   - Fetch https://quran.asimo.io/public/changelog
   - Review what changed and frontend impact
   - Update Flutter code accordingly
   - Test the changes
   - Push frontend changes to GitHub

### When Frontend Needs Backend Changes
1. Document the required API changes clearly
2. User will relay to backend Claude
3. Wait for backend to implement and update docs
4. Verify changes via changelog
5. Proceed with frontend implementation

### Questions for Backend
Format questions clearly with:
- Endpoint path
- Request/response format needed
- Use case / why it's needed
- Any error handling requirements

---

## File Structure Reference

```
/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/
├── lib/
│   └── main.dart              # Main app code (605 lines)
├── ios/
│   ├── Runner/
│   │   ├── Info.plist         # iOS permissions & config
│   │   └── Runner.xcodeproj/
│   │       └── project.pbxproj # Bundle ID, signing config
│   ├── Runner.xcworkspace/    # Xcode workspace
│   └── Podfile                # CocoaPods dependencies
├── android/                   # Android config
├── macos/                     # macOS config
├── web/                       # Web config
├── build/                     # Build outputs (gitignored)
│   └── ios/iphoneos/Runner.app # Release build for deployment
├── pubspec.yaml              # Flutter dependencies
└── PROJECT_CONTEXT.md        # This file!
```

---

## Quick Reference Commands

### Build & Deploy
```bash
# Release build
flutter build ios --release

# Deploy to iPhone
ios-deploy --id 00008020-0014188A3EA3002E --bundle build/ios/iphoneos/Runner.app --justlaunch --no-wifi

# Run on simulator
flutter run -d 'iPhone 16 Plus'

# Run on macOS
flutter run -d macos
```

### Flutter Development
```bash
# Clean build
flutter clean && flutter pub get

# Update dependencies
flutter pub upgrade

# Analyze code
flutter analyze

# Format code
flutter format lib/
```

### Device Management
```bash
# List all devices
flutter devices

# Check device via devicectl
xcrun devicectl list devices

# Check via libimobiledevice
idevice_id -l
idevicepair validate
```

---

## Important Notes for Future Sessions

1. **Always check the documentation URLs first** before making changes
2. **Use release builds** for physical iPhone testing
3. **Backend base is HTTPS** (not HTTP)
4. **Verse IDs use colon format** "surah:ayah" (e.g., "1:7")
5. **User ID is hardcoded** as "nazmy-dev" for now
6. **WebRTC uses OPUS codec** (optimized SDP)
7. **Audio format for recording** is WAV, 16kHz, mono
8. **The app is working** on physical iPhone as of last session

---

## Contact & Resources

- **Frontend Repository**: https://github.com/mohammednazmy/quran-voice-tutor
- **Backend Documentation Repository**: https://github.com/mohammednazmy/quran-voice-tutor-docs
- **Backend API Docs**: https://quran.asimo.io/public/api_doc (Primary) / GitHub (Backup)
- **Backend Changelog**: https://quran.asimo.io/public/changelog (Primary) / GitHub (Backup)
- **Backend Context**: https://quran.asimo.io/public/context
- **Backend URL**: https://quran.asimo.io
- **Developer Email**: mohammed.nazmy@gmail.com
- **Backend Claude**: Accessible via user on Ubuntu server

---

## Session Checklist

### At the START of each session, Claude should:

- [ ] Read this PROJECT_CONTEXT.md file
- [ ] Read FRONTEND_CHANGELOG.md to see recent changes
- [ ] Fetch https://quran.asimo.io/public/changelog to check for backend updates
- [ ] If backend changes detected, fetch https://quran.asimo.io/public/api_doc
- [ ] Review the "Current Implementation Status" section above
- [ ] Check FRONTEND_CHANGELOG.md "Next Session TODO" for pending work
- [ ] Ask user: "What would you like to work on today?"

### At the END of each session, Claude should:

- [ ] Update FRONTEND_CHANGELOG.md with all changes made during the session
- [ ] Update PROJECT_CONTEXT.md "Current Implementation Status" if features changed
- [ ] Update PROJECT_CONTEXT.md "Last Updated" date at the top
- [ ] Update FRONTEND_CHANGELOG.md "Session Tracking" section
- [ ] Add any pending work to "Next Session TODO" in changelog
- [ ] **IMPORTANT**: Run `./scripts/sync_frontend_docs.sh "Session summary"` to push documentation to GitHub
- [ ] Verify push succeeded so backend Claude can access updates
- [ ] Summarize what was accomplished

---

**Remember**: This is a coordinated frontend-backend project. When in doubt about API behavior, check the docs or ask the user to relay questions to the backend Claude!
