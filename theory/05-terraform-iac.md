# 🏗️ Terraform & Infrastructure as Code

> *"Décrivez votre infrastructure, Terraform s'occupe du reste."*

## 🎯 Objectifs pédagogiques
- Comprendre le concept d'Infrastructure as Code (IaC)
- Maîtriser le workflow Terraform : init → plan → apply → destroy
- Connaître la gestion de l'état et les bonnes pratiques

---

## 📖 Infrastructure as Code

### Pourquoi l'IaC ?

```
┌─────────────────────────────────────────────────────┐
│                   AVANT (Manuel)                     │
│                                                      │
│  Admin ──► Console Cloud ──► Clic clic clic ──► VM   │
│                                                      │
│  Problèmes :                                         │
│  - Non reproductible                                 │
│  - Non versionné                                     │
│  - Erreurs humaines                                  │
│  - "C'est qui qui a créé ce truc ?"                  │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                   APRÈS (IaC)                        │
│                                                      │
│  Admin ──► Code (.tf) ──► Git ──► terraform apply    │
│                                                      │
│  Avantages :                                         │
│  - Reproductible                                     │
│  - Versionné (Git)                                   │
│  - Revue de code possible                            │
│  - Auditabilité complète                             │
└─────────────────────────────────────────────────────┘
```

### Déclaratif vs Impératif

| Approche | Description | Exemple |
|----------|-------------|---------|
| **Déclaratif** | "Je veux 3 VMs" — l'outil fait le nécessaire | Terraform, Ansible |
| **Impératif** | "Créer VM1, puis VM2, puis VM3" — étape par étape | Scripts Bash, SDK |

> [!TIP]
> Terraform est **déclaratif** : on décrit l'état souhaité, Terraform calcule les actions nécessaires pour y arriver.

---

## 🔧 Qu'est-ce que Terraform ?

| Aspect | Détail |
|--------|--------|
| **Créé par** | HashiCorp (2014) |
| **Langage** | HCL (HashiCorp Configuration Language) |
| **Modèle** | Déclaratif, idempotent |
| **Multi-cloud** | AWS, Azure, GCP, Denv-r, Kubernetes... |
| **Licence** | BSL (anciennement open source) |
| **Alternative** | OpenTofu (fork open source) |

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                  TERRAFORM                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │  main.tf    │  │ variables.tf│  │  outputs.tf  │  │
│  │ (resources) │  │ (paramètres)│  │ (résultats)  │  │
│  └──────┬──────┘  └─────────────┘  └─────────────┘  │
│         │                                            │
│  ┌──────▼──────┐                                     │
│  │  Provider   │  ← Plugin pour chaque Cloud         │
│  │  (Warren)   │                                     │
│  └──────┬──────┘                                     │
│         │ API                                        │
│  ┌──────▼──────┐                                     │
│  │  Cloud API  │  ← Denv-r, AWS, Azure...            │
│  └─────────────┘                                     │
└─────────────────────────────────────────────────────┘
```

---

## 📁 Structure d'un projet

| Fichier | Rôle |
|---------|------|
| `main.tf` | Ressources à créer (VMs, réseaux, IPs...) |
| `variables.tf` | Déclaration des variables avec types et defaults |
| `terraform.tfvars` | Valeurs des variables (non commité si sensible) |
| `outputs.tf` | Informations exportées (IPs, URLs...) |
| `backend.tfvars` | Configuration du backend distant (S3...) |
| `terraform.tfstate` | État actuel de l'infrastructure (⚠️ secret !) |

### Exemple : Créer une VM Denv-r

```hcl
# main.tf
terraform {
  required_providers {
    warren = {
      source  = "WarrenCloudPlatform/warren"
      version = "0.1.3"
    }
  }
}

provider "warren" {
  api_url   = "https://api.denv-r.com/v1"
  api_token = var.api_token
}

resource "warren_virtual_machine" "vm" {
  count           = var.vm_number
  name            = "${var.vm_prefix}-${count.index}"
  disk_size_in_gb = var.disk_size
  memory          = var.ram
  vcpu            = var.cpu
  username        = var.username
  os_name         = "ubuntu"
  os_version      = "22.04"
  public_key      = var.ssh_public_key
  network_uuid    = data.warren_network.public.id
}
```

```hcl
# variables.tf
variable "api_token" {
  description = "API token for the Warren provider"
  type        = string
  sensitive   = true
}

variable "vm_prefix" {
  description = "Prefix for VM names"
  default     = "denvr"
}

variable "vm_number" {
  description = "Number of VMs to create"
  type        = number
  default     = 1
}
```

---

## 🔄 Workflow Terraform

### Le cycle de vie

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│   init   │────►│   plan   │────►│  apply   │────►│ destroy  │
│          │     │          │     │          │     │          │
│ Installe │     │ Prévisua-│     │ Exécute  │     │ Supprime │
│ providers│     │ lise     │     │ le plan  │     │ tout     │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
```

```bash
# 1. Initialise le projet (télécharge les providers)
terraform init

# 2. Planifie les changements (dry-run)
terraform plan -out tf.plan

# 3. Applique UNIQUEMENT le plan vérifié
terraform apply tf.plan

# 4. Détruit l'infrastructure (fin de lab)
terraform destroy
```

