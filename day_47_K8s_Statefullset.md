# **JOUR 47 : STATEFULSETS - APPLICATIONS AVEC ÉTAT** 🗂️

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Applications Stateful vs Stateless**
- **Stateless** : Pods interchangeables, sans données persistantes (frontend, API)
- **Stateful** : Pods avec identité, données persistantes (bases de données, cache)
- **Besoin spécifique** : Identité stable, ordre séquentiel, stockage unique

### **🔧 Les StatefulSets**
- **Pods ordonnés** : `app-0`, `app-1`, `app-2`
- **DNS stable** : `app-0.service`, `app-1.service`
- **Stockage unique** : Chaque Pod a son propre volume persistant
- **Opérations ordonnées** : Création/suppression/mise à jour séquentielles

---

## **📊 Comparaison Deployment vs StatefulSet**

| Aspect            | Deployment             | StatefulSet                  |
|-------------------|------------------------|------------------------------|
| **Identité Pod**  | Aléatoire              | Ordinale (`web-0`, `web-1`)  |
| **DNS**           | Service seulement      | Par Pod (`pod.service`)      |
| **Stockage**      | Partageable            | Unique par Pod               |
| **Scaling**       | Parallèle              | Séquentiel                   |
| **Mise à jour**   | RollingUpdate          | RollingUpdate ordonné        |
| **Usage**         | Applications stateless | Applications stateful        |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Gestion des StatefulSets**
| Commande                                       | Objectif                         | Exemple           |
|------------------------------------------------|----------------------------------|-------------------|
| `kubectl get statefulsets`                     | Lister les StatefulSets          | Vue d'ensemble    |
| `kubectl describe statefulset <nom>`           | Détails d'un StatefulSet         | Configuration     |
| `kubectl scale statefulset <nom> --replicas=N` | Modifier le nombre de réplicas   | Scaling ordonné   |
| `kubectl edit statefulset <nom>`               | Modifier un StatefulSet          | Mise à jour       |

### **🔍 Inspection**
| Commande                          | Ce qu'elle révèle         | Pourquoi c'est utile  |
|-----------------------------------|---------------------------|-----------------------|
| `kubectl get pods -l app=<label>` | Pods du StatefulSet       | Voir l'ordre          |
| `kubectl get pvc -l app=<label>`  | PVCs des Pods             | Vérifier stockage     |
| `kubectl logs <pod-name>-0`       | Logs d'un Pod spécifique  | Debug par réplica     |

### **🌐 Tests Réseau**
```bash
# Tester le DNS stable
kubectl run test --image=alpine --rm -it -- nslookup <pod-name>.<service>

# Tester la connectivité
kubectl exec <pod-name>-0 -- curl http://<pod-name>-1.<service>
```

---

## **📝 STRUCTURE DES STATEFULSETS**

### **Service Headless (obligatoire) :**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None  # ⚠️ Service headless
  selector:
    app: postgres
  ports:
  - port: 5432
```

### **StatefulSet avec stockage :**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"  # ⚠️ Doit matcher le service
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14-alpine
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:  # ⚠️ Crée un PVC par Pod
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi
```

### **DNS Stable :**
```
Format : <pod-name>.<service-name>.<namespace>.svc.cluster.local
Exemple : postgres-0.postgres.default.svc.cluster.local
Simplifié : postgres-0.postgres
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Identité Stable des Pods**
**Noms prévisibles :**
- `postgres-0`, `postgres-1`, `postgres-2`
- Même après redémarrage, même nom
- Important pour réplication master/slave

### **2. DNS Stable par Pod**
**Chaque Pod accessible directement :**
```bash
# Depuis un autre Pod
ping postgres-0.postgres
curl http://web-1.nginx

# Résolution DNS directe
nslookup postgres-0.postgres
```

### **3. Volume Claim Templates**
**Stockage unique par Pod :**
- Chaque Pod (`postgres-0`) a son PVC (`data-postgres-0`)
- Les données ne sont pas partagées entre réplicas
- Scaling crée automatiquement de nouveaux PVCs

### **4. Scaling Séquentiel**
**Ordre garanti :**
```
Scale up :   postgres-0 → postgres-1 → postgres-2
Scale down : postgres-2 → postgres-1 → postgres-0
Mise à jour : postgres-2 → postgres-1 → postgres-0
```

### **5. Service Headless**
**Différence cruciale :**
- **Service normal** : Load balancing, IP virtuelle
- **Service headless** : Pas d'IP, retourne les IPs des Pods directement
- Nécessaire pour la découverte DNS des Pods individuels

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Premier StatefulSet simple**
```yaml
# Service headless obligatoire
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None  # Headless
  selector:
    app: nginx

# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"  # Doit matcher
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
  volumeClaimTemplates:  # PVC par Pod
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### **2. PostgreSQL avec StatefulSet**
```bash
# Déploiement
kubectl apply -f postgres-config.yaml
kubectl apply -f postgres-service.yaml  # Headless
kubectl apply -f postgres-statefulset.yaml

# Vérification
kubectl get statefulsets
kubectl get pods -l app=postgres  # postgres-0, postgres-1...

# Scaling ordonné
kubectl scale statefulset postgres --replicas=2
# postgres-0 (existant), puis postgres-1 (nouveau)
```

