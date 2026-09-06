# 🚀 Memento MLOps : Comparaison Automatique d'Expériences (DVC + CML)

Ce projet est un guide pratique pour concevoir un pipeline **MLOps complet** capable de comparer automatiquement les performances d'un modèle de Machine Learning entre deux branches Git (votre branche de travail et la branche stable `main`).

---

## 🏗️ L'Architecture Globale (Qui fait quoi ?)

Pour assurer la reproductibilité, trois outils collaborent main dans la main :
1. **GitHub Actions (GHA)** : Le chef d'orchestre. Il démarre un serveur Ubuntu vierge à chaque Pull Request.
2. **DVC (Data Version Control)** : L'artisan du ML. Il sait dans quel ordre exécuter les scripts Python et compare les scores (`dvc metrics diff`).
3. **CML (Continuous Machine Learning)** : Le messager. Il prend les résultats de DVC et les écrit sous forme de commentaire dans GitHub.

---

## 🛠️ Guide Étape par Étape : Processus de Développement

Voici la routine exacte à suivre lorsque vous travaillez sur ce projet (ou sur n'importe quel projet MLOps) :

### Étape 1 : Figer la version de référence sur `main`
La branche `main` doit toujours contenir votre modèle stable de référence.
```bash
git checkout main
git pull origin main

# On s'assure que le pipeline DVC local est à jour
dvc repro

# On sauvegarde cet état stable dans Git
git add dvc.yaml dvc.lock metrics.json predictions.csv
git commit -m "feat: version de base stable sur main"
git push origin main
```

### Étape 2 : Créer une branche d'expérimentation pour tester des Hyperparamètres
Pour améliorer le modèle, on ne travaille **jamais** directement sur `main`. On crée une branche secondaire.
```bash
git checkout -b tuning-hyperparameters
```

### Étape 3 : Modifier les paramètres et réentraîner
1. Ouvrez `model.py` (ou le fichier de configuration de votre modèle).
2. Modifiez un hyperparamètre (ex: changez `n_estimators=100` par `n_estimators=20`).
3. Forcez DVC à réexécuter l'entraînement avec la nouvelle configuration :
   ```bash
   dvc repro
   ```
   *DVC réentraîne le modèle, écrase `metrics.json` avec les nouveaux scores, et met à jour `predictions.csv`.*

### Étape 4 : Tester la comparaison en Local
Avant d'envoyer sur GitHub, vous pouvez vérifier le gain (ou la perte) de performance directement dans votre terminal par rapport à l'historique de `main` :
```bash
# Compare le fichier metrics.json actuel avec celui sauvegardé sur main
dvc metrics diff main
```

### Étape 5 : Propulser sur GitHub et déclencher la CI/CD
Envoyez votre branche de test sur GitHub pour déclencher les automatisations :
```bash
git add model.py
git commit -m "perf: modification de n_estimators pour optimisation"
git push origin tuning-hyperparameters
```

### Étape 6 : La magie de la Pull Request
1. Allez sur GitHub et ouvrez une **Pull Request** (de `tuning-hyperparameters` vers `main`).
2. **Ne fusionnez pas tout de suite !** Attendez 1 à 2 minutes.
3. Le robot GitHub Actions va lire votre `.github/workflows/cml.yaml`, exécuter l'entraînement, et publier un tableau comparatif automatique en commentaire.

---

## 🔍 Décryptage du Fichier CI/CD (`cml.yaml`)

Voici pourquoi certaines lignes de votre fichier GitHub Actions sont cruciales :

* **`sudo apt-get install ... libpixman-1-dev`** : Obligatoire pour installer les dépendances graphiques Linux nécessaires au bon fonctionnement de CML sur les serveurs récents de GitHub.
* **`git fetch --prune --unshallow`** : L'étape la plus importante ! Par défaut, le serveur GitHub télécharge uniquement votre commit actuel et ignore le reste de l'historique. Cette commande force Git à télécharger la branche `main` distante. Sans elle, la commande suivante plante car la machine virtuelle ne sait pas ce qu'est `main`.
* **`dvc metrics diff main --md >> report.md`** : Demande à DVC de comparer les scores du "Workspace" actuel avec ceux de `main`, et l'option `--md` formate automatiquement le résultat en un joli tableau Markdown pour GitHub.
* **`cml comment create report.md`** : Prend le fichier texte généré et l'injecter sous forme de post dans la discussion de la Pull Request.

---

## 🗂️ Commandes de secours DVC à retenir
- `dvc repro` : Lance ou met à jour le pipeline d'entraînement ML.
- `dvc dag` : Affiche graphiquement l'arbre de dépendance de vos scripts dans le terminal.
- `dvc metrics show` : Affiche les scores de performance de votre répertoire de travail actuel.
