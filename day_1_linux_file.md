## DAY 1

### 🧭 Navigation
| Commande | FR | EN |
|---|---|---|
| `pwd`                             | Afficher le dossier courant   | Print Working Directory |
| `ls`                              | Lister fichiers/dossiers      | List directory contents |
| `ls -la`                          | Tout afficher (détails)       | List all incl. hidden (long format) |
| `cd`                              | Changer de dossier            | Change directory |
| `cd ..`                           | Dossier parent                | Go up one directory |
| `cd /etc`                         | Aller dans `/etc`             | Change directory to `/etc` |
| `find /path -name "*.log"`        | Trouver fichiers              | Find files |
| `locate fichier`                  | Recherche rapide indexée      | Database-based fast search |

### 📖 Lecture fichiers
| Commande | FR | EN |
|---|---|---|
| `cat file`        | Afficher fichier      | Concatenate & print file |
| `less file`       | Lire avec navigation  | Interactive file viewer |
| `head -n 10 file` | 10 premières lignes   | First 10 lines |
| `tail -n 20 file` | 20 dernières lignes   | Last 20 lines |
| `tail -f file`    | Suivi en temps réel   | Follow file updates |

### 🔎 Filtrage & Extraction
| Commande | FR | EN |
|---|---|---|
| `grep "mot"`              | Chercher texte        | Search text |
| `grep -i "error"`         | Sensibilité casse off | Case-insensitive search |
| `grep -R "mot"`           | Recherche récursive   | Recursive search |
| `awk '{print $1,$2}'`     | Colonnes              | Print specific columns |
| `sed -n '1,10p'`          | Lignes 1-10           | Print lines 1-10 |

### 🔐 Permissions
| Commande | FR | EN |
|---|---|---|
| `chmod 754 file`      | Permissions rwx r-x r--   | Set permissions |
| `chmod 600 file`      | RW owner seul             | Owner read/write only |
| `chown user:group`    | Changer propriétaire      | Change file owner |
| `umask`               | Voir masque               | Show permission mask |
| `umask 022`           | Définir masque            | Set permission mask |

### 🎯 Spécial logs
| Commande | FR | EN |
|---|---|---|
| `find /var -type f -name "*.log" -mtime -1` | Logs < 24h | Logs modified < 24h |
| `2>/dev/null` | Masquer erreurs | Suppress errors |

