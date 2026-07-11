# Docker — Concepts

## Image vs Container

| | Image | Conteneur |
|---|---|---|
| Définition | Plan en lecture seule | Instance en cours d'exécution d'une image |
| Stockage | Registre Docker (daemon) | Système de fichiers + état |
| Analogie | Classe (code) | Objet (instance) |

Une image est construite à partir d'un `Dockerfile`. Un conteneur est créé à partir d'une image (`docker run`).

## Couches d'images

Chaque instruction du Dockerfile crée une couche. Les couches sont cachées et empilées.
- Modifier une couche invalide toutes les couches au-dessus
- Placer les instructions qui changent rarement (apt install) avant celles qui changent souvent (COPY . .)

```dockerfile
# BON : couches ordonnées par fréquence de modification
FROM node:20-alpine
RUN apk add --no-cache curl        # change rarement
COPY package.json .                 # change parfois
RUN npm install                     # mis en cache si package.json inchangé
COPY . .                            # change souvent
```

---

## Mise en réseau

### Le réseau bridge (défaut)

Par défaut, Docker crée un réseau **bridge** pour chaque conteneur. Les conteneurs communiquent entre eux via une passerelle virtuelle.

- Chaque conteneur obtient une IP interne (ex: `172.17.0.2`)
- Les conteneurs se trouvent entre eux par leur **nom** ou **nom de service** (Compose), pas par IP
- Résolution DNS automatique par le daemon Docker

```bash
# Conteneur A peut atteindre conteneur B par son nom
ping conteneurB   # résolu en 172.17.0.3
```

### Le réseau host

Le conteneur partage directement la pile réseau de la machine hôte.
- Pas d'isolation réseau (le conteneur voit le même réseau que la machine)
- Utile quand les performances réseau sont critiques
- Le port `EXPOSE` du Dockerfile n'est plus nécessaire (tout est sur l'hôte)

```bash
docker run --network host nginx
# nginx écoute directement sur le port 80 de la machine
```

### Le réseau overlay

Réseau multi-hôtes, utilisé principalement avec **Docker Swarm** ou des plateformes comme Traefik.
- Permet la communication entre conteneurs sur des machines différentes
- Chiffrement des communications inter-nœuds

### Le réseau none

Aucun réseau. Le conteneur est complètement isolé.
- Utile pour des tâches batch qui n'ont pas besoin de réseau
- Ou pour des raisons de sécurité (conteneur sandboxé)

### Le port mapping

Mapping d'un port de la machine hôte vers un port du conteneur :

```
-p 8080:80
```

- `8080` → port sur la **machine hôte** (celui que tu utilises dans ton navigateur)
- `80` → port dans le **conteneur** (celui sur lequel l'application écoute)

```bash
# Le port 80 du conteneur nginx est exposé sur le port 8080 de la machine
docker run -p 8080:80 nginx

# Tu accèdes à nginx via http://localhost:8080
```

### Communication inter-conteneurs

Dans Docker Compose, les services se trouvent par leur **nom de service** :

```yaml
services:
  web:
    environment:
      DB_HOST: db    # le nom du service, pas une IP
  db:
    image: postgres
```

Le service `web` atteint `db` via le hostname `db` (résolu par le DNS interne de Docker).

---

## Stockage persistant vs éphémère

Les conteneurs sont **éphémères** : quand un conteneur est supprimé, son système de fichiers disparaît avec lui. Les **volumes** permettent de conserver les données au-delà de la vie du conteneur.

### Types de stockage

| Type | Description | Données persistentes ? |
|------|-------------|----------------------|
| **Named volume** | Géré par Docker, stocké dans `/var/lib/docker/volumes/` | Oui |
| **Bind mount** | Mappe un chemin de la machine hôte vers le conteneur | Oui (sur l'hôte) |
| **tmpfs** | En mémoire uniquement | Non |

### Quand utiliser quoi ?

| Cas d'usage | Type recommandé | Pourquoi |
|-------------|----------------|----------|
| Base de données (PostgreSQL, MySQL) | Named volume | Données critiques, Docker gère la persistance |
| Code source en développement | Bind mount | Modifier le code sur l'hôte, voir les changements dans le conteneur |
| Variables d'environnement secrètes | tmpfs | Jamais persisté sur disque |
| Fichiers de configuration | Bind mount | Éditer la config sur l'hôte sans rebuild |
| Cache, fichiers temporaires | tmpfs | Performances (RAM) + nettoyage automatique |

### Exemples

```bash
# Named volume : données BDD persistantes même après docker rm
docker run -v pgdata:/var/lib/postgresql/data postgres

# Bind mount : modifier le code source depuis l'hôte
docker run -v $(pwd)/src:/app/src node:20

# tmpfs : secrets en mémoire uniquement
docker run --tmpfs /run/secrets:mode=700 myapp
```

### Named volume vs Bind mount

| | Named volume | Bind mount |
|---|---|---|
| Gestion | Docker gère le stockage | Tu gères le chemin sur l'hôte |
| Portabilité | Fonctionne sur n'importe quel OS | Dépend du chemin absolu de l'hôte |
| Sécurité | Docker a le contrôle | Le conteneur peut modifier des fichiers de l'hôte |
| Backup | `docker volume` + copie du dossier | Copie directe du dossier |
| Cas d'usage | Données applicatives, BDD | Développement, config |

---

## Dockerfile — Instructions essentielles

```dockerfile
FROM node:20-alpine        # Image de base
WORKDIR /app               # Répertoire de travail
COPY package*.json ./      # Copier des fichiers dans l'image
RUN npm ci --production    # Exécuter une commande (cuit dans l'image)
EXPOSE 3000                # Documenter le port (ne l'expose PAS automatiquement)
ENV NODE_ENV=production    # Définir une variable d'environnement
USER node                  # Exécuter en tant que non-root
CMD ["node", "server.js"]  # Commande par défaut (peut être remplacée)
```

### CMD vs ENTRYPOINT

- `CMD` — Arguments par défaut, **remplaçables** : `docker run monimage <args>` remplace le CMD
- `ENTRYPOINT` — Commande fixe, le CMD devient ses arguments
- Exemple : `ENTRYPOINT ["python"]` + `CMD ["app.py"]` → exécute `python app.py`
- `docker run monimage test.py` → exécute `python test.py`

---

## Cycle de vie d'un conteneur

```
Créé → En cours → En pause → En cours → Arrêté → Supprimé
            ↓                           ↑
         Redémarrage ──────────────────┘
```

| État | Description |
|------|-------------|
| **Créé** | Alloué mais pas démarré |
| **En cours** | En exécution |
| **En pause** | Processus gelés (SIGSTOP) |
| **Arrêté** | Processus terminé (SIGTERM puis SIGKILL après timeout) |
| **Supprimé** | Disque nettoyé |

---

## Bonnes pratiques

- **Ne pas tourner en root** — utiliser l'instruction `USER`
- **Utiliser `.dockerignore`** — exclure node_modules, .git, etc.
- **Utiliser des images alpine** — empreinte réduite (quelques Mo vs centaines)
- **Un processus par conteneur** — ne pas mixer nginx + app dans le même conteneur
- **Builds multi-étapes** — séparer les dépendances de build du runtime
- **Pinner les versions** — `node:20.11-alpine` plutôt que `node:latest`
