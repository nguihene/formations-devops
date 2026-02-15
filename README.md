# Denv-r Template Project

> [!TIP]
> **🎓 Formation DevSecOps disponible !**
> Ce repo sert de support à un workshop de 4 jours. Consultez [WORKSHOP.md](./WORKSHOP.md) pour le programme complet.

## 📚 Ressources Formation

| Ressource | Description |
|-----------|-------------|
| [WORKSHOP.md](./WORKSHOP.md) | Programme structuré Jour 1 + Jour 2 |
| [theory/](./theory/) | Modules théoriques (DevOps, Cloud, GitOps) |
| [exercises/](./exercises/) | Exercices pratiques progressifs |
| [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) | Guide de résolution d'erreurs |
| [AI_TRAPS.md](./AI_TRAPS.md) | Pièges IA pour développer l'esprit critique |

---

This project is a template to:

- Build and push a containerized NextJS app to GitHub registry
- Manage VMs in Denv-r cloud environment using Terraform
- Deploy the containerized app on VMs using Ansible

> [!NOTE]
> This Terraform project is configured for Denv-r cloud using the Warren provider.
> You can also use the CI and Ansible to build, publish and deploy your NextJS app on your own VMs accessible via SSH.


## Prerequisites

Using the CI/CD only, to build, push and deploy, you just need to install :
- Terraform

If you want to run all this actions locally first then you need :
- npm : build and run locally the NextJS application following the README.md file on "my-app" sub-directory
- Docker with docker compose : build and run contenerized version of the app
- Terraform : deploy VMs in your Denv-r cloud environment
- Ansible : deploy the contenerized app on your VMs

## 📥 Setup & Submodules

This repository uses Git submodules (specifically the `capstone` project). 
Use the following commands to ensure you have all the necessary code:

**Clone with submodules:**
```bash
# SSH
git clone --recursive git@github.com:dis-bzh/formations-devops.git

# ou HTTPS
git clone --recursive https://github.com/dis-bzh/formations-devops.git
```

**If you already cloned the repo:**
```bash
git submodule update --init --recursive
```

## Github workflow

Le projet utilise 3 workflows CI/CD qui forment un pipeline DevSecOps complet :

| Workflow | Déclencheur | Rôle |
|----------|-------------|------|
| `security.yml` | Push/PR → `main` | Scans de sécurité : dépendances (Snyk), secrets (Gitleaks), code (CodeQL) |
| `build.yml` | Push d'un **tag** | Build Docker, push sur GHCR, scan Trivy de l'image |
| `deploy.yml` | Après `build.yml` (succès) | Terraform (conditionnel) + Ansible avec approbation manuelle |

> 📖 Voir [Exercice 02 — Premier Workflow](./exercises/devops-j1/02-premier-workflow.md) pour une analyse détaillée de chaque workflow.

### Secrets (Settings → Secrets and variables → Actions)

| Secret | Workflow | Usage |
|--------|----------|-------|
| `SNYK_TOKEN` | security.yml | Token API [snyk.io](https://snyk.io) pour scan des dépendances |
| `S3_ACCESS_KEY_ID` | deploy.yml | Accès au backend S3 (state Terraform) |
| `S3_SECRET_ACCESS_KEY` | deploy.yml | Accès au backend S3 (state Terraform) |
| `API_TOKEN` | deploy.yml | Token API du provider cloud (Denv-r) |
| `SSH_PRIVATE_KEY` | deploy.yml | Clé SSH pour Ansible |
| `ANSIBLE_USER` | deploy.yml | Utilisateur SSH sur les VMs |

### Variables (Settings → Secrets and variables → Actions → Variables)

| Variable | Workflow | Usage |
|----------|----------|-------|
| `S3_BUCKET` | deploy.yml | Nom du bucket S3 pour le state Terraform |
| `S3_KEY` | deploy.yml | Chemin du fichier state dans le bucket |
| `S3_REGION` | deploy.yml | Région du bucket S3 |
| `S3_ENDPOINT_URL` | deploy.yml | Endpoint S3 (Denv-r, OVH, Scaleway…) |

### Secrets automatiques (fournis par GitHub)

| Secret | Usage |
|--------|-------|
| `GITHUB_TOKEN` | Login GHCR, push d'images, approbations manuelles, Gitleaks |

## Ansible

Template to manage VMs configuration. It uses the inventory created by Terraform.
It can be run locally, automatically triggered in the CI when "Build and Push" workflow is "completed", manually in the CI.

To run it locally, use the following command :
```bash
ansible-playbook -i path/to/inventory path/to/playbook.yml \
--private-key path/to/sshPrivateKey \
-u username \
--ssh-common-args='-o StrictHostKeyChecking=no' \
--extra-vars ansible_user=username \
--extra-vars registry_username=github_username \
--extra-vars registry_token=github_access_token \
--extra-vars image_name=containerImage:tag
--extra-vars host_port=80
--extra-vars container_port=80
```

## Terraform

Terraform is not integrated in the CI for now.
To create and manage compute and storage resources like Virtual Machines using Terraform you must register a new API token at your [user interface](https://app.denv-r.com/)

### S3 backend

The Terraform State (tfstate) should be considered as a secret and its very important to keep it safe.
Using an S3 backend is one of the best practice option (avoid using static file in production).

You can create an S3 bucket from the UI or calling the API according to [the Denv-r API documentation](https://api.denv-r.com/#create-bucket).

```bash
curl --location --request PUT "https://api.denv-r.com/v1/storage/bucket" \
    -H "apikey: meowmeowmeow" \
    -d "name=pang1" \
    -d "billing_account_id=12345"
```

Then retrieve (or generate) `accessKey` and `secretKey`

```bash
curl "https://api.denv-r.com/v1/storage/user/keys" \
    -H "apikey: meowmeowmeow" \
    -X GET
```

### variables

backend.tfvars file contains S3 backend configuration, as backend is loading first.
terraform.tfvars file contains the details of your infrastructure like the networks, the number of Vms, the resources you will allocate to them, ...

```bash
cp backend.tfvars.example backend.tfars
cp terraform.tfvars.example terraform.tfars
```

To create and manage your infrastructure, just provide the Denv-r API Token, S3 access key and S3 secret key then execute the commands below.

```bash
export TF_VAR_api_token="xxx"
# $env:TF_VAR_api_token="xxx" in Powershell

terraform init -backend-config=backend.tfvars
terraform plan -out tf.plan
terraform apply "tf.plan"
```
