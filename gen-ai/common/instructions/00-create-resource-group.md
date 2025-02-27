---
title: Créer un groupe de ressources de labo
lab:
  title: Créer un groupe de ressources de labo
  module: Setup
layout: default
nav_order: 1
parent: Common setup instructions
---

# Créer un groupe de ressources Azure pour le labo

Avant de terminer ce labo, vous devez créer un [groupe de ressources][docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal] pour y placer la ressource Azure nouvellement déployée.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sur la page **Accueil**, sélectionnez **Groupes de ressources**.

    > &#128161; Sinon, développez le menu **&#8801;**, sélectionnez **Tous les services**, puis, dans la catégorie **Tout**, sélectionnez **Groupes de ressources**.

1. Sélectionnez **+ Créer**.

1. Dans la fenêtre indépendante **Créer un groupe de ressources**, créez un groupe de ressources avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Donnez un nom unique à votre groupe de ressources* |
    | **Région** | *Choisissez une région disponible* |

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

[docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]: https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal
