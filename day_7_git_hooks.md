# **DAY 7 - GIT CONFLITS & HOOKS - DU CHAOS À L'AUTOMATISATION** ⚡

## **🎯 CONCEPTS CLÉS APPRIS**

### **⚔️ Résolution de Conflits Avancée**
- **Structure des conflits** → Marqueurs `<<<<<<<`, `=======`, `>>>>>>>`
- **Types de conflits** → Modifications concurrentes, fichiers supprimés/modifiés
- **Stratégies de résolution** → Manuelle, outils, fusion intelligente

### **🤖 Git Hooks - Automatisation**
- **Hooks clients** → Scripts locaux automatisés
- **Points d'intégration** → pre-commit, pre-push, commit-msg
- **Qualité automatisée** → Vérifications avant partage du code

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Gestion des Conflits**
| Commande              | FR                        | EN                                    | Usage                 |
|-----------------------|---------------------------|---------------------------------------|-----------------------|
| `git mergetool`       | Outil résolution conflits | **MERGE TOOL** - Conflict resolution  | `git mergetool`       |
| `git status`          | Voir fichiers en conflit  | **STATUS** - Show conflicts           | `git status`          |
| `git add .`           | Marquer résolu            | **ADD** - Mark resolved               | `git add .`           |
| `git commit`          | Finaliser résolution      | **COMMIT** - Finalize resolution      | `git commit`          |
| `git merge --abort`   | Annuler merge             | **MERGE ABORT** - Cancel merge        | `git merge --abort`   |
|-----------------------|---------------------------|---------------------------------------|-----------------------|

### **⚡ Git Hooks**
| Commande                | FR                          | EN                                | Usage                             |
|-------------------------|-----------------------------|-----------------------------------|-----------------------------------|
| `chmod +x .git/hooks/*` | Rendre hooks exécutables    | **CHMOD** - Make executable       | `chmod +x .git/hooks/*`           |
| `git config merge.tool` | Configurer outil merge      | **CONFIG MERGE TOOL** - Set tool  | `git config merge.tool vimdiff`   |
|-------------------------|-----------------------------|-----------------------------------|-----------------------------------|

### **🔍 Inspection**
| Commande | FR | EN | Usage |
|-------------------------------|-----------------------|--------------------------------|------------------------------|
| `git log --oneline --graph`   | Historique graphique  | **LOG graph** - Visual history | `git log --oneline --graph`  |
| `git diff --name-only`        | Fichiers modifiés     | **DIFF names** - Changed files | `git diff --name-only`       |
|-------------------------------|-----------------------|--------------------------------|------------------------------|

---

## **⚡ TYPES DE CONFLITS & SOLUTIONS**

### **🎯 Conflits de Modifications**
```bash
# Fichier en conflit :
<<<<<<< HEAD
Version Feature A
=======
Version Feature B
>>>>>>> feature-b

# Solutions :
# 1. Garder A    → "Version Feature A"
# 2. Garder B    → "Version Feature B"  
# 3. Fusionner   → "Version A et B combinées"
# 4. Rédiger nouveau → "Nouvelle version"
```

### **🔧 Outils de Résolution**
| Outil | Commande | Usage |
|---|---|---|
| **VimDiff** | `git config merge.tool vimdiff` | Interface en ligne de commande |
| **VSCode** | `git config merge.tool vscode` | Éditeur visuel |
| **KDiff3** | `git config merge.tool kdiff3` | Outil graphique dédié |

---

## **🤖 GIT HOOKS - AUTOMATISATION QUALITÉ**

### **🎯 Hooks Essentiels**
| Hook | Déclencheur | Usage |
|---|---|---|
| **pre-commit** | Avant commit | Vérifications code, syntaxe |
| **pre-push** | Avant push | Protection branches, tests |
| **commit-msg** | Validation message | Format conventionnel |
| **post-merge** | Après merge | Mise à jour dépendances |

### **💡 Exemples de Hooks Créés**

#### **Pre-commit : Vérifications Qualité**
```bash
#!/bin/bash
echo "🚀 Vérifications pre-commit..."

# Syntaxe shell
find . -name "*.sh" -exec bash -n {} \;

# Taille fichiers
if [ $(wc -l < "$file") -gt 1000 ]; then
    echo "❌ Fichier trop volumineux"
    exit 1
fi

# Code debug
if git diff --cached | grep -q "console.log\|TODO\|FIXME"; then
    echo "⚠️  Code debug détecté!"
fi
```

#### **Pre-push : Protection Branches**
```bash
#!/bin/bash
current_branch=$(git symbolic-ref --short HEAD)
protected_branches="main develop"

if [[ " $protected_branches " =~ " $current_branch " ]]; then
    echo "❌ Pushes directs interdits sur $current_branch"
    echo "Utilisez une Pull Request"
    exit 1
fi
```

#### **Commit-msg : Standardisation**
```bash
#!/bin/bash
commit_msg=$(cat "$1")

# Format conventionnel
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore): "; then
    echo "❌ Format: type: description"
    echo "Ex: feat: ajout authentification"
    exit 1
fi
```

---

## **🚀 EXERCICES RÉALISÉS**

### **1. Création et Résolution de Conflits**
```bash
# Conflit simulé
git merge feature-b
# → CONFLIT dans fichier.txt

# Structure analysée :
<<<<<<< HEAD
Version Feature A
=======
Version Feature B
>>>>>>> feature-b

# Résolution manuelle
echo "Version fusionnée A+B" > fichier.txt
git add . && git commit -m "Résolution conflit"
```

### **2. Outils de Résolution**
```bash
# Configuration outil
git config --global merge.tool vimdiff

# Résolution assistée
git mergetool
# → Interface visuelle de fusion
```

### **3. Hooks d'Automatisation**
```bash
# Création hooks
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/pre-push  
chmod +x .git/hooks/commit-msg

# Test des hooks
git add . && git commit -m "test"
# → Vérifications automatiques exécutées
```

---

## **🎯 MÉTHODOLOGIE PROFESSIONNELLE**

### **Processus de Résolution Conflits :**
```bash
1. git status                    # Identifier conflits
2. git mergetool                 # Résoudre avec outil
3. git add .                     # Marquer résolu
4. git commit                    # Finaliser
5. git log --oneline --graph     # Vérifier historique
```

### **Stratégie Hooks :**
```bash
# Développement local
pre-commit    → Qualité code
commit-msg    → Standards messages

# Collaboration équipe  
pre-push      → Protection branches
post-merge    → Synchronisation
```

### **Bonnes Pratiques :**
- ✅ **Toujours** tester hooks sur branche de test
- ✅ **Toujours** messages de commit conventionnels
- ✅ **Toujours** protéger main/develop
- ✅ **Toujours** utiliser outils pour conflits complexes

---

## **📈 PROGRESSION DAY 7**

**✅ Compétences Acquises :**
- Résolution méthodique de conflits Git complexes
- Maîtrise des outils de merge (vimdiff, VSCode)
- Création et configuration de Git Hooks personnalisés
- Automatisation de la qualité du code et des processus
- Protection des branches critiques via hooks

**🎯 Mentalité DevOps :**
> Je ne subis plus les conflits Git  
> Je les résous méthodiquement et les préviens par l'automatisation

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 7 / 100 ✅`**

**#Git #DevOps #Conflits #GitHooks #Automatisation #QualitéCode #VersionControl**

---
