# Git — Concepts

## Les 3 zones de Git

```
Répertoire de travail (Working Directory)
        │
        │  git add
        ▼
Zone de préparation (Staging Area / Index)
        │
        │  git commit
        ▼
Historique (Repository / HEAD)
```

| Zone | Description |
|------|-------------|
| **Working Directory** | Tes fichiers tels qu'ils existent sur ton disque |
| **Staging Area** | Fichiers prêts à être commités (sélectionnés avec `git add`) |
| **Repository** | Historique complet des commits (`.git/`) |

## Le commit

Un commit est un **snapshot** (instantané) de tous les fichiers à un instant T. Chaque commit a :
- Un **SHA-1** unique (identifiant hash, ex: `a1b2c3d`)
- Un **message** décrivant le changement
- Un **pointeur vers le commit parent** (sauf le premier commit)
- L'**auteur** et la **date**

## Branches

Une branche est un **pointeur mobile** vers un commit. `main` (ou `master`) est la branche par défaut.

```
main:    A → B → C → D
                  \
feature:           E → F
```

- Créer une branche = créer un pointeur vers le commit courant
- Fusionner une branche = combiner les historiques
- Supprimer une branche = supprimer le pointeur (les commits restent tant qu'ils sont atteignables)

## Merge vs Rebase

### Merge

```bash
git merge feature
```

Crée un **commit de fusion** qui combine les deux branches. L'historique est conservé tel quel.

```
main:    A → B → C ──→ M (merge commit)
                  \   /
feature:           E → F
```

### Rebase

```bash
git merge feature
```

Rejoue les commits de la branche `feature` par-dessus `main`. L'historique est linéaire.

```
main:    A → B → C → E' → F'
                  (commits re-joués)
```

| | Merge | Rebase |
|---|---|---|
| Historique | Conserve la structure réelle | Linéaire, propre |
| Commits de fusion | Oui (commit `M`) | Non |
| Sécurité | Toujours sûr | Ne jamais rebase une branche publique |
| Utilisation | Intégrer une feature dans main | Mettre à jour sa feature depuis main |

## HEAD

`HEAD` est un pointeur vers le commit courant (la branche sur laquelle tu es).

- `HEAD` → pointe vers la branche actuelle
- `HEAD~1` → le commit parent (un en arrière)
- `HEAD~3` → 3 commits en arrière
- **Detached HEAD** : `HEAD` pointe directement vers un commit (pas vers une branche) — dangereux car les nouveaux commits ne seront sur aucune branche

## Le stash

Le stash met de côté les modifications en cours sans commit :

```
Working Directory → Stash (pile LIFO)
```

- Utile quand tu dois switcher de branche rapidement
- Les stashs s'empilent (dernier stash = premier sorti)

## Les tags

Un tag est un **pointeur immobile** vers un commit (contrairement aux branches qui bougent).

- **Lightweight tag** : juste un nom qui pointe vers un commit
- **Annotated tag** : tag avec message, auteur, date (recommandé pour les releases)

## Les hooks

Scripts déclenchés automatiquement par Git à différents moments :

| Hook | Déclenché quand |
|------|----------------|
| `pre-commit` | Avant de finaliser un commit |
| `commit-msg` | Après avoir tapé le message de commit |
| `pre-push` | Avant de pousser vers un remote |
| `post-merge` | Après un merge réussi |

Les hooks se trouvent dans `.git/hooks/`. Exemples d'usage :
- `pre-commit` : lancer le linter/formatter automatiquement
- `commit-msg` : vérifier le format du message de commit

## Les remotes

Un remote est un dépôt distant (GitHub, GitLab...).

- `origin` → remote par défaut (le dépôt d'origine)
- `upstream` → convention pour le dépôt original d'un fork

```bash
# Lister les remotes
git remote -v

# Ajouter un remote
git remote add upstream https://github.com/original/repo.git
```

## `.gitignore`

Fichier qui indique à Git quels fichiers ignorer :

```gitignore
# Dépendances
node_modules/
vendor/

# Build
dist/
build/
*.o

# Secrets
.env
*.pem
*.key

# OS
.DS_Store
Thumbs.db

# Logs
*.log
```

- Un fichier déjà tracké ne sera pas ignoré par `.gitignore`
- Pour l'arrêter : `git rm --cached fichier.txt`
