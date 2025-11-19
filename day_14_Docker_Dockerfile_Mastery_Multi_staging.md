# **DAY 14 - DOCKER MULTI-STAGE BUILDS & OPTIMISATION AVANCÉE** 🏗️⚡

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Multi-Stage**
- **Séparation build/runtime** → Environnements distincts
- **Images minimalistes** → Réduction drastique de la taille
- **Sécurité renforcée** → Users non-root en production

### **📊 Réduction de Taille**
- **Node.js** : 1.2 GB → 120 MB (**-90%**)
- **Go** : 800 MB → 15 MB (**-98%**)
- **Python** : 900 MB → 200 MB (**-78%**)

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔨 Multi-Stage Builds**
|---------------------------|-------------------|-----------------------|-----------------------------------|
| Commande                  | FR                | EN                    | Usage                             |
|---------------------------|-------------------|-----------------------|-----------------------------------|
| `docker build -t app .`   | Build multi-stage | **Multi-stage BUILD** | `docker build -t app-optimise .`  |
| `docker images`           | Comparer tailles  | **COMPARE sizes**     | `docker images \| grep app`       |
| `docker history image`    | Analyser layers   | **ANALYZE layers**    | `docker history app-optimise`     |
|---------------------------|-------------------|-----------------------|-----------------------------------|


### **📈 Analyse Performance**
|-----------------------|-----------------------|-------------------|-------------------------------|
| Commande              | FR                    | EN                | Usage                         |
|-----------------------|-----------------------|-------------------|-------------------------------|
| `docker image ls`     | Lister images         | **LIST images**   | `docker image ls`             |
| `docker system df`    | Espace disque         | **DISK usage**    | `docker system df`            |
| `time docker build`   | Mesurer temps build   | **BUILD time**    | `time docker build -t app .`  |
|-----------------------|-----------------------|-------------------|-------------------------------|

---

## **📝 ARCHITECTURE MULTI-STAGE**

### **Structure de Base**
```dockerfile
# STAGE 1: Environnement de build
FROM runtime:tag AS builder
WORKDIR /app
COPY . .
RUN command-de-build

# STAGE 2: Environnement de production  
FROM runtime:tag AS production
WORKDIR /app
COPY --from=builder /app/artefacts .
CMD ["start-command"]
```

### **Instructions Multi-Stage**
|-------------------|---------------------------|-------------------------------|-----------------------|
| Instruction       | Usage                     | Exemple                       | Effet                 |
|-------------------|---------------------------|-------------------------------|-----------------------|
| `FROM AS alias`   | Définir stage             | `FROM node AS builder`        | Crée un stage nommé   |
| `COPY --from`     | Copier entre stages       | `COPY --from=builder /app`    | Transfert d'artefacts |
| `--chown`         | Changement propriétaire   | `--chown=user:group`          | Sécurité              |
|-------------------|---------------------------|-------------------------------|-----------------------|

---

## **🚀 STRATÉGIES D'OPTIMISATION**

### **Réduction de Taille**
|-----------------------|---------------|-------------------------------|
| Technique             | Impact        | Usage                         |
|-----------------------|---------------|-------------------------------|
| **Images Alpine**     | -70% taille   | `node:alpine` vs `node`       |
| **Multi-stage**       | -80% taille   | Build séparé du runtime       |
| **Nettoyage cache**   | -10% taille   | `npm cache clean --force`     |
| **Suppression docs**  | -5% taille    | `find . -name "*.md" -delete` |
|-----------------------|---------------|-------------------------------|

### **Sécurité Renforcée**
|-----------------------|---------------------------|---------------------|
| Pratique              | Avantage                  | Implementation      |
|-----------------------|---------------------------|---------------------|
| **Users non-root**    | Réduction risques         | `USER node`         |
| **Images signées**    | Intégrité                 | `docker trust sign` |
| **Minimal runtime**   | Surface attaque réduite   | `FROM scratch` (Go) |
|-----------------------|---------------------------|---------------------|

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Le Principe Multi-Stage :**
```dockerfile
# MAUVAIS - Tout dans une image
FROM node:18
WORKDIR /app
COPY . .
RUN npm install && npm run build
CMD ["node", "dist/app.js"]

# BON - Séparation build/runtime
FROM node:18 AS builder
COPY . .
RUN npm run build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/app.js"]
```

### **Optimisation Node.js :**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY src/ ./src/
USER node
CMD ["node", "src/index.js"]
```

### **Optimisation Go (Extrême) :**
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

FROM scratch
COPY --from=builder /app/main .
CMD ["./main"]
```

### **Sécurité avec Users :**
```dockerfile
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001
USER nextjs
# L'application tourne maintenant en non-root
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Multi-Stage Node.js Complet**
```dockerfile
# STAGE 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# STAGE 2: Production
FROM node:18-alpine AS runtime
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs src/ ./src/
USER nextjs
EXPOSE 3000
CMD ["node", "src/index.js"]
```

### **2. Application Go Ultra-Légère**
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage  
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

### **3. Analyse Comparative**
```bash
# Build et comparaison
docker build -t app-single -f Dockerfile.single .
docker build -t app-multi -f Dockerfile.multi .

# Résultats taille
docker images | grep app
# app-single   1.2GB
# app-multi    120MB

# Test fonctionnement
docker run -d -p 3000:3000 app-multi
curl http://localhost:3000
```

### **4. Validation Sécurité**
```bash
# Vérifier user dans conteneur
docker exec -it mon-conteneur whoami
# → nextjs (non root)

# Inspecter les layers
docker history app-multi
# → Moins de layers, plus optimisé
```

---

## **🎯 MÉTHODOLOGIE MULTI-STAGE**

### **Approche Systématique :**
```dockerfile
1. FROM runtime AS builder
2. COPY fichiers sources
3. RUN compilation/build
4. FROM runtime-minimal  
5. COPY --from=builder artefacts
6. USER non-root
7. CMD démarrage
```

### **Checklist Optimisation :**
- ✅ **Toujours** utiliser multi-stage pour les applications compilées
- ✅ **Toujours** choisir des images de base minimales (Alpine)
- ✅ **Toujours** exécuter en tant qu'utilisateur non-root
- ✅ **Toujours** nettoyer les caches et fichiers temporaires
- ✅ **Toujours** tester les images optimisées en préproduction

### **Évaluation Performance :**
```bash
# Avant optimisation
docker images | grep app-old
# → 1.2GB

# Après optimisation  
docker images | grep app-new
# → 120MB

# Calcul gain
echo "Gain: $(( (1200 - 120) * 100 / 1200 ))%"
# → Gain: 90%
```

---

## **📈 PROGRESSION DAY 14**

**✅ Compétences Acquises :**
- Maîtrise complète des Dockerfiles multi-stage
- Techniques avancées de réduction de taille d'images
- Sécurisation des images avec users non-root
- Analyse comparative et métriques de performance
- Optimisation pour différents langages (Node.js, Go, Python)

**🎯 Mentalité DevOps :**
> Je ne déploie plus des applications lourdes  
> Je déploie des artefacts optimisés et sécurisés

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 14 / 100 ✅`**

**#Docker #MultiStage #Optimization #DevOps #CloudNative #Security #BestPractices**

---
