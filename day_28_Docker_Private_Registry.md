# **JOUR 28 - REGISTRY PRIVÉ & AUTOMATISATION** 🔐🤖

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture des Registries Privés**
- **Docker Registry** → Léger, standard OCI, déploiement simple
- **Harbor** → Entreprise avec UI, RBAC, scanning vulnérabilités
- **Registry vs Repository** → Registry héberge repositories, repositories contiennent images

### **🔐 Couches de Sécurité**
| Couche | Protection | Implémentation |
|--------|------------|----------------|
| **Réseau** | HTTPS/TLS | Certificats valides, pare-feu |
| **Authentification** | Contrôle d'accès | Basic Auth, JWT, LDAP, OAuth2 |
| **Autorisation** | Niveaux permissions | RBAC (Lecture/Écriture/Admin) |
| **Scanning** | Détection vulnérabilités | Trivy, Clair, Grype |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Opérations Registry Docker**

| Commande                      | FR                        | EN                         | Usage                                        |
|-------------------------------|---------------------------|----------------------------|----------------------------------------------|
| `docker run registry:2`       | Lancer registry           | **Run registry**           | `docker run -d -p 5000:5000 registry:2`      |
| `docker login localhost:5000` | Login registry local      | **Login local registry**   | `docker login localhost:5000`                |
| `docker tag`                  | Tag pour registry local   | **Tag for local registry** | `docker tag alpine localhost:5000/my-alpine` |
| `curl registry`               | Vérifier registry         | **Check registry**         | `curl http://localhost:5000/v2/_catalog`     |

### **🔄 Gestion Cycle de Vie Images**

| Commande                           | FR                           | EN                                | Usage                                         |
|------------------------------------|------------------------------|-----------------------------------|-----------------------------------------------|
| `docker push localhost:5000/image` | Pousser vers registry privé  | **Push to private registry**      | `docker push localhost:5000/myapp:latest`     |
| `docker pull localhost:5000/image` | Pull depuis registry privé   | **Pull from private registry**    | `docker pull localhost:5000/myapp:latest`     |
| `docker image prune`               | Nettoyage images             | **Cleanup images**                | `docker image prune -f --filter "until=24h"`  |

---

## **📝 STRATÉGIES AVANCÉES**

### **Dockerfile Multi-Stage pour CI/CD**
```dockerfile
# Stage développement
FROM node:16-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Stage production  
FROM node:16-alpine AS production
RUN addgroup -g 1001 -S nodegroup && adduser -S nodeuser -u 1001
WORKDIR /app
COPY --from=development /app/dist ./dist
COPY --chown=nodeuser:nodegroup package*.json ./
RUN npm ci --only=production
USER nodeuser
CMD ["node", "dist/main.js"]

# Sélection image finale
ARG ENVIRONMENT=development
FROM ${ENVIRONMENT} AS final
```

### **Script Pipeline CI/CD Automatisé**
```bash
#!/bin/bash
# build-push-deploy.sh
set -e

REGISTRY="localhost:5000"
IMAGE_NAME="myapp"
VERSION=$(git describe --tags --always --dirty 2>/dev/null || echo "v0.0.0-$(git rev-parse --short HEAD)")

echo "🚀 Building $IMAGE_NAME:$VERSION"
docker build --tag "$REGISTRY/$IMAGE_NAME:$VERSION" --tag "$REGISTRY/$IMAGE_NAME:latest" .

echo "📤 Pushing to $REGISTRY"
docker push "$REGISTRY/$IMAGE_NAME:$VERSION"
docker push "$REGISTRY/$IMAGE_NAME:latest"

echo "✅ Déploiement terminé!"
```

---

## **🚀 DÉPLOIEMENT REGISTRY PRIVÉ**

### **Configuration Docker Compose**
```yaml
# docker-compose.registry.yml
version: '3.8'
services:
  registry:
    image: registry:2
    container_name: private-registry
    ports:
      - "5000:5000"
    volumes:
      - ./registry-data:/var/lib/registry
    environment:
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      
  registry-ui:
    image: joxit/docker-registry-ui:static
    container_name: registry-ui
    ports:
      - "8080:80"
    environment:
      REGISTRY_URL: http://registry:5000
      DELETE_IMAGES: "true"
```

