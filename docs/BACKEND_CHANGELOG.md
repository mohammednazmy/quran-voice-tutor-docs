# Quran Recitation Training App - Change Log

**Backend-Frontend Coordination Log**

This document tracks all changes, updates, and architectural decisions for backend-frontend coordination.

---
## 2025-10-18 (Night) - Server-Side VAD Implementation Complete

### Added

- **Server-Side Voice Activity Detection (VAD)**:
  - Implemented `_audio_db_level()` helper function using `audioop.rms()` for RMS-based audio level analysis
  - Real-time audio chunk analysis with dB level calculation: `20 * log10(rms / 32768.0)`
  - Automatic silence detection and ayah_end triggering in `/ws/stream` WebSocket endpoint
  - Configurable via environment variables:
    - `WS_VAD_SILENCE_DB=-45` (default): Silence threshold in decibels
    - `WS_VAD_HOLD_MS=900` (default): Silence duration before auto-triggering evaluation

- **WebSocket Protocol Enhancements**:
  - `start` message now accepts `server_vad` boolean parameter (default: false)
  - Acknowledgment message includes `server_vad` status confirmation
  - Evaluation responses include `vad_triggered: true` flag when VAD auto-triggered
  - VAD state properly resets on each new ayah/session

### Technical Details

- **VAD Algorithm**:
  1. Each audio chunk analyzed for RMS (Root Mean Square) level
  2. RMS converted to dB relative to 16-bit audio range (32768)
  3. If dB level < threshold for hold duration → auto-trigger evaluation
  4. Silence tracking resets when audio level rises above threshold
  5. State variables: `below_start` (timestamp), `server_vad` (enabled flag)

- **Integration Points**:
  - Frontend sends `'server_vad': true` in WebSocket start message
  - Backend maintains VAD state per WebSocket session
  - Auto-evaluation follows same pipeline as manual `ayah_end`:
    - STT with primary/secondary fallback
    - Core evaluation with WER, ops, tips
    - Tajweed rules analysis
    - Learning payload with translation/tafsir
  - Buffer automatically resets and advances to next ayah

- **Performance**:
  - Zero latency overhead for VAD-disabled sessions
  - Minimal computational cost: single RMS calculation per audio chunk
  - No additional API calls or external dependencies

### Testing Results

- ✓ Status Endpoint (`/status`)
- ✓ API Versioning (headers)
- ✓ Morphology System (`/morphology/sets`)
- ✓ Tajweed Rules Schema (via KB search)
- ✓ Knowledge Base Admin (`/kb/upload`, `/kb/build`, `/kb/search`)
- ✓ Documentation Viewer (`/public/list_docs`, `/public/api_doc`)
- ✓ Multi-Ayah Span Evaluation (`/evaluate_span`)
- ✓ Attempts Pagination (limit parameter)
- ✓ HTTP Caching with ETags (documentation endpoints)
- ✓ User Notes Export (`/notes/export?fmt=json|md|csv`)
- ✓ **Server-Side VAD Functionality (WebSocket `/ws/stream`)**

**All 10 backend integrations now fully functional and tested.**

---
## 2025-10-18 (Late Evening) - Knowledge Base Build Performance Fix

### Fixed

- **Knowledge Base Build Optimization (`POST /kb/build`)**:
  - Replaced sequential API calls (1 per chunk) with token-aware batch processing
  - Now batches up to 2,048 inputs per API call while respecting OpenAI's 300k token limit
  - Performance improvement: 20,567 chunks indexed in ~2 minutes (was timing out after 15+ min)
  - Reduced API calls from 20,000+ to ~10 batch calls
  - Dynamic batch sizing based on estimated token count (1 token ≈ 4 chars)
  - Database schema validated with `page` column for citation support

### Technical Details

- Added three-phase processing:
  1. **Collection Phase**: Gather all chunks with metadata (source, title, page, chunk_ix, text)
  2. **Batch Embedding Phase**: Process in token-aware batches with extended timeout (300s)
  3. **Database Insertion Phase**: Bulk insert all chunks with embeddings
- Updated API timeout: 300s (was 60s) to handle large batches
- Maintains page-level metadata for accurate citations

---


## 2025-10-18 (Evening) - Voice-First Pack v2: Learning, Knowledge Base & WebSocket VAD

### Added

- **Learning & Translation Management**:
  - `GET /learn/sets` - List available translation and tafsir datasets
  - `POST /learn/set_defaults` - Switch active translation/tafsir sets at runtime
  
- **Morphology System**:
  - `GET /morphology/sets` - List available word-by-word morphology datasets
  - `POST /morphology/set_default` - Switch active morphology set dynamically
  - Seed morphology file created: `/opt/quran-rtc/knowledge/morphology/en_basic.json`
  - Pluggable morphology system for extensibility

