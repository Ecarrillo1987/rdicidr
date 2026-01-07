# RDICIDR — FullStack Labs DevOps Coding Challenge (AWS + Kubernetes)

RDICIDR is a React application originally created to practice IPv4 subnetting.  
This repository contains the required deliverables for the **FullStack Labs DevOps (AWS + Kubernetes) Coding Challenge**:

## Challenge Deliverables Implemented

### Part 1 — Continuous Integration (CI)
A GitHub Actions CI workflow runs on **Pull Requests** and performs:
- Install dependencies
- Lint (ESLint)
- Test (Jest)
- Build

Workflow file:
- `.github/workflows/ci.yml`

### Part 2 — Local Kubernetes Deployment
The application is deployed locally in Kubernetes (Docker Desktop) using manifest files:
- Namespace: **production**
- Workload: **StatefulSet** with **3 replicas**
- Service: listens on **port 8080**
- Local access via host: **http://fsl-challenge.me** (mapped to localhost)

Manifests:
- `k8/namespace.yml`
- `k8/service.yml` (Service + StatefulSet)
- `k8/ingress.yml`
- `k8/secret.yml` (optional, only if ingress requires basic auth)

### Continuous Deployment (CD) — AWS
A GitHub Actions CD workflow deploys the app to **AWS ECS (Fargate)** as a containerized application.
Workflow file:
- `.github/workflows/cd.yml`

> Note: After the recording, scale the ECS service down to avoid costs.

---

## Repository Structure (Key Paths)

.github/workflows/
ci.yml
cd.yml

codebase/rdicidr-0.1.0/
Dockerfile
package.json
src/
...

k8/
namespace.yml
service.yml
ingress.yml
secret.yml
deployment.yml (legacy/alternative)
service-web.yml (legacy/alternative)


### About the extra manifests (deployment.yml / service-web.yml)
These files exist as alternatives/iterations from setup.  
**For the challenge requirements**, the authoritative manifests are:

- `k8/namespace.yml`
- `k8/service.yml`  ✅ (contains StatefulSet with 3 replicas + Service on 8080 in namespace `production`)
- `k8/ingress.yml`  ✅ (routes `fsl-challenge.me`)
- `k8/secret.yml`   ✅ only if ingress requires it

---

## Requirements (Local)

- Node.js **18.x**
- Docker Desktop (Linux containers)
- Kubernetes enabled in Docker Desktop
- `kubectl`

---

## Run the App Locally (Node)

```bash
cd codebase/rdicidr-0.1.0
npm install
npm run lint
CI=true npm run test
npm run build
npm start

Run the App Locally (Docker)
cd codebase/rdicidr-0.1.0
docker build -t fsl-frontend:1.0 .
docker run --rm -p 8080:8080 --name fsl-frontend fsl-frontend:1.0


Open:
http://localhost:8080

Local Kubernetes (Docker Desktop) — Required for the Challenge
1) Verify Kubernetes Context
kubectl config current-context
kubectl get nodes

2) Build the Image Locally
cd codebase/rdicidr-0.1.0
docker build -t fsl-frontend:1.0 .

3) Deploy Manifests

From the repository root:

kubectl apply -f k8/namespace.yml
kubectl apply -f k8/service.yml
kubectl apply -f k8/secret.yml    # only if required by ingress
kubectl apply -f k8/ingress.yml

4) Validate Requirements

Namespace must exist:

kubectl get ns | grep -i production


StatefulSet must have 3 replicas:

kubectl get sts -n production
kubectl get pods -n production -o wide


Service must listen on 8080:

kubectl get svc -n production
kubectl describe svc -n production fsl-frontend


Ingress must route fsl-challenge.me:

kubectl get ingress -n production
kubectl describe ingress -n production fsl-frontend-ingress

5) Configure Local DNS (hosts file)

Edit as Administrator:

Windows
C:\Windows\System32\drivers\etc\hosts

Add:

127.0.0.1 fsl-challenge.me


Open in browser:

http://fsl-challenge.me

Quick check:

curl -I http://fsl-challenge.me

CI — How to Demonstrate (PR evidence)

Create a branch and push changes

Open a Pull Request to main

Show the Checks tab in the PR:

Install

Lint

Test

Build

Workflow:

.github/workflows/ci.yml

CD (AWS ECS/Fargate) — How to Verify in AWS

In AWS Console:

ECS → Clusters → (your cluster) → Services → (your service)

Confirm Running tasks > 0 during the recording

If using an ALB, validate the public URL responds

Cost control (after recording)

Scale down desired count:

aws ecs update-service \
  --cluster <cluster-name-or-arn> \
  --service <service-name-or-arn> \
  --desired-count 0 \
  --region <region>

Original Application Note

Subnetting logic is implemented here:

codebase/rdicidr-0.1.0/src/lib/ipv4.js

License

See LICENSE.


---

## Git: pasos para commit + push (sin enredos)

Como ya tuviste problemas de “non-fast-forward”, te dejo el flujo **más seguro**:

```bash
# 1) Asegúrate de estar en main
git checkout main

# 2) Trae cambios remotos sin romper historial
git pull --rebase origin main

# 3) Crea rama para el README (recomendado para evidenciar CI en PR)
git checkout -b docs/readme-challenge

# 4) Agrega y commitea
git add README.md
git commit -m "docs: update README for FSL DevOps challenge"

# 5) Push de la rama
git push -u origin docs/readme-challenge

