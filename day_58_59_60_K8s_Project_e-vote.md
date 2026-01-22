# **JOURS 58-60 : PROJET INTERMÉDIAIRE KUBERNETES** 🚀

## **🎯 OBJECTIF DU PROJET**
Migrer une application complète de Docker Compose vers Kubernetes, en appliquant toutes les compétences acquises pendant la semaine 9 (Auto-scaling, Health Checks, Self-Healing).

**Durée :** 3 jours

---

## **📊 ARCHITECTURE MIGRÉE**

### **🔧 Services Docker Compose → Kubernetes**
| Service Docker            | Équivalent Kubernetes | Type      | Spécificités                          |
|---------------------------|-----------------------|-----------|---------------------------------------|
| **PostgreSQL**            | StatefulSet           | Stateful  | PersistentVolume, ConfigMap, Secret   |
| **Spring Boot Backend**   | Deployment            | Stateless | HPA, Health Checks, ConfigMap         |
| **Angular Frontend**      | Deployment            | Stateless | HPA, ConfigMap, Ingress               |
| **SMTP4Dev**              | Deployment            | Stateless | Service ClusterIP                     |

### **🏗️ Architecture Kubernetes Finale**
```
NAMESPACE: evote
├── 📦 StatefulSet: postgres-db
│   ├── 🔐 Secret: credentials
│   ├── 📄 ConfigMap: configuration
│   └── 💾 PersistentVolumeClaim: data-storage
│
├── 📦 Deployment: backend-spring
│   ├── 🔄 HPA: auto-scaling (CPU 70%, Memory 80%)
│   ├── 🏥 Probes: liveness + readiness
│   ├── 📄 ConfigMap: app properties
│   └── 🔗 Service: backend-service (ClusterIP)
│
├── 📦 Deployment: frontend-angular  
│   ├── 🔄 HPA: auto-scaling (CPU 60%, Memory 70%)
│   ├── 🏥 Probes: simple HTTP checks
│   ├── 📄 ConfigMap: environment
│   └── 🔗 Service: frontend-service (ClusterIP)
│
├── 📦 Deployment: smtp-server
│   └── 🔗 Service: smtp-service (ClusterIP)
│
└── 🌐 Ingress: evote-ingress
    ├── 📍 / → frontend-service
    ├── 📍 /api → backend-service
    └── 📍 /smtp → smtp-service
```

---

## **🛠️ COMPÉTENCES APPLIQUÉES**

### **✅ De la Semaine 9 (Jours 53-57)**
1. **Auto-Scaling (HPA)** : Configuré sur backend et frontend
2. **Health Checks** : Liveness + Readiness probes sur tous les services
3. **Self-Healing** : Redémarrage auto, isolation des pannes
4. **Monitoring** : Metrics Server pour HPA

### **✅ Des Semaines Précédentes**
1. **Services & Networking** : Services ClusterIP, Ingress
2. **Configuration** : ConfigMaps et Secrets
3. **Storage** : PersistentVolumes pour la base de données
4. **Déploiement** : Deployments avec rolling updates

---

## **📁 STRUCTURE DES FICHIERS CRÉÉS**

```
e-vote-k8s/
├── 📂 00-namespace/              # Isolation logique
│   └── namespace.yaml
│
├── 📂 01-configuration/          # Configuration non sensible
│   ├── backend-configmap.yaml
│   ├── frontend-configmap.yaml
│   └── database-configmap.yaml
│
├── 📂 02-secrets/                # Données sensibles
│   ├── database-secret.yaml
│   └── backend-secret.yaml
│
├── 📂 03-storage/                # Persistance données
│   ├── postgres-pv.yaml
│   └── postgres-pvc.yaml
│
├── 📂 04-database/               # Service stateful
│   ├── postgres-statefulset.yaml
│   └── postgres-service.yaml
│
├── 📂 05-backend/                # Microservice Spring Boot
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   └── backend-hpa.yaml
│
├── 📂 06-frontend/               # Application Angular
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   └── frontend-hpa.yaml
│
├── 📂 07-smtp/                   Service mail de test
│   ├── smtp-deployment.yaml
│   └── smtp-service.yaml
│
├── 📂 08-networking/             # Exposition externe
│   └── ingress.yaml
│
├── 📂 09-monitoring/             # Observabilité
│   └── service-monitor.yaml
│
├── 🚀 deploy.sh                  # Script de déploiement
├── 📋 README.md                  # Documentation
└── 🔧 kustomization.yaml         # Gestion des versions (optionnel)
```

