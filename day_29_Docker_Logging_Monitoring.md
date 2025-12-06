# **JOUR 29 : LOGGING DOCKER & MONITORING DE BASE** 📊

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Logging Docker**
- **Drivers de logging intégrés** → JSON-file (défaut), journald, syslog, none
- **Drivers tiers avancés** → Fluentd, GELF, AWSlogs, Splunk
- **Format standardisé** → Tous les logs via STDOUT/STDERR
- **Séparation nette** → Logs d'application vs logs système Docker

### **📊 Métriques Containers Essentielles**
| Métrique          | Description                        | Importance               |
|-------------------|------------------------------------|--------------------------|
| **CPU Usage**     | Pourcentage CPU utilisé            | Performance application  |
| **Memory Usage**  | RAM consommée                      | Détection fuites mémoire |
| **Network I/O**   | Données réseau entrantes/sortantes | Performance réseau       |
| **Block I/O**     | Opérations disque lecture/écriture | Performance stockage     |
| **PIDs**          | Nombre de processus                | Stabilité container      |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion Logs Docker**
| Commande                      | FR                        | Usage                                 |
|-------------------------------|---------------------------|---------------------------------------|
| `docker logs CONTAINER`       | Afficher logs container   | `docker logs nginx --tail 50`         |
| `docker logs -f CONTAINER`    | Suivre logs en temps réel | `docker logs -f api-service`          |
| `docker inspect LOGS`         | Voir config logging       | `docker inspect --format LOGS`        |
| `docker stats`                | Métriques temps réel      | `docker stats --format table`         |

### **⚙️ Configuration Logging Drivers**
| Driver        | Configuration typique                             |
|---------------|---------------------------------------------------|
| **json-file** | `--log-driver json-file --log-opt max-size=10m`   |
| **journald**  | `--log-driver journald`                           |
| **local**     | `--log-driver local --log-opt compress=true`      |
| **none**      | `--log-driver none`                               |

---

## **📝 STRATÉGIES AVANCÉES**

### **Configuration Optimisée Production**
```yaml
# docker-compose.yml - Logging production ready
version: '3.8'
services:
  app:
    image: mon-app:latest
    logging:
      driver: "local"                    # Driver optimisé
      options:
        max-size: "10m"                  # Rotation automatique
        max-file: "5"                    # 5 fichiers max
        compress: "true"                 # Compression
        tag: "{{.ImageName}}|{{.Name}}"  # Tagging structuré
```

### **Rotation Logs Automatique**
```bash
# Conteneur avec rotation configurée
docker run -d \
  --name nginx-prod \
  --log-driver json-file \
  --log-opt max-size=10m \     # 10MB max par fichier
  --log-opt max-file=3 \       # Garder 3 fichiers max
  --log-opt compress=true \    # Compression des archives
  nginx:alpine
```

---

## **🚀 cAdvisor - MONITORING TEMPS RÉEL**

### **Architecture cAdvisor**
```
cAdvisor (Container Advisor)
├── Collecte métriques temps réel
├── Interface web intégrée
├── API REST complète
└── Intégration Prometheus
```

### **Déploiement cAdvisor**
```bash
# Lancement cAdvisor
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --privileged \
  gcr.io/cadvisor/cadvisor:latest
```

### **Interface cAdvisor**
- **URL** : http://localhost:8080
- **Containers** : Liste hiérarchique
- **Machine** : Métriques host système
- **Docker** : Informations Docker Engine
- **Subcontainers** : Vue détaillée par container

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Leçon 1 : Driver Local > JSON-File**
```bash
# ❌ ANCIENNE APPROCHE
docker run --log-driver json-file ...

# ✅ NOUVELLE APPROCHE (Docker 17.06+)
docker run --log-driver local \
  --log-opt max-size=10m \
  --log-opt compress=true
```

**Avantages driver local :**
- Compression intégrée
- Meilleures performances
- Rotation automatique
- Format optimisé

### **Leçon 2 : Journald pour systèmes Systemd**
```bash
# Intégration native avec systemd
docker run --log-driver journald \
  --log-opt tag="{{.ImageName}}/{{.Name}}"

# Consultation via journalctl
sudo journalctl -f CONTAINER_NAME=mon-container
```

