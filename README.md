Ajouter des métriques et des graphiques à dvc.yaml
Dans cet exercice, votre tâche est de compléter le contenu de dvc.yaml qui définit un workflow d’entraînement de modèle.

Ici, preprocess_dataset.py et train.py sont les fichiers qui effectuent le prétraitement des données et l’entraînement du modèle en prenant weather.csv comme entrée dans le dossier raw_dataset. En sortie, le code d’entraînement génère un fichier predictions.csv qui contient les prédictions et la vérité terrain, ainsi qu’un fichier metrics.json contenant des métriques structurées. Le premier sera utilisé pour générer un tracé de matrice de confusion normalisée afin de le comparer avec des commits précédents.

Instructions
100XP
Définissez la cible des métriques vers le fichier de métriques de sortie.
Définissez la cible du graphique vers le fichier de sortie contenant les données de prédictions.
Définissez le modèle de graphique sur confusion_normalized pour tracer la matrice de confusion normalisée.
Définissez la valeur correcte pour la clé cache afin de suivre les graphiques dans le dépôt Git plutôt que dans le remote DVC.