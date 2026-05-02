# Python Flask App — Dockerized CI/CD Pipeline

> A production-ready Flask blueprint with automated container builds, Git SHA tagging, and zero-downtime deployment semantics.


---

## ⚙️ Tech Stack (Developer View)

| Layer          | Technology                          |
|----------------|-------------------------------------|
| Language       | Python 3.13.3                       |
| Web Framework  | Flask 3.x                           |
| Containerization| Docker (OCI-compliant)             |
| CI/CD Pipeline | GitHub Actions (YAML-based)        |
| Registry       | Docker Hub                          |
| Versioning     | Git commit SHA (short, 7 chars)     |
| Base Image     | `python:3.13.3-slim` (Debian-based) |

---

## K8S setup for this project
Python-Src repo -> [github/shawn-exe-K8s-Resources-CICD_ONE](https://github.com/shawn-exe/CICD_ONE_Infra_Repo.git)

---


## 🔁 CI/CD Pipeline — What Happens on `git push main`

1. **Trigger** — Push to `main` branch  
2. **Checkout** — Full repo clone  
3. **Docker Build** — `docker build -t $IMAGE_NAME:$GIT_SHA .`  
4. **Docker Tag** — Tags with Git SHA  
5. **Authenticate** — Docker Hub login via GitHub Secrets  
6. **Push** — `docker push $IMAGE_NAME:$GIT_SHA`  
7. **No deploy step** (designed for downstream orchestration)

### 📦 Image Naming Convention
docker.io/<YOUR_USERNAME>/cicd_one_python_app:<7-char-git-sha>

text

Example:  
`docker.io/octocat/cicd_one_python_app:a1b2c3d`

---

## 🚀 Local Development (Fast Iteration)

### Option 1: Bare Metal (no Docker)

```bash
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
cd src
python app.py
Server → http://localhost:5000

Option 2: Docker (local build)
bash
docker build -t cicd_one_python_app:dev .
docker run -p 5000:5000 cicd_one_python_app:dev
Option 3: Docker with live reload (volume mount)
bash
docker run -p 5000:5000 -v $(pwd)/src:/src cicd_one_python_app:dev
Flask auto-reloads on code changes (requires debug=True in app.py)

🔐 Required GitHub Secrets
Secret Name	Purpose	Where to get it
DOCKER_USERNAME	Docker Hub login	Docker Hub account username
DOCKER_PASSWORD	Docker Hub auth	Docker Hub → Account Settings → Security → Access Token (recommended over password)
Security note: Use an access token with read,write scope, never your account password.

🐳 Dockerfile Explained (Optimized for caching)
dockerfile
FROM python:3.13.3-slim

# Dependency layer — changes rarely → cached
COPY requirements.txt /tmp/
RUN pip install --no-cache-dir -r /tmp/requirements.txt

# Application layer — changes frequently
COPY ./src /src

EXPOSE 5000
CMD ["python", "/src/app.py"]
Why this structure?

requirements.txt changes infrequently → Docker reuses cached layer

Code changes frequently → only last layer rebuilds

🧪 Testing the Container Locally (Pre-Push)
bash
# Build
docker build -t cicd_one_python_app:test .

# Run
docker run -d -p 5000:5000 --name flask_test cicd_one_python_app:test

# Health check
curl http://localhost:5000

# Cleanup
docker stop flask_test && docker rm flask_test
📦 Pulling from Docker Hub (Deployment)
bash
# Latest commit on main
docker pull <YOUR_USERNAME>/cicd_one_python_app:<GIT_SHA>

# Run it
docker run -d -p 5000:5000 <YOUR_USERNAME>/cicd_one_python_app:<GIT_SHA>
🧠 Advanced: Tagging Strategies
Current: Git SHA (immutable, traceable)
Recommended next:

Git tag versioning → v1.2.3 + latest

Environment tags → staging, production

Multi-arch builds → --platform linux/amd64,linux/arm64

🛠️ Troubleshooting (Developer Edition)
❌ docker: command not found
→ Install Docker Desktop or Docker Engine

❌ ERROR: Cannot connect to the Docker daemon
→ Start Docker service:
sudo systemctl start docker (Linux) or open Docker Desktop

❌ denied: requested access to the resource is denied
→ GitHub Actions secret misconfigured or expired token

❌ Flask not starting inside container
→ Check CMD in Dockerfile — must point to absolute path
→ Verify EXPOSE 5000 matches Flask's default port

❌ Address already in use on port 5000
→ Kill process using port:
lsof -i :5000 → kill -9 <PID>

📊 Performance Notes
Image size: python:3.13.3-slim → ~120MB compressed

Build time: ~30–45 seconds (cached dependencies)

Cold start: ~0.5s (Flask lightweight)

GitHub Actions runner: Ubuntu latest, ~7GB storage

🧩 Extensibility Hooks
You can easily extend this pipeline with:

Feature	Add in .github/workflows/cicd.yml
Run tests before build	pip install pytest && pytest tests/
Lint with flake8	pip install flake8 && flake8 src/
Push latest tag also	docker tag ... latest + second docker push
Slack notification	actions/slack@v1
Trivy vulnerability scan	aquasecurity/trivy-action

👨‍💻 Author / Maintainer
Built for developer velocity — contributions welcome via PRs.

⭐ Developer Cred
If this saved you CI/CD headache — star the repo.
PRs for multi-stage builds or Helm charts welcome.