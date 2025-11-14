# **DAY 10 - ANSIBLE AVANCÉ - VARIABLES, CONDITIONS ET ROLES** 🚀

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Organisation Professionnelle Ansible**
- **Structure modulaire** pour la maintenabilité
- **Séparation des préoccupations** via les rôles
- **Gestion multi-environnements** avec les inventaires

### **📦 Les 3 Piliers de l'Ansible Avancé**
- **Variables** → Configuration dynamique et hiérarchique
- **Conditions** → Logique métier et exécution contextuelle  
- **Roles** → Composants réutilisables et modulaires

---

## **🛠️ COMMANDES ESSENTIELLES**

### **📁 Structure de Projet**
| Commande                                              | Usage                                 | Description                           |
|-------------------------------------------------------|---------------------------------------|---------------------------------------|
| `mkdir -p roles/webserver/{tasks,handlers,templates}` | Créer structure rôle                  | Structure standard Ansible            |
| `ansible-playbook -i inventories/prod.ini site.yml`   | Playbook avec inventory spécifique    | Déploiement par environnement         |
| `ansible-playbook --limit web_servers`                | Limiter l'exécution                   | Cibler des hôtes spécifiques          |
| `ansible-playbook --tags "webserver"`                 | Exécuter par tags                     | Sélectionner des parties du playbook  |
|-------------------------------------------------------|---------------------------------------|---------------------------------------|

### **🔧 Gestion des Erreurs**
| Commande                  | Usage                 | Description                       |
|---------------------------|-----------------------|-----------------------------------|
| `ls roles/`               | Lister les rôles      | Vérifier les rôles disponibles    |
| `tree roles/webserver/`   | Voir structure rôle   | Inspector l'organisation          |
|---------------------------|-----------------------|-----------------------------------|

---

## **⚡ CONCEPTS AVANCÉS MAÎTRISÉS**

### **🎯 Variables Hiérarchiques**
```yaml
# Hiérarchie de priorité des variables :
1. `--extra-vars` en ligne de commande
2. `host_vars/` → Variables par hôte
3. `group_vars/` → Variables par groupe  
4. `roles/*/vars/` → Variables de rôle
5. `roles/*/defaults/` → Valeurs par défaut
6. `playbook vars:` → Variables de playbook
```

### **📊 Types de Variables**
```ini
# inventories/production.ini
[web_servers]
web1.example.com ansible_user=ubuntu  # ← Variables directes

[production:vars]
environment=production  # ← Variables de groupe
```

```yaml
# group_vars/web_servers.yml
web_server: nginx
web_port: 80
packages:
  - nginx
  - curl
```

```yaml
# host_vars/web1.example.com.yml
web_port: 8080  # ← Surcharge pour cet hôte
server_name: "web1.prod.com"
```

### **🚦 Conditions et Logique**
```yaml
- name: Tâche conditionnelle simple
  apt:
    name: nginx
    state: present
  when: deploy_web | bool  # ← Condition booléenne

- name: Condition complexe
  debug:
    msg: "Environnement de production"
  when: 
    - environment == "production"
    - deploy_web | bool

- name: Condition avec liste
  debug:
    msg: "Environnement valide"
  when: environment in ['development', 'staging', 'production']
```

### **🔄 Boucles et Itérations**
```yaml
- name: Installer plusieurs paquets
  apt:
    name: "{{ item }}"
    state: present
  loop: "{{ packages }}"  # ← Boucle sur liste
  when: packages is defined

- name: Boucle avec condition
  debug:
    msg: "Traitement de {{ item }}"
  loop:
    - "nginx"
    - "mysql"
    - "redis"
  when: item != "mysql" or deploy_db | bool
```

---

## **🏗️ ROLES ANSIBLE - ORGANISATION PRO**

### **📁 Structure Standard d'un Rôle**
```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml          # Tâches principales
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── templates/
    │   └── nginx.conf.j2     # Templates
    ├── files/
    │   └── index.html        # Fichiers statiques
    ├── vars/
    │   └── main.yml          # Variables du rôle
    └── defaults/
        └── main.yml          # Valeurs par défaut
```

