# **DAY 5 - BASH SCRIPTING : SCRIPTS DE PRODUCTION** 🚀

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture des Scripts Production**
- **Scripts modulaires** → Séparation des responsabilités
- **Gestion d'erreurs** → Robustesse en environnement réel
- **Logging et rapports** → Traçabilité des actions
- **Arguments dynamiques** → Flexibilité d'utilisation

### **📦 Les 4 Types de Scripts DevOps**
- **Monitoring** → Santé du système
- **Sauvegarde** → Protection des données
- **Analyse** → Investigation des logs
- **Validation** → Vérifications pré-déploiement

---

## **🛠️ COMMANDES SYSTÈME ESSENTIELLES**

### **🔍 Monitoring Système**
| Commande              | FR                        | EN                                                | Usage                             |
|-----------------------|---------------------------|---------------------------------------------------|-----------------------------------|
| `free -m`             | Mémoire en MB             | **FREE Memory** - Show memory usage               | `free -m \| awk 'NR==2{print $4}'`|
| `df /`                | Usage disque racine       | **Disk Free** - Show disk space                   | `df / \| awk 'NR==2{print $5}'`   |
| `systemctl status`    | Statut services           | **SYSTEM ConTroL** - Service status               | `systemctl is-active nginx`       |
| `ss -tuln`            | Ports écoutants           | **Socket Statistics** - Listening ports           | `ss -tuln \| grep :80`            |
|-----------------------|---------------------------|---------------------------------------------------|-----------------------------------|

### **💾 Gestion Fichiers & Archives**
| Commande              | FR                            | EN                                                            | Usage                          |
|-----------------------|-------------------------------|---------------------------------------------------------------|--------------------------------|
| `tar -czf`            | Créer archive compressée      | **Tape ARchive** - Create compressed archive                  | `tar -czf backup.tar.gz /data` |
| `ls -t`               | Trier par date                | **LiSt Time sorted** - Sort by time                           | `ls -t *.tar.gz`               |
| `tail -n +4`          | Supprimer premières lignes    | **TAIL from line** - Skip first lines                         | `ls -t \| tail -n +4`          |
| `grep -i`             | Recherche insensible casse    | **Grep Case Insensitive** - Case insensitive search           | `grep -i "error" file.log`     |
|-----------------------|-------------------------------|---------------------------------------------------------------|--------------------------------|

### **📊 Analyse Logs**
| Commande              | FR                        | EN                                                | Usage                             |
|-----------------------|---------------------------|---------------------------------------------------|-----------------------------------|
| `wc -l`               | Compter lignes            | **Word Count Lines** - Count lines                | `grep "error" file.log \| wc -l`  |
| `head -5`             | Premières 5 lignes        | **HEAD first lines** - First lines                | `grep "error" file.log \| head -5`|
| `tee -a`              | Afficher et écrire        | **TEE Append** - Display and write                | `echo "test" \| tee -a log.txt`   |
| `command -v`          | Vérifier commande         | **COMMAND exists** - Check command exists         | `command -v docker`               |
|-----------------------|---------------------------|---------------------------------------------------|-----------------------------------|

---

## **⚡ NOUVEAUX OPÉRATEURS AVANCÉS**

### **🔢 Gestion Tableaux**
| Opérateur             | Signification | Exemple |
|-----------------------|-----------------------------------|---------------------------------------------------|
| `TABLEAU=("a" "b")`   | Déclaration tableau | `SERVICES=("ssh" "nginx" "docker")` |
| `${TABLEAU[@]}`       | Tous les éléments | `for service in "${SERVICES[@]}"; do` |
| `TABLEAU+=("c")`      | Ajout élément | `ECHECS+=("docker_manquant")` |
| `${#TABLEAU[@]}`      | Nombre d'éléments | `if [ ${#ECHECS[@]} -eq 0 ]; then` |
|-----------------------|-----------------------------------|---------------------------------------------------|

### **🎯 Gestion Codes Sortie**
| Opérateur             | Signification                     | Exemple |
|-----------------------|-----------------------------------|---------------------------------------------------|
| `$?`                  | Code retour dernière commande     | `if [ $? -eq 0 ]; then echo "✅ Succès"; fi`      |
| `2>/dev/null`         | Rediriger erreurs                 | `command 2>/dev/null`                             |
| `exit 0`              | Sortie succès                     | `exit 0`                                          |
| `exit 1`              | Sortie erreur                     | `exit 1`                                          |
|-----------------------|-----------------------------------|---------------------------------------------------|

