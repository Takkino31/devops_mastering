# **JOUR 75 : BUILD DOCKER OPTIMISÉ ET REGISTRY**

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Docker Multi-Stage pour CI/CD**
- **Builder stage** : Environnement complet avec dépendances de développement
- **Production stage** : Image minimale avec seulement le nécessaire
- **Séparation des responsabilités** : Un stage = une fonction
- **Gain de taille** : 500MB → 150MB (Node.js)

### **⚡ Docker Layer Caching**
- **Ordre des couches** : Ce qui change peu en premier, ce qui change souvent en dernier
- **Cache GHA** : Persistance entre les runs GitHub Actions
- **Cache Registry** : Réutilisation entre branches et repositories
- **BuildKit** : Parallélisation et montage intelligent

### **🖥️ Build Multi-Architecture**
- **amd64** : Serveurs traditionnels (Intel/AMD)
- **arm64** : Apple Silicon, AWS Graviton
- **arm/v7** : Raspberry Pi, IoT
- **Buildx + QEMU** : Émulation transparente

### **🏷️ Stratégies de Tagging**
- **SHA long** : Unique et reproductible
- **Branche** : Développement continu
- **SemVer** : Versions sémantiques
- **Latest** : Dernière version stable

### **🔒 Sécurité des Images**
- **SBOM** : Inventaire des composants (Software Bill of Materials)
- **Trivy** : Scan de vulnérabilités dans le pipeline
- **Non-root user** : Moindre privilège dans le conteneur
- **Healthcheck** : Détection d'état pour l'orchestration

---

## **📊 Architecture Docker Industrielle Implémentée**

### **Dockerfile Multi-Stage Optimisé**
```dockerfile
# STAGE 1: Build (avec dev dependencies)
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./     # Change rarement → en premier
RUN npm ci                # Cache longue durée
COPY . .                  # Change souvent → en dernier
RUN npm test && npm run build

# STAGE 2: Production (minimal)
FROM node:18-alpine AS production
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # Seulement les dépendances runtime
COPY --from=builder /app/dist ./dist
USER nodejs
HEALTHCHECK CMD node -e "require('http').get('http://localhost:3000/health')"
CMD ["node", "dist/index.js"]
```

### **Pipeline Docker avec Buildx**
```yaml
# Configuration Buildx
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
  with:
    driver-opts: image=moby/buildkit:latest,network=host

# Cache GitHub Actions
cache-from: type=gha
cache-to: type=gha,mode=max

# Tags intelligentes
tags: |
  type=sha,format=long      # sha-a1b2c3d4...
  type=ref,event=branch     # main, develop
  type=semver,pattern={{version}} # 1.2.3
```

### **Build Multi-Architecture**
```yaml
# Matrix sur les plateformes
strategy:
  matrix:
    platform:
      - linux/amd64
      - linux/arm64
      - linux/arm/v7

# QEMU pour émulation
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

# Build pour une plateforme
- uses: docker/build-push-action@v5
  with:
    platforms: ${{ matrix.platform }}
    push: true
    tags: app:${{ github.sha }}-${{ matrix.platform }}
```

### **Manifeste Multi-Arch**
```bash
# Fusionner les images en un manifeste unique
docker buildx imagetools create \
  --tag ghcr.io/user/app:latest \
  ghcr.io/user/app:latest-linux/amd64 \
  ghcr.io/user/app:latest-linux/arm64 \
  ghcr.io/user/app:latest-linux/arm/v7

# Une seule tag, architecture détectée automatiquement
docker pull ghcr.io/user/app:latest
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. La Révolution Multi-Stage**
**Problème résolu :**
- Avant : Une seule image avec TOUT (dev dependencies, outils de build, sources)
- Taille : 500MB-1GB
- Surface d'attaque : Énorme

**Après multi-stage :**
- Image production : 150MB (Node.js alpine)
- Dev dependencies : Dans le stage builder, pas en production
- Sources : Copiées, pas présentes en clair
- **Gain : -70% de taille, -80% de vulnérabilités**

**Le vrai pouvoir :**
```dockerfile
# Ceci ne sera PAS dans l'image finale
FROM node:18-alpine AS builder
RUN npm install -g typescript eslint
RUN npm run build

# Ceci SEULEMENT sera en production
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

### **2. L'Ordre des Couches : La Clé du Cache**
**Principe fondamental :** Docker ne rebuild une couche que si elle ou ses parents changent.

