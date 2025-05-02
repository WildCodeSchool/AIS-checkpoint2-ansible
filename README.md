# 🛠️ AIS-checkpoint2-ansible

Ce projet Ansible automatise le déploiement d’un site web statique sécurisé via HTTPS, en intégrant une gestion des certificats SSL avec une autorité de certification interne AD CS.

Il s’inscrit dans un contexte de formation AIS (Administrateur d’Infrastructures Sécurisées).

---
Ce projet est composé de deux roles :
* `deploy_static_site` qui permet de déployer un site statique avec ou sans certificat
* `generate_ssl_key_csr` qui génère une clé privée et crée un fichier CSR

---

## 📁 Structure du projet

```
AIS-checkpoint2-ansible/
├── ansible.cfg
├── inventory.yml
├── group_vars/
│   ├── all.yml
│   └── webservers.yml
├── host_vars/
│   └── srv-web01.yml
├── generate_csr.yml
├── deploy_static_site.yml
├── requirements.txt
├── roles/
│   ├── generate_ssl_key_csr/
│   │   ├── tasks/
│   │   ├── vars/
│   │   ├── files/
│   │   └── templates/
│   └── deploy_static_site/
│       ├── tasks/
│       ├── vars/
│       ├── templates/
│       └── handlers/
└── logs/
```

---

## ⚙️ Prérequis

- Un serveur `controlnode` Linux (Debian/Ubuntu) avec :
  - Python 3
  - OpenSSL
  - Git
  - Un environnement virtuel Python avec Ansible
- Au moins un serveur web Linux (ex: `srv-web01`)

---

## 🚀 Installation et configuration

### ✨ 1. Cloner le dépôt

```bash
wilder@ControlNode:~$ cd ~
wilder@ControlNode:~$ git clone https://github.com/WildCodeSchool/AIS-checkpoint2-ansible.git
```

### ✨ 2. Créer et activer un environnement virtuel

```bash
wilder@ControlNode:~$ cd ~/AIS-checkpoint2-ansible
wilder@ControlNode:~/AIS-checkpoint2-ansible$ mkdir ansible-venv
wilder@ControlNode:~/AIS-checkpoint2-ansible$ python3 -m venv ansible-venv
wilder@ControlNode:~/AIS-checkpoint2-ansible$ source ansible-venv/bin/activate
```

### ✨ 3. Installer les dépendances

```bash
(ansible-venv) wilder@ControlNode:~/AIS-checkpoint2-ansible$ pip install -r requirements.txt
```

> [!success]
> Tu es pret(e) à utiliser le projet

---

## 🔧 Les variables


### `inventory.yml`

```yml
all:
  children:
    webservers:
      hosts:
        srv-web01:
          ansible_host: "IP du serveur"
          ansible_user: "wilder"
          ansible_ssh_private_key_file: "Le chemin vers ta clé privée SSH" # Si tu n'utilises pas d'agent SSH, sinon, commentes cette ligne
    controlnode:
      hosts:
        controlnode:
          ansible_connection: local
```

Fichier d'inventaire déclaré dans `ansible.cfg`.

> [!info] A modifier
> à modifier :
> * ansible_host:
> * ansible_ssh_private_key_file:



### `group_vars/webservers.yml`

Variables communes à tous les serveurs web.

```yml
---
# Ces deux variables définissent les chemins absolus où seront installés (copiés) les certificats SSL et les clés privées sur les serveurs web
ssl_certificate_path: "/etc/ssl/certs/{{ inventory_hostname }}.crt"
ssl_certificate_key_path: "/etc/ssl/private/{{ inventory_hostname }}.key"

# Ces variables sont utilisées dans le rôle generate_ssl_key_csr pour générer automatiquement les fichiers `.cnf`
# nécessaires à la création des **CSR** (Certificate Signing Requests) avec **OpenSSL**.
# Chaque CSR utilise ces informations génériques.
generate_ssl_key_csr_country: "FR"
generate_ssl_key_csr_locality: "TA VILLE"
generate_ssl_key_csr_organization: "checkpoint2.local"
generate_ssl_key_csr_organizational_unit: "WEB"
generate_ssl_key_csr_domain: "checkpoint2.local"

# Variable utilisée dans le fichier de site statique pour afficher ton nom dans la page
student_fullname: "Ton Prénom Ton Nom"
```
> [!info] A modifier
> Variable à modifier :
> * generate_ssl_key_csr_locality:
> * student_fullname:


### `host_vars/<host>.yml`

Variables propres à chaque serveur web.

