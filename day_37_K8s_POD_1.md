# **JOUR 37 : PODS KUBERNETES - MAÎTRISE DE L'UNITÉ FONDAMENTALE** 📦

**Durée : 90 minutes**

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Qu'est-ce qu'un Pod Kubernetes ?**
- **Unité de déploiement minimale** dans K8s (≠ un simple conteneur)
- **Wrapper intelligent** pour 1+ conteneurs fonctionnant ensemble
- **Analogie** : Un appartement (Pod) avec des colocataires (conteneurs) partageant adresse IP, stockage et réseau local

### **📊 Pod vs Conteneur Docker : La Différence Essentielle**
| Aspect              | Conteneur Docker        | Pod Kubernetes                                |
|---------------------|-------------------------|-----------------------------------------------|
| **Unité**           | Conteneur individuel    | Groupe de 1+ conteneurs                       |
| **Réseau**          | IP propre               | **IP partagée** (localhost entre conteneurs)  |
| **Stockage**        | Volumes Docker          | Volumes K8s **partagés** entre conteneurs     |
| **Orchestration**   | Manuel/Docker Compose   | **Native à K8s** (Deployments, Services)      |
| **Usage principal** | Développement local     | **Production avec orchestration**             |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Création de Pods**
| Commande              | Objectif                          | Exemple                                   |
|-----------------------|-----------------------------------|-------------------------------------------|
| `kubectl run`         | Création rapide (test)            | `kubectl run nginx --image=nginx:alpine`  |
| `kubectl apply -f`    | Création via YAML (production)    | `kubectl apply -f mon-pod.yaml`           |
| `kubectl create`      | Création impérative               | `kubectl create -f pod.yaml`              |

### **👁️ Inspection et Monitoring**

| Commande              | Ce que je vois            | Pourquoi utile                        |
|-----------------------|---------------------------|---------------------------------------|
| `kubectl get pods`    | Liste tous les Pods       | "Quels Pods tournent ?"               |
| `kubectl describe pod`| Détails complets d'un Pod | Debug, configuration, événements      |
| `kubectl logs`        | Logs d'un Pod             | "Pourquoi mon app ne marche pas ?"    |
| `kubectl logs -f`     | Logs en temps réel        | Monitoring live                       |
| `kubectl exec -it`    | Shell dans un Pod         | Debug interactif, inspection          |


### **🗑️ Suppression de Pods**

| Méthode               | Commande                              | Quand l'utiliser                          |
|-----------------------|---------------------------------------|-------------------------------------------|
| **Par nom**           | `kubectl delete pod <nom>`            | Suppression rapide d'un Pod spécifique    |
| **Par fichier YAML**  | `kubectl delete -f fichier.yaml`      | Cohérent avec `kubectl apply -f`          |
| **Par label**         | `kubectl delete pod -l app=web`       | Suppression groupée par critère           |
| **Force delete**      | `kubectl delete pod <nom> --force`    | Pod bloqué "Terminating"                  |

---

## **📝 STRUCTURE YAML D'UN POD**

### **Manifest Pod Complet**
```yaml
apiVersion: v1           # Version API K8s (toujours v1 pour Pod)
kind: Pod               # Type de ressource
metadata:               # Identité et métadonnées
  name: mon-app-web     # Nom unique du Pod
  labels:               # Étiquettes pour organisation
    app: frontend
    env: development
    version: "1.0"
spec:                   # Spécifications du Pod
  containers:           # Liste des conteneurs
  - name: nginx         # Nom du conteneur
    image: nginx:1.25-alpine  # Image Docker
    ports:              # Ports exposés
    - containerPort: 80
    env:                # Variables d'environnement
    - name: NGINX_PORT
      value: "80"
    resources:          # Limites ressources
      requests:
        memory: "64Mi"
        cpu: "250m"     # 0.25 CPU
      limits:
        memory: "128Mi"
        cpu: "500m"     # 0.5 CPU
```

### **Pourquoi YAML > kubectl run ?**
- ✅ **Versionnable** : Git-friendly, historique des changements
- ✅ **Reproductible** : Même résultat à chaque déploiement
- ✅ **Documentation** : Configuration auto-documentée
- ✅ **Complexité** : Gère tous les cas avancés (ressources, probes, volumes)

---

## **🌀 POD LIFECYCLE - CYCLE DE VIE**

