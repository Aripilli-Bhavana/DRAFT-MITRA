# DraftMitra - AI-Powered Government Form Assistant

## 🎯 Overview
DraftMitra is a full-stack AI-powered web application designed to simplify the process of filling Indian government forms. Built for the "Generative AI & LLMs for Governance" hackathon theme, it uses local LLMs to provide step-by-step guidance while ensuring data privacy.

## 🌟 Key Features
- **Smart PDF Analysis**: Upload government forms and get AI-powered analysis
- **Step-by-Step Guidance**: Interactive AI assistant guides users through each form field
- **Multilingual Support**: Hindi and English support with real-time translation
- **Local LLM Integration**: Uses Ollama for offline processing (Mistral/LLaMA 3)
- **Privacy-First**: All processing happens locally, no data leaves your device
- **Responsive Design**: Modern, accessible UI optimized for all devices
- **Dark/Light Mode**: Full theme support with user preference persistence

## 🛠️ Tech Stack

### Frontend
- **React 18** with TypeScript
- **Tailwind CSS** for styling
- **Lucide React** for icons
- **Context API** for state management

### Backend (Ready for Integration)
- **FastAPI** for REST API
- **Ollama** for local LLM integration
- **pdfplumber** for PDF text extraction
- **ReportLab** for PDF generation
- **Python** with async/await support

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ and npm
- Python 3.8+ (for backend)
- Ollama installed locally

### Frontend Setup
```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

### Backend Setup (Integration Ready)
```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Pull required models
ollama pull mistral
ollama pull llama3

# Install Python dependencies
pip install fastapi uvicorn pdfplumber reportlab python-multipart

# Start backend server
uvicorn main:app --reload --port 8000
```

## 📁 Project Structure

```
src/
├── components/           # React components
│   ├── Header.tsx       # Navigation and controls
│   ├── FileUploader.tsx # PDF upload interface
│   └── FormAssistant.tsx # AI chat interface
├── context/             # React contexts
│   ├── LanguageContext.tsx # i18n support
│   ├── ThemeContext.tsx    # Dark/light mode
│   └── FormContext.tsx     # Form state management
├── utils/               # Utility functions
│   ├── pdfUtils.ts     # PDF processing
│   └── aiUtils.ts      # AI/LLM integration
└── App.tsx             # Main application component
```

## 🔧 Backend Structure (Ready for Integration)

```
backend/
├── main.py              # FastAPI application
├── routers/
│   ├── upload.py       # File upload endpoints
│   ├── ai.py          # AI processing endpoints
│   └── forms.py       # Form management
├── services/
│   ├── pdf_service.py  # PDF extraction
│   ├── ai_service.py   # Ollama integration
│   └── translation.py # Language processing
├── models/
│   └── schemas.py      # Pydantic models
└── utils/
    ├── config.py       # Configuration
    └── helpers.py      # Utility functions
```

## 🌐 API Endpoints (Ready for Implementation)

```
POST /api/upload         # Upload PDF form
GET  /api/extract        # Extract form data
POST /api/chat          # AI assistant chat
POST /api/translate     # Translate text
POST /api/generate      # Generate filled PDF
GET  /api/health        # Health check
```

## 🎨 Design Features

- **Indian Government Theme**: Saffron, white, and green color palette
- **Accessibility**: WCAG 2.1 AA compliant with proper contrast ratios
- **Responsive**: Mobile-first design with breakpoints for all devices
- **Micro-interactions**: Smooth animations and loading states
- **Typography**: Optimized for readability in multiple languages

## 🔒 Privacy & Security

- **Local Processing**: All AI processing happens on your device
- **No Data Collection**: Forms and personal data never leave your computer
- **Offline Capable**: Works without internet connection (after initial setup)
- **Open Source**: Transparent and auditable codebase

## 🌍 Supported Languages

- **English**: Full interface and form support
- **Hindi**: Native support with Devanagari script
- **Extensible**: Architecture ready for additional Indian languages

## 📱 Supported Forms

Currently optimized for common Indian government forms:
- RTI (Right to Information) applications
- Income certificates
- Caste certificates
- Aadhar update forms
- PAN applications
- Passport applications

## 🎯 Target Users

- **Citizens**: Confused by complex government forms
- **NRIs**: Unfamiliar with Hindi or local processes
- **Elderly**: Need step-by-step guidance
- **Students**: Filing applications independently
- **NGOs**: Helping community members
- **Panchayat Staff**: Assisting villagers

## 🏆 Hackathon Benefits

- **Reduces Corruption**: Eliminates middlemen who charge for form filling
- **Increases Transparency**: Clear, step-by-step process
- **Improves Accessibility**: Supports multiple languages and disabilities
- **Enhances Digital India**: Promotes self-reliance in digital governance
- **Privacy-First**: Local processing ensures data security

## 🚀 Deployment

### Frontend (Netlify/Vercel)
```bash
npm run build
# Deploy dist/ folder to your hosting provider
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

## 🔮 Future Enhancements

- **OCR Integration**: Extract text from scanned documents
- **Voice Input**: Support for voice-based form filling
- **Blockchain**: Secure form submission and verification
- **Regional Languages**: Support for Tamil, Telugu, Bengali, etc.
- **Government Integration**: Direct submission to government portals

## 📄 License

MIT License - Open source and free for all users

## 🤝 Contributing

Contributions welcome! Please read our contributing guidelines and submit pull requests.

## 📞 Support

For issues and questions:
- GitHub Issues: [Create an issue](https://github.com/your-repo/issues)
- Email: support@draftmitra.in
- Documentation: [Wiki](https://github.com/your-repo/wiki)

---

Built with ❤️ for Digital India 🇮🇳