# **JOUR 43 : CONFIGMAPS - CONFIGURATION EXTERNALISÉE** ⚙️

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Externalisation de la Configuration**
- **Principe 12-factor** : Stocker la configuration dans l'environnement
- **Séparation code/config** : Même image Docker pour tous les environnements
- **Flexibilité** : Changer la configuration sans rebuild

### **🔧 Les ConfigMaps Kubernetes**
- **Stockage config non-sensible** : Variables, fichiers de configuration
- **Deux méthodes d'injection** : Variables d'environnement et volumes
- **Mise à jour** : Possible mais pas automatique

---

## **📊 Types de ConfigMaps**

| Type                  | Utilisation        | Exemple                                |
|-----------------------|--------------------|----------------------------------------|
| **Clé-valeur**        | Variables simples  | `APP_NAME="MonApp"`                    |
| **Fichier complet**   | Fichiers de config | `nginx.conf`, `application.properties` |
| **Dossier**           | Multiples fichiers | `config/` avec plusieurs fichiers      |

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🎯 Création de ConfigMaps**
| Commande                                                       | Objectif                  | Exemple                              |
|----------------------------------------------------------------|---------------------------|--------------------------------------|
| `kubectl create configmap <nom> --from-literal=<clé>=<valeur>` | Créer depuis littéraux    | `--from-literal=APP_NAME="MonApp"`   |
| `kubectl create configmap <nom> --from-file=<fichier>`         | Créer depuis un fichier   | `--from-file=config.properties`      |
| `kubectl create configmap <nom> --from-env-file=<fichier>`     | Créer depuis fichier .env | `--from-env-file=.env`               |

### **🔍 Inspection et Gestion**
| Commande                           | Ce qu'elle révèle        | Pourquoi c'est utile  |
|------------------------------------|--------------------------|-----------------------|
| `kubectl get configmaps`           | Liste des ConfigMaps     | Vue d'ensemble        |
| `kubectl describe configmap <nom>` | Détails d'un ConfigMap   | Contenu complet       |
| `kubectl edit configmap <nom>`     | Éditer un ConfigMap      | Modification rapide   |

### **🔄 Utilisation dans les Pods**
| Méthode                       | Quand l'utiliser      | Exemple YAML                    |
|-------------------------------|-----------------------|---------------------------------|
| **Variables d'environnement** | Valeurs simples       | `env.valueFrom.configMapKeyRef` |
| **Toutes les variables**      | Toute la config       | `envFrom.configMapRef`          |
| **Volume (fichier)**          | Fichiers de config    | `volumes.configMap`             |

---

## **📝 STRUCTURE DES CONFIGMAPS**

### **ConfigMap YAML de base :**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Variables simples
  APP_NAME: "Mon Application"
  ENVIRONMENT: "development"
  LOG_LEVEL: "DEBUG"
  
  # Fichier de configuration complet
  application.properties: |
    server.port=8080
    database.url=jdbc:mysql://localhost:3306/db
    cache.enabled=true
```

### **Injection en variables d'environnement :**
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: nginx:alpine
    env:
    # Variable spécifique
    - name: APP_NAME
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_NAME
    
    # Toutes les variables
    - envFrom:
      - configMapRef:
          name: app-config
```

### **Montage comme volume :**
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: nginx:alpine
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. Même Image, Configuration Différente**
```bash
# Image Docker unique
image: monapp:1.0.0

# Configurations différentes via ConfigMaps
dev-configmap → ENVIRONMENT="development"
prod-configmap → ENVIRONMENT="production"
staging-configmap → ENVIRONMENT="staging"
```

### **2. Pas de Mise à Jour Automatique**
**Important :**
- Variables d'environnement : Jamais mises à jour après création du Pod
- Volumes : Mises à jour après délai (cache TTL)
- Solution : Redéployer les Pods après modification

```bash
# Après modification d'un ConfigMap
kubectl rollout restart deployment/<nom>
```

### **3. Limites de Taille**
- ConfigMaps limités à ~1MB dans etcd
- Pour configurations plus grandes : utiliser des Volumes persistants

### **4. Pas pour les Données Sensibles**
- ConfigMaps stockent en clair
- Pour secrets : utiliser l'objet **Secret** (jour suivant)

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Création de ConfigMaps**
```bash
# Depuis littéraux
kubectl create configmap app-settings \
  --from-literal=APP_NAME="TodoApp" \
  --from-literal=VERSION="1.0.0"

# Depuis un fichier
echo "theme=dark" > ui.properties
kubectl create configmap ui-config --from-file=ui.properties

# Via YAML
kubectl apply -f configmap.yaml
```