- **Knowledge Base with Page-Level Citations**:
  - `POST /kb/upload` - Upload PDF/TXT documents to knowledge base
  - `POST /kb/build` - Process documents, extract by page, generate embeddings
  - `GET /kb/search` - Semantic search with page citations (e.g., "page 12")
  - Uses `pdftotext` for page-accurate text extraction
  - OpenAI embeddings (text-embedding-3-small) for vector search
  - Cosine similarity ranking with top-k results (max 50)
  - SQLite database for document storage at `/opt/quran-rtc/library/kb.sqlite`

- **User Notes System**:
  - `POST /notes/add` - Save study notes (verse-specific or general)
  - `GET /notes/list` - Retrieve all user notes
  - `GET /notes/export` - Export notes in JSON, CSV, or Markdown formats
  - `POST /notes/remove` - Delete specific notes
  - Notes stored in `/var/log/quran-rtc/notes/`

- **Quiz System**:
  - `GET /quiz/next` - Get next quiz question based on user history
  - Question types: vocab, tajweed, memorization
  - Prioritizes recently practiced verses
  - Quiz data stored in `/var/log/quran-rtc/quiz/`

- **WebSocket Streaming with Server-Side VAD**:
  - `WebSocket /ws/stream` - Real-time audio streaming endpoint
  - Optional server-side Voice Activity Detection (VAD)
  - Configurable silence threshold (`vad_thr_db`, default: -45dB)
  - Configurable silence hold time (`vad_hold_ms`, default: 900ms)
  - Auto-triggers evaluation when silence detected
  - Analyzes audio RMS in 200ms windows
  - Expects 16kHz 16-bit PCM audio from client
  - Returns evaluation blocks automatically on VAD trigger

### Changed

- **Backend** `/opt/quran-rtc/backend/server.py`:
  - Added imports: `uuid`, `base64`, `io`, `math`, `sqlite3`, `struct`, `audioop`
  - Added environment variable support:
    - `EMBED_MODEL` (default: text-embedding-3-small)
    - `MORPH_DEFAULT_SET` (default: en_basic)
    - `WS_VAD_SILENCE_DB` (default: -45)
    - `WS_VAD_HOLD_MS` (default: 900)
  - Added `MORPH_ROOT` constant and `_load_morph_set()` function
  - Added `_kb_db()` function for knowledge base database management
  - Added `_cosine()` function for similarity calculations
  - WebSocket handler enhanced with VAD state management

- **Environment** `/opt/quran-rtc/backend/.env`:
  - Added `EMBED_MODEL=text-embedding-3-small`
  - Added `TRANSL_DEFAULT_SET=en_primary`
  - Added `TAFSIR_DEFAULT_SET=en_short`
  - Added `MORPH_DEFAULT_SET=en_basic`
  - Added `WS_VAD_SILENCE_DB=-45`
  - Added `WS_VAD_HOLD_MS=900`

- **Apache Configuration** `/etc/apache2/sites-available/quran.asimo.io-le-ssl.conf`:
  - Added ProxyPass rules for `/morphology` endpoints

### Infrastructure

- **Directory Structure Created**:
  - `/opt/quran-rtc/knowledge/` - Root for translations, tafsir, morphology
    - `knowledge/translations/` - Translation dataset files
    - `knowledge/tafsir/` - Tafsir dataset files
    - `knowledge/morphology/` - Morphology dataset files (*.json)
  - `/opt/quran-rtc/library/` - Knowledge base document storage
    - `library/pdfs/` - Original PDF uploads
    - `library/text/` - Extracted text (organized by document)
    - `library/kb.sqlite` - SQLite database with embeddings
  - `/var/log/quran-rtc/notes/` - User notes storage
  - `/var/log/quran-rtc/quiz/` - Quiz history storage

- **Dependencies**:
  - `poppler-utils` - For PDF text extraction with `pdftotext`
  - `jq` - For JSON processing in scripts

### API Endpoints Summary

Total endpoints now: **28** (was 15)

**New Endpoints (13)**:
1. `GET /learn/sets` - List translation/tafsir sets
2. `POST /learn/set_defaults` - Switch translation/tafsir
3. `GET /morphology/sets` - List morphology sets
4. `POST /morphology/set_default` - Switch morphology set
5. `POST /kb/upload` - Upload KB document
6. `POST /kb/build` - Build KB index
7. `GET /kb/search` - Search KB with citations
8. `POST /notes/add` - Add user note
9. `GET /notes/list` - List user notes
10. `GET /notes/export` - Export user notes
11. `POST /notes/remove` - Remove user note
12. `GET /quiz/next` - Get quiz question
13. `WebSocket /ws/stream` - Streaming with VAD

