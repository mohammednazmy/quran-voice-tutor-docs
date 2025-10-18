# Frontend Changelog

This file tracks all changes made to the Flutter frontend application.

---

## 2025-10-18 - FastAPI Passthrough Endpoints Integration

### Added
- **FastAPI Passthrough Endpoints**: Integrated new primary documentation URLs
  - https://quran.asimo.io/public/api_doc - Backend API documentation
  - https://quran.asimo.io/public/changelog - Backend changelog
  - https://quran.asimo.io/public/context - Backend project context
  - All endpoints verified and working correctly
  - Proper content-type headers (text/markdown) and CORS support
  - Auto-sync every 5 minutes ensures documentation stays current

### Changed
- **PRIMARY Documentation URLs**: Switched from GitHub raw URLs to FastAPI passthrough endpoints
  - More reliable than raw GitHub URLs
  - Faster response times (served directly by backend)
  - Better content-type headers for markdown parsing
  - Automatic CORS support
- **Updated PROJECT_CONTEXT.md**: All documentation references now use FastAPI URLs as primary
  - Session checklist updated
  - Troubleshooting guide updated
  - Contact & Resources section updated
- **GitHub URLs**: Now listed as backup/alternative access method

### Technical Details
- Tested all three FastAPI endpoints with WebFetch tool
- Verified content matches source files
- Confirmed markdown content-type headers
- All 31KB+ of API documentation accessible
- Backend auto-sync ensures 5-minute freshness

### Benefits
- **Reliability**: FastAPI serves files directly, no GitHub API rate limits
- **Speed**: Direct serving is faster than GitHub raw URLs
- **CORS**: Proper headers enable browser/ChatGPT access
- **Consistency**: Same backend that serves API also serves documentation
- **Fresh**: Auto-sync keeps docs updated every 5 minutes

---

## 2025-10-18 - GitHub Integration & Automatic Documentation Sync

### Added
- **GitHub Repository Setup**: Initialized Git version control for the Flutter frontend project
  - Repository URL: https://github.com/mohammednazmy/quran-voice-tutor
  - Initial commit with 100 files, 6,404 insertions
  - Configured remote tracking with upstream branch
- **Automatic Documentation Sync Script**: Created `scripts/sync_frontend_docs.sh`
  - Automatically commits documentation changes
  - Pulls latest changes from remote (handles backend updates)
  - Pushes to GitHub so backend Claude can immediately see updates
  - Handles merge conflicts gracefully
  - Usage: `./scripts/sync_frontend_docs.sh "commit message"`
  - Integrated into session checklist for automatic execution
- **README.md**: Comprehensive repository documentation including:
  - Project overview with all three tabs (Voice, Read, Practice)
  - Feature descriptions and architecture
  - Setup and installation instructions
  - iOS deployment guide
  - Configuration details
  - Dependencies and permissions
  - Development workflow
  - Known limitations
- **Documentation Coordination System**: Integrated with backend GitHub documentation repository
  - Backend docs repository: https://github.com/mohammednazmy/quran-voice-tutor-docs
  - Direct access URLs for programmatic fetching:
    - Backend API: https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/BACKEND_API_DOCUMENTATION.md
    - Backend Changelog: https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/BACKEND_CHANGELOG.md
    - Backend Context: https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/PROJECT_CONTEXT.md
  - Both Claude instances can now read each other's documentation
  - Bidirectional coordination for frontend-backend development

### Changed
- **PROJECT_CONTEXT.md**: Updated all documentation references to use GitHub URLs
  - Replaced public2/public3.asimo.io URLs with GitHub raw URLs
  - Added frontend repository information
  - Updated coordination protocol to include GitHub push workflow
  - Added GitHub repository to Contact & Resources section
  - Updated session checklist with new GitHub URLs
- **Git Status**: Repository now under version control with remote tracking

### Technical Details
- **Git Setup**:
  - Initialized with `git init`
  - Added remote: https://github.com/mohammednazmy/quran-voice-tutor
  - Branch: main (tracked to origin/main)
  - Authentication: HTTPS with Personal Access Token