### **2. Application avec Configuration**
```yaml
# Déploiement configurable
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  template:
    spec:
      containers:
      - name: webapp
        image: nginx:alpine
        env:
        - name: ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: ENVIRONMENT
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
      volumes:
      - name: config
        configMap:
          name: nginx-config
```

### **3. Test de l'Application**
```bash
# Déployer
kubectl apply -f deployment-with-config.yaml

# Vérifier la configuration
kubectl exec deployment/webapp -- env | grep ENVIRONMENT
kubectl exec deployment/webapp -- cat /etc/nginx/conf.d/default.conf

# Modifier et redéployer
kubectl edit configmap app-config
kubectl rollout restart deployment/webapp
```

---

## **🎯 BEST PRACTICES DÉCOUVERTES**

### **✅ Organisation**
- **Nommage clair** : `<app>-config`, `<component>-config`
- **Regroupement logique** : Config liées ensemble
- **Séparation env/dev/prod** : ConfigMaps différents par environnement

### **⚠️ Sécurité**
- **Pas de secrets** : Utiliser l'objet Secret pour données sensibles
- **Contrôle d'accès** : RBAC pour limiter l'accès aux ConfigMaps

### **🔧 Maintenance**
- **Versioning** : Avec le code de l'application
- **Documentation** : Contenu et usage des ConfigMaps
- **Tests** : Vérifier la configuration dans CI/CD

### **📋 Checklist**
- [ ] Config externalisée du code
- [ ] Pas de données sensibles
- [ ] Nommage significatif
- [ ] Documentation
- [ ] Tests de configuration

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Séparation des Préoccupations**
**Code ≠ Configuration ≠ Données**
- Code : Dans l'image Docker
- Configuration : Dans ConfigMaps
- Données : Dans Volumes persistants

### **2. Portabilité Maximale**
**Même image pour :**
- Développement local
- Environnements de test
- Production
→ Seule la configuration change

### **3. Cycle de Vie Différent**
**Application vs Configuration :**
- Application : Build → Push → Deploy
- Configuration : Edit → Apply → (Restart)
→ Cycles indépendants

### **4. Préparation pour les Secrets**
**ConfigMaps = Données non sensibles**
**Secrets = Données sensibles**
→ Mêmes mécanismes d'injection

---

## **📈 PROGRESSION JOUR 43**

### **✅ ACQUIS TECHNIQUES :**
- **Externalisation configuration** : Séparation code/config
- **Création ConfigMaps** : 4 méthodes différentes
- **Injection dans Pods** : Variables d'env et volumes
- **Gestion cycle de vie** : Mise à jour et redéploiement
- **Bonnes pratiques** : Organisation, sécurité, maintenance

### **🎯 CHANGEMENT MENTAL :**
> **Avant :** "Ma configuration est dans mon code/dans mon image"  
> **Maintenant :** "Ma configuration est **externalisée** et **dynamique**"  
> **Résultat :** "Je change la config **sans rebuild** mon application"

### **🔗 ARCHITECTURE CONSTRUITE :**
```
APPLICATION CONFIGURABLE :

IMAGE DOCKER (code applicatif)
    ↓
CONFIGMAPS (configuration)
    ↓ (injection)
PODS (instance configurée)
    ↓
SERVICES (accès)
```

### **🚀 POUR DEMAIN (JOUR 44) :**
- **Secrets Kubernetes** : Gestion des données sensibles
- **Types de Secrets** : Opaque, docker-registry, tls
- **Sécurité** : Encodage base64 et ses limites
- **Best practices** : Rotation, accès sécurisé
- **Projet complet** : App avec config + secrets

---

## **💡 INSIGHTS FINAUX**

### **La Puissance de la Configuration Dynamique**
**ConfigMaps permettent :**
- ✅ Adaptation sans rebuild
- ✅ Multi-environnements facile
- ✅ Gestion centralisée
- ✅ Versioning séparé

### **Les Limites à Connaître**
- ⚠️ Pas pour données sensibles
- ⚠️ Taille limitée
- ⚠️ Mise à jour non instantanée
- ⚠️ Stockage en clair

**Prochaine étape :** Combiner avec les **Secrets** pour une gestion complète configuration + sécurité.

---

**📊 Progress: `Jour 43 / 100 ✅`**

**#Kubernetes #ConfigMaps #ConfigurationManagement #DevOps #12FactorApps #ExternalConfiguration #CloudNative #InfrastructureAsCode**