### Frontend Integration Notes

**Knowledge Base**:
- Upload study materials (tajweed guides, tafsir PDFs) via `/kb/upload`
- Build index once via `/kb/build` (can take time for large docs)
- Search semantically via `/kb/search?q=qalqala&k=5`
- Display results with page citations: "See page 12 of Tajweed Basics"

**Morphology**:
- Fetch available sets via `/morphology/sets`
- Allow users to switch between basic/advanced word-by-word glosses
- Changes apply immediately to all `/learn/ayah` responses

**Notes**:
- Add notes during practice via `/notes/add`
- Display notes list in study dashboard via `/notes/list`
- Export for backup/sharing via `/notes/export?fmt=md`

**Quiz**:
- Generate practice questions via `/quiz/next?user_id=X&type=vocab`
- System prioritizes verses user recently practiced
- Returns null when no questions available

**WebSocket VAD**:
- Send initial message with `"server_vad": true` to enable
- Send PCM audio chunks with type "audio" and base64-encoded bytes
- Receive automatic evaluation when user pauses (no manual trigger needed)
- Adjust sensitivity with `vad_thr_db` and `vad_hold_ms` parameters

### Deprecation Warning

- **audioop module**: Used for VAD RMS calculation, deprecated in Python 3.13
  - Consider migrating to modern audio processing library (e.g., `scipy`, `librosa`)
  - Does not affect functionality in Python 3.12

### Migration Notes

- No breaking changes to existing endpoints
- All new features are opt-in (backward compatible)
- Existing clients continue to work without modifications
- New endpoints require no authentication (same CORS policy)

### Documentation Updates

- **API Documentation**: Added "Learning & Knowledge Base Endpoints" section with 13 new endpoints
- **Table of Contents**: Reorganized to include new section
- **README**: Updated endpoint count from 15 to 28
- **PROJECT.md**: Added knowledge base and learning system references

---

## 2025-10-18 (Late Evening) - Backend Infrastructure Improvements & Operational Enhancements

### Added
- **Status Endpoint** (`GET /status`):
  - Extended health check with operational metrics
  - Reports: version, model, voice (marin), STT model, uptime, attempts today, last error hint, available reciters
  - Useful for monitoring and debugging

- **API Versioning**:
  - Global `X-API-Version: 0.1` header on all endpoints via middleware
  - Enables API evolution tracking and client compatibility checks

- **HTTP Caching Enhancements**:
  - **ETag headers** added to `/public/*` and `/rules_schema` endpoints
  - SHA-1 hash of file content for strong validation
  - Enables 304 Not Modified responses for bandwidth savings
  - If-None-Match middleware infrastructure

- **Attempts Pagination**:
  - Added `since` parameter (ISO8601 or epoch seconds) to `/attempts` endpoint
  - Added `offset` parameter for cursor-based pagination
  - Returns `next_offset` for easy page traversal
  - Example: `/attempts?since=2025-01-01T00:00:00Z&limit=20&offset=0`

- **Compression**:
  - Enabled gzip/deflate for `text/markdown` content type
  - Reduces documentation transfer size significantly

- **Documentation Enhancements**:
  - Updated voice reference from "verse" to "marin" throughout API docs
  - Replaced "Whisper" references with "OpenAI STT (default: gpt-4o-mini-transcribe)"
  - Added "Operational Notes" section with version, caching, status, paging details
  - Added "Quick cURL Examples" section with practical command-line usage

### Changed
- **Backend** `/opt/quran-rtc/backend/server.py`:
  - Added `from datetime import datetime, timezone` imports
  - Added `import hashlib` for ETag generation
  - Added `STARTED_AT = time.time()` for uptime tracking
  - Fixed datetime import conflict (removed duplicate `import datetime`)
  - Added `_etag_headers_for_file()` helper function
  - Enhanced `/public/*` endpoints with ETag support
  - Enhanced `/rules_schema` endpoint with ETag support
  - Added `VersionHeaderMiddleware` for X-API-Version header
  - Added `IfNoneMatchMiddleware` for conditional request handling
  - Enhanced `/attempts` endpoint with since/offset pagination
  - Added `/status` endpoint with comprehensive metrics

- **Apache Configuration**:
  - Added `ProxyPass /status` rules to both HTTP and HTTPS vhosts
  - Added `AddOutputFilterByType DEFLATE text/markdown` for compression
  - Enabled mod_deflate module

