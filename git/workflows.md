# Git — Workflows

## Git Flow

Structure de branches pour les projets avec des releases versionnées :

```
main ──────────────────────────────────────────→
  │                                             ↑
  └── develop ──── E ──── F ──── G ────────────┘ (merge)
       │                ↑        ↑
       └── feature/A ───┘        │
                                  │
       └── feature/B ─────────────┘
```

| Branche | Rôle | Fusionnée dans |
|---------|------|----------------|
| `main` | Code stable, production | — |
| `develop` | Intégration des features | `main` (release) |
| `feature/*` | Nouvelles fonctionnalités | `develop` |
| `release/*` | Préparation d'une release | `main` + `develop` |
| `hotfix/*` | Corrections urgences production | `main` + `develop` |

**Quand l'utiliser :** projets avec cycles de release définis, équipes grandes.

## Trunk-Based Development

Tout le monde travaille sur `main` (ou une branche à très courte vie) :

```
main ── A ── B ── C ── D ── E ──→
      │       │       │
      └─feat──┘       │
              └─feat──┘
```

- Les branches de feature durent **quelques heures à quelques jours**
- Intégration continue fréquente
- Les features inachevées utilisent **feature flags**
- Requiert de bons tests et CI/CD

**Quand l'utiliser :** projets agiles, déploiement continu, petite équipe.

## GitHub Flow

Workflow simplifié centré sur les Pull Requests :

```
main ───────────────────────────────→
  ↑          ↑          ↑
  │ PR merge │ PR merge │
  │          │          │
  └── feature/A    └── feature/B
```

1. Créer une branche depuis `main`
2. Travailler sur la feature
3. Ouvrir une Pull Request
4. Revue de code + CI passe
5. Merge dans `main`
6. Déployer

**Quand l'utiliser :** la plupart des projets web, déploiement continu.

## Conventions de messages de commit

Format recommandé (Conventional Commits) :

```
<type>(<scope>): <description>

[corps optionnel]

[footer optionnel]
```

### Types

| Type | Description | Exemple |
|------|-------------|---------|
| `feat` | Nouvelle fonctionnalité | `feat(auth): ajouter login OAuth` |
| `fix` | Correction de bug | `fix(api): corriger le timeout` |
| `docs` | Documentation | `docs: ajouter le README` |
| `style` | Formatage (pas de changement logique) | `style: corriger l'indentation` |
| `refactor` | Refactorisation (pas de nouveau feat ni fix) | `refactor: extraire le service auth` |
| `test` | Ajout/correction de tests | `test: ajouter le test login` |
| `chore` | Tâches de maintenance | `chore: mettre à jour les dépendances` |
| `perf` | Amélioration de performance | `perf: optimiser la requête SQL` |
| `ci` | CI/CD | `ci: ajouter le pipeline GitHub Actions` |

### Bonnes pratiques

- **Description à l'impératif** : "ajouter" pas "ajouté"
- **Pas de majuscule en début** : "ajouter" pas "Ajouter"
- **Pas de point à la fin**
- **Max 72 caractères** sur la première ligne
- **Corps optionnel** pour expliquer le pourquoi (pas le quoi)

```bash
feat(auth): ajouter l'authentification OAuth2

Implémente le flow Authorization Code avec Google et GitHub.
Le token est stocké en cookie HTTP-only.

Closes #123
```

## Bonnes pratiques

- **Commit atomique** : un commit = un changement logique
- **Commit fréquent** : mieux vaut 10 petits commits qu'un gros
- **Ne jamais commit de secrets** (clés API, mots de passe, .env)
- **Utiliser un `.gitignore`** pour exclure les fichiers générés
- **Taguer les releases** (ex: `v1.0.0`)
- **Protéger la branche `main`** (reviews + CI obligatoires)
