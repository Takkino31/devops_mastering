# **DAY 23 - DOCKER COMPOSE PROFILES & CONFIGURATION AVANCÉE** 🎯

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Docker Compose Profiles**
- **Activation conditionnelle** de services
- **Organisation logique** par fonctionnalité
- **Environnements multiples** dans un seul fichier

### **⚡ Configuration Avancée**
| Fonctionnalité    | Usage                 | Avantage                  |
|-------------------|-----------------------|---------------------------|
| **Profiles**      | `profiles: ["dev"]`   | Services optionnels       |
| **Ancres YAML**   | `&app-config`         | Références internes       |
| **Merge**         | `<<: *app-config`     | Héritage de configuration |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion des Profiles**
| Commande                      | FR                    | EN                        | Usage                     |
|-------------------------------|-----------------------|---------------------------|---------------------------|
| `docker-compose --profile`    | Activer profile       | **PROFILE activation**    | `--profile dev`           |
| `docker-compose config`       | Voir configuration    | **CONFIG preview**        | `docker-compose config`   |
| `docker-compose ps`           | Services actifs       | **SERVICES status**       | `docker-compose ps`       |

### **🎮 Scénarios d'Utilisation**
| Scénario              | Commande                              | Résultat                  |
|-----------------------|---------------------------------------|---------------------------|
| **Développement**     | `--profile dev`                       | Services dev uniquement   |
| **Production**        | `--profile prod`                      | Services production       |
| **Avec monitoring**   | `--profile dev --profile monitoring`  | Combinaison               |

---

## **📝 SYNTAXE AVANCÉE COMPOSE**

### **Ancres et Références YAML**
```yaml
# Définition d'une ancre
x-app-config: &app-config
  restart: unless-stopped
  logging:
    driver: json-file

# Utilisation de l'ancre
services:
  frontend:
    <<: *app-config          # Merge la configuration
    image: nginx:alpine
    ports: ["80:80"]
```

### **Profiles en Action**
```yaml
services:
  # Services de base
  frontend:
    image: nginx:alpine
    profiles: ["frontend"]
  
  # Services de développement
  phpmyadmin:
    image: phpmyadmin:latest
    profiles: ["dev-tools"]
  
  # Services de production
  backend-prod:
    image: python:3.9-slim
    profiles: ["prod"]
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Limitation des Aliases YAML**
```yaml
# ❌ NE FONCTIONNE PAS - Aliases entre fichiers
# docker-compose.base.yml
x-app-config: &app-config
  restart: unless-stopped

# docker-compose.dev.yml  
services:
  frontend:
    <<: *app-config  # ERREUR: alias non défini

# ✅ FONCTIONNE - Aliases dans même fichier
# docker-compose.yml
x-app-config: &app-config
  restart: unless-stopped

services:
  frontend:
    <<: *app-config  # OK: même fichier
```

### **Architecture avec Profiles**
```yaml
version: '3.8'

x-app-config: &app-config
  restart: unless-stopped
  logging:
    driver: json-file

services:
  # === DÉVELOPPEMENT ===
  frontend-dev:
    <<: *app-config
    image: nginx:alpine
    ports: ["3000:80"]
    profiles: ["dev"]

  backend-dev:
    <<: *app-config
    image: python:3.9-slim  
    ports: ["5000:5000"]
    profiles: ["dev"]

  # === PRODUCTION ===
  frontend-prod:
    <<: *app-config
    image: nginx:alpine
    ports: ["80:80"]
    profiles: ["prod"]

  backend-prod:
    <<: *app-config
    image: python:3.9-slim
    ports: ["5000:5000"]
    profiles: ["prod"]
```

### **Commandes par Scénario**
```bash
# Développement
docker-compose --profile dev up -d

# Production
docker-compose --profile prod up -d

# Mixte
docker-compose --profile dev --profile monitoring up -d

# Vérification
docker-compose ps
docker-compose config --services
```

---

## **🚀 STRATÉGIES D'IMPLÉMENTATION**

### **Approche Single-File (Recommandée)**
```yaml
# Un fichier, tous les profiles
version: '3.8'

x-common-config: &common-config
  restart: unless-stopped
  healthcheck:
    interval: 30s
    timeout: 10s

services:
  app-dev: 
    <<: *common-config
    profiles: ["dev"]
    
  app-prod:
    <<: *common-config  
    profiles: ["prod"]
```

### **Organisation par Fonctionnalité**
```yaml
services:
  # Core application
  frontend: 
    profiles: ["frontend"]
  backend:
    profiles: ["backend"]
  
  # Development tools  
  phpmyadmin:
    profiles: ["dev-tools"]
  redis-commander:
    profiles: ["dev-tools"]
  
  # Monitoring
  prometheus:
    profiles: ["monitoring"]
```

---

## 🛠️ EXERCICES RÉALISÉS

### **1. Configuration avec Profiles**
```yaml
# Services organisés par profil
services:
  database:
    image: postgres:13
    profiles: ["database"]
  
  redis:
    image: redis:alpine  
    profiles: ["cache"]
  
  phpmyadmin:
    image: phpmyadmin:latest
    profiles: ["dev-tools"]
```

### **2. Héritage avec Anchors**
```yaml
# Configuration partagée
x-app-config: &app-config
  restart: unless-stopped
  logging:
    driver: json-file

# Application aux services
services:
  frontend:
    <<: *app-config
    image: nginx:alpine
```

### **3. Commandes de Gestion**
```bash
# Activation sélective
docker-compose --profile database --profile cache up -d

# Vérification
docker-compose ps
docker-compose config --services

# Arrêt ciblé
docker-compose --profile dev-tools down
```

### **4. Résolution de Problème**
**Problème :** Aliases YAML entre fichiers  
**Cause :** Chaque fichier Compose est parsé indépendamment  
**Solution :** Utiliser un seul fichier ou dupliquer les aliases

---

## **🎯 BONNES PRATIQUES PROFILES**

### **Checklist Organisation :**
- ✅ **Un fichier principal** pour les aliases YAML
- ✅ **Profiles logiques** (dev, prod, monitoring, tools)
- ✅ **Configuration partagée** avec anchors
- ✅ **Documentation** des scénarios d'usage

### **Conventions de Nommage :**
```yaml
profiles: ["dev"]           # Développement
profiles: ["prod"]          # Production  
profiles: ["monitoring"]    # Monitoring
profiles: ["dev-tools"]     # Outils développement
profiles: ["cache"]         # Services de cache
```

### **Sécurité :**
```yaml
# Ne pas exposer les ports de base en production
database:
  profiles: ["dev"]     # Port exposé en dev
  ports: ["5432:5432"]

database-prod:
  profiles: ["prod"]    # Pas de port en prod
  # ports: []          # Accès réseau interne uniquement
```

---

## **📈 PROGRESSION DAY 23**

**✅ Compétences Acquises :**
- Maîtrise des Docker Compose Profiles
- Utilisation des ancres et références YAML
- Organisation d'architectures multi-environnements
- Commandes avancées d'activation sélective
- Résolution des limitations d'aliases

**🎯 Mentalité DevOps :**
> Je ne déploie plus tout ou rien  
> J'active précisément les services dont j'ai besoin

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 23 / 100 ✅`**

**#DockerCompose #Profiles #YAML #DevOps #ConfigurationManagement**

---