### **3. Test de persistance**
```bash
# Écrire des données
kubectl exec postgres-0 -- psql -c "CREATE TABLE test;"

# Supprimer le Pod
kubectl delete pod postgres-0

# Kubernetes recrée postgres-0
kubectl get pods -l app=postgres

# Vérifier que les données persistent
kubectl exec postgres-0 -- psql -c "SELECT * FROM test;"
```

### **4. DNS stable test**
```bash
# Depuis un Pod test
kubectl run test --image=alpine --rm -it -- sh

# Dans le conteneur :
nslookup postgres-0.postgres
nslookup postgres-1.postgres
# Résout vers les IPs spécifiques des Pods
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Architecture**
- **Service headless** : Obligatoire pour DNS stable
- **volumeClaimTemplates** : Pour stockage par Pod
- **Labels cohérents** : Entre Service et StatefulSet
- **Réplicas initiaux** : Commencer petit, scale progressivement

### **⚠️ Production Considerations**
- **Backup régulier** : Chaque volume individuellement
- **Monitoring par Pod** : Chaque réplica est unique
- **Tests de scaling** : Vérifier ordre et persistance
- **Récupération désastre** : Plan pour chaque réplica

### **🔧 Configuration**
- **StorageClass adaptée** : Performance selon les besoins
- **Resource limits** : Mémoire/CPU pour bases de données
- **Readiness probes** : Vérifier que le Pod est vraiment prêt
- **Update strategy** : `RollingUpdate` ou `OnDelete`

### **📋 Checklist StatefulSet**
- [ ] Service headless avec `clusterIP: None`
- [ ] `serviceName` qui matche le Service
- [ ] `volumeClaimTemplates` pour stockage persistant
- [ ] Scaling up/down testé
- [ ] DNS stable vérifié
- [ ] Persistance des données testée
- [ ] Backup planifié
- [ ] Monitoring configuré

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Pas pour Toutes les Applications**
**StatefulSets quand :**
- ✅ Données persistantes nécessaires
- ✅ Identité Pod importante
- ✅ Ordre des opérations critique
- ❌ Sinon, utiliser Deployment

### **2. Complexité Ajoutée**
**À considérer :**
- Plus complexe que Deployment
- Gestion manuelle parfois nécessaire
- Récupération plus compliquée
- Monitoring plus exigeant

### **3. Stockage Critique**
**Chaque Pod = Son Volume :**
- Pas de partage de données
- Réplication à gérer dans l'application
- Backup individualisé
- Coût potentiellement plus élevé

### **4. Patterns d'Usage**
**Cas typiques :**
- **Base de données** : PostgreSQL, MySQL avec réplication
- **Cache distribué** : Redis Cluster
- **File d'attente** : Kafka brokers
- **Stockage distribué** : Ceph, MinIO

---

## **📈 PROGRESSION JOUR 47**

### **✅ ACQUIS TECHNIQUES :**
- **Différence stateful/stateless** : Compréhension fondamentale
- **StatefulSets** : Configuration et déploiement
- **Service headless** : Rôle et configuration
- **Stockage par Pod** : volumeClaimTemplates
- **DNS stable** : Accès direct aux Pods
- **Scaling ordonné** : Contrôle séquentiel

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Je déploie tout avec des Deployments"  
> **Aujourd'hui :** "Je **choisis** le contrôleur adapté à chaque type d'application"  
> **Résultat :** "Stateful pour les données, Stateless pour le traitement"

### **🔗 ARCHITECTURE CONSTRUITE :**
```
APPLICATIONS STATEFUL :

SERVICE HEADLESS
├── clusterIP: None
└── DNS des Pods individuels
    ↓
STATEFULSET
├── Pods ordonnés: app-0, app-1, app-2
├── Stockage: volumeClaimTemplates → PVC par Pod
├── DNS stable: app-0.service, app-1.service
└── Scaling: Séquentiel garanti
```

---

## **💡 INSIGHTS FINAUX**

### **La Bonne Tool pour le Bon Job**
**StatefulSets ne remplacent pas Deployments :**
- **Deployments** : 90% des applications (stateless)
- **StatefulSets** : 10% spécialisés (stateful)
- **Choix conscient** : Basé sur les besoins réels

### **Équilibre Complexité/Fonctionnalité**
**Avantages :**
- ✅ Identité stable
- ✅ DNS prévisible  
- ✅ Stockage persistant
- ✅ Scaling contrôlé

**Coûts :**
- ❌ Plus complexe
- ❌ Gestion manuelle parfois
- ❌ Monitoring exigeant
- ❌ Récupération compliquée

**L'essentiel :** Utiliser StatefulSets quand nécessaire, pas par défaut.

---

**📊 Progress: `Jour 47 / 100 ✅`**

**#Kubernetes #StatefulSets #Databases #PostgreSQL #StatefulApplications #PersistentStorage #DevOps #CloudNative**
