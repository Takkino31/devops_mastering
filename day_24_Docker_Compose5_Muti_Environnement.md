# **DAY 24 - DOCKER COMPOSE MULTI-FICHIERS & HEALTH CHECKS AVANCÉS** 🏗️⚡

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Multi-Fichiers**
- **Séparation des préoccupations** → Base, Dev, Prod, Outils
- **Fusion intelligente** → Combinaison automatique/manuelle
- **Maintenance ciblée** → Modification sans affecter tout

### **🩺 Health Checks Avancés**
| Type              | Outils                | Usage                 |
|-------------------|-----------------------|-----------------------|
| **Système**       | curl, wget, socket    | Vérification ports    |
| **Métier**        | Python/Flask          | Logique applicative   |
| **Ressources**    | df, /proc             | CPU, mémoire, disque  |
| **Dépendances**   | ping, connect         | Services externes     |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion Multi-Fichiers**
| Commande | FR | EN | Usage |
|----------|-------|---------|-------|
| `docker-compose -f file1 -f file2`    | Combinaison fichiers  | **COMPOSE multiple**  | `-f base.yml -f prod.yml` |
| `docker-compose config`               | Voir fusion           | **CONFIG preview**    | `docker-compose config`   |
| `docker-compose --profile`            | Activer profiles      | **PROFILES**          | `--profile tools`         |

### **📊 Monitoring & Health Checks**
| Commande                      | FR                | EN                    | Usage                     |
|-------------------------------|-------------------|-----------------------|---------------------------|
| `docker inspect --format`     | Inspecter santé   | **INSPECT health**    | Format personnalisé       |
| `docker logs --tail`          | Voir logs erreurs | **LOG errors**        | `docker logs --tail 10`   |
| `docker stats --no-stream`    | Statistiques      | **STATS**             | Ressources conteneurs     |

---

## **📝 STRUCTURE MULTI-FICHIERS**

### **Hiérarchie des Fichiers**
```
docker-compose.yml           # Configuration de base
├── docker-compose.override.yml  # Développement (auto)
├── docker-compose.prod.yml      # Production  
└── docker-compose.tools.yml     # Outils optionnels
```

### **Règles de Fusion**
```yaml
# docker-compose.yml
services:
  backend:
    image: python:3.9
    ports: ["5000"]

# docker-compose.override.yml  
services:
  backend:
    volumes: ["./backend:/app"]
    ports: ["5000:5000"]  # REMPLACE la liste

# Résultat fusionné:
services:
  backend:
    image: python:3.9      # Gardé
    volumes: ["./backend:/app"]  # Ajouté
    ports: ["5000:5000"]   # Remplacé
```

---

## **🚀 HEALTH CHECKS AVANCÉS**

### **Types de Health Checks**
```yaml
# Simple (port/service)
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/"]
  interval: 30s

# Avancé (script shell)
healthcheck:
  test: >
    sh -c '
    curl -f http://localhost:5000/health/simple || exit 1 &&
    curl -s http://localhost:5000/health/advanced | grep -q "healthy"
    '
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

### **Health Check Métier (Python)**
```python
@app.route('/health/advanced')
def advanced_health():
    """Vérification complète avec métriques"""
    checks = {
        "system": check_system_resources(),
        "dependencies": check_dependencies(),
        "status": "healthy"
    }
    
    # Vérifier chaque composant
    if any("ERROR" in str(v) for v in checks["system"].values()):
        checks["status"] = "degraded"
    
    return jsonify(checks)
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Fusion Multi-Fichiers :**
```bash
# Développement (override automatique)
docker-compose up -d
# → Fusionne: docker-compose.yml + docker-compose.override.yml

# Production (manuel)
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
# → Fusionne les deux fichiers spécifiés

# Avec outils
docker-compose -f docker-compose.yml -f docker-compose.prod.yml -f docker-compose.tools.yml --profile tools up -d
```