- **Initial Commit**:
  - Commit message: "Initial commit: Flutter Quran Voice Tutor app with Voice, Read, and Practice tabs"
  - Includes all project files, dependencies, and platform-specific configurations
- **GitHub Push**: Successfully pushed to remote repository
- **Documentation Access**: Verified ability to read backend documentation via raw GitHub URLs

### Backend Coordination
- Backend Claude created documentation repository with automatic sync
- Both Claude instances now have persistent access to shared documentation
- Frontend can fetch backend updates programmatically
- Backend can view frontend progress via GitHub repository

### Files Modified
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/.git/` (created) - Git repository
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/README.md` (created) - Repository documentation
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/PROJECT_CONTEXT.md` (updated) - GitHub integration references
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/FRONTEND_CHANGELOG.md` (this file) - Session documentation

### Benefits
- **Version Control**: Full project history and change tracking
- **Collaboration**: Easy sharing with other developers or AI assistants
- **Documentation Sync**: Both frontend and backend documentation in one place
- **Persistent State**: Claude instances can reference documentation across sessions
- **Transparency**: All changes visible in commit history
- **Backup**: Remote repository ensures code safety

### Notes
- Legacy URLs (public2/public3.asimo.io) remain active for backward compatibility
- Future sessions should use GitHub URLs for documentation fetching
- Personal Access Token used for authentication (stored securely)
- Both repositories (frontend code + backend docs) now publicly accessible for AI tools

---

## 2025-10-18 - Added "Read" Tab with Continuous Surah Reading + Color-Coded Tajweed

### Added
- **Read Tab**: New third tab for continuous surah reading with advanced features:
  - **Full Surah Display**: Shows entire surah with color-coded tajweed tokens
  - **Silence-Based Auto-Segmentation**: Uses `noise_meter` to detect when user finishes reciting an ayah
  - **Per-Ayah Auto-Evaluation**: Automatically evaluates each ayah via `/evaluate_span` endpoint
  - **Auto-Scroll**: Automatically scrolls to current ayah being recited
  - **Auto-Advance**: Moves to next ayah after evaluation completes
  - **Visual Highlighting**: Current ayah highlighted with yellow background and golden border
  - **Tajweed Color-Coding**: Words colored by tajweed rules (15 categories) using backend palette
  - **Navigation Controls**: Prev/Next buttons for manual ayah navigation
  - **Surah Selector**: Dropdown to switch between different surahs
  - **Configurable Detection Parameters**:
    - Silence threshold (dB): -45 default (adjustable -70 to -20)
    - Hold duration: 900ms default (adjustable 300-2000ms)
    - Minimum speech duration: 1500ms default (adjustable 500-4000ms)
  - **Start/Stop Control**: Toggle listening mode on/off
  - **Auto-Scroll Toggle**: Enable/disable automatic scrolling

### Changed
- **App Structure**: Updated from 2-tab to 3-tab layout (Voice, Read, Practice)
- **Tab Order**: Voice → Read → Practice

### Backend Integration
- **New Endpoint**: `GET /surah_tokens?surah=S` - Returns full surah with:
  - Tokenized words with tajweed rule annotations
  - Color palette for 15 tajweed categories
  - Surah name and metadata
- **New Endpoint**: `POST /evaluate_span` - Evaluates multi-ayah or single-ayah recordings with:
  - Per-ayah evaluation blocks
  - Word-level alignment and timing
  - Madd duration analysis
  - Arabic coach feedback (`coach_line_ar`)
- **Tajweed Color Palette**: 15 rule categories with hex color codes:
  - Ghunna (nasal sound)
  - Idgham (merging)
  - Iqlab (conversion)
  - Ikhfa (concealment)
  - Qalqala (echoing)
  - Madd (elongation) - various types
  - Lam rules (solar/lunar)
  - Ra rules (velarized/uvularized)
  - And more specialized rules

### Technical Details
- **ReadTab State Management**:
  - Tracks current ayah index
  - Manages noise detection stream
  - Maintains GlobalKey map for scroll positioning
  - Handles recording lifecycle
- **Noise Detection Algorithm** (same as Voice tab):
  - Monitors audio levels via NoiseMeter
  - Detects speech start when above threshold
  - Triggers evaluation after silence hold duration
  - Prevents false positives with minimum speech duration