### **Fichier Configuration Registry**
```yaml
# registry-config.yml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Comparaison Registry Public vs Privé**
```bash
# Docker Hub (Public)
✅ Pas d'infrastructure nécessaire
✅ Grande communauté
✅ Builds automatisés
❌ Limitations rate limiting
❌ Moins de contrôle

# Registry Privé
✅ Contrôle total
✅ Pas de limites de rate
✅ Sécurité renforcée
✅ Politiques personnalisées
❌ Maintenance nécessaire
❌ Gestion stockage requise
```

### **Conventions de Noms (CRITIQUE !)**
```bash
# ✅ BON - minuscules, pas d'espaces
localhost:5000/myapp:latest
localhost:5000/backend-api:v1.2.3
localhost:5000/frontend-web:production

# ❌ MAUVAIS - échouera
localhost:5000/MyApp:latest      # Majuscule
localhost:5000/my app:latest     # Espace
localhost:5000/my.app:latest     # Caractères spéciaux
```

### **Configuration Authentification**
```bash
# Créer fichier mot de passe
docker run --entrypoint htpasswd registry:2 -Bbn admin secretpassword > auth/htpasswd

# Lancer avec authentification
docker run -d \
  --name registry-auth \
  -p 5001:5000 \
  -v ./auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2
```

### **Gestion des Permissions**
```bash
# Ajouter utilisateur au groupe docker (Linux)
sudo usermod -aG docker $USER
newgrp docker  # Appliquer sans déconnexion

# Vérifier permissions
docker ps  # Devrait fonctionner sans sudo
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Déploiement Registry Local**
```bash
# Démarrer registry basique
docker run -d --name registry -p 5000:5000 -v $(pwd)/data:/var/lib/registry registry:2

# Tester registry
curl http://localhost:5000/v2/_catalog
# → {"repositories":[]}

# Pousser image test
docker pull alpine
docker tag alpine:latest localhost:5000/my-alpine:latest
docker push localhost:5000/my-alpine:latest

# Vérifier
curl http://localhost:5000/v2/_catalog
# → {"repositories":["my-alpine"]}
```

### **2. Script CI/CD Automatisé Complet**
```bash
#!/bin/bash
# auto-deploy.sh
set -e

REGISTRY="localhost:5000"
IMAGE="takkino-app"
ENV=${1:-development}

# Versioning
VERSION="v1.0.0-$(git rev-parse --short HEAD 2>/dev/null || date +%Y%m%d-%H%M%S)"

echo "🔨 Building $IMAGE:$VERSION for $ENV"
docker build . -t "$REGISTRY/$IMAGE:$VERSION" --build-arg ENVIRONMENT="$ENV"

echo "📤 Pushing..."
docker push "$REGISTRY/$IMAGE:$VERSION"
docker push "$REGISTRY/$IMAGE:latest"

echo "✅ Déployé vers $REGISTRY"
curl -s "$REGISTRY/v2/_catalog"
```

### **3. Registry Docker avec Interface Web**
```bash
# Démarrer registry avec UI
docker-compose -f docker-compose.registry.yml up -d

# Accéder UI sur http://localhost:8080
# Accéder API sur http://localhost:5000/v2/_catalog

# Tester workflow
docker build -t localhost:5000/testapp:ui-test .
docker push localhost:5000/testapp:ui-test

# Vérifier dans UI (rafraîchir navigateur)
```

### **4. Automatisation Nettoyage Images**
```bash
#!/bin/bash
# cleanup.sh
echo "🧹 Nettoyage système Docker..."

# Supprimer images inutilisées
docker image prune -f

# Supprimer containers arrêtés
docker container prune -f

# Supprimer volumes inutilisés
docker volume prune -f

# Supprimer réseaux inutilisés
docker network prune -f

echo "✅ Nettoyage terminé!"
```

