# **JOUR 80 : VARIABLES, OUTPUTS ET RESSOURCES MULTIPLES**

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏷️ Variables Terraform**
- **Typage fort** : string, number, bool, list, map, object
- **Validation** : Contrôle des valeurs acceptées
- **Sensibilité** : Masquage des secrets dans les logs
- **Valeurs par défaut** : Paramétrage optionnel
- **Descriptions** : Documentation intégrée

### **📤 Outputs (Sorties)**
- **Récupération d'informations** après création
- **Valeurs sensibles** masquées dans les logs
- **Structures complexes** : listes, maps, objets
- **Chaînage** entre configurations

### **🔗 Dépendances**
- **Implicites** : Détectées par références directes
- **Explicites** : `depends_on` pour relations cachées
- **Count** : Création multiple de ressources
- **Conditions** : Ressources activées selon variables

### **📊 Data Sources**
- **Lecture de l'existant** sans modification
- **Informations dynamiques** (dernière image, IP)
- **Réutilisation** de ressources déjà créées

---

## **📊 Hiérarchie des Variables**

### **Ordre de précédence (du moins prioritaire au plus prioritaire)**

```
1. Valeur par défaut (dans variables.tf)
   └── default = "dev"

2. Fichier terraform.tfvars
   └── environment = "staging"

3. Fichiers *.auto.tfvars
   └── dev.auto.tfvars, prod.auto.tfvars

4. Variables d'environnement
   └── export TF_VAR_environment="prod"

5. Ligne de commande
   └── terraform apply -var="environment=production"
```

### **Types de variables**

| Type              | Description               | Exemple d'utilisation                 |
|-------------------|---------------------------|---------------------------------------|
| **string**        | Chaîne de caractères      | Nom de projet, environnement          |
| **number**        | Nombre entier ou décimal  | Nombre d'instances, ports             |
| **bool**          | Booléen (true/false)      | Activer/désactiver une feature        |
| **list(string)**  | Liste ordonnée            | Liste de régions, d'IPs               |
| **map(string)**   | Dictionnaire clé/valeur   | Tags, labels, configurations          |
| **object({})**    | Structure complexe        | Configuration complète d'un service   |
| **set(type)**     | Ensemble non ordonné      | Identifiants uniques                  |

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **1. La Paramétrisation Change Tout**

**Problème résolu :**
- Avant : Code dupliqué pour dev/staging/prod
- Après : **Une seule configuration, des variables**

**Impact :**
- ✅ **DRY** (Don't Repeat Yourself)
- ✅ **Maintenance simplifiée** (un seul endroit à modifier)
- ✅ **Cohérence** entre environnements
- ✅ **Documentation automatique** via les descriptions

**Exemple concret :**
```hcl
# AVANT (code dupliqué)
resource "aws_instance" "web_dev" {
  instance_type = "t2.micro"
  tags = { Environment = "dev" }
}

resource "aws_instance" "web_prod" {
  instance_type = "t2.large"
  tags = { Environment = "prod" }
}

# APRÈS (une seule définition)
resource "aws_instance" "web" {
  count         = var.environment == "prod" ? 3 : 1
  instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
  tags = { Environment = var.environment }
}
```

### **2. La Validation des Variables : Auto-défense**

**Pourquoi valider ?**
- Éviter les erreurs de configuration
- Garantir que les valeurs sont acceptables
- Documenter les contraintes

**Exemples de validations :**
```hcl
variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "L'environnement doit être dev, staging ou prod."
  }
}

variable "port" {
  type = number
  
  validation {
    condition     = var.port > 0 && var.port < 65536
    error_message = "Le port doit être entre 1 et 65535."
  }
}
```

**Résultat :** Le plan échoue avec un message clair plutôt qu'une erreur obscure.

### **3. La Sensibilité des Variables : Sécurité Intégrée**

**Problème :** Les secrets (mots de passe, tokens) ne doivent pas apparaître dans les logs.

**Solution :** `sensitive = true`

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}

