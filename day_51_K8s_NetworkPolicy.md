# **JOUR 51 : INTRODUCTION AUX NETWORK POLICIES** 🔒

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Problème de Sécurité Kubernetes**
- **Communication libre par défaut** : Tous les Pods peuvent communiquer entre eux
- **Risques de sécurité** : Lateral movement, data exfiltration
- **Besoins d'isolation** : Séparation entre environnements, services

### **🔧 Network Policies Solution**
- **Pare-feu pour Pods** : Contrôle trafic entrant/sortant
- **Principe Zero-Trust** : "Ne jamais faire confiance, toujours vérifier"
- **Basé sur les labels** : Sélection des Pods affectés
- **Additif** : Seulement ce qui est explicitement autorisé

### **⚙️ Prérequis Technique**
- **CNI plugins supportés** : Calico, Cilium, Weave Net
- **Pas Flannel par défaut** : Nécessite configuration spécifique
- **Minikube avec Calico** : `minikube start --network-plugin=cni --cni=calico`

---

## **📊 Composants d'une Network Policy**

| Composant         | Description               | Exemple                   |
|-------------------|---------------------------|---------------------------|
| **podSelector**   | Quels Pods sont affectés  | `app: frontend`           |
| **policyTypes**   | Types de règles           | `Ingress`, `Egress`       |
| **ingress**       | Règles trafic entrant     | Autoriser depuis...       |
| **egress**        | Règles trafic sortant     | Autoriser vers...         |
| **ports**         | Ports autorisés           | `port: 80`, `port: 5432`  |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Installation & Configuration**
| Commande                                                  | Objectif              | Exemple                       |
|-----------------------------------------------------------|-----------------------|-------------------------------|
| `minikube start --network-plugin=cni --cni=calico`        | Démarrer avec Calico  | Prérequis Network Policies    |
| `kubectl get pods -n kube-system -l k8s-app=calico-node`  | Vérifier Calico       | Installation réussie          |

### **🔍 Gestion des Policies**
| Commande                               | Ce qu'elle révèle    | Pourquoi c'est utile |
|----------------------------------------|----------------------|----------------------|
| `kubectl get networkpolicies`          | Liste des policies   | Vue d'ensemble       |
| `kubectl describe networkpolicy <nom>` | Détails d'une policy | Règles spécifiques   |
| `kubectl apply -f policy.yaml`         | Appliquer une policy | Déploiement          |

### **🌐 Tests de Connectivité**
```bash
# Test simple
kubectl exec <pod> -- nc -zv <service> <port>

# Test HTTP
kubectl exec <pod> -- wget -qO- <service>:<port>

# Test ping
kubectl exec <pod> -- ping <ip>
```

---

## **📝 STRUCTURE DES NETWORK POLICIES**

### **Policy Basique Deny-All :**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}  # Tous les Pods
  policyTypes:
  - Ingress
  - Egress
  # Pas de règles = tout bloqué
```

### **Policy Autorisant DNS :**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

### **Policy pour Application :**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
spec:
  podSelector:
    matchLabels:
      app: frontend
  ingress:
  - ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 8080
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. DNS Critique**
**Sans DNS, pas de résolution de noms :**
- Les Pods doivent accéder au service DNS (kube-dns)
- Ports 53 UDP/TCP
- Sans cela, même les communications autorisées échouent
- Toujours inclure une règle DNS dans les policies

### **2. Philosophie Zero-Trust**
**Par défaut : tout refuser**
1. Commencer avec `deny-all`
2. Ajouter DNS pour tous
3. Autoriser spécifiquement chaque communication nécessaire
4. Tester chaque règle

### **3. Labels Essentiels**
**Sélection basée sur les labels :**
- Les Pods doivent avoir des labels cohérents
- Les policies utilisent `matchLabels` pour cibler
- Organisation claire des labels nécessaire

### **4. Additivité des Policies**
**Pas de "deny" explicite :**
- Les policies sont cumulatives (union des règles)
- Si un Pod a plusieurs policies, toutes s'appliquent
- Impossible de dire "refuser ce trafic", seulement "ne pas l'autoriser"

### **5. Isolation Progressive**
**Approche recommandée :**
1. Tout permettre (développement)
2. Deny-all + DNS (staging)
3. Micro-segmentation complète (production)
4. Tests réguliers de sécurité

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Installation Calico**
```bash
# Démarrer Minikube avec Calico
minikube start --network-plugin=cni --cni=calico

# Vérification
kubectl get pods -n kube-system -l k8s-app=calico-node
# Doit montrer des pods Calico running
```

### **2. Deny-All Policy**
```yaml
# Bloquer tout trafic par défaut
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Effet :** Plus aucune communication Pod-to-Pod possible

### **3. Autorisation DNS**
```yaml
# Permettre DNS à tous les Pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

### **4. Architecture 3-Tiers Sécurisée**
```yaml
# Frontend → Backend seulement
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-backend
spec:
  podSelector:
    matchLabels:
      app: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 8080

# Backend → Database seulement
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-database
spec:
  podSelector:
    matchLabels:
      app: backend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