### **5. Backup Données Registry**
```bash
#!/bin/bash
# backup-registry.sh
BACKUP_DIR="/backups/registry"
DATE=$(date +%Y%m%d_%H%M%S)

echo "💾 Backup données registry..."

# Arrêter registry pour backup cohérent
docker-compose -f docker-compose.registry.yml stop registry

# Backup répertoire données
tar czf "$BACKUP_DIR/registry_$DATE.tar.gz" ./registry-data/

# Redémarrer registry
docker-compose -f docker-compose.registry.yml start registry

echo "✅ Backup sauvegardé: $BACKUP_DIR/registry_$DATE.tar.gz"
```

---

## **🎯 BONNES PRATIQUES PRODUCTION**

### **Checklist Production**
- ✅ **HTTPS avec certificats valides** (pas HTTP en production)
- ✅ **Authentification activée** (pas de push anonyme)
- ✅ **Backups réguliers** des données registry
- ✅ **Monitoring** (espace disque, mémoire, requêtes)
- ✅ **Garbage collection** configuré
- ✅ **RBAC** pour permissions équipe
- ✅ **Scanning vulnérabilités** intégré

### **Best Practices Sécurité**
```yaml
# Configuration registry sécurisée
version: 0.1
storage:
  filesystem:
    rootdirectory: /var/lib/registry
  delete:
    enabled: true
http:
  addr: :5000
  tls:
    certificate: /certs/domain.crt
    key: /certs/domain.key
auth:
  htpasswd:
    realm: basic-realm
    path: /auth/htpasswd
```

### **Convention Nommage Images**
```bash
# Structure organisationnelle
{registry}/{projet}/{composant}:{version}-{environnement}

# Exemples
registry.company.com/backend/api:v1.2.3-production
registry.company.com/frontend/web:latest-staging
registry.company.com/tools/backup:nightly
```

### **Exemple Intégration CI/CD**
```yaml
# Workflow GitHub Actions
name: Build and Push
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Docker image
        run: docker build -t localhost:5000/${{ github.event.repository.name }}:${{ github.sha }} .
      - name: Push to registry
        run: |
          docker push localhost:5000/${{ github.event.repository.name }}:${{ github.sha }}
          docker push localhost:5000/${{ github.event.repository.name }}:latest
```

---

## **📈 PROGRESSION JOUR 28**

### **✅ Compétences Acquises :**
- **Déploiement Registry Privé** → Docker Registry avec/sans authentification
- **Intégration Interface Web** → Interface web pour gestion images
- **Pipeline CI/CD Automatisé** → Automatisation build, scan, push, deploy
- **Implémentation Sécurité** → HTTPS, authentification, RBAC basics
- **Préparation Production** → Backup, monitoring, stratégies nettoyage

### **🎯 Principaux Enseignements :**
1. **Nommage critique** → Toujours minuscules, pas d'espaces noms images
2. **Sécurité d'abord** → Jamais registry production sans authentification
3. **Tout automatiser** → Pipeline CI/CD réduit erreurs humaines
4. **Monitorer stockage** → Données registry croissent vite, implémenter nettoyage
5. **Backup régulier** → Données registry infrastructure critique

### **🔗 Architecture Implémentée :**
```
Écosystème Registry Privé
├── Serveur Registry (localhost:5000)
├── Interface Web (localhost:8080)
├── Pipeline CI/CD Automatisé
├── Système Backup
└── Monitoring & Nettoyage
```

### **🚀 Prochaines Étapes :**
- **Intégration Kubernetes** → Utiliser registry privé comme source images
- **Scanning Avancé** → Implémenter Trivy/Grype détection vulnérabilités
- **Sync Multi-registry** → Réplication entre dev et prod
- **GitHub Actions/GitLab CI** → Automatisation complète pipeline CI/CD

---

**📊 Progress: `Jour 28 / 100 ✅`**

**#RegistryPrivé #DockerRegistry #CI_CD #Automatisation #DevOps**
