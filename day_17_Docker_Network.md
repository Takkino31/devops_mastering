# **DAY 17 - DOCKER NETWORKING** 🌐

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Réseau Docker**
- **Network Drivers** → Bridge, Host, None
- **DNS Automatique** → Résolution par nom de conteneur
- **Isolation Réseau** → Séparation des applications

### **🔗 Communication Inter-Conteneurs**
- **Bridge Network** → Réseau privé interne par défaut
- **Host Network** → Partage réseau avec l'hôte
- **Custom Networks** → Réseaux personnalisés pour applications

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion des Réseaux**
|---------------------------|-----------------------|-----------------------|-------------------------------------------|
| Commande                  | FR                    | EN                    | Usage                                     |
|---------------------------|-----------------------|-----------------------|-------------------------------------------|
| `docker network create`   | Créer réseau          | **CREATE network**    | `docker network create mon-reseau`        |
| `docker network ls`       | Lister réseaux        | **LIST networks**     | `docker network ls`                       |
| `docker network inspect`  | Inspecter réseau      | **INSPECT network**   | `docker network inspect mon-reseau`       |
| `docker network connect`  | Connecter conteneur   | **CONNECT container** | `docker network connect reseau conteneur` |
| `docker network rm`       | Supprimer réseau      | **REMOVE network**    | `docker network rm mon-reseau`            |
|---------------------------|-----------------------|-----------------------|-------------------------------------------|

### **🌐 Test de Connectivité**
|------------------------------|--------------------|-------------------------|---------------------------------------------|
| Commande                      | FR                | EN                      | Usage                                       |
|------------------------------|--------------------|-------------------------|---------------------------------------------|
| `docker exec conteneur ping` | Tester connexion   | **PING from container** | `docker exec app ping db`                   |
| `docker exec conteneur curl` | Tester HTTP        | **CURL from container** | `docker exec frontend curl backend:5000`    |
|------------------------------|--------------------|-------------------------|---------------------------------------------|

---

## **📝 NETWORK DRIVERS DOCKER**

### **Comparaison des Drivers**
|---------------|---------------------------------------|----------------|--------------------|
| Driver        | Usage                                 | Isolation      | Performance        |
|---------------|---------------------------------------|----------------|--------------------|
| **bridge**    | Défaut, applications multi-conteneurs | ✅ Complète    | 🔄 Standard        |
| **host**      | Applications réseau critiques         | ❌ Aucune      | ⚡ Maximale        |
| **none**      | Conteneurs sans réseau                | ✅ Totale      | 🚫 Aucun réseau    |
|---------------|---------------------------------------|----------------|--------------------|

### **DNS Docker Automatique**
```bash
# Les conteneurs peuvent communiquer par nom
conteneur-web → db (hostname) → 172.18.0.3

# Résolution DNS intégrée
docker exec frontend ping backend
# → Réponse depuis l'IP du backend
```

---

## **🚀 ARCHITECTURE APPLICATION MULTI-TIER**

### **Diagramme Réseau**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   FRONTEND      │    │    BACKEND      │    │   DATABASE      │
│                 │    │                 │    │                 │
│  nginx:alpine   │◄──►│  python:flask   │◄──►│   mysql:8.0     │
│                 │    │                 │    │                 │
│  Port 80 (host) │    │  Port 5000      │    │  Port 3306      │
│                 │    │                 │    │  (interne)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                        ┌────────┴────────┐
                        │  app-network    │
                        │  (bridge)       │
                        └─────────────────┘
```

### **Flux de Communication**
```bash
1. Utilisateur → Frontend (port 80)
2. Frontend → Backend (DNS: backend:5000) 
3. Backend → Database (DNS: mysql-db:3306)
4. Database → Backend → Frontend → Utilisateur
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Création Réseau Custom :**
```bash
# Créer un réseau bridge personnalisé
docker network create app-network

# Inspecter le réseau créé
docker network inspect app-network
# → Voir subnet, gateway, conteneurs connectés
```

### **Lancement avec Réseau Custom :**
```bash
# Lancer un conteneur sur un réseau spécifique
docker run -d \
  --name mysql-db \
  --network app-network \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

### **Communication Inter-Conteneurs :**
```bash
# Tester la connexion depuis un conteneur
docker exec frontend curl http://backend:5000/health
# → Réponse JSON du backend

# Tester la résolution DNS
docker exec backend ping mysql-db
# → Réponse depuis la base de données
```

### **Application Complète :**
```bash
# 1. Créer le réseau
docker network create app-network

# 2. Lancer la database
docker run -d --name mysql-db --network app-network mysql:8.0

# 3. Lancer le backend  
docker run -d --name backend --network app-network -p 5000:5000 backend-app

# 4. Lancer le frontend
docker run -d --name frontend --network app-network -p 80:80 nginx:alpine
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Création et Gestion Réseaux**
```bash
# Création réseau custom
docker network create mon-reseau-app

# Inspection détaillée
docker network inspect mon-reseau-app

# Liste complète réseaux
docker network ls
```

### **2. Application Multi-Conteneurs**
```dockerfile
# Backend Flask
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

### **3. Tests de Connectivité**
```bash
# Test backend
curl http://localhost:5000/health
# → {"status": "healthy", "database": "connected"}

# Test depuis conteneur
docker exec frontend curl http://backend:5000
# → Communication interne réussie

# Test DNS
docker exec backend ping mysql-db
# → Résolution par nom fonctionnelle
```

### **4. Monitoring Réseau**
```bash
# Voir tous les conteneurs connectés
docker network inspect app-network

# Voir les logs de communication
docker logs backend
docker logs mysql-db
```

---

## **🎯 MÉTHODOLOGIE RÉSEAU DOCKER**

### **Approche Systématique :**
```bash
1. docker network create [nom-reseau]
2. docker run --network [nom-reseau] --name [nom] [image]
3. Utiliser [nom-conteneur] comme hostname
4. Tester la connectivité
5. docker network inspect [nom-reseau] pour vérification
```

### **Bonnes Pratiques :**
- ✅ **Toujours** utiliser des réseaux custom pour les applications
- ✅ **Toujours** nommer les conteneurs pour le DNS
- ✅ **Toujours** tester la connectivité après déploiement
- ✅ **Toujours** inspecter le réseau pour le debugging

### **Débogage Réseau :**
```bash
# Problème de connexion ?
docker network inspect [reseau]
docker exec [conteneur] ping [autre-conteneur]
docker exec [conteneur] nslookup [autre-conteneur]
docker logs [conteneur]
```

---

## **📈 PROGRESSION DAY 17**

**✅ Compétences Acquises :**
- Maîtrise des network drivers Docker (bridge, host, none)
- Création et gestion de réseaux custom
- Communication inter-conteneurs via DNS automatique
- Architecture d'applications multi-tiers
- Tests de connectivité avancés

**🎯 Mentalité DevOps :**
> Je ne déploie plus des conteneurs isolés  
> Je déploie des écosystèmes connectés et communicants

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 17 / 100 ✅`**

**#Docker #Networking #DevOps #Containers #Microservices #DNS #Infrastructure**

---