### Technical Details
- **ETag Implementation**: SHA-1 hash of file content provides strong validation
- **Pagination**: Cursor-based with `next_offset` prevents page drift issues
- **Uptime Tracking**: Calculated from `STARTED_AT` timestamp set at server init
- **Status Endpoint**: Reads attempts from `/var/log/quran-rtc/attempts-*.jsonl` files for today
- **Error Tracking**: Reads last error hint from `/var/log/quran-rtc/error-last.log`
- **Reciters Discovery**: Scans `/opt/quran-rtc/audio/` directory for subdirectories

### Documentation
- Updated `API_DOCUMENTATION.md`:
  - Voice changed from "verse" to "marin" in health response
  - STT model clarified as "OpenAI STT (default: gpt-4o-mini-transcribe)"
  - Added Operational Notes section
  - Added Quick cURL Examples section with 8 practical examples
- Synced to public GitHub: https://github.com/mohammednazmy/quran-voice-tutor-docs

### Testing
```bash
# Verify X-API-Version header
curl -sI https://quran.asimo.io/health | grep x-api-version

# Verify ETag header
curl -sI https://quran.asimo.io/public/api_doc | grep etag

# Test status endpoint
curl -s https://quran.asimo.io/status | jq .

# Test pagination
curl -s "https://quran.asimo.io/attempts?limit=10&offset=0" | jq '.next_offset'
```

---

## 2025-10-18 (Evening) - Enhanced Tajweed Rules & Documentation Infrastructure

### Added
- **Tajweed Rule Detection Enhancements** in `/surah_tokens` endpoint:
  - **Izhar Detection**: Clear pronunciation when ن sakinah/tanween appears before throat letters (ء ه ع ح غ خ)
  - **Iqlab Detection**: Conversion to meem when ن sakinah/tanween appears before ب
  - Both rules integrated into existing idgham/ikhfa detection logic with proper priority ordering

- **Rules Schema Endpoint**:
  - `GET /rules_schema` - Serves `/opt/quran-rtc/data/rules_schema.json` with Arabic label translations for all 14 tajweed rule categories
  - Includes caching headers (Cache-Control: max-age=300, Last-Modified)
  - Enables frontend to display localized rule names (e.g., "مد لازم" for madd_lazim)

- **Documentation Endpoints** (`/public/*` with FastAPI passthrough):
  - `GET /public/api_doc` - Complete API documentation in Markdown
  - `GET /public/changelog` - Backend changelog with version history
  - `GET /public/context` - Full project context for development
  - All endpoints include HTTP caching headers:
    - `Cache-Control: max-age=300, public` (5-minute cache)
    - `Last-Modified: <file timestamp>` for client-side cache validation

### Changed
- **Backend**: `/opt/quran-rtc/backend/server.py`
  - Added `JSONResponse` to FastAPI imports for JSON endpoint support
  - Added `IZHAR_SET = set("ءههعحغخ")` constant for throat letters
  - Added `IQLAB_TARGET = 'ب'` constant for iqlab detection
  - Updated `tag_rules_for_ayah()` function:
    - Nun sakinah section: checks for idgham_ghunnah → idgham_no_ghunnah → iqlab → izhar → ikhfa
    - Tanween section: same priority ordering for rule detection
  - Enhanced `/public/*` endpoint responses with caching headers
  - Added `@app.get("/rules_schema")` endpoint handler with JSON response and caching

- **Apache Proxy**: Added ProxyPass rules for `/rules_schema` and `/public/*` paths
  - Routes requests to FastAPI backend (port 5056)
  - Configured in both SSL and non-SSL vhosts

### Technical Details
- **Izhar Logic**: When ن sakinah (ن with ْ) or tanween (ً ٍ ٌ) is followed by throat letters (hamza, ha, ayn, ha, ghayn, khah), apply izhar rule
- **Iqlab Logic**: When ن sakinah or tanween is followed by ب, apply iqlab rule
- **Priority Order**: idgham (with/without ghunnah) checked first, then iqlab, then izhar, finally ikhfa as default
- **HTTP Caching**: 5-minute cache allows near-real-time updates while reducing server load for AI tool access

### Documentation
- Updated `API_DOCUMENTATION.md`:
  - Added `GET /rules_schema` endpoint documentation with Arabic label examples
  - Enhanced izhar and iqlab rule descriptions with specific detection criteria
  - Added new "Documentation Endpoints" section with examples and alternative GitHub URLs
  - Updated inline changelog with recent enhancements
- Updated `CHANGELOG.md` with detailed technical implementation notes for all evening changes

### Coordination
- Documentation auto-syncs to public GitHub repository every 5 minutes
- Multiple access methods for AI tools:
  - FastAPI endpoints: https://quran.asimo.io/public/*
  - GitHub raw URLs: https://raw.githubusercontent.com/mohammednazmy/quran-voice-tutor-docs/main/docs/*

---

