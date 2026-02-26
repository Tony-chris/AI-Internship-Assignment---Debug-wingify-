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
## Importing libraries and files
from crewai import Task

from agents import financial_analyst, verifier, investment_advisor, risk_assessor
from tools import search_tool, FinancialDocumentTool


## Document verification task — runs first to validate the uploaded file
verification = Task(
    description=(
        "Verify the financial document located at {file_path} for completeness and authenticity.\n"
        "1. Check if the uploaded file is a valid financial document (PDF format)\n"
        "2. Verify document structure and required financial sections\n"
        "3. Identify any missing or incomplete information\n"
        "4. Validate data consistency and format\n"
        "5. Provide verification status and recommendations before further analysis proceeds"
    ),
    expected_output=(
        "Document verification report:\n"
        "- Document type and format verification\n"
        "- Completeness assessment\n"
        "- Data quality and consistency check\n"
        "- Identification of any issues or missing sections\n"
        "- Overall verification status and recommendations\n"
        "- File path and metadata confirmation"
    ),
    agent=verifier,  # Fixed: was incorrectly assigned to financial_analyst
    tools=[FinancialDocumentTool.read_data_tool],
    async_execution=False
)


## Creating a task to analyze financial documents
analyze_financial_document = Task(
    description=(
        "Analyze the financial document at {file_path} to address the user's query: {query}\n"
        "1. Read and understand the financial document thoroughly\n"
        "2. Extract key financial metrics, ratios, and performance indicators\n"
        "3. Identify trends, strengths, and areas of concern\n"
        "4. Provide context using market research when relevant\n"
        "5. Address the specific user query with data-driven insights\n"
        "6. Ensure all analysis is based on factual information from the document"
    ),
    expected_output=(
        "A comprehensive financial analysis report including:\n"
        "- Executive summary of key findings\n"
        "- Detailed financial metrics analysis\n"
        "- Key performance indicators and trends\n"
        "- Strengths and potential concerns\n"
        "- Specific answers to the user's query\n"
        "- Professional disclaimers about investment risks"
    ),
    agent=financial_analyst,
    tools=[FinancialDocumentTool.read_data_tool, search_tool],
    async_execution=False,
)


## Creating an investment analysis task
investment_analysis = Task(
    description=(
        "Based on the financial document at {file_path}, provide investment insights for the query: {query}\n"
        "1. Evaluate the company's financial health and performance using document data\n"
        "2. Assess growth prospects and competitive position\n"
        "3. Identify potential investment opportunities and risks grounded in the document\n"
        "4. Consider market conditions and industry trends\n"
        "5. Provide balanced investment perspective with appropriate risk considerations and disclaimers"
    ),
    expected_output=(
        "Investment analysis report containing:\n"
        "- Investment thesis and rationale based on document data\n"
        "- Key financial strengths and weaknesses\n"
        "- Growth prospects and market opportunities\n"
        "- Risk factors and mitigation strategies\n"
        "- Investment recommendation with clear disclaimers\n"
        "- Relevant market context and comparisons"
    ),
    agent=investment_advisor,
    tools=[FinancialDocumentTool.read_data_tool, search_tool],
    async_execution=False,
)


## Creating a risk assessment task
risk_assessment = Task(
    description=(
        "Conduct comprehensive risk assessment based on the financial document at {file_path} "
        "and user query: {query}\n"
        "1. Identify financial risks from the document analysis\n"
        "2. Assess market and industry-specific risks\n"
        "3. Evaluate operational and strategic risks\n"
        "4. Provide risk mitigation recommendations\n"
        "5. Consider regulatory and compliance risks"
    ),
    expected_output=(
        "Risk assessment report including:\n"
        "- Comprehensive risk identification and categorization\n"
        "- Risk impact and probability assessments\n"
        "- Key risk factors specific to the company/investment\n"
        "- Risk mitigation strategies and recommendations\n"
        "- Overall risk rating with supporting rationale\n"
        "- Regulatory and compliance considerations"
    ),
    agent=risk_assessor,
    tools=[FinancialDocumentTool.read_data_tool, search_tool],
    async_execution=False,
)