- **Color Parsing**: Converts hex color strings from backend palette to Flutter Color objects
- **ScrollController**: Ensures current ayah is visible with smooth animation (300ms, easeOut curve)
- **Directionality**: Uses RTL (right-to-left) text direction for Arabic content

### User Experience
- **Hands-Free Continuous Reading**: Users can recite entire surah without manual controls
- **Real-Time Feedback**: Each ayah evaluated immediately after completion
- **Visual Progress Tracking**: Current ayah highlighted, easy to see where you are
- **Tajweed Learning Aid**: Color-coding helps users identify and practice tajweed rules
- **Flexible Practice**: Manual navigation allows reviewing specific ayahs
- **Adaptive Detection**: Configurable parameters for different environments and speaking styles

### Files Modified
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/lib/main.dart`
  - Line 56: Changed TabController length from 2 to 3
  - Line 62: Added 'Read' tab to TabBar
  - Line 68: Added ReadTab() to TabBarView
  - Lines 364-617: Complete ReadTab implementation (253 lines)
    - State variables and initialization (lines 366-395)
    - Surah loading with tokenization (lines 397-412)
    - Listen toggle and noise detection (lines 414-462)
    - Ayah evaluation with auto-advance (lines 464-496)
    - Navigation and color helpers (lines 498-516)
    - UI build with controls and scrollable ayah list (lines 518-616)

### Known Limitations
- Auto-segmentation disabled on web (native audio APIs required)
- Requires microphone permissions for noise detection
- Color-coding depends on backend tajweed tokenization accuracy
- Evaluation quality depends on backend `/evaluate_span` endpoint

### Notes
- Read tab complements existing Voice and Practice tabs
- Voice tab: Real-time conversation with AI coach
- Read tab: Continuous surah reading with auto-evaluation
- Practice tab: Single ayah practice with detailed analysis
- Read tab uses same silence detection as Voice tab auto-evaluate feature
- Backend coordination via public2/public3.asimo.io documentation system

---

## 2025-10-18 - Major UI Modernization + Auto-Evaluate + Multi-Reciter Support

### Added
- **Material Design 3 UI**: Complete UI overhaul with Google Fonts (Inter), modern card layouts, and refined color scheme
- **Auto-Evaluate on Silence Detection** (Voice Tab):
  - Uses `noise_meter` package to detect speech vs. silence
  - Automatically records when user speaks, stops when silence detected
  - Configurable thresholds: silence dB level, hold duration, minimum speech duration
  - Hands-free evaluation during Voice sessions
  - Adjustable sliders for fine-tuning detection parameters
- **Multi-Reciter Support** (Practice Tab):
  - Dropdown to select different Quranic reciters
  - Backend fetches audio from `/audio?verse_id={S:V}&reciter={name}` endpoint
  - Fallback to 'default' reciter if not available
- **Playback Controls** (Practice Tab):
  - Speed control: 0.7×, 1.0×, 1.25×
  - Loop mode toggle for repeated practice
  - Integrated with canonical audio player
- **Proxy-Only Voice Mode**:
  - Simplified to proxy mode only (removed ephemeral toggle)
  - More reliable connection through backend
  - Cleaner user experience

### Changed
- **UI Framework**: Upgraded to Material 3 design language
- **Typography**: All text uses Inter font family via Google Fonts
- **Cards**: Rounded corners (16px radius), elevated shadows, consistent spacing
- **Buttons**: FilledButton and OutlinedButton with better padding
- **Voice Tab Layout**: Organized into three distinct card sections:
  1. Session controls (Start/Stop/Mute)
  2. Auto-evaluate settings with sliders
  3. Manual evaluate specific ayah
- **Practice Tab**: Recording duration increased from 3-10s slider to fixed 6s
- **Removed**: Ephemeral/Direct mode toggle (simplified to proxy-only)
- **Removed**: Timed transcript feature (will be re-added in future update)
- **Removed**: Word-level diff visualization (will be re-added in future update)

### Technical Details
- **New Dependencies**:
  - `noise_meter: ^5.1.0` - Audio level monitoring for silence detection
  - `google_fonts: ^6.3.2` - Inter font family
  - `permission_handler` (auto-included) - Microphone permissions
- **Noise Detection Algorithm**:
  - Listens to `NoiseMeter.noise` stream
  - Compares `meanDecibel` to configurable threshold (default -45 dB)
  - Tracks speech start time and silence start time
  - Triggers evaluation after minimum speech duration + silence hold time
  - Prevents false positives from brief pauses
- **Proxy Mode**: Connects through `$backendBase/rtc/offer` endpoint only
- **Multi-Reciter**: Calls `GET /reciters` to populate dropdown dynamically

### Files Modified
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/lib/main.dart`
  - Complete rewrite (577 lines)
  - New `QuranTutorApp` widget with Material 3 theme
  - `_VoiceTabState`: Added auto-evaluate with noise detection (lines 88-252)
  - `_PracticeTabState`: Added multi-reciter support and playback controls (lines 376-540)