## 2025-10-18 (Morning) - Surah Tokenization with Tajweed Color Tags & Multi-Ayah Span Evaluation

### Added
- **New Endpoint: `GET /surah_tokens?surah=S`** - Full-surah tajweed tokenization with rule-based color tagging
  - Returns every word (token) with detected tajweed rules and assigned color
  - Includes comprehensive palette (15 rules: madd_lazim, madd_muttasil, madd_munfasil, madd_badal, ghunnah, ikhfa, idgham_ghunnah, idgham_no_ghunnah, izhar, iqlab, qalqalah, tafkhim, tarqiq, waqf, default)
  - Rule detection is heuristic-based (pattern matching on diacritics)
  - First detected rule determines token's color
  - Enables UI color-highlighting for tajweed learning

- **New Endpoint: `POST /evaluate_span`** - Multi-ayah evaluation with madd duration analysis
  - Evaluates consecutive ayah spans in a single recording (e.g., Surah 1, Ayahs 1-7)
  - Returns per-ayah evaluation blocks with word alignment, tajweed rules, and madd checks
  - **Madd Duration Analysis**: Estimates actual vs. expected elongation counts using STT word timestamps
    - Calculates median word duration as tempo baseline
    - Compares expected harakat counts (madd_lazim: 6, madd_muttasil/munfasil: 4.5, madd_badal: 2)
    - Provides `delta_ms` guidance (e.g., "Hold 50ms longer")
  - **Greedy Sequential Alignment**: Aligns entire transcript across ayahs (not per-ayah boundaries)
  - Form fields: `surah`, `start_ayah`, `end_ayah`, `file`, `user_id` (optional)

- **New Data Files**:
  - `/opt/quran-rtc/data/palette.json` - Color mapping for 15 tajweed rule categories
  - `/opt/quran-rtc/data/rules_schema.json` - Arabic label translations for rules

### Changed
- **Backend**: Extended `/opt/quran-rtc/backend/server.py` with:
  - `load_palette()` - Loads color palette from disk
  - `tag_rules_for_ayah(text)` - Tags each word with detected tajweed rules
  - `madd_sites_for_ayah(text)` - Identifies madd occurrences by token index
  - `expected_counts_for(madd_type)` - Returns expected harakat counts per madd type
  - Enhanced with Unicode constants for Arabic diacritics (FATHA, KASRA, DAMMA, SUKUN, TANWEEN)
  - Sets for rule detection (QALQALAH_SET, IDGHAM_GHUNNAH_SET, IDGHAM_NO_GHUNNAH_SET, HEAVY_SET)

- **Apache Proxy**: Added proxy rules for `/surah_tokens` and `/evaluate_span`
  - Updated both HTTP and HTTPS vhosts
  - All requests routed to backend port 5056

### Backend Notes
- **File Modified**: `/opt/quran-rtc/backend/server.py`
- **New Functions**:
  - `load_palette()` - Loads JSON palette with fallback to default black
  - `tokenize_words_uthmani(txt)` - Simple whitespace tokenizer
  - `tag_rules_for_ayah(ayah_text)` - Returns list of `{t: "word", rules: [...], color: "#hex"}`
  - `madd_sites_for_ayah(ayah_text)` - Returns list of `{tokenIndex, type}`
  - `expected_counts_for(mt)` - Maps madd type to expected counts
  - `@app.get("/surah_tokens")` - Endpoint handler
  - `@app.post("/evaluate_span")` - Endpoint handler with full STT→alignment→madd-check pipeline
- **Backup Created**: `server.py.bak.{timestamp}`
- **Import Added**: `import statistics` (for median calculation in madd duration analysis)

### Rule Detection Heuristics
The system uses **text-based pattern matching** on diacritized Uthmani script:

1. **Qalqalah**: ق ط ب ج د with sukun or at word-end position (ignoring trailing diacritics)
2. **Idgham with Ghunnah**: ن sakinah or tanween followed by ي ن م و
3. **Idgham without Ghunnah**: ن sakinah or tanween followed by ر ل
4. **Ikhfa**: ن sakinah or tanween followed by other letters (catch-all for concealment)
5. **Madd Patterns**:
   - `madd_lazim`: Alif with madda sign (ٓ) and tatweel (ـ)
   - `madd_muttasil`: Fatha + alif within word (with tatweel indicator)
   - `madd_munfasil`: Kasra + ya sukun OR damma + waw sukun at word end
   - `madd_badal`: Fatha + alif without tatweel (substitution madd)
6. **Tafkhim**: Words containing heavy/emphatic letters (خ ص ض غ ط ق ظ)
7. **Tarqiq**: Words containing light letters (ر ل) - context-dependent

