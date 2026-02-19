# ☁️ Applications Cloud Native, Microservices & CNCF

> *"Cloud Native, c'est plus qu'héberger dans le cloud."*

## 🎯 Objectifs pédagogiques
- Comprendre ce qu'est une application Cloud Native
- Distinguer monolithique vs microservices
- Connaître les concepts Stateless/Stateful et l'écosystème CNCF

---

## 📖 Qu'est-ce qu'une application Cloud Native ?

### Définition CNCF

> *"Les technologies Cloud Native permettent aux organisations de construire et d'exécuter des applications scalables dans des environnements modernes et dynamiques comme les clouds publics, privés et hybrides."*
> — Cloud Native Computing Foundation

### Les 4 piliers

```
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION CLOUD NATIVE                      │
├────────────────┬────────────────┬────────────┬──────────────────┤
│  CONTENEURS    │  MICROSERVICES │   CI/CD    │  ORCHESTRATION   │
│                │                │            │                  │
│  Docker,       │  Services      │  Pipelines │  Kubernetes,     │
│  Podman        │  indépendants  │  auto      │  Nomad           │
│  Images OCI    │  API REST      │  GitOps    │  Scaling auto    │
└────────────────┴────────────────┴────────────┴──────────────────┘
```

### Cloud Native vs Traditionnel

| Aspect | Application traditionnelle | Application Cloud Native |
|--------|---------------------------|--------------------------|
| **Architecture** | Monolithique | Microservices |
| **Déploiement** | Serveur dédié, tous les 3 mois | Conteneurs, plusieurs fois/jour |
| **Scaling** | Vertical (plus gros serveur) | Horizontal (plus de conteneurs) |
| **État** | Stateful (session serveur) | Stateless (état externalisé) |
| **Résilience** | Redémarrage complet | Auto-healing, rolling updates |
| **Configuration** | Fichiers sur disque | Variables d'environnement, ConfigMaps |

---

## 🏗️ Monolithique vs Microservices

### Architecture monolithique

```
┌─────────────────────────────────────────────────────┐
│                  APPLICATION MONOLITHIQUE             │
│  ┌────────────────────────────────────────────────┐  │
│  │  UI + Business Logic + Data Access + Auth      │  │
│  │                                                │  │
│  │  Tout dans un seul processus                   │  │
│  │  Un seul déploiement                           │  │
│  │  Une seule base de données                     │  │
│  └────────────────────────────────────────────────┘  │
│                        │                             │
│                   ┌────▼─────┐                       │
│                   │    DB    │                       │
│                   └──────────┘                       │
└─────────────────────────────────────────────────────┘
```

### Architecture microservices

```
┌─────────────────────────────────────────────────────┐
│                    MICROSERVICES                      │
│                                                      │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐         │
│  │  UI  │──►│ Auth │   │ User │   │ Pay  │         │
│  │      │   │      │   │      │   │ ment │         │
│  └──────┘   └──┬───┘   └──┬───┘   └──┬───┘         │
│               │          │          │               │
│          ┌────▼──┐  ┌────▼──┐  ┌────▼──┐            │
│          │ DB 1  │  │ DB 2  │  │ DB 3  │            │
│          └───────┘  └───────┘  └───────┘            │
└─────────────────────────────────────────────────────┘
```

### Comparaison

| Aspect | Monolithique | Microservices |
|--------|-------------|---------------|
| **Complexité initiale** | ✅ Simple | ❌ Plus complexe |
| **Déploiement** | ❌ Tout ou rien | ✅ Service par service |
| **Scaling** | ❌ Tout l'app | ✅ Par service |
| **Technologie** | ❌ Unique | ✅ Mix possible (polyglot) |
| **Équipe** | ✅ Petite équipe | ✅ Grandes équipes |
| **Debugging** | ✅ Stack trace | ❌ Tracing distribué |
| **Latence** | ✅ Appels internes | ❌ Appels réseau |

> [!NOTE]
> Le monolithique n'est **pas mauvais**. Pour une petite application ou une petite équipe, c'est souvent le meilleur choix. Les microservices ajoutent de la complexité qui n'est justifiée qu'à une certaine échelle.

---

## 📡 Communication entre microservices

| Pattern | Description | Quand l'utiliser |
|---------|-------------|-----------------|
| **REST API** | HTTP/JSON synchrone | CRUD simple, requête-réponse |
| **gRPC** | Binaire, rapide, typé (protobuf) | Communication interne haute performance |
| **Message Broker** | Asynchrone (files de messages) | Découplage, événements, charge |
| **Service Mesh** | Proxy side-car (Envoy) | mTLS, observabilité, routage avancé |

### Outils courants

| Outil | Type | Usage |
|-------|------|-------|
| **RabbitMQ** | Message broker | Files de messages |
| **Apache Kafka** | Event streaming | Logs, événements temps réel |
| **Envoy / Istio** | Service mesh | mTLS, load balancing, tracing |
| **NGINX / Traefik** | API Gateway | Routage, TLS termination |

---

## 🔄 Stateless vs Stateful

### Définitions

| Type | Description | Exemple |
|------|-------------|---------|
| **Stateless** | Aucun état conservé entre les requêtes | API REST, serveur web |
| **Stateful** | État conservé entre les requêtes | Base de données, sessions |

### Pourquoi c'est important pour le Cloud ?

