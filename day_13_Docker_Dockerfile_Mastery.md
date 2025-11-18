# **DAY 13 - DOCKERFILE MASTERY** 🏗️

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Dockerfile**
- **Instructions séquentielles** → Chaque ligne = une couche
- **Build context** → Environnement de construction
- **Système de cache** → Optimisation des rebuilds

### **📦 Les Instructions Fondamentales**
- **FROM** → Image de base
- **RUN, COPY, ADD** → Construction de l'image
- **CMD, ENTRYPOINT** → Exécution du conteneur
- **ENV, ARG, WORKDIR** → Configuration environnement

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔨 Construction d'Images**
|---------------------------|-------------------|-----------------------|---------------------------------------|
| Commande                  | FR                | EN                    | Usage                                 |
|---------------------------|-------------------|-----------------------|---------------------------------------|
| `docker build -t nom .`   | Construire image  | **BUILD image**       | `docker build -t mon-app .`           |
| `docker build --no-cache` | Ignorer cache     | **Build NO CACHE**    | `docker build --no-cache -t app .`    |
| `docker history image`    | Voir les couches  | **HISTORY layers**    | `docker history mon-app`              |
|---------------------------|-------------------|-----------------------|---------------------------------------|

### **🏷️ Gestion des Images**
|-------------------------------|-------------------|-------------------|-----------------------------------------|
| Commande                      | FR                | EN                | Usage                                   |
|-------------------------------|-------------------|-------------------|-----------------------------------------|
| `docker tag source target`    | Tagger image      | **TAG image**     | `docker tag mon-app:1.0 mon-app:latest` |
| `docker image ls`             | Lister images     | **LIST images**   | `docker image ls`                       |
| `docker image rm`             | Supprimer image   | **REMOVE image**  | `docker image rm mon-app`               |
|-------------------------------|-------------------|-------------------|-----------------------------------------|

### **🚀 Exécution Avancée**
|-------------------------------|---------------|-------------------|--------------------------------|
| Commande                      | FR            | EN                | Usage                          |
|-------------------------------|---------------|-------------------|--------------------------------|
| `docker run --env VAR=val`    | Variables env | **ENV variables** | `docker run -e DEBUG=true app` |
| `docker run --user user`      | Utilisateur   | **USER context**  | `docker run --user node app`   |
|-------------------------------|---------------|-------------------|--------------------------------|
---

## **📝 INSTRUCTIONS DOCKERFILE**

### **Instructions de Base**
|---------------|-----------------------|-----------------------|-----------------------|
| Instruction   | Usage                 | Exemple               | Effet                 |
|---------------|-----------------------|-----------------------|-----------------------|
| `FROM`        | Image de base         | `FROM node:18-alpine` | Point de départ       |
| `WORKDIR`     | Répertoire travail    | `WORKDIR /app`        | Définit le dossier    |
| `COPY`        | Copier fichiers       | `COPY . /app`         | Copie fichiers        |
| `RUN`         | Exécuter commande     | `RUN npm install`     | Pendant build         |
|---------------|-----------------------|-----------------------|-----------------------|

### **Instructions d'Exécution**
|---------------|-----------------------|---------------------------|-------------------|
| Instruction   | Usage                 | Exemple                   | Effet             |
|---------------|-----------------------|---------------------------|-------------------|
| `CMD`         | Commande par défaut   | `CMD ["node", "app.js"]`  | Au lancement      |
| `ENTRYPOINT`  | Point d'entrée        | `ENTRYPOINT ["python"]`   | Exécutable fixe   |
| `EXPOSE`      | Documentation ports   | `EXPOSE 3000`             | Ports utilisés    |
|---------------|-----------------------|---------------------------|-------------------|

### **Instructions Configuration**
|---------------|---------------------------|---------------------------|-----------------------|
| Instruction   | Usage                     | Exemple                   | Effet                 |
|---------------|---------------------------|---------------------------|-----------------------|
| `ENV`         | Variables environnement   | `ENV NODE_ENV=production` | Variables permanentes |
| `ARG`         | Variables build           | `ARG APP_VERSION`         | Variables temporaires |
| `LABEL`       | Métadonnées               | `LABEL version="1.0"`     | Informations image    |
|---------------|---------------------------|---------------------------|-----------------------|

