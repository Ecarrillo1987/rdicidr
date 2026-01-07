# rdiCidr

Ok, let's get this out of the way. Why did I choose that name? I chose the name because palindromes are fun. The app's name stands for RDerik's Interactive CIDR (RDICIDR). I had to add the Interactive to make the name work. I hope you like it.

RDICIDR is a React app. If you want to focus on the subnetting logic, check:

```bash
src/lib/ipv4.js

You should find there how each property was calculated.
This application is not a final product. I built it to refresh my subnetting knowledge. Enjoy.
Live version
A live version can be found at:
https://rdicidr.rderik.com/
If you find this tool useful, you might enjoy reading my blog:
https://rderik.com/

FullStack Labs DevOps Challenge Notes
This repository was forked / copied into my own GitHub repository to implement the required CI pipeline and a local Kubernetes deployment using Docker Desktop.
What was added
•	GitHub Actions CI workflow:
o	npm ci
o	npm run lint
o	npm test -- --watchAll=false
o	npm run build
•	Dockerfile to build and serve the React production build via Nginx on port 8080
•	Kubernetes manifests:
o	Namespace
o	StatefulSet
o	Service on port 8080
•	Local access through http://fsl-challenge.me

Requirements
•	Node.js 18.x
•	Docker Desktop (Linux containers)
•	Kubernetes enabled in Docker Desktop
•	kubectl
Run locally (Node)
•	From the app directory:
cd codebase/rdicidr-0.1.0
npm ci
npm run lint
npm test -- --watchAll=false
npm run build
npm start

Run locally (Docker)

Build and run the container (exposes 8080):

cd codebase/rdicidr-0.1.0
docker build -t fsl-frontend:1.0 .
docker run --rm -p 8080:8080 --name fsl-frontend fsl-frontend:1.0

Open:

http://localhost:8080/

Local Kubernetes (Docker Desktop)
1) Verify Kubernetes is ready
kubectl config current-context
kubectl get nodes
You should see docker-desktop in Ready status.

2) Build Docker image
cd codebase/rdicidr-0.1.0
docker build -t fsl-frontend:1.0 .

3) Deploy (Namespace + StatefulSet + Service)
From the repository root:
kubectl apply -f k8/namespace.yml
kubectl apply -f k8/service.yml
kubectl -n fsl-challenge get pods
kubectl -n fsl-challenge get svc

Expected:

Pod fsl-frontend-0 should be Running

Service fsl-frontend exposes port 8080/TCP

4) Access the app at http://fsl-challenge.me

Start port-forwarding:
kubectl -n fsl-challenge port-forward svc/fsl-frontend 80:8080

Add a hosts entry (Windows):

Edit as Administrator:

C:\Windows\System32\drivers\etc\hosts

Add:
127.0.0.1 fsl-challenge.me
Open:

http://fsl-challenge.me

5) Cleanup (optional)
kubectl delete -f k8/service.yml
kubectl delete -f k8/namespace.yml

CI (GitHub Actions)

The CI workflow runs on pushes and pull requests to main and performs:

Install dependencies (npm ci)

Lint (npm run lint)

Tests (npm test -- --watchAll=false)

Build (npm run build)

Workflow file location:

.github/workflows/ci.yml

Troubleshooting
Kubernetes pod stuck in ImagePullBackOff

If Kubernetes tries to pull the image instead of using the local one, set:

imagePullPolicy: Never


in the StatefulSet container spec and re-apply:

kubectl apply -f k8/service.yml

Port already in use (80 or 8080)

Stop any process using the port, or change the port-forward mapping. Example:

kubectl -n fsl-challenge port-forward svc/fsl-frontend 8081:8080


Then open http://localhost:8081.

License

See LICENSE.


Si quieres, te doy el **commit exacto** recomendado para este README (y el resto de cambios), pero si ya estás listo:

```bash
git add README.md
git commit -m "docs: add CI, Docker and local Kubernetes deployment instructions"
git push origin HEAD

# ci check