```yml
---
# Définit le répertoire racine où seront stockés les fichiers du site statique à servir via Apache.
http_host: "srv-web01.checkpoint2.local"
document_root: "/var/www/html/{{ http_host }}"

# liste des noms DNS ou alias (SAN – Subject Alternative Names) supplémentaires que le certificat SSL doit couvrir, propre à chaque serveur web
generate_ssl_key_csr_subject_alt_names:
  - srv-web01.checkpoint2.local
  - srv-web01

# Active ou non le HTTPS pour ce serveur de façon granulaire
# Contrôle les tâches conditionnelles dans le rôle `deploy_static_site`
# Passe à true une fois le certificat signé par ton AC et déposé dans `roles/deploy_static_site/files
deploy_static_site_enable_ssl: false
```

> [!info] A modifier
> Variable à modifier :
> * http_host:
> * generate_ssl_key_csr_subject_alt_names:
> * deploy_static_site_enable_ssl:

> [!todo]
> Dans le cas ou tu veux déployer plusieurs serveurs web sur plusieurs noeuds, il te faudra dupliquer ce fichier
> et modifier les valeurs correspondantes à ces autres serveurs web.
> Tu devras aussi rajouter ces serveurs web dans ton inventaire.


### `roles/deploy_static_site/vars/main.yml`

```yml
---
deploy_static_site_apache_packages:
  - apache2
```
> [!info] A modifier
> Variable à modifier : Aucune


### `roles/generate_ssl_key_csr/vars/main.yml`

```yml
---
generate_ssl_key_csr_key_dir: "{{ playbook_dir }}/roles/generate_ssl_key_csr/files/keys"
generate_ssl_key_csr_csr_dir: "{{ playbook_dir }}/roles/generate_ssl_key_csr/files/csr"
generate_ssl_key_csr_cnf_dir: "{{ playbook_dir }}/roles/generate_ssl_key_csr/files/cnf"
```
> [!info] A modifier
> Variable à modifier : Aucune


---

## 🔑 Rôle `generate_ssl_key_csr`

Ce rôle permet de :
- Générer une clé privée ECC (courbe `prime256v1`)
- Générer un fichier de configuration `.cnf`
- Créer une CSR (Certificate Signing Request)
- Copier clé privée dans `roles/deploy_static_site/files/`


### ▶️ Lancement

```bash
ansible-playbook -i inventory.yml generate_csr.yml
```

> [!info]
> Ce playbook est configuré pour cibler les noeuds membre du groupe **webservers** dans ton inventaire

---

## 🌐 Rôle `deploy_static_site`

Ce rôle permet de :
- Installer Apache2
- Déployer un site HTML statique
- Configurer un VirtualHost Apache
- Activer le SSL avec un certificat interne (si variable `enable_ssl: true`)


### ▶️ Lancement

```bash
ansible-playbook -i inventory.yml deploy_static_site.yml --ask-become-pass
```

> [!info]
> Ce playbook est configuré pour cibler le noeud local **ControlNode** dans ton inventaire


---

## 🔐 Fichiers de certificats

Les certificats signés par l'AC doivent être placés à l'emplacement ci-dessous avant de passer la variable `enable_ssl: ` à `true` :

```bash
roles/deploy_static_site/files/
├── srv-web01.key    # Clé privée
├── srv-web01.crt    # Certificat signé par l'AC
```
Le certificat racine (`checkpoint2-rootCA`) pourra aussi etre placé ici, afin de le déployer manuellement sur le système et dans le navigateur de `ControlNode`

> [!attention]
> **Attention :** Le certificat signé devra etre au format `crt` !
> Si ton AC a produit un fichier de certificat au format `cer`, il te suffit de modifier son extention :
> ```bash
> mv roles/deploy_static_site/files/checkpoint2-rootCA.cer roles/deploy_static_site/files/checkpoint2-rootCA.crt
> mv roles/deploy_static_site/files/srv-web01.cer roles/deploy_static_site/files/srv-web01.crt
> ```

---

## 🧪 Tests et vérifications

Une fois le déploiement terminé, tu peux accéder au site web via :

* https://srv-web01.checkpoint2.local
ou
* https://srv-web01


> ⚠️ Assure-toi que le certificat racine est bien installé sur `controlnode` et son navigateur, sinon tu auras une erreur de certificat.


✅ Résultat attendu :
- Ton prénom et nom sont visibles sur la page d'accueil personnalisée
- Ton site est disponible en HTTPS sans erreur SSL avec les noms déclarés dans le fichier `host_vars/<host>.yml`

Exemple :
```yml
generate_ssl_key_csr_subject_alt_names:
  - srv-web01.checkpoint2.local
  - srv-web01
```

---

## 📝 Auteurs

Projet pédagogique dans le cadre de la formation **Administrateur d’Infrastructures Sécurisées (AIS)** à la [WildCodeSchool](https://www.wildcodeschool.com/)

Développé par [Julien GREGOIRE](https://github.com/JulienWild), formateur AIS