**Stratégie optimale :**
```
1. FROM node:18-alpine         ← Change rarement (base)
2. WORKDIR /app                ← Ne change jamais
3. COPY package*.json ./       ← Change uniquement sur nouvelle dépendance
4. RUN npm ci                  ← Rebuild si package.json change
5. COPY . .                    ← Change à chaque commit → INVALIDATION
6. RUN npm run build           ← Rebuild à chaque fois
```

**Impact :**
- Sans optimisation : 3-5 minutes par build
- Avec optimisation : 10-20 secondes (cache hit)
- **Gain : 90-95%**

### **3. Multi-Architecture : Un Monde, Plusieurs Processeurs**
**Révélation :** Une seule image peut fonctionner partout.

**Avant :**
- Image amd64 → Ne fonctionne pas sur Mac M1
- Image arm64 → Ne fonctionne pas sur serveurs Intel
- Build séparés, tags séparés, confusion

**Après manifeste multi-arch :**
```bash
# Pour l'utilisateur
docker pull mon-app:latest
# Docker détecte automatiquement l'architecture
# Télécharge la bonne variante
# Même tag, même expérience
```

**Technologie sous-jacente :**
- **Buildx** : Build multi-plateforme
- **QEMU** : Émulation pour les plateformes non-natives
- **Manifeste** : Table des matières des architectures disponibles

### **4. Tags Intelligents : L'Identité de l'Image**
**Problème des tags plats :**
- `latest` → Quelle version ? Quel commit ? On ne sait pas
- `v1` → Correctif de sécurité appliqué ? Aucune idée

**Solution : Hiérarchie de tags**

| Tag | But | Exemple | Utilisation |
|-----|-----|---------|-------------|
| **SHA** | Unique, reproductible | `sha-a1b2c3d4` | Rollback, debug |
| **Branche** | Développement | `main`, `develop` | CI quotidienne |
| **SemVer** | Release | `1.2.3`, `1.2`, `1` | Production |
| **Latest** | Dernière stable | `latest` | Découverte |

**Pattern implémenté :**
```yaml
# Tous ces tags pointent vers la MÊME image
ghcr.io/app:sha-a1b2c3d4   # Unique, permanent
ghcr.io/app:main           # Mobile, dernière version de main
ghcr.io/app:1.2.3          # Version spécifique
ghcr.io/app:1.2            # Dernière version mineure
ghcr.io/app:1              # Dernière version majeure
ghcr.io/app:latest         # Dernière version stable
```

### **5. SBOM et Sécurité : La Transparence Obligatoire**
**Ce qu'on ne voyait pas avant :**
- Quelles sont les dépendances exactes dans l'image ?
- Y a-t-il des vulnérabilités connues ?
- Peut-on certifier ce qui est en production ?

**Ce qu'on a maintenant :**
```
SBOM (Software Bill of Materials) = Inventaire complet
├── Node.js 18.19.0
├── express 4.18.2
├── lodash 4.17.21
└── 127 autres dépendances
```

**Intégration pipeline :**
1. **Build** → Image créée
2. **Scan** → Trivy cherche les CVE
3. **SBOM** → Syft génère l'inventaire
4. **Archive** → Conservation pour audit
5. **Blocage** → Si vulnérabilité critique (demain)

---

## **🎯 BEST PRACTICES IDENTIFIÉES**

### **✅ Dockerfile Production**
- **Multi-stage obligatoire** : Builder + Production séparés
- **Ordre des couches** : Ce qui change peu en premier
- **Utilisateur non-root** : `USER nodejs` après installation
- **Healthcheck** : Pour orchestration (K8s, Docker Swarm)
- **Labels** : `org.opencontainers.image.source`, `version`

### **⚠️ Pipeline Docker**
- **Cache** : Toujours activer `type=gha` pour les builds
- **Login registry** : `GITHUB_TOKEN` suffit pour GHCR
- **Tags** : Au moins SHA + branche + version
- **Multi-arch** : Pour les applications exposées au public
- **Scan** : Trivy en mode `--severity CRITICAL,HIGH`

### **🔧 GitHub Container Registry**
- **Permissions** : `packages: write` dans le workflow
- **Public/Privé** : Défini dans les paramètres du package
- **Nommage** : `ghcr.io/owner/repo` automatique
- **Rétention** : Configurable par tag

