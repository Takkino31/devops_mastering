# **DAY 4 - BASH SCRIPTING : DE ZÉRO À HÉROS** ⚡

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture d'un Script Bash**
- **Shebang** = `#!/bin/bash` → Spécifie l'interpréteur
- **Commentaires** = `# texte` → Documentation du code
- **Commandes** = Instructions exécutées séquentiellement

### **📦 Les 4 Piliers du Scripting**
- **Variables** → Stockage de données
- **Conditions** → Prises de décision  
- **Boucles** → Automatisation de tâches
- **Arguments** → Données d'entrée

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Création & Exécution**
| Commande | FR | EN | Usage |
|-----------------------|---------------------------|-----------------------------------------------|---------------------------|
| `chmod +x script.sh`  | Rendre exécutable         | **Change Mode eXecutable** - Make executable  | `chmod +x mon_script.sh`  |
| `./script.sh`         | Exécuter script           | **Execute Script** - Run script               | `./mon_script.sh`         |
| `nano fichier.sh`     | Éditeur texte             | **Nano Editor** - Text editor                 | `nano script.sh`          |
|-----------------------|---------------------------|-----------------------------------------------|---------------------------|

### **📝 Variables & Interaction**
| Commande | FR | EN | Usage |
|-----------------------|-----------------------|-------------------------------------------|-----------------------|
| `VARIABLE="valeur"`   | Déclarer variable     | **Variable Assignment** - Set variable    | `NOM="Yaya"`          |
| `echo $VARIABLE`      | Afficher variable     | **ECHO variable** - Print variable        | `echo $NOM`           |
| `read -p "text" var`  | Lire entrée           | **READ with Prompt** - User input         | `read -p "Nom?" NOM`  |
| `$(commande)`         | Exécuter commande     | **Command Substitution** - Run command    | `DATE=$(date)`        |
|-----------------------|-----------------------|-------------------------------------------|-----------------------|

### **🎯 Conditions & Tests**
| Commande                  | FR                    | EN                                            | Usage                         |
|---------------------------|-----------------------|-----------------------------------------------|-------------------------------|
| `if [ condition ]; then`  | Si condition          | **IF condition** - Conditional execution      | `if [ $AGE -gt 18 ]; then`    |
| `[ -f fichier ]`          | Fichier existe        | **File exists test** - Check file             | `[ -f "script.sh" ]`          |
| `[ -d dossier ]`          | Dossier existe        | **Directory exists test** - Check directory   | `[ -d "/tmp" ]`               |
| `[ -z variable ]`         | Variable vide         | **String is empty** - Check empty variable    | `[ -z "$NOM" ]`               |
|---------------------------|-----------------------|-----------------------------------------------|-------------------------------|

### **🔄 Boucles & Automation**
| Commande | FR | EN | Usage |
|---------------------------|-----------------------|-----------------------------------------------|-------------------------------|
| `for i in 1 2 3; do`      | Boucle for            | **FOR loop** - Iterate over list              | `for i in 1 2 3; do`          |
| `while [ condition ]; do` | Boucle while          | **WHILE loop** - Loop while true              | `while [ $i -lt 5 ]; do`      |
| `$((calcul))`             | Calcul arithmétique   | **Arithmetic expansion** - Math calculation   | `COMPTEUR=$((COMPTEUR+1))`    |
|---------------------------|-----------------------|-----------------------------------------------|-------------------------------|

---

## **⚡ OPÉRATEURS ESSENTIELS**

### **🔍 Tests de Fichiers**
| Opérateur        | Signification      | Exemple               |
|------------------|--------------------|-----------------------|
| `[ -f fichier ]` | Fichier existe     | `[ -f "script.sh" ]`  |
| `[ -d dossier ]` | Dossier existe     | `[ -d "/tmp" ]`       |
| `[ -r fichier ]` | Fichier lisible    | `[ -r "data.txt" ]`   |

### **🔢 Comparaisons Numériques**
| Opérateur       | Signification                   | Exemple               |
|-----------------|---------------------------------|-----------------------|
| `[ $a -eq $b ]` | Égal à (equal)                  | `[ $AGE -eq 18 ]`     |
| `[ $a -ne $b ]` | Différent de (not equal)        | `[ $NOTE -ne 0 ]`     |
| `[ $a -gt $b ]` | Plus grand que (greater than)   | `[ $AGE -gt 18 ]`     |
| `[ $a -lt $b ]` | Plus petit que (less than)      | `[ $SCORE -lt 100 ]`  |
|-----------------|---------------------------------|-----------------------|

### **📝 Comparaisons Textuelles**
| Opérateur             | Signification                 | Exemple |
|-----------------------|-------------------------------|---------------------------|
| `[ "$a" = "$b" ]`     | Égal à                        | `[ "$NOM" = "Yaya" ]`     |
| `[ "$a" != "$b" ]`    | Différent de                  | `[ "$STATUS" != "OK" ]`   |
| `[ -z "$var" ]`       | Variable vide (zero length)   | `[ -z "$NOM" ]`           |
| `[ -n "$var" ]`       | Variable non vide             | `[ -n "$EMAIL" ]`         |
|-----------------------|-------------------------------|---------------------------|