output "password" {
  value     = var.db_password
  sensitive = true  # Ne s'affiche pas dans les logs
}
```

**Ce que ça change :**
- ❌ Avant : `terraform output` affichait le secret
- ✅ Après : `terraform output` affiche `<sensitive>`
- ✅ On peut toujours l'obtenir explicitement avec `terraform output -json`

### **4. Count : La Multiplicité Simplifiée**

**Problème :** Créer plusieurs ressources identiques.

**Solution :** `count`

```hcl
resource "local_file" "instance" {
  count = 3  # Crée 3 fichiers
  
  filename = "file-${count.index + 1}.txt"
  content  = "Content for instance ${count.index + 1}"
}
```

**Accès aux instances :**
- `local_file.instance[0]` : Première instance
- `local_file.instance[*].filename` : Tous les noms de fichiers
- `count.index` : Index dans la boucle (commence à 0)

### **5. Conditions : L'Infrastructure Adaptative**

**Problème :** Activer/désactiver des ressources selon le contexte.

**Solution :** Conditions avec `count`

```hcl
# Ressource créée SEULEMENT en dev
resource "local_file" "debug" {
  count = var.environment == "dev" ? 1 : 0
  
  filename = "debug.log"
  content  = "Debug information"
}
```

**Accès conditionnel :**
```hcl
output "debug_file" {
  value = var.environment == "dev" ? local_file.debug[0].filename : null
}
```

### **6. Outputs : La Mémoire du Déploiement**

**Pourquoi les outputs sont cruciaux :**
- Ils **documentent** ce qui a été créé
- Ils permettent de **chaîner** des configurations
- Ils servent d'**interface** entre modules

**Types d'outputs :**
```hcl
# Simple
output "ip" {
  value = docker_container.nginx.ip_address
}

# Liste
output "all_ips" {
  value = docker_container.nginx[*].ip_address
}

# Objet structuré
output "summary" {
  value = {
    project   = var.project_name
    instances = var.container_count
    timestamp = timestamp()
  }
}

# Sensible
output "password" {
  value     = random_password.db.result
  sensitive = true
}
```

---

## **🎯 BEST PRACTICES IDENTIFIÉES**

### **✅ Organisation des Fichiers**

```
projet-terraform/
├── variables.tf        # Toutes les variables avec descriptions
├── terraform.tfvars     # Valeurs par défaut (ne pas commiter si secrets)
├── dev.tfvars          # Surcharges pour dev
├── staging.tfvars      # Surcharges pour staging
├── prod.tfvars         # Surcharges pour prod
├── main.tf             # Ressources principales
├── outputs.tf          # Tous les outputs
└── providers.tf        # Configuration des providers
```

### **⚠️ Bonnes Pratiques des Variables**

- **Toujours** documenter avec `description`
- **Toujours** typer les variables
- **Valider** les valeurs critiques
- **Marquer comme sensitive** les secrets
- **Nommer clairement** : `instance_count` pas `count`

### **🔧 Bonnes Pratiques des Outputs**

- **Toujours** décrire avec `description`
- **Marquer comme sensitive** les outputs sensibles
- **Structurer** les outputs complexes en objets
- **Nommer** de manière explicite
- **Documenter** ce que l'output représente

### **📊 Gestion des Environnements**

```bash
# Développement
terraform apply -var-file="dev.tfvars"

# Staging
terraform apply -var-file="staging.tfvars"

# Production
terraform apply -var-file="prod.tfvars"
```

---

## **🔍 LEÇONS IMPORTANTES**

### **1. Les Variables Transforment le Code en Template**

**Révélation :** Un code Terraform sans variables est comme une fonction sans paramètres. Avec des variables, il devient un **template réutilisable**.

**Avant :** `main.tf` spécifique à un environnement
**Après :** `main.tf` générique, `dev.tfvars` spécifique

### **2. La Validation est une Documentation Exécutable**

**Ce que les validations apportent :**
- ✅ **Contrat** sur les valeurs acceptables
- ✅ **Messages clairs** plutôt qu'erreurs cryptiques
- ✅ **Détection précoce** des erreurs de configuration
- ✅ **Documentation** des contraintes

### **3. Le Count Révèle la Puissance de l'Abstraction**

**Avant count :** Copier-coller de blocs de ressources
**Après count :** Une boucle qui s'adapte

**Gain :** 
- Réduction de la duplication
- Cohérence garantie
- Scalabilité sans effort

### **4. Les Outputs sont l'API de votre Infrastructure**

**Ils permettent :**
- **Aux humains** : De voir ce qui a été créé
- **Aux scripts** : D'automatiser des actions
- **Aux autres configurations** : De s'y connecter

### **5. La Sensibilité est Culturelle, pas Technique**

**Ce que `sensitive` change :**
- **Culture de sécurité** : On pense à protéger les secrets
- **Habitude** : Vérifier ce qui est exposé
- **Confiance** : On peut partager les logs sans risque

---

## **📈 PROGRESSION JOUR 80**

### **✅ ACQUIS TECHNIQUES :**
- **Variables typées** avec validations
- **Variables sensibles** et outputs protégés
- **Ressources multiples** avec `count`
- **Ressources conditionnelles** avec opérateur ternaire
- **Outputs structurés** (simples, listes, objets)
- **Fichiers tfvars** pour différents environnements
- **Providers locaux** (random, local) sans cloud

### **🎯 CHANGEMENT MENTAL :**
> **Hier :** "Je crée des ressources spécifiques"  
> **Aujourd'hui :** "Je crée des **templates paramétrables**"  
> **Résultat :** "Une configuration pour tous les environnements"

### **🔗 NOTRE ARCHITECTURE PARAMÉTRABLE :**

```
[ UTILISATEUR ]
    │
    ▼