### **Health Checks Intelligents :**
```yaml
services:
  database:
    healthcheck:
      # PostgreSQL ready
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  backend:
    healthcheck:
      # API répond ET métriques OK
      test: >
        sh -c '
        curl -f http://localhost:5000/health/simple || exit 1 &&
        curl -s http://localhost:5000/health/advanced | grep -q "healthy"
        '
      interval: 30s
      timeout: 10s
      retries: 3
```

### **Outils de Monitoring Linux :**
```bash
#!/bin/bash
# scripts/monitor.sh
echo "📦 Conteneurs:"
docker ps --format "table {{.Names}}\t{{.Status}}"

echo "🩺 Santé:"
for container in $(docker ps --format "{{.Names}}"); do
    health=$(docker inspect --format='{{.State.Health.Status}}' "$container")
    echo "  $container: $health"
done
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Architecture Multi-Fichiers Complète**
```yaml
# docker-compose.yml - Base
services:
  database:
    image: postgres:13
    healthcheck: [...]

  backend:
    build: ./backend
    depends_on:
      database:
        condition: service_healthy

# docker-compose.override.yml - Dev  
services:
  backend:
    ports: ["5000:5000"]
    volumes: ["./backend:/app"]
    environment:
      DEBUG: "true"

# docker-compose.prod.yml - Production
services:
  backend:
    environment:
      DEBUG: "false"
    deploy:
      restart_policy:
        condition: on-failure
```

### **2. Health Checks Avancés Implémentés**
```python
# Vérifications système
def check_system_resources():
    return {
        "memory": check_memory(),
        "disk": check_disk_space(),
        "load": check_cpu_load()
    }

# Vérifications dépendances
def check_dependencies():
    return [
        {"database": check_port("database", 5432)},
        {"cache": check_port("redis", 6379)}
    ]
```

### **3. Script de Déploiement Automatisé**
```bash
#!/bin/bash
# deploy.sh
ENVIRONMENT=${1:-dev}

case $ENVIRONMENT in
  dev)  docker-compose up -d ;;
  prod) docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d ;;
  tools) docker-compose -f docker-compose.yml -f docker-compose.tools.yml --profile tools up -d ;;
esac
```

### **4. Monitoring avec Outils Linux**
```bash
# Affichage état conteneurs
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Vérification santé
for container in $(docker ps --format "{{.Names}}"); do
    health=$(docker inspect --format='{{.State.Health.Status}}' "$container")
    echo "$container: $health"
done

# Statistiques ressources
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

## **🎯 BONNES PRATIQUES PRODUCTION**

### **Checklist Multi-Fichiers :**
- ✅ **Fichier de base** minimal et stable
- ✅ **Override développement** pour productivité
- ✅ **Configuration production** séparée
- ✅ **Outils optionnels** avec profiles
- ✅ **Variables d'environnement** externalisées

### **Health Checks Robustes :**
```yaml
healthcheck:
  # Tests réalistes
  test: ["CMD", "service-specific-check"]
  
  # Intervalles adaptés
  interval: 30s
  timeout: 10s
  
  # Tentatives suffisantes
  retries: 3
  
  # Période démarrage
  start_period: 40s
  start_interval: 5s
```

### **Organisation Projet :**
```
mon-projet/
├── docker-compose.yml
├── docker-compose.override.yml
├── docker-compose.prod.yml
├── docker-compose.tools.yml
├── backend/
├── frontend/
├── scripts/
│   └── monitor.sh
└── deploy.sh
```

---

## **📈 PROGRESSION DAY 24**

**✅ Compétences Acquises :**
- Maîtrise de la configuration multi-fichiers Docker Compose
- Implémentation de health checks avancés avec outils Linux
- Création de scripts de monitoring et déploiement
- Organisation modulaire des environnements
- Gestion des dépendances avec conditions

**🎯 Mentalité DevOps :**
> Je ne configure plus des applications monolithiques  
> Je construis des systèmes modulaires avec monitoring intégré

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 24 / 100 ✅`**

**#DockerCompose #MultiFile #HealthChecks #DevOps #Linux #Monitoring #ProductionReady**

---