### **5. Tests de Sécurité**
```bash
# Frontend peut accéder au Backend ?
kubectl exec frontend-pod -- nc -zv backend-service 8080

# Frontend peut accéder à la Database ? (devrait échouer)
kubectl exec frontend-pod -- nc -zv database-service 5432

# Backend peut accéder à la Database ?
kubectl exec backend-pod -- nc -zv database-service 5432
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Approche Progressive**
- **Développement** : Peu ou pas de restrictions
- **Staging** : Deny-all + DNS + règles basiques
- **Production** : Micro-segmentation complète
- **Documentation** : Règles expliquées et justifiées

### **⚠️ Points d'Attention**
- **DNS obligatoire** : Oublier = problèmes de résolution
- **Labels cohérents** : Essentiels pour la sélection
- **Tests réguliers** : Vérifier que les règles fonctionnent
- **Backward compatibility** : Ne pas casser les applications existantes

### **🔧 Organisation**
- **Nommage clair** : `deny-all`, `allow-dns`, `frontend-egress`
- **Séparation logique** : Une policy par fonction/équipe
- **Documentation inline** : Commentaires dans YAML
- **Version control** : Toutes les policies dans Git

### **📋 Checklist Network Policy**
- [ ] Calico/Cilium installé et fonctionnel
- [ ] Deny-all policy appliquée
- [ ] DNS autorisé pour tous
- [ ] Règles spécifiques pour chaque communication nécessaire
- [ ] Labels cohérents sur tous les Pods
- [ ] Tests de connectivité réussis
- [ ] Documentation des règles de sécurité

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Sécurité ≠ Complexité**
**Simple mais efficace :**
- Deny-all + DNS = sécurité de base
- Ajouter règles au besoin
- Mieux vaut trop restrictif que trop permissif

### **2. DNS oublié = Tout cassé**
**Erreur courante :**
- Les applications échouent silencieusement
- Les erreurs ne sont pas évidentes
- Toujours tester la résolution DNS

### **3. Isolation par Défaut**
**Changer la mentalité :**
- Avant : "Tout est permis sauf si interdit"
- Après : "Tout est interdit sauf si permis"
- Plus sécurisé, plus contrôlé

### **4. Compatibilité CNI**
**Pas universel :**
- Flannel (défaut Minikube) : ❌ Network Policies
- Calico, Cilium : ✅ Network Policies
- Vérifier avant de déployer

---

## **📈 PROGRESSION JOUR 51**

### **✅ ACQUIS TECHNIQUES :**
- **Installation Calico** : CNI plugin pour Network Policies
- **Philosophie Zero-Trust** : Deny-all par défaut
- **Création de policies** : Structure YAML et règles
- **DNS configuration** : Essentiel pour le fonctionnement
- **Tests de sécurité** : Vérification des restrictions
- **Architecture sécurisée** : Isolation 3-tiers

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Mes services communiquent librement"  
> **Aujourd'hui :** "J'**isole explicitement** chaque communication"  
> **Résultat :** "**Sécurité proactive** plutôt que réactive"

### **🔗 ARCHITECTURE CONSTRUITE :**
```
SÉCURITÉ RÉSEAU KUBERNETES :

PHILOSOPHIE ZERO-TRUST
├── Par défaut : Tout refuser
├── Explicitement : Autoriser seulement le nécessaire
└── Vérification : Tester chaque règle

IMPLEMENTATION
├── CNI Plugin : Calico installé
├── Policies :
│   ├── deny-all : Blocage général
│   ├── allow-dns : DNS pour tous
│   ├── frontend→backend : Communication spécifique
│   └── backend→database : Communication spécifique
└── Résultat : Isolation contrôlée
```

### **🚀 POUR DEMAIN (JOUR 52) :**
- **Policies avancées** : Namespace isolation, egress controls
- **Micro-segmentation fine** : Contrôle par labels complexes
- **Multi-namespace** : Communication cross-namespace
- **Projet complet** : Architecture sécurisée complète
- **Debug tools** : Outils Calico pour investigation

---

## **💡 INSIGHTS FINAUX**

### **La Sécurité comme Feature**
**Network Policies transforment :**
- ❌ Sécurité réactive → ✅ Sécurité proactive
- ❌ Trust implicite → ✅ Vérification explicite
- ❌ Risque élevé → ✅ Contrôle granulé

### **Équilibre Sécurité/Productivité**
**Trouver le bon niveau :**
- **Dev** : Minimal, pour rapidité
- **Test** : Modéré, pour validation
- **Prod** : Maximal, pour sécurité
- **Évolution** : Augmenter graduellement

### **Prochain Niveau**
**Pour production avancée :**
1. **CI/CD intégration** : Tests automatiques des policies
2. **Compliance scanning** : Vérification règles de sécurité
3. **Audit logging** : Traçabilité des communications
4. **Policy as Code** : Gestion déclarative

**Aujourd'hui, nous avons posé les bases.** Demain, nous approfondissons avec l'isolation avancée.

---

**📊 Progress: `Jour 51 / 100 ✅`**

**#Kubernetes #NetworkPolicies #Security #ZeroTrust #Calico #ContainerSecurity #DevOps #CloudNative**
