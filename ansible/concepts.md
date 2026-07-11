# Ansible — Concepts

## Qu'est-ce qu'Ansible ?

Ansible est un outil d'**automatisation de configuration** et de **provisioning**. Il exécute des instructions (tasks) sur des machines distantes via SSH, sans agent installé sur les cibles.

| Concept | Description |
|---------|-------------|
| **Hôte** | Une machine distante gérée par Ansible |
| **Inventaire** | Liste des hôtes cibles (fichier `inventory` ou `hosts`) |
| **Module** | Unité d'action (copy, apt, service, template...) |
| **Task** | Une seule instruction utilisant un module |
| **Playbook** | Fichier YAML contenant des plays (séries de tasks) |
| **Play** | Groupe de tasks appliquées à un ensemble d'hôtes |
| **Role** | Structure réutilisable de playbooks, tasks, templates et fichiers |
| **Handler** | Task déclenchée uniquement par un `notify` (ex: redémarrer un service après changement de config) |
| **Fact** | Variable récupérée automatiquement sur la machine cible (OS, RAM, IP...) |
| **Variable** | Donnée utilisée dans les tasks (définie dans le playbook, l'inventaire, ou un fichier vars) |

## Architecture

```
Machine contrôlante (Ansible)
        │
        │ SSH (sans agent)
        ▼
┌─────────────────┐
│  Machine cible  │
│  (python + ssh) │
└─────────────────┘
```

- **Sans agent** : pas de daemon à installer sur les cibles, juste Python et SSH
- **Push** : Ansible pousse les instructions vers les cibles (contrairement à Puppet/Chef qui pull)
- **Idempotent** : exécuter deux fois la même task produit le même résultat (pas d'effet de bord)

## Inventaire

### Inventaire simple (INI)

```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20 ansible_user=admin
```

### Inventaire YAML

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
          ansible_user: admin
```

### Inventaire dynamique

Un script qui retourne la liste des hôtes depuis une source externe (AWS, Azure, VMware, CMDB...). Utile pour les infrastructures cloud.

## Playbook

Structure de base d'un playbook :

```yaml
- name: Installer et configurer nginx
  hosts: webservers          # Cible : groupe d'hôtes
  become: yes                # Exécuter en sudo
  vars:                      # Variables locales au play
    nginx_port: 80

  tasks:
    - name: Installer nginx
      apt:
        name: nginx
        state: present
      notify: Redémarrer nginx    # Déclenche le handler si changement

    - name: Copier la configuration
      template:
        src: nginx.conf.j2        # Template Jinja2
        dest: /etc/nginx/nginx.conf
      notify: Redémarrer nginx

  handlers:                   # Handlers = tasks déclenchées par notify
    - name: Redémarrer nginx
      service:
        name: nginx
        state: restarted
```

## Modules essentiels

| Module | Utilité |
|--------|---------|
| `apt` / `yum` | Gestion de paquets |
| `copy` | Copier un fichier sur la cible |
| `template` | Appliquer un template Jinja2 |
| `file` | Créer/supprimer des fichiers/dossiers |
| `service` | Démarrer/arrêter/restart un service |
| `user` | Gérer les utilisateurs système |
| `group` | Gérer les groupes |
| `command` / `shell` | Exécuter une commande brute |
| `git` | Cloner/metjour un dépôt Git |
| `cron` | Gérer les tâches cron |
| `lineinfile` | Modifier une ligne dans un fichier |
| `blockinfile` | Insérer un bloc de lignes dans un fichier |
| `docker_container` | Gérer des conteneurs Docker |
| `docker_image` | Gérer des images Docker |
| `wait_for` | Attendre qu'une condition soit remplie (port ouvert, fichier existant...) |

## Rôles

Structure standardisée pour réutiliser du code Ansible :

```
roles/
  nginx/
    ├── tasks/
    │   └── main.yml         # Tasks du rôle
    ├── handlers/
    │   └── main.yml         # Handlers du rôle
    ├── templates/
    │   └── nginx.conf.j2    # Templates Jinja2
    ├── files/
    │   └── index.html       # Fichiers statiques
    ├── vars/
    │   └── main.yml         # Variables du rôle
    ├── defaults/
    │   └── main.yml         # Variables par défaut (écrasables)
    └── meta/
        └── main.yml         # Métadonnées (dépendances)
```

Utilisation dans un playbook :

```yaml
- hosts: webservers
  become: yes
  roles:
    - nginx
    - { role: common, tags: ['setup'] }
```

## Vault (gestion des secrets)

Ansible Vault chiffre les fichiers sensibles :

```bash
# Chiffrer un fichier
ansible-vault create secrets.yml

# Éditer un fichier chiffré
ansible-vault edit secrets.yml

# Lancer un playbook avec un fichier chiffré
ansible-playbook site.yml --ask-vault-pass

# Utiliser un fichier de mot de passe
ansible-playbook site.yml --vault-password-file=.vault_pass
```

## Jinja2 (templates)

Les templates permettent de dynamiciser les fichiers de configuration :

```jinja2
# nginx.conf.j2
server {
    listen {{ nginx_port }};
    server_name {{ ansible_fqdn }};

    {% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate /etc/ssl/{{ ansible_fqdn }}.crt;
    {% endif %}
}
```

Variables disponibles :
- `{{ variable }}` — variable définie dans le playbook/inventaire
- `{{ ansible_fqdn }}` — fait (fact) de la machine cible
- `{% if %}` / `{% for %}` — logique conditionnelle et boucles
