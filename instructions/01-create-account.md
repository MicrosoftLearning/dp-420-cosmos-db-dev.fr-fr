---
lab:
  title: Créer un compte Azure Cosmos DB for NoSQL
  module: Module 1 - Get started with Azure Cosmos DB for NoSQL
---

# Créer un compte Azure Cosmos DB for NoSQL

Avant de plonger trop profondément dans Azure Cosmos DB, il est important d’avoir une bonne idée des principes de base de la création des ressources que vous utiliserez le plus. Dans la plupart des scénarios, vous devrez être à l’aise en création de comptes, de bases de données, de conteneurs et d’éléments. Dans un scénario réel, vous devriez également avoir quelques requêtes de base « sous la main » pour tester que vous avez créé toutes vos ressources correctement.

Dans ce labo, vous allez créer un nouveau compte Azure Cosmos DB en utilisant for NoSQL. Vous utiliserez ensuite l’Explorateur de données pour créer une base de données, un conteneur et deux éléments. Enfin, vous interrogerez la base de données pour les éléments que vous avez créés.

## Créer un compte Azure Cosmos DB

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Lorsque vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que vous souhaitez que le compte prenne en charge (par exemple, **API Mongo** ou **API NoSQL**).

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au Portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Dans la catégorie **services Azure**, sélectionnez **Créer une ressource**, puis sélectionnez **Azure Cosmos DB**.

    > &#128161; Autrement, développez le menu **&#8801;**, sélectionnez **Tous les services**, dans la catégorie **Bases de données**, sélectionnez **Azure Cosmos DB**, puis sélectionnez **Créer**.

1. Dans le volet **Sélectionner l’API**, sélectionnez l’option **Créer** dans la section **Azure Cosmos DB for NoSQL** .

1. Dans le volet **Créer un compte Azure Cosmos DB**, observez l’onglet **Informations de base**.

1. Sous l’onglet **Informations de base**, entrez les valeurs suivantes pour chaque paramètre :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **Type de charge de travail** | **Formations** |
    | **Abonnement** | **Utilisez votre abonnement Azure existant.** *Toutes les ressources doivent appartenir à un groupe de ressources. Chaque groupe de ressources doit appartenir à un abonnement.* |
    | **Groupe de ressources** | **Utilisez un groupe de ressources existant ou créez-en un.** *Toutes les ressources doivent appartenir à un groupe de ressources.* |
    | **Nom du compte** | **Entrez un nom globalement unique.** *Nom du compte globalement unique. Ce nom sera utilisé comme élément de l’adresse DNS pour les requêtes.  Le portail vérifiera le nom en temps réel.* |
    | **Lieu** | **Choisissez une région disponible.** *Sélectionnez la région géographique depuis laquelle votre base de données sera initialement hébergée.* |
    | **Mode de capacité** | **Débit approvisionné** |
    | **Appliquer la remise de niveau Gratuit** | **Ne pas appliquer** |

1. Sélectionnez **Vérifier + créer** pour accéder à l’onglet **Vérifier + créer**, puis sélectionnez **Créer**.

    > &#128221; 10 à 15 minutes peuvent être nécessaires avant de pouvoir utiliser le compte Azure Cosmos DB for NoSQL.

1. Observez le volet **Déploiement**. Une fois le déploiement terminé, le volet se met à jour avec le message **Déploiement réussi**.

1. Toujours dan le volet **Déploiement**, sélectionnez **Accéder à la ressource**.

## Utiliser l’Explorateur de données pour créer une base de données et un conteneur

L’Explorateur de données sera votre outil principal pour gérer la base de données et les conteneurs Azure Cosmos DB for NoSQL dans le Portail Azure. Vous allez créer une base de données et un conteneur simples que vous utiliserez dans ce labo.

1. Dans le volet du **compte Azure Cosmos DB**, sélectionnez **Explorateur de données** dans le menu des ressources.

1. Sur le volet **Explorateur de données** , sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *cosmicworks* |
    | **Partager le débit entre les conteneurs** | *Non vérifié* |
    | **ID de conteneur** | *produits* |
    | **Clé de partition** | */categoryId* |
    | **Débit du conteneur (mise à l’échelle automatique)** | *Manuel* |
    | **RU/s** | *400* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks** , puis observez le nœud de conteneur des **produits** dans la hiérarchie.

## Utiliser l’Explorateur de données pour créer des éléments

L’Explorateur de données inclut également une suite de fonctionnalités pour interroger, créer et gérer des éléments dans un conteneur Azure Cosmos DB for NoSQL. Vous allez créer deux éléments de base en utilisant JSON brut dans l’Explorateur de données.

1. Dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks** , développez le nœud de conteneur des **produits**, puis sélectionnez **Éléments**.

1. Sélectionnez **Nouvel élément** depuis la barre de commandes et, dans l’éditeur, remplacez l’élément JSON d’espace réservé par le contenu suivant :

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Sélectionnez **Enregistrer** depuis la barre de commandes pour ajouter le premier élément JSON.

1. De retour dans l’onglet **Éléments**, sélectionnez **Nouvel élément**depuis la barre de commandes. Dans l’éditeur, remplacez l’élément JSON d’espace réservé par le contenu suivant :

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. Sélectionnez **Enregistrer** depuis la barre de commandes pour ajouter le deuxième élément JSON.

1. Dans l’onglet **Éléments** , observez les deux nouveaux éléments dans le volet **Éléments**.

## Utiliser l’Explorateur de données pour émettre une requête de base

Enfin, l’Explorateur de données a un éditeur de requête intégré qui est utilisé pour émettre des requêtes, observer les résultats et mesurer l’impact en termes d’unités de requête par seconde (request units per second/RU/s).

1. Dans le volet **Explorateur de données**, sélectionnez **Nouvelle requête SQL**.

1. Dans l’onglet Requête, sélectionnez **Exécuter la requête** pour afficher une requête standard qui sélectionne tous les éléments sans aucun filtre.

1. Supprimez le contenu de la zone de l’éditeur.

1. Remplacez la requête d’espace réservé par le contenu suivant :

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; Cette requête sélectionnera tous les éléments dont le **prix** est supérieur à 500 $.

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats de la requête, qui doivent inclure un seul élément JSON et toutes ses propriétés.

1. Dans l’onglet **Requête** , sélectionnez **Statistiques de requête**.

1. Toujours dans l’onglet **Requête** , observez la valeur du champ **Frais de requête** dans la section **Statistiques de requête**.

    > &#128221 ; En règle générale, les frais de requête pour cette requête simple sont compris entre 2 et 3 RU/s quand la taille du conteneur est petite.

1. Fermez votre fenêtre ou votre onglet de navigateur web.
