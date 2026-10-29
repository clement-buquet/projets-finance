# Mémo technique

## Début de session

**Pull en arrivant, push en partant.**

```
cd $env:USERPROFILE\projets-finance
.\.venv\Scripts\Activate.ps1
git pull
jupyter lab
```

- `git pull` d'abord : récupère ce qui a été poussé depuis un autre PC. Sans ça, on travaille sur une version en retard et le `push` du soir sera refusé.
- Si Windows bloque `jupyter.exe` (message de sécurité, programme bloqué) : lancer `python -m jupyterlab` à la place. Même résultat, mais Python lance JupyterLab lui-même, sans passer par l'exécutable bloqué.
- Réflexe à l'ouverture d'un notebook : **Run → Run All Cells**. Nouvelle session = kernel vide, aucune variable n'existe tant que les cellules n'ont pas été relancées.

## Fin de session (sauvegarde)

```
git add .
git commit -m "Ce que j'ai fait"
git push
```

Pas de push = pas de sauvegarde.

## Git : commandes utiles

- `git pull` : récupérer la dernière version depuis GitHub (en début de séance)
- `git status` : voir ce qui a changé / ce qui est prêt à partir
- `git log --oneline` : historique des commits
- `gh repo view --web` : ouvrir le dépôt sur GitHub

## Récupérer le projet sur un nouveau PC

```
winget install --id Git.Git -e --source winget
winget install --id GitHub.cli -e --source winget
winget install --id Python.Python.3.13 -e --source winget
gh auth login
git config --global user.name "Clément Buquet"
git config --global user.email "cbuquet.finance@gmail.com"
git config --global init.defaultBranch main
gh repo clone clement-buquet/projets-finance
cd projets-finance
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

## Ajouter une bibliothèque

1. L'ajouter dans `requirements.txt`
2. `python -m pip install -r requirements.txt`
3. Commit + push

## Pièges rencontrés

- Après une installation (winget), fermer et rouvrir PowerShell, sinon la commande n'est pas reconnue
- Ne pas travailler dans un terminal admin (`C:\Windows\system32`)
- Utiliser `py` (hors venv) ou `python` (venv activé), jamais `python` hors venv : conflit Microsoft Store
- Erreur "exécution de scripts désactivée" : `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`
- `jupyter.exe` bloqué par Windows : `python -m jupyterlab` (venv activé)
- Notepad en mode "Formaté" ajoute des `\` invisibles dans les .md : toujours éditer en mode Syntaxe
- Terminal bloqué sur `>>` : Ctrl + C
- Flèche du haut : rappeler la commande précédente
- PowerShell de Jupyter crashé : ne pas fermer l'onglet, relancer Jupyter, Ctrl + S si l'onglet se reconnecte, puis Run → Run All Cells
- Ne jamais ouvrir le même notebook dans deux onglets (risque d'écrasement)
- **Résultats affichés anciens** : un notebook enregistre les sorties de la dernière exécution de chaque cellule, pas l'état actuel du code. Une sortie peut dater d'avant une modification, ou venir d'une cellule exécutée dans le désordre (une variable utilisée avant la cellule qui la crée). Avant de recopier un chiffre ou de conclure : Kernel → Restart Kernel and Run All Cells, puis relire.
