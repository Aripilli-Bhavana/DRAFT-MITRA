# Backend Implementation Guide

This document provides the complete backend structure for DraftMitra. Due to environment limitations, the Python backend cannot be fully implemented here, but this provides the complete code structure for integration.

## 📁 Backend File Structure

```
backend/
├── main.py                 # FastAPI main application
├── requirements.txt        # Python dependencies
├── config.py              # Configuration settings
├── routers/               # API route handlers
│   ├── __init__.py
│   ├── upload.py          # File upload endpoints
│   ├── ai.py             # AI/LLM processing
│   ├── forms.py          # Form management
│   └── translate.py      # Translation services
├── services/              # Business logic services
│   ├── __init__.py
│   ├── pdf_service.py    # PDF processing
│   ├── ai_service.py     # Ollama integration
│   ├── translation_service.py
│   └── form_service.py
├── models/                # Data models
│   ├── __init__.py
│   ├── schemas.py        # Pydantic models
│   └── form_models.py
├── utils/                 # Utility functions
│   ├── __init__.py
│   ├── helpers.py
│   └── constants.py
└── tests/                 # Test files
    ├── __init__.py
    ├── test_api.py
    └── test_services.py
```

## 🔧 Core Files

### main.py
```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from routers import upload, ai, forms, translate
import uvicorn

app = FastAPI(
    title="DraftMitra API",
    description="AI-Powered Government Form Assistant",
    version="1.0.0"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(upload.router, prefix="/api", tags=["upload"])
app.include_router(ai.router, prefix="/api", tags=["ai"])
app.include_router(forms.router, prefix="/api", tags=["forms"])
app.include_router(translate.router, prefix="/api", tags=["translate"])

@app.get("/")
async def root():
    return {"message": "DraftMitra API is running"}

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "DraftMitra API"}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### requirements.txt
```txt
fastapi==0.104.1
uvicorn==0.24.0
python-multipart==0.0.6
pdfplumber==0.9.0
reportlab==4.0.7
requests==2.31.0
aiofiles==23.2.1
python-docx==0.8.11
Pillow==10.1.0
httpx==0.25.2
pydantic==2.5.0
python-jose==3.3.0
passlib==1.7.4
```

### routers/upload.py
```python
from fastapi import APIRouter, UploadFile, File, HTTPException
from services.pdf_service import PDFService
from services.ai_service import AIService
from models.schemas import FormExtractionResponse
import aiofiles
import os

router = APIRouter()
pdf_service = PDFService()
ai_service = AIService()

@router.post("/upload", response_model=FormExtractionResponse)
async def upload_form(file: UploadFile = File(...)):
    """Upload and process PDF form"""
    
    if not file.filename.endswith('.pdf'):
        raise HTTPException(status_code=400, detail="Only PDF files are allowed")
    
    if file.size > 10 * 1024 * 1024:  # 10MB limit
        raise HTTPException(status_code=400, detail="File size too large")
    
    try:
        # Save uploaded file temporarily
        temp_path = f"temp_{file.filename}"
        async with aiofiles.open(temp_path, 'wb') as f:
            content = await file.read()
            await f.write(content)
        
        # Extract text from PDF
        extracted_text = pdf_service.extract_text(temp_path)
        
        # Process with AI
        form_structure = await ai_service.analyze_form(extracted_text)
        
        # Clean up temp file
        os.remove(temp_path)
        
        return FormExtractionResponse(
            success=True,
            form_title=form_structure.get('title', 'Government Form'),
            fields=form_structure.get('fields', []),
            sections=form_structure.get('sections', []),
            language=form_structure.get('language', 'en'),
            total_fields=len(form_structure.get('fields', []))
        )
        
    except Exception as e:
        # Clean up on error
        if os.path.exists(temp_path):
            os.remove(temp_path)
        raise HTTPException(status_code=500, detail=f"Processing error: {str(e)}")
```

### services/ai_service.py
```python
import httpx
import json
from typing import Dict, List, Any
from config import OLLAMA_BASE_URL, OLLAMA_MODEL