**Note**: These are simplified heuristics. Complex tajweed rules (e.g., ra tafkhim/tarqiq based on context) are not fully implemented yet.

### Frontend Impact
- ✅ **Backward Compatible**: Existing endpoints unchanged
- **New Features Available**:
  1. **Visual Tajweed Highlighting**:
     - `GET /surah_tokens?surah=1` → display each word with color from `color` field
     - Show rule labels on hover/tap from `rules` array
     - Display palette legend for user education
  2. **Continuous Recitation Evaluation**:
     - Record multiple ayahs in one take
     - `POST /evaluate_span` with `start_ayah`/`end_ayah`
     - Display per-ayah WER, coach lines, and word-level errors
     - Visualize madd duration corrections (e.g., "Hold 50ms longer")
  3. **Customizable Palette**:
     - Edit `/opt/quran-rtc/data/palette.json` to match your Tajweed PDF colors
     - Changes apply immediately (palette loaded on each request)

### Usage Recommendations
1. **Tajweed Learning Mode**:
   - Fetch `GET /surah_tokens?surah=1` when user selects a surah
   - Cache response for offline use
   - Render each ayah with color-coded words
   - Tap/hover to show rule explanations (use `rules` array)

2. **Continuous Recitation Mode**:
   - Let user record full surah or multi-ayah span
   - POST to `/evaluate_span` with `surah=1&start_ayah=1&end_ayah=7&file=@recording.mp3`
   - Display per-ayah results:
     - Show WER score for each ayah
     - Highlight misaligned words (use `ops` array)
     - Display coach line with madd-specific feedback
   - Show overall tempo: `baseline_ms` (e.g., "Your pace: 180ms/word")

3. **Madd Duration Training**:
   - Parse `madd_checks` array in response
   - For each madd site, show:
     - Expected counts (e.g., "4.5 harakat")
     - Actual counts (e.g., "3.2 harakat")
     - Correction in ms (e.g., "+230ms")
   - Visualize with progress bars or audio waveform annotations

### Testing
- ✅ Backend restarted successfully
- ✅ Health check passed
- ✅ `/surah_tokens?surah=1` returns 7 ayahs with tokenized words and palette
- ✅ Palette has 15 color rules
- ✅ Apache proxy rules added for both new endpoints
- ⚠️ **Madd Count Estimation**: Coarse baseline (median word duration). Can be refined with phoneme-level timing in future.

### Known Limitations
1. **Rule Detection**:
   - Heuristic-based (not phonologically aware)
   - May miss context-dependent rules (e.g., ra tafkhim/tarqiq depends on preceding vowel)
   - Qalqalah detection at word-end assumes stop (lenient)
   - No waqf mark interpretation yet

2. **Madd Duration**:
   - Uses STT word timestamps (not phoneme-level)
   - Estimates harakat counts by dividing word duration by tempo baseline
   - **Inaccurate for words with multiple madd sites** (currently only checks first madd)
   - Can be improved with phoneme-aligned STT models (e.g., forced alignment with Arabic phonemes)

3. **Palette Colors**:
   - Current colors are placeholders
   - Users should customize `/opt/quran-rtc/data/palette.json` to match their Tajweed visual guide

4. **Continuous Evaluation Alignment**:
   - Greedy sequential alignment across ayahs
   - If STT misses/adds words early, alignment may drift in later ayahs
   - Per-ayah boundary hints may improve accuracy (future work)

### Migration Notes
- **No Breaking Changes**: All existing endpoints unchanged
- **New Dependencies**: `statistics` module (stdlib, no install needed)
- **Data Directory**: Ensure `/opt/quran-rtc/data/` exists and is writable by `quranrtc` user
- **Palette Customization**: Edit `/opt/quran-rtc/data/palette.json` as needed (changes apply immediately)

---


## 2025-10-18 - Multi-Reciter Canonical Audio Support

### Added
- **New Endpoint: `GET /reciters`** - List all available reciter identifiers
  - Returns array of reciter names (e.g., `["default", "husary", "mishary"]`)
  - Always includes "default" in the list
  - Reciters sorted alphabetically
  - Use these identifiers for the `reciter` parameter in audio endpoints

### Changed
- **⚠️ POST `/audio` Enhanced** (backward compatible):
  - Now accepts optional `reciter` parameter (form field)
  - Defaults to `"default"` if not specified
  - **Storage structure changed**: `/opt/quran-rtc/audio/{reciter}/{surah}/{ayah}.{ext}`
  - Response now includes `"reciter"` field
  - Example: `POST /audio -F "verse_id=1:1" -F "file=@audio.mp3" -F "reciter=mishary"`

