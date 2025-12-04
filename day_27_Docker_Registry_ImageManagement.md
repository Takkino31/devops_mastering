# **DAY 27 - DOCKER REGISTRY & IMAGE MANAGEMENT** 🐳📦

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture des Registries**
- **Registries Publics** → Docker Hub, GitHub Container Registry
- **Registries Privés** → Docker Registry, Harbor, Nexus
- **Cloud Registries** → ECR (AWS), ACR (Azure), GCR (GCP)

### **🏷️ Stratégies de Tagging**
| Pattern        | Usage                        | Exemple           |
|----------------|------------------------------|-------------------|
| `:latest`      | Dernière version stable      | `app:latest`      |
| `:x.y.z`       | Version spécifique (semver)  | `app:1.2.3`       | 
| `:x.y`         | Version majeure.mineure      | `app:1.2`         |
| `:commit-hash` | Build spécifique             | `app:abc123`      |
| `:env-feature` | Environnement/fonctionnalité | `app:prod-api`    |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Docker Hub Operations**
| Commande          | FR                 | EN                       | Usage                         |
|-------------------|--------------------|--------------------------|-------------------------------|
| `docker login`    | Connexion registry | **LOGIN to registry**    | `docker login`                |
| `docker logout`   | Déconnexion        | **LOGOUT from registry** | `docker logout`               |
| `docker push`     | Pousser image      | **PUSH image**           | `docker push user/image:tag`  |
| `docker pull`     | Télécharger image  | **PULL image**           | `docker pull user/image:tag`  |

### **🏷️ Gestion des Tags**
| Commande              | FR                | EN                | Usage                              |
|-----------------------|-------------------|-------------------|------------------------------------|
| `docker tag`          | Créer tag         | **TAG image**     | `docker tag source:tag target:tag` |
| `docker images`       | Lister images     | **LIST images**   | `docker images --format "table"`   |
| `docker image prune`  | Nettoyer images   | **PRUNE images**  | `docker image prune -a`            |

---

## **📝 STRATÉGIES AVANCÉES**

### **Tagging Automatisé**
```bash
# Multi-tagging pour différents usages
docker tag mon-app:1.2.3 user/mon-app:1.2.3
docker tag mon-app:1.2.3 user/mon-app:1.2
docker tag mon-app:1.2.3 user/mon-app:1
docker tag mon-app:1.2.3 user/mon-app:latest
docker tag mon-app:1.2.3 user/mon-app:production
```

### **Workflow Complet**
```bash
# 1. Build
docker build -t mon-app:1.2.3 .

# 2. Tag pour Docker Hub
docker tag mon-app:1.2.3 username/mon-app:1.2.3
docker tag mon-app:1.2.3 username/mon-app:latest

# 3. Push
docker push username/mon-app:1.2.3
docker push username/mon-app:latest

# 4. Pull ailleurs
docker pull username/mon-app:latest
```

---

## **🚀 DOCKER HUB MANAGEMENT**

### **Repository Structure**
```
username/repository-name
├── 📦 Images
│   ├── :latest (stable)
│   ├── :1.2.3 (version specific)
│   ├── :1.2 (major.minor)
│   └── :feature-branch (pre-release)
├️── 🔧 Automated Builds
├️── ⚙️ Webhooks
└── 🔒 Settings & Permissions
```

### **Permissions Levels**
- **Public** → Accessible à tous
- **Private** → Équipe seulement
- **Organization** → Gestion par équipes

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Docker Hub vs Privé :**
```bash
# Docker Hub (Public)
✅ Gratuit images publiques
✅ Large communauté
✅ CI/CD intégré
❌ Limites pull images privées

# Registry Privé
✅ Contrôle total
✅ Pas de limites
✅ Sécurité renforcée
❌ Maintenance nécessaire
```

### **Tagging Stratégique :**
```bash
# ❌ MAUVAIS - Un seul tag
docker push mon-app:latest

# ✅ BON - Tags multiples
docker push mon-app:1.2.3
docker push mon-app:1.2
docker push mon-app:1
docker push mon-app:latest
docker push mon-app:production
```

### **Gestion Cycle de Vie :**
```bash
# 1. Build avec version
docker build -t mon-app:$VERSION .

# 2. Tag pour registry
docker tag mon-app:$VERSION registry/mon-app:$VERSION

# 3. Push
docker push registry/mon-app:$VERSION

# 4. Cleanup local
docker image prune -a --filter "until=24h"
```

### **Authentification :**
```bash
# Login explicite
docker login docker.io
docker login registry.example.com

# Logout
docker logout

# Vérifier login
docker info | grep Username
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Compte Docker Hub**
```bash
# Configuration initiale
docker login
# → Username: votre_nom
# → Password: [token d'accès]

# Vérification
docker info | grep Username
# → Username: votre_nom
```

### **2. Premier Push/Pull**
```bash
# Création image test
docker build -t username/hello-world:1.0 .

# Tagging
docker tag username/hello-world:1.0 username/hello-world:latest

# Push
docker push username/hello-world:1.0
docker push username/hello-world:latest

# Pull depuis autre machine
docker pull username/hello-world:latest
```

### **3. Stratégie Tagging Avancée**
```bash
#!/bin/bash
# auto-tag.sh
VERSION="1.2.3"
IMAGE="username/mon-app"

docker build -t $IMAGE:$VERSION .
docker tag $IMAGE:$VERSION $IMAGE:latest
docker tag $IMAGE:$VERSION $IMAGE:$(git rev-parse --short HEAD)

docker push $IMAGE:$VERSION
docker push $IMAGE:latest
```

### **4. Gestion Images Locales**
```bash
# Lister avec format
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Recherche spécifique
docker images | grep username

# Nettoyage
docker image prune -a --filter "until=24h"

# Suppression tag
docker rmi username/image:old-tag
```

### **5. Automated Workflow**
```bash
# Build selon environnement
if [ "$ENV" = "prod" ]; then
    TAG="production"
elif [ "$ENV" = "staging" ]; then
    TAG="staging"
else
    TAG="latest"
fi

docker build -t username/app:$TAG .
docker push username/app:$TAG
```

---

## **🎯 BEST PRACTICES TAG MANAGEMENT**

### **Checklist Tagging :**
- ✅ **Semantic Versioning** (MAJOR.MINOR.PATCH)
- ✅ **Environment tags** (prod, staging, dev)
- ✅ **Commit hash** pour traçabilité
- ✅ **Latest tag** = version stable
- ✅ **Feature tags** pour développement

### **Conventions Recommandées :**
```bash
# Production
app:1.2.3
app:production
app:latest

# Staging
app:1.2.3-staging
app:staging

# Development
app:1.2.3-dev
app:feature-login
app:abc123 (commit hash)
```

### **Sécurité Docker Hub :**
```yaml
# .docker/config.json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "base64_encoded_credentials"
    }
  },
  "HttpHeaders": {
    "User-Agent": "Docker-Client"
  }
}
```

---

## **📈 PROGRESSION DAY 27**

**✅ Compétences Acquises :**
- Maîtrise complète de Docker Hub (push/pull/tag)
- Stratégies avancées de tagging (semver, env, commit)
- Gestion du cycle de vie des images
- Automatisation des workflows de build
- Best practices de gestion de registry

**🎯 Mentalité DevOps :**
> Mes images ne sont plus des artefacts locaux  
> Elles sont des composants versionnés et distribués globalement

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 27 / 100 ✅`**

**#DockerHub #ContainerRegistry #ImageManagement #DevOps #CI_CD #SemanticVersioning**

---
