# 🧠 Shodh-a-Code — Real-Time Coding Contest Platform

**Shodh-a-Code** is a full-stack, end-to-end **online coding contest platform** built as a prototype for *Shodh AI’s Coding Challenge System*.  
It features live code submission, secure containerized execution (via Docker), and a real-time leaderboard — replicating how modern competitive programming platforms like LeetCode or Codeforces handle submissions at scale.

---

## 🚀 Project Overview

**Goal:**  
To create a fully functional prototype that assesses end-to-end software engineering capability — integrating **backend execution orchestration**, **frontend interactivity**, and **DevOps containerization** into one cohesive product.

**Core Idea:**  
Students or users can join a live contest using a Contest ID, solve programming problems, and view a dynamic leaderboard as the judge system evaluates their code in real time inside isolated Docker containers.

---

## 🏗️ Architecture Overview

Frontend (Next.js + Tailwind)
│
│ REST API Calls (polling every 2-3s)
▼
Backend (Spring Boot)
├─ REST API Layer
├─ Judge Service (Docker Orchestration via ProcessBuilder)
├─ Submission Queue Worker
├─ PostgreSQL / H2 DB (JPA Repositories)
└─ Leaderboard Computation Engine
│
▼
Docker Runner (Java Execution Environment)


---

## ⚙️ Tech Stack

| Layer | Technology | Purpose |
|-------|-------------|----------|
| **Frontend** | Next.js (React 18), Tailwind CSS | Dynamic contest UI, code editor, leaderboard |
| **Backend** | Spring Boot (Java 17), JPA, Maven | REST APIs, judge orchestration, data persistence |
| **Database** | PostgreSQL / H2 | Contest, problem, user, submission data |
| **Containerization** | Docker, Docker Compose | Sandbox execution and environment consistency |
| **Language Supported** | Java (default) | Executed inside `shodh-runner-java` container |
| **Dev Tools** | VS Code, Git, GitHub | Development and version control |

---

## 📁 Folder Structure



CodingPlatform_Shodh/
├── backend/ # Spring Boot backend
│ ├── src/main/java/... # Source code (entities, repos, services, controllers)
│ ├── src/main/resources/ # Properties + seed data
│ ├── Dockerfile # Backend image builder
│ └── pom.xml # Maven project config
│
├── frontend/ # Next.js frontend app
│ ├── pages/ # Routes (Join, Contest, etc.)
│ ├── styles/ # Tailwind styles
│ ├── package.json # NPM config
│ └── Dockerfile # Frontend image
│
├── docker/runner/ # Code runner container (Java)
│ └── Dockerfile
│
├── docker-compose.yml # Multi-service orchestration
└── README.md # This file


---

## 💡 Key Features

### 🎯 1. Contest Management
- `GET /api/contests/{id}` → Fetch contest details & problems  
- Supports pre-seeded sample contests (via `data.sql`).

### ⚙️ 2. Code Submission System
- `POST /api/submissions` → Accepts user code, queues it for judging  
- `GET /api/submissions/{id}` → Fetches live status (`Pending`, `Running`, `Accepted`, etc.)

### 🧪 3. Docker-Based Judge Engine
- Executes user code inside a custom `shodh-runner-java` container.  
- Each container is:
  - **Isolated:** Network disabled (`--network none`)
  - **Restricted:** Memory & CPU limits applied
  - **Clean:** Auto-removed after each run

### 📊 4. Live Leaderboard
- `GET /api/contests/{id}/leaderboard` → Aggregates submissions  
- Polls every 15–30s on the frontend to simulate real-time updates.

### ⚡ 5. Real-Time Frontend
- Polling-based update system for status and leaderboard.  
- Clean UI built with Tailwind.  
- Separate “Join” page and “Contest” workspace with:
  - Problem selector
  - Code editor (textarea / Monaco-ready)
  - Submission button
  - Live status + leaderboard sidebar.

---

## 🧩 API Endpoints Summary

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `GET` | `/api/contests/{contestId}` | Fetch contest & problem info |
| `POST` | `/api/submissions` | Submit code for a problem |
| `GET` | `/api/submissions/{submissionId}` | Check submission result |
| `GET` | `/api/contests/{contestId}/leaderboard` | Live leaderboard data |

