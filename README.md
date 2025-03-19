## Description de la tâche

L'objectif de la tâche à effectué est de trouver automatiquement en fonction du titre et des instructions d'une recette s'il s'agit d'un Plat principal, une Entrée ou bien un Dessert.
Pour cela, nous avons à disposition un corpus de recettes de cuisines séparé en ensemble d'entraînement et de test.

## Statistiques corpus

**Information sur les données:** \
2 fichiers csv train et test \
7 colones chacun: doc_id, titre, type, difficulte, cout, ingredients, recette

**Pour train.csv :**
* Aucunes valeurs manquantes
* Répartition des 3 types de plats :
    * Plat principal 5802
    * Dessert 3762
    * Entrée 2909

**Pour test.csv :**
* Aucunes valeurs manquantes
* Répartition des 3 types de plats :
    * Plat principal 644
    * Dessert 407
    * Entrée 337


## Méthodes proposées

Tout est détaillé dans le fichier 'src/Projet_TAL.ipynb'. \
On aborde les RUN les plus important ci-dessous.

### Approche :
Le projet à été réalisé sur Google Colab.

**Netoyage des données:** \
Nous avons fait le choix de concaténer le titre et la recette afin de faciliter le traitement. \
**V1 avec spacy** : Très long et donne de moins bon résultas dans notre cas . \
**V2 expressions régulières:** Méthode rapide. Afin de normaliser le text, nous avons tester d'enlever les stop word mais cela donne un résultat très légerement moins bon.

**Encodage des labels:** \
Les modèles de classification que nous utilisons par la suite attendent uniquement des labels numériques.

**Test de différent descripteur:**
TF-IDF, embeddings

**Test de différent classifieur:**
SVM, Naïve Bayes, Random Forest, XGB, GridSearch

### Run1: baseline (méthode de référence)

exec: 1s TF-IDF + 1s random = 2s \
Description de la méthode:
- descripteurs : TfidfVectorizer de sklearn (TF-IDF)
    - solution rapide et efficace

- classifieur : random
    - avoir une base pour ne pas trouver pire résultats ensuite

### Run2: TF-IDF + SVM

exec: 1s TF-IDF + 120s SVM = 2min \
Description de la méthode:
- descripteurs : TfidfVectorizer de sklearn (TF-IDF)
    - solution rapide et efficace

- classifieur : SVC de sklearn (SVM)
    - SVM fonctionne bien avec TF-IDF


### Run3: Embeddings + SVM

exec: 30s Embeddings + 40s SVM = 1min10s \
Description de la méthode:
- descripteurs : Word2Vec de gensim (Embeddings)
    - transformer du texte en vecteurs

- classifieur : SVC de sklearn (SVM)
    - SVM fonctionne bien avec TF-IDF


### Run4: TF-IDF + SVM (optimisation)
Après plusieurs test empiriques, la meilleure configuration que nous avons trouvé est la suivante : \
**Nettoyage :** Concaténer titre et recette, supprimer les chiffres, supprimer la ponctuation. \
**TF-IDF :** TfidfVectorizer(max_features=5000, sublinear_tf=True) \
\- max_features=5000 : Sélectionne les 5000 mots les plus fréquents en fonction de leur score TF-IDF. \
\- sublinear_tf :Applique une transformation logarithmique à la fréquence des mots, et donc réduit l'impact des mots très fréquents.

**SVM :** SVC(kernel="rbf") \
\- kernel="rbf" : Gère des séparations non linéaires pour mieux classer les textes.


## Résultats

|  Run                  | Accuracy |
| ------                | ------   |
| baseline              | 0.33     |
|TF-IDF + SVM           | 0.87     |
|Embeddings + SVM       | 0.85     |
|TF-IDF + SVM (optimisé)| 0.89     |

## Analyse de résultats

L'analyse est réaliser sur le modèle le plus performant : TF-IDF + SVM (optimisé). L'analyse est aussi plus détaillé dans le fichier 'src/Projet_TAL.ipynb'.

