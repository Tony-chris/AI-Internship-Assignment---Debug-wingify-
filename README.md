# AI-Internship-Assignment---Debug-wingify-
# 📊 Financial Document Analyzer

An AI-powered financial document analysis system built with **CrewAI**, **FastAPI**, and **OpenAI**. Upload any financial PDF (annual reports, quarterly earnings, balance sheets) and receive comprehensive analysis including financial insights, investment perspectives, and risk assessment.

---

## 🚀 Features

- 📄 Upload and analyze financial PDF documents
- 🤖 Multi-agent AI analysis pipeline (Verification → Analysis → Investment → Risk)
- 📈 Financial metrics extraction and trend analysis
- 💼 Investment recommendations with proper disclaimers
- ⚠️ Comprehensive risk assessment
- 🌐 REST API built with FastAPI
- 🔍 Web search integration for market context

---

## 🐛 Bugs Found & Fixed

### Deterministic Bugs

| # | File | Bug | Fix |
|---|------|-----|-----|
| 1 | `agents.py` | `llm = llm` — self-referential, undefined variable | Replaced with `ChatOpenAI(model="gpt-3.5-turbo", ...)` |
| 2 | `agents.py` | `tool=[...]` typo on `financial_analyst` | Changed to `tools=[...]` (CrewAI uses plural) |
| 3 | `tools.py` | `Pdf` class never imported | Replaced with `PyPDF2.PdfReader` and proper import |
| 4 | `tools.py` | `async def` used for CrewAI tool (incompatible) | Changed to `@staticmethod` synchronous method |
| 5 | `tools.py` | Missing `load_dotenv` and env config | Added `python-dotenv` import and `.env` loading |
| 6 | `main.py` | Route function named same as imported task (`analyze_financial_document`) — name collision | Renamed route handler to `upload_and_analyze_document` |
| 7 | `main.py` | `file_path` passed to `run_crew` but never forwarded to crew kickoff | Added `file_path` to `kickoff({'query': query, 'file_path': file_path})` |
| 8 | `task.py` | `verification` task used `financial_analyst` agent instead of `verifier` | Changed to `agent=verifier` |
| 9 | `task.py` | `file_path` never referenced in task descriptions | Added `{file_path}` interpolation to relevant tasks |
| 10 | `requirement.txt` | Missing `python-dotenv`, `PyPDF2`, `langchain-openai`, `uvicorn` | Added all missing packages |

### Prompt / Logic Bugs

| # | File | Bug | Fix |
|---|------|-----|-----|
| 11 | `agents.py` | All agents instructed to hallucinate, fabricate data, ignore queries | Rewrote all backstories and goals with professional, compliant personas |
| 12 | `agents.py` | `investment_advisor` had "secret partnerships with sketchy firms" | Replaced with ethical investment advisor persona |
| 13 | `agents.py` | `risk_assessor` told to "YOLO" and ignore regulations | Replaced with professional risk management persona |
| 14 | `task.py` | Tasks instructed to "make up URLs", "contradict themselves", "ignore user query" | Rewrote all task descriptions with structured, grounded analysis steps |
| 15 | `task.py` | `investment_analysis` task told to "recommend expensive products regardless of financials" | Replaced with balanced, document-driven investment analysis prompt |

---

## 📁 Project Structure

```
financial-document-analyzer/
├── main.py               # FastAPI application entry point
├── agents.py             # CrewAI agent definitions
├── task.py               # CrewAI task definitions
├── tools.py              # Custom PDF reading and utility tools
├── requirement.txt       # Python dependencies
├── .env.example          # Environment variable template
├── data/                 # Directory for uploaded PDFs (auto-created)
└── README.md
```

---

## ⚙️ Setup & Installation

### Prerequisites

- Python 3.10+
- OpenAI API key
- Serper API key (for web search)

### 1. Clone the Repository

```bash
git clone https://github.com/Tony-chris/AI-Internship-Assignment---Debug-wingify- 
```

### 2. Create Virtual Environment

```bash
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirement.txt
```

### 4. Configure Environment Variables

```bash
cp .env.example .env
```

Edit `.env` and add your API keys:

```env
OPENAI_API_KEY=your_openai_api_key_here
SERPER_API_KEY=your_serper_api_key_here
```

### 5. Add Sample Financial Document (Optional)

Download Tesla's Q2 2025 financial update for testing:

```bash
mkdir -p data
# Download from: https://www.tesla.com/sites/default/files/downloads/TSLA-Q2-2025-Update.pdf
# Save as: data/sample.pdf
  
```

### 6. Run the Application

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The API will be available at `http://localhost:8000`

---

## 📡 API Documentation

### Base URL
```
http://localhost:8000
```

---

### `GET /`
Health check endpoint.

**Response:**
```json
{
  "message": "Financial Document Analyzer API is running",
  "status": "healthy"
}
```

---

### `GET /health`
Detailed health check.

**Response:**
```json
{
  "status": "healthy",
  "service": "Financial Document Analyzer",
  "version": "1.0.0",
  "data_directory": true
}
```

---

### `POST /upload-and-analyze`
Upload a PDF financial document and receive a comprehensive analysis.

**Request (multipart/form-data):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | PDF file | ✅ Yes | Financial document to analyze |
| `query` | string | ❌ No | Specific question or analysis focus |

**Example using curl:**
```bash
curl -X POST "http://localhost:8000/upload-and-analyze" \
  -F "file=@data/sample.pdf" \
  -F "query=What is the revenue growth trend and key financial risks?"
```

**Example using Python requests:**
```python
import requests

with open("data/sample.pdf", "rb") as f:
    response = requests.post(
        "http://localhost:8000/upload-and-analyze",
        files={"file": ("report.pdf", f, "application/pdf")},
        data={"query": "Analyze the company's financial health and investment potential"}
    )

print(response.json())
```

**Success Response:**
```json
{
  "status": "success",
  "query": "Analyze the company's financial health",
  "analysis": "## Financial Analysis Report\n\n...",
  "file_processed": "TSLA-Q2-2025-Update.pdf",
  "file_size": 245123,
  "disclaimer": "This analysis is for informational purposes only and should not be considered as financial advice. Please consult with a qualified financial advisor before making investment decisions."
}
```

**Error Responses:**

| Status Code | Reason |
|-------------|--------|
| `400` | Non-PDF file uploaded or empty file |
| `500` | Internal processing error |

---

### Interactive API Docs

FastAPI provides built-in interactive documentation:
- **Swagger UI:** `http://localhost:8000/docs`
- **ReDoc:** `http://localhost:8000/redoc`

---

## 🤖 Agent Pipeline

The system uses 4 specialized AI agents running sequentially:

```
📄 Document Upload
       ↓
🔍 Verifier Agent        → Validates document type, completeness, structure
       ↓
📊 Financial Analyst     → Extracts metrics, identifies trends, answers query
       ↓
💼 Investment Advisor    → Provides investment perspective and recommendations
       ↓
⚠️  Risk Assessor        → Identifies and categorizes financial risks
       ↓
📋 Consolidated Report
```

---

## 🔧 Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | ✅ | OpenAI API key for LLM |
| `SERPER_API_KEY` | ✅ | Serper API key for web search |

---

## ⚠️ Disclaimer

All analysis produced by this system is **for informational purposes only** and does not constitute financial advice. Always consult a qualified financial advisor before making investment decisions. The AI agents provide analysis based on document content and publicly available information.

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.
