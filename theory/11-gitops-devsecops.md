# 🔄 De DevOps à GitOps & DevSecOps

> *"Git est la source de vérité pour tout. La sécurité aussi."*

## 🎯 Objectifs pédagogiques
- Comprendre l'évolution DevOps → GitOps
- Maîtriser les principes du GitOps
- Connaître les outils : Flux CD, Argo CD

---

## 📅 Évolution

### DevOps classique : Push-based

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Dev      │────►│    CI/CD    │────►│   Cluster   │
│  git push   │     │   Jenkins   │     │   K8s/VMs   │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                           ▼
                    "Je pousse vers
                     la production"
```

**Problèmes du modèle push :**
- CI/CD a des credentials pour accéder à la prod
- Difficile de savoir l'état réel vs désiré
- Pas d'auto-healing si quelqu'un modifie manuellement

### 2017 : Weaveworks invente GitOps

**Alexis Richardson** (CEO Weaveworks) crée le terme **GitOps** pour décrire comment ils gèrent Kubernetes.

> *"GitOps is an operating model for cloud native applications, using Git as the source of truth."*

### GitOps : Pull-based

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Dev      │────►│    Git      │◄────│  GitOps     │
│  git push   │     │   Repo      │     │  Operator   │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │   Cluster   │
                                        │   K8s/VMs   │
                                        └─────────────┘
                                               │
                                               ▼
                                    "Je me réconcilie
                                     avec ce que dit Git"
```

---

## 🔑 Les 4 principes GitOps

| # | Principe | Description |
|---|----------|-------------|
| 1 | **Déclaratif** | Tout est décrit de manière déclarative (YAML, HCL...) |
| 2 | **Versionné dans Git** | Git = source of truth unique |
| 3 | **Appliqué automatiquement** | L'agent GitOps réconcilie automatiquement |
| 4 | **Réconciliation continue** | L'état réel converge vers l'état désiré |

### Avantages

| Avantage | Explication |
|----------|-------------|
| **Auditabilité** | Tout changement = commit Git avec auteur, date, message |
| **Rollback facile** | `git revert` pour annuler un changement |
| **Sécurité** | L'agent pull depuis Git, pas de credentials CI vers prod |
| **Auto-healing** | Si quelqu'un modifie manuellement, l'agent corrige |

---

## 🛠️ Outils GitOps

### Flux CD

