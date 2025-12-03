# **DAY 26 - DOCKER SECURITY: AVANCÉ & HARDENING** 🔒

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Sécurité Avancée Docker**
- **Seccomp Profiles** → Filtrage appels système noyau
- **AppArmor/SELinux** → Mandatory Access Control
- **Image Hardening** → Réduction surface attaque

### **🔍 Techniques de Hardening**
| Technique          | Usage                | Protection                |
|--------------------|----------------------|---------------------------|
| **Seccomp**        | Filtrage syscalls    | Escalation privilèges     |
| **AppArmor**       | Profils par app      | Accès fichiers/réseau     |
| **Minimal Images** | Alpine/Distroless    | Réduction vulnérabilités  |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Sécurité Avancée**
| Commande                  | FR                | EN                    | Usage                     |
|---------------------------|-------------------|-----------------------|---------------------------|
| `--security-opt seccomp`  | Profil Seccomp    | **SECCOMP profile**   | `seccomp=profile.json`    |
| `--security-opt apparmor` | Profil AppArmor   | **APPARMOR profile**  | `apparmor=my-profile`     |
| `--read-only`             | FS lecture seule  | **READ-ONLY FS**      | `--read-only`             |

### **📊 Audit & Monitoring**
| Commande                      | FR                | EN                    | Usage             |
|-------------------------------|-------------------|-----------------------|-------------------|
| `grep Seccomp /proc/status`   | Vérifier Seccomp  | **CHECK Seccomp**     | Dans conteneur    |
| `aa-status`                   | Statut AppArmor   | **APPARMOR status**   | Sur hôte          |
| `dmesg \| grep seccomp`       | Debug kernel      | **DEBUG Seccomp**     | Erreurs syscalls  |

---

## **📝 PROFILS SECURITY AVANCÉS**

### **Profil Seccomp Personnalisé**
```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": [
        "accept", "bind", "close", "connect", "execve",
        "fstat", "ioctl", "listen", "mmap", "open",
        "read", "recvfrom", "sendto", "socket", "write"
      ],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

### **Utilisation avec Docker**
```bash
# Lancer avec profil Seccomp
docker run -d \
  --security-opt seccomp=nginx-seccomp.json \
  -p 80:80 \
  nginx:alpine

# Profil par défaut Docker
docker run --security-opt seccomp=default nginx:alpine

# Désactiver Seccomp (DANGEREUX)
docker run --security-opt seccomp=unconfined nginx:alpine
```

---

## **🚀 STRATÉGIES DE HARDENING**

### **Approche Défense en Profondeur**
| Couche | Technique | Exemple |
|--------|-----------|---------|
| **Image** | Scanning | Trivry, Grype |
| **Build** | Non-root | USER dans Dockerfile |
| **Runtime** | Seccomp | Filtrage syscalls |
| **Host** | AppArmor | Profils système |

### **Hardening Progressif**
1. **Base** → Non-root + capabilities
2. **Intermediaire** → Seccomp par défaut
3. **Avancé** → Profils personnalisés
4. **Expert** → AppArmor + SELinux

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Seccomp en Pratique :**
```bash
# ❌ Crash - Profil trop restrictif
docker run --security-opt seccomp=too-strict.json nginx
# → Container exited immediately

# ✅ Fonctionnel - Profil adapté
docker run --security-opt seccomp=nginx-profile.json nginx
# → Container running

# Debug avec dmesg
sudo dmesg | grep seccomp | tail -5
# → Voir les syscalls bloqués
```

### **Images Minimales :**
```dockerfile
# ❌ Lourd et vulnérable
FROM ubuntu:latest
RUN apt update && apt install -y python3 python3-pip
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]

# ✅ Léger et sécurisé
FROM python:3.11-slim
RUN adduser --disabled-password --gecos '' appuser
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=appuser:appuser . .
USER appuser
CMD ["python", "app.py"]
```

### **AppArmor Basics :**
```bash
# Vérifier statut
sudo aa-status

# Profil par défaut Docker
docker run --security-opt apparmor=docker-default nginx:alpine

