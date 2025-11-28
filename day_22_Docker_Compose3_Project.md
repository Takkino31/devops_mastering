# **DAY 22 - PROJET COMPLET DOCKER COMPOSE** 🎯

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture 3-Tiers avec Docker**
- **Présentation** → Frontend (React/Node.js)
- **Business** → Backend API (Python/Flask)
- **Données** → Database (PostgreSQL) + Cache (Redis)

### **🌍 Multi-Environnements**
| Environnement | Configuration | Usage |
|---------------|---------------|--------|
| **Development** | Debug activé, hot reload | Développement local |
| **Production** | Optimisé, sécurisé | Déploiement |
| **Staging** | Configuration intermédiaire | Tests |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion Multi-Environnements**
| Commande                              | FR                        | EN                | Usage                                     |
|---------------------------------------|---------------------------|-------------------|-------------------------------------------|
| `docker-compose -f file1 -f file2`    | Composition fichiers      | **COMPOSE files** | `docker-compose -f base.yml -f prod.yml`  |
| `docker-compose override`             | Surcharge développement   | **OVERRIDE**      | Automatique                               |
| `docker system prune`                 | Nettoyage système         | **SYSTEM prune**  | `docker system prune -f`                  |

### **🚀 Déploiement Automatisé**
| Commande | FR | EN | Usage |
|---------------------|-------------------------|-----------------------|-----------------------|
| `./deploy.sh [env]` | Script déploiement      | **DEPLOY script**     | `./deploy.sh prod`    |
| `docker-compose ps` | Vérification services   | **CHECK services**    | `docker-compose ps`   |

---

## **📝 ARCHITECTURE COMPOSE AVANCÉE**

### **Structure Fichiers Multi-Environnements**
```
app-3-tiers/
├── docker-compose.yml           # Configuration base
├── docker-compose.override.yml  # Développement (automatique)
├── docker-compose.prod.yml      # Production
├── .env.dev                     # Variables développement
├── .env.prod                    # Variables production
└── deploy.sh                    # Script déploiement
```

### **Fichier de Base (docker-compose.yml)**
```yaml
version: '3.8'

services:
  database:
    image: postgres:13
    env_file: .env.${ENVIRONMENT:-dev}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
    networks:
      - backend

  backend:
    build:
      context: ./backend
      target: ${BUILD_TARGET:-development}
    env_file: .env.${ENVIRONMENT:-dev}
    depends_on:
      database:
        condition: service_healthy
    networks:
      - backend
      - frontend

  frontend:
    build:
      context: ./frontend  
      target: ${BUILD_TARGET:-development}
    env_file: .env.${ENVIRONMENT:-dev}
    depends_on:
      backend:
        condition: service_healthy
    networks:
      - frontend
```

### **Surcharge Développement (docker-compose.override.yml)**
```yaml
version: '3.8'

services:
  backend:
    ports: ["5000:5000"]
    volumes: ["./backend:/app"]
    environment:
      - DEBUG=true
      - RELOAD=true

  frontend:
    ports: ["3000:3000"] 
    volumes: ["./frontend:/app"]
    environment:
      - CHOKIDAR_USEPOLLING=true
```

---

## **🚀 DOCKERFILES MULTI-STAGE**

### **Backend Multi-Stage**
```dockerfile
FROM python:3.9-slim AS development
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]

FROM development AS production
RUN pip install gunicorn
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
```

### **Frontend Multi-Stage**
```dockerfile
FROM node:16-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

FROM node:16-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Variables d'Environnement par Configuration**
```yaml
# docker-compose.yml
services:
  backend:
    build:
      target: ${BUILD_TARGET:-development}
    env_file:
      - .env.${ENVIRONMENT:-dev}
```

```env
# .env.dev
ENVIRONMENT=dev
BUILD_TARGET=development
DEBUG=true
POSTGRES_DB=myapp_dev

# .env.prod  
ENVIRONMENT=prod
BUILD_TARGET=production
DEBUG=false
POSTGRES_DB=myapp
```

### **Script de Déploiement Automatisé**
```bash
#!/bin/bash
ENVIRONMENT=${1:-dev}

echo "🚀 Déploiement: $ENVIRONMENT"
export ENVIRONMENT=$ENVIRONMENT

if [ "$ENVIRONMENT" = "prod" ]; then
    docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
else
    docker-compose up -d --build
fi

echo "✅ Déploiement terminé"
docker-compose ps
```

### **Résolution Problème Port PostgreSQL**
```yaml
# MAUVAIS - Conflit port en développement
database:
  ports:
    - "5432:5432"

# BON - Pas d'exposition, réseau interne uniquement
database:
  # Pas de ports en développement
  # Le backend accède via 'database:5432'
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Architecture 3-Tiers Complète**
```yaml
# Stack complète avec isolation réseau
services:
  database: PostgreSQL (backend network)
  cache: Redis (backend network) 
  backend: API (backend + frontend networks)
  frontend: Application (frontend network)
```

### **2. Configuration Multi-Environnements**
```bash
# Développement
./deploy.sh dev
# → Utilise docker-compose.override.yml automatiquement

# Production  
./deploy.sh prod
# → Combine avec docker-compose.prod.yml
```

### **3. Dockerfiles Multi-Stage Optimisés**
```dockerfile
# Développement : sources montées, rechargement automatique
# Production : build optimisé, dépendances production uniquement
```

### **4. Gestion des Dépendances Robustes**
```yaml
services:
  backend:
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy

  frontend:
    depends_on:
      backend:
        condition: service_healthy
```

### **5. Résolution Problèmes Rencontrés**
- **Port 5432 occupé** → Retrait exposition port database en dev
- **Warnings Dockerfile** → Correction casse (`AS` vs `as`)
- **État corrompu** → Nettoyage complet avec `docker system prune`

---

## **🎯 BONNES PRATIQUES PRODUCTION**

### **Checklist Déploiement Production :**
- ✅ **Environnements séparés** (dev/staging/prod)
- ✅ **Builds optimisés** avec multi-stage
- ✅ **Variables sécurisées** dans .env
- ✅ **Health checks** configurés
- ✅ **Réseaux isolés** par couche
- ✅ **Documentation** complète

### **Sécurité Multi-Environnements :**
```bash
# .gitignore
.env.dev
.env.prod
docker-compose.override.yml
```

### **Optimisation Production :**
```yaml
# docker-compose.prod.yml
services:
  backend:
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
    environment:
      - DEBUG=false
      - LOG_LEVEL=INFO
```

---

## **📈 PROGRESSION DAY 22**

**✅ Compétences Acquises :**
- Architecture 3-tiers complète avec Docker Compose
- Gestion des multi-environnements (dev/prod)
- Dockerfiles multi-stage optimisés
- Scripts de déploiement automatisés
- Résolution de problèmes avancés
- Documentation professionnelle

**🎯 Mentalité DevOps :**
> Je ne déploie plus des applications simples  
> Je déploie des architectures complexes prêtes pour la production

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 22 / 100 ✅`**

**#DockerCompose #Microservices #ProductionReady #DevOps #Architecture3Tiers #MultiEnvironment**

---