| Aspect | Détail |
|--------|--------|
| **Créé par** | Weaveworks (2016) |
| **Status** | CNCF Graduated Project |
| **Spécialité** | Kubernetes-native, modulaire |
| **Site** | [fluxcd.io](https://fluxcd.io) |

```yaml
# Exemple Flux: GitRepository
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
spec:
  interval: 1m
  url: https://github.com/org/my-app
  ref:
    branch: main
```

### Argo CD

| Aspect | Détail |
|--------|--------|
| **Créé par** | Intuit (2018) |
| **Status** | CNCF Graduated Project |
| **Spécialité** | UI riche, multi-cluster |
| **Site** | [argo-cd.readthedocs.io](https://argo-cd.readthedocs.io) |

```yaml
# Exemple Argo CD: Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  project: default
  source:
    repoURL: https://github.com/org/my-app
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

### Comparaison

| Critère | Flux CD | Argo CD |
|---------|---------|---------|
| **UI** | Minimale (extensions) | Riche, native |
| **Architecture** | Modulaire (toolkit) | Monolithique |
| **Multi-tenant** | Via namespaces | Application Projects |
| **Courbe apprentissage** | Modérée | Facile |

---

## 🔗 GitOps dans ce repo

Ce repository utilise des concepts GitOps :

```
git push tag ──► GitHub Actions ──► Build image ──► Push registry
                                                         │
                                                         ▼
                             Ansible déploie depuis le registry
```

> [!NOTE]
> C'est du **GitOps simplifié** : le déclencheur est Git, mais le déploiement reste push-based via Ansible.
> Un "vrai" GitOps utiliserait Flux ou Argo qui **tire** (pull) les changements.

---

## 🔒 DevSecOps : la sécurité intégrée

### Shift Left Security

```
┌─────────────────────────────────────────────────────────────────┐
│                    PIPELINE DEVSECOPS                             │
│                                                                  │
│  Code ──► Build ──► Test ──► Scan ──► Deploy ──► Monitor         │
│    │        │        │        │         │          │              │
│   SAST    Image    Tests   Trivy    Network     Falco            │
│   Lint    Build    Sécu    Snyk    Policies    Alertes           │
│                                    RBAC                          │
│                                                                  │
│  ◄──────────── Sécurité intégrée partout ──────────────►        │
└─────────────────────────────────────────────────────────────────┘
```

> [!IMPORTANT]
> DevSecOps ≠ "ajouter de la sécurité à la fin".
> C'est **intégrer la sécurité à chaque étape**, dès le développement.

---

### 🔍 Sécurisation des images conteneurs

#### Scan avec Trivy

```bash
# Scanner une image pour les vulnérabilités
trivy image python:3.11

# Scanner avec seuil critique (échoue si HIGH ou CRITICAL)
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:latest

# Scanner un Dockerfile
trivy config Dockerfile

# Scanner un filesystem
trivy fs --security-checks vuln,config .
```

#### Bonnes pratiques images

| Pratique | Pourquoi |
|----------|----------|
| **Images officielles** | Maintenues, scannées régulièrement |
| **Images minimales** (alpine, distroless) | Moins de paquets = moins de CVEs |
| **Pas de `latest`** | Tag versionné = reproductible |
| **Non-root** | `USER 1001` dans le Dockerfile |
| **Scan dans la CI** | Bloquer le déploiement si vulnérabilité critique |

---

### 🛡️ Sécurité à l'exécution : Falco

Falco détecte les **comportements anormaux** dans les conteneurs en temps réel.

```
┌─────────────────────────────────────────────────────┐
│                    FALCO                             │
│                                                      │
│  Événements système ──► Règles ──► Alertes           │
│  (syscalls)              YAML      Slack/PagerDuty   │
│                                                      │
│  Détecte :                                           │
│  - Shell dans un conteneur                           │
│  - Lecture de /etc/shadow                            │
│  - Connexion réseau inattendue                       │
│  - Modification de binaires système                  │
│  - Exécution de processus non autorisé               │
└─────────────────────────────────────────────────────┘
```

```bash
# Installation via Helm
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco
```

---

### 🌐 Kubernetes : Network Policies

Par défaut, tous les pods peuvent communiquer entre eux. Les Network Policies restreignent ce flux.

```yaml
# Exemple : seul le pod "frontend" peut accéder au pod "api"
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow-frontend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

```
┌─────────────────────────────────────────────────────┐
│                   SANS Network Policy                │
│                                                      │
│   frontend ◄──► api ◄──► database ◄──► monitoring   │
│   (tout le monde parle à tout le monde)              │
│                                                      │
│   ⚠️ Un conteneur compromis peut atteindre la DB     │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                   AVEC Network Policy                │
│                                                      │
│   frontend ──► api ──► database                      │
│                          ▲                           │
│                          │ (seul api peut accéder)   │
│   monitoring ──► api     ✗ frontend ──✗ database     │
│                                                      │
│   ✅ Principe du moindre privilège réseau             │
└─────────────────────────────────────────────────────┘
```

---

### 🔐 Kubernetes : RBAC

RBAC (Role-Based Access Control) contrôle **qui peut faire quoi** dans le cluster.

```yaml
# Rôle : permissions de lecture sur les pods
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

---
# Liaison : attacher le rôle à un utilisateur
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: production
  name: read-pods
subjects:
  - kind: User
    name: dev-alice
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

| Concept | Scope | Description |
|---------|-------|-------------|
| **Role** | Namespace | Permissions dans un namespace |
| **ClusterRole** | Cluster | Permissions cluster-wide |
| **RoleBinding** | Namespace | Lie un Role à un utilisateur |
| **ClusterRoleBinding** | Cluster | Lie un ClusterRole à un utilisateur |

---

### 📰 Cas réel : attaque Tesla K8s (2018)

> [!CAUTION]
> En 2018, des attaquants ont compromis le cluster Kubernetes de Tesla pour miner du crypto :
> - **Dashboard K8s exposé** sans authentification
> - **Pas de Network Policies** → accès libre entre pods
> - **Pas de RBAC** → accès admin complet
> - **Pas de monitoring** → détecté tardivement par un chercheur de RedLock
>
> **Leçon** : l'orchestrateur doit être sécurisé avec la même rigueur que l'application.

---

## ❓ Pourquoi c'est important en 2026 ?

> [!IMPORTANT]
> GitOps est devenu le **standard de facto** pour Kubernetes.
> DevSecOps est **non négociable** en production :
> - Attaques sur les supply chains (SolarWinds, Log4j)
> - Réglementations (NIS2, DORA) exigent la sécurité intégrée
> - Les conteneurs non sécurisés sont la première surface d'attaque

---

## 📚 Sources officielles

| Ressource | Lien |
|-----------|------|
| GitOps.tech (principes) | [gitops.tech](https://www.gitops.tech) |
| Flux CD Documentation | [fluxcd.io/docs](https://fluxcd.io/docs/) |
| Argo CD Documentation | [argo-cd.readthedocs.io](https://argo-cd.readthedocs.io) |
| OpenGitOps (CNCF) | [opengitops.dev](https://opengitops.dev) |
| Weaveworks Blog (origine) | [weave.works/blog](https://www.weave.works/technologies/gitops/) |

---

## 🤔 Questions de réflexion

1. Quelle est la différence entre CI/CD "classique" et GitOps ?
2. Pourquoi le modèle "pull" est-il plus sécurisé que le modèle "push" ?
3. GitOps est-il applicable sans Kubernetes ?