## Requirements
click==8.1.7
# Keep crewai at version 0.130.0
crewai==0.130.0
crewai-tools==0.47.1
fastapi==0.110.3
google-ai-generativelanguage==0.6.4
google-api-core==2.10.0
google-api-python-client==2.131.0
google-auth==2.29.0
google-auth-httplib2==0.2.0
google-cloud-aiplatform==1.53.0
google-cloud-bigquery==3.23.1
google-cloud-core==2.4.1
google-cloud-resource-manager==1.12.3
google-cloud-storage==2.16.0
google-crc32c==1.5.0
google-generativeai==0.5.4
google-resumable-media==2.7.0
googleapis-common-protos==1.63.0
Jinja2==3.1.4
jsonschema==4.22.0
langchain-core==0.1.52
langchain-openai==0.1.8
langsmith==0.1.67
numpy==1.26.4
oauthlib==3.2.2
onnxruntime==1.18.0
openai==1.30.5
opentelemetry-api==1.25.0
opentelemetry-exporter-otlp-proto-common==1.25.0
opentelemetry-exporter-otlp-proto-grpc==1.25.0
opentelemetry-exporter-otlp-proto-http==1.25.0
opentelemetry-instrumentation==0.46b0
opentelemetry-instrumentation-asgi==0.46b0
opentelemetry-instrumentation-fastapi==0.46b0
opentelemetry-proto==1.25.0
opentelemetry-sdk==1.25.0
opentelemetry-semantic-conventions==0.46b0
opentelemetry-util-http==0.46b0
pandas==2.2.2
pillow==10.3.0
pip==24.0
protobuf==4.25.3
pydantic==1.10.13
pydantic_core==2.8.0
python-dotenv==1.0.0
PyPDF2==3.0.1
uvicorn==0.29.0

# Tools
## Importing libraries and files
import os
from dotenv import load_dotenv
load_dotenv()

from crewai_tools import SerperDevTool
import PyPDF2

## Creating search tool
search_tool = SerperDevTool()


## Creating custom pdf reader tool
class FinancialDocumentTool:
    @staticmethod
    def read_data_tool(path: str = 'data/sample.pdf') -> str:
        """Tool to read data from a PDF file.

        Args:
            path (str, optional): Path of the PDF file. Defaults to 'data/sample.pdf'.

        Returns:
            str: Full financial document content as plain text.
        """
        try:
            with open(path, 'rb') as file:
                pdf_reader = PyPDF2.PdfReader(file)
                full_report = ""

                for page in pdf_reader.pages:
                    content = page.extract_text()
                    if content:
                        # Clean and format the content
                        content = content.replace('\n\n', '\n').strip()
                        full_report += content + "\n"

                if not full_report.strip():
                    return "Warning: No text content could be extracted from the PDF."

                return full_report

        except FileNotFoundError:
            return f"Error: File not found at path '{path}'"
        except Exception as e:
            return f"Error reading PDF file: {str(e)}"


## Creating Investment Analysis Tool
class InvestmentTool:
    @staticmethod
    def analyze_investment_tool(financial_document_data: str) -> dict:
        """Analyze financial document data for investment insights.

        Args:
            financial_document_data (str): Raw text from a financial document.

        Returns:
            dict: Cleaned data summary and status.
        """
        processed_data = financial_document_data

        # Basic data cleaning — collapse multiple spaces
        processed_data = ' '.join(processed_data.split())

        return {
            "data_summary": processed_data[:1000] + "..." if len(processed_data) > 1000 else processed_data,
            "status": "ready_for_analysis"
        }


## Creating Risk Assessment Tool
class RiskTool:
    @staticmethod
    def create_risk_assessment_tool(financial_document_data: str) -> dict:
        """Prepare financial document data for risk assessment.

        Args:
            financial_document_data (str): Raw text from a financial document.

        Returns:
            dict: Risk data payload and readiness flag.
        """
        return {
            "risk_data": financial_document_data,
            "assessment_ready": True
        }

# Agents
## Importing libraries and files
import os
from dotenv import load_dotenv
load_dotenv()

from crewai import Agent
from langchain_openai import ChatOpenAI

from tools import search_tool, FinancialDocumentTool

### Loading LLM
llm = ChatOpenAI(
    model="gpt-3.5-turbo",
    temperature=0.1,
    api_key=os.getenv("OPENAI_API_KEY")
)


# Creating an Experienced Financial Analyst agent
financial_analyst = Agent(
    role="Senior Financial Analyst",
    goal="Provide accurate and comprehensive financial analysis based on the user query: {query}",
    verbose=True,
    memory=True,
    backstory=(
        "You are a seasoned financial analyst with over 15 years of experience in analyzing "
        "corporate financial statements, investment opportunities, and market trends. "
        "You have a CFA designation and have worked with Fortune 500 companies. "
        "You provide data-driven insights, identify key financial metrics, and offer "
        "balanced investment perspectives based on thorough analysis. "
        "You always consider regulatory compliance and provide disclaimers when appropriate."
    ),
    tools=[FinancialDocumentTool.read_data_tool],  # Fixed: was 'tool=' (typo)
    llm=llm,
    max_iter=3,
    max_rpm=10,
    allow_delegation=True
)


# Creating a document verifier agent
verifier = Agent(
    role="Financial Document Verification Specialist",
    goal="Verify the authenticity and completeness of financial documents and ensure they contain relevant financial information.",
    verbose=True,
    memory=True,
    backstory=(
        "You are a meticulous document verification expert with expertise in financial reporting standards. "
        "You have worked in regulatory compliance for major financial institutions and are skilled at "
        "identifying authentic financial documents, checking for completeness, and flagging any inconsistencies. "
        "You ensure all documents meet professional standards before analysis."
    ),
    tools=[FinancialDocumentTool.read_data_tool],
    llm=llm,
    max_iter=2,
    max_rpm=10,
    allow_delegation=False
)


