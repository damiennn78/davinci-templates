---
description: Commit toutes les modifications et push vers GitHub (origin/main)
---

Commit et push les modifications du projet vers GitHub.

Le message de commit est : $ARGUMENTS
Si $ARGUMENTS est vide, génère un message de commit court et descriptif basé sur les fichiers modifiés (ex: "Update TITRE_TEST template", "Fix afficher logo control", etc.)

Exécute dans l'ordre :

1. Vérifie qu'on est bien dans un repo git (`git status`). Si ce n'est pas un repo git, arrête et signale l'erreur.
2. Affiche les fichiers modifiés/ajoutés/supprimés avec `git status --short`.
3. Stage tout avec `git add .`
4. Commit avec le message fourni ou généré :
   ```powershell
   $msg = "$ARGUMENTS"
   if (-not $msg.Trim()) { $msg = "Update templates" }
   git commit -m $msg
   ```
5. Push vers origin/main :
   ```powershell
   git push origin main
   ```
6. Confirme avec le hash du commit et l'URL du repo.
