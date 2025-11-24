# **DAY 18 - DOCKER VOLUMES - PERSISTENCE DES DONNÉES** 💾

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture de Stockage Docker**
- **Volumes Docker** → Stockage géré par Docker
- **Bind Mounts** → Lien direct avec répertoire hôte
- **tmpfs Mounts** → Stockage en mémoire RAM

### **📊 Types de Persistance**
|-------------------|------------|--------------|-----------------------|
| Type              | Persistence| Performance  | Usage                 |
|-------------------|------------|--------------|-----------------------|
| **Volumes**       | ✅         | ✅✅        | Production            |
| **Bind Mounts**   | ✅         | ✅✅        | Développement         |
| **tmpfs**         | ❌         | ✅✅✅      | Données temporaires   |
|-------------------|------------|--------------|-----------------------|

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion des Volumes**
|---------------------------|-------------------|-----------------------|-----------------------------------|
| Commande                  | FR                | EN                    | Usage                             |
|---------------------------|-------------------|-----------------------|-----------------------------------|
| `docker volume create`    | Créer volume      | **CREATE volume**     | `docker volume create mon-volume` |
| `docker volume ls`        | Lister volumes    | **LIST volumes**      | `docker volume ls`                |
| `docker volume inspect`   | Inspecter volume  | **INSPECT volume**    | `docker volume inspect mon-volume`|
| `docker volume rm`        | Supprimer volume  | **REMOVE volume**     | `docker volume rm mon-volume`     |
| `docker volume prune`     | Nettoyer volumes  | **PRUNE volumes**     | `docker volume prune`             |
|---------------------------|-------------------|-----------------------|-----------------------------------|

### **🐳 Gestion des Conteneurs**
|---------------------------|-----------------------|-----------------------|-------------------------------|
| Commande                  | FR                    | EN                    | Usage                         |
|---------------------------|-----------------------|-----------------------|-------------------------------|
| `docker rm -f`            | Arrêter+supprimer     | **REMOVE force**      | `docker rm -f mon-conteneur`  |
| `docker container prune`  | Nettoyer conteneurs   | **PRUNE containers**  | `docker container prune`      |
|---------------------------|-----------------------|-----------------------|-------------------------------|

---

## **📝 TYPES DE STOCKAGE DOCKER**

### **Volumes Docker (Recommandé)**
```bash
# Création et utilisation
docker volume create mes-donnees
docker run -v mes-donnees:/chemin/interne nginx:alpine

# Emplacement sur l'hôte
/var/lib/docker/volumes/mes-donnees/_data
```

### **Bind Mounts (Développement)**
```bash
# Lien direct avec répertoire hôte
docker run -v /chemin/local:/chemin/conteneur nginx:alpine
```

### **tmpfs (Mémoire)**
```bash
# Stockage temporaire en RAM
docker run --tmpfs /chemin nginx:alpine
```

---

## **🚀 STRATÉGIES DE PERSISTANCE**

### **Base de Données Persistante**
```bash
# Volume dédié pour MySQL
docker volume create mysql-data
docker run -d --name mysql-persistant \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

### **Application avec État**
```bash
# Application qui maintient son état
docker run -d --name mon-app \
  -v app-data:/data \
  -p 5000:5000 \
  mon-image-app
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Cycle de Vie des Données :**
```bash
# 1. Créer le volume
docker volume create mes-donnees

# 2. Utiliser dans un conteneur
docker run -d -v mes-donnees:/data --name app1 mon-app

# 3. Arrêter et supprimer le conteneur
docker rm -f app1

# 4. Réutiliser le même volume
docker run -d -v mes-donnees:/data --name app2 mon-app
# → Les données sont toujours présentes !
```

### **MySQL avec Persistance :**
```bash
# Configuration base de données persistante
docker volume create mysql-data
docker run -d --name mysql-db \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# Les données survivent au redémarrage
docker rm -f mysql-db
docker run -d --name nouveau-mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
# → Données intactes !
```

