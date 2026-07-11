# Docker — Commandes

## Images

```bash
# Construire une image à partir d'un Dockerfile
docker build -t monimage:tag .
# -t monimage:tag  → nomme et tagge l'image (facilite la recherche)
# .                → chemin du contexte de build (là où se trouve le Dockerfile)

# Lister les images locales
docker images

# Supprimer une image
docker rmi monimage:tag

# Supprimer toutes les images inutilisées
docker image prune -a
# -a → supprime aussi les images sans conteneur (pas seulement les orphelines)

# Télécharger une image depuis un registre (Docker Hub par défaut)
docker pull nginx:1.25
# nginx:1.25 → image officielle nginx, version 1.25

# Taguer une image pour un push vers un registre
docker tag monimage:latest monrepo/monimage:v1
# monrepo/monimage:v1 → chemin du registre + nom + tag

# Pousser une image vers un registre
docker push monrepo/monimage:v1
```

## Conteneurs

```bash
# Lancer un conteneur
docker run -d --name monconteneur -p 8080:80 nginx
# -d                → mode détaché (lance en arrière-plan, pas bloquant dans le terminal)
# --name monconteneur → donne un nom au conteneur pour le retrouver facilement
# -p 8080:80        → expose le port 80 du conteneur sur le port 8080 de la machine hôte
# nginx             → l'image à utiliser

# Lancer un conteneur interactivement avec un shell
docker run -it alpine sh
# -i → interactif (garde stdin ouvert)
# -t → alloue un TTY (pseudo-terminal, permet la frappe au clavier)
# sh → commande exécutée dans le conteneur

# Lancer avec un volume
docker run -v $(pwd)/data:/app/data monimage
# -v $(pwd)/data:/app/data → bind mount : le dossier ./data de la machine est monté dans /app/data du conteneur

# Lancer avec des variables d'environnement
docker run -e MY_VAR=value -e DB_HOST=localhost monimage
# -e MY_VAR=value → passe une variable d'environnement au conteneur

# Lancer avec un fichier .env
docker run --env-file .env monimage
# --env-file .env → lit toutes les variables depuis le fichier .env

# Lister les conteneurs en cours d'exécution
docker ps

# Lister tous les conteneurs (y compris arrêtés)
docker ps -a

# Arrêter un conteneur (SIGTERM, puis SIGKILL après 10s)
docker stop monconteneur

# Démarrer un conteneur arrêté
docker start monconteneur

# Redémarrer un conteneur
docker restart monconteneur

# Supprimer un conteneur arrêté
docker rm monconteneur

# Forcer la suppression d'un conteneur en cours d'exécution
docker rm -f monconteneur
# -f → force l'arrêt (SIGKILL) puis supprime

# Supprimer tous les conteneurs arrêtés
docker container prune
```

## Débogage

```bash
# Afficher les logs (mode suivant, se met à jour en temps réel)
docker logs -f monconteneur
# -f → follow, suit les nouvelles sorties en temps réel

# Afficher les 100 dernières lignes de logs
docker logs --tail 100 monconteneur

# Exécuter une commande dans un conteneur en cours d'exécution
docker exec -it monconteneur sh
# -it → interactif + TTY (comme pour docker run)

# Inspecter les détails d'un conteneur (IP, variables, mounts, etc.)
docker inspect monconteneur

# Afficher les statistiques en temps réel (CPU, RAM, réseau)
docker stats

# Afficher les processus en cours dans un conteneur
docker top monconteneur

# Copier un fichier du conteneur vers la machine hôte
docker cp monconteneur:/app/logs/error.log ./error.log
```

## Nettoyage

```bash
# Supprimer tous les conteneurs arrêtés, réseaux inutilisés, images orphelines
docker system prune

# Nettoyage complet (y compris les volumes !)
docker system prune -a --volumes
# -a        → supprime toutes les images inutilisées (pas seulement les orphelines)
# --volumes → supprime aussi les volumes inutilisés

# Afficher l'espace disque utilisé par Docker
docker system df
```

## Dockerfile

```bash
# Construire sans cache (fresh build complète)
docker build --no-cache -t monimage .
# --no-cache → ignore le cache des couches précédentes, force tout le rebuild

# Construire avec des arguments de build
docker build --build-arg VERSION=1.0 -t monimage .
# --build-arg VERSION=1.0 → passe une variable au Dockerfile (ARG VERSION)

# Afficher les couches d'une image
docker history monimage
```
