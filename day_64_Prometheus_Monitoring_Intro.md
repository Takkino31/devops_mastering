# **JOUR 64 : MONITORING AVEC PROMETHEUS - INSTALLATION ET BASES** 📊

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Prometheus**
- **Pull-based** : Prometheus tire les métriques (vs push traditionnel)
- **Auto-discovery** : Détection automatique via Kubernetes API
- **Multi-dimensionnel** : Métriques enrichies avec labels
- **Stack complète** : Prometheus + Grafana + Alertmanager + Exporters

### **📈 Types de Métriques Essentielles**
- **Counter** : Compteur monotone croissant (requêtes totales)
- **Gauge** : Valeur instantanée variable (mémoire, CPU)
- **Histogram/Summary** : Distributions (latences, tailles)

### **🔧 Composants Installés**
- **Prometheus Server** : Collecte et stocke les métriques
- **Grafana** : Visualisation via dashboards
- **Node Exporter** : Métriques système (CPU, mémoire, disk)
- **Kube-State-Metrics** : Métriques objets Kubernetes
- **Alertmanager** : Gestion des alertes (pour plus tard)

---

## **📊 Architecture Prometheus dans Kubernetes**

| Composant              | Rôle                 | Comment ça marche                            |
|------------------------|----------------------|----------------------------------------------|
| **Prometheus Server**  | Collecte et stocke   | Scrape les endpoints /metrics toutes les 30s |
| **ServiceMonitor**     | Découverte cibles    | Configure ce que Prometheus doit surveiller  |
| **Node Exporter**      | Métriques système    | Expose métriques machine via DaemonSet       |
| **Kube-State-Metrics** | Métriques K8s        | Convertit l'état K8s en métriques            |
| **Grafana**            | Visualisation        | Lit depuis Prometheus, affiche dashboards    |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Installation & Configuration**
| Commande                              | Objectif           | Exemple              |
|---------------------------------------|--------------------|----------------------|
| `helm repo add prometheus-community`  | Ajouter repo       | Installation charts  |
| `helm install prometheus-stack`       | Installer stack    | Tout-en-un           |
| `kubectl create namespace monitoring` | Préparer namespace | Isolation            |

### **🔍 Accès & Inspection**
| Commande                              | Ce qu'elle révèle | Pourquoi utile            |
|---------------------------------------|-------------------|---------------------------|
| `kubectl port-forward svc/grafana`    | Accès Grafana     | Dashboards web            |
| `kubectl port-forward svc/prometheus` | Accès Prometheus  | Requêtes PromQL           |
| `kubectl get pods -n monitoring`      | État installation | Vérifier fonctionnement   |
| `kubectl get svc -n monitoring`       | Services exposés  | Ports d'accès             |

### **📊 Premières Requêtes PromQL**
```promql
# BASICS
count(kube_pod_info)                    # Nombre total de pods
sum(kube_pod_info) by (namespace)       # Pods par namespace
kube_pod_status_phase{phase="Running"}  # Pods en état Running

# SYSTÈME
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)  # CPU utilisé %
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100  # Mémoire utilisée %

# FILTRAGE
kube_pod_info{namespace="monitoring"}   # Pods dans namespace spécifique
kube_pod_info{pod=~".*grafana.*"}       # Pods avec nom contenant "grafana"
```

---

## **📝 STRUCTURE INSTALLÉE**

### **Namespace Monitoring :**
```bash
kubectl get pods -n monitoring
# prometheus-stack-prometheus-xxx           # Serveur principal
# prometheus-stack-grafana-xxx              # Interface web
# prometheus-stack-kube-state-metrics-xxx   # Métriques K8s  
# prometheus-stack-prometheus-node-exporter # Métriques node
# prometheus-stack-prometheus-operator-xxx  # Gestion automatique
# prometheus-stack-alertmanager-xxx         # Alertes
```

### **Services Exposés :**
```bash
kubectl get svc -n monitoring
# prometheus-stack-grafana          ClusterIP   10.96.x.x    80/TCP
# prometheus-stack-prometheus       ClusterIP   10.96.x.x    9090/TCP
# prometheus-stack-alertmanager     ClusterIP   10.96.x.x    9093/TCP
```

