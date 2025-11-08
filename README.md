# Gitea Deployment: From Development to Production

In this assignment, we are taking on a basic development deployment of Gitea, a lightweight self-hosted Git service, and transforming it into production-ready deployment running on Kubernetes.

In the development environment, Gitea is deployed using an Ansible playbook and runs with a local SQLite database and port-forwarding, which is suitable for testing but not reliable for real usage.

The goal of the assignment is to upgrade this setup into a production environment by:

- Deploying Gitea using Helm
- Ensuring repository data persists
- Configuring Gitea to use an external MariaDB (MySQL) database instead of SQLite
- Exposing the application publicly using an Ingress and a real domain

By completing these changes, the application becomes:
- Durable (data survives restarts)
- Scalable (standardized Helm deployment)
- Secure and maintainable (real database backend)
- Accessible over the internet

This README explains the steps taken to perform the migration from dev → production and provides verification output for each requirement.
