# Ansible — Templates de playbook

## Installation et configuration d'un service

```yaml
- name: Installer et configurer nginx
  hosts: webservers
  become: yes
  vars:
    nginx_port: 80
    server_name: "example.com"

  tasks:
    - name: Installer nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
      tags: install

    - name: Copier la configuration nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: Redémarrer nginx
      tags: config

    - name: Démarrer et activer nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Ouvrir le pare-feu
      ufw:
        rule: allow
        port: "{{ nginx_port }}"
        proto: tcp

  handlers:
    - name: Redémarrer nginx
      service:
        name: nginx
        state: restarted
```

## Déploiement d'une application

```yaml
- name: Déployer l'application
  hosts: appservers
  become: yes
  vars:
    app_repo: "https://github.com/monorg/monapp.git"
    app_dir: "/opt/monapp"
    app_user: "appuser"

  tasks:
    - name: Créer l'utilisateur applicatif
      user:
        name: "{{ app_user }}"
        system: yes
        shell: /bin/bash
        home: "{{ app_dir }}"

    - name: Cloner le dépôt
      git:
        repo: "{{ app_repo }}"
        dest: "{{ app_dir }}"
        version: main
      become_user: "{{ app_user }}"
      register: git_result

    - name: Installer les dépendances
      npm:
        path: "{{ app_dir }}"
      become_user: "{{ app_user }}"
      when: git_result.changed

    - name: Créer le service systemd
      template:
        src: app.service.j2
        dest: /etc/systemd/system/monapp.service
      notify: Redémarrer monapp

  handlers:
    - name: Redémarrer monapp
      systemd:
        name: monapp
        state: restarted
        daemon_reload: yes
```

## Gestion des utilisateurs et sécurité

```yaml
- name: Sécuriser les serveurs
  hosts: all
  become: yes

  tasks:
    - name: Créer les utilisateurs
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop:
        - { name: "deploy", groups: "sudo" }
        - { name: "admin", groups: "sudo" }

    - name: Configurer SSH
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: '^#?PermitRootLogin', line: 'PermitRootLogin no' }
        - { regexp: '^#?PasswordAuthentication', line: 'PasswordAuthentication no' }
        - { regexp: '^#?Port', line: 'Port 2222' }
      notify: Redémarrer SSH

    - name: Installer et configurer UFW
      ufw:
        state: enabled
        policy: deny

    - name: Autoriser SSH
      ufw:
        rule: allow
        port: "2222"
        proto: tcp

  handlers:
    - name: Redémarrer SSH
      service:
        name: sshd
        state: restarted
```

## Structure de fichier `requirements.yml` pour Galaxy

```yaml
# requirements.yml
roles:
  - name: geerlingguy.nginx
    version: "3.1.0"
  - name: geerlingguy.docker
    version: "6.1.0"

collections:
  - name: community.general
    version: ">=5.0.0"
  - name: community.docker
```
