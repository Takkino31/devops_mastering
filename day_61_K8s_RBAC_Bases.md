# **JOUR 61 : RBAC FONDAMENTAUX** 🔐

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Le Problème de Sécurité Sans RBAC**
- **Accès trop permissif** : Par défaut, les comptes peuvent avoir trop de permissions
- **Pas de séparation des responsabilités** : Tous les utilisateurs/applications ont les mêmes droits
- **Manque d'auditabilité** : Difficile de savoir qui fait quoi

### **🔧 La Solution : RBAC (Role-Based Access Control)**
- **ServiceAccounts** : Identités pour les applications dans Kubernetes
- **Roles/ClusterRoles** : Définition des permissions (namespace vs cluster)
- **RoleBindings/ClusterRoleBindings** : Attribution des rôles aux comptes
- **Principle of Least Privilege** : Donner seulement les permissions nécessaires

### **⚙️ Composants du Système RBAC**
- **Roles** : Permissions limitées à un namespace spécifique
- **ClusterRoles** : Permissions applicables à tout le cluster
- **ServiceAccounts** : Identifiants utilisés par les Pods et applications
- **Bindings** : Lien entre les comptes et les rôles

---

## **📊 Différences Clés RBAC**

| Composant                 | Portée    | Utilisation                   | Exemple                           |
|---------------------------|-----------|-------------------------------|-----------------------------------|
| **ServiceAccount**        | Namespace | Identité pour applications    | `monitoring-sa`, `backend-sa`     |
| **Role**                  | Namespace | Permissions dans 1 namespace  | Accès aux pods dans `dev`         |
| **ClusterRole**           | Cluster   | Permissions globales          | Voir tous les namespaces          |
| **RoleBinding**           | Namespace | Lie Role à ServiceAccount     | monitoring-sa → pod-reader        |
| **ClusterRoleBinding**    | Cluster   | Lie ClusterRole à sujet       | system:masters → cluster-admin    |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Configuration RBAC**
| Commande                                                                          | Objectif              | Exemple                               |
|-----------------------------------------------------------------------------------|-----------------------|---------------------------------------|
| `kubectl create serviceaccount <nom>`                                             | Créer ServiceAccount  | `monitoring-sa`                       |
| `kubectl create role <nom> --verb=<actions> --resource=<ressources>`              | Créer Role            | `pod-reader`                          |
| `kubectl create rolebinding <nom> --role=<role> --serviceaccount=<namespace:sa>`  | Créer RoleBinding     | Lier `monitoring-sa` à `pod-reader`   |

### **🔍 Vérification des Permissions**
| Commande                                                          | Ce qu'elle révèle                     | Pourquoi c'est utile          |
|-------------------------------------------------------------------|---------------------------------------|-------------------------------|
| `kubectl auth can-i <verbe> <ressource>`                          | Teste une permission spécifique       | Validation avant déploiement  |
| `kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>`  | Liste toutes les permissions d'un SA  | Audit complet                 |
| `kubectl describe role <nom>`                                     | Détails d'un Role                     | Vérification des permissions  |

### **🏗️ Création RBAC Complète**
```bash
# Création séquentielle
kubectl create namespace rbac-test
kubectl create serviceaccount monitoring-sa -n rbac-test
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n rbac-test
kubectl create rolebinding monitoring-binding --role=pod-reader --serviceaccount=rbac-test:monitoring-sa -n rbac-test
```

---

## **📝 STRUCTURE YAML RBAC**

### **ServiceAccount (Identité) :**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring-sa
  namespace: rbac-test
```

### **Role (Permissions Namespace) :**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: rbac-test
rules:
- apiGroups: [""]           # "" = groupe API core
  resources: ["pods"]       # Ressource concernée
  verbs: ["get", "list", "watch"]  # Actions autorisées
```

### **RoleBinding (Attribution) :**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitoring-binding
  namespace: rbac-test
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader          # Référence au Role
subjects:
- kind: ServiceAccount
  name: monitoring-sa       # ServiceAccount concerné
  namespace: rbac-test