### **Les 4 Phases Principales**
```
1. Pending (En attente)     → K8s a accepté le Pod, allocation ressources en cours
2. Running (En cours)       → Pod affecté à un nœud, tous conteneurs créés
3. Succeeded (Réussi)       → Tous conteneurs terminés avec succès (ex: Jobs)
4. Failed (Échec)           → Au moins un conteneur a échoué
```

### **États Additionnels**
- **Unknown** : État indéterminé (problème communication avec le nœud)
- **Terminating** : En cours de suppression (grace period)
- **CrashLoopBackOff** : Conteneur crash et redémarre en boucle

### **Graceful Shutdown**
```yaml
# Dans le spec du Pod
spec:
  terminationGracePeriodSeconds: 60  # Délai avant SIGKILL (défaut: 30)
```

**Processus de suppression :**
1. `kubectl delete pod` → SIGTERM envoyé aux conteneurs
2. Attente de `terminationGracePeriodSeconds`
3. Si conteneurs toujours vivants → SIGKILL
4. Pod supprimé de etcd, ressources libérées

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Leçon 1 : Pods ≠ Applications de Production**
```bash
# ❌ MAUVAIS - Pod direct pour production
kubectl apply -f mon-pod.yaml

# ✅ BON - Pod managé par un Deployment (Jour 39)
kubectl apply -f mon-deployment.yaml
```

**Pourquoi ?** Les Pods nus sont :
- ❌ **Éphémères** : IP change à chaque redémarrage
- ❌ **Non résilients** : Pas de redémarrage auto garanti
- ❌ **Non scalables** : Pas de réplication automatique
- ❌ **Difficiles à updater** : Pas de rolling updates

### **Leçon 2 : Labels = Organisation Intelligente**
```yaml
metadata:
  labels:
    app: "mon-application"      # Nom logique
    component: "api"           # Rôle dans l'architecture
    version: "v1.2.3"          # Version sémantique
    env: "staging"             # Environnement
    managed-by: "helm"         # Outil de déploiement
    team: "backend"            # Équipe responsable
```

**Avantages des labels :**
- ✅ **Filtrage** : `kubectl get pods -l env=production`
- ✅ **Sélection** : Services trouvent leurs Pods via labels
- ✅ **Organisation** : Vue claire par projet/équipe/env

### **Leçon 3 : Resources Limits = Stabilité**
```yaml
resources:
  requests:     # Réservation garantie (scheduling)
    memory: "128Mi"    # K8s garantit cette mémoire
    cpu: "250m"        # 0.25 CPU réservé
  
  limits:       # Plafond strict (sécurité)
    memory: "256Mi"    # Le conteneur sera tué si > 256Mi
    cpu: "500m"        # Limit à 0.5 CPU max
```

**Sans limits** : Un Pod peut consommer toutes les ressources du nœud → "Noisy neighbor"

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Premier Pod avec YAML**
```bash
# Création du manifest
cat > premier-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: hello-k8s
spec:
  containers:
  - name: web
    image: nginx:alpine
    ports: [{containerPort: 80}]
EOF

# Déploiement
kubectl apply -f premier-pod.yaml

# Vérification
kubectl get pods -o wide
kubectl describe pod hello-k8s
```

### **2. Exploration Complète d'un Pod**
```bash
# 1. Voir l'état
kubectl get pod hello-k8s -o yaml  # Configuration complète

# 2. Accéder aux logs
kubectl logs hello-k8s
kubectl logs -f hello-k8s  # Follow en temps réel

# 3. Shell interactif
kubectl exec -it hello-k8s -- sh
# Dans le conteneur :
#   ls -la /usr/share/nginx/html
#   cat /etc/nginx/nginx.conf

# 4. Port forwarding pour test local
kubectl port-forward hello-k8s 8080:80
# Test : curl http://localhost:8080
```

### **3. Gestion du Cycle de Vie**
```bash
# Créer un Pod qui s'auto-terminera
cat > completable-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: one-time-job
spec:
  containers:
  - name: task
    image: busybox
    command: ["sh", "-c", "echo 'Travail terminé'; sleep 5"]
EOF

# Appliquer et observer le lifecycle
kubectl apply -f completable-pod.yaml
kubectl get pods -w  # Watch en temps réel
# Observe : Pending → Running → Succeeded
```

