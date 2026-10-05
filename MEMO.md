# Mémo technique



## Début de session


Nouvelle session = kernel vide : Run → Run All Cells pour recharger les variables.


```

cd $env:USERPROFILEprojets-finance

..venvScriptsActivate.ps1

jupyter lab

```



## Fin de session (sauvegarde)



```

git add .

git commit -m "Ce que j'ai fait"

git push

```



Pas de push = pas de sauvegarde.



## Git : commandes utiles



- `git status` : voir ce qui a changé / ce qui est prêt à partir

- `git log --oneline` : historique des commits

- `gh repo view --web` : ouvrir le dépôt sur GitHub



## Récupérer le projet sur un nouveau PC



```

winget install --id Git.Git -e --source winget

winget install --id GitHub.cli -e --source winget

winget install --id Python.Python.3.13 -e --source winget

gh auth login

git config --global

gh repo clone clement-buquet/projets-finance

cd projets-finance

Set-ExecutionPolicy -Scope CurrentUser RemoteSigned

py -m venv .venv

..venvScriptsActivate.ps1

python -m pip install -r requirements.txt

```



## Ajouter une bibliothèque



1. L'ajouter dans `requirements.txt`

2. `python -m pip install -r requirements.txt`

3. Commit + push



## Pièges rencontrés



- Après une installation (winget), fermer et rouvrir PowerShell, sinon la commande n'est pas reconnue

- Ne pas travailler dans un terminal admin (`C:Windowssystem32`)

- Utiliser `py` (hors venv) ou `python` (venv activé), jamais `python` hors venv : conflit Microsoft Store

- Erreur "exécution de scripts désactivée" : `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

- Notepad en mode "Formaté" ajoute des `` invisibles dans les .md : toujours éditer en mode Syntaxe

- Terminal bloqué sur `>>` : Ctrl + C

- Flèche du haut : rappeler la commande précédente

- PowerShell de Jupyter crashé : ne pas fermer l'onglet, relancer Jupyter, Ctrl + S si l'onglet se reconnecte, puis Run → Run All Cells

- Ne jamais ouvrir le même notebook dans deux onglets (risque d'écrasement)

- Notepad : toujours ouvrir les .md en mode Syntaxe

