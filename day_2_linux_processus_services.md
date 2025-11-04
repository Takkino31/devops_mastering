---

## **DAY 2 - PROCESSUS & SERVICES**

### **📊 Gestion des Processus**
*Comprendre et contrôler les programmes en cours d'exécution sur le système*

| Commande              | FR                        | EN                                |
|-----------------------|---------------------------|-----------------------------------|
| `ps aux`              | Voir tous les processus   | Process Status (all users)        |
| `top`                 | Monitoring temps réel     | Table of Processes (real-time)    |
| `htop`                | Interface avancée         | Enhanced process viewer           |
| `pstree`              | Arbre des processus       | Process tree visualization        |
| `pgrep process`       | Trouver PID par nom       | Process Grep (find PID)           |

### **⚙️ Création & Contrôle**
*Lancer et gérer les processus de manière sécurisée*

| Commande              | FR                        | EN                                |
|-----------------------|---------------------------|-----------------------------------|
| `commande &`          | Lancer en arrière-plan    | Run command in background         |
| `jobs`                | Voir mes jobs             | Display background jobs           |
| `timeout 60 cmd`      | Limiter dans le temps     | Run command with time limit       |
| `kill PID`            | Arrêter processus         | Send TERM signal to process       |
| `kill -9 PID`         | Tuer forcément            | Send KILL signal (force)          |
| `killall process`     | Tuer par nom              | Kill processes by name            |
| `kill -STOP PID`      | Suspendre processus       | Stop/pause process                |
| `kill -CONT PID`      | Reprendre processus       | Continue paused process           |

### **🔧 Services Systemd**
*Gérer les services système (démarrer, arrêter, activer)*

| Commande                          | FR                        | EN                                |
|-----------------------------------|---------------------------|-----------------------------------|
| `systemctl status service`        | État du service           | Service status details            |
| `systemctl start service`         | Démarrer service          | Start service now                 |
| `systemctl stop service`          | Arrêter service           | Stop service now                  |
| `systemctl restart service`       | Redémarrer service        | Restart service                   |
| `systemctl enable service`        | Activer au démarrage      | Enable auto-start on boot         |
| `systemctl disable service`       | Désactiver démarrage      | Disable auto-start                |
| `systemctl is-active service`     | Vérifier si actif         | Check if service is running       |
| `systemctl is-enabled service`    | Vérifier si activé        | Check if service enabled          |

### **📋 Logs avec Journalctl**
*Lire et analyser les logs système en temps réel*

| Commande                      | FR                        | EN                                |
|-------------------------------|---------------------------|-----------------------------------|
| `journalctl -f`               | Suivre logs temps réel    | Follow journal (real-time)        |
| `journalctl -u service`       | Logs d'un service         | Journal for specific unit         |
| `journalctl -p err`           | Seulement erreurs         | Show only error priority          |
| `journalctl --since "1h"`     | Depuis 1 heure            | Show logs since 1 hour ago        |
| `journalctl -n 20`            | 20 dernières lignes       | Last 20 journal entries           |

### **🎯 États des Processus**
*Comprendre ce que font vos processus*

| État  | FR                | EN            | Signification             |
|-------|-------------------|---------------|---------------------------|
| **R** | En cours          | Running       | Utilise le CPU            |
| **S** | Endormi           | Sleeping      | Attend un événement       |
| **D** | Sommeil disque    | Disk Sleep    | Bloqué sur I/O            |
| **Z** | Zombie            | Zombie        | Terminé mais pas nettoyé  |
| **T** | Arrêté            | Stopped       | Suspendu (Ctrl+Z)         |

### **🚨 Signaux Importants**
*Communiquer avec les processus*

| Signal | Numéro | Usage |
|---|---|---|
| **SIGTERM** | 15 | Arrêt propre (défaut) |
| **SIGKILL** | 9 | Arrêt forcé (danger) |
| **SIGSTOP** | 19 | Suspendre le processus |
| **SIGCONT** | 18 | Reprendre le processus |

---

## **💡 BONNES PRATIQUES DAY 2**

- **Toujours** utiliser `timeout` avec les commandes gourmandes
- **Toujours** mettre un `sleep` dans les boucles infinies  
- **Toujours** essayer `kill` avant `kill -9`
- **Toujours** tester sans `-i` avant de modifier des fichiers
- **Toujours** surveiller avec `top` ou `htop` les nouveaux processus

---
