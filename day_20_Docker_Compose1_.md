# **DAY 20 - DOCKER COMPOSE FONDATIONS** 🐳📦

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Docker Compose**
- **Orchestration multi-conteneurs** → Gestion simplifiée
- **Fichier YAML unique** → Configuration déclarative
- **Réseaux et volumes automatiques** → Gestion transparente

### **📝 Syntaxe YAML Docker Compose**
|---------------|---------------------------|---------------|
| Section       | Description               | Usage         |
|---------------|---------------------------|---------------|
| `version`     | Version Compose           | `'3.8'`       |
| `services`    | Définition des conteneurs | Applications  |
| `networks`    | Configuration réseau      | Communication |
| `volumes`     | Stockage persistant       | Données       |
|---------------|---------------------------|---------------|

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion Application Compose**
|-----------------------|-----------------------|-----------------------|---------------------------|
| Commande              | FR                    | EN                    | Usage                     |
|-----------------------|-----------------------|-----------------------|---------------------------|
| `docker-compose up -d`| Démarrer application  | **UP detached**       | `docker-compose up -d`    |
| `docker-compose down` | Arrêter application   | **DOWN**              | `docker-compose down`     |
| `docker-compose ps`   | Voir services         | **PROCESS status**    | `docker-compose ps`       |
| `docker-compose logs` | Voir logs             | **LOGS**              | `docker-compose logs -f`  |
|-----------------------|-----------------------|-----------------------|---------------------------|

### **🛠️ Développement avec Compose**
|---------------------------|--------------------|-------------|------------------------------------|
| Commande                  | FR                 | EN          | Usage                              |
|---------------------------|--------------------|-------------|------------------------------------|
| `docker-compose build`    | Build images       | **BUILD**   | `docker-compose build`             |
| `docker-compose exec`     | Exécuter commande  | **EXECUTE** | `docker-compose exec service bash` |
| `docker-compose restart`  | Redémarrer service | **RESTART** | `docker-compose restart web`       |
|---------------------------|--------------------|-------------|------------------------------------|

---

## **📝 STRUCTURE DOCKER-COMPOSE.YML**

### **Architecture de Base**
```yaml
version: '3.8'

services:
  service1:
    image: nom:tag
    ports:
      - "hote:conteneur"
    networks:
      - mon-reseau

  service2:
    build: ./dossier
    depends_on:
      - service1

networks:
  mon-reseau:

volumes:
  mon-volume:
```

### **Sections Principales**
```yaml
# Services - Cœur de l'application
services:
  web:
    image: nginx:alpine
    ports: ["80:80"]
    networks: ["frontend"]

  api:
    build: ./backend
    environment:
      - DB_HOST=database

  database:
    image: postgres:13
    volumes:
      - db_data:/var/lib/postgresql/data

# Infrastructure
networks:
  frontend:
  backend:

volumes:
  db_data:
```

---

## **🚀 STRATÉGIES D'ORCHESTRATION**

### **Application Simple vs Complexe**
|---------------|---------------|-----------|-----------------------|
| Type          | Services      | Complexité| Usage                 |
|---------------|---------------|-----------|-----------------------|
| **Simple**    | 1-2 services  | Faible    | Démonstration         |
| **Standard**  | 3-5 services  | Moyenne   | Applications réelles  |
| **Complexe**  | 5+ services   | Élevée    | Microservices         |
|---------------|---------------|-----------|-----------------------|

### **Build vs Image**
```yaml
# Utilisation d'image existante
web:
  image: nginx:alpine

# Build depuis Dockerfile
api:
  build:
    context: ./api
    dockerfile: Dockerfile
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Avantages Docker Compose :**
```bash
# AVANT - Commandes manuelles complexes
docker network create app-net
docker run -d --name db --network app-net mysql
docker run -d --name api --network app-net -p 5000:5000 api-image
docker run -d --name web --network app-net -p 80:80 web-image

# APRÈS - Une seule commande
docker-compose up -d
```

### **Fichier Compose Minimal :**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
```

### **Application Multi-Services :**
```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    networks: ["app-network"]

  backend:
    build: ./backend  
    ports: ["5000:5000"]
    environment:
      - DATABASE_URL=postgresql://user:pass@database:5432/app
    depends_on:
      - database
    networks: ["app-network"]

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks: ["app-network"]

volumes:
  postgres_data:

networks:
  app-network:
```

### **Gestion des Volumes et Réseaux :**
```yaml
# Volumes nommés (recommandé)
volumes:
  db_data:
  app_logs:

# Réseaux personnalisés
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Application Simple Nginx + MySQL**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    container_name: mon-nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - app-network

  database:
    image: mysql:8.0
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
```

### **2. Application avec Build Personnalisé**
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    networks:
      - app-network

  backend:
    build:
      context: ./backend  
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=database
    networks:
      - app-network

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
```

### **3. Commandes de Gestion**
```bash
# Démarrage complet
docker-compose up -d

# Vérification statut
docker-compose ps

# Logs en temps réel
docker-compose logs -f frontend

# Arrêt propre
docker-compose down

# Build forcé
docker-compose up --build
```

### **4. Structure de Projet**
```
mon-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
└── html/
    └── index.html
```

---

## **🎯 BONNES PRATIQUES COMPOSE**

### **Checklist Fichier Compose :**
- ✅ **Version spécifiée** (3.8+)
- ✅ **Services nommés** logiquement
- ✅ **Réseaux personnalisés** pour l'isolation
- ✅ **Volumes nommés** pour la persistance
- ✅ **Ports exposés** seulement si nécessaire

### **Organisation Projet :**
```bash
# Structure recommandée
project/
├── docker-compose.yml
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile  
├── database/
│   └── init.sql
└── README.md
```

### **Sécurité de Base :**
```yaml
# Éviter les mots de passe en clair
database:
  image: postgres
  environment:
    POSTGRES_PASSWORD: ${DB_PASSWORD}  # Variable d'environnement
```

---

## **📈 PROGRESSION DAY 20**

**✅ Compétences Acquises :**
- Maîtrise de la syntaxe YAML Docker Compose
- Création de fichiers docker-compose.yml
- Orchestration d'applications multi-conteneurs
- Gestion des réseaux et volumes dans Compose
- Commandes essentielles de gestion

**🎯 Mentalité DevOps :**
> Je ne lance plus des conteneurs individuellement  
> Je déploie des applications complètes avec une seule commande

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 20 / 100 ✅`**

**#DockerCompose #DevOps #Orchestration #YAML #Containers #InfrastructureAsCode**

---

**PRÊT POUR LES DÉPENDANCES ET HEALTH CHECKS AVANCÉS ?** 🚀