### **🎯 Rôle Webserver Complet**
```yaml
# roles/webserver/defaults/main.yml
web_port: 80
web_root: /var/www/html
server_name: "localhost"
web_packages:
  - nginx
  - curl
```

```yaml
# roles/webserver/tasks/main.yml
- name: Installer paquets web
  apt:
    name: "{{ item }}"
    state: present
  loop: "{{ web_packages }}"
  when: web_packages is defined

- name: Configurer nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify: restart nginx

- name: Démarrer nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

```jinja2
{# roles/webserver/templates/nginx.conf.j2 #}
server {
    listen {{ web_port }};
    server_name {{ server_name }};
    root {{ web_root }};
}
```

### **🔧 Utilisation des Rôles**
```yaml
# site.yml
- name: Déploiement web
  hosts: web_servers
  become: yes
  roles:
    - role: webserver
      web_port: 8080                    # ← Surcharge variable
      server_name: "mon-site.com"
```

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Gestion des Erreurs de Rôles**
```bash
# ERREUR : Rôle non trouvé
ERROR! the role 'database' was not found

# SOLUTIONS :
1. Créer le rôle manquant
2. Modifier le playbook pour utiliser des tasks directs
3. Limiter l'exécution aux rôles existants
```

### **Bonnes Pratiques Variables**
```yaml
# DÉFAUTS pour valeurs modifiables
# roles/webserver/defaults/main.yml
web_port: 80

# VARS pour valeurs fixes  
# roles/webserver/vars/main.yml
required_packages:
  - nginx
  - openssl
```

### **Conditions Avancées**
```yaml
- name: Exécution conditionnelle avec registre
  command: /usr/bin/quelque-chose
  register: result
  changed_when: result.rc == 0
  failed_when: 
    - result.rc != 0
    - environment == "production"
```

---

## **🚀 PROJET RÉALISÉ : INFRASTRUCTURE ORCHESTRÉE**

### **Structure Finale**
```
ansible-jour10/
├── inventories/
│   └── production.ini
├── group_vars/
│   └── web_servers.yml
├── roles/
│   └── webserver/
│       ├── tasks/
│       ├── handlers/
│       └── templates/
└── site.yml
```

### **Playbook Principal**
```yaml
# site.yml
- name: Orchestration infrastructure
  hosts: web_servers
  become: yes
  roles:
    - webserver
```

### **Avantages Obtenus**
- ✅ **Réutilisabilité** → Rôles utilisables sur multiple projets
- ✅ **Maintenabilité** → Code organisé et lisible
- ✅ **Configurabilité** → Variables par environnement
- ✅ **Évolutivité** → Ajout facile de nouveaux composants

---

## **🎯 MÉTHODOLOGIE ANSIBLE AVANCÉE**

### **Approche de Développement**
1. **Commencer simple** → Tasks directs dans le playbook
2. **Extraire en rôles** → Quand la complexité augmente
3. **Configurer avec variables** → Pour la flexibilité
4. **Ajouter conditions** → Pour l'intelligence contextuelle

### **Gestion des Environnements**
```bash
# Développement
ansible-playbook -i inventories/dev.ini site.yml

# Production
ansible-playbook -i inventories/prod.ini site.yml

# Test spécifique
ansible-playbook -i inventories/prod.ini site.yml --limit web_servers --tags "webserver"
```

### **Bonnes Pratiques**
- ✅ **Toujours** utiliser des valeurs par défaut dans les rôles
- ✅ **Toujours** documenter les variables obligatoires
- ✅ **Toujours** tester les rôles indépendamment
- ✅ **Toujours** versionner la structure complète

---

## **📈 PROGRESSION DAY 10**

**✅ Compétences Acquises :**
- Maîtrise des variables hiérarchiques Ansible
- Implémentation de conditions et boucles avancées
- Création et utilisation de rôles modulaires
- Gestion d'erreurs de dépendances entre rôles
- Organisation professionnelle de projets Ansible

**🎯 Mentalité DevOps :**
> Je ne crée plus des playbooks, j'orchestre des infrastructures  
> Mon code est réutilisable, configurable et maintenable

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 10 / 100 ✅`**

**#Ansible #DevOps #InfrastructureAsCode #Automation #Roles #Variables #Conditions**

---
