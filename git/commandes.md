# Git — Commandes

## Configuration

```bash
# Définir le nom et email (nécessaire pour les commits)
git config --global user.name "Mon Nom"
git config --global user.email "mon@email.com"

# Définir l'éditeur par défaut
git config --global core.editor "code --wait"

# Définir la branche par défaut pour git push
git config --global push.autoSetupRemote true
# push.autoSetupRemote → configure automatiquement le tracking au premier push

# Afficher toute la config
git config --list
```

## Dépôt et initialization

```bash
# Créer un nouveau dépôt
git init
# Crée le dossier .git/ dans le répertoire courant

# Cloner un dépôt distant
git clone https://github.com/user/repo.git
# Clone le dépôt + configure le remote "origin"

# Cloner dans un dossier spécifique
git clone https://github.com/user/repo.git mon-dossier
```

## Ajouter et commit

```bash
# Ajouter un fichier au staging
git add fichier.txt
# Ajoute le fichier pour le prochain commit

# Ajouter tous les fichiers modifiés/nouveaux
git add .
# Le point = tous les fichiers du répertoire courant et sous-dossiers

# Ajouter tous les fichiers .js modifiés
git add "*.js"

# Voir l'état du staging
git status

# Voir les modifications en attente
git diff             # différences entre working directory et staging
git diff --staged    # différences entre staging et dernier commit

# Committer les fichiers staged
git commit -m "Ajouter la page d'accueil"
# -m → permet de passer le message directement en argument

# Committer avec un message multi-lignes
git commit
# Ouvre l'éditeur pour écrire un message détaillé

# Ajouter et committer en une seule commande (pour les fichiers déjà trackés)
git commit -am "Corriger le bug de login"
# -a → ajoute tous les fichiers modifiés (pas les nouveaux fichiers)
# -m → message direct

# Modifier le dernier commit (message + fichiers)
git commit --amend -m "Nouveau message"
# Réécrit le dernier commit (utile avant de push)
```

## Branches

```bash
# Lister les branches locales
git branch

# Lister toutes les branches (locales + distantes)
git branch -a

# Créer une nouvelle branche
git branch feature/login
# Crée la branche mais ne switch pas dessus

# Créer et switcher sur une nouvelle branche
git checkout -b feature/login
# -b → crée la branche + switch automatiquement

# Switcher de branche
git checkout main
# OU (plus récent)
git switch main

# Renommer une branche
git branch -m ancien-nom nouveau-nom

# Supprimer une branche (locale)
git branch -d feature/login
# -d → supprime seulement si la branche est fusionnée

# Forcer la suppression d'une branche
git branch -D feature/login
# -D → force la suppression même non fusionnée

# Supprimer une branche distante
git push origin --delete feature/login
```

## Merge et rebase

```bash
# Fusionner une branche dans la branche courante
git merge feature/login
# Crée un commit de fusion (merge commit) si les historiques ont divergé

# Fusionner sans commit de fusion (fast-forward)
git merge --ff-only feature/login
# Ne fonctionne que si la branche courante est un ancestor direct

# Rebase la branche courante sur main
git rebase main
# Rejoue tes commits au-dessus de main (historique linéaire)

# Continuer après un conflit de rebase
git rebase --continue
# Après avoir résolu les conflits, continue le rebase

# Annuler un rebase en cours
git rebase --abort

# Annuler un merge en cours
git merge --abort
```

## Conflits

```bash
# Les conflits sont marqués dans les fichiers :
<<<<<<< HEAD
contenu de la branche courante
=======
contenu de l'autre branche
>>>>>>> feature

# Après avoir résolu les conflits (édité les fichiers) :
git add fichier.txt
git commit   # termine le merge/rebase
```

## Push et pull

```bash
# Pousser les commits vers le remote
git push
# Pousse la branche courante vers le remote (tracking configuré)

# Pousser en créant la branche distante
git push -u origin feature/login
# -u → configure le tracking pour les prochains push/pull

# Tirer les changements du remote (fetch + merge)
git pull
# Récupère les changements distants et les fusionne dans la branche courante

# Récupérer sans fusionner (fetch)
git fetch
# Met à jour les refs distants sans toucher au working directory

# Voir les différences avant de merge
git fetch origin
git log HEAD..origin/main
# Affiche les commits qui sont sur origin/main mais pas chez toi
```

## Historique

```bash
# Afficher l'historique
git log

# Historique compact (1 ligne par commit)
git log --oneline

# Historique avec graphique
git log --oneline --graph --all
# --graph → affiche les branches et merges en ASCII art
# --all   → affiche toutes les branches

# Afficher les changements d'un commit
git show a1b2c3d

# Chercher un commit par message
git log --grep="bug fix"

# Chercher un commit qui a modifié un fichier
git log -- path/fichier.txt
```

## Stash

```bash
# Mettre de côté les modifications en cours
git stash
# Sauvegarde les changements et nettoie le working directory

# Mettre de côté avec un nom
git stash push -m "WIP: feature login"

# Lister les stashs
git stash list

# Restaurer le dernier stash
git stash pop
# Applique le stash ET le supprime de la pile

# Restaurer sans supprimer
git stash apply

# Supprimer un stash
git stash drop stash@{0}

# Vider tous les stashs
git stash clear
```

## Annulation

```bash
# Annuler un fichier (revenir au dernier commit)
git checkout -- fichier.txt
# OU (plus récent)
git restore fichier.txt

# Retirer un fichier du staging (garder les modifications)
git reset HEAD fichier.txt
# OU (plus récent)
git restore --staged fichier.txt

# Réinitialiser à un commit spécifique (garde les modifications)
git reset --soft HEAD~1
# --soft → annule le dernier commit, garde les fichiers staged

# Réinitialiser à un commit spécifique (garde les fichiers non-stagés)
git reset HEAD~1
# (sans --soft ni --hard) → annule le commit, garde les modifications dans le working directory

# Réinitialiser complètement (SUPPRIME les modifications !)
git reset --hard HEAD~1
# --hard → supprime TOUT (commit + modifications). IRRÉVERSIBLE.

# Annuler un commit déjà poussé (créé un commit inverse)
git revert a1b2c3d
# Crée un NOUVEAU commit qui annule les changements du commit ciblé
# Sûr car ne réécris pas l'historique
```

## Tags

```bash
# Créer un tag léger
git tag v1.0.0
# Tag le commit courant

# Créer un tag annoté (recommandé)
git tag -a v1.0.0 -m "Release v1.0.0"
# -a → annotated tag avec message

# Lister les tags
git tag

# Pousser un tag
git push origin v1.0.0

# Pousser tous les tags
git push origin --tags

# Supprimer un tag
git tag -d v1.0.0
# Supprime le tag local

git push origin --delete v1.0.0
# Supprime le tag distant
```
