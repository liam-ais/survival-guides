# Ansible — Commandes

## Ad-hoc (exécution rapide sans playbook)

```bash
# Pinguer tous les hôtes de l'inventaire
ansible all -i inventory.ini -m ping
# -i inventory.ini → fichier d'inventaire à utiliser
# -m ping          → module à exécuter (ping = test de connectivité)

# Exécuter une commande sur tous les hôtes
ansible webservers -i inventory.ini -m command -a "uptime"
# webservers       → groupe d'hôtes cible
# -m command       → module "command" (exécution de commande)
# -a "uptime"      → arguments passés au module

# Installer un paquet sur les dbservers
ansible dbservers -i inventory.ini -m apt -a "name=htop state=present" --become
# -m apt            → module de gestion de paquets Debian/Ubuntu
# --become          → exécuter en sudo (apt nécessite root)

# Copier un fichier vers les hôtes
ansible webservers -i inventory.ini -m copy -a "src=./motd dest=/etc/motd mode=0644" --become
# -m copy           → module de copie de fichiers
# src=./motd        → fichier source sur la machine contrôlante
# dest=/etc/motd    → chemin destination sur la machine cible
# mode=0644         → permissions du fichier

# Redémarrer un service
ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become
# -m service        → module de gestion de services systemd
# state=restarted   → redémarrer le service

# Lister les hôtes d'un groupe
ansible webservers -i inventory.ini --list-hosts
```

## Playbooks

```bash
# Exécuter un playbook
ansible-playbook site.yml -i inventory.ini
# site.yml          → fichier playbook à exécuter

# Exécuter avec sudo
ansible-playbook site.yml -i inventory.ini --become

# Exécuter avec un mot de passe sudo
ansible-playbook site.yml -i inventory.ini --become --ask-become-pass
# --ask-become-pass → demande le mot de passe sudo au lancement

# Exécuter un play spécifique (tag)
ansible-playbook site.yml -i inventory.ini --tags "install"
# --tags "install"  → exécute uniquement les tasks marquées avec ce tag

# Exclure un tag
ansible-playbook site.yml -i inventory.ini --skip-tags "debug"

# Mode sec (dry run, pas de changements)
ansible-playbook site.yml -i inventory.ini --check
# --check           → simule les changements sans rien appliquer

# Afficher les différences
ansible-playbook site.yml -i inventory.ini --diff
# --diff            → montre le contenu avant/après modification

# Limiter à un hôte spécifique
ansible-playbook site.yml -i inventory.ini --limit web1
# --limit web1      → n'exécute que sur l'hôte web1

# Mode verbose (débogage)
ansible-playbook site.yml -i inventory.ini -vv
# -vv              → verbose niveau 2 (utile pour comprendre les erreurs)
```

## Inventaire

```bash
# Tester la connectivité de tous les hôtes
ansible all -i inventory.ini -m ping

# Afficher les facts (informations système) d'un hôte
ansible web1 -i inventory.ini -m setup
# -m setup         → récupère les facts (OS, RAM, CPU, IP...)

# Filtrer les facts
ansible web1 -i inventory.ini -m setup -a "filter=ansible_distribution"
# filter=ansible_distribution → affiche uniquement la distribution OS

# Tester un groupe spécifique
ansible webservers -i inventory.ini -m ping
```

## Rôles et Galaxy

```bash
# Installer un rôle depuis Ansible Galaxy
ansible-galaxy install geerlingguy.nginx
# ansible-galaxy   → gestionnaire de rôles/community
# geerlingguy.nginx → rôle populaire pour nginx

# Installer depuis un fichier requirements
ansible-galaxy install -r requirements.yml
# requirements.yml → liste de rôles à installer

# Créer la structure d'un rôle
ansible-galaxy role init mon_role
# Crée la structure de dossiers standard (tasks/, handlers/, templates/...)

# Lister les rôles installés
ansible-galaxy list
```

## Vault

```bash
# Créer un fichier chiffré
ansible-vault create secrets.yml
# Demande un mot de passe → le fichier est chiffré

# Éditer un fichier chiffré
ansible-vault edit secrets.yml

# Chiffrer un fichier existant
ansible-vault encrypt vars/secrets.yml

# Déchiffrer un fichier
ansible-vault decrypt vars/secrets.yml

# Afficher le contenu déchiffré (sans éditer)
ansible-vault view secrets.yml

# Lancer un playbook avec Vault
ansible-playbook site.yml --ask-vault-pass
# --ask-vault-pass → demande le mot de passe Vault au lancement

# Utiliser un fichier de mot de passe (CI/CD)
ansible-playbook site.yml --vault-password-file=.vault_pass
# .vault_pass → fichier contenant le mot de passe Vault en clair
```
