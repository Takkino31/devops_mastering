# **JOUR 63 : PROJET MULTI-TENANT AVEC RBAC** 🏢

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Multi-Tenant Kubernetes**
- **Définition** : Un cluster unique hébergeant plusieurs clients/équipes/projets de manière isolée
- **Modèle namespace par tenant** : Approche simple et native à Kubernetes
- **Avantages** : Économies de coûts, simplicité de gestion, consistance des outils
- **Défis** : Isolation stricte, contrôle des ressources, prévention des fuites

### **🔧 Composants Essentiels Multi-Tenant**
- **ResourceQuotas** : Limitation des ressources par tenant
- **LimitRanges** : Contraintes par défaut sur les conteneurs
- **RBAC Hiérarchique** : Rôles adaptés à chaque type d'utilisateur
- **Cross-Namespace Access** : Accès contrôlé entre tenants pour services partagés

### **⚙️ ServiceAccount Token Projection**
- **Problème des tokens classiques** : Pas d'expiration, accès trop large
- **Solution** : Tokens JWT avec audience spécifique et expiration
- **Sécurité améliorée** : Tokens limités dans le temps et la portée

### **🔗 Services Partagés Cross-Tenant**
- **Monitoring** : Collecte de métriques depuis tous les tenants
- **Backup** : Sauvegarde des données de tous les tenants
- **Logging** : Centralisation des logs avec séparation par tenant
- **Administration** : Gestion centralisée de la plateforme

---

## **📊 Modèles d'Isolation Multi-Tenant**

| Modèle                            | Approche                              | Avantages                             | Cas d'usage                                       |
|-----------------------------------|---------------------------------------|---------------------------------------|---------------------------------------------------|
| **Namespace par Tenant**          | Isolation via namespaces Kubernetes   | Simple, natif, bon support outillage  | SaaS basique, équipes internes                    |
| **Virtual Clusters (vClusters)**  | Cluster virtuel par tenant            | Isolation forte, frontières claires   | Clients enterprise, compliance stricte            |
| **Hybride**                       | Mix des deux approches                | Flexibilité, adapté aux besoins       | Environnements complexes, évolution progressive   |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Configuration Multi-Tenant**
| Commande | Objectif | Exemple |
|----------|----------|---------|
| `kubectl create namespace tenant-<nom>` | Créer namespace tenant | `tenant-acme` |
| `kubectl create quota <nom> --hard=pods=50,services=10 --namespace=<tenant>` | Définir quotas | Limites ressources |
| `kubectl create clusterrole <nom> --verb=<actions> --resource=<ressources>` | Créer rôles cross-tenant | `monitoring-role` |

### **🔍 Audit et Validation**
| Commande | Ce qu'elle révèle | Pourquoi c'est utile |
|----------|-------------------|----------------------|
| `kubectl auth can-i <action> --as=system:serviceaccount:<tenant>:<sa> -n <autre-tenant>` | Test isolation tenant | Validation sécurité |
| `kubectl describe quota -n <tenant>` | Utilisation quotas | Monitoring ressources |
| `kubectl get rolebindings --all-namespaces -o wide` | Voir tous les bindings | Audit RBAC complet |

### **🏗️ Token Projection**
```bash
# Vérifier les Pods sans token projection
kubectl get pods --all-namespaces -o jsonpath='{range .items[?(@.spec.automountServiceAccountToken!=false)]}{.metadata.namespace}/{@.metadata.name}{"\n"}{end}'

# Examiner un token projeté
kubectl exec <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token | cut -d '.' -f 2 | base64 -d | jq
```

---

## **📝 STRUCTURES DE CONFIGURATION AVANCÉES**

### **ResourceQuota par Tenant :**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-acme
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "50"
    services: "20"
    configmaps: "50"
    persistentvolumeclaims: "10"
    secrets: "30"