- Backup created at: `lib/main.dart.backup`

### Backend Integration
- Leverages existing `coach_line_ar` field from backend's recent update
- Compatible with multi-reciter audio endpoint
- Uses `/reciters` endpoint for dynamic reciter list

### User Experience Improvements
- Cleaner, more modern interface aligned with Material Design guidelines
- Hands-free operation with auto-evaluate (especially useful while practicing)
- Flexible practice with different reciters and playback speeds
- Loop mode enables focused repetition of difficult verses
- Better visual hierarchy with card-based layout
- Consistent spacing and padding throughout

### Known Limitations
- Auto-evaluate disabled on web (native audio APIs required)
- Timed transcript temporarily removed (will be restored)
- Word-level diff temporarily removed (will be restored)
- Ephemeral mode removed (simplified to proxy-only)

### Notes
- Original version backed up to `lib/main.dart.backup`
- Auto-evaluate works best with headphones to prevent feedback loop
- Silence detection parameters can be adjusted per user's environment
- Loop mode is useful for memorization and tajweed practice
- Speed control helps beginners follow along more easily

---

## 2025-10-18 - Added "Evaluate Ayah" Feature to Voice Tab

### Added
- **Voice Tab Enhancement**: New "Evaluate āyah" feature that allows users to:
  - Enter an ayah reference (e.g., "1:1") in a text field
  - Record 5 seconds of recitation audio
  - Upload audio to backend `/evaluate_audio` endpoint
  - Receive Arabic coach feedback (`coach_line_ar`)
  - Have the AI coach speak the feedback through the Realtime agent
- New state variables in `_VoiceTabState`:
  - `_ayahCtl`: TextEditingController for ayah input (defaults to "1:1")
  - `_recForVoice`: AudioRecorder instance for voice evaluation
- New method `_evaluateCurrentAyah()` in `lib/main.dart:142-186` that:
  - Validates ayah format (S:V pattern)
  - Records 5 seconds of audio at 16kHz WAV
  - Uploads to backend with `user_id: 'voice-mode'`
  - Extracts `coach_line_ar` from response
  - Sends feedback to Realtime agent via data channel
- New UI row in Voice Tab with:
  - "Ayah:" label
  - TextField for ayah input (90px width, hint: "S:V")
  - "Evaluate āyah" button

### Changed
- Voice Tab UI now includes evaluation controls below Start/Stop/Mute buttons
- Status message displays coach feedback in Arabic when evaluation completes

### Backend Integration
- Integrated with backend changes from 2025-10-18:
  - Enhanced Arabic normalization
  - New `coach_line_ar` and `coach_line_en` fields in evaluation response
  - Improved tajweed rule detection
- Checked backend documentation:
  - API Specs: https://public2.asimo.io/QURAN_API_DOCUMENTATION.md
  - Changelog: https://public3.asimo.io/CHANGELOG.md

### Technical Details
- Feature uses existing `record` and `path_provider` dependencies (already imported)
- Audio recorded to temporary directory as `voice_eval.wav`
- Multipart form upload with fields: `verse_id`, `user_id`, `file`
- Graceful error handling for invalid ayah format, missing permissions, upload failures
- Fallback message if voice channel not open when coach line ready

