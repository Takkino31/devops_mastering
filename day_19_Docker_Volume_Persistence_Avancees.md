# **DAY 19 - VOLUMES DOCKER AVANCÉS & SAUVEGARDES** 🔄💾

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture de Sauvegarde**
- **Sauvegardes automatisées** → Scripts et cron
- **Réplication de données** → MySQL Master-Slave
- **Gestion du cycle de vie** → Rotation des sauvegardes

### **📊 Stratégies de Persistance Avancée**
|-------------------------------|-----------------------|---------------------------|
| Type                          | Usage                 | Avantage                  |
|-------------------------------|-----------------------|---------------------------|
| **Sauvegarde manuelle**       | Développement         | Contrôle total            |
| **Sauvegarde automatisée**    | Production            | Régularité et fiabilité   |
| **Réplication**               | Haute disponibilité   | Redondance des données    |
|-------------------------------|-----------------------|---------------------------|

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Sauvegarde et Restauration**
|---------------------------------------|-----------------------|-----------------------|---------------------------|
| Commande                              | FR                    | EN                    | Usage                     |
|---------------------------------------|-----------------------|-----------------------|---------------------------|
| `docker run --rm -v volume:/source`   | Sauvegarder volume    | **BACKUP volume**     | Voir script ci-dessous    |
| `tar czf /backup/volume.tar.gz`       | Compresser données    | **CREATE archive**    | Sauvegarde compressée     |
| `tar xzf backup.tar.gz`               | Extraire sauvegarde   | **EXTRACT archive**   | Restauration              |
|---------------------------------------|-----------------------|-----------------------|---------------------------|

### **🔄 Réplication MySQL**
|-----------------------|---------------|-------------------|-----------------------------------|
| Commande              | FR            | EN                | Usage                             |
|-----------------------|---------------|-------------------|-----------------------------------|
| `SHOW MASTER STATUS`  | Statut master | **MASTER status** | `mysql -e "SHOW MASTER STATUS"`   |
| `SHOW SLAVE STATUS`   | Statut slave  | **SLAVE status**  | `mysql -e "SHOW SLAVE STATUS"`    |
|-----------------------|---------------|-------------------|-----------------------------------|

---

## **📝 STRATÉGIES AVANCÉES**

### **Sauvegarde Automatisée de Volumes**
```bash
#!/bin/bash
# backup-volumes.sh - Sauvegarde automatique de tous les volumes

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

echo "🔄 Début de la sauvegarde des volumes Docker..."

for volume in $(docker volume ls -q); do
    echo "📦 Sauvegarde du volume: $volume"
    
    mkdir -p $BACKUP_DIR/$DATE
    
    docker run --rm \
      -v $volume:/source \
      -v $BACKUP_DIR/$DATE:/backup \
      alpine \
      tar czf /backup/$volume.tar.gz -C /source .
    
    echo "✅ $volume sauvegardé"
done

# Nettoyage des vieilles sauvegardes
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;
```

### **Réplication MySQL avec Volumes**
```bash
# Master avec volume dédié
docker volume create mysql-master-data
docker run -d --name mysql-master \
  -v mysql-master-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_REPLICATION_MODE=master \
  mysql:8.0

# Slave avec volume séparé  
docker volume create mysql-slave-data
docker run -d --name mysql-slave \
  -v mysql-slave-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_REPLICATION_MODE=slave \
  mysql:8.0
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Sauvegarde Automatisée Complète :**
```bash
# Script de sauvegarde avec gestion d'erreurs
#!/bin/bash
BACKUP_DIR="/backups/docker"
LOG_FILE="/var/log/volume-backup.log"
DATE=$(date +%Y%m%d)

{
    echo "=== Sauvegarde du $(date) ==="
    
    for volume in $(docker volume ls -q); do
        echo "Sauvegarde: $volume"
        docker run --rm \
          -v $volume:/source:ro \
          -v $BACKUP_DIR:/backup \
          alpine \
          tar czf /backup/${volume}_${DATE}.tar.gz -C /source . \
          2>/dev/null && echo "✅ Succès" || echo "❌ Échec"
    done
    
    echo "Sauvegarde terminée"
} >> $LOG_FILE 2>&1
```

### **Réplication MySQL Fonctionnelle :**
```bash
# Configuration de la réplication
# Sur le master
docker exec mysql-master mysql -uroot -psecret -e "
  CREATE USER 'repl'@'%' IDENTIFIED BY 'replsecret';
  GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
  FLUSH PRIVILEGES;
"

# Sur le slave
docker exec mysql-slave mysql -uroot -psecret -e "
  CHANGE MASTER TO
  MASTER_HOST='mysql-master',
  MASTER_USER='repl',
  MASTER_PASSWORD='replsecret',
  MASTER_AUTO_POSITION=1;
  START SLAVE;
"
```

### **Vérification Réplication :**
```bash
# Vérifier le statut du master
docker exec mysql-master mysql -uroot -psecret -e "SHOW MASTER STATUS\G"

# Vérifier le statut du slave
docker exec mysql-slave mysql -uroot -psecret -e "SHOW SLAVE STATUS\G"

