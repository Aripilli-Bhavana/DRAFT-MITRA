# DraftMitra – AI-Powered Government Form Assistant

## 🎯 Overview

DraftMitra is an AI-powered full-stack web application that simplifies the process of filling Indian government forms. Built for the "Generative AI & LLMs for Governance" hackathon, it leverages local LLMs to guide users step-by-step while ensuring data privacy through offline processing.

## 🌟 Key Features

* **Form Analysis**: Upload government forms (PDF) and get structured AI-driven analysis
* **Interactive Assistant**: Field-by-field guidance through an intuitive AI chat interface
* **Multilingual Support**: Real-time translation between Hindi and English
* **Local LLMs**: Works offline using models like Mistral and LLaMA 3 via Ollama
* **Privacy-First**: User data stays entirely on-device
* **Responsive UI**: Mobile-first design with dark/light mode support

## 🛠️ Tech Stack

### Frontend

* React 18 with TypeScript
* Tailwind CSS for utility-first styling
* Lucide React for modern icons
* Context API for global state

### Backend

* FastAPI for high-performance APIs
* Python with async support
* Ollama for running local LLMs
* pdfplumber for text extraction
* ReportLab for PDF generation

## 🚀 Quick Start

### Prerequisites

* Node.js 18+
* Python 3.8+
* Ollama installed locally

### Frontend Setup

```bash
npm install
npm run dev
```

### Backend Setup

```bash
# Install dependencies
pip install fastapi uvicorn pdfplumber reportlab python-multipart

# Run backend
uvicorn main:app --reload --port 8000

# Install and run LLMs
curl -fsSL https://ollama.ai/install.sh | sh
ollama pull mistral
ollama pull llama3
```

## 📂 Project Structure

### Frontend

```
src/
├── components/        # UI Components
├── context/           # Global contexts (Theme, Language, Form)
├── utils/             # Helper functions
└── App.tsx            # Main App logic
```

### Backend

```
backend/
├── main.py             # FastAPI entry point
├── routers/            # API endpoints
├── services/           # Business logic (PDF, AI, translation)
├── models/             # Pydantic schemas
└── utils/              # Configs and helpers
```

## 🌐 API Overview (Coming Soon)

```
POST /api/upload       - Upload government form (PDF)
GET  /api/extract      - Extract data from form
POST /api/chat         - LLM-based guidance
POST /api/translate    - Translate between Hindi/English
POST /api/generate     - Generate filled PDF
GET  /api/health       - Health check
```

## 🌍 Use Cases

* Citizens struggling with form complexity
* NRIs or those unfamiliar with regional languages
* Elderly requiring guided form completion
* NGOs and social workers assisting communities
* Panchayat officials digitizing workflows

## 💼 Supported Forms

* Aadhaar Enrolment & Update
* RTI Applications
* PAN & Passport Applications
* Income, Caste & Domicile Certificates

## 🔒 Privacy & Security

* All processing is local — no internet required after setup
* No data leaves the device
* No third-party analytics or cloud APIs

## 🚜 Deployment

### Frontend (Netlify/Vercel)

```bash
npm run build
# Deploy the `dist/` directory
```

### Backend (Docker)

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 🌈 Vision

Empower every Indian citizen to complete government paperwork with ease, confidence, and privacy using AI.

## 🚀 Future Plans

* OCR for scanned form support
* Voice input and screen reading
* Regional language support
* Blockchain-based form authentication
* Direct integration with government portals

## 📅 License

MIT License — Open-source and community driven

## 🤝 Contributing

Contributions are welcome! Please fork the repo, open pull requests, and check out the contributing guide.

## 🛌 Support

* GitHub: [Issues](https://github.com/your-repo/issues)
* Email: [support@draftmitra.in](mailto:support@draftmitra.in)
* Wiki: [Project Docs](https://github.com/your-repo/wiki)