### **📊 Observabilité des Images**
- **SBOM** : Générer et archiver pour chaque release
- **Taille** : Monitorer l'évolution
- **Âge** : Images trop vieilles = risque de sécurité
- **Pull count** : Popularité, dépendances

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Docker n'est pas un outil de build, c'est un format de livraison**
**Réalisation fondamentale :**
Docker ne sert pas à "compiler le code". Docker sert à **empaqueter l'application dans un format universel, déployable partout**.

**Conséquence sur le pipeline :**
- Le build doit être **reproductible**
- L'image est un **artifact** au même titre qu'un .jar ou .exe
- Le Dockerfile est une **spécification d'infrastructure**

### **2. Le Cache est un Investissement**
**Temps moyen sans cache :** 3 min 45 s
**Temps moyen avec cache :** 22 s
**Économie par build :** 3 min 23 s

**Calcul :**
- 50 builds/jour × 3.5 min = 175 min/jour (presque 3 heures)
- 175 min × 220 jours/an = 38 500 min ≈ **640 heures/an**

**Conclusion :** Optimiser le cache, c'est offrir **16 semaines de travail** à votre équipe.

### **3. Multi-Arch n'est pas une option**
**Tendance observée :**
- 2020 : 95% amd64, 5% arm
- 2023 : 70% amd64, 30% arm64
- 2026 : La moitié des nouveaux déploiements sont sur arm

**Pourquoi maintenant :**
- Apple Silicon (développeurs)
- AWS Graviton (coût -40% à performance égale)
- Raspberry Pi (edge computing)

**Stratégie :** Toute nouvelle image devrait être multi-arch par défaut.

### **4. La Sécurité par Transparence**
**Avant :** "On fait confiance à nos dépendances"
**Maintenant :** "On audite systématiquement"

**Ce qui change :**
- SBOM = Preuve de ce qui est déployé
- Scan = Détection proactive des vulnérabilités
- Non-root = Réduction de la surface d'attaque

**Nouveau réflexe :** Jamais d'image sans SBOM associé.

### **5. L'Industrialisation du Build**
**Maturité du processus Docker :**

| Niveau | Pratique | Statut |
|--------|----------|--------|
| 1 | Dockerfile basique | ❌ Dépassé |
| 2 | Multi-stage | ✅ Implémenté |
| 3 | Cache optimisé | ✅ Implémenté |
| 4 | Multi-architecture | ✅ Implémenté |
| 5 | SBOM + Scan | ✅ Implémenté |
| 6 | Quality gates | ⏳ Demain |
| 7 | Signature | 🔜 Prochain |

---

## **📈 PROGRESSION JOUR 75**

### **✅ ACQUIS TECHNIQUES :**
- **Dockerfile multi-stage** avec optimisation de cache
- **Pipeline Docker industrialisé** via GitHub Actions
- **Build multi-architecture** (amd64, arm64, arm/v7)
- **Manifeste fusionné** pour tag unique
- **Tags intelligents** (SHA, branche, semver, latest)
- **Scan de sécurité Trivy** intégré
- **SBOM Syft** généré et archivé
- **GitHub Container Registry** configuré

### **🎯 CHANGEMENT MENTAL :**
> **Hier :** "Je build une image Docker pour mon app"  
> **Aujourd'hui :** "Je **produis un artifact standardisé, multi-plateforme, sécurisé et traçable**"  
> **Résultat :** "L'image Docker n'est plus un output, c'est le **produit fini** de mon pipeline"

### **🔗 ARCHITECTURE DOCKER INDUSTRIELLE :**
```
                    USINE À IMAGES DOCKER
                    
[ CODE SOURCE ]
      ↓
┌─────────────────────────────────────────────────────┐
│              CHAÎNE DE CONTRÔLE                     │
├─────────────────────────────────────────────────────┤
│  Tests unitaires → Linting → Coverage              │
└─────────────────────────────────────────────────────┘
      ↓
┌─────────────────────────────────────────────────────┐
│              CHAÎNE DE PRODUCTION                   │
├─────────────────────────────────────────────────────┤
│  Docker Buildx (cache GHA)                          │
│  ├── linux/amd64  → Tag: sha, branch, semver       │
│  ├── linux/arm64  → Tag: sha, branch, semver       │
│  └── linux/arm/v7 → Tag: sha, branch, semver       │
└─────────────────────────────────────────────────────┘
      ↓
┌─────────────────────────────────────────────────────┐
│              FUSION & CERTIFICATION                 │
├─────────────────────────────────────────────────────┤
│  • Merge manifest → ghcr.io/app:latest              │
│  • Scan Trivy → Vulnérabilités                      │
│  • SBOM Syft → Inventaire complet                   │
└─────────────────────────────────────────────────────┘
      ↓
┌─────────────────────────────────────────────────────┐
│              LIVRAISON                              │
├─────────────────────────────────────────────────────┤
│  GitHub Container Registry                          │
│  ├── Tags: sha-a1b2c3, main, 1.2.3, latest          │
│  └── Pullable par: amd64, arm64, arm/v7             │
└─────────────────────────────────────────────────────┘
      ↓
[ DÉPLOIEMENT STAGING ]
```

