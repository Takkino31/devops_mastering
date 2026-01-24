# **JOUR 62 : RBAC AVANCÉ ET SECURITY CONTEXTS** 🔐

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Principle of Least Privilege Appliqué**
- **Approche fondamentale** : Donner seulement les permissions absolument nécessaires
- **Multi-niveaux** : RBAC (utilisateurs/applications) + Security Contexts (conteneurs)
- **Défense en profondeur** : Couches de sécurité empilées pour protection maximale

### **🔧 Pod Security Standards (PSS)**
- **Niveaux standardisés** : 
  - **Privileged** : Aucune restriction (dangereux)
  - **Baseline** : Restrictions minimales (développement)
  - **Restricted** : Sécurité maximale (production)
- **Enforcement par namespace** : Labels pour appliquer automatiquement les standards

### **⚙️ Security Contexts**
- **Contrôle au niveau Pod/Container** : Limitation des privilèges d'exécution
- **Options critiques** : `runAsNonRoot`, `allowPrivilegeEscalation`, `readOnlyRootFilesystem`
- **Capabilities management** : Suppression des privilèges système non nécessaires

### **🔗 Défense en Profondeur**
- **Multiples couches** : RBAC + Security Contexts + Network Policies + PSS
- **Audit logging** : Traçabilité complète des accès et actions
- **Outils spécialisés** : kube-bench (CIS compliance), kube-hunter (pen testing)

---

## **📊 Pod Security Standards - Niveaux de Sécurité**

| Caractéristique   | Privileged | Baseline             | Restricted            |
|-------------------|------------|----------------------|-----------------------|
| **Privilèges**    | Tous       | Limités              | Très limités          |
| **Capabilities**  | Toutes     | Certaines supprimées | La plupart supprimées |
| **RunAsNonRoot**  | Non requis | Non requis           | **Requis**            |
| **Seccomp**       | Optionnel  | Optionnel            | **Requis**            |
| **Usage**         | Systèmes   | Dev/Legacy           | **Production**        |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Configuration Security Contexts**
| Commande                                                                      | Objectif               | Exemple                    |
|-------------------------------------------------------------------------------|------------------------|----------------------------|
| `kubectl label namespace <nom> pod-security.kubernetes.io/enforce=<niveau>`   | Appliquer PSS          | `baseline` ou `restricted` |
| `kubectl describe namespace <nom>`                                            | Vérifier labels PSS    | Audit configuration        |
| `kubectl get pods -o yaml \| grep -A 5 securityContext`                       | Voir Security Contexts | Validation déploiements    |

### **🔍 Audit Sécurité**
| Commande                                                                                                                                                         | Ce qu'elle révèle     | Pourquoi c'est utile     |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|--------------------------|
| `kubectl get pods --all-namespaces -o jsonpath='{range .items[?(@.spec.securityContext.runAsNonRoot!=true)]}{.metadata.namespace}/{.metadata.name}{"\\n"}{end}'` | Pods exécutés en root | Détection vulnérabilités |
| `kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>`                                                                                                 | Toutes permissions SA | Audit RBAC complet       |
| `kubectl get events --field-selector reason=FailedCreate`                                                                                                        | Échecs création Pods  | Détection violations PSS |

### **🏗️ Configuration Multi-Équipes**
```bash
# Labels PSS par environnement
kubectl label namespace dev pod-security.kubernetes.io/enforce=baseline
kubectl label namespace prod pod-security.kubernetes.io/enforce=restricted

# Création RBAC hiérarchique
kubectl create clusterrole team-dev-role --verb="*" --resource="*"
kubectl create rolebinding dev-binding --clusterrole=team-dev-role --serviceaccount=dev:dev-sa -n dev
```

---

## **📝 STRUCTURES AVANCÉES DE SÉCURITÉ**

