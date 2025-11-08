# Gitea Deployment: From Development to Production

In this assignment, we are taking on a basic development deployment of Gitea, a lightweight self-hosted Git service, and transforming it into production-ready deployment running on Kubernetes.

In the development environment, Gitea is deployed using an Ansible playbook and runs with a local SQLite database and port-forwarding, which is suitable for testing but not reliable for real usage.

The goal of the assignment is to upgrade this setup into a production environment by:

- Deploying Gitea using Helm
- Ensuring repository data persists
- Configuring Gitea to use an external MariaDB (MySQL) database instead of SQLite
- Exposing the application publicly using an Ingress and a real domain

## Exposing the Gitea instance publicly

In the development setup, Gitea was only accessible locally with:
    `kubectl port-forward svc/gitea-http 3000:3000`

To make it accessible outside the cluster publicly, we configured an Ingress resource that exposes the service through a Ingress Controller and mapped it to a public domain name. 

Steps:
- `kubectl get pods -n ingress-nginx`: To verify that the Ingress controller was running
- I modified ingress.yml file to route traffic to Gitea HTTP service
- Got external IP from ingress controller: `kubectl get svc -n ingress-nginx`
- Checked the UI at: `http://cibisharan-gitea.k3p.dev`

## Using Gitea Helm Chart to make repository data persistent

Until now, the data stored was not persistent. If the pod restarted, all repositories and config data would be lost. To ensure the data persistence, I redeployed gitea with the official Helm Chart. It allows the Persistent Volumn Claim (PVC) to store repository data.

Steps:
- Created a namespace for prod deployment: `kubectl create namespace gitea-prod`
- Added Gitea Helm chart repository: 

    `helm repo add gitea https://dl.gitea.com/charts/`

    `helm repo update`

- Created values-prod.yml to enable persistent storage.
- Deployed gitea through Helm with: `helm upgrade --install gitea gitea/gitea -n gitea-prod -f values-prod.yaml`

## Make Gitea Use an External Database

Until now, Gitea used the default SQLite Database, which stores all data inside application container. This storage type is not suitable for production as the data would be lost if pod is replaced. To make deployment for production-ready use cases, Gitea was configured to use external MariaDB database running inside the cluster. 

Steps:
- Deployed MariaDB in production namespace using Helm

 `helm repo add bitnami https://charts.bitnami.com/bitnami`

`helm install gitea-mariadb bitnami/mariadb -n gitea-prod \
  --set auth.username=gitea \
  --set auth.password=gitea123 \
  --set auth.database=gitea \
  --set primary.persistence.size=5Gi`

- Updated the Gitea Helm values to connect to MariaDB
- Applied the updated configuration: `helm upgrade --install gitea gitea/gitea -n gitea-prod -f values-prod.yaml`

