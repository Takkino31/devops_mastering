# **JOUR 87 : COLLECTION ET STRUCTURATION DES LOGS**

## **🎯 CONCEPTS CLÉS APPRIS**

### **📊 Logs Structurés vs Non-Structurés**
- **Logs non-structurés** (texte libre) : Difficiles à filtrer, impossible d'agréger, parsing complexe
- **Logs structurés** (JSON) : Filtrage précis, agrégation possible, parsing automatique, corrélation facile

**Exemple de log structuré :**
```json
{
  "timestamp": "2026-03-03T10:30:45Z",
  "level": "ERROR",
  "service": "auth-api",
  "action": "login",
  "user_id": "12345",
  "error": "database timeout",
  "duration_ms": 5000
}
```

### **🔄 Pipeline de Traitement des Logs**
1. **Découverte** : Promtail identifie les pods à surveiller
2. **Filtrage** : Ignorer certains logs (DEBUG en production)
3. **Parsing** : Extraire des champs (regex, JSON)
4. **Labels** : Ajouter des métadonnées (namespace, pod, service)
5. **Envoi** : À Loki pour stockage et indexation

### **🔍 LogQL Avancé**

| Opération         | Syntaxe                | Utilisation                                  |
|-------------------|------------------------|----------------------------------------------|
| **Filtre texte**  | `\|=`                  | `{app="api"} \|= "ERROR"`                    |
| **Regex**         | `\|~`                  | `{app="api"} \|~ "ERR\|WARN"`                |
| **Exclusion**     | `!=`                   | `{app="api"} != "DEBUG"`                     |
| **Parsing JSON**  | `\| json`              | `{app="api"} \| json \| user_id="123"`       |
| **Comptage**      | `count_over_time()`    | `count_over_time({app="api"}[5m])`           |
| **Taux**          | `rate()`               | `rate({app="api"}[1m])`                      |
| **Quantile**      | `quantile_over_time()` | `quantile_over_time(0.95, duration_ms[5m])`  |
| **Agrégation**    | `sum by (label)`       | `sum by (service) (count_over_time(...))`    |

---

## **📊 Architecture d'Exploitation des Logs**

```
┌─────────────────────────────────────────────────────────────┐
│                   EXPLOITATION DES LOGS                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [ APPLICATIONS JSON ]                                      │
│  ├── auth-api : logs de connexion                           │
│  ├── order-api : logs de commandes                          │
│  └── json-logger : générateur de logs variés                │
│                                                             │
│  [ PROMTAIL - PIPELINE ]                                    │
│  ├── 1. Découverte des pods                                 │
│  ├── 2. Filtrage (ignorer les logs inutiles)                │
│  ├── 3. Parsing JSON → extraction champs                    │
│  └── 4. Envoi à Loki avec labels enrichis                   │
│                                                             │
│  [ LOKI ]                                                   │
│  ├── Stockage optimisé                                      │
│  ├── Indexation par labels                                  │
│  └── Moteur de requêtes LogQL                               │
│                                                             │
│  [ GRAFANA ]                                                │
│  ├── Explore : requêtes ad-hoc                              │
│  ├── Dashboards : visualisation                             │
│  │   ├── Volume de logs par service                         │
│  │   ├── Taux d'erreurs                                     │
│  │   ├── Latence 95e percentile                             │
│  │   ├── Activité utilisateur                               │
│  │   └── Erreurs en direct                                  │
│  └── Alerting : détection de pics d'erreurs                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. La Révolution des Logs Structurés**

**Avant (texte libre) :**
```
2026-03-03T10:30:45Z INFO User 12345 logged in from 192.168.1.100 in 245ms
```
→ Pour extraire l'utilisateur : regex complexe et fragile
→ Pour calculer le temps moyen : parsing manuel
→ Pour corréler avec d'autres services : impossible

**Après (JSON) :**
```json
{"timestamp":"...","level":"INFO","user_id":"12345","duration_ms":245}
```
→ Filtrage direct : `user_id="12345"`
→ Agrégation : `avg(duration_ms) by (service)`
→ Corrélation : `trace_id` commun entre services

### **2. Le Pipeline Promtail : Usine à Transformation**

Promtail n'est pas qu'un simple collecteur. C'est une **chaîne de traitement** :

```yaml
pipeline_stages:
- regex:                          # Étape 1 : Extraction par regex
    expression: '^(?P<time>.*) (?P<level>.*) (?P<message>.*)$'