---

## **🔍 PROBLÈMES RENCONTRÉS ET SOLUTIONS**

### **1. Ingress Controller Non Disponible**
**Problème :** `Error creating ingress resource`
**Solution :** Installation manuelle de Nginx Ingress Controller
```bash
# Sur Minikube
minikube addons enable ingress

# Vérification
kubectl get pods -n ingress-nginx
```

### **2. Images Privées Docker Hub**
**Problème :** `ImagePullBackOff` pour les images privées
**Solution :** Création d'un secret Docker Registry
```yaml
# Dans les Deployments
spec:
  template:
    spec:
      imagePullSecrets:
      - name: dockerhub-secret
```

### **3. Connexion Base de Données**
**Problème :** Backend ne peut pas se connecter à PostgreSQL
**Solution :** Utilisation des Services DNS Kubernetes
```yaml
# ConfigMap backend
data:
  SPRING_DATASOURCE_URL: jdbc:postgresql://postgres-service.evote.svc.cluster.local:5432/evote
```

### **4. HPA Sans Métriques**
**Problème :** `HPA unable to fetch metrics`
**Solution :** Activation de Metrics Server
```bash
minikube addons enable metrics-server
kubectl top pods -n evote
```

---

## **🎯 BONNES PRATIQUES IMPLÉMENTÉES**

### **1. Séparation des Préoccupations**
- **Namespace dédié** : Isolation logique de l'application
- **ConfigMaps vs Secrets** : Séparation configuration sensible/non sensible
- **Services par composant** : Découplage des responsabilités

### **2. Haute Disponibilité**
- **Multiples réplicas** : minReplicas: 2 sur les services critiques
- **Health Checks** : Liveness + Readiness probes
- **Rolling Updates** : Mise à jour sans interruption

### **3. Auto-Scaling Optimal**
```yaml
# Backend HPA
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 80
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # 5 minutes
```

### **4. Sécurité**
- **Secrets chiffrés** : Données sensibles en base64
- **Services ClusterIP** : Exposition interne uniquement
- **Ingress avec TLS** : Chiffrement des communications

---

## **🚀 COMMANDES DE DÉPLOIEMENT**

### **Déploiement Complet**
```bash
# Appliquer tous les manifests
./deploy.sh

# Ou étape par étape
kubectl apply -f 00-namespace/
kubectl apply -f 01-configuration/
kubectl apply -f 02-secrets/
# ... etc pour chaque dossier
```

### **Vérification**
```bash
# Voir l'état global
kubectl get all -n evote

# Voir les événements
kubectl get events -n evote --sort-by=.lastTimestamp

# Vérifier les endpoints
kubectl get endpoints -n evote

# Tester l'application
curl http://evote.local
```

### **Monitoring**
```bash
# Voir l'utilisation des ressources
kubectl top pods -n evote
kubectl top nodes

# Surveiller les HPA
kubectl get hpa -n evote -w

# Voir les logs
kubectl logs -n evote -l app=backend --tail=50 -f
```

---

## **📊 ÉQUIVALENCES DOCKER COMPOSE ↔ KUBERNETES**

| Concept Docker Compose | Concept Kubernetes | Avantage Kubernetes |
|------------------------|-------------------|---------------------|
| `services:` | `Deployment` + `Service` | Scaling auto, health checks |
| `environment:` | `ConfigMap` + `Secret` | Séparation config/sensible |
| `volumes:` | `PersistentVolumeClaim` | Abstraction stockage |
| `ports:` | `Service` + `Ingress` | Load balancing avancé |
| `depends_on:` | Readiness Probes | Détection automatique |
| `restart: always` | Liveness Probes | Self-healing intelligent |
| `scale:` | `HorizontalPodAutoscaler` | Auto-scaling dynamique |

---

## **🔐 GESTION DES SECRETS**

**Approche sécurisée :**
```bash
# Ne JAMAIS commiter en clair
# Utiliser des outils comme SealedSecrets ou Vault en production

# Pour le développement
kubectl create secret generic db-secret \
  --from-literal=password='1234' \
  --namespace=evote \
  --dry-run=client \
  -o yaml > database-secret.yaml
```

---

## **🌐 ACCÈS À L'APPLICATION**

### **Via Ingress (Production)**
```bash
# Ajouter au hosts local
echo "$(minikube ip) evote.local" | sudo tee -a /etc/hosts

# URLs
http://evote.local          # Frontend
http://evote.local/api      # Backend API
http://evote.local/api/actuator/health  # Health checks
```

