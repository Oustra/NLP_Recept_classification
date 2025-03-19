## Description de la tâche

L'objectif de cette tâche est de classifier automatiquement une recette en tant que **Plat principal, Entrée ou Dessert**, en se basant sur son titre et ses instructions. 
Nous disposons d'un corpus de recettes de cuisine, divisé en un ensemble d'entraînement et un ensemble de test.

## Statistiques du corpus

### Informations sur les données
- Deux fichiers CSV : `train.csv` et `test.csv`
- 7 colonnes : `doc_id`, `titre`, `type`, `difficulte`, `cout`, `ingredients`, `recette`

### Détails du jeu de données
**Entraînement (`train.csv`)** :
- Aucune valeur manquante
- Répartition des classes :
  - Plat principal : **5 802**
  - Dessert : **3 762**
  - Entrée : **2 909**

**Test (`test.csv`)** :
- Aucune valeur manquante
- Répartition des classes :
  - Plat principal : **644**
  - Dessert : **407**
  - Entrée : **337**

## Méthodes proposées

Les détails des implémentations sont disponibles dans **`src/Projet_TAL.ipynb`**. Voici un aperçu des principales approches :

### Prétraitement des données
- **Concaténation du titre et de la recette** pour un traitement simplifié.
- **Nettoyage du texte** : suppression des chiffres, de la ponctuation et des stop words (expérimentation).

### Approches testées
- **TF-IDF + SVM** : Approche performante et rapide.
- **Embeddings (Word2Vec) + SVM** : Exploration des représentations sémantiques.
- **Optimisation des hyperparamètres** avec `GridSearch`.

## Expérimentations et résultats

|  Modèle                   | Accuracy |
|---------------------------|----------|
| Baseline (random)         | 0.33     |
| TF-IDF + SVM              | 0.87     |
| Embeddings + SVM          | 0.85     |
| TF-IDF + SVM (optimisé)   | 0.89     |

### Analyse des résultats
- **Modèle le plus performant :** TF-IDF + SVM (optimisé)
- **Distribution des scores :**
  - Score < 0.05 : **2050 documents**
  - Score entre 0.4 et 0.6 : **183 documents** (ambiguïté)
  - Score > 0.95 : **678 documents** (haute confiance)
- **Matrice de confusion :**
  - Erreurs principalement entre **Entrée** et **Plat principal**.

## Limitations et perspectives

### Problèmes identifiés
- **Déséquilibre des classes** : La catégorie "Entrée" est sous-représentée.
- **Vocabulaire spécifique** : Les entrées peuvent partager du vocabulaire avec d'autres classes.
- **Modèles classiques limités** : SVM et TF-IDF ne capturent pas bien la sémantique.

### Vers des approches avancées
- **BERT (Transformers)** : Meilleure prise en compte du contexte, mais coûteux en calcul.
- **LSTM (RNNs)** : Capture des relations séquentielles dans le texte.
- **Augmentation des données** : Génération de nouvelles recettes pour équilibrer les classes.

## Conclusion
L'approche **TF-IDF + SVM optimisé** donne de bons résultats, mais des améliorations sont possibles en explorant des **modèles plus avancés** comme **BERT** ou **LSTM**, ou en équilibrant le jeu de données. 

📌 **Les détails et les codes sont disponibles dans `src/Projet_TAL.ipynb`**.