- json:                            # Étape 2 : Parsing JSON si présent
    expressions:
      user_id: user_id
      duration_ms: duration_ms
- metrics:                          # Étape 3 : Métriques en temps réel
    error_count:
      type: Counter
      source: level
      value: ERROR
- labels:                           # Étape 4 : Ajout de labels
    level:
```

Chaque étape transforme les logs avant qu'ils n'arrivent dans Loki.

### **3. La Puissance des Agrégations Temporelles**

LogQL permet de traiter les logs comme des séries temporelles :

| Fonction               | Rôle                     | Exemple                                       |
|------------------------|--------------------------|-----------------------------------------------|
| `count_over_time()`    | Nombre d'événements      | `count_over_time({app="api"}[5m])`            |
| `rate()`               | Événements par seconde   | `rate({app="api"}[1m])`                       |
| `avg_over_time()`      | Moyenne sur période      | `avg_over_time(duration_ms[5m])`              |
| `quantile_over_time()` | Distribution             | `quantile_over_time(0.95, duration_ms[5m])`   |
| `max_over_time()`      | Pic                      | `max_over_time(duration_ms[5m])`              |

### **4. La Corrélation Métriques-Logs : L'Observabilité Complète**

**Scénario typique :**
1. **Métrique** : `rate(http_requests_total{status="500"}[5m])` → Pics d'erreurs 500
2. **Logs** : `{app="api"} |= "ERROR"` → Détails des erreurs
3. **Dashboard combiné** : Visualiser les deux simultanément
4. **Cause racine** : Corréler les logs d'erreur avec les métriques

### **5. L'Alerting Intelligent basé sur les Logs**

**Ne pas alerter sur chaque erreur, mais sur les patterns :**

```logql
# Alerte si plus de 10 erreurs en 5 minutes
sum(count_over_time({service="auth-api"} |= "ERROR"[5m])) > 10

# Alerte si le taux d'erreur dépasse 5% sur 10 minutes
(
  sum(count_over_time({service="auth-api"} |= "ERROR"[10m]))
  /
  sum(count_over_time({service="auth-api"}[10m]))
) * 100 > 5
```

---

## **🎯 BEST PRACTICES IDENTIFIÉES**

### **✅ Structuration des Logs**
- **Toujours en JSON** pour exploitation automatique
- **Champs obligatoires** : timestamp, level, service, action
- **Champs contextuels** : user_id, request_id, trace_id, duration_ms
- **Niveaux cohérents** : ERROR, WARN, INFO, DEBUG
- **Pas de DEBUG en production** (volume trop important)

### **⚠️ Configuration Promtail**
- **Labels limités** (max 5-6) pour performance
- **Parsing côté Promtail** plutôt que dans Loki
- **Filtrage en amont** pour réduire le volume
- **Pipeline stages** organisés logiquement

### **🔧 Requêtes LogQL**
- **Toujours commencer par les labels** `{service="api"}`
- **Parser JSON avant de filtrer** sur champs
- **Utiliser `count_over_time()` pour les tendances**
- **Quantiles pour les distributions** (pas moyennes seules)

### **📊 Dashboards Efficaces**
- **Volume de logs** : Détecter les anomalies
- **Taux d'erreur** : Santé du service
- **Latence** : Performance (95e, 99e percentile)
- **Logs en direct** : Debug immédiat
- **Alertes** : Pics d'erreurs, patterns suspects

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Les Logs Deviennent des Données**

Avec la structuration, les logs ne sont plus du texte. Ce sont des **données exploitables** :
- Des champs qu'on peut filtrer
- Des valeurs numériques qu'on peut agréger
- Des identifiants pour corréler

### **2. L'Agrégation Change la Perspective**

**Avant :** "Il y a beaucoup d'erreurs"
**Maintenant :**
- "Le taux d'erreur est passé de 1% à 15%"
- "La latence 95e percentile est à 2.3s"
- "L'utilisateur 12345 a eu 10 erreurs consécutives"

### **3. La Corrélation Métriques-Logs est le Saint Graal**

- **Métriques** : Détectent le problème ("quoi")
- **Logs** : Expliquent le problème ("pourquoi")
- **Traces** : Montrent le chemin ("où")

### **4. L'Alerting Intelligent Évite le Bruit**

**Ne pas alerter sur :**
- Une erreur isolée (peut être transitoire)
- Des erreurs connues (maintenance, déploiement)
- Des services non-critiques

**Alerter sur :**
- Pics d'erreurs (> seuil sur période)
- Patterns répétés (même erreur x fois)
- Dégradation progressive (latence qui augmente)

### **5. Les Logs Sont la Mémoire du Système**

Avec Loki, les logs survivent aux pods :
- Debug après incident
- Analyse de tendances sur longue période
- Audit et conformité

---

## **📈 PROGRESSION JOUR 87**

### **✅ ACQUIS TECHNIQUES :**
- **Application JSON logger** déployée
- **Parsing automatique** configuré dans Promtail
- **LogQL avancé** : agrégations, quantiles, taux
- **Dashboard complet** : 5 panels pertinents
- **Alerte sur pic d'erreurs** configurée
- **Corrélation métriques-logs** testée

### **🎯 CHANGEMENT MENTAL :**
> **Hier :** "J'ai des logs centralisés que je peux consulter"  
> **Aujourd'hui :** "J'**exploite** mes logs pour comprendre et anticiper"  
> **Résultat :** "Debug proactif, métriques métier, alertes intelligentes"

### **🔗 ARCHITECTURE D'EXPLOITATION :**

```
[ LOGS BRUTS ]
    │
    ▼