# Tester la réplication
docker exec mysql-master mysql -uroot -psecret -e "
  CREATE DATABASE test_replication;
  USE test_replication;
  CREATE TABLE test (id INT);
  INSERT INTO test VALUES (1);
"

docker exec mysql-slave mysql -uroot -psecret -e "
  USE test_replication;
  SELECT * FROM test;
"
# → Doit afficher la ligne insérée sur le master
```

---

## **🛠️ EXERCICES RÉALISÉS**

### **1. Script de Sauvegarde Automatisée**
```bash
#!/bin/bash
# backup-docker-volumes.sh

BACKUP_BASE="/opt/backups/docker"
RETENTION_DAYS=7
LOG_FILE="/var/log/docker-backup.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

log "🔁 Démarrage de la sauvegarde des volumes Docker"

# Créer le dossier de sauvegarde du jour
BACKUP_DIR="$BACKUP_BASE/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Sauvegarder chaque volume
for volume in $(docker volume ls -q); do
    log "📦 Sauvegarde du volume: $volume"
    
    if docker run --rm \
      -v "$volume":/source:ro \
      -v "$BACKUP_DIR":/backup \
      alpine \
      tar czf "/backup/${volume}.tar.gz" -C /source . > /dev/null 2>&1; then
        log "✅ $volume - Sauvegarde réussie"
    else
        log "❌ $volume - Échec de la sauvegarde"
    fi
done

# Nettoyer les vieilles sauvegardes
find "$BACKUP_BASE" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \; 2>/dev/null

log "🎉 Sauvegarde terminée - Dossier: $BACKUP_DIR"
```

### **2. Configuration MySQL Master-Slave**
```bash
# Lancer MySQL Master
docker volume create mysql-master-vol
docker run -d --name mysql-master \
  -v mysql-master-vol:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=masterpass \
  -e MYSQL_REPLICATION_MODE=master \
  -e MYSQL_REPLICATION_USER=replica_user \
  -e MYSQL_REPLICATION_PASSWORD=replica_pass \
  mysql:8.0

# Lancer MySQL Slave
docker volume create mysql-slave-vol  
docker run -d --name mysql-slave \
  -v mysql-slave-vol:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=slavepass \
  -e MYSQL_REPLICATION_MODE=slave \
  -e MYSQL_MASTER_HOST=mysql-master \
  -e MYSQL_MASTER_USER=replica_user \
  -e MYSQL_MASTER_PASSWORD=replica_pass \
  mysql:8.0

# Vérification
echo "=== Statut Master ==="
docker exec mysql-master mysql -uroot -pmasterpass -e "SHOW MASTER STATUS\G"

echo "=== Statut Slave ==="  
docker exec mysql-slave mysql -uroot -pslavepass -e "SHOW SLAVE STATUS\G"
```

### **3. Automatisation avec Cron**
```bash
# Ajouter dans crontab -e
# Sauvegarde tous les jours à 2h du matin
0 2 * * * /opt/scripts/backup-docker-volumes.sh

# Vérification des sauvegardes
0 6 * * * du -h /opt/backups/docker/ | mail -s "Rapport sauvegardes Docker" admin@localhost
```

### **4. Monitoring des Volumes**
```bash
# Script de monitoring de l'espace des volumes
#!/bin/bash
echo "=== ESPACE DISQUE VOLUMES DOCKER ==="

for volume in $(docker volume ls -q); do
    size=$(docker run --rm -v $volume:/data alpine du -sh /data 2>/dev/null | cut -f1)
    echo "📊 $volume: $size"
done
```

---

## **🎯 BONNES PRATIQUES SAUVEGARDE**

### **Checklist Sauvegarde Production :**
- ✅ **Sauvegarde automatique** quotidienne
- ✅ **Rotation** des sauvegardes (7 jours)
- ✅ **Test de restauration** régulier
- ✅ **Monitoring** de l'espace disque
- ✅ **Logs** détaillés des opérations

### **Stratégie de Rétention :**
```bash
# Garder les sauvegardes
- 7 sauvegardes quotidiennes
- 4 sauvegardes hebdomadaires  
- 12 sauvegardes mensuelles
```

### **Sécurité des Sauvegardes :**
```bash
# Chiffrement des sauvegardes sensibles
docker run --rm \
  -v sensitive-volume:/source \
  -v /backups:/backup \
  alpine \
  sh -c "tar cz -C /source . | openssl enc -aes-256-cbc -pass pass:secret -out /backup/volume_$(date +%Y%m%d).tar.gz.enc"
```

---

## **📈 PROGRESSION DAY 19**

**✅ Compétences Acquises :**
- Création de scripts de sauvegarde automatisés
- Configuration de réplication MySQL avec volumes
- Automatisation via cron pour les sauvegardes
- Monitoring de l'espace des volumes
- Gestion du cycle de vie des sauvegardes

**🎯 Mentalité DevOps :**
> Mes données ne sont plus à risque  
> J'ai mis en place un système de sauvegarde et réplication robuste

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 19 / 100 ✅`**

**#Docker #Backup #Replication #Automation #DevOps #MySQL**

---

**PRÊT POUR DOCKER COMPOSE ET L'ORCHESTRATION MULTI-SERVICES ?** 🚀
