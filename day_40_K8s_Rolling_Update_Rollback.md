# **JOUR 40 : ROLLING UPDATES & ROLLBACKS KUBERNETES** 🔄

**Durée : 90 minutes**

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Le Pouvoir des Rolling Updates**
- **Mise à jour sans interruption** : Remplacer la version d'une application sans downtime
- **Processus progressif** : Kubernetes remplace les Pods un par un pendant que l'application continue de servir du trafic
- **Contrôle fin** : Paramètres `maxSurge` et `maxUnavailable` pour adapter la stratégie à chaque besoin

### **🛡️ La Sécurité des Rollbacks**
- **Retour arrière garanti** : Si une nouvelle version a un bug, retour immédiat à la version précédente
- **Historique complet** : Toutes les versions déployées sont tracées et accessibles
- **Plan de secours intégré** : Plus besoin de procédures manuelles de restauration

---

## **📊 Les Deux Stratégies Principales**

| Stratégie | Comment ça marche | Quand l'utiliser | Avantage principal |
|-----------|-------------------|------------------|-------------------|
| **RollingUpdate** (défaut) | Remplace les Pods progressivement, un par un | Mises à jour courantes, applications critiques | **Zero downtime** garanti |
| **Recreate** | Supprime tous les Pods, puis recrée la nouvelle version | Maintenance, changements cassants | Simplicité, cohérence |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Gestion des Updates**
| Commande | Objectif | Résultat |
|----------|----------|----------|
| `kubectl set image deployment/...` | Changer la version d'une image | Lance un rolling update |
| `kubectl rollout status deployment/... --watch` | Surveiller une mise à jour en cours | Feedback en temps réel |
| `kubectl rollout pause deployment/...` | Mettre en pause un rollout | Permet d'inspecter avant de continuer |
| `kubectl rollout resume deployment/...` | Reprendre un rollout | Continue la mise à jour |

### **🔄 Gestion des Rollbacks**
| Commande                                  | Objectif                          | Usage typique                 |
|-------------------------------------------|-----------------------------------|-------------------------------|
| `kubectl rollout history deployment/...`  | Voir l'historique des versions    | Auditer les changements       |
| `kubectl rollout undo deployment/...`     | Annuler le dernier déploiement    | Bug critique détecté          |
| `kubectl rollout undo --to-revision=N`    | Revenir à une version spécifique  | Choix précis de la version    |

### **👁️ Inspection et Monitoring**
| Commande                          | Ce qu'elle révèle                 | Pourquoi c'est utile      |
|-----------------------------------|-----------------------------------|---------------------------|
| `kubectl get replicasets`         | Les différentes versions en cours | Comprendre la transition  |
| `kubectl describe deployment`     | Événements et statut détaillé     | Debug des problèmes       |
| `kubectl get pods --show-labels`  | Version de chaque Pod             | Vérifier la progression   |

---

## **📝 LES PARAMÈTRES CLÉS D'UN ROLLING UPDATE**

### **Dans le fichier YAML :**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          # ⚡ Crée max 1 Pod supplémentaire
    maxUnavailable: 0    # 🛡️ Zéro Pod indisponible (zero-downtime)
```

**Explication :**
- **`maxSurge: 1`** : "Tu peux créer 1 Pod de plus que le nombre désiré pendant l'update"
- **`maxUnavailable: 0`** : "Jamais plus de 0 Pod indisponible en même temps" → **100% disponibilité**

**Autres configurations courantes :**
- `maxSurge: 25%`, `maxUnavailable: 25%` → Update plus rapide, tolère un peu de downtime
- `minReadySeconds: 30` → Attendre 30s avant de considérer un nouveau Pod comme "prêt"

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Le Processus en 4 Étapes**
```
1. 🟢 Ancienne version : [v1.0] [v1.0] [v1.0] (3 Pods)
2. 🔄 Transition     : [v1.0] [v1.0] [v1.0] [v2.0] (+1 nouveau)
3. 🔄 Transition     : [v1.0] [v1.0] [v2.0] [v2.0] (-1 ancien, +1 nouveau)
4. 🟢 Nouvelle version: [v2.0] [v2.0] [v2.0] (transition complète)
```

**Pendant toute la transition :** L'application reste disponible à 100% !

### **2. L'Historique est Ton Ami**
```bash
# Voir TOUTES les versions déployées
kubectl rollout history deployment/mon-app

# Révision 1 : Version initiale
# Révision 2 : Premier update
# Révision 3 : Update avec bug (rollbacké)
# Révision 4 : Version corrigée

# Chaque révision est conservée et restaurable !
```

### **3. Les Rollbacks Sont Instantanés**
**Scénario :** Vous déployez la version 2.0, elle a un bug critique.
```bash
# Détection du problème
kubectl get pods  # Certains Pods en erreur

# Solution : 1 commande
kubectl rollout undo deployment/mon-app

# Résultat : En 30 secondes, retour à la version 1.0
# Le service n'a jamais été interrompu
```

### **4. Les Deux ReplicaSets**
Pendant un rolling update, **deux ReplicaSets coexistent** :
- L'ancien ReplicaSet : Gère les Pods de l'ancienne version
- Le nouveau ReplicaSet : Gère les Pods de la nouvelle version

À la fin de l'update, l'ancien ReplicaSet garde 0 réplica mais est conservé pour l'historique.

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Premier Rolling Update**
```bash
# Déployer la version 1.0
kubectl apply -f app-v1.yaml

