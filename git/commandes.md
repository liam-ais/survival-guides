# Git — Commandes

Cette partie regroupe les commandes vues pendant mon apprentissage de Git. Elles sont classées par utilisation afin de les retrouver plus facilement.

Les explications des différentes notions se trouvent dans [concepts.md](concepts.md). Les valeurs entre chevrons, comme `<url>`, sont à remplacer sans conserver les chevrons.

## 1. Initialiser un dépôt et enregistrer des modifications

| Commande | Rôle |
| --- | --- |
| `git init` | Initialiser un dépôt local. |
| `git status` | Vérifier l’état des fichiers. |
| `git add .` | Préparer les ajouts, modifications et suppressions sous le répertoire courant. |
| `git commit -m "message"` | Enregistrer un commit avec le message indiqué par l’option `-m`. |
| `git log` | Consulter l’historique des commits. |

Pour enregistrer les modifications, nous commençons par les préparer, puis nous créons un commit :

```bash
git add .
git commit -m "Mise à jour de la documentation"
```

## 2. Utiliser un dépôt distant

| Commande | Rôle |
| --- | --- |
| `git clone <url>` | Copier un dépôt distant en local. |
| `git remote add origin <url>` | Ajouter un dépôt distant nommé `origin`. |
| `git remote -v` | Afficher les dépôts distants et leurs URL. |
| `git push` | Envoyer les commits selon la configuration de la branche. |
| `git push -u origin feature` | Envoyer la branche `feature` sur `origin` et configurer son suivi distant avec `-u`. |
| `git pull` | Récupérer et intégrer les changements selon la configuration. |
| `git pull origin main` | Récupérer `main` depuis `origin` et l’intégrer dans la branche courante. |
| `git fetch` | Récupérer les changements distants sans les intégrer à la branche courante. |

Après un clone, `origin` est déjà configuré. Il n’est donc pas nécessaire de l’ajouter à nouveau.

Pour mettre à jour la branche locale `main`, nous nous plaçons d’abord dessus :

```bash
git checkout main
git pull origin main
```

## 3. Créer et changer de branche

| Commande | Rôle |
| --- | --- |
| `git branch` | Lister les branches locales. |
| `git branch feature` | Créer la branche `feature` sans changer de branche. |
| `git checkout feature` | Se placer sur `feature`. |
| `git checkout -b feature` | Créer `feature` et se placer dessus avec l’option `-b`. |
| `git switch feature` | Se placer sur `feature`, avec une commande dédiée au changement de branche. |
| `git switch -c feature` | Créer `feature` et se placer dessus avec l’option `-c`. |
| `git branch -d feature` | Supprimer une branche locale fusionnée, selon les vérifications effectuées par Git. |

Pour créer une branche et commencer à travailler dessus, nous pouvons choisir l’une de ces deux commandes :

```bash
git checkout -b feature
```

Ou :

```bash
git switch -c feature
```

Il n’est pas nécessaire d’exécuter les deux : elles répondent ici au même besoin.

## 4. Fusionner une branche

Pour fusionner `feature` dans `main` :

```bash
git checkout main
git merge feature
```

La première commande peut aussi être remplacée par `git switch main`.

En cas de conflit, nous corrigeons les fichiers concernés puis nous terminons la fusion :

```bash
git add .
git commit
```

## 5. Rebase et rebase interactif

Pour rejouer les commits de `feature` sur la version locale de `main` :

```bash
git checkout feature
git rebase main
```

Pour modifier les trois derniers commits avec un rebase interactif :

```bash
git rebase -i HEAD~3
```

- `-i` ouvre le mode interactif.
- `HEAD~3` désigne le commit situé trois générations avant `HEAD` en suivant le premier parent ; dans un historique simple, cela permet de reprendre les trois derniers commits.
- Dans la liste, `pick`, `reword`, `squash` et `drop` permettent de choisir ce que nous souhaitons faire avec les commits.

Attention : éviter de rebaser des commits déjà partagés sans se coordonner avec les autres contributeurs.

## 6. Récupérer un commit avec cherry-pick

Pour appliquer les modifications d’un commit précis sur `main` :

```bash
git checkout main
git cherry-pick <commit-hash>
```

`<commit-hash>` correspond à l’identifiant du commit, que nous pouvons retrouver avec `git log`.

## 7. Revenir en arrière avec reset

Dans les commandes suivantes, `HEAD~1` désigne le commit précédent.

| Commande | Résultat |
| --- | --- |
| `git reset --soft HEAD~1` | Retirer le dernier commit de la branche et garder les modifications préparées. |
| `git reset --mixed HEAD~1` | Retirer le dernier commit de la branche et garder les modifications hors de la Staging Area. |
| `git reset --hard HEAD~1` | Revenir au commit précédent en remettant l’index et les fichiers suivis dans son état. |

Attention : `--hard` peut supprimer les modifications locales concernées. Cette option est à utiliser seulement si nous sommes sûrs de ne pas vouloir les conserver.

## 8. Annuler un commit avec revert

```bash
git revert <commit-hash>
```

Cette commande crée un nouveau commit qui annule les modifications du commit ciblé. Elle est à privilégier pour annuler un commit déjà partagé.

## 9. Mettre les modifications de côté

| Commande | Rôle |
| --- | --- |
| `git stash` | Mettre temporairement de côté les modifications des fichiers suivis. |
| `git stash list` | Afficher les stash disponibles. |
| `git stash pop` | Restaurer le dernier stash et le retirer si l’application réussit. |
| `git stash apply` | Restaurer le dernier stash sans le retirer. |
| `git stash -u` | Inclure aussi les nouveaux fichiers non suivis, hors fichiers ignorés. |

## 10. Retrouver les anciennes positions de HEAD

```bash
git reflog
```

Cette commande affiche les déplacements locaux de `HEAD`. Elle peut être utile pour retrouver un commit après un reset ou un rebase, tant qu’il est encore disponible dans le dépôt.

[Retour au README](README.md) · [Consulter les concepts](concepts.md)
