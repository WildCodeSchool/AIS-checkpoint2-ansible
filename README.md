Très bien ! Voici ton **README.md** complet, structuré, prêt pour être remis aux élèves.

Je l’ai rédigé avec un ton **clair, précis et pédagogique**, tout en respectant **la procédure de génération de certificat, dépôt des fichiers et déploiement Ansible**.

---

# 📚 Checkpoint 2 - Déploiement d'un site statique avec Ansible

---

## 🎯 Objectif

Déployer automatiquement un serveur web statique sous Apache, en sécurisant l'accès avec un certificat SSL/TLS signé par une Autorité de Certification interne (AC).

---

## 🛠️ Travail demandé

Tu disposes d'un projet Ansible générique à personnaliser.

Ton travail est de :
- Générer une clé privée et un fichier CSR pour le serveur web.
- Obtenir un certificat signé auprès de l'AC (Autorité de Certification).
- Déposer la clé privée et le certificat signé dans ton projet Ansible.
- Personnaliser les variables de configuration.
- Déployer automatiquement ton serveur web en HTTPS.

---

## 📂 Structure du projet

```
checkpoint2_ansible/
├── ansible.cfg
├── deploy_static_site.yml
├── inventory.yml
├── group_vars/
│   └── webservers.yml
└── roles/
    └── deploy_static_site/
        ├── tasks/main.yml
        ├── handlers/main.yml
        ├── templates/
        │   └── vhost.conf.j2
        ├── files/
        │   ├── index.html.j2
        │   ├── srv-web01.crt (à déposer)
        │   └── srv-web01.key (à déposer)
        └── vars/main.yml
```

---

## 🔥 Procédure détaillée

### 1. Générer la clé privée et la CSR

Sur ta machine **ControlNode** :

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout srv-web01.key -out srv-web01.csr -subj "/CN=srv-web01.checkpoint2.local"
```

✅ Résultat :
- `srv-web01.key` : clé privée
- `srv-web01.csr` : fichier de demande de certificat

---

### 2. Soumettre la demande CSR

- Accède à l'interface web de ton **Autorité de Certification** (AC) :  
  Exemple : http://srv-adds-adcs/certsrv
- Choisis "Demander un certificat".
- Téléverse ton fichier `srv-web01.csr`.
- Télécharge ton certificat signé (`srv-web01.crt`).

---

### 3. Déposer les fichiers dans Ansible

Dépose les deux fichiers suivants dans ton projet :
- `srv-web01.crt` → dans `roles/deploy_static_site/files/`
- `srv-web01.key` → dans `roles/deploy_static_site/files/`

---

### 4. Personnaliser les fichiers de configuration

- **`inventory.yml`** : renseigne ton IP de serveur et ton utilisateur SSH.

Exemple :

```yaml
all:
  hosts:
    srv-web01:
      ansible_host: 192.168.100.10
      ansible_user: ansible
  children:
    webservers:
      hosts:
        srv-web01
```

- **`group_vars/webservers.yml`** : complète les variables suivantes :

```yaml
http_host: "srv-web01.checkpoint2.local"
document_root: "/var/www/html"

ssl_certificate_path: "/etc/ssl/certs/srv-web01.checkpoint2.local.crt"
ssl_certificate_key_path: "/etc/ssl/private/srv-web01.checkpoint2.local.key"

enable_ssl: true

student_fullname: "Ton Prénom Ton Nom"
```

N’oublie pas :
- Mettre `enable_ssl: true` une fois ton certificat déployé.
- Renseigner ton **prénom et nom** dans `student_fullname`.

---

### 5. Lancer le déploiement Ansible

Active ton environnement virtuel si besoin :

```bash
source ~/ansible-venv/bin/activate
```

Puis exécute ton playbook :

```bash
ansible-playbook deploy_static_site.yml
```

✅ Résultat attendu :
- Apache2 est installé et démarré.
- Ton site est disponible sur `https://srv-web01.checkpoint2.local/` sans erreur SSL.
- Ton prénom et nom sont visibles sur la page d'accueil personnalisée.

---

## ✅ Critères de validation

- Le serveur web fonctionne en HTTPS sans alerte.
- Le certificat SSL est bien celui émis par l'AC.
- Ton prénom et ton nom sont affichés sur la page.
- Le déploiement a été entièrement réalisé via Ansible (pas d'intervention manuelle sur le serveur web).

---

## ⚡ Astuces

- Pour voir plus rapidement les erreurs, ajoute `-vvv` à la commande Ansible.
- Vérifie les permissions des fichiers `/etc/ssl/certs/` et `/etc/ssl/private/` sur le serveur web.
- Utilise `ansible-lint` pour valider ton projet si besoin.

---

# ✨ Bonne réussite pour ton checkpoint ! 🚀

---

# 📌 Résumé rapide des étapes

| Étape | Action |
|:-----|:-------|
| 1 | Générer CSR + clé |
| 2 | Envoyer CSR à l'AC |
| 3 | Récupérer certificat signé |
| 4 | Déposer `.key` et `.crt` dans Ansible |
| 5 | Compléter `inventory.yml` et `group_vars/webservers.yml` |
| 6 | Lancer `ansible-playbook deploy_static_site.yml` |
