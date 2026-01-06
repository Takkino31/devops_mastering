# **JOUR 49 : INGRESS RULES AVANCÉES - ROUTING & TLS** 🔐

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Routing Avancé Ingress**
- **Path-based routing** : Direction basée sur le chemin URL
- **Host-based routing** : Direction basée sur le nom de domaine
- **Annotations Nginx** : Personnalisation du comportement

### **🔐 TLS Configuration**
- **Certificats Kubernetes** : Stockés dans des Secrets
- **Terminaison TLS** : Chiffrement au niveau Ingress
- **Redirections** : HTTP → HTTPS automatique

---

## **📊 Types de Routing**

| Type           | Description            | Exemple                             |
|----------------|------------------------|-------------------------------------|
| **Path-based** | Basé sur le chemin URL | `/api` → API, `/web` → Frontend      |
| **Host-based** | Basé sur le nom d'hôte | `api.domain.com`, `app.domain.com`  |
| **Mixte**      | Combinaison des deux   | `api.domain.com/v1`                 |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Gestion du Routing**
| Commande                         | Objectif                   | Exemple           |
|----------------------------------|----------------------------|-------------------|
| `kubectl apply -f ingress.yaml`  | Appliquer règles Ingress   | Déploiement       |
| `kubectl describe ingress <nom>` | Voir détails Ingress       | Configuration     |
| `kubectl get ingress`            | Lister toutes les règles   | Vue d'ensemble    |

### **🔐 TLS Management**
| Commande                        | Objectif                      | Exemple             |
|---------------------------------|-------------------------------|---------------------|
| `openssl req -x509 ...`         | Générer certificat auto-signé | Dev/test            |
| `kubectl create secret tls ...` | Créer Secret TLS              | Stockage certificat |
| `kubectl get secret <nom>`      | Vérifier Secret TLS           | Validation          |

### **🌐 Tests de Routing**
```bash
# Test path-based
curl http://<ip>:<port>/api

# Test host-based
curl -H "Host: app.domain.com" http://<ip>:<port>

# Test HTTPS
curl -k https://<ip>:<port>  # -k pour certificats auto-signés
```

---

## **📝 STRUCTURE DES RÈGLES AVANCÉES**

### **Path-based Routing :**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port: 80
```

### **Host-based Routing :**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-ingress
spec:
  rules:
  - host: "api.example.com"
    http:
      paths:
      - path: /
        backend:
          service:
            name: api-service
            port: 80
  - host: "app.example.com"
    http:
      paths:
      - path: /
        backend:
          service:
            name: web-service
            port: 80
```

### **TLS Configuration :**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - "secure.example.com"
    secretName: tls-secret  # Secret contenant certificat
  rules:
  - host: "secure.example.com"
    http:
      paths:
      - path: /
        backend:
          service:
            name: web-service
            port: 80
```

### **Annotations Courantes :**
```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Deux Types de Routing Complémentaires**
**Quand utiliser quoi :**
- **Path-based** : Applications dans même domaine, organisation par fonction
- **Host-based** : Séparation complète, multi-tenant, environnements séparés
- **Les deux** : Organisation complexe (`api.env.com/v1`)

### **2. TLS Simplified**
**Workflow typique :**
1. Générer certificat (auto-signé ou CA)
2. Créer Secret Kubernetes avec certificat
3. Référencer Secret dans l'Ingress
4. Ingress gère la terminaison TLS

**Avantage :** Application backend ne gère pas TLS

### **3. Headers HTTP Critiques**
**Pour host-based routing :**
```bash
# Sans header Host, routing impossible
curl -H "Host: app.example.com" http://IP
```
Le header `Host` est essentiel pour le routing par domaine.

### **4. Annotations = Puissance**
**Personnalisation sans modifier l'application :**
- Réécriture URLs
- Redirections
- Limitations
- Headers custom
- Configuration Nginx avancée

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Path-based Routing**
```yaml
# Définition simple
paths:
- path: /api
  backend:
    service:
      name: api-service
- path: /web
  backend:
    service:
      name: web-service
```

**Test :**
```bash
curl http://IP/api      # → API Service
curl http://IP/web      # → Web Service
```

### **2. Host-based Routing**
```yaml
# Utilisation du header Host
rules:
- host: "api.local"
  http:
    paths:
    - path: /
      backend:
        service:
          name: api-service
- host: "app.local"
  http:
    paths:
    - path: /
      backend:
        service:
          name: web-service
```

**Test :**
```bash
curl -H "Host: api.local" http://IP
curl -H "Host: app.local" http://IP
```

