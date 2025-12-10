


DocScanner is a robust, asynchronous document processing pipeline designed to convert complex PDF documents (such as exams, academic papers, and textbooks) into structured formats.

The system utilizes a combination of traditional Computer Vision models (RT-DETR, GhostNet) and Large Language Models (Google Gemini) to detect layouts, extract text/formulas/figures, and parse content into a semantic structure.

## 🚀 Features

*   **PDF Processing**: Upload PDFs directly or provide URLs for processing.
*   **Intelligent Extraction**:
    *   **Object Detection**: Uses RT-DETR (ONNX) to identify Figures and Tables.
    *   **Classification**: Uses GhostNet to classify image regions.
    *   **LLM Extraction**: Uses Google Gemini Pro/Flash to extract text and convert math formulas to LaTeX.
*   **Structured Parsing**: Converts raw text into a custom DSL named **SLURP** and finally into structured **JSON**.
*   **Asynchronous Architecture**: Built with FastAPI and a custom Redis-based worker system for scalable background processing.
*   **Storage Integration**: Supports local storage for temporary files and Cloudflare R2 for processed image assets.
*   **Math Support**: Full support for converting LaTeX math formulas to MathML/HTML.

## 🛠 Tech Stack

*   **Language**: Python 3.10+
*   **API Framework**: FastAPI
*   **Database**: PostgreSQL (SQLAlchemy + Alembic)
*   **Queue/Cache**: Redis
*   **ML/AI**:
    *   `onnxruntime` (RT-DETR, GhostNet)
    *   `google-genai` (Gemini API)
    *   `PyMuPDF` (PDF manipulation)
    *   `OpenCV` / `Pillow` (Image processing)
*   **Infrastructure**: Docker, Docker Compose, Supervisor

## 📂 Project Structure

```text
daominhwysi-docscanner-som/
├── app/
│   ├── main.py                 # API Entry point
│   ├── run_worker.py           # Background Worker Entry point
│   ├── db/                     # Database models and connection logic
│   ├── lib/                    # Core libraries (Redis client, Worker logic)
│   ├── ml_models/              # ONNX models (Classifier, RT-DETR)
│   ├── postprocessing/         # Conversion logic (SLURP to JSON, HTML)
│   ├── prompt/                 # LLM Prompts (XML/Text based)
│   ├── services/               # Business logic (Logging, Task creation)
│   ├── utils/                  # Utilities (Agent, Draw boxes, Upload R2)
│   └── worker/                 # Worker tasks (PDF, Image, Extraction)
├── migrations/                 # Alembic database migrations
├── weights/                    # (Create this) Store .onnx models here
├── Dockerfile                  # Container definition
├── docker-compose.yml          # Service orchestration
├── supervisord.conf            # Process manager config
└── requirements.txt            # Python dependencies
```

## ⚙️ Prerequisites

*   Docker & Docker Compose
*   A Google Gemini API Key
*   A PostgreSQL Database (or use the one in Docker)
*   A Redis instance (or use the one in Docker)
*   Cloudflare R2 credentials (for image hosting)

## 🚀 Installation & Setup

### 1. Clone the repository

### 2. Configure Environment Variables

Create a `.env` file in the root directory:

```ini
# Database
DATABASE_URL=postgresql://user:password@db_host:5432/dbname

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=your_redis_password

# AI Models
GEMINI_API_KEY=key1,key2,key3 # Supports multiple keys for rotation

# Security
X_FILE_TOKEN=your_secret_token_for_media_access

# Cloudflare R2 (Object Storage)
ENDPOINT_URL_R2=https://<accountid>.r2.cloudflarestorage.com
AWS_ACCESS_KEY_ID_R2=your_access_key
AWS_SECRET_ACCESS_KEY_R2=your_secret_key
PUBLIC_URL_R2=https://pub-<id>.r2.dev
```

### 3. Download Model Weights

Place your ONNX models in `app/ml_models/weights/`:
*   `ghostnet_classifier.onnx`
*   `rfdetr-figure.onnx`

### 4. Run with Docker Compose

The `docker-compose.yml` sets up the API, Worker (via Supervisor), and Redis.

```bash
docker-compose up --build -d
```

## 📡 API Usage

The server typically runs on `http://localhost:8000`.

### 1. Process a PDF
**Endpoint:** `POST /process-pdf`

Upload a file or provide a URL.

```bash
curl -X POST "http://localhost:8000/process-pdf" \
  -H "x-token: YOUR_X_FILE_TOKEN" \
  -F "file=@/path/to/document.pdf"
# OR
curl -X POST "http://localhost:8000/process-pdf" \
  -F "url=https://example.com/exam.pdf"
```

**Response:**
```json
{
  "task_id": "uuid-string"
}
```

### 2. Check Task Status
**Endpoint:** `GET /tasks/{task_id}`

```bash
curl "http://localhost:8000/tasks/{task_id}"
```

**Response:**
```json
{
  "id": "uuid-string",
  "type": "parseDocumentPDF",
  "status": "done",
  "result": "JSON_STRING_OR_TEXT",
  "createdAt": "ISO_DATE",
  "updatedAt": "ISO_DATE"
}
```

### 3. Direct Text Parsing (SLURP)
**Endpoint:** `POST /document-parsing`

Parses raw text/MML into structured data.

```json
{
  "text": "Raw text content..."
}
```


