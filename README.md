# CRUD-OPERATION-CHATBOT
# User Management AI Chatbot

A full-stack user management application with an AI-powered conversational chatbot built using **FastAPI**, **SQLite**, **Google Gemini AI**, **React 19**, **Vite**, and **Tailwind CSS**.

---

## 🌟 Key Features

- **Modern Dark-Themed UI:** Built with React 19, Tailwind CSS, and Lucide Icons.
- **Bilingual AI Chatbot:** Supports queries in **English**, **Roman Urdu** (e.g. *"afaq ko add kr do"*), and mixed language queries.
- **Natural Language Database Control:** Perform CRUD actions (Create, Read, Update, Delete, Count) via natural language.
- **Multi-Turn Conversational Flows:** Smart parameter collection that asks clarifying questions if required fields (name, email, phone, address) are missing.
- **Flexible User Lookup:** Find users by **ID**, **email** (exact match), or **name** (fuzzy match with typo tolerance).
- **Email-Only Auto-Login:** No password required — login with any registered email or admin email.
- **Dashboard with Edit & Delete:** Visual data table with inline Edit (pencil icon) and Delete (trash icon) actions via modals.
- **Persistent Chat Session:** Chat messages and session state are preserved across page navigation.
- **JWT Authentication:** Secure access-token-based authentication and protected routes.
- **Railway Deployment Ready:** Runs on Railway via `uvicorn` with production-built React frontend.

---

## 🏗 System Architecture

```
                 +-----------------------------------+
                 | React 19 Frontend (Vite + Tailwind)|
                 +-----------------+-----------------+
                                   |
                                   | HTTP / JSON API (JWT)
                                   v
                 +-----------------+-----------------+
                 |   FastAPI Backend Server (:8001)  |
                 +--------+-----------------+--------+
                          |                 |
            Natural Query |                 | DB CRUD Operations
                          v                 v
               +----------+-----+    +------+------+
               | Google Gemini  |    | SQLite Database
               | LLM (Intent   |    | (user_management.db)
               |  Parsing)     |    +-------------+
               +----------------+
```

---

## 📂 Project Structure

```
user-management-chatbot/
├── .env                  # API Keys & JWT Secret Configuration
├── schema.sql            # SQLite Database Schema
├── requirements.txt      # Python Dependencies
├── README.md             # Project Overview & Setup Instructions
├── PRODUCT_DOCUMENTATION.md  # Detailed Architecture Documentation
├── PROJECT_FLOW.md       # Detailed Execution Flow
├── PROMPT.md             # System Prompts & LLM Engineering Documentation
├── app/                  # FastAPI Backend Package
│   ├── main.py           # FastAPI Entrypoint, CORS, Static File Serving
│   ├── config.py         # JWT, Gemini API, Embedding & FAISS Config
│   ├── models.py         # Pydantic Schemas (UserCreate, UserUpdate, ChatRequest)
│   ├── routes.py         # API Route Handlers (/login, /users, /chat)
│   ├── db.py             # SQLite CRUD, Fuzzy Name Match, Schema Migration
│   ├── vector_store.py   # Top-level FAISS Vector Store (startup init)
│   ├── chatbot/          # Chatbot Subsystem
│   │   ├── llm.py        # Gemini API Integration
│   │   ├── conversation.py # Thread-Safe Session State Manager
│   │   └── rag/          # RAG Retrieval & Vector Store (reserved for future)
│   │       ├── embeddings.py   # SentenceTransformer Embedding
│   │       ├── vector_store.py # FAISS Index Wrapper
│   │       └── retriever.py    # Top-k Similarity Search
│   └── services/         # Business Logic Layer
│       ├── auth.py       # JWT Token Creation & Verification
│       ├── chat_service.py # Intent Parsing, Multi-Turn State, Chat Handler
│       └── user_service.py # User CRUD Logic & Validation
├── data/
│   └── user_management.db # SQLite Database File
└── frontend/             # React 19 Frontend Package
    ├── vite.config.js    # Vite Server & Proxy Config
    ├── package.json      # Dependencies
    └── src/
        ├── App.jsx       # App Navigation & Protected Routes
        ├── main.jsx      # Entry Point with AuthProvider
        ├── context/
        │   └── AuthContext.jsx # JWT State, Email-Only Login, authFetch
        ├── components/
        │   ├── Navbar.jsx      # Header Bar with Navigation
        │   └── ProtectedRoute.jsx # Auth Guard
        └── pages/
            ├── LoginPage.jsx   # Email-Only Auto-Login Screen
            ├── DashboardPage.jsx # User Table with Edit/Delete & Add Modal
            └── ChatPage.jsx    # AI Chat Interface
```

---

## 🚀 Quick Start & Installation

