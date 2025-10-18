# Quran Recitation Training App - Project Context

## Project Overview
This is a Quran recitation training app with an AI-powered coach that evaluates user recitations using speech-to-text, word alignment, and tajweed rule detection. The system provides real-time feedback on pronunciation accuracy and tajweed compliance.

## Architecture

### Backend
- **Framework**: FastAPI (Python 3.12)
- **Location**: `/opt/quran-rtc/backend/server.py`
- **Service**: systemd service `quran-rtc.service`
- **Port**: 5056 (localhost only)
- **User**: `quranrtc`

### Frontend
- **Platform**: Flutter (iOS app)
- **Development**: Mac environment (separate Claude instance)

### Infrastructure
- **Web Server**: Apache2 with reverse proxy
- **Domain**: `https://quran.asimo.io`
- **SSL**: Let's Encrypt certificates
- **CORS**: Configured for `https://quran.asimo.io`

### External Services
- **OpenAI Realtime API**: Voice coaching (gpt-4o-realtime-preview)
- **OpenAI Whisper**: Speech-to-text transcription (gpt-4o-mini-transcribe)
- **WebRTC**: Real-time audio communication

## Key Directories

```
/opt/quran-rtc/
├── backend/
│   ├── server.py              # Main FastAPI application
│   ├── .env                   # Environment variables (OPENAI_API_KEY)
│   └── .venv/                 # Python virtual environment
├── data/
│   ├── quran.json             # Full Quran text (Uthmani script)
│   ├── palette.json           # Tajweed rule color mappings
│   └── rules_schema.json      # Arabic labels for tajweed rules
└── audio/
    ├── default/               # Default reciter audio files
    ├── {reciter}/             # Other reciters (mishary, husary, etc.)
    │   └── {surah}/
    │       └── {ayah}.mp3

/var/log/quran-rtc/
├── attempts-{date}.jsonl      # Evaluation attempt logs
├── error-last.log             # Last error traceback
├── last-offer.sdp             # Last WebRTC SDP offer
└── last-realtime-error.json   # Last OpenAI Realtime API error

/var/www/
├── quran/                     # Main app web root (if needed)
├── public2.asimo.io/          # API documentation
│   ├── index.html             # Styled markdown viewer
│   └── QURAN_API_DOCUMENTATION.md
└── public3.asimo.io/          # Changelog
    ├── index.html             # Styled markdown viewer
    └── CHANGELOG.md
```

## API Endpoints (15 total)

### Core Data (4)
1. `GET /health` - Health check
2. `GET /surahs` - List all surahs
3. `GET /verses?surah=S` - Get verses for surah
4. `GET /canon?verse_id=S:V` - Get canonical verse text

### Evaluation (4)
5. `POST /evaluate` - Evaluate text recitation (JSON)
6. `POST /evaluate_audio` - Evaluate audio recitation (multipart)
7. `GET /surah_tokens?surah=S` - Get surah with tajweed color tags
8. `POST /evaluate_span` - Evaluate multi-ayah span with madd duration

### Audio Management (3)
9. `POST /audio` - Upload canonical audio (multi-reciter)
10. `GET /audio?verse_id=S:V&reciter=name` - Get canonical audio
11. `GET /reciters` - List available reciters

### Real-Time Communication (4)
12. `POST /rtc/offer` - WebRTC SDP proxy (legacy)
13. `POST /rtc/offer_ephemeral` - WebRTC SDP proxy with ephemeral secret
14. `GET /realtime/ephemeral` - Get ephemeral client secret
15. `GET /persona` - Get AI coach persona instructions

## Documentation

### Full API Reference
**URL**: https://public2.asimo.io
**File**: `/var/www/public2.asimo.io/QURAN_API_DOCUMENTATION.md`
**Raw Markdown**: https://public2.asimo.io/QURAN_API_DOCUMENTATION.md
**Description**: Comprehensive API documentation with request/response examples, error codes, and integration guides

### Changelog
**URL**: https://public3.asimo.io
**File**: `/var/www/public3.asimo.io/CHANGELOG.md`
**Raw Markdown**: https://public3.asimo.io/CHANGELOG.md
**Description**: Backend-frontend coordination log tracking all changes, updates, and architectural decisions

### Protocol
When making backend changes:
1. Update `/var/www/public2.asimo.io/QURAN_API_DOCUMENTATION.md`
2. Add changelog entry to `/var/www/public3.asimo.io/CHANGELOG.md`
3. Update timestamp in changelog footer
4. Notify frontend team with links to both docs

## Key Technologies

### Arabic Text Processing
- **Normalization**: Strips diacritics, unifies letter variants (أ إ آ ٱ ء → ا, ى → ي, ة → ه)
- **Alignment**: Levenshtein distance (edit distance) for word-level comparison
- **WER Calculation**: Word Error Rate = (substitutions + insertions + deletions) / reference_word_count

