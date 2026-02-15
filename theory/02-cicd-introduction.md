# 🔄 Introduction au CI/CD

> *"Livrer souvent, livrer tôt, automatiser tout."*

## 🎯 Objectifs pédagogiques
- Comprendre les concepts CI (Continuous Integration) et CD (Continuous Delivery/Deployment)
- Identifier les bénéfices de l'automatisation des pipelines
- Connaître les outils principaux : GitHub Actions, GitLab CI, Jenkins

---

## 📖 Définitions

### Continuous Integration (CI)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Dev 1    │────►│             │     │             │
│  git push   │     │    Git      │────►│    Build    │
├─────────────┤     │    Repo     │     │    Test     │
│    Dev 2    │────►│             │     │             │
│  git push   │     └─────────────┘     └─────────────┘
└─────────────┘                               │
                                              ▼
                                    Feedback rapide ✅/❌
```

**Principes :**
- Chaque développeur intègre son code **plusieurs fois par jour**
- Chaque intégration déclenche **build + tests automatiques**
- Les problèmes sont détectés **immédiatement**

### Continuous Delivery (CD)

Le code est **toujours déployable** en production :
- Après CI, le code passe par des environnements (staging, QA)
- Le déploiement en prod est **manuel** mais préparé

### Continuous Deployment

Étape supplémentaire : le déploiement en production est **automatique** :
- Chaque commit qui passe les tests va en production
- Nécessite une confiance élevée dans les tests

---

## 📊 Comparaison

| Aspect | Sans CI/CD | Avec CI/CD |
|--------|-----------|------------|
| **Fréquence release** | Mensuelle/trimestrielle | Quotidienne/continue |
| **Détection bugs** | Tard (intégration) | Immédiat (commit) |
| **Stress déploiement** | Élevé ("Big Bang") | Faible (routine) |
| **Feedback** | Lent | Rapide |
| **Qualité** | Variable | Constante |

---

## 🛠️ Anatomie d'un pipeline CI/CD

```yaml
# Exemple GitHub Actions
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4      # 1. Récupérer le code
      
      - name: Setup Python
        uses: actions/setup-python@v5  # 2. Préparer l'environnement
        with:
          python-version: '3.11'
      
      - name: Install dependencies     # 3. Installer dépendances
        run: pip install -r requirements.txt
      
      - name: Run tests               # 4. Exécuter tests
        run: pytest
      
      - name: Build                   # 5. Construire l'artifact
        run: docker build -t myapp .
```

### Étapes typiques

| Étape | Description | Outils |
|-------|-------------|--------|
| **Checkout** | Récupérer le code source | Git |
| **Build** | Compiler, installer dépendances | npm, pip, maven |
| **Test** | Tests unitaires, intégration | pytest, jest, JUnit |
| **Lint** | Vérifier qualité code | eslint, flake8, black |
| **Security** | Scanner vulnérabilités | Trivy, Snyk, Dependabot |
| **Deploy** | Déployer sur environnement | Ansible, Terraform, kubectl |

---

## 🏆 Outils populaires

| Outil | Type | Avantages |
|-------|------|-----------|
| **GitHub Actions** | SaaS (intégré GitHub) | Simple, gratuit pour open source |
| **GitLab CI** | SaaS/Self-hosted | Complet, intégré GitLab |
| **Jenkins** | Self-hosted | Flexible, plugins nombreux |
| **CircleCI** | SaaS | Rapide, bonne UX |
| **Azure DevOps** | SaaS | Intégration Microsoft |

> [!TIP]
> Pour cette formation, on utilise **GitHub Actions** car :
> - Gratuit pour les repos publics
> - Intégré directement dans GitHub
> - Syntaxe YAML simple

---

## 💡 GitHub Actions : Tips & bonnes pratiques

### Variables globales

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}  # org/repo automatique

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build image
        run: docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
```

> [!TIP]
> Utilisez `${{ github.repository }}` pour nommer automatiquement vos images.
> Ça évite de hardcoder le nom du repo et rend le workflow réutilisable via fork.

### GitHub Container Registry (GHCR)

```yaml
jobs:
  push:
    runs-on: ubuntu-latest
    permissions:
      packages: write    # Nécessaire pour pousser sur GHCR
    steps:
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Push image
        run: docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

> [!NOTE]
> GHCR est **intégré à GitHub**, gratuit pour les repos publics.
> Pas besoin de Docker Hub ni de créer de secrets supplémentaires.

### Dependabot : sécurité automatique

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pip"         # Python
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"  # Actions elles-mêmes
    directory: "/"
    schedule:
      interval: "weekly"
```

### Bonnes pratiques résumées

| Pratique | Pourquoi |
|----------|----------|
| **`github.repository`** | Pas de duplication, fonctionne avec les forks |
| **GHCR** | Registry intégré, authentification via `GITHUB_TOKEN` |
| **Dependabot** | Mises à jour sécurité automatiques |
| **`permissions`** explicites | Principe du moindre privilège |
| **`GITHUB_TOKEN`** (pas de PAT) | Token temporaire, scope limité au workflow |
| **Cache des dépendances** | Builds plus rapides (`actions/cache`) |

---

## ❓ Pourquoi c'est important en 2026 ?

> [!IMPORTANT]
> Le CI/CD est **obligatoire** dans toute équipe moderne :
> - Réduit le "it works on my machine"
> - Permet des releases fréquentes et sûres
> - Base de tout workflow DevOps/GitOps

---

## 📚 Sources officielles

| Ressource | Lien |
|-----------|------|
| GitHub Actions Docs | [docs.github.com/actions](https://docs.github.com/en/actions) |
| GitLab CI Docs | [docs.gitlab.com/ee/ci](https://docs.gitlab.com/ee/ci/) |
| Continuous Delivery (livre) | Jez Humble, David Farley |
| Martin Fowler - CI | [martinfowler.com/articles/continuousIntegration.html](https://martinfowler.com/articles/continuousIntegration.html) |

---

## 🤔 Questions de réflexion

1. Quelle est la différence entre Continuous Delivery et Continuous Deployment ?
2. Pourquoi les tests automatisés sont-ils essentiels au CI/CD ?
3. Quels sont les risques d'un pipeline CI/CD mal configuré ?
