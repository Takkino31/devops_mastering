# **DAY 6 - GIT INTERNALS & COMMANDES AVANCÉES** 🔧

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Interne de Git**
- **Système content-adressable** → Tout est stocké via hash SHA-1
- **Base de données d'objets** → Plus qu'un système de fichiers
- **Références** → Pointeurs vers les commits

### **📦 Les 4 Objets Git Fondamentaux**
- **BLOB** → Contenu des fichiers
- **TREE** → Structure des répertoires  
- **COMMIT** → Snapshots + métadonnées
- **TAG** → Références permanentes

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔍 Exploration Git Internals**
| Commande                      | FR                | EN                                | Usage                         |
|-------------------------------|-------------------|-----------------------------------|-------------------------------|
| `git cat-file -p HASH`        | Afficher objet    | **CATenate FILE** - Show object   | `git cat-file -p a1b2c3`      |
| `git hash-object fichier`     | Hash d'un fichier | **HASH OBJECT** - File hash       | `git hash-object monfichier`  |
| `find .git/objects -type f`   | Lister objets     | **FIND objects** - List objects   | `find .git/objects -type f`   |
|-------------------------------|-------------------|-----------------------------------|-------------------------------|

### **🔄 Commandes Avancées Workflow**
| Commande                  | FR                    | EN                                        | Usage                     |
|---------------------------|-----------------------|-------------------------------------------|---------------------------|
| `git rebase -i HEAD~3`    | Rebase interactif     | **Interactive REBASE** - Clean history    | `git rebase -i HEAD~3`    |
| `git cherry-pick HASH`    | Récupérer commit      | **CHERRY-PICK** - Apply specific commit   | `git cherry-pick a1b2c3`  |
| `git stash push -m "msg"` | Stash avec message    | **STASH with message** - Save work        | `git stash push -m "WIP"` |
| `git stash list`          | Lister stashes        | **STASH LIST** - Show stashes             | `git stash list`          |
| `git stash pop`           | Appliquer stash       | **STASH POP** - Apply stash               | `git stash pop`           |
|---------------------------|-----------------------|-------------------------------------------|---------------------------|

### **📊 Visualisation**
| Commande | FR | EN | Usage |
|-----------------------|-----------------------|---------------------------------------|-----------------------|
| `git log --oneline`   | Historique compact    | **LOG one line** - Compact history    | `git log --oneline`   |
| `git log --graph`     | Historique graphique  | **LOG graph** - Visual history        | `git log --graph`     |
|-----------------------|-----------------------|---------------------------------------|-----------------------|

---

## **⚡ OPÉRATEURS & STRATÉGIES**

### **🌊 Stratégies de Branching**
| Stratégie | Usage | Structure |
|-------------------|-----------------------|--------------------------------------|
| **GitFlow**       | Entreprises, releases | `main` ← `develop` ← `feature/`       |
| **GitHub Flow**   | Startups, CI/CD       | `main` ← `feature/`                  |
| **Trunk-Based**   | Équipes agiles        | `main` seulement                     |
|-------------------|-----------------------|--------------------------------------|

### **🔧 Commandes Rebase Interactif**
| Commande  | Signification                 | Effet                             |
|-----------|-------------------------------|-----------------------------------|
| `pick`    | Garder le commit              | Commit inchangé                   |
| `reword`  | Modifier message              | Change commit message             |
| `edit`    | Modifier le commit            | Ouvre l'éditeur                   |
| `squash`  | Fusionner avec précédent      | Combine les commits               |
| `fixup`   | Fusionner (ignore message)    | Combine, garde message précédent  |
|-----------|-------------------------------|-----------------------------------|

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Le Mystère des Objets Git :**
```bash
# Chaque fichier = un BLOB
echo "content" > file.txt
git hash-object file.txt          # → 8ab686...

# Chaque dossier = un TREE
git cat-file -p TREE_HASH         # → montre structure

# Chaque commit = SNAPSHOT complet
git cat-file -p COMMIT_HASH       # → auteur, date, tree, parent
```

### **Rebase Interactif en Action :**
```bash
# Avant : historique désordonné
a1b2c3 Add function
b2c3d4 Fix typo  
c3d4e5 Update function

# Après git rebase -i HEAD~3 :
d4e5f6 Implement complete function
```

### **Cherry-pick Stratégique :**
```bash
# Récupérer un correctif sans la feature
git checkout main
git cherry-pick security-fix-hash
# → Seul le correctif est appliqué
```

### **Stash Contextuel :**
```bash
git stash push -m "WIP: authentication"
git stash list
# → stash@{0}: WIP: authentication
git stash pop stash@{0}
```

---

## **🚀 EXERCICES RÉALISÉS**

### **1. Exploration Objets Git**
```bash
# Découverte de l'architecture interne
find .git/objects -type f
git cat-file -p COMMIT_HASH
git cat-file -p TREE_HASH  
git cat-file -p BLOB_HASH
```

### **2. Nettoyage Historique**
```bash
# Rebase interactif pour fusionner commits
git rebase -i HEAD~3
# → pick, squash, reword
```

### **3. Récupération Ciblée**
```bash
# Cherry-pick d'un correctif critique
git cherry-pick security-fix
# → Application sélective
```

### **4. Sauvegarde Temporaire**
```bash
# Stash avec contexte de travail
git stash push -m "Feature en cours"
git stash list
git stash pop
```

---

## **🎯 MÉTHODOLOGIE GIT AVANCÉE**

### **Approche Rebase vs Merge :**
```bash
# REBASE → Historique linéaire propre
git fetch origin
git rebase origin/main

# MERGE → Historique de collaboration
git fetch origin  
git merge origin/main
```

### **Workflow Feature Branch :**
```bash
1. git checkout -b feature/nouvelle-fonction
2. git commit -m "Implementation"
3. git fetch origin && git rebase origin/main
4. git push origin feature/nouvelle-fonction
5. Pull Request + Review
6. Merge sur main
```

### **Bonnes Pratiques :**
- ✅ **Toujours** rebase avant de push
- ✅ **Toujours** tester après cherry-pick
- ✅ **Toujours** message clair avec stash
- ✅ **Toujours** comprendre l'historique avant rebase

---

## **📈 PROGRESSION DAY 6**

**✅ Compétences Acquises :**
- Compréhension profonde du fonctionnement interne de Git
- Maîtrise du rebase interactif pour historiques propres
- Utilisation stratégique de cherry-pick
- Gestion de contexte avec stash avancé
- Choix éclairé des stratégies de branching

**🎯 Mentalité DevOps :**
> Je ne pousse plus juste du code  
> Je maintiens un historique propre et stratégique

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 6 / 100 ✅`**

**#Git #DevOps #VersionControl #GitInternals #Rebase #CherryPick**

---
