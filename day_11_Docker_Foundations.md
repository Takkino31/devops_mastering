# **DAY 11 - DOCKER ARCHITECTURE & BASICS** 🐳

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Docker**
- **Client-Server** → Docker CLI communique avec Docker Daemon
- **Conteneurs légers** → Partage du kernel hôte vs VMs complètes
- **Images immuables** → Modèles read-only pour créer des conteneurs

### **⚔️ Conteneurs vs Machines Virtuelles**
| Aspect            | Conteneurs Docker         | Machines Virtuelles       |
|-------------------|---------------------------|---------------------------|
| **Démarrage**     | Secondes                  | Minutes                   |
| **Performance**   | Native (~1-2% overhead)   | Émulée (~15-20% overhead) |
| **Taille**        | Mo ~ Go                   | Go ~ dizaines de Go       |
| **Isolation**     | Niveau processus          | Niveau matériel           |
|-------------------|---------------------------|---------------------------|

### **🔧 Les 3 Piliers Techniques**
- **Namespaces** → Isolation PID, réseau, fichiersystem
- **cgroups** → Limitation CPU, mémoire, I/O
- **Union FS** → Système fichiers en couches (overlay2)

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🚀 Cycle de Vie Conteneurs**
| Commande          | FR                | EN                | Usage                         |
|-------------------|-------------------|-------------------|-------------------------------|
| `docker run`      | Lancer conteneur  | Run container     | `docker run -d nginx`         |
| `docker ps`       | Lister conteneurs | List containers   | `docker ps -a`                |
| `docker stop`     | Arrêter conteneur | Stop container    | `docker stop mon-conteneur`   |
| `docker start`    | Redémarrer        | Start container   | `docker start mon-conteneur`  |
| `docker rm`       | Supprimer         | Remove container  | `docker rm mon-conteneur`     |
|-------------------|-------------------|-------------------|-------------------------------|

### **🔍 Inspection et Debug**
| Commande          | FR                        | EN                | Usage                             |
|-------------------|---------------------------|-------------------|-----------------------------------|
| `docker logs`     | Voir logs                 | View logs         | `docker logs mon-conteneur`       |
| `docker exec`     | Exécuter commande         | Execute command   | `docker exec -it bash`            |
| `docker inspect`  | Inspecter détails         | Inspect details   | `docker inspect mon-conteneur`    |
| `docker stats`    | Statistiques ressources   | Resource stats    | `docker stats`                    |
| `docker top`      | Voir processus            | View processes    | `docker top mon-conteneur`        |
|-------------------|---------------------------|-------------------|-----------------------------------|

### **📦 Gestion Images**
| Commande          | FR                | EN            | Usage                 |
|-------------------|-------------------|---------------|-----------------------|
| `docker images`   | Lister images     | List images   | `docker images`       |
| `docker pull`     | Télécharger image | Pull image    | `docker pull nginx`   |
| `docker rmi`      | Supprimer image   | Remove image  | `docker rmi nginx`    |
|-------------------|-------------------|---------------|-----------------------|

### **🧹 Nettoyage**
| Commande                  | FR                    | EN                | Usage                     |
|---------------------------|-----------------------|-------------------|---------------------------|
| `docker container prune`  | Nettoyer conteneurs   | Clean containers  | `docker container prune`  |
| `docker system prune`     | Nettoyer système      | System cleanup    | `docker system prune`     |
|---------------------------|-----------------------|-------------------|---------------------------|

---

## **⚡ FLAGS IMPORTANTS**

### **🎯 Flags de Execution**
```bash
# Démarrer en arrière-plan
docker run -d nginx

# Donner un nom personnalisé
docker run --name mon-nginx nginx

# Porter forwarding (hôte:conteneur)
docker run -p 8080:80 nginx

# Mode interactif avec terminal
docker run -it ubuntu bash

# Monter un volume
docker run -v /chemin/local:/chemin/conteneur nginx

# Variables d'environnement
docker run -e VARIABLE=valeur nginx
```

### **🔧 Flags d'Inspection**
```bash
# Exec avec terminal interactif
docker exec -it mon-conteneur bash

# Voir tous les conteneurs (même arrêtés)
docker ps -a

# Format personnalisé pour ps
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Logs en temps réel
docker logs -f mon-conteneur
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Architecture Docker Complète**
```
┌─────────────────┐    ┌─────────────────┐
│   DOCKER CLI    │    │  DOCKER HUB     │
│   (Client)      │◄──►│  (Registry)     │
└─────────────────┘    └─────────────────┘
         │                       │
         ▼                       ▼
