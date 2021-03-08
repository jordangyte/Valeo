
# Valeo - Détection de défaults

<img src="https://cdn.worldvectorlogo.com/logos/valeo-logo-1.svg" alt="Logo Valeo" height=150px/>
[Lien de la compétition](https://challengedata.ens.fr/participants/challenges/58)

## **Description**
Lors de l'assemblage du module, une "inspection optique automatique" (AOI) est effectuée après un processus de soudage des fils pour vérifier la conformité et la qualité des pièces. Cette inspection est basée sur des photos prises par une caméra et des algorithmes de base utilisés pour mesurer certains paramètres spécifiques sur les pièces. La machine AOI est efficace pour mesurer les dimensions sur les pièces (largeur du fil de soudure par exemple) mais beaucoup moins pour les défauts "d'aspect". Cette difficulté à analyser correctement ce type de défaut conduit à un grand nombre de pièces qui doivent être confirmées manuellement par les opérateurs. Dans certaines conditions, le taux de "faux défaut" (pièces considérées comme KO par la machine mais OK par l'opérateur) pourrait atteindre 10 ou 20% de la production.

**Objectif** : Définir un modèle qui pourrait fournir un meilleur résultat que l'AOI pour confirmer la présence de défaut sur les pièces d'un module de puissance à partir de photos prises lors de la production de celui-ci dans l'usine Valeo de Sablé sur Sarthe.

## **Fichiers**
*`Challenge Proposal_Valeo_PES_2020.pdf` : Valeo, contacts, description de la compétition, données et métrique du score.
*`Random_Submission.csv` : format des données de sortie
*`Xception_tfrec.ipynb` : modélisation
*`hdf5_tfrec.ipynb` : transformer un dossier d'images vers le format hdf5 ou tfrecord
*`sample_submission.csv` : données de sortie du modèle