### **Accès Local :**
```bash
# Grafana (Dashboards) - http://localhost:3000
kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring &

# Prometheus (Requêtes) - http://localhost:9090  
kubectl port-forward svc/prometheus-stack-prometheus 9090:9090 -n monitoring &

# Credentials Grafana
# Username: admin
# Password: admin123 (configuré lors de l'installation)
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Modèle Pull vs Push**
**Prometheus est différent :**
- ❌ Traditionnel : Apps poussent leurs métriques
- ✅ Prometheus : Tire depuis les endpoints /metrics
- **Avantage** : Meilleur pour environnements dynamiques (K8s)

### **2. Auto-discovery Kubernetes**
**Pas de configuration statique :**
- Prometheus interroge l'API K8s
- Découvre nouveaux pods/services automatiquement
- Via **ServiceMonitors** et **PodMonitors**

### **3. Labels = Super-pouvoir**
**Une métrique, plusieurs dimensions :**
```promql
http_requests_total{
  method="GET", 
  path="/api/users", 
  status="200", 
  pod="app-xyz",
  namespace="production"
}
```
→ Filtrable par n'importe quelle combinaison de labels

### **4. Stack Tout-en-un**
**kube-prometheus-stack inclut :**
- ✅ Prometheus (collecte)
- ✅ Grafana (visualisation)  
- ✅ Alertmanager (alertes)
- ✅ Node exporter (métriques système)
- ✅ Kube-state-metrics (métriques K8s)
- ✅ Prometheus Operator (gestion)

### **5. Problème Identifié**
**Nos applications ne sont pas monitorées :**
- L'app test `nginx-test` existe mais n'est pas scrappée
- **Raison** : Pas d'endpoint /metrics exposé
- **Raison** : Pas de ServiceMonitor configuré
- **Solution demain** : Ajouter exporter + ServiceMonitor

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Installation Stack Complète**
```bash
# 1. Préparation
kubectl create namespace monitoring

# 2. Installation Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# 3. Installation stack
helm install prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword='admin123'
```

### **2. Vérification Installation**
```bash
# Attendre que tous les pods soient Running
watch kubectl get pods -n monitoring

# Vérifier les services
kubectl get svc -n monitoring
```

### **3. Accès aux Interfaces**
```bash
# Port-forward pour accès local
kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring &
kubectl port-forward svc/prometheus-stack-prometheus 9090:9090 -n monitoring &
```

### **4. Exploration Grafana**
1. **Connect** : http://localhost:3000 (admin/admin123)
2. **Browse dashboards** : Chercher "Kubernetes"
3. **Ouvrir** : "Kubernetes / Compute Resources / Cluster"
4. **Observer** : Métriques CPU, mémoire, pods en temps réel

### **5. Premières Requêtes Prometheus**
Dans http://localhost:9090 :
```promql
# Test 1: Combien de pods ?
count(kube_pod_info)

# Test 2: CPU utilisé
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Test 3: Pods par namespace
sum(kube_pod_info) by (namespace)
```

### **6. Déploiement App Test**
```bash
# Création namespace
kubectl create namespace app-monitoring-test

# Déploiement app simple
kubectl create deployment my-webapp --image=nginx:alpine -n app-monitoring-test
kubectl scale deployment my-webapp --replicas=3 -n app-monitoring-test
kubectl expose deployment my-webapp --port=80 -n app-monitoring-test

# Vérification
kubectl get all -n app-monitoring-test
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Installation**
- **Namespace dédié** : `monitoring` pour isolation
- **Helm simplifié** : kube-prometheus-stack = tout-en-un
- **Attendre readiness** : Tous pods `Running` avant utilisation
- **Port-forward** : Accès rapide pour développement

### **⚠️ Premiers Pas**
- **Grafana login** : admin/admin123 (à changer en production)
- **Prometheus UI** : Pour tests et debug, Grafana pour daily use
- **Dashboards prêts** : Utiliser ceux fournis avant de créer les siens
- **PromQL test** : Toujours tester dans Prometheus avant Grafana

### **🔧 Exploration Initiale**
1. **Vérifier targets** : http://localhost:9090/targets
2. **Dashboard cluster** : Premier dashboard à regarder
3. **Requêtes simples** : Commencer par `count()`, `sum()` 
4. **Labels exploration** : Comprendre la structure des métriques

### **📋 Checklist Jour 64**
- [ ] Namespace `monitoring` créé
- [ ] Stack Prometheus installée via Helm
- [ ] Tous pods `Running` dans namespace monitoring
- [ ] Accès Grafana fonctionnel (localhost:3000)
- [ ] Accès Prometheus fonctionnel (localhost:9090)
- [ ] Dashboard Kubernetes visualisé
- [ ] Premières requêtes PromQL exécutées
- [ ] Application test déployée (mais non monitorée)

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Prometheus ≠ Grafana**
**Deux outils distincts :**
- **Prometheus** : Base de données + moteur de requêtes
- **Grafana** : Interface de visualisation
- **Relation** : Grafana se connecte à Prometheus (et autres)