### **Security Context Maximum Sécurité :**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: production
spec:
  template:
    spec:
      securityContext:
        runAsUser: 1000           # User non-root
        runAsGroup: 1000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault    # Profil seccomp sécurisé
      containers:
      - name: app
        image: myapp:latest
        securityContext:
          runAsNonRoot: true      # Garantie non-root
          allowPrivilegeEscalation: false  # Pas d'élévation
          readOnlyRootFilesystem: true     # Système fichiers lecture seule
          capabilities:
            drop: ["ALL"]         # Supprime toutes capabilities
          seccompProfile:
            type: RuntimeDefault
```

### **RBAC Multi-Équipes :**
```yaml
# ClusterRole pour équipe Dev
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: team-dev-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["*"]
  verbs: ["*"]                    # Tout dans son namespace
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]          # Lecture seule secrets

# RoleBinding spécifique
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: team-dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: team-dev-role
subjects:
- kind: ServiceAccount
  name: dev-team-sa
  namespace: team-dev
```

### **Pod Security Standards Labels :**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted      # Enforcement strict
    pod-security.kubernetes.io/enforce-version: latest  # Dernière version
    pod-security.kubernetes.io/audit: restricted        # Audit niveau
    pod-security.kubernetes.io/warn: restricted         # Warning niveau
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Défense en Profondeur Essentielle**
**Une seule couche ≠ Suffisante :**
- **RBAC seul** : Conteneurs pourraient encore exécuter en root
- **Security Contexts seul** : Utilisateurs pourraient déployer n'importe quoi
- **PSS seul** : Pas de contrôle fin des permissions
- **Combinaison** : Protection maximale à tous les niveaux

### **2. Pod Security Standards Automatique**
**Avantages des PSS :**
- **Validation automatique** : Kubernetes rejette les Pods non conformes
- **Standards cohérents** : Même niveau de sécurité par environnement
- **Évolution facile** : Changement de niveau via labels
- **Audit intégré** : Warning et audit modes pour transition

### **3. Security Contexts Granulaires**
**Contrôle à plusieurs niveaux :**
- **Pod level** : `runAsUser`, `fsGroup`, `seccompProfile`
- **Container level** : `runAsNonRoot`, `capabilities`, `readOnlyRootFilesystem`
- **Volumes** : `readOnly` mount pour fichiers temporaires
- **Combinaison** : Sécurité maximale avec compatibilité

### **4. RBAC Hiérarchique Multi-Équipes**
**Architecture scalable :**
- **Équipe Dev** : Accès complet à `dev`, lecture à `staging`
- **Équipe QA** : Accès complet à `staging`, lecture à `dev` et `prod`
- **Équipe Prod** : Accès limité à `prod` seulement
- **Admin** : Lecture globale, pas d'écriture

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Architecture Multi-Équipes Complète**
```bash
# Création namespaces et équipes
kubectl create namespace team-dev
kubectl create namespace team-qa  
kubectl create namespace team-prod
kubectl create namespace admin

# ServiceAccounts par équipe
kubectl create serviceaccount dev-team-sa -n team-dev
kubectl create serviceaccount qa-team-sa -n team-qa
kubectl create serviceaccount prod-team-sa -n team-prod
kubectl create serviceaccount cluster-admin-sa -n admin
```

### **2. Configuration RBAC Hiérarchique**
```yaml
# ClusterRoles adaptés
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: team-dev-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["*"]
  verbs: ["*"]                    # Tout sauf...
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]          # Secrets lecture seule

# RoleBindings avec namespace cible
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: team-dev             # Seulement dans dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: team-dev-role
subjects:
- kind: ServiceAccount
  name: dev-team-sa
  namespace: team-dev
```

### **3. Security Contexts par Niveau de Sécurité**
**Développement (baseline) :**
```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  allowPrivilegeEscalation: false
```

**Production (restricted) :**
```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
```

### **4. Pod Security Standards Enforcement**
```bash
# Dev - Baseline (permissif)
kubectl label namespace team-dev \
  pod-security.kubernetes.io/enforce=baseline \
  pod-security.kubernetes.io/enforce-version=latest