### Prerequisites
- **Python:** 3.10+
- **Node.js:** 18+
- **Google Gemini API Key**

---

### Step 1: Environment Setup

Create or update `.env` in the project root:

```env
GOOGLE_GEMINI_API_KEY=your_gemini_api_key_here
JWT_SECRET=your_jwt_secret_key
```

---

### Step 2: Start Backend Server

```bash
# Navigate to project root
cd user-management-chatbot

# Install Python dependencies
pip install -r requirements.txt

# Start FastAPI Uvicorn server on port 8001
uvicorn app.main:app --reload --port 8001
```

*Backend API Documentation available at: [http://127.0.0.1:8001/docs](http://127.0.0.1:8001/docs)*

---

### Step 3: Start Frontend Server

```bash
# Navigate to frontend directory
cd frontend

# Install Node modules
npm install

# Run Vite dev server on port 5173
npm run dev -- --port 5173
```

*Frontend Application available at: [http://localhost:5173](http://localhost:5173)*

---

## 🔐 Authentication

This application uses **email-only auto-login** (no password required):

- **System Admin Emails (always authorized):** `admin@example.com`, `chatbot@gmail.com`
- **Registered Users:** Any email present in the `users` table is authorized to log in.
- **Unregistered Emails:** Returns error "Email not found. Please contact an administrator to be added first."

On successful login, a JWT access token is issued and stored in `localStorage`.

---

## 📡 API Endpoint Reference

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/login` | `POST` | Public | Email-only login. Returns JWT access token. |
| `/api/users` | `GET` | JWT | Fetch list of all users. |
| `/api/users` | `POST` | JWT | Create a new user record. |
| `/api/users/{id}` | `GET` | JWT | Fetch a specific user by ID. |
| `/api/users/{id}` | `PUT` | JWT | Update user details by ID. |
| `/api/users/{id}` | `DELETE` | JWT | Remove user record by ID. |
| `/api/users/count` | `GET` | JWT | Get total user count. |
| `/api/chat` | `POST` | JWT | Process natural language chat queries. |

---

## 🗄️ User Data Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | Optional (derived from email if omitted) | User's full name |
| `email` | `string` | Required | Unique email address |
| `phone` | `string` | Required | Phone number (7-15 digits, optional `+` prefix) |
| `address` | `string` | Required for standard creation | Physical address |

> **Note:** The deprecated `city` and `country` fields have been consolidated into a single `address` field.

---

## 🧪 Example Chat Queries

### English
- **Count Users:** `"count users"` or `"how many users"`
- **Read User (by ID):** `"get user 1"`
- **Read User (by name):** `"get afaq"` or `"show me afaq"` or `"who is afaq"`
- **Read User (by email):** `"get john.smith@xyz.com"`
- **Create User:** `"create user Bilal"` → bot asks for email, phone, address
- **Quick Create:** `"add the user john@xyz.com with phone +923001234567"` (address optional)
- **Update User:** `"update samantha's address to Cordoba"`
- **Delete User:** `"remove the user john.smith@xyz.com"` → bot asks for confirmation

### Roman Urdu
- **Count:** `"total users kitne hain"`
- **Read:** `"ali ka record dekhao"` or `"afaq ki details dikhao"` or `"afaq kaun hai"`
- **Create:** `"afaq naam ka user banao"`
- **Delete:** `"mohsin ko hatao"`

---

## 💻 Deployment

### Railway (Production)
- **Live URL:** `https://wp-chatbot-production-3246.up.railway.app`
- **Start Command:** `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
- **Frontend:** Pre-built React assets served from `frontend_dist/`

### XAMPP (Local Development)
- **Location:** `/Applications/xampp/htdocs/user-management-chatbot`
- **Backend:** `uvicorn app.main:app --reload --port 8001`
- **Frontend:** `npm run dev -- --port 5173` inside `frontend/`

---

## 🧬 RAG Module (Future Extension)

The RAG (Retrieval-Augmented Generation) infrastructure is present but not active in the main chat flow:
- `app/chatbot/rag/embeddings.py` — SentenceTransformer embeddings
- `app/chatbot/rag/vector_store.py` — FAISS index persistence
- `app/chatbot/rag/retriever.py` — Top-k similarity search

These files are preserved for future semantic search capabilities.

---

## 📦 Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI, Uvicorn, Python 3.10+ |
| Database | SQLite 3 |
| AI / LLM | Google Gemini (gemini-flash-lite-latest) |
| Frontend | React 19, Vite, Tailwind CSS, Lucide Icons |
| Auth | JWT (PyJWT) |
| Embeddings | SentenceTransformer (all-MiniLM-L6-v2) |
| Vector Store | FAISS |
| Deployment | Railway |
