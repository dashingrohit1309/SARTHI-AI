# 💧 Sarthi: India Groundwater Voice Assistant (Sarvam AI)

Ask about India's groundwater **by voice, in any of 22 Indian languages**, and hear the answer spoken back.
Built entirely on **Sarvam AI** APIs.

> "महाराष्ट्र में भूजल की स्थिति क्या है?" → *"Maharashtra's groundwater is safe: the stage of development is 54.99%…"* (spoken in Hindi)

## How it works

```
🎤 Voice ──► Saaras v4 (speech-to-text, auto language detection, 22 languages)
              │
              ▼
        Groundwater RAG pipeline (groundwater_rag.py)
          1. Sarvam 105B         → English rewrite + SQL query
          2. e5-large-v2 + Chroma → passages from the CGWB groundwater report
          3. SQLite              → state-wise 2020 groundwater figures
          4. Sarvam 105B Conversations → answer + state figures, in the user's language
              │
              ▼
🔊 Bulbul v3 (text-to-speech) ──► spoken answer
```

The pipeline is rebuilt from our earlier SIH project
[SIH-chatbot-for-INDIA-groundwater](https://github.com/karanw330/SIH-chatbot-for-INDIA-groundwater),
with Google Gemini replaced by Sarvam models.

## Features

- 🗣️ **Hands-free voice agent**: tap once and talk. It detects the end of your turn, answers aloud, lets you interrupt it, and listens again.
- 🌐 **22 Indian languages + English** for speech and answers. Spoken replies in the 11 languages Bulbul supports; text in the rest.
- 🔎 **Text-to-SQL** over CGWB state-wise data (draft, recharge, availability, stage of development).
- 📚 **Semantic search** over the CGWB *Dynamic Ground Water Resources of India* report.
- 🟢🟡🟠🔴 State figures colour-coded as safe, semi-critical, critical or over-exploited.
- 🔒 Generated SQL is run read-only and SELECT-only.

## Setup

1. Python 3.10+.
2. `pip install -r requirements-rag.txt` (full pipeline with semantic search), or `pip install -r requirements.txt` (slim: SQL data only)
3. Copy `.env.example` to `.env` and set `SARVAM_API_KEY` (from https://dashboard.sarvam.ai).
4. `python app.py`, then open http://localhost:8000 in Chrome or Edge and allow the microphone.

The first start downloads the `intfloat/e5-large-v2` embedding model (~1.3 GB) and builds the vector index in the background.
Until it's ready, answers use the SQL data only.

## Deploy on Vercel & install on mobile

- Vercel deploys `app.py` (FastAPI) using the slim `requirements.txt`. The embedding model is too large for Vercel's 250 MB function limit, so the hosted version answers from the SQL data only.
- Set `SARVAM_API_KEY` in the Vercel project's environment variables.
- **Install on Android:** open the site in Chrome, then tap **📲 Install app** (or menu → *Install app*). Chrome builds a WebAPK that appears on the home screen like a normal app. On iPhone: Safari → Share → *Add to Home Screen*.

## Example questions

| Ask | Expected |
|---|---|
| Which states are over-exploiting groundwater? | Punjab 164%, Rajasthan 150%, Haryana 135%, Daman & Diu 113%, Delhi 101% |
| महाराष्ट्र में भूजल की स्थिति क्या है? | Safe: 54.99% |
| தமிழ்நாட்டில் நிலத்தடி நீர் நிலை என்ன? | Semi-critical: 82.93% |
| Compare Maharashtra and Karnataka | 54.99% vs 64.85% |
| Which state has the most groundwater recharge? | Uttar Pradesh: 72.2 bcm |

## Project structure

```
app.py               FastAPI server and API routes
groundwater_rag.py   Groundwater pipeline: vector search + text-to-SQL + answer (Sarvam LLM)
sarvam_client.py     Sarvam API calls: speech-to-text, chat, text-to-speech, translate
prompts.py           Languages (22 + English), topics, prompts
static/              Web UI (index.html, app.js, style.css)
data/                CGWB state-wise CSV + report text
sarvam_hello.py      Minimal Sarvam SDK check (one chat call)
```

## API

| Route | What it does |
|---|---|
| `POST /api/chat` | `{message, language_code, doc_type: "Groundwater"}` → `{reply, map_data}` |
| `POST /api/groundwater` | `{prompt, language_code}` → `{summary, map_data}` (same contract as the original SIH `/chat`) |
| `POST /api/transcribe` | audio → `{transcript, language_code}` |
| `POST /api/speak` | `{text, language_code}` → base64 WAV clips |

## Data

- `data/groundwater_state_wise_2020.csv`: state-wise groundwater resources, 2020 (bcm; stage of development in %).
- `data/groundwater_theory.txt`: text of the CGWB report *Dynamic Ground Water Resources of India* (as on 31 March 2011).

> General information only. For official figures, refer to CGWB / India-WRIS.