# Prod - Restricted (strict)
kubectl label namespace team-prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest

# Test enforcement
kubectl run test --image=nginx -n team-prod
# Échoue si pas de securityContext
```

### **5. Tests de Permissions et Sécurité**
```bash
# Tester RBAC équipes
kubectl auth can-i create deployment --as=system:serviceaccount:team-dev:dev-team-sa -n team-dev
# yes

kubectl auth can-i get pods --as=system:serviceaccount:team-qa:qa-team-sa -n team-dev  
# yes (lecture cross-namespace)

kubectl auth can-i delete pods --as=system:serviceaccount:team-prod:prod-team-sa -n team-prod
# no (prod limité)

# Tester Security Contexts
kubectl exec deploy/secure-app -- whoami
# nginx (non-root)

kubectl exec deploy/insecure-app -- whoami
# root (alerte sécurité)
```

### **6. Outils d'Audit Sécurité**
```bash
# kube-bench (CIS compliance)
docker run --rm aquasec/kube-bench:latest run

# kube-hunter (penetration testing)
docker run --rm aquasec/kube-hunter:latest --remote <cluster-ip>

# Audit manuel
kubectl get pods --all-namespaces -o jsonpath='{range .items[?(@.spec.securityContext.runAsNonRoot!=true)]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}'
```

---

## **🎯 BEST PRACTICES PRODUCTION**

### **✅ Configuration Sécurisée par Défaut**
- **RBAC activé** : Toujours, sans exception
- **ServiceAccounts dédiés** : Jamais utiliser `default`
- **Security Contexts obligatoires** : `runAsNonRoot: true` minimum
- **PSS labels** : Sur tous les namespaces

### **⚠️ Sécurité Maximale Production**
- **Principle of Least Privilege** : Appliqué à tous les niveaux
- **readOnlyRootFilesystem** : Quand possible
- **Capabilities droppées** : `drop: ["ALL"]`, ajouter seulement si nécessaire
- **Seccomp profiles** : `RuntimeDefault` minimum
- **Network Policies** : En plus de RBAC

### **🔧 Monitoring & Audit**
- **Audit logging activé** : Tous les accès API
- **Alertes sécurité** : Sur violations RBAC/PSS
- **Scans réguliers** : kube-bench mensuel
- **Pen testing** : kube-hunter trimestriel
- **Revues RBAC** : Trimestrielles

### **📋 Checklist Hardening Production**
- [ ] **RBAC** configuré avec least privilege
- [ ] **Security Contexts** sur tous les déploiements
- [ ] **PSS labels** sur tous les namespaces
- [ ] **Network Policies** pour isolation
- [ ] **Audit logging** activé et monitoré
- [ ] **Secrets management** sécurisé
- [ ] **Images scannées** pour vulnérabilités
- [ ] **Runtime protection** (Falco/ACS)
- [ ] **Backup/DR** testé régulièrement
- [ ] **Incident response** plan documenté

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Sécurité ≠ Optionnel**
**En production :**
- Une vulnérabilité peut compromettre tout le cluster
- RBAC mal configuré = porte ouverte aux attaquants
- Security Contexts absents = conteneurs privilégiés
- **Investissement nécessaire** : Temps et ressources pour bien faire

### **2. Défense en Profondeur Réelle**
**Couches nécessaires :**
1. **RBAC** : Contrôle qui fait quoi
2. **Security Contexts** : Contrôle ce que peuvent faire les conteneurs
3. **PSS** : Standards minimum par défaut
4. **Network Policies** : Contrôle communication réseau
5. **Runtime security** : Détection anomalies exécution

### **3. Équilibre Sécurité vs Productivité**
**Trouver le bon niveau :**
- **Dev** : Plus permissif, focus développement
- **Staging** : Intermédiaire, validation sécurité
- **Prod** : Maximum sécurité, stabilité prioritaire
- **Évolution** : Renforcer graduellement avec la maturité

### **4. Automatisation Critique**
**Sécurité automatisée :**
- **PSS** : Rejet automatique Pods non conformes
- **Admission Controllers** : Validation policies custom
- **CI/CD pipelines** : Tests sécurité avant déploiement
- **GitOps** : Security as Code, review avant application

---

## **📈 PROGRESSION JOUR 62**

### **✅ ACQUIS TECHNIQUES :**
- **RBAC avancé** : Architecture multi-équipes hiérarchique
- **Security Contexts** : Configuration granulaire Pod/Container
- **Pod Security Standards** : Niveaux baseline/restricted et enforcement
- **Défense en profondeur** : Combinaison multiples couches sécurité
- **Outils d'audit** : kube-bench, kube-hunter, audit manuel
- **Hardening production** : Checklist complète sécurité

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Je configure la sécurité au cas par cas"  
> **Aujourd'hui :** "J'implémente une **défense en profondeur** avec sécurité par défaut"  
> **Résultat :** "Cluster **résilient aux attaques** avec **contrôle granulaire**"

### **🔗 ARCHITECTURE SÉCURITÉ IMPLÉMENTÉE :**
```
DÉFENSE EN PROFONDEUR KUBERNETES PRODUCTION :

