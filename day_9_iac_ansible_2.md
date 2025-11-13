# **DAY 9 - MODULES ANSIBLE ESSENTIELS - AUTOMATISATION CONCRÈTE** ⚡

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Les 4 Modules Ansible Indispensables**
- **`apt`** → Gestionnaire de paquets (supermarket logiciel)
- **`copy`** → Fichiers statiques (photocopie)
- **`template`** → Fichiers dynamiques (plan d'architecte)
- **`service`** → Services système (télécommande)

### **🔑 Idempotence en Pratique**
- **Exécution multiple = même résultat**
- **Éviter les scripts non-idempotents**
- **Ansible gère les états, pas les actions**

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🚀 Exécution Playbooks**
| Commande                                          | FR                            | EN                | Usage                                                                 |
|---------------------------------------------------|-------------------------------|-------------------|-----------------------------------------------------------------------|
| `ansible-playbook -i inventory.ini playbook.yml`  | Exécuter playbook             | Run playbook      | `ansible-playbook -i inventory.ini apt-playbook.yml`                  |
| `ansible-playbook --ask-become-pass`              | Demander mot de passe sudo    | Ask sudo password | `ansible-playbook -i inventory.ini playbook.yml --ask-become-pass`    |
| `ansible-playbook --check`                        | Mode test                     | Dry-run mode      | `ansible-playbook -i inventory.ini playbook.yml --check`              |
|---------------------------------------------------|-------------------------------|-------------------|-----------------------------------------------------------------------|

### **📊 Analyse Résultats**
| Couleur | Signification | Action |
|-----------|-----------|-------------------------------|
| **VERT**  | `ok`      | Rien à faire - état déjà bon  |
| **JAUNE** | `changed` | Modification effectuée        |
| **ROUGE** | `failed`  | Erreur à résoudre             |
|-----------|-----------|-------------------------------|

---

## **⚡ MODULES MAÎTRISÉS**

### **📦 Module APT - Gestion Paquets**
```yaml
- name: Installer nginx
  apt:
    name: nginx
    state: present
    update_cache: no  # Éviter problèmes dépôts

- name: Installer plusieurs paquets
  apt:
    name:
      - curl
      - git
      - htop
    state: present

- name: Nettoyer le système
  apt:
    autoremove: yes
    autoclean: yes
```

**Paramètres clés :**
- `state: present/absent` → Installer/supprimer
- `update_cache: yes/no` → Mettre à jour cache apt
- `autoremove: yes` → Supprimer paquets inutiles

### **📄 Modules COPY & TEMPLATE - Gestion Fichiers**
```yaml
# Fichier statique
- name: Copier fichier
  copy:
    src: files/message.txt
    dest: /tmp/message.txt
    owner: root
    mode: '0644'

# Contenu direct
- name: Créer fichier avec contenu
  copy:
    content: "Ligne 1\nLigne 2"
    dest: /tmp/fichier-direct.txt

# Fichier dynamique
- name: Template avec variables
  template:
    src: templates/info.j2
    dest: /tmp/info-systeme.txt
```

**Différence copy vs template :**
- **`copy`** → Fichier identique
- **`template`** → Fichier personnalisé avec variables Jinja2

### **🎛️ Module SERVICE - Gestion Services**
```yaml
- name: Démarrer service
  service:
    name: nginx
    state: started

- name: Activer au démarrage
  service:
    name: nginx  
    enabled: yes

- name: Redémarrer service
  service:
    name: nginx
    state: restarted
```

**États disponibles :**
- `started` → Démarré (idempotent)
- `stopped` → Arrêté (idempotent) 
- `restarted` → Redémarré (toujours changed)
- `reloaded` → Rechargé (si supporté)

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Gestion des Erreurs Rencontrées :**
```bash
# Problème : Dépôt APT cassé
"E:The repository 'https://dl.pstmn.io/download/latest/linux64 ./ Release' does not have a Release file."

# Solution 1 : Désactiver update_cache
apt:
  name: nginx
  state: present
  update_cache: no

# Solution 2 : Ignorer erreurs au niveau TÂCHE
- name: Mettre à jour cache
  apt:
    update_cache: yes
  ignore_errors: yes  # ← Niveau TÂCHE, pas module!
```

### **Structure YAML Correcte :**
```yaml
- name: Tâche Ansible
  nom_module:          # ← Paramètres MODULE
    param1: valeur1
    param2: valeur2
  ignore_errors: yes   # ← Paramètres TÂCHE
  register: resultat   # ← Paramètres TÂCHE
  when: condition      # ← Paramètres TÂCHE
```

### **Handlers - Actions Déclenchées :**
```yaml
handlers:
  - name: redemarrer nginx
    service:
      name: nginx
      state: restarted

tasks:
  - name: Modifier config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: redemarrer nginx  # ← Déclenche le handler
```

---

## **🚀 PROJET RÉALISÉ : SERVEUR WEB ÉCLAIR**

### **Playbook Complet :**
```yaml
- name: Serveur web en 1 commande
  hosts: all
  become: yes
  
  vars:
    page_titre: "Mon Site Ansible"
    
  handlers:
    - name: redemarrer nginx
      service:
        name: nginx
        state: restarted

  tasks:
    - name: Installer nginx
      apt:
        name: nginx
        state: present
        
    - name: Déployer page web
      template:
        src: |
          <h1>{{ page_titre }}</h1>
          <p>Serveur: {{ ansible_hostname }}</p>
        dest: /var/www/html/index.html
      notify: redemarrer nginx
      
    - name: Démarrer nginx
      service:
        name: nginx
        state: started
```

### **Validation Automatique :**
```yaml
- name: Tester serveur web
  uri:
    url: http://localhost
    method: GET
  register: test_web

- name: Afficher résultat
  debug:
    msg: "✅ Status: {{ test_web.status }}"
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
# → Ansible gère tous les cas edge
```

### **Gestion des Erreurs :**
1. **Lire le message** → Ansible est très verbeux
2. **Identifier la source** → Module, connexion, permissions
3. **Solution progressive** → Désactiver fonctionnalité problématique
4. **Tester alternatives** → Différentes approches

### **Bonnes Pratiques :**
- ✅ **Toujours** utiliser `become: yes` pour les actions système
- ✅ **Toujours** tester avec `--check` avant production
- ✅ **Toujours** versionner les playbooks avec Git
- ✅ **Toujours** documenter les variables et handlers

---

## **📈 PROGRESSION DAY 9**

**✅ Compétences Acquises :**
- Maîtrise des 4 modules Ansible essentiels
- Gestion des erreurs courantes en production
- Création de playbooks idempotents
- Déploiement automatique de serveur web
- Différenciation paramètres tâche vs module

**🎯 Mentalité DevOps :**
> Je ne corrige plus les erreurs manuellement  
> J'écris du code qui gère automatiquement les cas edge

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 9 / 100 ✅`**

**#Ansible #Modules #Automation #DevOps #Idempotence #InfrastructureAsCode**

---