### **2. Pas de Magie**
**L'installation ne monitorera rien automatiquement :**
- Métriques système : OUI (via node-exporter)
- Métriques K8s : OUI (via kube-state-metrics)
- **Vos applications** : NON (à configurer explicitement)

### **3. Pull Model Avantages**
**Pour Kubernetes :**
- Pas besoin de connaître les IPs des pods
- Découverte automatique des nouveaux services
- Meilleure résilience (Prometheus retry si échec)

### **4. Écosystème Riche**
**kube-prometheus-stack fournit :**
- Monitoring du cluster (nodes, pods, ressources)
- Alerting de base (pré-configuré)
- Dashboards pré-créés (Grafana)
- Gestion automatique (Prometheus Operator)

---

## **📈 PROGRESSION JOUR 64**

### **✅ ACQUIS TECHNIQUES :**
- **Architecture Prometheus** : Compréhension du modèle pull-based
- **Installation stack** : Via Helm en une commande
- **Accès interfaces** : Grafana + Prometheus UI
- **PromQL basique** : `count()`, `sum()`, filtres par labels
- **Dashboard par défaut** : Visualisation métriques cluster

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Monitoring = logs et alertes manuelles"  
> **Aujourd'hui :** "Monitoring = **métriques multi-dimensionnelles** + **dashboards temps réel**"  
> **Résultat :** "Visibilité immédiate sur l'état du cluster"

### **🔗 ARCHITECTURE CONSTRUITE :**
```
STACK MONITORING INSTALLÉE :

NAMESPACE: monitoring
├── Prometheus Server (collecte)
│   ├── Scrape: node-exporter (métriques système)
│   ├── Scrape: kube-state-metrics (métriques K8s)
│   └── Storage: TSDB (séries temporelles)
│
├── Grafana (visualisation)
│   ├── Data source: Prometheus
│   ├── Dashboards: Kubernetes prédéfinis
│   └── Interface: Web UI
│
└── Alertmanager (prêt pour plus tard)
    ├── Gestion alertes
    └── Notifications

PROBLÈME IDENTIFIÉ :
└── Nos applications (comme my-webapp) → NON monitorées
    → Pas d'endpoint /metrics
    → Pas de ServiceMonitor configuré
```

### **🚀 POUR DEMAIN (JOUR 65) :**
- **Exposition métriques** : Ajouter /metrics à nos applications
- **ServiceMonitors** : Configurer la découverte automatique
- **Dashboards custom** : Créer nos propres visualisations
- **PromQL avancé** : `rate()`, `increase()`, agrégations
- **Application monitoring** : Monitorer enfin nos propres apps

---

## **💡 INSIGHTS FINAUX**

### **La Puissance du Modèle Pull**
**Prometheus transforme :**
- ❌ Configuration statique → ✅ Découverte dynamique
- ❌ Métriques plates → ✅ Multi-dimensionnelles avec labels
- ❌ Monitoring séparé → ✅ Stack intégrée cloud-native

### **Base Solide Posée**
**Aujourd'hui on a :**
1. **Infrastructure** : Stack complète installée
2. **Accès** : Interfaces opérationnelles
3. **Compréhension** : Architecture et concepts clés
4. **Problème identifié** : Comment monitorer NOS apps


---

## **📊 QUICK REFERENCE**

### **URLs d'accès :**
- **Grafana** : http://localhost:3000 (admin/admin123)
- **Prometheus** : http://localhost:9090
- **Alertmanager** : http://localhost:9093

### **Commandes de redémarrage :**
```bash
# Si les port-forward sont perdus
pkill -f "kubectl port-forward"

# Redémarrer
kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring &
kubectl port-forward svc/prometheus-stack-prometheus 9090:9090 -n monitoring &
```

### **Vérification rapide :**
```bash
# Tout va bien ?
kubectl get pods -n monitoring
curl -s http://localhost:3000/api/health | jq .
curl -s http://localhost:9090/-/healthy
```

---

**📊 Progress: `Jour 64 / 100 ✅`**

**#Kubernetes #Prometheus #Monitoring #Grafana #DevOps #SRE #CloudNative #Observability #PromQL**