### **🧮 Opérateurs Arithmétiques**
| Type             | Opérateur  | Signification         | Exemple           |
|------------------|------------|-----------------------|-------------------|
| **Test**         | `[ ]`      | Test conditionnel     | `[ -f fichier ]`  |
| **Arithmétique** | `(( ))`    | Calculs mathématiques | `(( a + b ))`     |
| **Substitution** | `$( )`     | Exécution de commande | `$(date)`         |
| **Arithmétique** | `$(( ))`   | Calcul + substitution | `result=$((a+b))` |
|------------------|------------|-----------------------|-------------------|

--- 

## **💡 DÉCOUVERTES IMPORTANTES**

### **Le Mystère des Quotes :**
```bash
echo "Bonjour $NOM"        # ✅ Interprète la variable
echo 'Bonjour $NOM'        # ❌ Affiche $NOM littéralement
echo "Date: $(date)"       # ✅ Exécute la commande
```

### **Structure Conditionnelle :**
```bash
if [ "$NOM" = "Yaya" ]; then
    echo "Accès admin ✅"
elif [ -z "$NOM" ]; then
    echo "Nom manquant ❌"
else
    echo "Utilisateur standard 👋"
fi
```

### **Boucle Automatique :**
```bash
# Lister tous les scripts
for script in *.sh; do
    echo "📜 $script"
done

# Décompte avec while
compteur=3
while [ $compteur -gt 0 ]; do
    echo "⏳ $compteur..."
    compteur=$((compteur - 1))
done
```

### **Comparaisons Numériques :**
```bash
AGE=25
if [ $AGE -gt 18 ]; then
    echo "✅ Majeur"
else
    echo "❌ Mineur"
fi
```

### **Calculs Arithmétiques :**
```bash
# Addition
SOMME=$((5 + 3))                    # → 8

# Avec variables
A=10
B=5
RESULTAT=$((A * B))                 # → 50

# Incrémentation
((COMPTEUR++))                      # → COMPTEUR + 1
```

### **Combinaison de Tests :**
```bash
if [ -f "config.txt" ] && [ -r "config.txt" ]; then
    echo "✅ Fichier existe et est lisible"
fi

if [ $AGE -gt 18 ] || [ $AUTORISATION = "oui" ]; then
    echo "✅ Accès autorisé"
fi
```

---

## **🚀 SCRIPTS CRÉÉS AUJOURD'HUI**

### **1. Premier Script "Hello World"**
```bash
#!/bin/bash
echo "=== MON PREMIER SCRIPT ==="
echo "🚀 Bonjour le monde !"
echo "👤 Je suis : $(whoami)"
echo "📅 Date : $(date)"
```

### **2. Script Variables & Quotes**
```bash
#!/bin/bash
PRENOM="Yaya"
echo "Double quotes: $PRENOM"    # → Yaya
echo 'Single quotes: $PRENOM'    # → $PRENOM
```

### **3. Script Conditions**
```bash
#!/bin/bash
read -p "Ton prénom ? " NOM
if [ "$NOM" = "Yaya" ]; then
    echo "👑 Bonjour chef !"
fi
```

### **4. Script Boucles**
```bash
#!/bin/bash
for i in 1 2 3; do
    echo "Numéro: $i"
done
```

### **5. Script avec Opérateurs :**
```bash
#!/bin/bash
read -p "Quel est votre âge ? " AGE

if [ $AGE -lt 18 ]; then
    echo "❌ Accès interdit - Mineur"
elif [ $AGE -ge 18 ] && [ $AGE -lt 65 ]; then
    echo "✅ Accès autorisé - Adulte"
else
    echo "✅ Accès senior"
fi

# Calcul moyenne
NOTE1=15
NOTE2=18
MOYENNE=$(( (NOTE1 + NOTE2) / 2 ))
echo "Moyenne: $MOYENNE"
```

---

## **🎯 MÉTHODOLOGIE DE SCRIPTING**

### **Approche Systématique :**
```bash
1. #!/bin/bash                  # Shebang obligatoire
2. # Commentaires               # Documentation
3. Déclaration variables        # NOM="valeur"
4. Logique métier              # Conditions, boucles
5. Messages utilisateur        # echo "Résultat"
```

### **Bonnes Pratiques :**
- ✅ **Toujours** mettre le shebang `#!/bin/bash`
- ✅ **Toujours** commenter son code
- ✅ **Toujours** tester sur machine de test d'abord
- ✅ **Toujours** utiliser `chmod +x` avant exécution

---

## **📈 PROGRESSION DAY 4**

**✅ Compétences Acquises :**
- Créer et exécuter des scripts bash
- Utiliser variables et différents types de quotes
- Implémenter des conditions if/elif/else avec opérateurs
- Automatiser avec des boucles for/while
- Effectuer des calculs arithmétiques
- Interagir avec l'utilisateur

**🎯 Mentalité DevOps :**
> Je ne répète plus les commandes manuellement  
> Je les automatise dans des scripts réutilisables

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 4 / 100 ✅`**

**#Linux #Bash #Scripting #DevOps #Automation #Opérateurs #Apprentissage**

---


