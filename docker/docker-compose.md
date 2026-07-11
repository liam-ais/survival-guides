# Docker Compose

## Structure d'un `compose.yml`

```yaml
services:
  web:
    build: ./web              # Build from Dockerfile
    image: nginx:1.25         # OU pull image
    ports:
      - "8080:80"             # host:container
    volumes:
      - ./html:/usr/share/nginx/html  # Bind mount
      - appdata:/data                  # Named volume
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    networks:
      - frontend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 5

volumes:
  pgdata:
  appdata:

networks:
  frontend:
```

## Commandes

```bash
# Démarrer les services en arrière-plan
docker compose up -d
# -d → mode détaché (pas bloquant dans le terminal)

# Démarrer avec rebuild forcé des images
docker compose up -d --build
# --build → force le rebuild des images même si le cache existe

# Arrêter et supprimer les conteneurs
docker compose down

# Arrêter, supprimer les conteneurs ET les volumes
docker compose down -v
# -v → supprime aussi les volumes déclarés dans le compose

# Lister les services et leur état
docker compose ps

# Afficher les logs de tous les services (mode suivant)
docker compose logs -f

# Afficher les logs d'un service spécifique
docker compose logs -f web

# Exécuter une commande dans un service
docker compose exec web sh
# exec → exécute dans le conteneur EN COURS (pas un nouveau conteneur)

# Redémarrer un service
docker compose restart web

# Mettre à l'échelle un service (3 instances)
docker compose up -d --scale worker=3
# --worker=3 → lance 3 conteneurs du service worker
```

## Points clés

- `depends_on` avec `condition: service_healthy` attend le healthcheck, pas juste le démarrage du conteneur
- Les services se trouvent entre eux par **nom de service** (ex: `db`, `web`), pas par IP
- Les variables `${VAR}` sont lues depuis le fichier `.env` ou l'environnement hôte
- `docker compose down` ne supprime PAS les volumes par défaut (il faut `-v`)
- `docker compose up --build` force le rebuild des images même si le cache est valide