---

## **⚡ OPTIMISATIONS & BONNES PRATIQUES**

### **🏗️ Stratégies de Construction**
|-------------------|----------------|------------------------------|
| Stratégie         | Avantage       | Usage                        |
|-------------------|----------------|------------------------------|
| **Layer caching** | Builds rapides | Instructions stables en haut |
| **Multi-stage**   | Images légères | Build et runtime séparés     |
| **Slim images**   | Taille réduite | `alpine`, `slim`             |
|-------------------|----------------|------------------------------|

### **🔧 Optimisations Dockerfile**
```dockerfile
# MAUVAIS - Crée plusieurs couches inutiles
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2
RUN rm -rf /var/lib/apt/lists/*

# BON - Une seule couche optimisée
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    rm -rf /var/lib/apt/lists/*
```

### **📁 Gestion Build Context**
```dockerfile
# .dockerignore essentiel
node_modules/
.git/
*.log
.env
Dockerfile
.gitignore
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Le Mystère du Cache Docker :**
```bash
# Chaque instruction Dockerfile = une couche
# Couche modifiée = cache invalidé pour les suivantes
# → Placer les instructions stables en premier
```

### **COPY vs ADD :**
```bash
# COPY → Simple copie de fichiers
COPY package.json ./

# ADD → Copie + extraction automatique
ADD app.tar.gz /app/
```

### **CMD vs ENTRYPOINT :**
```bash
# CMD → Commande par défaut modifiable
CMD ["npm", "start"]

# ENTRYPOINT → Exécutable fixe
ENTRYPOINT ["node"]
```

### **Variables ENV vs ARG :**
```bash
# ENV → Disponible dans le conteneur
ENV NODE_ENV=production

# ARG → Uniquement pendant le build
ARG BUILD_VERSION
```

---

## **🚀 EXERCICES RÉALISÉS**

### **1. Dockerfile Node.js Optimisé**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src/ ./src/
USER node
EXPOSE 3000
CMD ["node", "src/index.js"]
```

### **2. Dockerfile Python Professionnel**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN apt-get update && apt-get install -y gcc && rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["python", "app.py"]
```

### **3. Build et Test**
```bash
# Construction d'image
docker build -t mon-app-node:1.0 .

# Exécution avec port mapping
docker run -d -p 8093:3000 --name app-node mon-app-node:1.0

# Test de l'application
curl http://localhost:8093

# Inspection des couches
docker history mon-app-node:1.0
```

### **4. Gestion du Cache**
```bash
# Build avec cache normal
docker build -t mon-app .

# Build sans cache (clean build)
docker build --no-cache -t mon-app .

# Voir l'utilisation du cache
docker build -t mon-app . --progress=plain
```

---

## **🎯 MÉTHODOLOGIE DOCKERFILE**

### **Approche de Construction :**
```bash
1. FROM → Choisir image de base appropriée
2. RUN → Installer dépendances système
3. COPY → Fichiers de dépendances (package.json, requirements.txt)
4. RUN → Installer dépendances applicatives
5. COPY → Code applicatif
6. CMD/ENTRYPOINT → Commande de démarrage
```

### **Bonnes Pratiques :**
- ✅ **Toujours** utiliser des images officielles
- ✅ **Toujours** spécifier des tags explicites
- ✅ **Toujours** utiliser USER non-root
- ✅ **Toujours** optimiser l'ordre des instructions
- ✅ **Toujours** utiliser .dockerignore

### **Sécurité :**
```dockerfile
# Exécuter en tant qu'utilisateur non-privilégié
USER node

# Utiliser des images signées
FROM node:18-alpine@sha256:...

# Mettre à jour régulièrement les images
```

---

## **📈 PROGRESSION DAY 13**

**✅ Compétences Acquises :**
- Maîtrise complète des instructions Dockerfile
- Optimisation du cache de construction
- Gestion du build context avec .dockerignore
- Création d'images sécurisées et optimisées
- Bonnes pratiques de construction professionnelle

**🎯 Mentalité DevOps :**
> Je ne lance plus des applications  
> Je construis des images reproductibles et optimisées

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 13 / 100 ✅`**

**#Docker #Dockerfile #DevOps #Containerization #BestPractices #Optimization**

---
