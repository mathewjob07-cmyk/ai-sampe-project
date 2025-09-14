# Intake Analyzer (Django)

A minimal Django app with a single-page UI to upload a PDF intake form. On upload the server:

- Extracts text from the PDF (native text for searchable PDFs or OCR for scanned PDFs via Tesseract)
- Sends the extracted text to Google Gemini for analysis
- Returns compact JSON with: `summary`, `risk_assessment`, `fraud_detection` (flag + reason), and `next_steps`

## Quickstart

1. Create and activate the virtual environment (already set up):
   ```powershell
   .venv\Scripts\activate
   ```

2. Install dependencies (already installed in this workspace):
   ```powershell
   pip install -r requirements.txt
   ```

3. Configure environment variables:
   - Copy `.env.example` to `.env` and set `GEMINI_API_KEY`:
     ```ini
     GEMINI_API_KEY=your_api_key_here
     # Optional: choose a specific model
     GEMINI_MODEL=gemini-2.5-flash
     # Optional (Windows): set if Tesseract is not on PATH
     # TESSERACT_PATH=C:\\Program Files\\Tesseract-OCR\\tesseract.exe
     ```

4. Ensure Tesseract OCR is installed (for scanned PDFs):
   - Download from: https://github.com/UB-Mannheim/tesseract/wiki
   - If it isn't on PATH, set `TESSERACT_PATH` to the `tesseract.exe` path as above.

5. Run database migrations and start the dev server:
   ```powershell
   python manage.py migrate
   python manage.py runserver
   ```

6. Open http://127.0.0.1:8000/ and upload a PDF.

## How it works

- UI: `analyzer/templates/analyzer/index.html` – simple page with an upload control and results area.
- Routes: `analyzer/urls.py` included at root in `intake_analyzer/urls.py`.
- Views: `analyzer/views.py`
  - `index`: renders the single-page UI
  - `analyze_upload`: POST endpoint `/api/analyze` to accept PDF and return JSON
- PDF Extraction: `analyzer/utils.py`
  - Uses PyMuPDF to get native text first
  - Falls back to OCR (Tesseract) if native text is minimal
- Gemini integration: `analyzer/utils.py`
  - Calls Gemini Flash model (prefers `gemini-2.5-flash` if available) and normalizes response to the required JSON schema

## API

- `POST /api/analyze` with `multipart/form-data` and field `file` = PDF file
- Response (example):
  ```json
  {
    "summary": "Short summary...",
    "risk_assessment": "Medium risk due to ...",
    "fraud_detection": { "flag": false, "reason": "No indicators" },
    "next_steps": ["Verify identity", "Schedule call"]
  }
  ```

## Notes

- CSRF is disabled on the upload endpoint with `@csrf_exempt` for simplicity in this demo.
- Large or image-only PDFs will take longer due to OCR.
- If Gemini returns non-JSON, a compact fallback structure is returned with the raw model text truncated in `summary` and a note in `fraud_detection.reason`.