# Lancer la mise à jour vers 2.0
kubectl set image deployment/mon-app nginx=nginx:1.26-alpine

# Observer en direct
kubectl rollout status deployment/mon-app --watch
# Waiting for deployment "mon-app" rollout to finish: 2 out of 3 new replicas...
# Waiting for deployment "mon-app" rollout to finish: 1 old replicas are pending...
# deployment "mon-app" successfully rolled out ✓
```

### **2. Surveillance des Deux Versions**
```bash
# Voir les deux ReplicaSets
kubectl get replicasets
# NAME               DESIRED   CURRENT   READY   AGE
# mon-app-5f6b89685  0         0         0       5m    # Ancien (v1.0)
# mon-app-7d8c9a12b  3         3         3       1m    # Nouveau (v2.0)

# Voir les labels pour identifier la version
kubectl get pods --show-labels | grep version
# mon-app-7d8c9a12b-abc1   version=2.0.0
# mon-app-7d8c9a12b-abc2   version=2.0.0
# mon-app-7d8c9a12b-abc3   version=2.0.0
```

### **3. Rollback de Sécurité**
```bash
# Simuler un bug (image inexistante)
kubectl set image deployment/mon-app nginx=nginx:version-inexistante

# Voir l'échec
kubectl get pods
# Certains Pods en ImagePullBackOff ou ErrImagePull

# Rollback immédiat
kubectl rollout undo deployment/mon-app

# Vérification
kubectl rollout status deployment/mon-app
# deployment "mon-app" successfully rolled out
```

### **4. Exploration de l'Historique**
```bash
# Voir toutes les révisions
kubectl rollout history deployment/mon-app
# REVISION  CHANGE-CAUSE
# 1         kubectl apply --filename=app-v1.yaml
# 2         kubectl set image deployment/mon-app nginx=nginx:1.26-alpine
# 3         kubectl set image deployment/mon-app nginx=nginx:version-inexistante

# Voir les détails d'une révision
kubectl rollout history deployment/mon-app --revision=2
# Affiche la configuration exacte de cette version
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Stratégies selon le Contexte**
- **Application critique** : `maxUnavailable: 0` → Zero downtime garanti
- **Update rapide** : `maxUnavailable: 1` → Plus rapide, tolère 1 Pod down
- **Migration majeure** : Considerer Blue-Green deployment
- **Nouvelle feature risquée** : Considerer Canary release

### **⚠️ Anti-patterns à Éviter**
```yaml
# ❌ Trop agressif
maxUnavailable: 50%  # Perd la moitié de la capacité!

# ❌ Pas assez de contrôle
# Pas de minReadySeconds → Trafic routé trop tôt

# ❌ Mauvaise pratique
image: nginx:latest  # Version ambiguë, reproductibilité faible
```

### **🔧 Configuration Production Recommandée**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1           # Un Pod supplémentaire maximum
    maxUnavailable: 0     # Zéro downtime
minReadySeconds: 30       # Attendre que l'app soit vraiment prête
revisionHistoryLimit: 10  # Garder 10 révisions pour rollback
```

### **📋 Checklist avant un Update**
1. **Backup** : Exporter la configuration actuelle
2. **Historique** : Vérifier les révisions disponibles
3. **Tests** : Valider la nouvelle version en pré-production
4. **Monitoring** : Préparer la surveillance pendant l'update
5. **Plan B** : Savoir exactement comment rollbacker

---

## **📈 PROGRESSION JOUR 40**

### **✅ ACQUIS TECHNIQUES :**
- **Maîtrise des Rolling Updates** : Mises à jour sans interruption de service
- **Rollbacks garantis** : Retour sécurisé à n'importe quelle version précédente
- **Monitoring temps réel** : Surveillance active des déploiements
- **Gestion d'historique** : Traçabilité complète des versions
- **Paramétrage avancé** : `maxSurge`, `maxUnavailable`, `minReadySeconds`

### **🎯 CHANGEMENT MENTAL :**
> **Les déploiements ne sont plus des moments de stress mais des opérations contrôlées**  
> **Je peux innover sans peur car je peux toujours revenir en arrière**  
> **La qualité de service est préservée pendant les changements, pas compromise par eux**

### **🔗 WORKFLOW PROFESSIONNEL ÉTABLI :**
```
DÉVELOPPEMENT → Nouvelle version testée
        ↓
DÉPLOIEMENT → kubectl apply (déclenche rolling update)
        ↓
SURVEILLANCE → kubectl rollout status --watch
        ↓
VALIDATION → Tests pendant la transition
        ↓
CONFIRMATION → Succès ou Rollback automatisé
```

### **🚀 POUR DEMAIN (JOUR 41-42) :**
- **Services Kubernetes** : Exposer vos applications
- **Load Balancing** : Distribution intelligente du trafic
- **DNS Interne** : Découverte de service automatique
- **Types de Services** : ClusterIP, NodePort, LoadBalancer
- **Communication inter-services** : Votre frontend parle à votre backend

---

**📊 Progress: `Jour 40 / 100 ✅`**

**#Kubernetes #RollingUpdate #Rollback #ZeroDowntime #DevOps #ContinuousDeployment #CloudNative #SiteReliability**