### **4. Nettoyage et Bonnes Pratiques**
```bash
# Suppression propre
kubectl delete -f premier-pod.yaml

# Suppression par nom
kubectl delete pod hello-k8s

# Vérification
kubectl get pods
kubectl get events | tail -10  # Voir les événements récents
```

### **5. Script d'Automatisation**
```bash
#!/bin/bash
# pod-manager.sh
ACTION=$1
POD_NAME=$2

case $ACTION in
  create)
    kubectl apply -f ${POD_NAME}.yaml
    ;;
  logs)
    kubectl logs -f $POD_NAME
    ;;
  shell)
    kubectl exec -it $POD_NAME -- sh
    ;;
  delete)
    kubectl delete pod $POD_NAME
    ;;
  *)
    echo "Usage: $0 {create|logs|shell|delete} pod-name"
    ;;
esac
```

---

## **🎯 BONNES PRATIQUES PRODUCTION**

### **Checklist Création Pod**
- [ ] **Nom significatif** : `frontend-v1` plutôt que `pod-123`
- [ ] **Labels complets** : app, version, env, team
- [ ] **Resources définies** : requests + limits CPU/mémoire
- [ ] **Image tag spécifique** : `nginx:1.25-alpine` pas `nginx:latest`
- [ ] **Readiness/Liveness probes** (à venir J38)
- [ ] **Security context** : non-root user si possible

### **Éviter les Anti-patterns**
```yaml
# ❌ ANTI-PATTERN : Trop de conteneurs dans un Pod
spec:
  containers:
  - name: app
    image: mon-app
  - name: db          # Base de données dans le même Pod
    image: postgres   # ❌ Mauvaise idée !
  - name: cache       # Cache aussi ?
    image: redis      # ❌ Très mauvaise idée !

# ✅ BON PATTERN : Un conteneur par fonction métier
# Séparer en différents Pods + Services
```

### **Debug Checklist**
```bash
# Quand un Pod ne démarre pas :
1. kubectl describe pod <nom>        # Événements et erreurs
2. kubectl logs <nom> --previous     # Logs du conteneur précédent
3. kubectl get events --sort-by-time # Événements récents cluster
4. kubectl get pod <nom> -o yaml     # Configuration appliquée
```

### **Organisation des Fichiers**
```
mon-projet/
├── pods/                    # Définitions de Pods
│   ├── frontend-pod.yaml
│   ├── backend-pod.yaml
│   └── database-pod.yaml
├── bases/                   # Bases communes (Kustomize)
│   └── kustomization.yaml
└── overlays/               # Environnements spécifiques
    ├── development/
    └── production/
```

---

## **📈 PROGRESSION JOUR 37**

### **✅ ACQUIS TECHNIQUES :**
- **Compréhension profonde** des Pods comme unité fondamentale
- **Maîtrise YAML** pour définition déclarative des Pods
- **Commandes kubectl complètes** : création, inspection, logs, exec, suppression
- **Cycle de vie des Pods** : phases, graceful shutdown, debugging
- **Best practices** : labels, resources, organisation

### **🎯 CHANGEMENT MENTAL :**
> **Je ne déploie plus des conteneurs, je orchestre des Pods**  
> **Mon infrastructure n'est plus statique, elle a un cycle de vie managé**  
> **Je pense en "état désiré déclaratif" plutôt qu'en "commandes impératives"**

### **🔗 ARCHITECTURE MENTALE ÉTABLIE :**
```
MON APPLICATION
     ↓
POD KUBERNETES (unité de déploiement)
     ├── Conteneur 1 : Mon application
     ├── Conteneur 2 : Sidecar (logs, monitoring)
     └── Volume partagé : Données communes
           ↓
     IP UNIQUE + Stockage partagé
           ↓
     Géré par Kubelet sur un Worker Node
```

### **🚀 PROCHAINES ÉTAPES (JOUR 38) :**
- **Pods multi-conteneurs** : Pattern Sidecar avancé
- **Volumes dans Pods** : Stockage persistant et partagé
- **Probes** : Liveness, Readiness, Startup probes
- **Conversion projet** : Transformer ton app Docker → Pod K8s
- **Sécurité** : Security context, non-root containers

---

**📊 Progress: `Jour 37 / 100 ✅`**

**#Kubernetes #Pods #DevOps #Containers #YAML #Kubectl #CloudNative #InfrastructureAsCode**