### **Via Port-Forward (Développement)**
```bash
# Accès direct aux services
kubectl port-forward service/frontend-service 8080:80 -n evote
# http://localhost:8080

kubectl port-forward service/backend-service 8081:8080 -n evote
# http://localhost:8081
```

---

## **📈 MÉTRICS ET MONITORING**

### **Metrics Server**
```bash
# Activation
minikube addons enable metrics-server

# Vérification
kubectl get apiservices | grep metrics
kubectl top nodes
```

### **Métriques Collectées**
- **Utilisation CPU/Mémoire** : Pour HPA
- **Restart counts** : Indicateur de stabilité
- **Ready pods** : Pour décisions de scaling
- **Endpoint changes** : Pour disponibilité

---

## **🧹 NETTOYAGE**

```bash
# Supprimer l'application complète
kubectl delete namespace evote

# Supprimer les volumes persistants
kubectl delete pv --all

# Réinitialiser Minikube (si nécessaire)
minikube stop
minikube delete
minikube start
```

---

## **🎯 LEÇONS APPRISES**

### **1. Kubernetes ≠ Docker Compose**
- **Architecture différente** : Plus de composants, plus de flexibilité
- **Déclaration vs Exécution** : Kubernetes gère l'état désiré
- **Résilience intégrée** : Self-healing, auto-scaling, rolling updates

### **2. Importance des Health Checks**
- **Sans probes** : Kubernetes ne sait pas si l'app fonctionne
- **Readiness critique** : Pour le routing Ingress et HPA
- **Startup nécessaire** : Pour les apps lentes (Spring Boot)

### **3. Auto-Scaling Pratique**
- **HPA simple à configuer** : Basé sur CPU/mémoire
- **Impact des probes** : Seuls les pods "ready" comptent
- **Stabilisation importante** : Éviter le thrashing

### **4. Debug Indispensable**
- `kubectl describe` : Comprendre les problèmes
- `kubectl logs` : Voir ce qui se passe
- `kubectl get events` : Chronologie des événements

---

## **🚀 PROCHAINES ÉTAPES POSSIBLES**

### **Pour la Production**
- [ ] **Helm Charts** : Packaging et gestion des versions
- [ ] **Cert-manager** : Certificats TLS automatiques
- [ ] **Prometheus + Grafana** : Monitoring avancé
- [ ] **ArgoCD** : GitOps pour déploiements
- [ ] **Network Policies** : Sécurité réseau fine
- [ ] **PodDisruptionBudget** : HA pendant maintenance

### **Améliorations**
- [ ] **Resource Quotas** : Limiter l'utilisation ressources
- [ ] **Affinity/Anti-affinity** : Répartition optimale des pods
- [ ] **Custom Metrics HPA** : Scaling basé sur métriques métier
- [ ] **Service Mesh** : Istio/Linkerd pour observabilité avancée
- [ ] **Chaos Engineering** : Tests de résilience proactifs

---

## **📊 BILAN DES 3 JOURS**

### **✅ ACCOMPLISSEMENTS**
- ✅ **Migration complète** Docker Compose → Kubernetes
- ✅ **Architecture production-ready** avec tous les services
- ✅ **Auto-scaling implémenté** sur services critiques
- ✅ **Health checks avancés** avec self-healing
- ✅ **Sécurité** via Secrets et ConfigMaps
- ✅ **Documentation complète** et reproductible

### **🔧 COMPÉTENCES DÉMONTRÉES**
- **Déploiement** : Deployments, Services, Ingress
- **Configuration** : ConfigMaps, Secrets
- **Stockage** : PersistentVolumes, StatefulSets
- **Auto-scaling** : HPA avec métriques ressources
- **Résilience** : Health checks, rolling updates
- **Monitoring** : Metrics Server, logging, debugging

### **🎯 RÉSULTAT FINAL**
Une application complète migrée vers Kubernetes, scalable, résiliente, observable et prête pour le déploiement en production. Toutes les compétences des 57 premiers jours ont été appliquées dans un projet concret et réaliste.

---

**📌 CONCLUSION** : Ce projet a consolidé toutes les compétences Kubernetes acquises jusqu'à présent. L'application est maintenant déployable sur n'importe quel cluster Kubernetes, avec auto-scaling, self-healing et une architecture professionnelle.

**#Kubernetes #Migration #DockerToK8s #ProductionReady #DevOps #CloudNative #Microservices #AutoScaling #SelfHealing**
