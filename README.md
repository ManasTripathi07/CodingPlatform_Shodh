# 🧠 Shodh-a-Code — Real-Time Coding Contest Platform

**Shodh-a-Code** is a full-stack, end-to-end **online coding contest platform** built as a prototype for *Shodh AI’s Coding Challenge System*.  
It enables users to join coding contests, solve programming problems, submit code for live evaluation, and view results instantly on a real-time leaderboard.

---

## 📘 Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Tech Stack](#tech-stack)
4. [Folder Structure](#folder-structure)
5. [Setup Instructions](#setup-instructions)
6. [API Design](#api-design)
7. [Design Choices & Justifications](#design-choices--justifications)
8. [Judge Workflow](#judge-workflow)
9. [Troubleshooting](#troubleshooting)
10. [Future Enhancements](#future-enhancements)
11. [Author](#author)
12. [License](#license)

---

## 🧩 Project Overview

**Objective:**  
To design and implement a complete **live coding contest system** capable of:
- Accepting user-submitted source code securely
- Running it inside isolated Docker containers
- Comparing results with problem test cases
- Displaying live updates on a leaderboard

This project demonstrates **end-to-end full-stack development, systems integration, and DevOps orchestration**.

---

## 🏗️ Architecture Overview

```
Frontend (Next.js + Tailwind)
        │
        │ REST API Calls (polling every 2–3s)
        ▼
Backend (Spring Boot)
 ├─ REST API Layer
 ├─ Service Layer (ContestService, SubmissionService)
 ├─ Judge Engine (Docker Orchestration via ProcessBuilder)
 ├─ Database (Contest, Problem, Submission, User)
 └─ Leaderboard Service
        │
        ▼
Docker Runner (Java Execution Environment)
```

---

## ⚙️ Tech Stack

| Layer | Technology | Purpose |
|-------|-------------|----------|
| **Frontend** | Next.js (React 18), Tailwind CSS | UI for joining contests, coding, and live updates |
| **Backend** | Spring Boot, Java 17, Maven | REST API & business logic |
| **Database** | PostgreSQL / H2 | Data persistence |
| **Containerization** | Docker, Docker Compose | Code execution sandbox |
| **Language Supported** | Java (default) | Executed in runner container |
| **Version Control** | Git + GitHub | Source management |

---

## 📁 Folder Structure

```
CodingPlatform_Shodh/
├── backend/                  # Spring Boot backend
│   ├── src/main/java/...     # Controllers, Services, Entities, Repos
│   ├── src/main/resources/   # application.properties + seeds
│   ├── Dockerfile            # Backend image builder
│   └── pom.xml
│
├── frontend/                 # Next.js + Tailwind frontend
│   ├── pages/                # Join & Contest pages
│   ├── components/           # UI components
│   ├── package.json
│   └── Dockerfile
│
├── docker/runner/            # Docker runner for executing code
│   └── Dockerfile
│
├── docker-compose.yml        # Orchestration config
└── README.md
```

---

## 🛠️ Setup Instructions

### 1️⃣ Clone Repository
```bash
git clone https://github.com/ManasTripathi07/CodingPlatform_Shodh.git
cd CodingPlatform_Shodh
```

### 2️⃣ Build Runner Image
```bash
docker build -t shodh-runner-java:latest docker/runner
```

### 3️⃣ Build Backend JAR
```bash
cd backend
mvn -DskipTests package
cd ..
```

### 4️⃣ Start Entire Stack
```bash
docker-compose up --build
```

### 5️⃣ Access
- **Frontend:** http://localhost:3000  
- **Backend:** http://localhost:8080  

---

## 🧠 API Design

### 1️⃣ Get Contest Details
**Endpoint:**  
`GET /api/contests/{contestId}`  

**Response Example:**
```json
{
  "id": 1,
  "title": "Shodh Coding Challenge",
  "problems": [
    { "id": 1, "title": "Sum Two Numbers", "description": "Add two integers." },
    { "id": 2, "title": "Echo", "description": "Print input as output." }
  ]
}
```

### 2️⃣ Submit Code
**Endpoint:**  
`POST /api/submissions`

**Request Body:**
```json
{
  "contestId": 1,
  "problemId": 1,
  "username": "manas",
  "language": "java",
  "sourceCode": "public class Main { public static void main(String[] args){ System.out.println(2+3); } }"
}
```

**Response:**
```json
{ "submissionId": 101, "status": "PENDING" }
```

### 3️⃣ Get Submission Status
**Endpoint:**  
`GET /api/submissions/{submissionId}`

**Response Example:**
```json
{
  "submissionId": 101,
  "status": "ACCEPTED",
  "runtime": "0.89s",
  "memory": "32MB"
}
```

### 4️⃣ Get Leaderboard
**Endpoint:**  
`GET /api/contests/{contestId}/leaderboard`

**Response Example:**
```json
[
  { "rank": 1, "username": "manas", "score": 200, "accepted": 2 },
  { "rank": 2, "username": "alok", "score": 100, "accepted": 1 }
]
```

---

## 🧩 Design Choices & Justifications

### Backend Architecture
- **Layered design:** Controllers → Services → Repositories
- **Judge Engine:** Uses `ProcessBuilder` to execute Docker containers securely
- **Database:** Tables normalized for easy leaderboard aggregation

### Frontend Architecture
- **Framework:** Next.js + Tailwind CSS for modular UI
- **State Management:** Local React state + polling instead of Redux for simplicity
- **Real-Time:** Polling endpoints every few seconds for live updates

### Docker Orchestration
- Each code submission runs inside an isolated container:
  ```bash
  docker run --rm --network none --memory 256m --cpus 0.5 ...
  ```
- **Trade-offs:** simple and fast, but shares host Docker socket in development

### DevOps
- Single `docker-compose.yml` brings up everything with one command
- H2 DB for local use, PostgreSQL for production

---

## 🧮 Judge Workflow

1. User submits code
2. Backend saves it as `PENDING`
3. Code runs inside Docker container
4. Output is compared against expected
5. Status updated & leaderboard refreshed

---

## 🧰 Troubleshooting

| Issue | Cause | Fix |
|--------|--------|-----|
| `404 /api/...` | Frontend hitting wrong port | Update `API_BASE` or use rewrite |
| `ERR_CONNECTION_REFUSED` | Backend not running | Start backend or rebuild JAR |
| `File >100MB push error` | node_modules in Git | Add `.gitignore`, clean with BFG |
| `CORS error` | Missing config | Add `CorsConfig.java` |

---

## 🧾 Future Enhancements

- Multi-language support (Python, C++)
- WebSocket-based real-time updates
- User auth & admin dashboard
- Docker pool optimization
- Cloud deployment (AWS ECS / Kubernetes)

---

## 👨‍💻 Author

**Manas Tripathi**  
Full Stack & AI Engineer  
📧 manastripathi.contact@gmail.com  
🔗 [GitHub: @ManasTripathi07](https://github.com/ManasTripathi07)

---

## 📜 License

Licensed under the **MIT License**.