[ PROMTAIL PIPELINE ]
├── Découverte pods
├── Filtrage (DEBUG ignoré)
├── Parsing JSON → extraction champs
├── Calcul métriques temps réel
└── Envoi avec labels
    │
    ▼
[ LOKI STORAGE ]
    │
    ▼
[ GRAFANA ]
├── Explore : requêtes ad-hoc
├── Dashboard "Observabilité Applicative"
│   ├── Volume logs/service
│   ├── Taux d'erreur (avec seuils)
│   ├── Latence 95e percentile
│   ├── Activité utilisateur
│   └── Erreurs en direct
└── Alerting
    └── Pics d'erreurs > 10/5min
```

### **🚀 POUR DEMAIN (JOUR 88)**

Projet complet :
- 3 applications avec logs structurés différents
- Dashboard unifié métriques (Prometheus) + logs (Loki)
- Alertes intelligentes multi-seuils
- Tests de charge pour valider la scalabilité
- Documentation production-ready

---

## **💡 INSIGHTS FINAUX**

### **L'Observabilité est un Cercle Vertueux**

1. **Logs structurés** → Données exploitables
2. **Agrégations** → Métriques dérivées
3. **Dashboards** → Visibilité en temps réel
4. **Alertes** → Détection proactive
5. **Debug** → Résolution rapide
6. **Amélioration** → Moins d'erreurs → Retour à l'étape 1

### **La Valeur des Logs Augmente avec la Structuration**

Un log non-structuré a une valeur proche de zéro.
Un log structuré et corrélé a une valeur inestimable pour :
- **Le debug** : Temps de résolution /10
- **La performance** : Optimisations ciblées
- **Le métier** : Analytics utilisateur
- **La sécurité** : Détection d'intrusions

### **Préparation pour la Production**

Demain, nous consoliderons tout dans un projet complet :
- Applications réelles avec logs métier
- Dashboards prêts pour l'exploitation
- Alertes paramétrées pour la production
- Tests de charge pour valider la scalabilité

---

## **📊 CHECKLIST ACCOMPLIE**

- [ ] **Application JSON logger** déployée avec logs structurés
- [ ] **Configuration Promtail** mise à jour pour parsing JSON
- [ ] **Pipeline de traitement** configuré (regex, json, labels)
- [ ] **Requêtes LogQL avancées** testées (count, rate, quantile)
- [ ] **Dashboard complet** créé (5 panels)
- [ ] **Alerte sur pic d'erreurs** configurée et testée
- [ ] **Corrélation métriques-logs** vérifiée
- [ ] **Compréhension** des bonnes pratiques de structuration

---

**Les logs ne sont plus des fichiers :**  
**Ce sont des données exploitables en temps réel, corrélées, et alertées.** 📊

**📊 Progress: `Jour 87 / 100 ✅`**

**#Kubernetes #Loki #LogQL #StructuredLogging #Observability #Grafana #DevOps #SRE**