* Combien de documents ont un score de 0 ? de 0.5 ? de 1 ? (Courbe ROC)

Nombre de documents avec un score < 0.05 : 2050 \
Nombre de documents avec un score entre 0.4 et 0.6 : 183 \
Nombre de documents avec un score > 0.95 : 678 

Le modèle est très confiant pour 678 cas. \
Le modèle rejette fortement certaines classes pour 2050 cas. \
Le modèle ne sait pas bien classer 183 cas.

D'après la courbe ROC les classes 'Entrée' et 'Plat principal' ont quelques erreurs mais restent très bien séparées.
La classe 'Dessert' quant à elle, est classée de manière prèsque parfaite.

* Où est-ce que l'approche se trompe ? (matrice de confusion)

D'après la matrice de confusion le modèle se trompe majoritairement sur la différence entre les classes 'Plat principal' et 'Entrée'.

### Problèmes identifiés : la classification des entrées

Le principal problème rencontré dans ce modèle semble concerner la catégorie des entrées. Bien que les entrées soient correctement classées dans une majorité de cas, les faux négatifs sont élevés. Cela signifie que plusieurs recettes d'entrées sont mal classées comme plats principaux ou dans d'autres catégories. Plusieurs hypothèses peuvent expliquer cette mauvaise performance :

> Vocabulaire spécifique dans les entrées : Il est possible que certaines recettes d'entrées utilisent un vocabulaire plus spécifique ou moins fréquent, ce qui rend la classification plus difficile. Par exemple, certains ingrédients ou termes peuvent apparaître uniquement dans les recettes d'entrées, mais aussi dans celles des plats principaux. Cela peut créer de la confusion pour le modèle.

> Fréquence des termes dans les données d'entraînement : Le problème pourrait également résider dans le fait que les termes utilisés pour décrire les entrées sont moins fréquents dans les données d'entraînement, ce qui entraîne un déséquilibre dans la classification. Les modèles comme le SVM avec TF-IDF comptent sur les fréquences des termes pour effectuer leur classification, et un faible nombre de termes spécifiques peut nuire à la précision du modèle.

> Données déséquilibrées : La catégorie des entrées est sous-représentée dans le jeu de données, ce qui peut expliquer la mauvaise performance de classification. Les modèles de machine learning classiques ont du mal à apprendre les caractéristiques des classes minoritaires, en particulier lorsque les données sont déséquilibrées.

### Limites des modèles classiques

Les modèles simples et classiques comme le SVM avec TF-IDF, XGBoost, Random Forest et Naïve Bayes, bien qu'efficaces dans certains cas, présentent des limitations notables lorsqu'il s'agit de capturer la sémantique et le sens profond des données textuelles. Ces modèles ne parviennent pas à saisir pleinement les relations contextuelles entre les mots dans un titre ou une recette. Ils se basent essentiellement sur des correspondances de fréquence et de termes, ce qui peut ignorer des aspects importants du langage comme la polysémie ou la synonymie.
Proposition de solutions : vers des méthodes avancées

Pour améliorer les performances, notamment en ce qui concerne la classification des entrées, il serait pertinent d'explorer des méthodes plus avancées de traitement du langage naturel, telles que :

> BERT (Bidirectional Encoder Representations from Transformers) : BERT est un modèle pré-entraîné qui capte mieux les relations contextuelles entre les mots grâce à son architecture de transformeur bidirectionnelle. En utilisant BERT, nous pourrions potentiellement améliorer la compréhension du modèle du contexte global d'une recette et donc améliorer la classification des entrées. Cependant, l'utilisation de BERT nécessite plus de ressources de calcul et de données d'entraînement suffisantes.

> LSTM (Long Short-Term Memory) : Les réseaux de neurones LSTM, bien que plus simples que BERT, sont également capables de saisir les relations séquentielles dans les données textuelles. LSTM pourrait être utile pour identifier des dépendances à long terme entre les mots dans une recette et mieux classer les types de recettes, en particulier dans les cas où le contexte des mots joue un rôle important