# Charger profil personnalisé
sudo apparmor_parser -r /etc/apparmor.d/my-profile
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Création Profil Seccomp**
```json
// nginx-seccomp.json - Profil Nginx minimal
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["accept", "bind", "close", "listen", "socket"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

```bash
# Test du profil
docker run -d --name nginx-secure \
  --security-opt seccomp=nginx-seccomp.json \
  -p 8080:80 nginx:alpine

# Vérification
curl -I http://localhost:8080
docker exec nginx-secure grep Seccomp /proc/self/status
```

### **2. Hardening d'Image Existante**
```dockerfile
# Refactoring sécurisé
FROM node:18-alpine

# Non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001

# Installation sécurisée
COPY package*.json ./
RUN npm ci --only=production --audit=false

# Permissions
COPY --chown=appuser:appgroup . .
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s CMD node healthcheck.js

CMD ["node", "server.js"]
```

### **3. Script d'Audit Sécurité**
```bash
#!/bin/bash
# security-audit.sh

echo "🔍 Audit Sécurité Docker"
echo "======================="

# 1. Images running
echo "📦 Conteneurs actifs:"
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"

# 2. Non-root check
echo ""
echo "👤 Vérification non-root:"
for container in $(docker ps -q); do
  user=$(docker exec $container whoami 2>/dev/null || echo "unknown")
  echo "  $(docker inspect --format '{{.Name}}' $container): $user"
done

# 3. Seccomp status
echo ""
echo "🛡️ Statut Seccomp:"
for container in $(docker ps -q); do
  status=$(docker exec $container grep Seccomp /proc/self/status 2>/dev/null || echo "not running")
  echo "  $(docker inspect --format '{{.Name}}' $container): $status"
done
```

### **4. Checklist Production**
```markdown
# Checklist Sécurité Production

## ✅ Images
- [ ] Base image minimaliste
- [ ] Version spécifique (pas latest)
- [ ] Scanning vulnérabilités
- [ ] Signatures vérifiées

## ✅ Build
- [ ] Utilisateur non-root
- [ ] Capabilities réduites
- [ ] Health check configuré
- [ ] Caches nettoyés

## ✅ Runtime
- [ ] Seccomp activé
- [ ] AppArmor/SELinux
- [ ] Resources limitées
- [ ] FS read-only si possible

## ✅ Monitoring
- [ ] Logs centralisés
- [ ] Alertes sécurité
- [ ] Audit régulier
- [ ] Mises à jour automatiques
```

### **5. Debug Seccomp**
```bash
# Container crashé à cause de Seccomp
docker logs nginx-crashed

# Voir erreurs kernel
sudo dmesg | grep -A5 -B5 seccomp

# Mode debug
docker run -it --rm \
  --security-opt seccomp=unconfined \
  nginx:alpine sh
# → Tester commandes, puis ajouter au profil
```

---

## **🎯 BONNES PRATIQUES PRODUCTION**

### **Approche Progressive :**
1. **Start** → Non-root + scanning
2. **Intermediate** → Seccomp default
3. **Advanced** → Profils personnalisés
4. **Expert** → Full MAC (AppArmor/SELinux)

### **Gestion des Profils :**
```bash
# Profils par application
/profiles/
├── nginx-seccomp.json
├── postgres-seccomp.json
├── redis-seccomp.json
└── node-seccomp.json

# Versionning
git add profiles/
git commit -m "Add security profiles"
```

### **CI/CD Sécurité :**
```yaml
# .gitlab-ci.yml ou GitHub Actions
security-scan:
  stage: test
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE
    - docker run --security-opt seccomp=profiles/$APP.json $IMAGE
    - ./security-audit.sh
```

---

## **📈 PROGRESSION DAY 26**

**✅ Compétences Acquises :**
- Création et gestion de profils Seccomp personnalisés
- Hardening d'images Docker existantes
- Implémentation de stratégies défense en profondeur
- Debug et résolution problèmes sécurité
- Checklist sécurité production complète

**🎯 Mentalité Security-First :**
> Je ne déploie plus des conteneurs, je déploie des forteresses
> Chaque couche de sécurité est délibérée et testée

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 26 / 100 ✅`**

**#DockerSecurity #Seccomp #AppArmor #Hardening #DevSecOps #ProductionSecurity**

---