```
┌─────────────────────────────────────────────────────┐
│                    STATELESS                         │
│                                                      │
│  Requête 1 ──► Instance A    (pas de mémoire)       │
│  Requête 2 ──► Instance B    (pas de mémoire)       │
│  Requête 3 ──► Instance C    (pas de mémoire)       │
│                                                      │
│  ✅ Scaling facile : ajouter/supprimer des instances │
│  ✅ Pas de perte de données si crash                 │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                    STATEFUL                          │
│                                                      │
│  Requête 1 ──► Instance A    (écrit en mémoire)     │
│  Requête 2 ──► Instance A    (doit lire la mémoire) │
│                                                      │
│  ⚠️ Scaling complexe : sticky sessions, réplication │
│  ⚠️ Perte de données si crash sans persistance      │
└─────────────────────────────────────────────────────┘
```

### Stratégies Cloud Native

| Stratégie | Description |
|-----------|-------------|
| **Externaliser l'état** | Stocker sessions dans Redis, DB dans service managé |
| **PersistentVolumes (K8s)** | Stockage persistant attaché aux pods Stateful |
| **StatefulSets (K8s)** | Garantit identité stable et stockage persistant |
| **12-Factor App** | Processus stateless, config externalisée |

---

## 🌐 Écosystème CNCF

### Qu'est-ce que la CNCF ?

| Aspect | Détail |
|--------|--------|
| **Nom** | Cloud Native Computing Foundation |
| **Créée** | 2015, au sein de la Linux Foundation |
| **Mission** | Promouvoir les technologies Cloud Native open source |
| **Membres** | Google, AWS, Microsoft, Red Hat, IBM... |
| **Projets** | 180+ projets open source |

### Niveaux de maturité

| Niveau | Signification | Exemples |
|--------|---------------|----------|
| **Graduated** | Production-ready, adoption massive | Kubernetes, Prometheus, Envoy, Flux, Argo |
| **Incubating** | En cours de maturation | Falco, Dapr, OpenTelemetry |
| **Sandbox** | Expérimental, early stage | Projets innovants et émergents |

### Projets clés par catégorie

| Catégorie | Projet | Rôle |
|-----------|--------|------|
| **Orchestration** | Kubernetes | Orchestration de conteneurs |
| **Monitoring** | Prometheus | Collecte de métriques |
| **Visualisation** | Grafana* | Dashboards et alertes |
| **Logs** | Fluentd | Collecte et routage de logs |
| **Tracing** | Jaeger | Tracing distribué |
| **Service Mesh** | Istio / Linkerd | mTLS, routage, observabilité |
| **Proxy** | Envoy | Load balancing L7 |
| **Package Manager** | Helm | Packages Kubernetes |
| **GitOps** | Flux CD / Argo CD | Déploiement pull-based |
| **Sécurité runtime** | Falco | Détection d'anomalies |
| **Observabilité** | OpenTelemetry | Standard unifié metrics/logs/traces |

*Grafana n'est pas un projet CNCF mais fait partie de l'écosystème.

### Certifications CNCF

| Certification | Public | Focus |
|---------------|--------|-------|
| **CKA** | Administrateurs | Gérer un cluster K8s |
| **CKAD** | Développeurs | Déployer des apps sur K8s |
| **CKS** | Sécurité | Sécuriser un cluster K8s |

---

## 🔒 Sécurité Cloud Native

### Niveaux de sécurité

```
┌─────────────────────────────────────────────────────┐
│                 SÉCURITÉ MULTICOUCHE                  │
├─────────────────────────────────────────────────────┤
│  4. APPLICATION  │ Auth, secrets, OWASP              │
├──────────────────┼──────────────────────────────────┤
│  3. ORCHESTRATION│ RBAC, Network Policies, PSP       │
├──────────────────┼──────────────────────────────────┤
│  2. CONTENEUR    │ Image scan, non-root, limits      │
├──────────────────┼──────────────────────────────────┤
│  1. INFRA        │ Firewall, SSH, mTLS               │
└─────────────────────────────────────────────────────┘
```

### Types d'analyse sécurité

| Type | Approche | Quand | Exemple |
|------|----------|-------|---------|
| **SAST** | Code source (white box) | Pendant le dev | SonarQube, Semgrep |
| **DAST** | App en exécution (black box) | Après déploiement | OWASP ZAP |
| **SCA** | Dépendances tierces | Pendant le dev | Snyk, Dependabot |
| **Image Scan** | Images conteneur | Avant déploiement | Trivy, Clair |

---

## ❓ Pourquoi c'est important en 2026 ?

> [!IMPORTANT]
> Les applications Cloud Native sont le **standard de l'industrie** :
> - Architecture dominante pour les nouvelles applications
> - L'écosystème CNCF est le socle de l'infrastructure moderne
> - Comprendre Stateless/Stateful est essentiel pour le design d'applications scalables
> - La CNCF définit les standards et certifications de référence

---

## 📚 Sources officielles

| Ressource | Lien |
|-----------|------|
| CNCF Definition | [github.com/cncf/toc](https://github.com/cncf/toc/blob/main/DEFINITION.md) |
| CNCF Landscape | [landscape.cncf.io](https://landscape.cncf.io/) |
| 12-Factor App | [12factor.net](https://12factor.net/) |
| Microservices Patterns | Chris Richardson |
| Cloud Native Patterns | Cornelia Davis |
| CNCF Security Whitepaper | [cncf.io](https://www.cncf.io/reports/cloud-native-security-whitepaper/) |

---

## 🤔 Questions de réflexion

1. Quand est-il préférable de rester avec un monolithique plutôt que des microservices ?
2. Pourquoi le design Stateless est-il privilégié dans le Cloud ?
3. Comment la CNCF influence-t-elle les choix technologiques des entreprises ?