### **3. TLS Configuration**
```bash
# Génération certificat
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=app.local"

# Création Secret
kubectl create secret tls app-tls \
  --key tls.key --cert tls.crt
```

```yaml
# Référence dans Ingress
spec:
  tls:
  - hosts:
    - "secure.app.local"
    secretName: app-tls
```

### **4. Annotations de Base**
```yaml
metadata:
  annotations:
    # Redirection HTTPS
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    
    # Réécriture URL
    nginx.ingress.kubernetes.io/rewrite-target: /$1
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Organisation des Règles**
- **Nommage clair** : `api-ingress`, `web-ingress`, `tls-ingress`
- **Séparation logique** : Par environnement ou fonction
- **Documentation** : Commentaires dans YAML pour règles complexes

### **⚠️ Sécurité TLS**
- **Dev/Test** : Certificats auto-signés acceptables
- **Production** : Certificats valides (Let's Encrypt, CA)
- **Renouvellement** : Automatiser avec cert-manager
- **Redirections** : Toujours HTTP → HTTPS

### **🔧 Configuration**
- **Path types** : `Prefix` (par défaut) vs `Exact`
- **Backend services** : Doivent exister avant l'Ingress
- **Annotations** : Vérifier compatibilité version Nginx
- **Testing** : Tester tous les chemins et hosts

### **📋 Checklist Routing Avancé**
- [ ] Path-based routing testé
- [ ] Host-based routing testé
- [ ] TLS configuré si besoin
- [ ] Redirections HTTP→HTTPS fonctionnelles
- [ ] Annotations documentées
- [ ] Backend services accessibles
- [ ] DNS/hosts configurés pour tests

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Header Host Essentiel**
**Oublier = Échec :**
- Host-based routing nécessite header `Host`
- Tests doivent inclure `-H "Host: ..."`
- Navigateurs envoient automatiquement, curl non

### **2. TLS au Bon Niveau**
**Avantage Ingress :**
- Terminaison TLS centralisée
- Applications simplifiées (pas de gestion certs)
- Renouvellement centralisé
- Performance (hardware SSL possible)

### **3. Annotations Spécifiques**
**Vérifier la documentation :**
- Annotations dépendent du Ingress Controller
- Nginx ≠ Traefik ≠ autres
- Versions différentes = annotations différentes

### **4. Debug Simplement**
**Outils de base :**
```bash
# Voir configuration appliquée
kubectl describe ingress <nom>

# Voir logs du controller
kubectl logs -n ingress-nginx <controller-pod>

# Tester rapidement
curl -v -H "Host: ..." http://...
```

---

## **📈 PROGRESSION JOUR 49**

### **✅ ACQUIS TECHNIQUES :**
- **Routing path-based** : Organisation par chemin URL
- **Routing host-based** : Séparation par domaine
- **TLS configuration** : Certificats et Secrets
- **Annotations de base** : Réécriture, redirections
- **Tests avancés** : Curl avec headers, HTTPS

### **🎯 CHANGEMENT MENTAL :**
> **Hier :** "Je route simplement vers mes applications"  
> **Aujourd'hui :** "Je **choisis la stratégie de routing** adaptée à chaque cas"  
> **Résultat :** "Routing intelligent selon le besoin (chemin ou domaine)"

### **🔗 ARCHITECTURE CONSTRUITE :**
```
ROUTING INTELLIGENT :

OPTION 1 - PATH-BASED
http://monapp.com/api     → Service API
http://monapp.com/web     → Service Web
http://monapp.com/        → Service Web (défaut)

OPTION 2 - HOST-BASED
http://api.monapp.com     → Service API  
http://app.monapp.com     → Service Web

+ TLS OPTIONNEL
https://... avec certificat
```
---

## **💡 INSIGHTS FINAUX**

### **La Flexibilité du Routing**
**Ingress offre plusieurs stratégies :**
- **Simple** : Un chemin, une application
- **Organisé** : Par fonction (/api, /web)
- **Séparé** : Par domaine (api., app.)
- **Sécurisé** : Avec TLS

**Choix selon :**
- Architecture de l'application
- Besoins de séparation
- Contraintes DNS/domaines
- Exigences sécurité

### **Évolution Progressive**
**Du basique à l'avancé :**
1. **Routing simple** (un chemin)
2. **Multi-paths** (organisation)
3. **Multi-hosts** (séparation)
4. **TLS** (sécurité)
5. **Annotations** (optimisation)

**Aujourd'hui, nous maîtrisons les étapes 2-4.**

---

**📊 Progress: `Jour 49 / 100 ✅`**

**#Kubernetes #Ingress #Routing #TLS #HTTPS #Nginx #Networking #DevOps**
