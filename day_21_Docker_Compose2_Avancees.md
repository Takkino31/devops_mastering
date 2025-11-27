# **DAY 21 - DOCKER COMPOSE AVANCÉ** 🏗️⚡

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Orchestration Avancée**
- **Dépendances conditionnelles** → Services qui attendent que d'autres soient prêts
- **Health Checks** → Vérification automatique de l'état des services
- **Gestion des variables** → Configuration sécurisée avec .env

### **📊 Gestion des Dépendances**
|-------------------------------|---------------------------|-------------------------------|
| Type                          | Usage                     | Avantage                      |
|-------------------------------|---------------------------|-------------------------------|
| `depends_on`                  | Ordre de démarrage        | Démarrage séquentiel          |
| `condition: service_healthy`  | Dépendance conditionnelle | Services réellement prêts     |
| Health Checks                 | Monitoring santé          | Détection précoce des pannes  |
|-------------------------------|---------------------------|-------------------------------|

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion des Health Checks**
|---------------------------|---------------------------|-----------------------|-------------------------------|
| Commande                  | FR                        | EN                    | Usage                         |
|---------------------------|---------------------------|-----------------------|-------------------------------|
| `docker-compose ps`       | Vérifier état services    | **Check STATUS**      | `docker-compose ps`           |
| `docker-compose logs`     | Voir logs santé           | **HEALTH logs**       | `docker-compose logs service` |
| `docker-compose up -d`    | Démarrer avec santé       | **UP with health**    | `docker-compose up -d`        |
|---------------------------|---------------------------|-----------------------|-------------------------------|

### **🐳 Inspection des Services**
|-------------------------------|-------------------|---------------------------|---------------------------------------|
| Commande                      | FR                | EN                        | Usage                                 |
|-------------------------------|-------------------|---------------------------|---------------------------------------|
| `docker-compose exec service` | Inspecter service | **EXECUTE in service**    | `docker-compose exec backend bash`    |
| `docker-compose config`       | Vérifier config   | **VALIDATE config**       | `docker-compose config`               |
|-------------------------------|-------------------|---------------------------|---------------------------------------|

---

## **📝 CONFIGURATION AVANCÉE COMPOSE**

### **Health Checks par Service**
```yaml
services:
  database:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d dbname"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  backend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### **Dépendances Conditionnelles**
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

---

## **🚀 ARCHITECTURE 2-TIERS AVANCÉE**

### **Stack Complète avec Dépendances**
```yaml
version: '3.8'

services:
  database:
    image: postgres:13
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  backend:
    build: ./backend
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build: ./frontend
    depends_on:
      backend:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### **Gestion des Variables d'Environnement**
```yaml
# docker-compose.yml
services:
  backend:
    environment:
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@database:5432/${DB_NAME}
      - SECRET_KEY=${SECRET_KEY}
      - DEBUG=${DEBUG}
```

```env
# .env
DB_NAME=myapp
DB_USER=appuser
DB_PASSWORD=supersecret
SECRET_KEY=myverysecretkey
DEBUG=false
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Problème Résolu : Health Check Frontend**
```yaml
# MAUVAIS - curl non disponible dans node:alpine
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]

# BON - wget disponible ou test simplifié
healthcheck:
  test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/"]
```

### **Dépendances Robustes :**
```yaml
# Fragile - démarre même si le service n'est pas prêt
depends_on:
  - database

# Robuste - attend que le service soit healthy
depends_on:
  database:
    condition: service_healthy
```

### **Configuration Sécurisée :**
```python
# Backend - utilisation sécurisée des variables
import os

DB_NAME = os.getenv('DB_NAME', 'default_db')
SECRET_KEY = os.getenv('SECRET_KEY')
DEBUG = os.getenv('DEBUG', 'false').lower() == 'true'

@app.route('/health')
def health():
    return jsonify({
        "status": "healthy",
        "database": DB_NAME,
        "debug_mode": DEBUG
    })
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Stack 2-Tiers Complète**
```yaml
# Architecture : Frontend → Backend → (Database + Cache)
services:
  database: PostgreSQL avec health check
  cache: Redis avec monitoring
  backend: API Flask avec dépendances conditionnelles  
  frontend: Application Node.js avec health check
```

### **2. Health Checks Configurés**
```bash
# PostgreSQL
test: ["CMD-SHELL", "pg_isready -U user -d dbname"]

# Redis  
test: ["CMD", "redis-cli", "ping"]

# Backend API
test: ["CMD", "curl", "-f", "http://localhost:5000/health"]

# Frontend
test: ["CMD", "wget", "--spider", "http://localhost:3000/"]
```

### **3. Variables d'Environnement Sécurisées**
```env
# .env - jamais commité
DB_NAME=myapp
DB_USER=appuser
DB_PASSWORD=supersecretpassword
SECRET_KEY=myverysecretkey123
DEBUG=false
```

### **4. Démarrage Conditionnel Réussi**
```bash
# Les services démarrent dans le bon ordre
1. database ✅ (healthy)
2. cache ✅ (healthy)  
3. backend ✅ (healthy)
4. frontend ✅ (healthy)
```

### **5. Résolution de Problème**
**Problème :** Health check frontend échoue  
**Cause :** `curl` non disponible dans node:alpine  
**Solution :** Remplacer par `wget` dans le test health check

---

## **🎯 BONNES PRATIQUES AVANCÉES**

### **Checklist Health Checks :**
- ✅ **Tests réalistes** qui vérifient la fonctionnalité
- ✅ **Intervalles adaptés** à chaque service
- ✅ **Timeouts raisonnables** pour éviter les faux positifs
- ✅ **Retries suffisantes** pour les services lents à démarrer

### **Gestion des Dépendances :**
```yaml
# DÉVELOPPEMENT - démarrage rapide
depends_on:
  - service

# PRODUCTION - robustesse
depends_on:
  service:
    condition: service_healthy
```

### **Sécurité Variables :**
- ✅ **Fichier .env** dans .gitignore
- ✅ **Valeurs par défaut** pour le développement
- ✅ **Secrets managés** pour la production
- ✅ **Configuration par environnement**

---

## **📈 PROGRESSION DAY 21**

**✅ Compétences Acquises :**
- Configuration de health checks avancés
- Gestion des dépendances conditionnelles entre services
- Sécurisation des variables d'environnement
- Résolution de problèmes d'orchestration
- Architecture 2-tiers robuste avec Compose

**🎯 Mentalité DevOps :**
> Mes applications ne démarrent plus au hasard  
> Chaque service attend que ses dépendances soient réellement prêtes

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 21 / 100 ✅`**

**#DockerCompose #HealthChecks #DevOps #Dependencies #EnvironmentVariables #Orchestration**

---

**PRÊT POUR LES STACKS 3-TIERS ET MULTI-ENVIRONNEMENTS ?** 🚀