COUCHE 1 : IDENTITÉ & ACCÈS (RBAC)
├── ServiceAccounts dédiés par application
├── Roles spécifiques avec least privilege
├── Multi-équipes avec isolation namespace
└── Audit logging complet

COUCHE 2 : SÉCURITÉ RUNTIME (SECURITY CONTEXTS)
├── runAsNonRoot obligatoire
├── No privilege escalation
├── readOnlyRootFilesystem quand possible
└── Capabilities minimales

COUCHE 3 : STANDARDS (POD SECURITY STANDARDS)
├── Baseline pour développement
├── Restricted pour production
├── Enforcement automatique
└── Audit et warning

COUCHE 4 : RÉSEAU & ISOLATION
├── Network Policies strictes
├── Services ClusterIP par défaut
├── Ingress avec TLS/WAF
└── Service Mesh pour sécurité avancée

COUCHE 5 : MONITORING & RESPONSE
├── Audit logs centralisés
├── Alertes temps réel
├── Scans réguliers sécurité
└── Incident response plan
```

### **🚀 POUR DEMAIN (JOUR 63) :**
- **Projet multi-tenant complet** : Architecture production avec isolation
- **ServiceAccount token projection** : Sécurisation avancée des tokens
- **Cross-namespace access patterns** : Best practices accès entre équipes
- **Intégration SSO/IDP** : Connexion avec providers externes
- **Policy as Code** : OPA/Gatekeeper pour policies complexes
- **GitOps sécurité** : Gestion RBAC via Git et ArgoCD

---

## **💡 INSIGHTS FINAUX**

### **La Nécessité de la Sécurité Stratifiée**
**Kubernetes expose de nombreuses surfaces d'attaque :**
- API Server : RBAC essentiel
- Conteneurs runtime : Security Contexts critiques
- Réseau pod-to-pod : Network Policies nécessaires
- Images containers : Scanning requis
- **Seule une approche multi-couches protège complètement**

### **Évolution avec la Maturité Organisationnelle**
**Niveaux de maturité sécurité :**
1. **Basique** : RBAC + Security Contexts simples
2. **Standard** : PSS + Network Policies + Audit
3. **Avancé** : Admission Controllers + Runtime Security + GitOps
4. **Expert** : Zero Trust + Service Mesh + Continuous Pen Testing
5. **Resilient** : Chaos Engineering + Red Team + Automated Response

---

**📊 Progress: `Jour 62 / 100 ✅`**

**#Kubernetes #Security #RBAC #PodSecurity #SecurityContext #DevSecOps #CloudSecurity #LeastPrivilege #DefenseInDepth #K8sHardening**
