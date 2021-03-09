
## Valeo - Détection de défaults

<img src="https://cdn.worldvectorlogo.com/logos/valeo-logo-1.svg" alt="Logo Valeo" height=75px/>

[Lien de la compétition](https://challengedata.ens.fr/participants/challenges/58)

### **Description**
Lors de l'assemblage du module, une "inspection optique automatique" (AOI) est effectuée après un processus de soudage des fils pour vérifier la conformité et la qualité des pièces. Cette inspection est basée sur des photos prises par une caméra et des algorithmes de base utilisés pour mesurer certains paramètres spécifiques sur les pièces. La machine AOI est efficace pour mesurer les dimensions sur les pièces (largeur du fil de soudure par exemple) mais beaucoup moins pour les défauts "d'aspect". Cette difficulté à analyser correctement ce type de défaut conduit à un grand nombre de pièces qui doivent être confirmées manuellement par les opérateurs. Dans certaines conditions, le taux de "faux défaut" (pièces considérées comme KO par la machine mais OK par l'opérateur) pourrait atteindre 10 ou 20% de la production.

**Objectif** : Définir un modèle qui pourrait fournir un meilleur résultat que l'AOI pour **confirmer la présence de défaut sur les pièces d'un module de puissance à partir de photos** prises lors de la production de celui-ci dans l'usine Valeo de Sablé sur Sarthe.

### **Fichiers**
Les données sont disponibles à l'adresse de la compétition pour qui souhaite exécuter ce projet. Il faudra ensuite les convertir au format TFRecord à l'aide du fichier `hdf5_tfrec.ipynb` pour pouvoir exécuter le notebook du modèle `Xception_tfrec.ipynb`. Par ailleurs, le notebook est configuré pour être exécuté sur le TPU mis à disposition par le site Kaggle. La documentation sur les TPU Kaggle est disponible [ici](https://www.kaggle.com/docs/tpu).
* `Challenge Proposal_Valeo_PES_2020.pdf` : Valeo, contacts, description de la compétition, données et métrique du score
* `Random_Submission.csv` : format des données de sortie
* `Xception_tfrec.ipynb` : modélisation
* `hdf5_tfrec.ipynb` : transformer un dossier d'images vers le format hdf5 ou tfrecord
* `sample_submission.csv` : données de sortie du modèle

### **Données**
* **Images** : 10609 images d'entrainement et 1989 images de test au format JPEG de formats différents > à 1000x1000px
* **Labels** : 0 le défaut a été confirmé par l'opérateur, 1 il n'a pas été confirmé. Au total, il y a 38% de défauts dans le jeu d'entrainement. 
* **Score** : Le but de ce défi est d'éviter au mieux les faux positifs c'est-à-dire les images n'ayant pas de défauts identifiés par l'algorithme alors que réellement il y avait un (ou plusieurs) défaut(s) puisqu'elles ont un coût élevé pour l'entreprise. Ainsi l'erreur qui minimise les faux positifs et faux négatifs (qui ont un coût plus léger) est : `C = (1\N)*(FN + 100FP)`

### **Méthodologie**
La démarche adoptée pour la solution finale est la suivante :
1. **Convertir** les images et labels au format **TFRecord** qui est optimisé pour l'usage de Tensorflow et aux TPU en re-ajustant les images au format 512x512px.
2. **Augmenter** les données de façon à réduire le sur-ajustement des données lors de la modélisation (ajustement de la teinte, du contraste, de la luminosité, de la saturation aléatoires et renversement haut-bas, gauche-droite aléatoire).
3. **Re-balancement des données** en modifiant les poids associés à chaque classe.
4. **Modélisation** avec apprentissage par transfert en utilisant le modèle Xception préformé sur l'ensemble d'images "Images Net" par la méthode **out-of-fold.**
  * Séparer le jeu d'entraînement en 5 groupes (ou blocs).
    + Pour chaque groupe, entraîner avec les 4 autres groupes, évaluer sur ce groupe et prédire sur le jeu de test.
    + Stocker les prédictions du jeu de validation et de test.
  * Moyenner les 5 prédictions par image du jeu de test.
  * Evaluer l'ensemble du jeu de données à partir des prédictions effectuées sur les 5 groupes.
 4. **Optimisation du seuil de décision** à partir duquel une observation est déclarée comme anomalie ou non.

### **Expérimentations**
+ Format HDF5 : Environ 90sec/epoch contre 4sec/epoch avec des images au format TFRecord taille 175x175 + RAM qui sature très vite. [-]
+ Fine-tuning : Entraîner les couches supérieures du modèle (les plus spécialisées) de façon à l'affiner et à ce qu'elles soient plus adaptées à notre ensemble de données[=].
+ Rebalancement des classes par sur-échantillonnage : Dupliquer aléatoirement la classe minoritaire de façon à re-équilibrer les classes. Attention au sur-ajustement ! [=]
+ Mécanisme d'attention [=]
+ Extraction des variables du modèle CNN préformé et classifieur SVM [-], classifieur Random Forest [-]
+ MobileNetV2 [-], VGG16 [+-], InceptionV3 [+-], InceptionResNetV2 [-+]

### **Piste d'amélioration**
**Méthode d'ensemble : le stacking**. Le principe est d'utiliser les prédictions (probabilités) de différents modèles type VGG16, EfficientNet, ResNet avec la méthode out-of-fold comme variables explicatives. Il est possible de jouer sur la taille des images pour permettre au modèle de distinguer certaines formes avec une certaine taille et d'autres formes avec d'autres tailles. Une fois les prédictions collectées, elles servent de variables explicatives à un modèle de classification LightGBM, XGBoost, SVM, etc. Pour aller encore plus loin, il est possible d'y incorporer des variables descriptives extraites des images.

## **Références**
[Apprentissage par transfert et mise au point - TensorFlow](https://www.tensorflow.org/tutorials/images/transfer_learning), [Augmentation des données - TensorFlow](https://www.tensorflow.org/tutorials/images/data_augmentation), [Triple Stratified K-Flod with TFRecords - Chris Deotte](https://www.kaggle.com/cdeotte/triple-stratified-kfold-with-tfrecords)