> [!CAUTION]
> **Règle d'or** : Toujours faire `plan -out` puis `apply` sur le plan.
> **Ne JAMAIS utiliser `terraform apply -auto-approve`** en dehors d'un pipeline CI contrôlé.
> Le `plan -out` garantit que seules les modifications prévues et vérifiées seront appliquées.

### Commandes d'état utiles

```bash
terraform state list          # Lister les ressources gérées
terraform state show <ress>   # Détails d'une ressource
terraform state mv <old> <new># Renommer dans l'état
terraform state rm <ress>     # Retirer de l'état (sans détruire)
```

---

## 💾 Gestion de l'état (State)

### Pourquoi le state est important

Le fichier `terraform.tfstate` est la **mémoire** de Terraform :
- Il sait quelles ressources existent réellement
- Il compare l'état désiré (code) vs l'état réel (infra)
- Il calcule les changements nécessaires

### State local vs distant

| Aspect | Local | Distant (S3) |
|--------|-------|---------------|
| **Fichier** | `terraform.tfstate` | Bucket S3 |
| **Collaboration** | ❌ Un seul utilisateur | ✅ Équipe |
| **Verrouillage** | ❌ Non | ✅ Oui (DynamoDB) |
| **Sécurité** | ⚠️ En clair sur disque | ✅ Chiffré |
| **Sauvegarde** | ❌ Manuelle | ✅ Automatique |

> [!WARNING]
> Le state contient des **données sensibles** (tokens, IPs, configurations). Il doit être traité comme un **secret** :
> - Ne **jamais** le commiter dans Git
> - Utiliser un backend distant (S3) en production
> - Configurer le chiffrement

---

## 🧩 Expressions et dynamique

### Boucles et conditions

```hcl
# Créer N instances avec count
resource "warren_virtual_machine" "vm" {
  count = var.vm_number
  name  = "${var.vm_prefix}-${count.index}"
}

# Condition : créer seulement si activé
resource "warren_floating_ip" "ip" {
  count       = var.enable_public_ip ? var.vm_number : 0
  name        = "ip-${count.index}"
  assigned_to = warren_virtual_machine.vm[count.index].id
}
```

### for_each (itération sur map/set)

```hcl
variable "vms" {
  default = {
    web  = { cpu = 2, ram = 2048 }
    api  = { cpu = 1, ram = 1024 }
  }
}

resource "warren_virtual_machine" "vm" {
  for_each = var.vms
  name     = each.key
  vcpu     = each.value.cpu
  memory   = each.value.ram
}
```

---

## 🔗 Intégration Terraform + Ansible

```
┌──────────┐          ┌──────────┐          ┌──────────┐
│Terraform │──output──►│Inventory │──input───►│ Ansible  │
│(infra)   │  (IPs)   │ (.ini)   │          │(config)  │
└──────────┘          └──────────┘          └──────────┘
```

### Outputs vers Ansible

```hcl
# outputs.tf
output "vm_ips" {
  description = "Public IPs of VMs"
  value       = warren_floating_ip.ip[*].ip_address
}
```

> [!TIP]
> Utilisez les **outputs** et le **templating** Terraform pour générer l'inventaire Ansible automatiquement.
> Cela permet une chaîne complète : `Terraform crée l'infra` → `Output les IPs` → `Ansible configure les VMs`.

---

## 🏢 Workspaces (multi-environnement)

```bash
terraform workspace new dev       # Créer l'env dev
terraform workspace new prod      # Créer l'env prod
terraform workspace select dev    # Basculer sur dev
terraform workspace list          # Lister les envs
```

Chaque workspace a son propre state → isolation des environnements.

---

## ⚠️ Erreurs courantes

| Erreur | Cause | Solution |
|--------|-------|----------|
| Variables manquantes | `terraform.tfvars` incomplet | Vérifier toutes les variables requises |
| State corrompu | Modification manuelle du `.tfstate` | Restaurer backup, ne jamais éditer manuellement |
| Conflit de version provider | Version incompatible | Contraindre dans `required_providers` |
| Dépendance circulaire | Ressources A ↔ B | Utiliser `depends_on` ou restructurer |
| Lock sur le state | Exécution concurrente | `terraform force-unlock <ID>` |

---

## ❓ Pourquoi c'est important en 2026 ?

> [!IMPORTANT]
> L'IaC est **indispensable** dans tout environnement moderne :
> - Reproductibilité des environnements (dev = staging = prod)
> - Revue de code sur l'infrastructure (Pull Requests)
> - Audit et conformité automatisés
> - Base du GitOps appliqué à l'infrastructure

---

## 📚 Sources officielles

| Ressource | Lien |
|-----------|------|
| Terraform Documentation | [developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform/docs) |
| Terraform Registry | [registry.terraform.io](https://registry.terraform.io/) |
| OpenTofu (alternative OSS) | [opentofu.org](https://opentofu.org/) |
| Warren Provider (Denv-r) | [registry.terraform.io/providers/WarrenCloudPlatform/warren](https://registry.terraform.io/providers/WarrenCloudPlatform/warren/) |

---

## 🤔 Questions de réflexion

1. Pourquoi ne pas simplement utiliser des scripts Bash pour créer l'infrastructure ?
2. Quels sont les risques de stocker le state localement dans une équipe ?
3. Comment Terraform gère-t-il les ressources créées manuellement (hors Terraform) ?
