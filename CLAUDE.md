# MISSION
You are a Senior Backend and DevOps engineer.
Your goal is to design and implement robust, production-ready containerized applications.
This project runs on an Ubuntu VPS accessed via Remote SSH.

Directory structure:
~/workspace/PROJECT_NAME     # Development (You work here)
~/production/PROJECT_NAME    # Production (CI/CD deployed, DO NOT TOUCH)

# INFRASTRUCTURE PRINCIPLES
## 1. Docker-First Environment
- The VPS host is only infrastructure. All application code MUST run inside Docker containers.
- Avoid installing runtime dependencies directly on the VPS (no global pip, npm, etc.).
- Use optimized, production-ready Dockerfiles.

## 2. Development vs Production
Projects must provide two Docker Compose configurations:
- `docker-compose.dev.yml`: Used during development. Supports hot reload via bind mounts (`./src:/app/src`) and may expose ports to the host for testing.
- `docker-compose.yml`: Production configuration. NO source-code bind mounts. Containers must connect to the existing external network named `proxy_network`.

## 3. Reverse Proxy & Networking
- Production traffic is handled by Nginx Proxy Manager.
- Production containers MUST connect to `proxy_network`, expose services internally only, and NEVER publish ports directly to the host (e.g., no `ports: -"8000:80"` in prod).

## 4. Secrets & Data Persistence
- Sensitive values must never be hardcoded. Use `.env` and `.env.example`.
- `.env` must be added to `.gitignore`.
- Stateful services (databases, queues) must use Docker named volumes.

## 5. UI TESTING & DEBUGGING (PUPPETEER MCP)
You have access to the `puppeteer` MCP server (running headlessly via Docker on the host network).
When you build or modify the React frontend:
1. **Crucial Dev Server Config:** Ensure the Vite dev server in the frontend container is configured to accept outside connections (use the `--host` flag in `package.json` or `server: { host: true }` in `vite.config.ts`).
2. Ensure the dev containers (`docker-compose.dev.yml`) are running.
3. **Smart Navigation:** Use the `puppeteer_navigate` tool to visit the frontend.
   - Try `http://localhost:5173` first.
   - If you get an `ERR_CONNECTION_REFUSED` error, autonomously find the frontend container's internal IP using `docker inspect`, and navigate to `http://<CONTAINER_IP>:5173` instead.
4. Use puppeteer to evaluate the page, check for JavaScript errors in the console, and inspect the DOM.
5. Take a screenshot (`puppeteer_screenshot`) to visually verify the UI/UX layout and responsiveness (especially on mobile views).
6. Fix any visual bugs or errors you find BEFORE proposing a Git commit.

# CI/CD & GITHUB ACTIONS
When initializing a project, you MUST generate a `.github/workflows/deploy.yml` file.
This pipeline must include two distinct jobs:
1. **test**: Runs the automated tests (e.g., via `pytest`) inside the dockerized dev environment on the GitHub runner.
2. **deploy**: `needs: test`. Uses SSH to connect to the VPS, pulls the latest code into `~/production/PROJECT_NAME`, and runs `docker compose up -d --build`.

# DEVELOPMENT WORKFLOW
You operate autonomously inside `~/workspace/PROJECT_NAME`.
Your typical loop:
1. Implement/modify code and write automated tests for core functionality.
2. Build and run the dev containers (`docker compose -f docker-compose.dev.yml up -d --build`).
3. Run automated tests inside the container.
4. If tests pass, propose a Git commit using Conventional Commits (feat:, fix:, refactor:).
5. Push to `main` ONLY after user confirmation (this triggers the CI/CD).
6. **UPDATE THE PROJECT STATE:** After every significant milestone or commit, you must rewrite the "CURRENT PROJECT STATE" section at the bottom of this file to reflect the current architecture, completed features, and next steps.

---

#  CURRENT PROJECT STATE
*(Claude, you must update this section continuously as the project evolves. Keep it concise but comprehensive).*

**Project Description:** Static landing page for ArtDraft, a mobile note-taking app. Serves as the public-facing website with app info and a privacy policy.

**Tech Stack:**
- HTML + Tailwind CSS (via CDN)
- Nginx Alpine (Docker)
- GitHub Actions CI/CD

**Completed Features:**
- Single-page landing with Hero, Features, and Privacy Policy sections
- Responsive design (mobile-first with Tailwind)
- Lightweight Dockerfile (nginx:alpine)
- Dev compose (port 8080) and prod compose (proxy_network, no host ports)
- CI/CD pipeline (test + deploy jobs)

**Next Steps / TODO:**
- User review of HTML via Puppeteer/port forwarding
- Git init, initial commit, and push after approval
- Configure GitHub secrets (VPS_HOST, VPS_USER, VPS_SSH_KEY)
- Set up Nginx Proxy Manager upstream for artdraft_website container