### **⚠️ LIMITATIONS ACTUELLES :**
- ✅ **Multi-stage** : Implémenté
- ✅ **Cache optimisé** : Actif
- ✅ **Multi-architecture** : Opérationnel
- ✅ **Tags intelligents** : Configurés
- ✅ **Scan sécurité** : Intégré
- ✅ **SBOM** : Généré
- ❌ **Quality gates** : Blocage sur seuils non implémenté
- ❌ **Seuil de couverture** : Pas encore automatisé
- ❌ **Taille maximale** : Pas de vérification
- ❌ **Production approval** : Pour demain

### **🚀 POUR DEMAIN (JOUR 76) :**
- **Quality gates automatisés** : Coverage ≥ 80%, vulnérabilités critiques = 0
- **Blocage conditionnel** : Échec du pipeline si seuils non atteints
- **Taille d'image** : Alerte si > 500MB
- **Déploiement production** : Avec approbation manuelle
- **Notifications** : Slack/Teams sur succès/échec
- **Pipeline complet** : Du code à la production

---

## **💡 INSIGHTS FINAUX**

### **L'Industrialisation du Build Docker**
**Ce que nous avons construit aujourd'hui :**
Ce n'est plus un simple `docker build`. C'est une **chaîne de production** avec :
- **Matière première** : Code source
- **Contrôle qualité** : Tests, linting
- **Fabrication** : Build multi-architecture optimisé
- **Certification** : Scan sécurité, SBOM
- **Emballage** : Tags intelligents, manifeste
- **Livraison** : Registry, staging

### **Le Passage à l'Échelle**
**Ce qui était impossible avant :**
- Builder pour 3 architectures manuellement → 30 minutes
- Maintenir la cohérence des tags → Erreurs fréquentes
- Auditer la composition des images → Processus manuel

**Ce qui est possible maintenant :**
- 3 architectures en parallèle → 2 minutes
- Tags automatisés et cohérents → Zéro erreur
- SBOM automatique pour chaque build → Audit prêt

### **La Plateforme, Pas l'Outil**
**Changement de perspective :**
Docker n'est plus "l'outil que j'utilise pour builder". Docker est devenu **la plateforme de livraison standard** qui :
- Connecte le développement (Dockerfile)
- S'intègre au CI/CD (GitHub Actions)
- Sert le déploiement (Kubernetes, ArgoCD)

**Résultat :** Une chaîne logistique logicielle complète, de la matière première au produit en production.

---

## **📊 CHECKLIST ACCOMPLIE**

- [ ] **Dockerfile multi-stage** optimisé (builder + production)
- [ ] **Ordre des couches** optimisé pour le cache
- [ ] **Utilisateur non-root** en production
- [ ] **Healthcheck** configuré
- [ ] **Build local** testé et fonctionnel
- [ ] **Docker Compose** pour développement
- [ ] **Pipeline GitHub Actions** avec build Docker
- [ ] **Cache Docker layer** via GitHub Actions
- [ ] **Login GHCR** avec GITHUB_TOKEN
- [ ] **Tags intelligents** (SHA, branche, semver, latest)
- [ ] **Build multi-architecture** (amd64, arm64, arm/v7)
- [ ] **QEMU** pour émulation multi-plateforme
- [ ] **Merge manifeste** pour image unique
- [ ] **Scan Trivy** intégré au pipeline
- [ ] **SBOM Syft** généré et archivé
- [ ] **Images disponibles** sur GHCR
- [ ] **Déploiement staging** avec image Docker

---

**Le build Docker n'est plus artisanal :**  
**C'est une chaîne de production industrialisée, multi-plateforme, sécurisée et prête pour l'audit.** 🏭

**📊 Progress: `Jour 75 / 100 ✅`**

**#Docker #GitHubActions #MultiArch #DevOps #ContainerRegistry #SBOM #SecurityScanning #Industrialisation #CI/CD**