investment_advisor = Agent(
    role="Investment Strategy Advisor",
    goal="Provide balanced investment recommendations based on financial analysis while considering risk tolerance and market conditions.",
    verbose=True,
    memory=True,
    backstory=(
        "You are a certified investment advisor with deep expertise in portfolio management "
        "and investment strategy. You have helped individuals and institutions make informed "
        "investment decisions for over 12 years. You always consider risk-return profiles, "
        "diversification principles, and align recommendations with investor goals. "
        "You provide clear disclaimers about investment risks and market volatility."
    ),
    llm=llm,
    max_iter=3,
    max_rpm=10,
    allow_delegation=False
)


risk_assessor = Agent(
    role="Risk Management Specialist",
    goal="Conduct thorough risk analysis and provide comprehensive risk assessments for investment decisions.",
    verbose=True,
    memory=True,
    backstory=(
        "You are a risk management professional with extensive experience in financial risk assessment. "
        "You specialize in identifying market risks, operational risks, and financial risks in investment opportunities. "
        "Your analysis helps investors understand potential downsides and implement appropriate risk mitigation strategies. "
        "You use quantitative models and qualitative analysis to provide balanced risk perspectives."
    ),
    llm=llm,
    max_iter=3,
    max_rpm=10,
    allow_delegation=False
)

# main
from fastapi import FastAPI, File, UploadFile, Form, HTTPException
import os
import uuid

from crewai import Crew, Process
from agents import financial_analyst, investment_advisor, risk_assessor, verifier
from task import analyze_financial_document, investment_analysis, risk_assessment, verification

app = FastAPI(
    title="Financial Document Analyzer",
    description="AI-powered financial document analysis using multi-agent CrewAI pipeline.",
    version="1.0.0"
)


def run_crew(query: str, file_path: str = "data/sample.pdf"):
    """Run the full financial analysis crew sequentially."""
    try:
        financial_crew = Crew(
            agents=[verifier, financial_analyst, investment_advisor, risk_assessor],
            tasks=[verification, analyze_financial_document, investment_analysis, risk_assessment],
            process=Process.sequential,
            verbose=True
        )

        # Fixed: file_path now correctly passed into crew kickoff inputs
        result = financial_crew.kickoff({'query': query, 'file_path': file_path})
        return result

    except Exception as e:
        raise Exception(f"Crew execution failed: {str(e)}")


@app.get("/")
async def root():
    """Health check endpoint."""
    return {"message": "Financial Document Analyzer API is running", "status": "healthy"}


@app.get("/health")
async def health_check():
    """Detailed health check endpoint."""
    return {
        "status": "healthy",
        "service": "Financial Document Analyzer",
        "version": "1.0.0",
        "data_directory": os.path.exists("data")
    }


# Fixed: renamed from 'analyze_financial_document' to avoid collision with imported task of same name
@app.post("/upload-and-analyze")
async def upload_and_analyze_document(
    file: UploadFile = File(...),
    query: str = Form(default="Provide a comprehensive financial analysis of this document")
):
    """Upload a PDF financial document and receive a comprehensive AI-powered analysis.

    Args:
        file: PDF file to analyze.
        query: Optional specific question or analysis focus.

    Returns:
        JSON with analysis results and professional disclaimer.
    """

    # Validate file type
    if not file.filename.lower().endswith('.pdf'):
        raise HTTPException(status_code=400, detail="Only PDF files are supported.")

    file_id = str(uuid.uuid4())
    file_path = f"data/financial_document_{file_id}.pdf"

    try:
        # Ensure data directory exists
        os.makedirs("data", exist_ok=True)

        # Save uploaded file
        content = await file.read()
        if len(content) == 0:
            raise HTTPException(status_code=400, detail="Empty file uploaded.")

        with open(file_path, "wb") as f:
            f.write(content)

        # Validate and clean query
        if not query or query.strip() == "":
            query = "Provide a comprehensive financial analysis of this document"

        query = query.strip()

        # Process the financial document through all agents
        response = run_crew(query=query, file_path=file_path)

        return {
            "status": "success",
            "query": query,
            "analysis": str(response),
            "file_processed": file.filename,
            "file_size": len(content),
            "disclaimer": (
                "This analysis is for informational purposes only and should not be considered "
                "as financial advice. Please consult with a qualified financial advisor before "
                "making investment decisions."
            )
        }

    except HTTPException:
        raise
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error processing financial document: {str(e)}")

    finally:
        # Clean up uploaded file after processing
        if os.path.exists(file_path):
            try:
                os.remove(file_path)
            except Exception:
                pass  # Ignore cleanup errors — file will be cleared on next restart


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000, reload=True)


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


## ⚠️ Disclaimer

All analysis produced by this system is **for informational purposes only** and does not constitute financial advice. Always consult a qualified financial advisor before making investment decisions. The AI agents provide analysis based on document content and publicly available information.