```

### **ClusterRole (Permissions Globales) :**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-viewer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. ServiceAccount ≠ User Account**
**Différence fondamentale :**
- **ServiceAccounts** : Pour les applications, Pods, automations
- **User Accounts** : Pour les humains (gérés par le fournisseur d'identité)
- **Par défaut** : Chaque namespace a un ServiceAccount `default`

### **2. Principe de Moindre Privilège**
**Approche recommandée :**
- Commencer avec **zéro permissions**
- Ajouter **seulement ce qui est nécessaire**
- **Tester avec `kubectl auth can-i`** avant déploiement
- **Auditer régulièrement** les permissions

### **3. Rôles Prédéfinis Kubernetes**
**ClusterRoles intégrés :**
- **view** : Lecture seule de la plupart des ressources
- **edit** : Lecture/écriture (sauf RBAC et quotas)
- **admin** : Accès complet dans un namespace (sauf quotas)
- **cluster-admin** : **DANGEREUX** - Accès complet au cluster

### **4. Portée des Permissions**
**Namespace vs Cluster :**
- **Role + RoleBinding** = Permissions limitées à un namespace
- **ClusterRole + RoleBinding** = Permissions ClusterRole mais limitées au namespace
- **ClusterRole + ClusterRoleBinding** = Permissions globales

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Création de ServiceAccounts**
```bash
# Création namespace test
kubectl create namespace rbac-test

# Création ServiceAccounts
kubectl create serviceaccount monitoring-sa -n rbac-test
kubectl create serviceaccount deployment-sa -n rbac-test
kubectl create serviceaccount read-only-sa -n rbac-test

# Vérification
kubectl get serviceaccounts -n rbac-test
# NAME               SECRETS   AGE
# default            1         10s
# monitoring-sa      1         10s  
# deployment-sa      1         10s
# read-only-sa       1         10s
```

### **2. Création de Roles Spécifiques**
```yaml
# Role pour lecture seule des pods
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: rbac-test
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# Role pour gestion complète des deployments
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: rbac-test
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

### **3. Attribution avec RoleBindings**
```yaml
# Lier monitoring-sa à pod-reader
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitoring-binding
  namespace: rbac-test
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: rbac-test
```

### **4. Tests de Permissions**
```bash
# Tester des permissions spécifiques
kubectl auth can-i get pods --as=system:serviceaccount:rbac-test:monitoring-sa -n rbac-test
# yes

kubectl auth can-i create pods --as=system:serviceaccount:rbac-test:monitoring-sa -n rbac-test
# no

# Voir toutes les permissions
kubectl auth can-i --list --as=system:serviceaccount:rbac-test:monitoring-sa -n rbac-test

# Tester depuis un Pod
kubectl exec -it test-pod -n rbac-test -- \
  kubectl get pods -n rbac-test
# Fonctionne si le Pod utilise monitoring-sa
```

