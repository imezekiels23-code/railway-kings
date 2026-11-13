# Railway Kings — AWS & DevOps Project

Modern responsive cricket club website plus Docker, Jenkins pipeline and Kubernetes manifests.

## Structure
- `index.html`: page content
- `css/style.css`: responsive design
- `js/script.js`: sample squad data and mobile menu
- `Dockerfile`: packages the site into Nginx
- `Jenkinsfile`: validates, builds, pushes to Docker Hub and deploys to Kubernetes
- `k8s/`: namespace, Deployment and LoadBalancer Service

## Run locally
Open `index.html` in a browser or use VS Code Live Server.

## Customize
Edit the `players` array in `js/script.js` and replace sample match content in `index.html`. Replace the placeholder gallery backgrounds with your own team photos.

## Docker test
```bash
docker build -t railway-kings:local .
docker run --name railway-kings-local -d -p 8080:80 railway-kings:local
```
Open `http://localhost:8080`. Stop with `docker stop railway-kings-local` and remove with `docker rm railway-kings-local`.

## Docker Hub
Create a repository named `railway-kings`, replace `YOUR_DOCKERHUB_USERNAME`, then:
```bash
docker login
docker build -t YOUR_DOCKERHUB_USERNAME/railway-kings:v1 .
docker push YOUR_DOCKERHUB_USERNAME/railway-kings:v1
```
Use a Docker Hub access token in Jenkins, never a password in source code.

## Jenkins prerequisites
- Jenkins agent has Docker CLI/daemon and `kubectl`.
- Jenkins username/password credential ID `dockerhub-creds` holds Docker Hub username and access token.
- Jenkins Secret file credential ID `eks-kubeconfig` holds authorized EKS kubeconfig.
- Replace `YOUR_DOCKERHUB_USERNAME` in `Jenkinsfile` and `k8s/deployment.yaml`.
- Configure the job to load this repository's `Jenkinsfile`.

## EKS deployment
Configure `kubectl` for the EKS cluster, then apply manifests:
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods -n railway-kings
kubectl get svc -n railway-kings
```

## Cost and security
EKS, EC2 worker nodes, NAT Gateway, public IPv4 and Kubernetes LoadBalancer can incur AWS charges. Do not create cloud resources until local and Docker stages work and costs are checked. Never commit AWS credentials, kubeconfig or Docker Hub tokens to GitHub. Use least-privilege IAM permissions. Sample player names, results and email are placeholders.