```

### **ClusterRole avec Aggregation (Platform Admin) :**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-admin-role
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.platform.io/aggregate-to-platform-admin: "true"

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-admin-rules
  labels:
    rbac.platform.io/aggregate-to-platform-admin: "true"
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

### **ServiceAccount avec Token Projection :**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: tenant-acme
spec:
  template:
    spec:
      automountServiceAccountToken: false  # Désactive token classique
      serviceAccountName: app-deployer
      containers:
      - name: app
        image: nginx:alpine
        volumeMounts:
        - name: kube-api-access
          mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          readOnly: true
      volumes:
      - name: kube-api-access
        projected:
          sources:
          - serviceAccountToken:
              audience: "kubernetes"
              expirationSeconds: 3600  # Expire après 1h
              path: token
          - configMap:
              name: kube-root-ca.crt
              items:
              - key: ca.crt
                path: ca.crt
          - downwardAPI:
              items:
              - path: namespace
                fieldRef:
                  fieldPath: metadata.namespace
```

### **RoleBinding pour Service Partagé :**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitoring-tenant-acme
  namespace: tenant-acme  # Dans le namespace du tenant
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: monitoring-role  # Role avec permissions lecture
subjects:
- kind: ServiceAccount
  name: monitoring-agent
  namespace: shared-services  # ServiceAccount dans namespace partagé
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Isolation par Design**
**Approche fondamentale :**
- **Namespace comme frontière** : Séparation logique claire
- **RBAC comme gardien** : Contrôle d'accès granulaire
- **Quotas comme limite** : Prévention du "noisy neighbor"
- **Network Policies** : Isolation réseau additionnelle

### **2. RBAC Hiérarchique Indispensable**
**Rôles typiques dans une plateforme multi-tenant :**
- **Platform Admin** : Administration complète (aggregation de rôles)
- **Tenant Admin** : Tout dans son namespace (gestion locale)
- **App Deployer** : Déploiement applications (limité)
- **Viewer** : Lecture seule (audit, support)
- **Monitoring Agent** : Lecture cross-tenant (services partagés)
- **Backup Agent** : Lecture + snapshots (services partagés)

### **3. Token Projection pour Sécurité Avancée**
**Avantages par rapport aux tokens classiques :**
- ✅ **Expiration** : Tokens limités dans le temps
- ✅ **Audience spécifique** : Tokens pour usage spécifique
- ✅ **Moindre privilège** : Tokens avec permissions minimales
- ✅ **Rotation automatique** : Pas de gestion manuelle
- ✅ **Traçabilité** : Meilleur audit des accès

### **4. Services Partagés avec Accès Contrôlé**
**Pattern essentiel :**
- **ServiceAccount dédié** dans namespace `shared-services`
- **ClusterRole avec permissions limitées** (lecture, snapshots)
- **RoleBinding dans chaque namespace tenant**
- **Isolation préservée** : Services ne peuvent que lire, pas modifier

### **5. Évolution Progressive**
**Du simple au complexe :**
1. **Namespace isolation** → Aujourd'hui
2. **Resource Quotas** → Contrôle ressources
3. **Network Policies** → Isolation réseau
4. **Pod Security Standards** → Sécurité runtime
5. **vClusters** → Isolation plus forte si nécessaire

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Architecture Multi-Tenant Complète**
```bash
# Structure namespace
kubectl create namespace platform-admin
kubectl create namespace shared-services
kubectl create namespace tenant-acme
kubectl create namespace tenant-globex
kubectl create namespace tenant-soylent

# Quotas par tenant
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-acme
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    pods: "20"
EOF
```

### **2. RBAC Hiérarchique avec Rôles Spécialisés**
```yaml
# Rôles pour différents besoins
---
# Tenant Admin (tout dans son namespace)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tenant-admin-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["*"]
  verbs: ["*"]

---
# Monitoring (lecture cross-tenant)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-role
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services"]
  verbs: ["get", "list", "watch"]

---
# Platform Admin (aggregation)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-admin-role
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.platform.io/aggregate-to-platform-admin: "true"
```

### **3. ServiceAccount Token Projection**
```yaml
# Application avec token sécurisé
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: tenant-acme
spec:
  template:
    spec:
      automountServiceAccountToken: false
      serviceAccountName: app-deployer
      containers:
      - name: app
        image: nginx:alpine
        volumeMounts:
        - name: kube-api-access
          mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      volumes:
      - name: kube-api-access
        projected:
          sources:
          - serviceAccountToken:
              expirationSeconds: 3600
              path: token
```