### Files Modified
- `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter/lib/main.dart`
  - Line 63-64: Added state variables
  - Line 142-186: Added `_evaluateCurrentAyah()` method
  - Line 353-358: Added UI row with TextField and button

### Notes
- This feature enables users to practice specific ayahs during a live Voice session
- The AI coach provides personalized feedback based on backend evaluation
- Feedback is spoken by the Realtime agent, creating a seamless conversational experience
- User can evaluate multiple ayahs without stopping the Voice session

---

## 2025-10-18 - Initial Deployment & Documentation

### Added
- **PROJECT_CONTEXT.md** - Comprehensive project documentation for future Claude sessions
- **FRONTEND_CHANGELOG.md** - This changelog file to track frontend changes
- iOS deployment via ios-deploy workaround for iOS 18.5 beta + Xcode 16.4 beta pairing issue

### Changed
- **Bundle Identifier**: Changed from `com.example.mobileFlutter` to `com.nazmy.quranvoicetutor`
- **iOS Signing**: Configured with Apple Developer ID (Team 2F2XVQPM5R)

### Fixed
- iOS deployment to physical device (iPhone XR) now working using release builds
- Resolved CoreDevice pairing issue by using ios-deploy with legacy pairing mechanism

### Deployment
- Successfully deployed release build to iPhone XR (UDID: 00008020-0014188A3EA3002E)
- App confirmed working on physical device

### Configuration
- Backend base URL: `https://quran.asimo.io`
- User ID: `nazmy-dev`
- Microphone permissions configured in iOS Info.plist

### Current Features Working
- **Voice Tab**: WebRTC realtime conversation with AI coach
  - Direct mode (ephemeral OpenAI connection)
  - Proxy mode (through backend)
  - Microphone mute/unmute
  - Automatic SDP optimization

- **Practice Tab**: Record and evaluate recitation
  - Surah/verse selection
  - Configurable recording duration (3-10s)
  - Audio upload and evaluation
  - WER statistics display
  - Color-coded word diff visualization
  - Tajweed rules display
  - Coach tips
  - Timed transcript with synchronized playback
  - A/B comparison (user vs canonical audio)

### Backend Integration
- Integrated with all 12 API endpoints
- Documentation coordination system established
  - API Specs: https://public2.asimo.io/QURAN_API_DOCUMENTATION.md
  - Backend Changelog: https://public3.asimo.io/CHANGELOG.md

### Known Issues
- Debug builds crash on physical device (use release builds only)
- iOS 18.5 beta + Xcode 16.4 beta CoreDevice framework incompatibility
- Device shows as "unavailable" in Xcode (workaround: ios-deploy)

### Development Environment
- Project path: `/Users/mohammednazmy/dev/quran_voice_tutor/mobile_flutter`
- Flutter version: Latest (via Homebrew)
- Xcode version: 16.4 (16F6)
- macOS version: 15.3.1 (24D70)

### Next Session TODO
- Test all features thoroughly on physical iPhone
- Consider implementing user authentication
- Evaluate need for progress tracking
- Review UI/UX improvements

---

## Session Tracking

**Session End**: 2025-10-18
**Claude Instance**: Mac Claude (Claude Code)
**Status**: ✅ App successfully deployed and working on iPhone
**Pending Work**: None - ready for next feature development

---

## Change Log Format

For future entries, use this format:

```markdown
## YYYY-MM-DD - Brief Description

### Added
- New features, files, or capabilities

### Changed
- Modifications to existing functionality
- ⚠️ Mark breaking changes

### Fixed
- Bug fixes and corrections

### Removed
- Deprecated or removed features

### Deployment
- Deployment-related changes

### Backend Coordination
- Changes related to backend integration
- API version updates

### Notes
- Important context or decisions
- Technical debt
- Future considerations
```

---

## Instructions for Claude

**At the END of each work session**:
1. Update this FRONTEND_CHANGELOG.md with all changes made
2. Update PROJECT_CONTEXT.md "Current Implementation Status" if features added/removed
3. Update PROJECT_CONTEXT.md "Last Updated" date
4. Commit changes if git is initialized

**At the START of each work session**:
1. Read this changelog to understand recent changes
2. Check "Next Session TODO" for pending work
3. Update "Session Tracking" with new session info when session ends