### **5. Audit RBAC Existante**
```bash
# Voir les ClusterRoles prédéfinis
kubectl get clusterroles | grep -E "(admin|edit|view|cluster-admin)"

# Voir les permissions de "view"
kubectl describe clusterrole view

# Voir les ClusterRoleBindings
kubectl get clusterrolebindings -o wide

# Voir ses propres permissions
kubectl auth can-i --list
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Configuration Sécurisée**
- **ServiceAccount dédié par application** : Ne pas utiliser `default`
- **Roles spécifiques** : Éviter les permissions trop larges
- **Namespace isolation** : Limiter au namespace nécessaire
- **Principle of Least Privilege** : Donner seulement le nécessaire

### **⚠️ Sécurité**
- **Éviter cluster-admin** : Très dangereux en production
- **Auditer régulièrement** : Vérifier qui a accès à quoi
- **Tester les permissions** : Avant et après les changements
- **Documenter les rôles** : Pour la maintenabilité

### **🔧 Monitoring & Audit**
- **Utiliser `kubectl auth can-i`** : Pour tester les permissions
- **Vérifier les ClusterRoleBindings** : Qui a des permissions globales
- **Logs d'audit** : Activer pour suivre les accès
- **Revues régulières** : Des permissions RBAC

### **📋 Checklist RBAC Basique**
- [ ] ServiceAccount créé pour chaque application
- [ ] Roles définis avec permissions minimales
- [ ] RoleBindings correctement configurés
- [ ] Permissions testées avec `kubectl auth can-i`
- [ ] Audit des ClusterRoles/ClusterRoleBindings
- [ ] Documentation des permissions
- [ ] Tests de sécurité validés

---

## **🔍 LEÇONS IMPORTANTES**

### **1. RBAC ≠ Optionnel en Production**
**Importance critique :**
- Sans RBAC : Sécurité compromise
- Avec RBAC mal configuré : Peut bloquer les applications
- Bonne configuration : Sécurité + Fonctionnalité

### **2. ServiceAccounts sont Essentiels**
**Pour les applications :**
- Identité unique par application
- Tokens automatiquement injectés dans les Pods
- Meilleure traçabilité et audit
- Séparation des responsabilités

### **3. Tester Avant de Déployer**
**Approche recommandée :**
1. Créer les objets RBAC
2. Tester avec `kubectl auth can-i`
3. Tester depuis un Pod de test
4. Déployer en production
5. Continuer à monitorer

### **4. Évolution avec la Croissance**
**Scaling RBAC :**
- Début : Rôles simples par équipe
- Croissance : Rôles plus granulaires
- Entreprise : RBAC avec groupes, hiérarchie
- Production avancée : Intégration SSO, audit détaillé

---

## **📈 PROGRESSION JOUR 61**

### **✅ ACQUIS TECHNIQUES :**
- **Architecture RBAC** : Compréhension des 4 composants
- **ServiceAccounts** : Création et utilisation pratique
- **Roles/RoleBindings** : Définition et attribution permissions
- **Tests de permissions** : Commande `kubectl auth can-i`
- **Audit RBAC** : Analyse configuration existante
- **Bonnes pratiques** : Principle of Least Privilege appliqué

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Tout le monde peut tout faire dans le cluster"  
> **Aujourd'hui :** "Chaque application a **seulement les permissions nécessaires**"  
> **Résultat :** "**Sécurité granulaire** et **auditabilité** améliorées"

### **🔗 SYSTÈME IMPLÉMENTÉ :**
```
SYSTÈME RBAC FONDAMENTAL :

IDENTITÉS (ServiceAccounts)
├── monitoring-sa → Surveillance
├── deployment-sa → Déploiements
└── read-only-sa → Lecture seule

PERMISSIONS (Roles)
├── pod-reader : pods [get, list, watch]
├── deployment-manager : deployments [full]
└── config-viewer : configmaps [get, list]

ATTRIBUTION (RoleBindings)
├── monitoring-sa → pod-reader
├── deployment-sa → deployment-manager
└── read-only-sa → config-viewer

RÉSULTAT : Contrôle d'accès granulaire, sécurité améliorée
```

### **🚀 POUR DEMAIN (JOUR 62) :**
- **RBAC avancé** : ClusterRoles, agrégation de rôles
- **Security Contexts** : Contrôle privilèges des conteneurs
- **Pod Security Standards** : Niveaux de sécurité production
- **Multi-équipes** : Architecture RBAC pour plusieurs équipes
- **Audit avancé** : Monitoring et logging des accès

---

## **💡 INSIGHTS FINAUX**

### **La Puissance du Contrôle Granulaire**
**RBAC permet :**
- ✅ **Sécurité fine** : Permissions spécifiques par application
- ✅ **Auditabilité** : Savoir qui fait quoi
- ✅ **Séparation des responsabilités** : Équipes différentes = permissions différentes
- ✅ **Protection contre les erreurs** : Applications ne peuvent pas tout faire

### **Les Prochaines Étapes**
**Évolution naturelle :**
1. **RBAC basique** → Aujourd'hui ✓
2. **RBAC avancé** → Demain (Security Contexts, PSS)
3. **Intégration SSO** → Connexion avec Active Directory, OAuth
4. **GitOps RBAC** : Gestion des permissions via Git
5. **Policy as Code** : OPA/Gatekeeper pour politiques complexes

---

**📊 Progress: `Jour 61 / 100 ✅`**

**#Kubernetes #RBAC #Security #ServiceAccount #RoleBasedAccessControl #DevOps #K8sSecurity #LeastPrivilege**