### **📝 Arguments Dynamiques**
| Opérateur             | Signification                     | Exemple                                           |
|-----------------------|-----------------------------------|---------------------------------------------------|
| `$1, $2, $3`          | Arguments positionnels            | `SOURCE=$1`                                       |
| `${1:-default}`       | Valeur par défaut                 | `FICHIER="${1:-/var/log/syslog}"`                 |
| `$#`                  | Nombre d'arguments                | `if [ $# -eq 0 ]; then`                           |
| `-z "$1"`             | Argument vide                     | `if [ -z "$1" ]; then`                            |
|-----------------------|-----------------------------------|---------------------------------------------------|

---

## **💡 SCRIPTS DE PRODUCTION CRÉÉS**

### **🩺 HEALTH CHECK SYSTEM**
```bash
#!/bin/bash
echo "🩺 === HEALTH CHECK SYSTÈME ==="

MEMOIRE_LIBRE=$(free -m | awk 'NR==2{print $4}')
USAGE_DISQUE=$(df / | awk 'NR==2{print $5}' | sed 's/%//')

SERVICES=("ssh" "nginx" "docker")
for service in "${SERVICES[@]}"; do
    systemctl is-active --quiet $service && echo "✅ $service" || echo "❌ $service"
done
```

### **💾 BACKUP AUTOMATISÉ AVEC ROTATION**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

# Créer backup
tar -czf "$BACKUP_FILE" "$SOURCE"

# Rotation - garder 3 derniers
ls -t backup_*.tar.gz | tail -n +4 | xargs rm -f
```

### **📊 LOG PARSER AVEC ALERTES**
```bash
#!/bin/bash
ERREURS=$(grep -i "error" "$FICHIER_LOG" | wc -l)
WARNINGS=$(grep -i "warning" "$FICHIER_LOG" | wc -l)

if [ $ERREURS -gt 10 ]; then
    echo "🚨 ALERTE: Erreurs élevées ($ERREURS)"
fi
```

### **🚀 VÉRIFICATEUR DÉPLOIEMENT**
```bash
#!/bin/bash
ECHECS=()
DEPENDANCES=("curl" "docker" "git")

for dep in "${DEPENDANCES[@]}"; do
    command -v $dep >/dev/null || ECHECS+=("$dep")
done

if [ ${#ECHECS[@]} -eq 0 ]; then
    echo "✅ Déploiement possible"
    exit 0
else
    echo "❌ Problèmes: ${ECHECS[@]}"
    exit 1
fi
```

---

## **🎯 MÉTHODOLOGIE SCRIPTING PRO**

### **Approche Systématique Production :**
```bash
1. #!/bin/bash                          # Shebang
2. Vérification arguments               # -z "$1", $#
3. Déclaration variables/tableaux       # SERVICES=(), FICHIER_LOG=""
4. Logique métier principale            # Coeur du script
5. Gestion erreurs et rapports          # $?, exit codes
6. Nettoyage et rotation                # rm, rotation
7. Logging et sortie                    # echo, tee, exit
```

### **Bonnes Pratiques Production :**
- ✅ **Toujours** valider les arguments en entrée
- ✅ **Toujours** gérer les codes de sortie appropriés
- ✅ **Toujours** implémenter la rotation pour les backups
- ✅ **Toujours** fournir des rapports clairs et actionnables
- ✅ **Toujours** tester sur environnement de test d'abord

---

## **📈 PROGRESSION DAY 5**

**✅ Compétences Acquises :**
- Créer des scripts de monitoring système professionnels
- Implémenter des systèmes de backup avec rotation automatique
- Développer des analyseurs de logs avec alertes intelligentes
- Construire des vérificateurs de prérequis de déploiement
- Maîtriser la gestion avancée des tableaux et codes de sortie

**🎯 Mentalité DevOps Production :**
> Je ne me contente plus d'automatiser  
> Je crée des outils robustes pour l'environnement de production

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 5 / 100 ✅`**

**#Linux #Bash #Scripting #DevOps #Automation #Production #Monitoring #Backup**