### **Application de Compteur :**
```python
# Application Flask avec état persistant
from flask import Flask
import os

app = Flask(__name__)
COUNTER_FILE = '/data/counter.txt'

def read_counter():
    try:
        with open(COUNTER_FILE, 'r') as f:
            return int(f.read().strip())
    except:
        return 0

def write_counter(value):
    with open(COUNTER_FILE, 'w') as f:
        f.write(str(value))

@app.route('/')
def increment():
    count = read_counter() + 1
    write_counter(count)
    return f'Compteur: {count}'
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Gestion Complète des Volumes**
```bash
# Création et inspection
docker volume create mon-volume
docker volume ls
docker volume inspect mon-volume

# Utilisation avec conteneur
docker run -d --name test-volume -v mon-volume:/data nginx:alpine
docker exec test-volume ls -la /data

# Nettoyage
docker rm -f test-volume
docker volume rm mon-volume
```

### **2. Base de Données MySQL Persistante**
```bash
# Volume pour données MySQL
docker volume create mysql-persistent
docker run -d --name mysql-db \
  -v mysql-persistent:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# Création de données de test
docker exec mysql-db mysql -uroot -psecret -e "CREATE DATABASE testdb;"

# Test de persistance
docker rm -f mysql-db
docker run -d --name mysql-new \
  -v mysql-persistent:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# Vérification données intactes
docker exec mysql-new mysql -uroot -psecret -e "SHOW DATABASES;"
```

### **3. Application avec État Préservé**
```bash
# Application compteur avec volume
docker build -t app-compteur .
docker run -d --name compteur \
  -p 5000:5000 \
  -v compteur-data:/data \
  app-compteur

# Test d'incrémentation
curl http://localhost:5000
# → Compteur: 1
curl http://localhost:5000  
# → Compteur: 2

# Redémarrage avec persistance
docker rm -f compteur
docker run -d --name compteur-new \
  -p 5000:5000 \
  -v compteur-data:/data \
  app-compteur

curl http://localhost:5000
# → Compteur: 3 (état préservé !)
```

### **4. Sauvegarde et Restauration**
```bash
# Sauvegarde d'un volume
docker run --rm -v mysql-persistent:/source -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /source .

# Restauration vers nouveau volume
docker volume create mysql-restore
docker run --rm -v mysql-restore:/target -v $(pwd):/backup alpine \
  tar xzf /backup/backup.tar.gz -C /target
```

---

## **🎯 BONNES PRATIQUES VOLUMES**

### **Checklist Production :**
- ✅ **Utiliser volumes nommés** pour la production
- ✅ **Éviter bind mounts** en environnement critique
- ✅ **Sauvegarder régulièrement** les volumes importants
- ✅ **Documenter les volumes** utilisés

### **Sécurité :**
```bash
# Volumes nommés pour contrôle d'accès
docker volume create app-data-prod
docker run -v app-data-prod:/app/data mon-app

# Éviter les chemins absolus sensibles
# MAUVAIS: -v /etc:/app/config
# BON: -v config-data:/app/config
```

### **Gestion du Cycle de Vie :**
```bash
# Suppression sécurisée
docker rm -f mon-conteneur
docker volume rm mon-volume

# Nettoyage global
docker container prune
docker volume prune
```

---

## **📈 PROGRESSION DAY 18**

**✅ Compétences Acquises :**
- Maîtrise des volumes Docker et leur gestion
- Configuration de bases de données persistantes
- Applications avec état préservé entre redémarrages
- Techniques de sauvegarde et restauration
- Bonnes pratiques pour la production

**🎯 Mentalité DevOps :**
> Je ne perds plus les données à chaque déploiement  
> Je gère des applications avec état préservé et contrôlé

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 18 / 100 ✅`**

**#Docker #Volumes #Persistance #DataManagement #DevOps #Storage**

---

**PRÊT POUR LES VOLUMES AVANCÉS ET LA RÉPLICATION ?** 🚀