┌─────────────────────────────────────────┐
│           DOCKER DAEMON                 │
│                                         │
│  ┌─────────────┐  ┌─────────────────┐   │
│  │   IMAGES    │  │   CONTAINERS    │   │
│  │  (Read-Only)│  │  (Read-Write)   │   │
│  └─────────────┘  └─────────────────┘   │
│                                         │
│  ┌─────────────────────────────────────┐│
│  │         LINUX KERNEL                ││
│  │  - Namespaces (isolation)           ││
│  │  - cgroups (ressources)             ││
│  │  - Union FS (overlay2)              ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

### **Système de Fichiers Union (overlay2)**
```
Container Layer (Read-Write)    ← Couche modifiable
        ↑
Image Layers (Read-Only)        ← Couches image
        ↑
Base Image (Read-Only)          ← Image de base
```

### **Namespaces - Isolation des Ressources**
```bash
# Chaque conteneur a ses propres :
- PID Namespace   → Arbre de processus isolé
- NET Namespace   → Interfaces réseau isolées
- MNT Namespace   → Points de montage isolés
- UTS Namespace   → Hostname isolé
- IPC Namespace   → Mémoire partagée isolée
- User Namespace  → UIDs/GIDs isolés
```

### **cgroups - Contrôle des Ressources**
```bash
# Limitations par conteneur :
- memory          → Mémoire RAM maximale
- cpu             → Temps CPU alloué
- blkio           → E/S disque
- devices         → Accès aux périphériques
- freezer         → Geler/reprendre processus
```

---

## **🚀 EXERCICES RÉALISÉS**

### **1. Installation et Validation**
```bash
# Installation Docker
sudo apt install docker-ce

# Validation avec Hello World
docker run hello-world
# → "Hello from Docker!" ✅
```

### **2. Premier Serveur Web**
```bash
# Lancer nginx
docker run -d --name mon-nginx -p 8080:80 nginx

# Vérifier fonctionnement
curl http://localhost:8080
# → Page nginx par défaut ✅
```

### **3. Exploration Conteneur**
```bash
# Accéder au conteneur
docker exec -it mon-nginx bash

# Explorer le filesystem
ls -la /usr/share/nginx/html/
cat /etc/nginx/nginx.conf
exit
```

### **4. Personnalisation avec Volumes**
```bash
# Créer répertoire local
mkdir html
echo "<h1>Ma page personnalisée</h1>" > html/index.html

# Monter le volume
docker run -d --name nginx-perso -p 8081:80 -v $(pwd)/html:/usr/share/nginx/html nginx

# Vérifier personnalisation
curl http://localhost:8081
# → "Ma page personnalisée" ✅
```

### **5. Inspection et Métriques**
```bash
# Voir les logs
docker logs nginx-perso

# Inspecter la configuration
docker inspect nginx-perso

# Voir les statistiques ressources
docker stats nginx-perso

# Voir les processus
docker top nginx-perso
```

---

## **🎯 MÉTHODOLOGIE DOCKER**

### **Approche de Démarrage**
```bash
1. docker pull image          # Télécharger l'image
2. docker run options image   # Lancer le conteneur
3. docker ps                  # Vérifier l'état
4. docker logs conteneur      # Debugger si besoin
5. docker exec -it conteneur  # Explorer l'intérieur
6. docker stop conteneur      # Arrêter proprement
7. docker rm conteneur        # Nettoyer
```

### **Bonnes Pratiques**
- ✅ **Toujours** nommer les conteneurs (`--name`)
- ✅ **Toujours** mapper les ports explicitement (`-p`)
- ✅ **Toujours** vérifier les logs après démarrage
- ✅ **Toujours** nettoyer les conteneurs inutilisés
- ✅ **Toujours** utiliser des volumes pour les données persistantes

### **Gestion des Erreurs Courantes**
```bash
# Conteneur déjà existe
docker run --name existant nginx  # → ERROR
docker rm existant                # → SOLUTION

# Port déjà utilisé  
docker run -p 8080:80 nginx      # → ERROR
docker ps                         # → Voir qui utilise le port

# Image non trouvée
docker run image-inexistante     # → ERROR
docker pull image-inexistante    # → Vérifier le nom
```

---

## **📈 PROGRESSION DAY 11**

**✅ Compétences Acquises :**
- Compréhension approfondie de l'architecture Docker
- Maîtrise du cycle de vie complet des conteneurs
- Utilisation des commandes d'inspection et debug
- Gestion des volumes pour la persistance des données
- Compréhension des namespaces et cgroups Linux

**🎯 Mentalité DevOps :**
> Je ne déploie plus des applications, je lance des conteneurs  
> Mon environnement est reproductible et portable partout

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 11 / 100 ✅`**

**#Docker #Containers #DevOps #Virtualization #Linux #DockerBasics**

---

🐳