class AIService:
    def __init__(self):
        self.base_url = OLLAMA_BASE_URL
        self.model = OLLAMA_MODEL
    
    async def analyze_form(self, pdf_text: str) -> Dict[str, Any]:
        """Analyze PDF text and extract form structure"""
        
        prompt = f"""
        Analyze this Indian government form text and extract the structure:
        
        {pdf_text}
        
        Return a JSON response with:
        - title: Form title
        - fields: Array of field objects with id, label, type, required
        - sections: Array of section objects
        - language: detected language (en/hi/mixed)
        
        Focus on identifying input fields, checkboxes, and required information.
        """
        
        try:
            async with httpx.AsyncClient() as client:
                response = await client.post(
                    f"{self.base_url}/api/generate",
                    json={
                        "model": self.model,
                        "prompt": prompt,
                        "stream": False,
                        "format": "json"
                    },
                    timeout=30.0
                )
                
                if response.status_code == 200:
                    result = response.json()
                    return json.loads(result['response'])
                else:
                    return self._fallback_form_structure()
                    
        except Exception as e:
            print(f"AI Service error: {e}")
            return self._fallback_form_structure()
    
    async def chat_response(self, user_message: str, context: str) -> str:
        """Generate AI response for user queries"""
        
        system_prompt = """You are a helpful assistant for Indian government forms. 
        Guide users step-by-step through filling forms. Be patient and clear.
        Respond in simple language and ask for one piece of information at a time."""
        
        prompt = f"""
        System: {system_prompt}
        
        Context: {context}
        
        User: {user_message}
        
        Assistant: """
        
        try:
            async with httpx.AsyncClient() as client:
                response = await client.post(
                    f"{self.base_url}/api/generate",
                    json={
                        "model": self.model,
                        "prompt": prompt,
                        "stream": False
                    },
                    timeout=20.0
                )
                
                if response.status_code == 200:
                    result = response.json()
                    return result['response']
                else:
                    return "I'm having trouble processing your request. Please try again."
                    
        except Exception as e:
            print(f"Chat error: {e}")
            return "I'm experiencing technical difficulties. Please try again."
    
    def _fallback_form_structure(self) -> Dict[str, Any]:
        """Fallback form structure when AI fails"""
        return {
            "title": "Government Application Form",
            "language": "mixed",
            "fields": [
                {"id": "name", "label": "Full Name", "type": "text", "required": True},
                {"id": "father_name", "label": "Father's Name", "type": "text", "required": True},
                {"id": "address", "label": "Address", "type": "text", "required": True},
                {"id": "phone", "label": "Phone Number", "type": "text", "required": True},
                {"id": "purpose", "label": "Purpose", "type": "text", "required": True}
            ],
            "sections": [
                {"id": "personal", "title": "Personal Information", "fields": ["name", "father_name", "address", "phone"]},
                {"id": "application", "title": "Application Details", "fields": ["purpose"]}
            ]
        }
```

### services/pdf_service.py
```python
import pdfplumber
from typing import List, Dict, Any
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter
import io

class PDFService:
    def extract_text(self, pdf_path: str) -> str:
        """Extract text from PDF file"""
        try:
            with pdfplumber.open(pdf_path) as pdf:
                text = ""
                for page in pdf.pages:
                    page_text = page.extract_text()
                    if page_text:
                        text += page_text + "\n"
                return text
        except Exception as e:
            raise Exception(f"Failed to extract PDF text: {str(e)}")
    
    def extract_fields(self, pdf_path: str) -> List[Dict[str, Any]]:
        """Extract form fields from PDF"""
        try:
            with pdfplumber.open(pdf_path) as pdf:
                fields = []
                for page in pdf.pages:
                    # Extract tables and forms
                    tables = page.extract_tables()
                    for table in tables:
                        for row in table:
                            if row and len(row) >= 2:
                                label = row[0] if row[0] else ""
                                if label and ":" in label:
                                    fields.append({
                                        "label": label.replace(":", "").strip(),
                                        "type": "text",
                                        "required": True
                                    })
                return fields
        except Exception as e:
            raise Exception(f"Failed to extract form fields: {str(e)}")
    
    def generate_filled_pdf(self, form_data: Dict[str, str], template_path: str = None) -> bytes:
        """Generate filled PDF from form data"""
        try:
            buffer = io.BytesIO()
            p = canvas.Canvas(buffer, pagesize=letter)
            
            # Title
            p.setFont("Helvetica-Bold", 16)
            p.drawString(100, 750, "Government Application Form")
            
            # Form fields
            y_position = 700
            p.setFont("Helvetica", 12)
            
            for field_id, value in form_data.items():
                label = self._get_field_label(field_id)
                p.drawString(100, y_position, f"{label}: {value}")
                y_position -= 30
            
            # Footer
            p.setFont("Helvetica-Oblique", 10)
            p.drawString(100, 100, "Generated by DraftMitra - AI-Powered Form Assistant")
            
            p.save()
            buffer.seek(0)
            return buffer.read()
            
        except Exception as e:
            raise Exception(f"Failed to generate PDF: {str(e)}")
    
    def _get_field_label(self, field_id: str) -> str:
        """Get human-readable label for field ID"""
        labels = {
            "name": "Full Name",
            "father_name": "Father's Name",
            "address": "Address",
            "phone": "Phone Number",
            "purpose": "Purpose of Application"
        }
        return labels.get(field_id, field_id.replace("_", " ").title())
