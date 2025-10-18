# Quran Recitation Training App - API Documentation

**Base URL:** `https://quran.asimo.io`
**Local Development:** `http://127.0.0.1:5056`

**Tech Stack:** FastAPI (Python), OpenAI Realtime API, WebRTC

---

## Table of Contents

1. [Authentication](#authentication)
2. [Core Data Endpoints](#core-data-endpoints)
3. [Evaluation Endpoints](#evaluation-endpoints)
4. [Audio Management Endpoints](#audio-management-endpoints)
5. [Real-Time Communication Endpoints](#real-time-communication-endpoints)
6. [Data Models](#data-models)
7. [Error Handling](#error-handling)

---

## Authentication

All endpoints are currently open for the Flutter frontend. The server uses CORS middleware configured to allow requests from `https://quran.asimo.io`.

**CORS Configuration:**
- **Allowed Origins:** `https://quran.asimo.io`
- **Credentials:** Enabled
- **Methods:** All (`*`)
- **Headers:** All (`*`)

---

## Core Data Endpoints

### 1. Health Check

**Endpoint:** `GET /health`

**Description:** Check server health and configuration.

**Request:** None

**Response:**
```json
{
  "ok": true,
  "model": "gpt-realtime",
  "voice": "verse",
  "stt": "gpt-4o-mini-transcribe"
}
```

**Status Codes:**
- `200` - Success

---

### 2. List All Surahs

**Endpoint:** `GET /surahs`

**Description:** Retrieve a list of all surahs with metadata.

**Request:** None

**Response:**
```json
{
  "count": 114,
  "items": [
    {
      "id": 1,
      "name_ar": "الفاتحة",
      "name_en": "Al-Fātiḥah",
      "verses_count": 7
    },
    {
      "id": 2,
      "name_ar": "البقرة",
      "name_en": "Al-Baqarah",
      "verses_count": 286
    }
    // ... more surahs
  ]
}
```

**Status Codes:**
- `200` - Success

---

### 3. Get Verses for a Surah

**Endpoint:** `GET /verses`

**Description:** Get all verses for a specific surah.

**Query Parameters:**
- `surah` (required, integer) - Surah ID (1-114)

**Example Request:**
```
GET /verses?surah=1
```

**Response:**
```json
{
  "surah": 1,
  "name_ar": "الفاتحة",
  "verses": [
    "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",
    "الْحَمْدُ لِلَّهِ رَبِّ الْعَالَمِينَ",
    "الرَّحْمَٰنِ الرَّحِيمِ",
    "مَالِكِ يَوْمِ الدِّينِ",
    "إِيَّاكَ نَعْبُدُ وَإِيَّاكَ نَسْتَعِينُ",
    "اهْدِنَا الصِّرَاطَ الْمُسْتَقِيمَ",
    "صِرَاطَ الَّذِينَ أَنْعَمْتَ عَلَيْهِمْ غَيْرِ الْمَغْضُوبِ عَلَيْهِمْ وَلَا الضَّالِّينَ"
  ]
}
```

**Status Codes:**
- `200` - Success
- `404` - Surah not found

---

### 4. Get Canonical Verse Text

**Endpoint:** `GET /canon`

**Description:** Get the canonical Arabic text for a specific verse.

**Query Parameters:**
- `verse_id` (required, string) - Verse ID in format `{surah}:{ayah}` (e.g., `"1:1"`)

**Example Request:**
```
GET /canon?verse_id=1:1
```

**Response:**
```json
{
  "verse_id": "1:1",
  "text": "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ"
}
```

**Status Codes:**
- `200` - Success
- `404` - Verse not found

---

## Evaluation Endpoints

### 5. Evaluate Text Recitation

**Endpoint:** `POST /evaluate`

**Description:** Evaluate user's text recitation against canonical verse.

**Content-Type:** `application/json`

**Request Body:**
```json
{
  "verse_id": "1:1",
  "user_transcript": "بسم الله الرحمن الرحيم",
  "user_id": "user123"  // optional, defaults to "anon"
}
```

**Response:**
```json
{
  "verse_id": "1:1",
  "reference": "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",
  "ref_norm": "بسم الله الرحمن الرحيم",
  "hyp_norm": "بسم الله الرحمن الرحيم",
  "ops": [
    {
      "type": "ok",
      "ref": "بسم",
      "hyp": "بسم",
      "i": 0,
      "j": 0
    }
    // ... word-by-word alignment operations
  ],
  "stats": {
    "ref_word_count": 4,
    "hyp_word_count": 4,
    "sub": 0,      // substitutions
    "ins": 0,      // insertions
    "del": 0,      // deletions
    "wer": 0.0     // Word Error Rate
  },
  "tips": [
    // Array of coaching tips based on common mistakes
  ],
  "rules": [
    {
      "rule": "qalqalah",
      "char": "ق",
      "context": "word context"
    },
    {
      "rule": "madd",
      "kind": "alif",
      "context": "word context"
    }
    // ... tajweed rules found in the verse
  ],
  "coach_line_ar": "ممتاز. الدقّة 100%. استمرّ على هذا الإيقاع.",
  "coach_line_en": "Excellent. Accuracy 100%. Nice pace."
}
```

**Operation Types:**
- `ok` - Words match exactly
- `sub` - Substitution (wrong word)
- `ins` - Insertion (extra word)
- `del` - Deletion (missing word)

**Tajweed Rules Detected:**
- `qalqalah` - Qalqalah letters (ق ط ب ج د) with sukun or at word end
- `idgham_ghunnah` - Idgham with ghunnah (ن followed by ي ن م و)
- `idgham_no_ghunnah` - Idgham without ghunnah (ن followed by ر ل)
- `madd` - Elongation rules (alif, ya, waw patterns)

**Status Codes:**
- `200` - Success
- `400` - Invalid verse_id or missing parameters

---

### 6. Evaluate Audio Recitation

**Endpoint:** `POST /evaluate_audio`

**Description:** Transcribe user's audio using OpenAI Whisper, then evaluate against canonical verse.

**Content-Type:** `multipart/form-data`

**Form Fields:**
- `verse_id` (required, string) - Verse ID in format `{surah}:{ayah}`
- `file` (required, file) - Audio file (supports: mp3, wav, m4a, aac, ogg, webm, etc.)
- `user_id` (optional, string) - User identifier, defaults to `"anon"`

**Example Request (curl):**
```bash
curl -X POST https://quran.asimo.io/evaluate_audio \
  -F "verse_id=1:1" \
  -F "file=@recitation.mp3" \
  -F "user_id=user123"
```

**Response:**
```json
{
  "verse_id": "1:1",
  "reference": "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",
  "transcript": "بسم الله الرحمن الرحيم",
  "ref_norm": "بسم الله الرحمن الرحيم",
  "hyp_norm": "بسم الله الرحمن الرحيم",
  "ops": [
    // Same as /evaluate endpoint
  ],
  "stats": {
    "ref_word_count": 4,
    "hyp_word_count": 4,
    "sub": 0,
    "ins": 0,
    "del": 0,
    "wer": 0.0
  },
  "tips": [
    // Coaching tips
  ],
  "timings": {
    "segments": [
      {
        "start": 0.0,
        "end": 2.5,
        "text": "بسم الله الرحمن الرحيم"
      }
    ],
    "words": [
      {
        "text": "بسم",
        "start": 0.0,
        "end": 0.5
      },
      {
        "text": "الله",
        "start": 0.5,
        "end": 1.0
      }
      // ... more words with timestamps
    ]
  },
  "rules": [
    // Tajweed rules detected in verse
  ],
  "coach_line_ar": "ممتاز. الدقّة 100%. استمرّ على هذا الإيقاع.",
  "coach_line_en": "Excellent. Accuracy 100%. Nice pace."
}
```

**Status Codes:**
- `200` - Success
- `400` - Invalid verse_id, empty audio, or missing file
- `500` - Server error (check `/var/log/quran-rtc/error-last.log`)

**Notes:**
- Audio is transcribed using OpenAI's STT model (configured via `OPENAI_STT_MODEL`)
- Word-level timestamps are provided when available
- All audio evaluations are logged to `/var/log/quran-rtc/attempts-{date}.jsonl`

---


### 7. Get Surah with Tajweed Token Colors

**Endpoint:** `GET /surah_tokens`

**Description:** Retrieve a full surah with each word tagged by tajweed rules and assigned a color from the palette. This enables visual highlighting of tajweed rules in the UI.

**Query Parameters:**
- `surah` (required, integer) - Surah number (1-114)

**Example Request:**
```bash
curl -X GET "https://quran.asimo.io/surah_tokens?surah=1"
```

**Response:**
```json
{
  "surah": 1,
  "name_ar": "الفاتحة",
  "ayahs": [
    {
      "ayah": 1,
      "tokens": [
        {
          "t": "بِسْمِ",
          "rules": ["tafkhim"],
          "color": "#B71C1C"
        },
        {
          "t": "اللَّهِ",
          "rules": ["madd_badal", "tafkhim"],
          "color": "#1E88E5"
        },
        {
          "t": "الرَّحْمَٰنِ",
          "rules": ["qalqalah", "tafkhim"],
          "color": "#E53935"
        },
        {
          "t": "الرَّحِيمِ",
          "rules": ["tafkhim"],
          "color": "#B71C1C"
        }
      ]
    }
    // ... more ayahs
  ],
  "palette": {
    "madd_lazim": "#8E24AA",
    "madd_muttasil": "#EF6C00",
    "madd_munfasil": "#00897B",
    "madd_badal": "#1E88E5",
    "ghunnah": "#2E7D32",
    "ikhfa": "#EC407A",
    "idgham_ghunnah": "#FBC02D",
    "idgham_no_ghunnah": "#FFE082",
    "izhar": "#90A4AE",
    "iqlab": "#6D4C41",
    "qalqalah": "#E53935",
    "tafkhim": "#B71C1C",
    "tarqiq": "#64B5F6",
    "waqf": "#9E9E9E",
    "default": "#111111"
  }
}
```

**Tajweed Rules Tagged:**
- `madd_lazim` - Madd lazim (≈6 counts) - e.g., alif with madda (آ)
- `madd_muttasil` - Madd muttasil (≈4-5 counts) - e.g., fatha + alif within word
- `madd_munfasil` - Madd munfasil (≈4-5 counts) - e.g., kasra + ya sukun, damma + waw sukun
- `madd_badal` - Madd badal (≈2 counts) - substitution madd
- `ghunnah` - Ghunnah (nasal sound)
- `ikhfa` - Ikhfa (concealment of noon sakinah/tanween)
- `idgham_ghunnah` - Idgham with ghunnah (ن followed by ي ن م و)
- `idgham_no_ghunnah` - Idgham without ghunnah (ن followed by ر ل)
- `izhar` - Izhar (clear pronunciation)
- `iqlab` - Iqlab (conversion of noon sakinah to meem)
- `qalqalah` - Qalqalah (echoing sound on ق ط ب ج د)
- `tafkhim` - Tafkhim (heavy/emphatic letters)
- `tarqiq` - Tarqiq (light letters)
- `waqf` - Waqf marks (pause signs)

**Token Structure:**
- `t` - The word token with full diacritics (Uthmani script)
- `rules` - Array of rule IDs detected for this word
- `color` - Hex color code from palette (based on first rule detected)

**Status Codes:**
- `200` - Success
- `404` - Surah not found
- `500` - Tokenization error

**Notes:**
- Colors can be customized by editing `/opt/quran-rtc/data/palette.json`
- Rule detection is heuristic-based (text-pattern matching on diacritics)
- The first detected rule determines the token's color
- Palette colors are placeholders; adjust to match your Tajweed PDF visual guide

**Frontend Integration:**
1. Fetch surah with `GET /surah_tokens?surah=1`
2. Display each ayah with tokens colored by `color` field
3. Show rule labels on hover/tap using `rules` array
4. Use palette legend to explain colors to users

---

### 8. Evaluate Multi-Ayah Span with Madd Duration Checks

**Endpoint:** `POST /evaluate_span`

**Description:** Evaluate a recorded audio span covering one or more consecutive ayahs. Returns per-ayah evaluation blocks with word alignment, tajweed rules, and madd duration analysis (comparing expected vs. actual elongation counts).

**Content-Type:** `multipart/form-data`

**Form Fields:**
- `surah` (required, integer) - Surah number
- `start_ayah` (required, integer) - First ayah in the span
- `end_ayah` (required, integer) - Last ayah in the span (inclusive)
- `file` (required, file) - Audio file containing the recitation of the span
- `user_id` (optional, string) - User identifier, defaults to `"span"`

**Example Request:**
```bash
# Evaluate Surah 1, Ayahs 1-3
curl -X POST https://quran.asimo.io/evaluate_span \
  -F "surah=1" \
  -F "start_ayah=1" \
  -F "end_ayah=3" \
  -F "file=@my_fatiha_1_3.mp3" \
  -F "user_id=user123"
```

**Response:**
```json
{
  "surah": 1,
  "blocks": [
    {
      "ayah": 1,
      "stats": {
        "ref_word_count": 4,
        "hyp_word_count": 4,
        "sub": 0,
        "ins": 0,
        "del": 0,
        "wer": 0.0
      },
      "ops": [
        {
          "type": "ok",
          "ref": "بسم",
          "hyp": "بسم"
        }
        // ... more alignment operations
      ],
      "rules": [
        {
          "rule": "madd_badal",
          "context": "اللَّهِ"
        },
        {
          "rule": "tafkhim",
          "context": "الرَّحْمَٰنِ"
        }
      ],
      "madd_checks": [
        {
          "tokenIndex": 1,
          "type": "madd_badal",
          "expected_counts": 2,
          "est_counts": 2.1,
          "delta_ms": -18
        }
      ],
      "coach_line_ar": "ممتاز (100%). المَدّ هنا ≈2 حركات؛ قرأت ≈2.1. زد/خفف بنحو 18ms."
    }
    // ... more ayah blocks
  ],
  "baseline_ms": 180
}
```

**Response Fields:**
- `surah` - The surah number
- `blocks` - Array of per-ayah evaluation results
  - `ayah` - Ayah number in this block
  - `stats` - Word-level accuracy statistics (same as single-verse eval)
  - `ops` - Word-by-word alignment operations
  - `rules` - Tajweed rules detected in this ayah
  - `madd_checks` - Array of madd duration analysis:
    - `tokenIndex` - Word position in ayah (0-indexed)
    - `type` - Type of madd (e.g., `madd_lazim`, `madd_muttasil`, `madd_badal`)
    - `expected_counts` - Expected duration in harakat counts (e.g., 2, 4.5, 6)
    - `est_counts` - Estimated duration from STT timestamps
    - `delta_ms` - Difference in milliseconds (positive = too short, negative = too long)
  - `coach_line_ar` - Synthesized Arabic feedback with specific madd guidance if needed
- `baseline_ms` - Median word duration in ms (used as tempo baseline for madd calculations)

**Madd Duration Analysis:**
- The endpoint uses STT word timestamps to estimate how many harakat counts the user held each madd
- Expected counts:
  - `madd_lazim`: 6 counts
  - `madd_muttasil`, `madd_munfasil`: 4.5 counts
  - `madd_badal`: 2 counts
- The `delta_ms` field tells the user how many milliseconds to add/reduce
- This is a coarse estimate; can be refined with phoneme-level timing

**Status Codes:**
- `200` - Success
- `400` - Invalid parameters (missing fields, invalid ayah range)
- `404` - Surah not found
- `500` - Evaluation error (STT failure, alignment issues)

**Notes:**
- Audio must contain the full span from `start_ayah` to `end_ayah` (inclusive)
- The alignment is **greedy sequential** across ayahs (not per-ayah boundaries)
- If STT produces extra/missing words, alignment may drift; review `ops` for diagnosis
- Madd count estimates use the median word duration as a tempo baseline
- Coach lines prioritize madd feedback if present, then tajweed rules, then general encouragement

**Frontend Integration:**
1. Record a multi-ayah span (e.g., user recites Ayahs 1-7 of Al-Fatiha in one go)
2. POST the audio with surah/ayah range
3. Display per-ayah results:
   - Show WER score and coach line for each ayah
   - Highlight words with errors (using `ops`)
   - Visualize madd duration corrections (e.g., "Hold the madd 50ms longer")
4. Use `baseline_ms` to show tempo feedback ("Your pace: 180ms/word")

---


## Audio Management Endpoints

### 9. Upload Canonical Audio (Multi-Reciter)

**Endpoint:** `POST /audio`

**Description:** Upload reference audio for a verse with optional reciter specification.

**Content-Type:** `multipart/form-data`

**Form Fields:**
- `verse_id` (required, string) - Verse ID in format `{surah}:{ayah}`
- `file` (required, file) - Audio file (supports: mp3, wav, m4a, aac, ogg)
- `reciter` (optional, string) - Reciter identifier, defaults to `"default"`

**Example Requests:**
```bash
# Upload to default reciter
curl -X POST https://quran.asimo.io/audio \
  -F "verse_id=1:1" \
  -F "file=@canonical_1_1.mp3"

# Upload to specific reciter
curl -X POST https://quran.asimo.io/audio \
  -F "verse_id=1:1" \
  -F "file=@mishary_1_1.mp3" \
  -F "reciter=mishary"
```

**Response:**
```json
{
  "ok": true,
  "stored": "/opt/quran-rtc/audio/default/1/1.mp3",
  "reciter": "default"
}
```

**Status Codes:**
- `200` - Success
- `400` - Invalid verse_id, empty file, or missing parameters

**Storage Structure:**
```
/opt/quran-rtc/audio/
  ├── default/
  │   ├── 1/
  │   │   ├── 1.mp3
  │   │   ├── 2.mp3
  │   │   └── ...
  │   ├── 2/
  │   │   ├── 1.mp3
  │   │   └── ...
  ├── mishary/
  │   ├── 1/
  │   │   ├── 1.mp3
  │   │   └── ...
  ├── husary/
  │   └── ...
```

---

### 10. Get Canonical Audio (Multi-Reciter)

**Endpoint:** `GET /audio`

**Description:** Retrieve reference audio for a verse from a specific reciter.

**Query Parameters:**
- `verse_id` (required, string) - Verse ID in format `{surah}:{ayah}`
- `reciter` (optional, string) - Reciter identifier, defaults to `"default"`

**Example Requests:**
```
# Get default reciter
GET /audio?verse_id=1:1

# Get specific reciter
GET /audio?verse_id=1:1&reciter=mishary
```

**Response:**
- **Content-Type:** `audio/mpeg` (for mp3), `audio/wav` (for wav), or `audio/aac` (for aac)
- **Body:** Audio file binary data

**Status Codes:**
- `200` - Success (audio file returned)
- `400` - Invalid verse_id format
- `404` - No audio found for this verse and reciter

**Notes:**
- Automatically selects first available format: mp3, wav, m4a, aac, or ogg
- Returns audio file as streaming response
- If reciter not found or no audio for that reciter, returns 404

---

### 11. List Available Reciters

**Endpoint:** `GET /reciters`

**Description:** Get a list of all available reciter identifiers.

**Request:** None

**Response:**
```json
{
  "count": 3,
  "items": [
    "default",
    "husary",
    "mishary"
  ]
}
```

**Status Codes:**
- `200` - Success

**Notes:**
- Returns reciter folder names from `/opt/quran-rtc/audio/`
- "default" is always included in the list
- Reciters are sorted alphabetically
- Hidden folders (starting with `.`) are excluded
- Use these identifiers for the `reciter` parameter in `/audio` endpoints

**Usage:**
1. GET `/reciters` to discover available reciters
2. Use reciter name in GET `/audio?verse_id=1:1&reciter=mishary`
3. Or use in POST `/audio` with `-F "reciter=mishary"`

---

## Real-Time Communication Endpoints

### 10. WebRTC Offer (Original)

**Endpoint:** `POST /rtc/offer`

**Description:** Proxy WebRTC SDP offer to OpenAI Realtime API.

**Content-Type:** `application/sdp`

**Request Body:**
```
SDP offer string (raw text)
```

**Response:**
- **Content-Type:** `application/sdp`
- **Body:** SDP answer string

**Status Codes:**
- `200` - Success
- `400+` - OpenAI API error

**Notes:**
- Uses server's OpenAI API key
- Connects to: `https://api.openai.com/v1/realtime?model={MODEL}&voice={VOICE}`
- Timeout: 30 seconds (connect: 10 seconds)

---

### 11. WebRTC Offer (Ephemeral)

**Endpoint:** `POST /rtc/offer_ephemeral`

**Description:** Mint ephemeral client secret and proxy WebRTC SDP offer to OpenAI.

**Content-Type:** `application/sdp`

**Request Body:**
```
SDP offer string (raw text)
```

**Response:**
- **Content-Type:** `application/sdp`
- **Body:** SDP answer string

**Status Codes:**
- `200` - Success
- `400` - Proxy failed or invalid SDP
- `500` - Failed to mint ephemeral secret

**Notes:**
- First mints a 5-minute ephemeral secret from OpenAI
- Then uses that secret to establish WebRTC connection
- Falls back from `/v1/realtime` to `/v1/realtime/calls` if needed
- Debug logs stored in `/var/log/quran-rtc/last-offer.sdp` and `last-realtime-error.json`

---

### 12. Get Ephemeral Client Secret

**Endpoint:** `GET /realtime/ephemeral`

**Description:** Mint a short-lived client secret for direct WebRTC connection (client-side).

**Request:** None

**Response:**
```json
{
  "client_secret": "cs_abc123...",
  "session_model": "gpt-realtime",
  "session_voice": "verse",
  "raw": {
    // Full OpenAI API response
    "client_secret": {
      "value": "cs_abc123...",
      "expires_at": 1234567890
    }
  }
}
```

**Status Codes:**
- `200` - Success
- `400+` - OpenAI API error

**Notes:**
- Secret expires in 5 minutes (300 seconds)
- Use this for direct client-to-OpenAI WebRTC connections
- Eliminates need for server-side SDP proxy

---

### 13. Get Persona Instructions

**Endpoint:** `GET /persona`

**Description:** Get system instructions for the AI coach persona.

**Request:** None

**Response:**
```json
{
  "type": "session.update",
  "session": {
    "instructions": "أنت مدرّب تلاوة موجز. لا تقدّم تصحيحًا من نفسك. انتظر تعليمات تصل عبر القناة البيانية ثم انطقها بإيجاز. حيِّ بإيجاز، واطلب قراءة الآية المحددة، ثم اصمت حتى تستلم ملاحظات جاهزة من النظام."
  }
}
```

**Translation:** "You are a concise recitation coach. Do not provide corrections on your own. Wait for instructions that arrive via the data channel, then speak them concisely. Greet briefly, ask to recite the specified verse, then remain silent until you receive ready-made feedback from the system."

**Purpose:** This stricter persona prevents the AI from "freestyling" corrections. It ensures the AI only relays the backend-provided `coach_line` feedback, maintaining consistency and accuracy.

**Status Codes:**
- `200` - Success

---

## Data Models

### Verse ID Format

**Format:** `"{surah}:{ayah}"`

**Examples:**
- `"1:1"` - Al-Fatiha, verse 1
- `"2:255"` - Al-Baqarah, Ayat al-Kursi
- `"114:6"` - An-Nas, verse 6

**Ranges:**
- Surah: 1-114
- Ayah: 1-286 (varies by surah)

---

### Alignment Operations

Each operation in the `ops` array has the following structure:

**OK (Match):**
```json
{
  "type": "ok",
  "ref": "بسم",
  "hyp": "بسم",
  "i": 0,    // index in reference
  "j": 0     // index in hypothesis
}
```

**Substitution (Wrong word):**
```json
{
  "type": "sub",
  "ref": "الرحمن",
  "hyp": "الرحيم",
  "i": 2,
  "j": 2
}
```

**Deletion (Missing word):**
```json
{
  "type": "del",
  "ref": "الله",
  "i": 1
}
```

**Insertion (Extra word):**
```json
{
  "type": "ins",
  "hyp": "والله",
  "j": 3
}
```

---

### Statistics Object

```json
{
  "ref_word_count": 10,     // Number of words in reference
  "hyp_word_count": 9,      // Number of words in user's recitation
  "sub": 1,                 // Number of substitutions
  "ins": 0,                 // Number of insertions
  "del": 1,                 // Number of deletions
  "wer": 0.2                // Word Error Rate: (sub+ins+del)/ref_word_count
}
```

**Word Error Rate (WER):**
- `0.0` - Perfect recitation
- `0.0 - 0.1` - Excellent (< 10% errors)
- `0.1 - 0.3` - Good (10-30% errors)
- `0.3 - 0.5` - Needs improvement (30-50% errors)
- `> 0.5` - Poor (> 50% errors)

---

### Coach Line (New!)

Both `/evaluate` and `/evaluate_audio` now return concise, actionable feedback strings in Arabic and English.

**Fields:**
- `coach_line_ar` (string) - Arabic coaching feedback
- `coach_line_en` (string) - English coaching feedback

**Format:**
```
[Score feedback] + [Tip/Rule focus] + [Encouragement]
```

**Examples:**

**Perfect recitation (WER = 0.0):**
```json
{
  "coach_line_ar": "ممتاز. الدقّة 100%. استمرّ على هذا الإيقاع.",
  "coach_line_en": "Excellent. Accuracy 100%. Nice pace."
}
```

**Good recitation with tips (WER = 0.05, has tip):**
```json
{
  "coach_line_ar": "ممتاز. الدقّة 95%. ملاحظة: ق (qāf): مفخَّمة من أقصى اللسان—لا تُرقّقها كالكاف.",
  "coach_line_en": "Excellent. Accuracy 95%. Note: ق (qāf): مفخَّمة من أقصى اللسان—لا تُرقّقها كالكاف."
}
```

**Good recitation with tajweed focus (WER = 0.12, no tips, has rules):**
```json
{
  "coach_line_ar": "جيد جدًّا. الدقّة 88%. راقب حكم qalqalah في «الْخَلْقِ».",
  "coach_line_en": "Very good. Accuracy 88%. Watch the qalqalah in \"الْخَلْقِ\"."
}
```

**Needs improvement (WER = 0.45):**
```json
{
  "coach_line_ar": "تابع التحسين. الدقّة 55%. استمرّ على هذا الإيقاع.",
  "coach_line_en": "Keep going. Accuracy 55%. Nice pace."
}
```

**Usage:**
- **Voice Mode:** The AI coach speaks these lines directly to the user
- **Text Mode:** Display as immediate feedback in the UI
- **Priority:** Tips (specific corrections) > Tajweed rules (focus areas) > General encouragement

**Benefits:**
- Consistent feedback across all evaluation modes
- Backend-driven corrections (no AI "freestyling")
- Concise, actionable guidance
- Bilingual support (Arabic/English)

---

### Tajweed Rules

**Qalqalah:**
```json
{
  "rule": "qalqalah",
  "char": "ق",           // The qalqalah letter
  "context": "الْخَلْقِ"  // Word containing the letter
}
```

**Letters:** ق ط ب ج د (qāf, ṭā', bā', jīm, dāl)

---

**Idgham with Ghunnah:**
```json
{
  "rule": "idgham_ghunnah",
  "context": "مِنْ يَوْمِ"  // Context showing nun sakinah before ghunnah letter
}
```

**Letters:** ي ن م و (yā', nūn, mīm, wāw)

---

**Idgham without Ghunnah:**
```json
{
  "rule": "idgham_no_ghunnah",
  "context": "مِنْ رَبِّهِمْ"
}
```

**Letters:** ر ل (rā', lām)

---

**Madd (Elongation):**
```json
{
  "rule": "madd",
  "kind": "alif",        // or "ya" or "waw"
  "context": "الْعَالَمِينَ"
}
```

---

## Error Handling

### Standard Error Response

```json
{
  "detail": "Error message description"
}
```

### Common Error Codes

**400 Bad Request:**
- Invalid `verse_id` format
- Missing required parameters
- Empty audio file
- Unknown verse_id

**404 Not Found:**
- Surah not found
- Verse not found
- Audio not found

**500 Internal Server Error:**
- OpenAI API failures
- STT transcription errors
- Server-side exceptions

**Error Logs:**
- All errors are logged to `/var/log/quran-rtc/error-last.log`
- Attempts logged to `/var/log/quran-rtc/attempts-{YYYYMMDD}.jsonl`

---

## Configuration

### Environment Variables

The backend requires the following environment variables in `/opt/quran-rtc/backend/.env`:

```bash
OPENAI_API_KEY=sk-...                           # Required
OPENAI_REALTIME_MODEL=gpt-realtime              # Default
OPENAI_REALTIME_VOICE=verse                     # Default
OPENAI_STT_MODEL=gpt-4o-mini-transcribe         # Default
QURAN_DATA=/opt/quran-rtc/data/quran.json       # Default
```

### Data Paths

- **Quran Text Data:** `/opt/quran-rtc/data/quran.json`
- **Audio Files:** `/opt/quran-rtc/audio/{surah}/{ayah}.{ext}`
- **Logs:** `/var/log/quran-rtc/`
  - `attempts-{YYYYMMDD}.jsonl` - All evaluation attempts
  - `error-last.log` - Last error traceback
  - `last-offer.sdp` - Last WebRTC SDP offer
  - `last-realtime-error.json` - Last realtime API error

---

## Rate Limits & Performance

- **OpenAI API:** Subject to OpenAI's rate limits
- **Audio Upload:** Max file size typically 25MB (configurable via FastAPI)
- **STT Timeout:** 90 seconds (connect: 15 seconds)
- **WebRTC Timeout:** 30 seconds (connect: 10 seconds)
- **Ephemeral Secret:** 20 seconds timeout

---

## WebRTC Integration Guide

### Option 1: Server-Proxied (Legacy)

1. Client creates WebRTC offer
2. POST offer SDP to `/rtc/offer` or `/rtc/offer_ephemeral`
3. Receive answer SDP
4. Set remote description and establish connection

### Option 2: Direct Client Connection (Recommended)

1. GET `/realtime/ephemeral` to obtain `client_secret`
2. Use `client_secret` to connect directly to OpenAI:
   ```
   https://api.openai.com/v1/realtime?model={model}
   ```
3. Include headers:
   ```
   Authorization: Bearer {client_secret}
   OpenAI-Beta: realtime=v1
   ```

**Benefits:**
- Lower latency
- Reduced server load
- Direct peer connection

---

## Arabic Text Processing

### Normalization Rules

The backend normalizes Arabic text before comparison to improve accuracy:

1. **Remove Tatweel:** `ـ` (U+0640)
2. **Remove Quranic Marks:** Stop/pause marks and ornamental marks (U+06D6-U+06ED, U+0617-U+061A, U+0657-U+065F, U+0670)
3. **Unify Alif/Hamza Forms:** أ إ آ ٱ ء → ا (all normalized to plain alif)
4. **Unify Alif Maqsurah:** ى → ي (common STT/orthography mismatch)
5. **Unify Ta Marbuta:** ة → ه (lenient comparison to avoid false negatives)
6. **Strip Diacritics:** All harakat (َ ِ ُ ً ٍ ٌ ْ ّ, etc.)
7. **Keep Only Arabic:** Letters in range U+0600 to U+06FF and spaces
8. **Collapse Whitespace:** Multiple spaces → single space

**Example:**
```
Original:  "بِسْمِ اللَّـهِ الرَّحْمَٰنِ الرَّحِيمِ"
Normalized: "بسم الله الرحمن الرحيم"

With variants: "إِنَّ اللّٰهَ غَفُورٌ رَّحِيمٌ"
Normalized: "ان الله غفور رحيم"
```

**Benefits:**
- Handles STT transcription variations (e.g., ى/ي confusion)
- Accounts for different Quranic script styles (Uthmani vs. Simplified)
- Reduces false negatives from hamza/alif variations
- More forgiving for ta marbuta pronunciation at pause

### Word Alignment Algorithm

- Uses **Levenshtein distance** (edit distance) with dynamic programming
- Compares normalized word tokens
- Generates operation sequence: OK, SUB, INS, DEL
- Calculates Word Error Rate (WER)

---

## Example Flutter Integration

### Fetching Surahs

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

Future<List<Surah>> fetchSurahs() async {
  final response = await http.get(
    Uri.parse('https://quran.asimo.io/surahs'),
  );

  if (response.statusCode == 200) {
    final data = json.decode(response.body);
    return (data['items'] as List)
        .map((item) => Surah.fromJson(item))
        .toList();
  } else {
    throw Exception('Failed to load surahs');
  }
}
```

### Evaluating Audio

```dart
import 'package:http/http.dart' as http;
import 'dart:io';

Future<Map<String, dynamic>> evaluateAudio(
  String verseId,
  File audioFile,
  String userId,
) async {
  var request = http.MultipartRequest(
    'POST',
    Uri.parse('https://quran.asimo.io/evaluate_audio'),
  );

  request.fields['verse_id'] = verseId;
  request.fields['user_id'] = userId;
  request.files.add(
    await http.MultipartFile.fromPath('file', audioFile.path),
  );

  final response = await request.send();
  final responseData = await response.stream.bytesToString();

  if (response.statusCode == 200) {
    return json.decode(responseData);
  } else {
    throw Exception('Failed to evaluate audio');
  }
}
```

### Playing Canonical Audio

```dart
import 'package:audioplayers/audioplayers.dart';

Future<void> playCanonicalAudio(String verseId) async {
  final player = AudioPlayer();
  await player.play(
    UrlSource('https://quran.asimo.io/audio?verse_id=$verseId'),
  );
}
```

---

## Changelog

**Version:** Current (as of October 2025)

**Recent Changes:**
- Added `/rtc/offer_ephemeral` endpoint for improved WebRTC reliability
- Added word-level timestamps in `/evaluate_audio` response
- Enhanced tajweed rule detection (qalqalah, idgham, madd)
- Improved error logging and diagnostics

---

## Support & Contact

For issues or questions about the API, please contact the development team or check the logs at `/var/log/quran-rtc/`.

---

**End of Documentation**
