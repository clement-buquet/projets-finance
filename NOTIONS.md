# Notions techniques — Finance & Python

Chaque notion : **définition** → **formule / code** → **à retenir**.

---

## 1. Données de marché

### Ticker
Code qui identifie un titre sur une place de cotation.
- `SAN.PA` = Sanofi sur Euronext **Pa**ris. Le suffixe indique la place (`.SW` = Suisse, pas de suffixe = US).
- **À retenir :** même entreprise, places différentes = tickers différents, devises différentes.

### DataFrame
Tableau pandas : des lignes (ici une par jour) et des colonnes (ici les prix).
- `san.head()` / `san.tail()` : 5 premières / dernières lignes
- `san.shape` : (nombre de lignes, nombre de colonnes)
- **À retenir :** toujours regarder ses données avant de calculer quoi que ce soit.

### Index
L'étiquette des lignes. Pour des cours de bourse, c'est la **date**.
- `san.index.min()` / `san.index.max()` : première et dernière date

### OHLCV
Les colonnes standard renvoyées par yfinance :
- **O**pen (ouverture), **H**igh (plus haut), **L**ow (plus bas), **C**lose (clôture), **V**olume (nombre de titres échangés)

### Cours de clôture ajusté
Prix de clôture corrigé des **dividendes** et des **splits** (division d'actions).
- yfinance l'applique par défaut (`auto_adjust=True`).
- **À retenir :** sans ajustement, le jour du versement d'un dividende, le prix baisse mécaniquement → on croirait à une perte qui n'en est pas une.

### Jours de cotation (252)
La bourse est fermée les week-ends et jours fériés → **~252 jours de cotation par an**, pas 365.
- 5 ans ≈ 1 260 à 1 280 lignes, pas 1 825.
- **À retenir :** 252 est le chiffre utilisé pour tout annualiser.

### NaN (valeur manquante)
*Not a Number* : case vide.
- `san.isna().sum()` : compte les valeurs manquantes par colonne
- `ffill()` (*forward fill*) : remplace un vide par la dernière valeur connue → « marché fermé, le prix n'a pas bougé »
- **À retenir :** plusieurs places = plusieurs calendriers de jours fériés = des NaN à traiter.

### ffill ou dropna : selon l'origine des NaN
Avant de traiter des NaN, se demander **pourquoi** ils sont là.
- **Jour férié sur une seule place** (ex. un ETF coté à Francfort, fermé un jour où Paris est ouvert) : le prix existe, il n'a simplement pas bougé → `prix.ffill()`
- **Pas de veille** (première ligne après `pct_change()`) : il n'y a rien à reporter → `.dropna()`
- **Titre pas encore coté** (NaN au début de l'historique) : `ffill()` ne peut rien remplir (aucune valeur avant), et `dropna()` coupe **toutes** les colonnes à la date du titre le plus récent
- Contrôle : `prix.isna().sum()` (combien) et `prix.notna().idxmax()` (première date disponible par colonne)
- Ordre type : `prix.ffill()` puis `prix.pct_change().dropna()`
- **À retenir :** `ffill` invente une valeur (« le prix n'a pas bougé »), `dropna` supprime des lignes. Le bon choix dépend de ce que signifie le trou.

### Décalage horaire entre places
Paris ferme à 17h30, New York à 22h. Les mouvements américains de fin de journée n'apparaissent à Paris que le lendemain.
- **À retenir :** ce décalage fait baisser artificiellement les corrélations et les bêtas calculés en rendements quotidiens. Solution : passer en rendements hebdomadaires.

---

## 2. Rendement

### Rendement quotidien
Variation en pourcentage du prix d'un jour à l'autre.
- Formule : `(prix du jour − prix de la veille) / prix de la veille`
- Code : `san["Close"].pct_change()`
- **À retenir :** la première ligne vaut NaN (pas de veille). On compare des **rendements**, jamais des niveaux de prix (un titre à 100 € n'est pas « plus cher » qu'un titre à 10 €).

### Rendements hebdomadaires
Variation du prix d'une semaine à l'autre : on garde le dernier prix de chaque semaine, puis on calcule le rendement.
- `resample("W")` : regroupe les lignes par semaine · `.last()` : garde le dernier prix de chaque semaine
- Code : `portefeuille.resample("W").last().pct_change().dropna()`
- Annualisation : × 52 (rendement) et × √52 (volatilité), au lieu de 252
- **À retenir :** pour comparer des places aux horaires différents, l'hebdomadaire évite le biais du décalage horaire (NVDA-CW8 : 0,37 en quotidien, 0,59 en hebdomadaire). Moins d'observations, donc des estimations moins précises.

---

## 3. Risque

### Écart-type
Mesure de la dispersion des valeurs autour de leur moyenne. Plus il est grand, plus les rendements s'éloignent de la moyenne.
- Code : `.std()`

### Volatilité
Écart-type des rendements = amplitude typique des variations. C'est la mesure de risque la plus utilisée.
- Quotidienne : `san["rendement"].std()` → Sanofi ≈ **1,47 %**
- Annualisée : `vol_quotidienne * (252 ** 0.5)` → Sanofi ≈ **23,3 %**
- **À retenir :** on multiplie par **√252**, pas par 252 : la volatilité croît avec la racine du temps.
- Repères : grande capitalisation européenne 15–25 %, tech US 40–60 %.

### Clustering de volatilité
Les périodes agitées se regroupent : une grosse journée est souvent suivie d'autres grosses journées.
- **À retenir :** le risque arrive par vagues, pas de façon régulière.

### Asymétrie / queues épaisses
Les chutes extrêmes sont plus violentes et plus fréquentes que ne le prévoit la loi normale.
- Sanofi, 27/10/2023 : −19 % en une séance ≈ 13 écarts-types — « impossible » selon une loi normale.
- **À retenir :** la volatilité sous-estime le risque de krach.

### Plus haut historique courant
Le maximum atteint depuis le début de la période, jour après jour.
- Code : `san["Close"].cummax()`

### Drawdown
Perte depuis le dernier sommet atteint.
- Formule : `prix / plus haut courant − 1`
- Code : `san["Close"] / san["Close"].cummax() - 1`
- Vaut 0 quand le titre est à son sommet, négatif sinon.

### Drawdown maximum (max drawdown)
La pire perte depuis un sommet sur toute la période.
- Code : `san["drawdown"].min()` et sa date `san["drawdown"].idxmin()`
- Sanofi : **−27,8 % le 9 mars 2026**
- **À retenir :** c'est ce que l'investisseur *vit* réellement — la perte maximale qu'il aurait subie en achetant au pire moment.

### Volatilité vs drawdown
| | Volatilité | Drawdown |
|---|---|---|
| Voit bien | les chocs brutaux (oct. 2023) | l'érosion lente et durable (2025–2026) |
| Voit mal | la baisse lente | l'agitation sans tendance |

- **À retenir :** aucune mesure de risque ne suffit seule.

### Corrélation
Mesure à quel point deux actifs bougent ensemble, de −1 à +1.
- +1 : ils montent et baissent toujours ensemble
- 0 : aucun lien entre leurs mouvements
- −1 : quand l'un monte, l'autre baisse
- Code : `rendements.corr()`
- **À retenir :** plus les corrélations d'un portefeuille sont faibles, plus le risque global baisse (exemple : glaces + parapluies).

### Diversification
Combiner des actifs peu corrélés pour réduire le risque global sans réduire le rendement espéré.
- Exemple : 4 titres à 25 % → volatilité 18,2 % contre 30,2 % en moyenne
- **À retenir :** le risque d'un portefeuille est inférieur à la moyenne des risques, sauf si corrélation = 1. C'est le « seul repas gratuit en finance » (Markowitz, 1952).

### Risque de change
Pour un investisseur en euros, le rendement d'un actif étranger dépend aussi de l'évolution de sa devise face à l'euro.
- Exemple : Nvidia +10 %, dollar −5 % face à l'euro → environ +5 % pour toi

### Ratio de Sharpe
Rendement obtenu par unité de risque, au-delà du taux sans risque.
- Formule : `(rendement annuel − taux sans risque) / volatilité annuelle`
- Repères : < 0 mauvais · 0 à 1 correct · > 1 très bon · > 2 exceptionnel (souvent suspect)
- **À retenir :** permet de comparer deux investissements qui ont des risques différents.

### Taux sans risque
Rendement d'un placement quasi sans risque (livret, emprunt d'État court). C'est une **hypothèse** qu'on choisit, pas un calcul.
- **À retenir :** un investissement risqué n'a de valeur que s'il rapporte plus que ce taux.

### Rendement annualisé
- Code : `rendements.mean() * 252`
- **À retenir :** le rendement s'annualise × 252, la volatilité × √252.

### Bêta
Sensibilité d'un titre aux mouvements du marché.
- Formule : `Cov(titre, marché) / Var(marché)`
- Code : `rendements["SAN.PA"].cov(marche) / marche.var()`
- = 1 : suit le marché · > 1 : amplifie (Nvidia 1,34) · < 1 : amortit, titre défensif (Sanofi 0,35)
- **À retenir :** la volatilité mesure le risque total, le bêta seulement la part liée au marché.

### Relation bêta / corrélation
Le bêta combine deux choses : à quel point le titre suit le marché (corrélation) et avec quelle amplitude (rapport des volatilités).
- Formule : `bêta = corrélation × (volatilité du titre / volatilité du marché)`
- Code : `titre.corr(marche) * titre.std() / marche.std()` donne le même résultat que `titre.cov(marche) / marche.var()`
- **À retenir :** quand la corrélation monte, le bêta monte aussi (NVDA : 1,34 en quotidien, 2,14 en hebdomadaire). Un titre très volatil mais peu corrélé peut avoir un bêta faible.

### Covariance et variance
- **Variance** : volatilité au carré, mesure l'ampleur des mouvements d'une série. Code : `.var()`
- **Covariance** : mesure si deux séries bougent ensemble, en tenant compte de l'ampleur. Code : `serie_a.cov(serie_b)`

---

## 4. Construction de portefeuille

### Approche cœur-satellites
Un **cœur** large et peu coûteux (ETF monde) qui porte l'essentiel du portefeuille, et des **satellites** plus petits (actions, ETF sectoriels) choisis pour un objectif précis : diversifier, amortir les baisses, exprimer une conviction.
- Exemple étudié : cœur ETF monde + poche Europe + 15 %, 30 % ou 40 % de satellites en actions européennes défensives
- **À retenir :** le cœur fixe le risque de base, les satellites le modulent. Un satellite n'a d'intérêt que s'il apporte quelque chose que le cœur n'a pas.

### ETF de remplacement (proxy)
Quand l'ETF réellement étudié a un historique trop court (lancé récemment), on le remplace dans les calculs par un autre ETF **plus ancien qui réplique le même indice**.
- Exemple : CW8 (Amundi MSCI World, coté depuis des années) pour représenter n'importe quel ETF MSCI World ; IUSQ pour le MSCI ACWI
- Vérifier la profondeur d'historique : `prix.notna().idxmax()`
- **À retenir :** même indice = mêmes mouvements, à quelques frais et écarts de suivi près. On analyse l'indice, pas le produit.

### Poids d'un portefeuille et somme égale à 1
Le poids = la part du portefeuille investie dans chaque actif. La somme des poids doit valoir **1** (100 %).
- Contrôle : `poids.sum()`
- Somme < 1 : une partie du portefeuille est oubliée (comme du cash à 0 %) → risque sous-estimé
- Somme > 1 : on investit plus que ce qu'on a (levier) → risque surestimé
- Arrondis : 11 actifs à `0.0909` donnent 0,9999. Écrire le poids comme une division (`0.40 / 6`, `1 / 11`) pour que Python calcule la valeur exacte, au lieu de l'arrondir à la main
- **À retenir :** toujours vérifier la somme avant de comparer des portefeuilles, sinon on ne compare pas la même chose.

### Series pandas de poids
Ranger les poids dans une `pd.Series` dont l'**index** contient les tickers : chaque poids reste attaché à son nom.
- Code : `poids = pd.Series({"A": 0.6, "B": 0.4})`
- Rendement du portefeuille : `(rendements[poids.index] * poids).sum(axis=1)`
- `rendements[poids.index]` : ne garde que les colonnes du portefeuille
- `* poids` : pandas aligne sur les noms de colonnes, pas sur l'ordre
- `.sum(axis=1)` : additionne **par ligne** (un rendement de portefeuille par jour) ; sans `axis=1`, on additionnerait chaque colonne sur toute la période
- **À retenir :** un seul objet porte les tickers et les poids ; changer de variante = changer la Series, le reste du code ne bouge pas.

### Courbe de valeur avec cumprod
Valeur d'1 € investi au départ, jour après jour, reconstruite à partir des rendements.
- Code : `valeur = (1 + rendement_ptf).cumprod()`
- `1 + r` : facteur de croissance du jour (+2 % → 1,02) · `cumprod()` : produit cumulé de ces facteurs
- **À retenir :** les rendements se **multiplient**, ils ne s'additionnent pas (+10 % puis −10 % = 0,99, pas 1).

### Drawdown d'un portefeuille
Même calcul que pour un titre, mais sur la courbe de valeur du portefeuille.
- Code : `dd = valeur / valeur.cummax() - 1`, puis `dd.min()` et `dd.idxmin()`
- **À retenir :** le drawdown se calcule sur une **valeur** (prix ou courbe cumulée), jamais directement sur des rendements. Pour un portefeuille, il faut donc d'abord reconstruire la courbe avec `cumprod()`.

### Rendements décroissants de la diversification
Chaque ajout d'actifs peu corrélés réduit le risque, mais de moins en moins.
- Exemple étudié (2021-2026) :

| Satellites | Volatilité | Drawdown max |
|---|---|---|
| 0 % (ETF monde seul) | 13,8 % | −21,7 % |
| 15 % | 12,3 % | −18,0 % |
| 30 % | 11,4 % | −15,3 % |
| 40 % | 11,1 % | −13,8 % |

- La volatilité gagne 1,5 point puis 0,9 puis 0,3 ; le drawdown continue de baisser nettement
- **À retenir :** au-delà d'un certain point, diversifier davantage n'apporte presque plus rien en volatilité. Regarder aussi le drawdown : il peut continuer à s'améliorer quand la volatilité stagne.

### Défensif vs diversifiant
Deux qualités différentes, souvent confondues.
- **Défensif** : baisse moins que le marché → bêta < 1 (ex. Sanofi 0,35)
- **Diversifiant** : bouge différemment du reste du portefeuille → corrélation faible avec lui
- Un actif peut être volatil **et** diversifiant : des actions européennes plus volatiles qu'un ETF monde, mais peu corrélées au Nasdaq-100, font baisser le risque du portefeuille
- Un actif peut être peu volatil **sans** diversifier : un ETF S&P 500 est corrélé à 0,98 avec un ETF MSCI World, il n'ajoute presque rien
- **À retenir :** pour juger un ajout, regarder sa **corrélation avec ce qu'on détient déjà**, pas seulement sa volatilité.

---

## 5. Lecture de marché

### Écart au consensus
Le **consensus** = la moyenne des prévisions des analystes. Le marché réagit à l'écart entre le résultat publié et ce qui était attendu, pas au résultat lui-même.
- Sanofi, 27/10/2023 : prévisions 2024 en baisse alors que le consensus attendait une hausse → −19 %.
- **À retenir :** un bon résultat peut faire chuter le cours s'il était moins bon qu'attendu.

### Biais de sélection
Choisir les titres d'une étude en connaissant déjà leur performance passée. Le résultat paraît excellent, mais il était impossible à obtenir à l'époque.
- **À retenir :** une performance passée ne dit rien sur la suite.

### Marge vs croissance
Deux ratios qui ne mesurent pas la même chose : la marge compare deux lignes d'une **même année**, la croissance compare une même ligne sur **deux années**.
- **Marge** : bénéfice / chiffre d'affaires de la même année (SMCI 2026 : 2,2 / 39,1 = 5,6 %)
- **Croissance** : variation d'une année sur l'autre (SMCI : chiffre d'affaires +78 %)
- **À retenir :** une croissance de 100 % est possible, une marge de 100 % ne l'est pas. Une marge nette proche de 100 % signale un gain exceptionnel.

---

## 6. Réflexes Jupyter

- `Shift + Entrée` : exécuter la cellule
- `fonction?` : afficher la documentation (ex. `yf.download?`)
- Touche `M` sur une cellule : la passer en Markdown (texte)
- `Kernel → Restart Kernel and Run All Cells` : tout relancer dans l'ordre si les résultats deviennent incohérents
- `NameError` : variable inconnue → kernel redémarré, cellule non exécutée, ou faute de frappe
- Lire une erreur **de bas en haut** : la dernière ligne donne le diagnostic, la flèche `---->` la ligne fautive

---

## 7. Syntaxe Python

- `rendements["CW8.PA"]` : sélectionner une colonne d'un tableau
- `marche = rendements["CW8.PA"]` : donner un nom court à une colonne
- `serie.mean()` → moyenne · `serie.std()` → écart-type · `serie * 252` → multiplie chaque ligne
- `.iloc[0]` / `.iloc[-1]` : première / dernière valeur d'une série
- Méthode : `ce_sur_quoi_je_calcule.methode()` ; si elle compare deux séries, la deuxième va entre les parenthèses
- Python distingue majuscules et minuscules : `print`, pas `Print`

### Boucle for
Répète les mêmes instructions pour chaque élément d'une liste.
- La ligne `for` finit par `:`
- Les lignes répétées sont décalées avec Tab
- Dans la boucle, la variable (ex. `ticker`) s'utilise sans guillemets : `rendements[ticker]`

### Erreurs fréquentes
- `IndentationError` : lignes pas décalées sous un `for`
- `TypeError: missing 1 required positional argument` : il manque un élément entre les parenthèses
- `AttributeError: 'list' / 'str' object has no attribute` : méthode appliquée à du texte ou à une liste au lieu d'une colonne de données