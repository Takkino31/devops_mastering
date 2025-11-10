# **DAY 8 - INTRODUCTION À L'INFRASTRUCTURE AS CODE AVEC ANSIBLE** 🏗️

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Infrastructure as Code (IaC)**
- **Définition** → Gérer l'infrastructure via du code plutôt que manuellement
- **Avantage principal** → Reproductibilité et versioning
- **Problème résolu** → Éliminer les "snowflake servers" (serveurs uniques)

### **🔑 Concepts Fondamentaux IaC**
- **Idempotence** → Exécution multiple = même résultat
- **Déclaratif** → Décrire le "QUOI" plutôt que le "COMMENT"
- **Versioning** → Git pour l'infrastructure

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔧 Installation & Configuration**
| Commande                                  | FR                    | EN                    | Usage                                     |
|-------------------------------------------|-----------------------|-----------------------|-------------------------------------------|
| `sudo apt install ansible -y`             | Installer Ansible     | **Install Ansible**   | `sudo apt install ansible -y`             |
| `ansible --version`                       | Vérifier version      | **Check version**     | `ansible --version`                       |
| `ansible --version \| grep "config file"` | Voir fichier config   | **Show config file**  | `ansible --version \| grep "config file"` |
|-------------------------------------------|-----------------------|-----------------------|-------------------------------------------|

### **📋 Gestion Inventory**
| Commande                                      | FR                        | EN                    | Usage                                         |
|-----------------------------------------------|---------------------------|-----------------------|-----------------------------------------------|
| `ansible -i inventory.ini all --list-hosts`   | Lister tous les hôtes     | **List all hosts**    | `ansible -i inventory.ini all --list-hosts`   |
| `ansible -i inventory.ini web --list-hosts`   | Lister groupe spécifique  | **List group hosts**  | `ansible -i inventory.ini web --list-hosts`   |
| `ansible -i inventory.ini all -m ping`        | Tester connexion          | **Test connection**   | `ansible -i inventory.ini all -m ping`        |
|-----------------------------------------------|---------------------------|-----------------------|-----------------------------------------------|

### **🚀 Exécution Playbooks**
| Commande                                                  | FR                    | EN                | Usage                                                    |
|-----------------------------------------------------------|-----------------------|-------------------|----------------------------------------------------------|
| `ansible-playbook -i inventory.ini playbook.yml`          | Exécuter playbook     | **Run playbook**  | `ansible-playbook -i inventory.ini premier-playbook.yml` |
| `ansible-playbook -i inventory.ini playbook.yml --check`  | Mode test (dry-run)   | **Check mode**    | `ansible-playbook -i inventory.ini playbook.yml --check` |
|-----------------------------------------------------------|-----------------------|-------------------|----------------------------------------------------------|

---

## **⚡ CONCEPTS THÉORIQUES**

### **🎯 Idempotence en Pratique**
```yaml
# IDEMPOTENT - Peut s'exécuter 1000 fois
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present    # ✅ Présent = même résultat

# NON-IDEMPOTENT - Problème en ré-exécution  
- name: Install nginx
  command: apt-get install nginx  # ❌ Erreur si déjà installé
```

### **🏗️ Architecture Ansible**
```
Controller Node (Votre Machine)
    ↓ SSH (Agentless)
Managed Nodes (Serveurs)
    ↓
Inventory (Liste Hôtes)
    ↓  
Playbooks (YAML)
    ↓
Modules (Actions)
```

### **📦 Glossaire Ansible**
| Terme | Signification | Analogie |
|---------------|-----------------------|------------------------|
| **Inventory** | Liste des serveurs    | Carnet d'adresses 📇   |
| **Playbook**  | Automatisation YAML   | Livre de recettes 📖   |
| **Play**      | Ensemble de tâches    | Chapitre 🎯            |
| **Task**      | Action unitaire       | Étape recette 👨‍🍳       |
| **Module**    | Fonction prédéfinie   | Outil cuisine 🔪       |
| **Role**      | Playbook réutilisable | Recette standard 🍝    |
|---------------|-----------------------|------------------------|

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Structure Inventory :**
```ini
[web]
localhost ansible_connection=local
# web1 ansible_host=192.168.1.10 ansible_user=ubuntu

[db] 
# db1 ansible_host=192.168.1.20 ansible_user=ubuntu

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### **Premier Playbook Réussi :**
```yaml
---
- name: Mon premier playbook Ansible
  hosts: all
  become: yes
  gather_facts: yes

  tasks:
    - name: Message de bienvenue
      debug:
        msg: "🎉 Premier playbook réussi !"

    - name: Info système
      debug:
        msg: "Je suis {{ ansible_hostname }} sur {{ ansible_distribution }}"
```

### **Facts Ansible - Découverte Automatique :**
```bash
# Ansible collecte automatiquement :
- ansible_hostname      # Nom de l'hôte
- ansible_distribution  # OS (Ubuntu, CentOS...)
- ansible_memory_mb     # Mémoire RAM
- ansible_processor     # Processeur
- ansible_mounts        # Système de fichiers
```

---

## **🚀 EXERCICES RÉALISÉS**

### **1. Installation Ansible**
```bash
sudo apt update
sudo apt install ansible -y
ansible --version
# → Ansible 2.9+ avec Python 3
```

### **2. Création Inventory**
```bash
mkdir ansible-jour8 && cd ansible-jour8
cat > inventory.ini << EOF
[local]
localhost ansible_connection=local

[all:vars]
ansible_python_interpreter=/usr/bin/python3
EOF
```

### **3. Premier Playbook**
```yaml
# premier-playbook.yml
- name: Exploration système
  hosts: all
  become: yes
  tasks:
    - name: Afficher hostname
      debug:
        msg: "Host: {{ ansible_hostname }}"
    
    - name: Afficher OS
      debug: 
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

### **4. Exécution & Validation**
```bash
ansible-playbook -i inventory.ini premier-playbook.yml
# ✅ Tâches exécutées avec succès
# ✅ Facts système collectés automatiquement
```

---

## **🎯 MÉTHODOLOGIE ANSIBLE**

### **Approche Déclarative :**
```yaml
# AU LIEU DE :
- command: systemctl start nginx

# ON ÉCRIT :
- service:
    name: nginx
    state: started
# → Ansible gère tous les cas (déjà démarré, existe pas, etc.)
```

### **Structure de Projet :**
```
ansible-project/
├── inventory.ini          # Liste serveurs
├── playbook.yml          # Recette principale
├── group_vars/           # Variables par groupe
├── host_vars/           # Variables par hôte
└── roles/               # Playbooks réutilisables
```

### **Bonnes Pratiques :**
- ✅ **Toujours** utiliser `become: yes` pour les privilèges
- ✅ **Toujours** activer `gather_facts: yes` pour les infos système
- ✅ **Toujours** tester avec `--check` avant en production
- ✅ **Toujours** versionner avec Git

---

## **📈 PROGRESSION DAY 8**

**✅ Compétences Acquises :**
- Compréhension des concepts fondamentaux de l'IaC
- Installation et configuration d'Ansible
- Création et gestion d'inventories
- Écriture de premiers playbooks YAML
- Utilisation des facts Ansible pour l'exploration système

**🎯 Mentalité DevOps :**
> Je ne configure plus manuellement  
> Je décris mon infrastructure en code pour la reproduire à l'infini

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 8 / 100 ✅`**

**#Ansible #InfrastructureAsCode #DevOps #Automation #IaC #YAML #Git**

---
