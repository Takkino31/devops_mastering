---

# **DAY 3 - RÉSEAU LINUX : DE L'AVEUGLEMENT À LA MAÎTRISE** 🌐

## **🎯 CONCEPTS CLÉS APPRIS**

### **🏗️ Architecture Réseau Simplifiée**
- **IP** = Adresse de la maison 🏠
- **Port** = Porte d'entrée 🚪
- **Socket** = Conversation active 🗣️
- **Firewall** = Vigile de sécurité 🛡️

### **📦 Modèle TCP/IP DevOps**
```
APPLICATION    → HTTP, SSH, DNS (CE QUE tu veux faire)
TRANSPORT      → TCP, UDP (COMMENT l'envoyer)  
RÉSEAU         → IP, Routage (OÙ l'envoyer)
ACCÈS RÉSEAU   → Câble, Wi-Fi (CÂBLE physique)
```

---

## **🛠️ COMMANDES ESSENTIELLES**

### **🔍 Diagnostic Réseau**
| Commande | FR | EN | Usage |
|---------------------------|---------------------------|---------------------------------------------------|---------------------------|
| `ping IP`                 | Tester accessibilité      | **Packet Internet Groper** - Test connectivity    | `ping 8.8.8.8`            |
| `traceroute IP`           | Tracer chemin             | **Trace Route** - Trace network path              | `traceroute google.com`   |
| `nslookup domaine`        | Résolution DNS            | **Name Server Lookup** - DNS resolution           | `nslookup google.com`     |
| `ss -tulnp`               | Sockets écoutants         | **Socket Statistics** - Listening sockets         | `ss -tulnp`               |
| `netstat -tulnp`          | Alternative à ss          | **Network Statistics** - Network connections      | `netstat -tulnp`          |

### **🌐 Test Services**
| Commande | FR | EN | Usage |
|---------------------------|-----------------------|-----------------------------------------------|-------------------------------|
| `curl http://IP:PORT`     | Tester HTTP           | **Client URL** - Test HTTP service            | `curl http://localhost:80`    |
| `wget URL`                | Télécharger           | **Web Get** - Web download                    | `wget http://example.com`     |
| `telnet IP PORT`          | Test connexion        | **Teletype Network** - Raw connection test    | `telnet localhost 22`         |

### **🛡️ Firewall UFW**
| Commande                    | FR                        | EN                                                        | Usage                     |
|-----------------------------|---------------------------|-----------------------------------------------------------|---------------------------|
| `sudo ufw status`           | Voir règles               | **Uncomplicated Firewall Status** - Show firewall rules   | `sudo ufw status`         |
| `sudo ufw allow PORT`       | Ouvrir port               | **UFW Allow** - Allow port traffic                        | `sudo ufw allow 8080`     |
| `sudo ufw deny PORT`        | Bloquer port              | **UFW Deny** - Deny port traffic                          | `sudo ufw deny 8080`      |
| `sudo ufw status verbose`   | Statut détaillé           | **UFW Status Verbose** - Detailed status                  | `sudo ufw status verbose` |
| `sudo ufw delete deny PORT` | Supprimer règle port      | **UFW Delete Verbose** - Delete rule                      | `sudo ufw delete deny 8080` |

### **🔧 Commandes Système**
| Commande              | FR                                                                                | EN                    |
|-----------------------|-----------------------------------------------------------------------------------|-----------------------|
| `ip addr show`        | Voir interfaces | **IP Address Show** - Show network interfaces                   | `ip addr show`        |
| `hostname -I`         | Afficher IP | **Hostname Show IP** - Display IP addresses                         | `hostname -I`         |
| `lsof -i :PORT`       | Processus utilisant port | **List Open Files** - Show processes using port        | `lsof -i :80`         |

---

## **🎯 MÉTHODOLOGIE DE DIAGNOSTIC**

### **Approche Systématique :**
```bash
# 1. Connectivité de base
ping IP

# 2. Service écoute-t-il ?
ss -tulnp | grep PORT

# 3. Test local du service  
curl http://localhost:PORT

# 4. Firewall bloque-t-il ?
sudo ufw status

# 5. Test depuis l'extérieur
curl http://IP_LOCALE:PORT
```

### **🚨 Les 5 Blocages Réseau Courants :**
1. **Problème de Connectivité** → `ping` échoue
2. **Service Non Démarré** → Port pas en écoute
3. **Firewall Bloque** → Local OK, externe KO
4. **Problème DNS** → IP marche, nom échoue
5. **Mauvaise Interface** → Service écoute sur 127.0.0.1 au lieu de 0.0.0.0

---

## **💡 DÉCOUVERTES IMPORTANTES**

### **Le Mystère du Firewall :**
```bash
# Règle active :
sudo ufw deny 8090
sudo iptables -L | grep 8090
# → DROP tcp dpt:8090

# Mais curl local fonctionne quand même !
# → Le trafic local peut bypasser certaines règles firewall
```

### **Solution : Lier le service à une IP spécifique**
```bash
# Pour forcer l'écoute locale seulement :
python3 -m http.server 8090 --bind 127.0.0.1

# Résultat :
curl http://localhost:8090      # ✅ SUCCÈS
curl http://192.168.1.89:8090   # ❌ ÉCHEC
```

---

## **🎓 PORTS ESSENTIELS À CONNAÎTRE**

| Port | Service | Signification |
|---|---|---|
| **22** | SSH | **Secure Shell** - Connexions sécurisées |
| **80** | HTTP | **HyperText Transfer Protocol** - Web non sécurisé |
| **443** | HTTPS | **HTTP Secure** - Web sécurisé |
| **53** | DNS | **Domain Name System** - Résolution de noms |
| **3306** | MySQL | **MySQL Database** - Base de données |

---

## **🚀 SCÉNARIO RÉUSSI**

### **Panne Simulée & Résolue :**
**Problème :** Service web sur port 8090 inaccessible depuis l'extérieur

**Diagnostic :**
```bash
ss -tulnp | grep 8090
# → 0.0.0.0:8090 = écoute toutes interfaces ✅

sudo ufw status | grep 8090  
# → 8090 DENY = firewall bloque ✅

curl http://localhost:8090
# ✅ SUCCÈS = service fonctionne

curl http://192.168.1.89:8090
# ✅ SUCCÈS = mystère firewall!
```

**Découverte :** Le trafic local peut contourner certaines règles firewall

**Solution :** Lier le service à `127.0.0.1` seulement

---

## **📈 PROGRESSION DAY 3**

**✅ Compétences Acquises :**
- Diagnostiquer la connectivité réseau
- Identifier les ports en écoute
- Tester les services localement et à distance
- Configurer le firewall UFW
- Comprendre les subtilités du trafic local vs externe

**🎯 Mentalité DevOps :**
> Je ne dis plus "Ça marche pas"  
> Je dis "Le service écoute sur 127.0.0.1:8090 au lieu de 0.0.0.0:8090, donc il n'est pas accessible depuis l'extérieur"

---

**🔗 [GitHub - Notes Complètes](https://github.com/Takkino31/devops_mastering)**

**📊 Progress: `Day 3 / 100 ✅`**


---