---

## 🛠️ Setup Instructions

### 🔹 1. Clone Repository

git clone https://github.com/ManasTripathi07/CodingPlatform_Shodh.git
cd CodingPlatform_Shodh

🔹 2. Build the Runner Image
docker build -t shodh-runner-java:latest docker/runner

🔹 3. Build Backend JAR
cd backend
mvn -DskipTests package
cd ..

🔹 4. Start Everything with Docker Compose
docker-compose up --build


The following services will start:

🐘 PostgreSQL (port 5432)

⚙️ Backend (port 8080)

🌐 Frontend (port 3000)

Access the app at http://localhost:3000

🧠 How the Judge Works

User submits code → /api/submissions

Backend saves submission → sets status = PENDING

Worker thread picks it → runs inside Docker using:

docker run --rm --network none --memory 256m \
  -v /tmp/submissions/<id>:/home/src \
  shodh-runner-java:latest bash -lc "javac Main.java && timeout 2s java Main"


The stdout is compared to the expected output (from testcases).

Status updated to:

✅ ACCEPTED

❌ WRONG_ANSWER

⚙️ RUNTIME_ERROR

⏱️ TIME_LIMIT_EXCEEDED

Leaderboard recalculates automatically.

🖥️ Developer Shortcuts
Run Backend Locally (H2 Database)
cd backend
mvn spring-boot:run -Dspring-boot.run.arguments="--spring.datasource.url=jdbc:h2:mem:shodh"

Run Frontend Locally
cd frontend
npm install
npm run dev


Access at http://localhost:3000

🧩 Design Choices & Justifications
Decision	Rationale
Spring Boot + ProcessBuilder	Simple, reliable orchestration of Docker commands; easy async queue integration
H2 + PostgreSQL	H2 for local testing, Postgres for production
Next.js + Tailwind	Modern UI stack with automatic routing and reactive updates
Polling vs WebSockets	Simpler implementation for prototype; stable across browsers
Docker Runner	Prevents untrusted code from affecting host; resource-limited & isolated
Single docker-compose file	One-command environment setup for evaluators
🔒 Security Considerations

Containers run without network access (--network none).

Strict resource caps (--memory 256m, --cpus 0.5).

Temporary directories deleted post-execution.

Docker socket access limited to internal development; not for production.

Future hardening: integrate gVisor / Firecracker sandboxes.

🧾 Sample Test Contest (Pre-Seeded)
Contest ID	Problems	Example
1	A - Sum Two Numbers, B - Echo	Run A with inputs 2 3 → 5, 10 25 → 35

To test:

curl -X POST http://localhost:8080/api/submissions \
  -H "Content-Type: application/json" \
  -d '{"contestId":1,"problemId":1,"username":"alice","language":"java","sourceCode":"public class Main { public static void main(String[] a){ System.out.println(5); } }"}'

🧰 Troubleshooting
Issue	Cause	Fix
ERR_CONNECTION_REFUSED	Backend not running	Start backend (java -jar target/...jar or docker-compose up)
404 /api/...	Frontend hitting port 3000	Update API_BASE in [contestId].js or use next.config.js rewrite
Database connection error	Postgres container not ready	Wait 10s and restart backend
File too large when pushing to GitHub	node_modules committed	Use .gitignore and BFG to clean history
CORS error	Missing backend CORS config	Add CorsConfig.java (see docs)
🧱 Future Enhancements

✅ Add Python and C++ runner images

🔁 Replace polling with WebSocket push updates

🧑‍🏫 Instructor dashboard & contest creation UI

🧩 Multi-language sandbox pool

☁️ Deploy to AWS ECS or Kubernetes for scalability

📜 License

This project is open-sourced under the MIT License.
Feel free to use, modify, or extend it for educational or technical evaluations.

👨‍💻 Author

Manas Tripathi
Full-Stack Engineer & AI Developer
📧 [manastripathi.contact@gmail.com
]
🔗 GitHub: @ManasTripathi07
