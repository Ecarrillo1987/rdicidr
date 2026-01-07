# RDICIDR (React) — FullStack Labs DevOps Challenge (AWS + Kubernetes)

RDICIDR is a small React application originally created to practice IPv4 subnetting.  
This repository contains **DevOps challenge additions** required by FullStack Labs:

- **CI (GitHub Actions)** triggered on Pull Requests:
  - Install dependencies
  - Lint
  - Test
  - Build
- **Containerization** (Docker + Nginx) exposing the app on **port 8080**
- **Local Kubernetes deployment** using manifest files:
  - `Namespace`: `production`
  - `StatefulSet`: `fsl-frontend` with **3 replicas**
  - `Service`: listens on **port 8080**
  - `Ingress`: exposes `http://fsl-challenge.me` pointing to localhost
- **CD to AWS** (ECS/Fargate) via GitHub Actions to make the app publicly available during the recording.

---

## Repository Structure

├─ codebase/rdicidr-0.1.0/ # React application
│ ├─ Dockerfile # Builds and serves production build via Nginx (8080)
│ └─ ...
├─ k8/ # Kubernetes manifests (local)
│ ├─ namespace.yml # Namespace: production
│ ├─ service-web.yml # Service + StatefulSet (3 replicas) on port 8080
│ └─ ingress.yml # Host: fsl-challenge.me
└─ .github/workflows/
├─ ci.yml # CI pipeline (PR-triggered)
└─ cd.yml # CD pipeline (AWS ECS/Fargate)


---

## Requirements

### Local
- Node.js **18.x**
- Docker Desktop (Linux containers)
- Kubernetes enabled in Docker Desktop
- `kubectl`

### Cloud (for CD)
- An AWS account (ECS/Fargate + ECR recommended)
- GitHub repository secrets configured for the CD workflow (see CD section)

---

## Part 1 — Continuous Integration (CI)

### What it does
The CI workflow runs on **Pull Requests** and executes:
- Install dependencies (commonly `npm ci` for reproducible installs)
- Lint: `npm run lint`
- Test: `CI=true npm run test`
- Build: `npm run build`

Workflow file:
- `.github/workflows/ci.yml`

### How to demonstrate (for the video)
1. Create a feature branch and open a PR against `main`
2. Show the CI checks running in the PR “Checks” tab
3. (Optional but recommended) Create a second PR that fails (e.g., introduce an ESLint issue) to show feedback.

---

## Run Locally (Node)

From the app directory:

```bash
cd codebase/rdicidr-0.1.0
npm install
npm run lint
CI=true npm run test
npm run build
npm start

Run Locally (Docker)

Build and run the container (port 8080):

cd codebase/rdicidr-0.1.0
docker build -t fsl-frontend:1.0 .
docker run --rm -p 8080:8080 --name fsl-frontend fsl-frontend:1.0

Open:
 http://localhost:8080

Part 2 — Local Kubernetes (Docker Desktop)
1) Verify Kubernetes context

kubectl config current-context
kubectl get nodes

2) Build the Docker image (local)

cd codebase/rdicidr-0.1.0
docker build -t fsl-frontend:1.0 .

Note: Docker Desktop Kubernetes can use local images with imagePullPolicy: IfNotPresent.

3) Deploy manifests
From repository root:

kubectl apply -f k8/namespace.yml
kubectl apply -f k8/service-web.yml
kubectl apply -f k8/ingress.yml


4) Validate resources (must match challenge requirements)
Namespace:

kubectl get ns | findstr production

StatefulSet must be 3 replicas:

kubectl get sts -n production
kubectl get pods -n production -o wide

Service must expose 8080:

kubectl get svc -n production

Ingress must use host fsl-challenge.me:

kubectl get ingress -n production
kubectl describe ingress -n production fsl-frontend-ingress


5) Access the app at http://fsl-challenge.me
 (localhost mapping)

Edit hosts file as Administrator:

Windows
C:\Windows\System32\drivers\etc\hosts

Add:
127.0.0.1 fsl-challenge.me


Then open:

http://fsl-challenge.me

Optional: quick smoke test via curl

curl -I http://fsl-challenge.me


Continuous Deployment (CD) — AWS ECS/Fargate
The CD workflow deploys the container image to AWS so the app is publicly reachable during the recording.
Workflow file:
.github/workflows/cd.yml
Expected outcomes (for the video)
GitHub Actions shows a successful CD run
In AWS ECS: cluster/service has running tasks
Public endpoint (e.g., ALB DNS) responds during the recording
Cost control (recommended)
After recording, scale down the ECS service desired count to 0 to avoid costs.

Cleanup
Local Kubernetes
kubectl delete -f k8/ingress.yml
kubectl delete -f k8/service-web.yml
kubectl delete -f k8/namespace.yml

AWS ECS (scale down)

(Use your cluster/service values)

aws ecs update-service \
  --cluster <cluster-arn-or-name> \
  --service <service-arn-or-name> \
  --desired-count 0 \
  --region <region>

Notes about the original app
Subnetting logic can be reviewed here:
codebase/rdicidr-0.1.0/src/lib/ipv4.js

License

---
# Git: pasos para commit + push (recomendado con PR para que CI corra)
Como el reto exige PRs, lo mejor es **no empujar directo a `main`**, sino hacerlo por rama y PR (así queda evidencia del CI).
## Opción recomendada: rama + PR
```bash
# 1) Asegúrate de estar actualizado
git checkout main
git pull --rebase origin main

# 2) Crea rama de docs
git checkout -b docs/readme-challenge

# 3) Agrega y commitea
git add README.md
git commit -m "docs: improve README for FSL DevOps challenge"

# 4) Push de la rama
git push -u origin docs/readme-challenge

Opción alternativa: push directo a main (solo si así lo estás llevando)
git checkout main
git pull --rebase origin main
git add README.md
git commit -m "docs: improve README for FSL DevOps challenge"
git push origin main

Luego en GitHub: Open Pull Request hacia main y muestras el CI pasando.