### **4. Tests d'Isolation et Validation**
```bash
# Test isolation tenant
kubectl auth can-i get pods --as=system:serviceaccount:tenant-acme:tenant-admin -n tenant-globex
# no (bon - isolation fonctionne)

# Test services partagés
kubectl auth can-i get pods --as=system:serviceaccount:shared-services:monitoring-agent -n tenant-acme
# yes (bon - monitoring peut lire)

kubectl auth can-i delete pods --as=system:serviceaccount:shared-services:monitoring-agent -n tenant-acme
# no (bon - monitoring seulement lecture)

# Test quotas
kubectl describe quota -n tenant-acme
# Vérifier utilisation vs limites
```

### **5. Audit RBAC Multi-Tenant**
```bash
# Script d'audit
#!/bin/bash
echo "=== Audit RBAC Multi-Tenant ==="

# ClusterRoleBindings dangereux
kubectl get clusterrolebindings | grep -v "system:" | grep cluster-admin

# Cross-namespace access
for ns in $(kubectl get ns -o name | cut -d/ -f2); do
  echo "Namespace: $ns"
  kubectl get rolebindings -n $ns -o yaml | grep -B2 -A2 "namespace:"
done

# Tokens non projetés
kubectl get pods --all-namespaces -o jsonpath='{range .items[?(@.spec.automountServiceAccountToken!=false)]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'
```

---

## **🎯 BEST PRACTICES PRODUCTION MULTI-TENANT**

### **✅ Architecture de Base**
- **Namespace par tenant** : Séparation logique claire
- **ResourceQuotas obligatoires** : Prévention noisy neighbor
- **LimitRanges par défaut** : Contraintes conteneurs
- **RBAC hiérarchique** : Rôles adaptés aux besoins réels
- **Network Policies** : Isolation réseau additionnelle

### **⚠️ Sécurité Avancée**
- **Token Projection** : Tokens avec expiration
- **Pod Security Standards** : Restricted en production
- **Secrets externalisés** : Vault ou solutions cloud
- **Audit logging activé** : Traçabilité complète
- **Scan régulier images** : Prévention vulnérabilités

### **🔧 Services Partagés**
- **Monitoring centralisé** : Métriques tous tenants
- **Logging centralisé** : Logs avec séparation
- **Backup automatisé** : Par tenant, testé régulièrement
- **Service Mesh** : Pour communication sécurisée inter-services
- **API Gateway** : Exposition contrôlée des APIs

### **📋 Checklist Production Multi-Tenant**
- [ ] **Isolation namespace** : Un namespace par tenant
- [ ] **Resource Quotas** : Définis et monitorés
- [ ] **RBAC configuré** : Rôles hiérarchiques adaptés
- [ ] **Network Policies** : Isolation réseau configurée
- [ ] **Token Projection** : Tokens sécurisés
- [ ] **Monitoring cross-tenant** : ServiceAccount dédié
- [ ] **Backup cross-tenant** : ServiceAccount dédié
- [ ] **Audit logging** : Activé et centralisé
- [ ] **Alerting** : Sur violations quotas/RBAC
- [ ] **Documentation** : Architecture et runbooks
- [ ] **Tests isolation** : Réguliers et automatisés
- [ ] **Plan de recovery** : Par tenant et global

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Multi-Tenancy ≠ Simplement Plusieurs Namespaces**
**Approche holistique nécessaire :**
- **Isolation** : RBAC + Network Policies + Quotas
- **Gouvernance** : Policies, quotas, limites
- **Observabilité** : Monitoring, logging, tracing par tenant
- **Opérations** : Backup, recovery, scaling par tenant
- **Sécurité** : Defense in depth spécifique multi-tenant

### **2. Importance des Services Partagés**
**Services qui doivent traverser les frontières :**
- **Monitoring** : Vue globale pour operations
- **Logging** : Centralisation pour compliance
- **Backup** : Protection données tous tenants
- **Security scanning** : Détection menaces globale
- **Cost management** : Tracking coûts par tenant

