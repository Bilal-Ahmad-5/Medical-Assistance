# Medical Healthcare Chatbot

A production-oriented **Healthcare Chatbot** that combines conversational AI with a lightweight clinical database and a Retrieval-Augmented Generation (RAG) knowledge base. The system is designed to assist patients and staff with appointment management, doctor discovery, and authoritative information about *Medical City Hospital*.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Key Features](#key-features)
3. [Architecture](#architecture)
4. [Technology Stack](#technology-stack)
5. [Requirements](#requirements)
6. [Installation](#installation)
7. [Configuration](#configuration)
8. [Running the Chatbot](#running-the-chatbot)
9. [Usage Examples](#usage-examples)
10. [Data Model (SQLite)](#data-model-sqlite)
11. [RAG Knowledge Base](#rag-knowledge-base)
12. [Security & Privacy](#security--privacy)
13. [Testing](#testing)
14. [Deployment](#deployment)
15. [Contributing](#contributing)
16. [License & Contact](#license--contact)

---

## Project Overview

This repository provides a conversational assistant for a hospital setting. It integrates a lightweight relational database (SQLite) for transactional operations (appointments, patients, doctors) with a RAG pipeline to surface institutional knowledge (policies, services, FAQs). The assistant exposes a Gradio-powered UI for interactive conversations and administrative workflows.

The system is intended as a reference implementation and rapid prototype. It is organized for clarity, maintainability, and extensibility so teams can adapt it for production environments.

---

## Key Features

- **Appointment Management**
  - Search, book, update, and cancel appointments.
  - Validation for patient/doctor existence, scheduling conflicts, and business hours.
- **Doctor Discovery**
  - Query doctors by name, specialization, and availability for a given date.
- **Hospital Information**
  - Authoritative answers about services, departments, insurance & billing, visiting hours, emergency procedures, and patient programs.
- **RAG Knowledge Base**
  - Embeddings-based retrieval (HuggingFace) + FAISS vector store to augment model responses with hospital-specific documents.
- **Conversational UI**
  - Gradio chat interface for end-user interaction and demoing the assistant.

---

## Architecture

```
[User (Browser)]
      |
   Gradio UI
      |
   Chat Controller (LangChain + LangGraph)
      |                  ↘
  SQLite (patients/doctors/appointments)   RAG Retrieval (HF embeddings + FAISS)
      |                                         |
  Business Logic / Validators               Vector Store (local .idx/.db files)
      |
  Google Gemini (LLM) — API-backed generation
```

Notes:
- LangChain orchestrates calling the LLM and retrieving relevant documents from the vector store.
- LangGraph is used to model stateful, multi-step interactions (e.g., appointment workflows) when needed.

---

## Technology Stack
- **Python 3.8+**
- **LangChain** — LLM application framework
- **LangGraph** — multi-actor/stateful orchestration
- **Gradio** — web UI/prototyping
- **SQLite** — relational data store for demo/test data
- **HuggingFace Embeddings** — text vectorization
- **FAISS** — vector similarity search
- **Google Gemini** — primary LLM (via API)

---

## Requirements
- Python 3.8 or newer
- `pip` (recommended to use within a virtualenv)
- Google Gemini API key (for LLM access)

---

## Installation

```bash
# clone the repo
git clone https://github.com/your-org/medical-healthcare-chatbot.git
cd medical-healthcare-chatbot

# create virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate   # macOS / Linux
.\\.venv\\Scripts\\activate  # Windows (PowerShell)

# install dependencies
pip install -r requirements.txt
```

If you do not have a `requirements.txt`, install the core packages manually:

```bash
pip install langchain langgraph gradio faiss-cpu transformers sentence-transformers sqlite3
```

> For GPU-backed FAISS or other optional packages, see `requirements.txt` and adapt as needed.

---

## Configuration

Create a file named `.env` (recommended) or set environment variables directly. Example `.env`:

```
GOOGLE_API_KEY=your_google_gemini_api_key_here
SQLITE_DB_PATH=./data/medical_city.db
FAISS_INDEX_PATH=./data/faiss_index
HF_EMBEDDINGS_MODEL=sentence-transformers/all-MiniLM-L6-v2
GRADIO_SHARE=False
```

**Google Colab notes**
- If using Google Colab, store your Gemini key in the Colab secrets (`GOOGLE_API_KEY`) and enable secret access for the notebook.

**Important**: Never commit your API keys or `.env` to source control.

---

## Running the Chatbot

There are two primary ways to run the project:

### 1) From the provided Jupyter / Colab notebook
1. Open `MedicalAssistant.ipynb` in Colab or Jupyter.
2. Ensure the `.env` / secrets are available to the runtime.
3. Execute cells in order — this initializes the SQLite DB, seeds sample data, builds the RAG index, and launches the Gradio interface.

### 2) Run as a local Python app (example)

```bash
# Run the app (if you have a runnable script, e.g., app.py)
python app.py
# or, if the notebook exposes a launch cell, run that cell.
```

When the Gradio interface launches, it will print a local URL (e.g., `http://127.0.0.1:7860`) and — if `GRADIO_SHARE=True` — a public URL. Use these to access the chat UI.

---

## Usage Examples

- **Book an appointment**: "Book an appointment with Dr. Ayesha Khan, cardiology, on 2025-09-20 at 15:30 for a consultation."  
- **Find a doctor**: "Show me available neurologists on 2025-09-18."  
- **Hospital info**: "What are the visiting hours for the pediatric ward?"

Responses will include verification steps (appointment conflicts, patient existence) when applicable.

---

## Data Model (SQLite)

A minimal schema used by the demo:

- `patients` (id INTEGER PRIMARY KEY, name TEXT, dob DATE, phone TEXT, email TEXT)
- `doctors` (id INTEGER PRIMARY KEY, name TEXT, specialty TEXT, contact TEXT, availability JSON)
- `appointments` (id INTEGER PRIMARY KEY, patient_id INTEGER, doctor_id INTEGER, date DATE, time TIME, status TEXT, reason TEXT, notes TEXT)
- `services` (id INTEGER PRIMARY KEY, name TEXT, description TEXT, department TEXT)

> The provided notebook includes a DB initialization cell that creates and seeds these tables with sample data.

---

## RAG Knowledge Base

- Documents are vectorized using the HuggingFace embeddings model configured in `HF_EMBEDDINGS_MODEL`.
- FAISS is used to store and query vectors; retrieved documents are passed to the LLM prompt to ground responses.
- The notebook includes helper utilities to (re)build the vector store when knowledge base content changes.

Best practices:
- Keep knowledge base documents short and focused.
- Version the source documents for traceability.
- Periodically re-embed documents after content updates.

---

## Security & Privacy

- **Authentication**: This prototype does not include production authentication. Do not expose the app publicly without adding authentication and access controls.
- **PII Handling**: The system stores patient-identifiable information in a local SQLite file — treat this as sensitive. Use encrypted storage or a proper database for production.
- **Secrets**: Store API keys in environment variables or secret management systems; never commit them to Git.
- **Audit Logging**: Add logging for appointment creation, updates, and cancellations for audit and troubleshooting.

---

## Testing

- Unit test the appointment business logic (conflict detection, validation).
- Integration test the RAG retrieval and prompt pipeline to verify retrieved documents are relevant.
- End-to-end test the UI flows in Gradio and verify behaviour with realistic sample data.

---

## Deployment

This repository is a prototype. For productionization consider:
- Replacing SQLite with a managed database (Postgres, MySQL, etc.).
- Deploying the model-inference layer behind an authenticated API gateway.
- Hosting vector indices on a managed vector DB (e.g., Pinecone, Milvus) for scale and persistence.
- Adding role-based access control and secure storage for PII.

---

## Contributing

Contributions are welcome. Please follow these steps:
1. Fork the repository.
2. Create a feature branch: `git checkout -b feat/your-feature`.
3. Write tests for new functionality.
4. Open a pull request with a clear description of changes.

---

## License & Contact

This project is provided under the `MIT` license — see `LICENSE` for details.

For questions, contact the maintainer: **Bilal Bhatti** (or the project team).

---

*Prepared for Medical City Hospital — reference implementation for conversational healthcare assistants.*