┌─────────────────────────────────────────────────────┐
│               INTERFACE (variables.tf)               │
├─────────────────────────────────────────────────────┤
│  project_name     = "demo"                          │
│  environment      = "dev" | "staging" | "prod"     │
│  container_count  = 1..5                           │
│  enable_nginx     = true | false                   │
│  labels           = { managed_by = "terraform" }    │
└─────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────┐
│               MOTEUR (main.tf)                       │
├─────────────────────────────────────────────────────┤
│  if environment == "prod" → resources = max         │
│  if enable_nginx → docker_container                  │
│  count = var.container_count → ressources multiples │
│  merge(var.labels, {env = var.environment})         │
└─────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────┐
│               SORTIES (outputs.tf)                   │
├─────────────────────────────────────────────────────┤
│  project_name                                      │
│  random_prefix                                     │
│  db_password (sensitive)                           │
│  instances (list)                                  │
│  nginx_urls (if enabled)                           │
│  summary (objet complet)                           │
└─────────────────────────────────────────────────────┘
    │
    ▼
[ RÉSULTAT : INFRASTRUCTURE ADAPTÉE ]
```

### **🚀 POUR DEMAIN (JOUR 81) :**
- **Modules** : Encapsuler la logique réutilisable
- **Registre Terraform** : Modules publics
- **Composition** : Modules imbriqués
- **Versionnement** : Gérer les versions des modules
- **Réutilisabilité** maximale

---

## **💡 INSIGHTS FINAUX**

### **La Paramétrisation est un Investissement**

**Coût immédiat :** Écrire des variables et des validations prend du temps.
**Bénéfice long terme :** Chaque nouvel environnement est instantané.

**Rentabilité :**
- 1 environnement : ROI négatif
- 2 environnements : Break-even
- 3+ environnements : ROI positif exponentiel

### **L'Infrastructure devient un Produit**

**Avec des variables et outputs, l'infrastructure devient :**
- **Configurable** : Adaptable sans modifier le code
- **Interrogeable** : On peut demander ce qui a été créé
- **Documentée** : Les variables expliquent les options
- **Fiable** : Les validations préviennent les erreurs

### **La Prochaine Étape : Les Modules**

Si les variables rendent le code paramétrable, les modules le rendent **réutilisable**. Demain, nous encapsulerons des patterns complets pour les partager entre projets.

---

## **📊 CHECKLIST ACCOMPLIE**

- [ ] **Variables typées** (string, number, bool, list, map)
- [ ] **Descriptions** sur toutes les variables
- [ ] **Validations** sur les valeurs critiques
- [ ] **Variables sensibles** marquées
- [ ] **Fichiers tfvars** pour différents environnements
- [ ] **Ressources avec count** pour la multiplicité
- [ ] **Ressources conditionnelles** (count avec ternaire)
- [ ] **Outputs variés** (simples, listes, objets)
- [ ] **Outputs sensibles** protégés
- [ ] **Data sources** pour lire l'existant
- [ ] **Providers locaux** (random, local) sans cloud
- [ ] **Plan et apply** testés avec différentes valeurs
- [ ] **Destroy** systématique après tests

---

**L'infrastructure devient paramétrable et adaptable :**  
**Une configuration, tous les environnements.** 🎛️

**📊 Progress: `Jour 80 / 100 ✅`**

**#Terraform #IaC #Variables #DevOps #InfrastructureAsCode #Automation #DevOps**