### **3. Évolution avec la Croissance**
**Scaling de la plateforme :**
- **Petite échelle** : Namespaces + RBAC basique
- **Moyenne échelle** : Quotas + Network Policies
- **Grande échelle** : vClusters + Service Mesh
- **Enterprise** : Multi-cluster + GitOps + Policy as Code

### **4. Balance Isolation vs Partage**
**Trade-off à gérer :**
- **Plus d'isolation** → Plus de sécurité, plus de complexité
- **Plus de partage** → Plus d'efficacité, plus de risque
- **Bon équilibre** : Selon besoins sécurité et compliance
- **Évolution possible** : Commencer isolé, partager progressivement

---

## **📈 PROGRESSION JOUR 63**

### **✅ ACQUIS TECHNIQUES :**
- **Architecture multi-tenant complète** : Design et implémentation
- **RBAC hiérarchique avancé** : Rôles spécialisés par besoin
- **Resource management** : Quotas et limites par tenant
- **Token security** : Projection avec expiration et audience
- **Services partagés** : Monitoring et backup cross-tenant
- **Audit et validation** : Tests isolation et sécurité

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Un cluster pour une application/équipe"  
> **Aujourd'hui :** "Une **plateforme multi-tenant** qui héberge plusieurs clients de manière **sécurisée et isolée**"  
> **Résultat :** "**Efficacité opérationnelle** avec **sécurité garantie**"

### **🔗 PLATEFORME IMPLÉMENTÉE :**
```
PLATEFORME MULTI-TENANT ENTERPRISE :

STRUCTURE FONDAMENTALE :
├── PLATFORM ADMIN : Administration globale
├── SHARED SERVICES : Monitoring, Backup, Logging
├── TENANT SPACES : Namespaces isolés par client
│   ├── Resource Quotas : Limites ressources
│   ├── RBAC Hiérarchique : Rôles adaptés
│   ├── Security Contexts : Sécurité runtime
│   └── Network Policies : Isolation réseau

SERVICES PARTAGÉS INTELLIGENTS :
├── MONITORING : Lecture cross-tenant, pas d'écriture
├── BACKUP : Snapshots cross-tenant, données protégées
├── LOGGING : Centralisation avec séparation
└── SECURITY : Scanning global, protection tous tenants

GOUVERNANCE ET OPÉRATIONS :
├── QUOTA MANAGEMENT : Prévention noisy neighbor
├── COST TRACKING : Attribution par tenant
├── COMPLIANCE : Audit logging centralisé
└── DISASTER RECOVERY : Plans par tenant et global
```

### **🚀 POUR LA PRODUCTION ENTERPRISE :**
- **Multi-cluster** : Distribution géographique, isolation renforcée
- **Service Mesh** : Istio/Linkerd pour sécurité réseau avancée
- **GitOps** : ArgoCD pour gestion déclarative multi-tenant
- **Policy as Code** : OPA/Gatekeeper pour gouvernance
- **Chaos Engineering** : Tests résilience par tenant
- **FinOps** : Optimisation coûts et chargeback
- **SOC 2/ISO 27001** : Frameworks compliance multi-tenant

---

## **💡 INSIGHTS FINAUX**

### **La Puissance de la Plateforme Multi-Tenant**
**Ce que cela permet :**
- ✅ **Efficacité opérationnelle** : Un seul cluster à gérer
- ✅ **Économies d'échelle** : Partage coûts infrastructure
- ✅ **Consistance** : Mêmes outils, mêmes processus tous clients
- ✅ **Rapidité déploiement** : Nouveaux clients en minutes
- ✅ **Sécurité centralisée** : Protection uniforme tous tenants

### **Les Prochaines Évolutions**
**Roadmap typique :**
1. **Multi-tenant basique** → Aujourd'hui ✓
2. **Automation et self-service** → Portail client, APIs
3. **Advanced isolation** → vClusters, service mesh
4. **Enterprise features** : SSO, compliance, chargeback
5. **Platform as a Product** : Commercialisation externe

---

**📊 Progress: `Jour 63 / 100 ✅`**

**#Kubernetes #MultiTenancy #PlatformEngineering #RBAC #DevOps #SRE #CloudNative #EnterpriseArchitecture #Security #ResourceManagement**
