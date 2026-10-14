<div align="center">

# 🌾 Harit Krishi Mitra

### *AI-Powered Agriculture Assistant for Indian Farmers*

[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)](https://supabase.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

> Empowering Indian farmers with conversational AI guidance on crops, pests, irrigation, soil health, market prices, and government schemes — powered by a local Llama model with optional RAG retrieval.

</div>

---

## ✨ Features

| Feature | Description |
|---|---|
| 🌱 **Crop Guidance** | Pest control, disease diagnosis, irrigation, soil management |
| 📈 **Market & Schemes** | Real-time market intel and government scheme awareness |
| 🔒 **Auth & Persistence** | Supabase-backed authentication and full chat history |
| 🔍 **RAG Retrieval** | Optional SearXNG integration for grounded, up-to-date answers |
| 🌦️ **Weather Enrichment** | Localized weather context for farming decisions |
| 🛡️ **Response Hardening** | Stale/late backend reply protection for reliability |
| 📞 **Smart Helplines** | Conditional helpline suggestions only when truly relevant |
| 🗣️ **Hindi & Marathi** | Devanagari detection routes answers back in the farmer's language |

---

## 🎓 Project Context

Final-year B.Tech capstone project (Computer Science and Engineering,
MIT Art, Design and Technology University, Pune), built by a four-person team.

| Contributor |
|---|
| Raj Ambavane |
| Dhruv Dubey |
| Vansh Jain |
| Shantanu Ramavat |

Development was done collaboratively on a shared machine, so the upstream
commit history sits under a single account and does not reflect individual
contribution. Upstream repository:
[RajAmbavane/Krishi-Mitra-An-Agricultural-LLM](https://github.com/RajAmbavane/Krishi-Mitra-An-Agricultural-LLM).

---

## 🗣️ Multilingual Handling

Language support is handled in the prompt layer rather than through separate
per-language models, implemented in [`backend/prompt_builder.py`](backend/prompt_builder.py):

- `contains_devanagari()` scans the query for characters in the Devanagari
  Unicode block (U+0900–U+097F).
- The system prompt then instructs the model to answer in the user's own
  language — Marathi if the query was Marathi, otherwise Hindi.
- Greeting and acknowledgement detection covers transliterated Hindi and
  Marathi (`namaste`, `shukriya`, `dhanyavad`) alongside English, and short
  acknowledgements get a quick reply without a full model call.
- The base model is **Llama 3.2 3B Instruct**, which supports Hindi among its
  officially listed languages.

---

## 🏗️ Tech Stack

```
Frontend      →  React · TypeScript · Vite · Tailwind CSS · shadcn/ui
Backend       →  FastAPI · llama-cpp-python · Python 3.10+
Auth & Data   →  Supabase
Retrieval     →  SearXNG (optional but recommended)
```

---

## 📁 Repository Structure

```text
.
├── backend/
│   ├── main.py              # FastAPI app & routes
│   ├── prompt_builder.py    # Prompt construction logic
│   ├── search_engine.py     # SearXNG RAG integration
│   ├── weather_service.py   # Weather enrichment
│   └── llama_runner.py      # Local Llama model loader
├── src/                     # React frontend source
├── finetuning/              # Fine-tuning scripts & data
├── supabase/                # DB schema & migrations
└── README.md
```

---

## ⚙️ Prerequisites

- **Node.js** 18+
- **Python** 3.10+
- A **GGUF model file** compatible with `llama-cpp-python`
- A **Supabase** project (URL + anon key)
- *(Optional)* A local **SearXNG** instance

---

## 🔑 Environment Variables

### Frontend — `.env.local`

```bash
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
VITE_BACKEND_URL=http://localhost:8000
```

### Backend — `backend/.env`

```bash
# Optional if model is at workspace root with this filename:
# Llama-3.2-3B-Instruct-Q4_K_M.gguf
LLAMA_MODEL_PATH=d:/Desktop/LLAMA3.2/Llama-3.2-3B-Instruct-Q4_K_M.gguf

# Optional: comma-separated SearXNG endpoints
SEARXNG_URLS=http://127.0.0.1:8888
SEARXNG_BACKOFF_SECONDS=60
```

---

## 🔍 RAG Setup (SearXNG)

RAG is enabled through `backend/search_engine.py` and used by `POST /ask`.

Retrieval is **restricted to a whitelist of Indian government and meteorological
domains** rather than the open web, so answers about schemes, prices and
advisories are grounded in authoritative sources: `krishi.maharashtra.gov.in`,
`mahadbt.maharashtra.gov.in`, `agricoop.gov.in`, `agmarknet.gov.in`,
`enam.gov.in`, `soilhealth.dac.gov.in`, `pmkisan.gov.in`, `msp.gov.in`,
`mkisan.gov.in`, `mausam.imd.gov.in`, plus two weather providers. Unreachable
SearXNG endpoints are backed off for a configurable interval instead of being
retried on every request.

### Option A — Use your local `searxng` folder
If you already have a local SearXNG setup, start it and ensure it serves on `http://127.0.0.1:8888`.

### Option B — Run SearXNG via Docker

```bash
docker run --rm -d --name searxng -p 8888:8080 searxng/searxng
```

Then set:
```bash
SEARXNG_URLS=http://127.0.0.1:8888
```

**Validate your setup:**
```
# Test search API
http://127.0.0.1:8888/search?q=health&format=json

# RAG health endpoint
GET http://localhost:8000/health/rag
```

> ℹ️ If SearXNG is unavailable, the app automatically falls back to the non-RAG path.

---

## 🤖 Model Setup

The backend loads the model from:
1. `LLAMA_MODEL_PATH` *(preferred)*, or
2. Workspace default: `Llama-3.2-3B-Instruct-Q4_K_M.gguf`

### Base Model

```bash
# 1. Download your GGUF model file
# 2. Place it anywhere on disk
# 3. Point the env var to it
LLAMA_MODEL_PATH=D:/models/Llama-3.2-3B-Instruct-Q4_K_M.gguf
```

### Fine-tuned Model

```bash
# After merging adapter + converting to GGUF:
LLAMA_MODEL_PATH=D:/models/krishi-mitra-finetuned-q4_k_m.gguf
```

> ⚠️ Raw LoRA/adapter files are not loaded directly. `llama_runner.py` expects a single merged GGUF file. See `finetuning/README.md` for the full training workflow.

---

## 🚀 Local Development

### 1. Frontend

```bash
npm install
npm run dev
# Runs on http://localhost:5173
```

### 2. Backend

```bash
cd backend
python -m venv venv
venv\Scripts\activate       # Windows
# source venv/bin/activate  # macOS/Linux

pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### 3. End-to-End Check

```
✅ Backend health  →  http://localhost:8000/health/rag
✅ Frontend        →  npm run dev
✅ Test a query    →  Verify answer + sources in UI
```

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/ask` | Main chat endpoint |
| `GET` | `/health/rag` | Retrieval health check |

---

## 🧪 Quality Checks

```bash
npm run lint
npm run build
```

---

## 🌐 Deploy Notes

- 🔒 Lock down CORS origins in `backend/main.py` for production
- 🔑 Store all secrets in environment variables — never in source code
- 🛡️ Use HTTPS and authenticated Supabase project keys

---

## 📚 Further Reading

- [`finetuning/README.md`](finetuning/README.md) — Fine-tuning workflow
- [`finetuning/data/DATASET_SCHEMA.md`](finetuning/data/DATASET_SCHEMA.md) — Dataset schema
- [`finetuning/scripts/`](finetuning/scripts/) — Training scripts

---

## 📄 License

```
MIT License

Copyright (c) 2025 Harit Krishi Mitra Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">

</div>
