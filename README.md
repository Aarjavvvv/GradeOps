# GradeOps — Setup Guide

## Project Structure
```
gradeops/
├── backend/
│   ├── main.py          ← FastAPI server (all routes)
│   └── requirements.txt
└── frontend/
    ├── src/
    │   ├── App.jsx              ← Root app + routing + auth context
    │   ├── main.jsx             ← React entry point
    │   ├── index.css            ← Global styles
    │   └── pages/
    │       ├── LoginPage.jsx    ← Login with role selection
    │       ├── DashboardPage.jsx← All exams overview
    │       ├── UploadPage.jsx   ← Professor upload portal
    │       ├── ExamDetailPage.jsx← All submissions for one exam
    │       └── ReviewPage.jsx   ← TA review dashboard (keyboard shortcuts)
    ├── index.html
    ├── package.json
    └── vite.config.js
```

---

## Step 1 — Backend Setup

```bash
cd gradeops/backend

# Create a virtual environment
python -m venv venv

# Activate it
# On Windows:
venv\Scripts\activate
# On Mac/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run the server
uvicorn main:app --reload --port 8000
```

Backend will run at: http://localhost:8000
API docs (auto-generated): http://localhost:8000/docs

---

## Step 2 — Frontend Setup

```bash
cd gradeops/frontend

# Install dependencies
npm install

# Start dev server
npm run dev
```

Frontend will run at: http://localhost:5173

---

## Step 3 — Test It

Open http://localhost:5173

**Demo login credentials:**
- Professor: `prof@iitg.ac.in` / `password123`
- TA: `ta@iitg.ac.in` / `password123`

**Workflow:**
1. Login as Professor → Upload Exam → attach PDF(s) + fill rubric → Submit
2. AI grades automatically (mock for now — see below to plug in real AI)
3. Login as TA → Dashboard → Click exam → Review each submission
4. Use keyboard shortcuts: `A` = Approve, `O` = Override, `D/→` = Next, `S/←` = Prev

---

## Step 4 — Plug In Your ML Pipeline

In `backend/main.py`, find the `mock_ai_grade()` function and replace it:

```python
def mock_ai_grade(rubric: list, student_id: str):
    # REPLACE THIS with your actual ML pipeline call
    # Your pipeline should:
    # 1. Take the rubric and the PDF path
    # 2. Run OCR (Nougat/Qwen-VL) to extract handwritten text
    # 3. Run LLM (via LangChain/LangGraph) to grade each question
    # 4. Return the same dict format shown below

    # Expected return format:
    return {
        "grades": [
            {
                "question_number": 1,
                "max_marks": 10.0,
                "awarded_marks": 7.5,
                "justification": "Correct method, minor arithmetic error.",
                "plagiarism_flag": False
            }
            # ... one entry per rubric item
        ],
        "total_score": 7.5,
        "max_score": 10.0,
        "percentage": 75.0
    }
```

---

## Step 5 — Add PDF Viewer (Optional but recommended)

In `ReviewPage.jsx`, find the PDF viewer section and replace the placeholder with:

```jsx
// Install: npm install react-pdf
import { Document, Page } from 'react-pdf'

<Document file={`${API}/files/${grade.file_path}`}>
  <Page pageNumber={1} />
</Document>
```

And add a static files route in `main.py`:
```python
app.mount("/files", StaticFiles(directory="uploads"), name="uploads")
```

---

## Database (Production)

The current setup uses in-memory Python dicts (data resets on server restart).
To persist data, replace with PostgreSQL:

```bash
pip install sqlalchemy psycopg2-binary alembic
```

Create a `database.py` with SQLAlchemy models — or ask your mentor for the schema.

---

## API Endpoints Summary

| Method | URL | Auth | Description |
|--------|-----|------|-------------|
| POST | /auth/login | None | Login → get JWT token |
| GET | /auth/me | Any | Get current user |
| POST | /exams/upload | Instructor | Upload PDFs + rubric |
| GET | /exams | Any | List all exams |
| GET | /exams/{id} | Any | Get one exam |
| GET | /grades/exam/{id} | Any | All grades for an exam |
| PATCH | /grades/{id} | TA/Instructor | Approve or override grade |
| GET | /grades/exam/{id}/stats | Any | Stats for an exam |
