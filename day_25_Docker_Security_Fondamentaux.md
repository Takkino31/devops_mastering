# **DAY 25 - DOCKER SECURITY: FONDAMENTAUX & SCANNING** 🛡️

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture de Sécurité Docker**
- **User Namespaces** → Isolation utilisateurs hôte/conteneur
- **Linux Capabilities** → Droits granulaires vs root complet
- **Principe du moindre privilège** → Minimum nécessaire

### **🔍 Scanning Sécurité**
| Outil                 | Usage                     | Avantage          |
|-----------------------|---------------------------|-------------------|
| **Trivy**             | Scanner vulnérabilités    | Rapide, complet   |
| **Capsh**             | Vérifier capabilities     | Intégré Linux     |
| **Docker inspect**    | Analyse configuration     | Native Docker     |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Scanning Sécurité**
| Commande              | FR                    | EN                    | Usage                      |
|-----------------------|-----------------------|-----------------------|----------------------------|
| `trivy image nom:tag` | Scanner image         | **SCAN image**        | `trivy image nginx:latest` |
| `trivy config .`      | Scanner Dockerfile    | **SCAN config**       | `trivy config .`           |
| `docker inspect`      | Inspecter conteneur   | **INSPECT container** | `docker inspect nom`       |

### **🛡️ Sécurisation Conteneurs**
| Commande         | FR                     | EN                    | Usage                                         |
|------------------|------------------------|-----------------------|-----------------------------------------------|
| `--user uid:gid` | Utilisateur spécifique | **SPECIFIC user**     | `docker run --user 1000:1000`                 |
| `--cap-drop ALL` | Supprimer capabilities | **DROP capabilities** | `--cap-drop ALL --cap-add NET_BIND_SERVICE`   |
| `--security-opt` | Options sécurité       | **SECURITY options**  | `--security-opt no-new-privileges`            |

---

## **📝 CONFIGURATION SÉCURISÉE**

### **Dockerfile Non-Root**
```dockerfile
FROM ubuntu:latest

# Créer utilisateur/groupe non-root
RUN groupadd -r appgroup && \
    useradd -r -g appgroup -s /bin/false appuser

# Installer avec permissions minimales
RUN apt update && apt install -y package && \
    apt clean && rm -rf /var/lib/apt/lists/*

# Changer propriétaire fichiers
RUN chown -R appuser:appgroup /app

# Basculer vers utilisateur non-root
USER appuser

# Exposer ports
EXPOSE 8080

CMD ["mon-application"]
```

### **Capabilities Linux Essentielles**
```yaml
# Capabilities courantes
NET_BIND_SERVICE:  # Lier ports < 1024
SETGID, SETUID:    # Changer GID/UID  
DAC_OVERRIDE:      # Ignorer permissions
SYS_ADMIN:         # Administration système
NET_RAW:           # Paquets réseau bruts
```

---

## **🚀 STRATÉGIES DE SÉCURITÉ**

### **Approche en Couches**
| Couche        | Technique         | Impact                    |
|---------------|-------------------|---------------------------|
| **Image**     | Scanning Trivy    | Vulnérabilités connues    |
| **Build**     | USER non-root     | Réduction privilèges      |
| **Run**       | Capabilities      | Droits granulaires        |
| **Réseau**    | Policies          | Isolation                 |

### **Scanning avec Trivy**
```bash
# Scanner complet
trivy image mon-image:tag

# Focus risques élevés
trivy image --severity HIGH,CRITICAL mon-image

# Ignorer non-fixables
trivy image --ignore-unfixed mon-image

# Format rapport
trivy image --format json mon-image > rapport.json
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Problème Root par Défaut :**
```bash
# ❌ DANGEREUX - Root complet
docker run nginx:latest
# → UID 0 = accès root sur hôte

# ✅ SÉCURISÉ - Non-root
docker run --user 1000:1000 nginx:latest
# → UID 1000 = utilisateur limité
```

### **Réduction Capabilities :**
```bash
# Avant - Toutes les capabilities
docker run nginx:latest
# → 38 capabilities Linux

# Après - Minimum nécessaire
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx:latest
# → 1 capability uniquement
```

### **Scanning Automatique :**
```bash
# Intégration CI/CD
trivy image --exit-code 1 --severity HIGH,CRITICAL mon-image
# → Échec build si vulnérabilités critiques
```

### **Dockerfile Sécurisé :**
```dockerfile
# AVANT - Risques
FROM node:latest
COPY . .
RUN npm install
CMD ["node", "app.js"]

# APRÈS - Sécurisé
FROM node:16-alpine
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001
COPY --chown=appuser:appgroup . .
USER appuser
CMD ["node", "app.js"]
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Installation et Usage Trivy**
```bash
# Installation
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scanner image test
trivy image nginx:latest
# → Liste vulnérabilités CVE

# Scanner avec filtrage
trivy image --severity CRITICAL nginx:latest
# → Uniquement critiques
```

### **2. Création Conteneur Non-Root**
```dockerfile
# Dockerfile sécurisé
FROM nginx:alpine
RUN addgroup -g 1001 -S nginxgroup && \
    adduser -S nginxuser -u 1001
USER nginxuser
```

```bash
# Build et test
docker build -t nginx-secure .
docker run -d -p 8080:80 nginx-secure
docker exec nginx-secure whoami
# → nginxuser (non root!)
```

### **3. Gestion Capabilities**
```bash
# Conteneur avec capabilities minimales
docker run -d \
  --name nginx-minimal \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  -p 80:80 \
  nginx:alpine

# Vérification
docker exec nginx-minimal capsh --print
# → Current: cap_net_bind_service=eip
```

### **4. Scan Intégré Dockerfile**
```bash
# Scanner configuration
trivy config .
# → Vulnérabilités configuration

# Intégration build
docker build -t mon-app .
trivy image --exit-code 1 --severity HIGH,CRITICAL mon-app
# → Build échoue si vulnérabilités critiques
```

---

## **🎯 CHECKLIST SÉCURITÉ JOUR 25**

### **Dockerfile :**
- ✅ **Image de base** officielle et versionnée
- ✅ **Utilisateur non-root** créé et utilisé
- ✅ **Permissions fichiers** correctes
- ✅ **Caches nettoyés** après installation
- ✅ **Ports exposés** explicitement

### **Exécution :**
- ✅ **Scan Trivy** avant utilisation
- ✅ **Capabilities réduites** au minimum
- ✅ **Utilisateur non-root** spécifié
- ✅ **Ressources limitées** (memory, CPU)
- ✅ **Volumes read-only** si possible

### **Monitoring :**
- ✅ **Scan régulier** des images
- ✅ **Journalisation** des actions sécurité
- ✅ **Audit configurations** périodique
- ✅ **Mise à jour** patches sécurité

---

## **📈 PROGRESSION DAY 25**

**✅ Compétences Acquises :**
- Maîtrise du scanning d'images avec Trivy
- Création de conteneurs non-root sécurisés
- Gestion des Linux Capabilities granulaires
- Implémentation du principe du moindre privilège
- Documentation des pratiques sécurité

**🎯 Mentalité DevSecOps :**
> Mes conteneurs ne tournent plus en root par défaut  
> Chaque privilege est explicitement accordé et justifié

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 25 / 100 ✅`**

**#DockerSecurity #Trivy #NonRoot #LinuxCapabilities #DevSecOps #ContainerSecurity**

---
