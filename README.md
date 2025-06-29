# ClientConnect App – Fullstack Deployment Guide

ClientConnect is a full-featured CRM-style web application where users can manage their clients, assign projects, track payments, write notes, and view key business metrics in a centralized dashboard. It’s built with a modern tech stack using FastAPI and React, and is fully containerized for smooth deployment to Render.

![image](https://github.com/user-attachments/assets/141c90cd-ff20-44d6-b7ac-7b71325f8174)
![image](https://github.com/user-attachments/assets/9f8ae494-ac9d-4f25-9126-5c9fc03a3a3d)

## ✅ App Features Summary

* ✅ JWT-based authentication system
* ✅ CRUD operations for users, clients, projects, payments, and notes
* ✅ Dashboard with summaries
* ✅ Fully responsive React UI
* ✅ Dockerized backend
* ✅ Deployed on Render with static frontend serving

---

## 🎯 Minimum Viable Product (MVP)

* User registration and login with JWT tokens
* Users can manage clients and assign projects
* Add and track payments related to projects
* Notes feature for client/project communication
* Admin dashboard with key statistics and data insights

---

## 🧰 Tech Stack

### Backend

* FastAPI (Python 3.11)
* PostgreSQL (via SQLAlchemy/asyncpg)
* Alembic (migrations)
* Docker

### Frontend

* React
* Vite
* Axios
* Tailwind CSS (optional)

### DevOps / Deployment

* Docker
* Render.com
* GitHub (for CI/CD)

---

## 🏗 Project Structure

```
clientconnect/
├── frexta-backend/
│   ├── app/
│   │   └── ... FastAPI backend files
│   ├── static/             # <-- Vite build output goes here
│   ├── Dockerfile
│   └── requirements.txt
├── front/
│   ├── public/
│   ├── src/
│   ├── .env.production     # <-- API URL config for deployment
│   └── vite.config.js
└── README.md (this file)
```

---

## ⚙️ Backend Setup – FastAPI (frexta-backend)

### 1. Install dependencies

```bash
cd frexta-backend
pip install -r requirements.txt
```

### 2. Set up the database

Ensure `DATABASE_URL` and other environment variables are set in `.env` or via Render dashboard.

### 3. Run locally

```bash
uvicorn app.main:app --reload
```

### 4. Static frontend mounting in `app/main.py`

```python
from fastapi.staticfiles import StaticFiles
from fastapi.responses import FileResponse

app.mount("/static", StaticFiles(directory="static"), name="static")

@app.get("/", include_in_schema=False)
async def serve_frontend():
    return FileResponse("static/index.html")
```

---

## 💻 Frontend Setup – React + Vite (front)

### 1. Install and configure

```bash
cd front
npm install
```

### 2. API environment variable

Create `.env.production` in `front/`:

```env
VITE_API_URL=https://frexta-backend.onrender.com/api
```

### 3. Use API URL in Axios

Example:

```js
axios.post(`${import.meta.env.VITE_API_URL}/register`, data)
```

### 4. Build frontend for production

```bash
npm run build
```

### 5. Copy frontend build to backend static folder

```bash
cp -r front/dist/* frexta-backend/static/
```

---

## 🐳 Dockerfile (in frexta-backend)

```Dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y gcc && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV PORT=8000
EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

> ⚠️ Note: Avoid using `$PORT` in CMD. Render will still map external ports correctly.

---

## 🚀 Deployment on Render

### 1. Create a new Web Service

* Environment: **Docker**
* Region: closest to users
* Root directory: `frexta-backend/`
* Environment Variables:

  * `DATABASE_URL=...`
  * `SECRET_KEY=...`
  * `ALGORITHM=HS256`
  * `ACCESS_TOKEN_EXPIRE_MINUTES=30`

### 2. GitHub deployment

Push your project to GitHub.
Render will automatically build and deploy your backend.

Your live URL will be:

```
https://frexta-backend.onrender.com
```

This URL also serves your static frontend files.

---

## 🧪 Local Testing

### Frontend:

```bash
cd front
npm run dev
```

### Backend:

```bash
cd frexta-backend
uvicorn app.main:app --reload
```

Ensure the frontend `.env` file points to `http://127.0.0.1:8000/api` for local development.

---

## 🐞 Troubleshooting

* **Network Error / 127.0.0.1 API requests in production:**

  * Solution: Set `VITE_API_URL` to correct deployed URL in `.env.production`.

* **`static/index.html` not found:**

  * Solution: Run Vite build and copy `dist/` contents to `static/` folder inside `frexta-backend/`.

* **Docker/uvicorn `$PORT` error:**

  * Solution: Use a hardcoded port like `8000` instead of `$PORT` in `Dockerfile` CMD.

---

## 📝 License

This project is licensed under the MIT License.