- **⚠️ GET `/audio` Enhanced** (backward compatible):
  - Now accepts optional `reciter` query parameter
  - Defaults to `"default"` if not specified
  - Example: `GET /audio?verse_id=1:1&reciter=mishary`
  - Returns 404 if reciter not found or no audio available for that reciter

### Backend Notes
- **File Modified**: `/opt/quran-rtc/backend/server.py`
- **Modified Functions**:
  - `upload_audio()` - Added `reciter` parameter, updated storage path logic
  - `get_audio()` - Added `reciter` parameter, updated retrieval path logic
- **New Function**: `reciters()` - Lists available reciter directories
- **Apache Configs Updated**: Added proxy rules for `/reciters` endpoint
- **Backup Created**: `server.py.bak.{timestamp}` before changes

### Storage Migration
**Old Structure:**
```
/opt/quran-rtc/audio/
  ├── 1/
  │   ├── 1.mp3
  │   └── ...
```

**New Structure:**
```
/opt/quran-rtc/audio/
  ├── default/
  │   ├── 1/
  │   │   ├── 1.mp3
  │   │   └── ...
  ├── mishary/
  │   ├── 1/
  │   │   └── ...
```

**Migration Notes:**
- Existing audio files in old structure remain accessible as reciter "1" (detected as directory)
- New uploads using default behavior will go to `/audio/default/{surah}/{ayah}`
- Manual migration recommended: Move old files to `/audio/default/` structure

### Frontend Impact
- ✅ **Backward Compatible**: Existing calls to `/audio` work without changes (use "default" reciter)
- **New Features Available**:
  1. **Discover Reciters**: `GET /reciters` → list available reciters
  2. **Select Reciter**: `GET /audio?verse_id=1:1&reciter=mishary`
  3. **Upload to Reciter**: `POST /audio -F "reciter=husary"`
- **UI Enhancements**: Can now offer users choice of reciters for playback
- **Response Change**: POST `/audio` response now includes `"reciter"` field (non-breaking addition)

### Usage Recommendations
1. **Initial Load**: Call `GET /reciters` to populate reciter selection UI
2. **User Selection**: Store user's preferred reciter in app state
3. **Audio Playback**: Include `&reciter={name}` in GET `/audio` requests
4. **Admin Uploads**: Specify `reciter` when uploading canonical audio files

### Testing
- ✅ Backend restarted successfully
- ✅ Health check passed
- ✅ `/reciters` endpoint returns list (currently: `["default", "1"]`)
- ✅ Audio upload and retrieval with `reciter` parameter functional
- ✅ Apache proxy for `/reciters` configured and operational

---

## 2025-10-18 - Enhanced Arabic Normalization & Coach Line Feedback

### Added
- **Coach Line Fields**: Both `/evaluate` and `/evaluate_audio` now return `coach_line_ar` and `coach_line_en` fields
  - Concise, actionable feedback synthesized from WER score, tips, and tajweed rules
  - Format: `[Score feedback] + [Tip/Rule focus] + [Encouragement]`
  - Examples:
    - Perfect (WER=0.0): "ممتاز. الدقّة 100%. استمرّ على هذا الإيقاع." / "Excellent. Accuracy 100%. Nice pace."
    - With tip (WER=0.05): "ممتاز. الدقّة 95%. ملاحظة: [specific correction]"
    - With rule focus (WER=0.12): "جيد جدًّا. الدقّة 88%. راقب حكم qalqalah في «الْخَلْقِ»."
  - Priority: Tips (corrections) > Tajweed rules (focus) > General encouragement

### Changed
- **⚠️ Enhanced Arabic Normalization** (improves accuracy, should be non-breaking):
  - Now unifies Alif/Hamza variants: أ إ آ ٱ ء → ا
  - Now unifies Alif Maqsurah: ى → ي (fixes STT transcription mismatches)
  - Now unifies Ta Marbuta: ة → ه (more forgiving for pause pronunciation)
  - Added removal of Quranic stop/pause marks (U+06D6-U+06ED range)
  - **Result**: Significantly fewer false negatives, better handling of STT variations

- **⚠️ Stricter Persona Instructions** at `/persona`:
  - Old: "حيِّ بإيجاز، واطلب قراءة الفاتحة بتأنٍ. لا تقاطع إلا إذا تكرّر الخطأ."
  - New: "لا تقدّم تصحيحًا من نفسك. انتظر تعليمات تصل عبر القناة البيانية ثم انطقها بإيجاز."
  - **Purpose**: Prevents AI from "freestyling" corrections; ensures it only relays backend-provided `coach_line` feedback
  - **Impact**: Voice mode will now speak exactly what the backend determines, maintaining consistency