```

### models/schemas.py
```python
from pydantic import BaseModel
from typing import List, Optional, Dict, Any

class FormField(BaseModel):
    id: str
    label: str
    type: str = "text"
    required: bool = True
    options: Optional[List[str]] = None
    placeholder: Optional[str] = None
    section: Optional[str] = None

class FormSection(BaseModel):
    id: str
    title: str
    description: Optional[str] = None
    fields: List[str]

class FormExtractionResponse(BaseModel):
    success: bool
    form_title: str
    fields: List[FormField]
    sections: List[FormSection]
    language: str
    total_fields: int

class ChatMessage(BaseModel):
    message: str
    context: Optional[str] = None
    form_data: Optional[Dict[str, Any]] = None

class ChatResponse(BaseModel):
    response: str
    next_field: Optional[str] = None
    suggestions: Optional[List[str]] = None

class TranslationRequest(BaseModel):
    text: str
    source_language: str = "auto"
    target_language: str = "en"

class TranslationResponse(BaseModel):
    translated_text: str
    source_language: str
    target_language: str

class PDFGenerationRequest(BaseModel):
    form_data: Dict[str, str]
    template_id: Optional[str] = None

class PDFGenerationResponse(BaseModel):
    success: bool
    download_url: str
    file_size: int
```

### config.py
```python
import os
from pathlib import Path

# Ollama Configuration
OLLAMA_BASE_URL = os.getenv("OLLAMA_BASE_URL", "http://localhost:11434")
OLLAMA_MODEL = os.getenv("OLLAMA_MODEL", "mistral")

# File Upload Settings
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB
ALLOWED_EXTENSIONS = [".pdf"]
UPLOAD_DIRECTORY = Path("uploads")
TEMP_DIRECTORY = Path("temp")

# Create directories
UPLOAD_DIRECTORY.mkdir(exist_ok=True)
TEMP_DIRECTORY.mkdir(exist_ok=True)

# API Settings
API_VERSION = "1.0.0"
API_TITLE = "DraftMitra API"
API_DESCRIPTION = "AI-Powered Government Form Assistant"

# Security Settings
SECRET_KEY = os.getenv("SECRET_KEY", "your-secret-key-here")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Translation Settings
SUPPORTED_LANGUAGES = ["en", "hi", "te", "ta", "bn"]
DEFAULT_LANGUAGE = "en"

# PDF Generation Settings
PDF_OUTPUT_DIRECTORY = Path("generated_pdfs")
PDF_OUTPUT_DIRECTORY.mkdir(exist_ok=True)
```

## 🚀 Setup Instructions

1. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Install Ollama**:
   ```bash
   curl -fsSL https://ollama.ai/install.sh | sh
   ollama pull mistral
   ollama pull llama3
   ```

3. **Run Backend**:
   ```bash
   uvicorn main:app --reload --port 8000
   ```

4. **Test API**:
   ```bash
   curl http://localhost:8000/health
   ```

The backend is now ready for integration with the frontend React application!