### **Leçon 3 : Métriques sans outils externes**
```bash
# Docker stats natif
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# Événements Docker
docker events --filter 'type=container' --since '1h'
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Comparaison Logging Drivers**
```bash
# Test json-file avec rotation
docker run --log-driver json-file \
  --log-opt max-size=1m \
  --log-opt max-file=2 \
  alpine echo "Test rotation logs"

# Test journald intégration
docker run --log-driver journald \
  alpine echo "Test journald logging"

# Test driver local optimisé
docker run --log-driver local \
  --log-opt compress=true \
  alpine echo "Test local driver"
```

### **2. Monitoring avec cAdvisor**
```bash
# Déploiement cAdvisor
docker run -d --name=cadvisor --publish=8080:8080 gcr.io/cadvisor/cadvisor

# Conteneur de test
docker run -d --name load-test --cpus="0.5" alpine sh -c "while true; do stress-cpu; done"

# Visualisation
echo "Accédez à: http://localhost:8080"
echo "Observez CPU/Memory de 'load-test'"
```

### **3. Gestion Espace Logs**
```bash
# Vérification espace logs
docker system df

# Nettoyage logs obsolètes
docker container prune
docker image prune

# Recherche logs volumineux
sudo find /var/lib/docker/containers -name "*.log" -size +100M
```

### **4. Script Automatisation Monitoring**
```bash
#!/bin/bash
# monitor-containers.sh
echo "=== MONITORING DOCKER ==="
echo "1. Métriques temps réel:"
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}"

echo "2. Top 5 containers CPU:"
docker stats --no-stream --format "{{.CPUPerc}}\t{{.Name}}" | sort -hr | head -5

echo "3. Logs récents erreurs:"
docker ps -q | xargs -I {} sh -c 'docker logs {} --tail 5 2>&1 | grep -i error'
```

---

## **🎯 BONNES PRATIQUES PRODUCTION**

### **Checklist Logging Production**
- [ ] **Driver local** activé avec compression
- [ ] **Rotation configurée** (max-size: 10-100MB, max-file: 3-10)
- [ ] **Tagging structuré** pour identification
- [ ] **Monitoring espace disque** logs
- [ ] **Nettoyage automatique** configuré
- [ ] **Journald** sur systèmes systemd
- [ ] **STDOUT/STDERR** uniquement pour logs application

### **Checklist Monitoring Base**
- [ ] **cAdvisor** déployé pour métriques
- [ ] **Docker stats** intégré aux scripts
- [ ] **Alertes CPU/Memory** configurées
- [ ] **Health checks** Docker activés
- [ ] **Logs erreurs** monitorés
- [ ] **Événements Docker** tracés

### **Conventions Logging**
```bash
# Format recommandé
{"timestamp":"ISO-8601","level":"INFO","service":"app","message":"..."}

# Tags Docker recommandés
tag="{{.ImageName}}|{{.Name}}|{{.ID}}"
```

---

## **📈 PROGRESSION JOUR 29**

### **✅ Compétences Acquises :**
- **Maîtrise drivers logging** : json-file, journald, local, syslog
- **Configuration rotation** : Gestion automatique espace logs
- **Déploiement cAdvisor** : Monitoring containers temps réel
- **Métriques Docker natives** : CPU, mémoire, réseau, disque
- **Best practices production** : Configurations optimisées

### **🎯 Mentalité DevOps :**
> Mes logs ne sont plus des fichiers obscurs  
> Ils sont structurés, rotatifs et monitorés  
> Mes containers ne sont plus des boîtes noires  
> Leurs métriques sont visibles et analysables

### **🔗 Architecture Implémentée :**
```
Infrastructure Docker Jour 29
├── Logging
│   ├── Drivers configurés (local/journald)
│   ├── Rotation automatique
│   └── Compression activée
└── Monitoring
    ├── cAdvisor déployé
    ├── Métriques temps réel
    └── Interface web accessible
```

### **🚀 Prochaines Étapes (Jour 30) :**
- **Centralisation logs** avec Fluentd/Elasticsearch
- **Monitoring avancé** avec Prometheus
- **Dashboard Grafana** visualisation complète
- **Alerting automatique** seuils configurables

---

**📊 Progress: `Jour 29 / 100 ✅`**

**#DockerLogging #LogManagement #ContainerMonitoring #cAdvisor #DevOps #Observability**


