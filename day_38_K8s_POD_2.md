---

# **JOUR 38 : PODS AVANCÉS & PATTERNS MULTI-CONTENEURS** 🚀

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Pod Kubernetes – Rappels Essentiels**

* Un **Pod = unité minimale de déploiement**
* Les conteneurs d’un Pod partagent :

  * **Le réseau** (`localhost`)
  * **Les volumes**
  * **Le cycle de vie**

---

### **🧩 Patterns Multi-Conteneurs**

| Pattern        | Rôle                   | Exemple          |
| -------------- | ---------------------- | ---------------- |
| **Sidecar**    | Étend l’app principale | Logs, monitoring |
| **Adapter**    | Transforme les données | JSON → CSV       |
| **Ambassador** | Abstraction réseau     | Proxy local      |

**Règle clé** :
➡️ **Un Pod = des conteneurs fortement couplés**

---

## **🗂️ VOLUMES PARTAGÉS (emptyDir)**

```yaml
volumes:
- name: shared-data
  emptyDir: {}
```

**À retenir :**

* Créé au démarrage du Pod
* Supprimé à la destruction du Pod
* Sert à la **communication inter-conteneurs**
* Option possible : `medium: Memory` (RAM)

---

## **🛠️ EXERCICES PRATIQUES**

### **1️⃣ Pattern Sidecar – Logs**

* App principale : **Nginx**
* Sidecar : **Fluentd**
* Partage via `emptyDir`

🎯 **But** : Collecter les logs sans modifier l’application

---

### **2️⃣ Pattern Adapter – Transformation**

* Conteneur 1 : génère des logs **JSON**
* Conteneur 2 : convertit en **CSV**
* Traitement en temps réel (`tail -f`)

🎯 **But** : Normaliser les données dans le Pod

---

### **3️⃣ Pod Production-Ready**

* `securityContext` (non-root)
* `resources` (requests / limits)
* `livenessProbe`
* `readinessProbe`

---

## **❤️ PROBES DE SANTÉ**

| Probe         | Question posée                 |
| ------------- | ------------------------------ |
| **Liveness**  | L’app est-elle vivante ?       |
| **Readiness** | Peut-elle recevoir du trafic ? |

```yaml
livenessProbe  → redémarre le conteneur
readinessProbe → retire du Service
```

---

## **🔐 SÉCURITÉ POD**

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

**Bonnes pratiques :**

* Jamais en root
* Capabilities minimales
* Sécurité définie **dans le YAML**

---

## **⚙️ RESSOURCES (CPU / MÉMOIRE)**

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

* **Requests** → scheduling garanti
* **Limits** → stabilité cluster
* Dépassement mémoire → **OOMKill**

---

## **🧠 RÈGLES D’OR À RETENIR**

### ✅ Quand utiliser un Pod multi-conteneurs

* Conteneurs **fortement liés**
* Communication locale
* Même cycle de vie

### ❌ Quand éviter

* Services indépendants
* Besoin de scaling séparé

---

## **📌 COMMANDES UTILES**

```bash
# Lister conteneurs du Pod
kubectl get pod POD -o jsonpath='{.spec.containers[*].name}'

# Logs d’un conteneur précis
kubectl logs POD -c CONTAINER

# Shell dans un conteneur
kubectl exec -it POD -c CONTAINER -- sh

# Ressources utilisées
kubectl top pod POD --containers
```

---

## **📈 PROGRESSION JOUR 38**

### **✅ Compétences acquises**

* Pods multi-conteneurs maîtrisés
* Patterns Sidecar & Adapter compris
* Volumes `emptyDir` utilisés efficacement
* Probes et ressources configurées
* Sécurité intégrée dès la conception

### **🎯 Mentalité Kubernetes**

> Un Pod n’est pas un simple conteneur
> C’est une **brique applicative complète**,
> sécurisée, observable et résiliente.

---

### **➡️ Prochaine étape (Jour 39–40)**

* Deployments
* ReplicaSets
* Rolling Updates
* Services & Load Balancing

---

**📊 Progress: `Jour 38 / 100 ✅`**

**#Kubernetes #Pods #MultiContainer #Sidecar #Adapter #K8sSecurity #DevOps**

---