### Backend Notes
- **File Modified**: `/opt/quran-rtc/backend/server.py`
- **New Functions**:
  - `coach_line_ar(stats, tips, rules)` - Arabic feedback synthesizer
  - `coach_line_en(stats, tips, rules)` - English feedback synthesizer
- **Modified Function**: `normalize_ar()` - Enhanced with letter variant unification
- **Backup Created**: `server.py.bak.{timestamp}` before changes

### Frontend Impact
- ✅ **Non-Breaking Change**: Existing API calls continue to work
- **New Response Fields** (optional to use):
  - `coach_line_ar` (string) - Ready-to-display/speak Arabic feedback
  - `coach_line_en` (string) - Ready-to-display/speak English feedback
- **Improved Accuracy**: Users should see fewer false negatives due to better normalization
- **Voice Mode Integration**: Display or speak the `coach_line_*` fields directly to users
- **Text Mode Integration**: Show `coach_line_*` as immediate post-evaluation feedback

### Usage Recommendations
1. **Voice Mode**: Have the AI coach speak `coach_line_ar` or `coach_line_en` after each recitation
2. **Text Mode**: Display the coach line prominently in the UI after evaluation
3. **Bilingual Support**: Choose language based on user preference
4. **Fallback**: If `coach_line_*` is empty (shouldn't happen), fall back to displaying WER percentage

### Testing
- Backend restarted successfully
- Health check confirmed: ✅ Service running
- All endpoints tested and operational

---

## 2025-10-18 - Initial Documentation & Coordination System

### Added
- **API Documentation System**: Created comprehensive API documentation at public2.asimo.io
- **Changelog System**: Created this changelog at public3.asimo.io
- **12 API Endpoints Documented**:
  - Core Data: `/health`, `/surahs`, `/verses`, `/canon`
  - Evaluation: `/evaluate`, `/evaluate_audio`
  - Audio Management: `POST /audio`, `GET /audio`
  - Real-Time: `/rtc/offer`, `/rtc/offer_ephemeral`, `/realtime/ephemeral`, `/persona`

### Backend Architecture
- **Server**: FastAPI (Python) running on port 5056
- **Proxy**: Apache2 reverse proxy on quran.asimo.io
- **Location**: `/opt/quran-rtc/backend/server.py`
- **Data**: Quran text stored in `/opt/quran-rtc/data/quran.json`
- **Audio**: Canonical recordings in `/opt/quran-rtc/audio/{surah}/{ayah}.{ext}`
- **Logs**: `/var/log/quran-rtc/` (attempts, errors, diagnostics)

### Key Features
- **Arabic Text Processing**: Normalization, diacritic stripping, word alignment
- **Audio Evaluation**: OpenAI Whisper STT → alignment → coaching tips
- **Tajweed Rules**: Automatic detection of qalqalah, idgham, madd
- **WebRTC**: Direct and proxied modes for real-time AI coaching
- **Word-Level Timestamps**: Available in audio evaluation responses

### Data Models
- **Verse ID Format**: `"{surah}:{ayah}"` (e.g., `"1:1"`)
- **Word Error Rate (WER)**: Primary metric for recitation accuracy
- **Alignment Operations**: `ok`, `sub`, `ins`, `del` for word-by-word comparison

### CORS Configuration
- **Allowed Origins**: `https://quran.asimo.io`
- Frontend should use this domain for all API requests

### Frontend Integration Notes
- All endpoints return JSON (except `/audio` which returns binary audio)
- Audio upload uses `multipart/form-data`
- Text evaluation uses `application/json`
- WebRTC offers use `application/sdp`

### Environment
- **Production URL**: `https://quran.asimo.io`
- **Models**:
  - Realtime: `gpt-realtime`
  - STT: `gpt-4o-mini-transcribe`
  - Voice: `verse`

---

## Change Log Format

Future entries will follow this format:

```markdown
## YYYY-MM-DD - Change Title

### Added
- New features, endpoints, or capabilities

### Changed
- Modifications to existing functionality
- Breaking changes (marked with ⚠️)

### Fixed
- Bug fixes

### Removed
- Deprecated features

### Backend Notes
- Technical implementation details
- File locations
- Configuration changes

### Frontend Impact
- What the frontend team needs to know
- Required changes in Flutter app
- Migration steps if needed
```

---

## Coordination Protocol

When backend changes are made:

1. ✅ Update API documentation at `/var/www/public2.asimo.io/QURAN_API_DOCUMENTATION.md`
2. ✅ Add changelog entry to `/var/www/public3.asimo.io/CHANGELOG.md`
3. ✅ Notify frontend team with:
   - Link to public2.asimo.io (full API docs)
   - Link to public3.asimo.io (what changed)
   - Summary of breaking changes (if any)

---

**Last Updated**: 2025-10-18 02:58 CDT