### Tajweed Rule Detection (Heuristic-based)
- **Qalqalah**: ق ط ب ج د with sukun or at word-end
- **Idgham with Ghunnah**: ن sakinah/tanween before ي ن م و
- **Idgham without Ghunnah**: ن sakinah/tanween before ر ل
- **Ikhfa**: ن sakinah/tanween before other letters
- **Madd Types**: madd_lazim (6 counts), madd_muttasil (4.5), madd_munfasil (4.5), madd_badal (2)
- **Tafkhim**: Heavy letters (خ ص ض غ ط ق ظ)
- **Tarqiq**: Light letters (context-dependent)

### Madd Duration Analysis
- Uses STT word timestamps to estimate elongation duration
- Calculates median word duration as tempo baseline
- Compares expected vs. actual harakat counts
- Provides delta_ms corrections (e.g., "Hold 50ms longer")

## Coach Line Synthesis
Both evaluation endpoints return concise feedback:
- **Arabic**: `coach_line_ar`
- **English**: `coach_line_en`
- **Format**: [Score feedback] + [Tip/Rule focus] + [Encouragement]
- **Priority**: Specific corrections > Tajweed rules > General encouragement

## Environment Variables
```bash
OPENAI_API_KEY=sk-...                           # Required
OPENAI_REALTIME_MODEL=gpt-4o-realtime-preview   # Default
OPENAI_REALTIME_VOICE=verse                     # Default
OPENAI_STT_MODEL=gpt-4o-mini-transcribe         # Default
QURAN_DATA=/opt/quran-rtc/data/quran.json       # Default
```

## Service Management
```bash
# Status
sudo systemctl status quran-rtc

# Restart
sudo systemctl restart quran-rtc

# Logs
sudo journalctl -u quran-rtc -f

# Apache reload
sudo systemctl reload apache2
```

## Common Tasks

### Deploy Backend Changes
1. Backup: `sudo cp /opt/quran-rtc/backend/server.py{,.bak.$(date +%s)}`
2. Edit: `sudo nano /opt/quran-rtc/backend/server.py`
3. Restart: `sudo systemctl restart quran-rtc`
4. Test: `curl https://quran.asimo.io/health`

### Update Documentation
1. Edit API docs: `/var/www/public2.asimo.io/QURAN_API_DOCUMENTATION.md`
2. Add changelog: `/var/www/public3.asimo.io/CHANGELOG.md`
3. Update timestamp in changelog footer

### Add New Endpoint to Apache Proxy
1. Edit: `/etc/apache2/sites-available/quran.asimo.io{,-le-ssl}.conf`
2. Add ProxyPass lines after existing endpoints
3. Reload: `sudo systemctl reload apache2`

### Customize Tajweed Colors
1. Edit: `/opt/quran-rtc/data/palette.json`
2. Changes apply immediately (loaded on each request)

## Known Limitations

### Tajweed Rule Detection
- Heuristic-based (not phonologically aware)
- May miss context-dependent rules (e.g., ra tafkhim/tarqiq)
- Qalqalah detection at word-end assumes stop (lenient)
- No waqf mark interpretation yet

### Madd Duration Analysis
- Uses word-level timestamps (not phoneme-level)
- Inaccurate for words with multiple madd sites
- Can be improved with forced alignment

### Continuous Evaluation
- Greedy sequential alignment across ayahs
- Alignment may drift if STT produces errors early in span
- Per-ayah boundary hints may improve accuracy

## Recent Updates

### 2025-10-18 - Surah Tokenization & Multi-Ayah Span Evaluation
- Added `GET /surah_tokens` with tajweed color tags
- Added `POST /evaluate_span` with madd duration analysis
- Created `/opt/quran-rtc/data/palette.json` (15 rule colors)
- Created `/opt/quran-rtc/data/rules_schema.json` (Arabic labels)

### 2025-10-18 - Multi-Reciter Audio Support
- Added `GET /reciters` endpoint
- Enhanced `POST /audio` with reciter parameter
- Enhanced `GET /audio` with reciter query parameter
- Changed storage: `/audio/{reciter}/{surah}/{ayah}.{ext}`

### 2025-10-18 - Enhanced Arabic Normalization & Coach Lines
- Improved normalization (unifies alif/hamza, alif maqsurah, ta marbuta)
- Added `coach_line_ar` and `coach_line_en` fields
- Stricter persona instructions (no AI "freestyling")

## Testing Endpoints
```bash
# Health
curl https://quran.asimo.io/health

# Surah tokens
curl "https://quran.asimo.io/surah_tokens?surah=1" | jq

# Reciters
curl https://quran.asimo.io/reciters | jq

# Evaluate audio
curl -X POST https://quran.asimo.io/evaluate_audio \
  -F "verse_id=1:1" \
  -F "file=@test.mp3" \
  -F "user_id=test" | jq

# Evaluate span
curl -X POST https://quran.asimo.io/evaluate_span \
  -F "surah=1" \
  -F "start_ayah=1" \
  -F "end_ayah=3" \
  -F "file=@test.mp3" | jq
```

## Support
- API Documentation: https://public2.asimo.io
- Changelog: https://public3.asimo.io
- Logs: `/var/log/quran-rtc/`
- Service: `sudo systemctl status quran-rtc`
