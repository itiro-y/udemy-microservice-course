### Arquitetura CI/CD

1) Developer -> pushes code to Github
2) Github -> hook kicks off in Github Actions
3) Github Actions -> builds Docker images and pushes to Docker Hub
	1) Docker Hub -> can now utilize our image in any machine with Docker installed
4) Google Artifact Registry -> can use our Docker image to deploy our app on Google Cloud
5) Google Cloud Run -> can deploy our app in one Google Cloud Run


### Usuario IAM

- Create Service Account
- GITHUB_ACTIONS
- Permissions
	- Artifact Registry Administrator
	- Google Cloud Run
	- Kubernetes Engine Admin
- Actions -> Manage Details -> Keys -> Add Key -> JSON

### Google Cloud Command Line Interface (CLI)

Installer at
https://cloud.google.com/sdk/docs/install?authuser=1


### Kubernetes Engine API (GKE)

1) Create Cluster
2) Abrir o Cloud Shell
3) Executar comandos para iniciar o deployment
4) kubectl create deployment hello-kubernetes --image=leandrocgsi/hello-kubernetes:erudio-0.0.1
5) kubectl expose deployment hello-kubernetes --type=LoadBalancer --port=8080
6) para acessar deployment -> Networking -> Gateways,.. -> Services -> Endpoints/hello-kubernetes



