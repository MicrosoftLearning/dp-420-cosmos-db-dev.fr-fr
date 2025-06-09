---
lab:
  title: Optimiser une stratégie d’index du conteneur Azure Cosmos DB for NoSQL pour une requête spécifique
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimiser une stratégie d’index d’un conteneur Azure Cosmos DB for NoSQL pour une requête

Lors de la planification d’un compte Azure Cosmos DB for NoSQL, le fait de connaître nos requêtes les plus populaires peut nous aider à régler la stratégie d’indexation afin que les requêtes soient aussi performantes que possible.

Dans ce labo, nous allons utiliser l’Explorateur de données pour tester les requêtes SQL avec la stratégie d’indexation par défaut et une stratégie d’indexation qui inclut un index composite.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple l’**API Mongo** ou l’**API NoSQL**). Une fois l’approvisionnement du compte Azure Cosmos DB for NoSQL effectué, vous pouvez récupérer le point de terminaison et la clé, puis les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant le kit Azure SDK pour .NET ou tout autre kit SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Type de charge de travail** | **Formations** |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Explorateur de données**.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre contextuelle **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *``cosmicworks``* |
    | **Approvisionner le débit** | activé |
    | **Débit de la base de données** | **Manuel** |
    | **RU/s de base de données requises** | ``1000`` |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **cosmicworks** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *``products``* |
    | **Clé de partition** | *``/category/name``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur des **produits** dans la hiérarchie.

1. Dans le panneau des ressources, accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Plus précisément :

    1. Remarquez le champ **PRIMARY CONNECTION STRING**. Vous utiliserez cette valeur de **chaîne de connexion** plus loin dans cet exercice.

1. Ouvrez **Visual Studio Code**.

## Remplir initialement votre compte Azure Cosmos DB for NoSQL avec des exemples de données

Vous allez vous servir d’un utilitaire en ligne de commande qui crée une base de données **cosmicworks** et un conteneur **products**. L’outil crée ensuite un ensemble d’éléments que vous allez observer à l’aide du processeur de flux de modification qui s’exécute dans votre fenêtre de terminal.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir un nouveau terminal.

1. Installez l’outil en ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; L’exécution de cette commande peut prendre quelques minutes. Cette commande génère le message d’avertissement (*L’outil « cosmicworks » est déjà installé), si vous avez déjà installé la dernière version de cet outil.

1. Exécutez cosmicworks pour remplir initialement votre compte Azure Cosmos DB avec les options de ligne de commande suivantes :

    | **Option** | **Valeur** |
    | ---: | :--- |
    | **-c** | *Valeur de la chaîne de connexion que vous avez vérifiée précédemment dans ce labo* |
    | **--number-of-employees** | *La commande cosmicworks remplit votre base de données avec les employés et les conteneurs de produits,1 000 et 200 éléments respectivement, sauf indication contraire* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; Par exemple, si votre point de terminaison est **https&shy;://dp420.documents.azure.com:443/** et si votre clé est **fDR2ci9QgkdkvERTQ==**, la commande est : ``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. Attendez que la commande **cosmicworks** ait fini de remplir le compte avec une base de données, un conteneur et des éléments.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code** et revenez à votre navigateur.

## Exécuter des requêtes SQL et mesurer leurs frais unitaires de requête

Avant de modifier la stratégie d’indexation, vous allez d’abord exécuter quelques exemples de requêtes SQL pour obtenir des frais d’unités de requête de référence exprimés en unités de requête.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, sélectionnez le nœud de conteneur **products**, puis sélectionnez **Nouvelle requête SQL**.

1. Sélectionnez **Exécuter la requête** pour exécuter la requête par défaut :

    ```
    SELECT * FROM c
    ```

1. Observez les résultats de la requête. Sélectionnez **Statistiques de requête** pour afficher les frais d’unité de requête dans les unités de requête.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne les trois valeurs de tous les documents :

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p    
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats et les statistiques de la requête. Les frais d’unité de requête sont presque identiques à la première requête.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne trois valeurs de tous les documents classés par **categoryName** :

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats et les statistiques de la requête. Les frais d’unité de requête ont augmenté en raison de la clause **ORDER BY**.

## Créer un index composite dans la stratégie d’indexation

À présent, vous devez créer un index composite si vous triez vos éléments en utilisant plusieurs propriétés. Dans cette tâche, vous allez créer un index composite pour trier les éléments par leur nom de catégorie, puis leur nom réel.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, sélectionnez le nœud de conteneur **products**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui trie les résultats par **catégorie** en ordre décroissant, puis par **prix** en ordre croissant :

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.price ASC
    ```

1. Sélectionnez **Exécuter la requête**.

1. La requête devrait échouer avec l’erreur **La requête de tri n'a pas d'index composite correspondant à partir duquel elle peut être servie**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks** , développez le nœud de conteneur des **produits**, puis sélectionnez **Paramètres**.

1. Sous l’onglet **Paramètres**, accédez à la section **Stratégie d’indexation**.

1. Observez la stratégie d’indexation par défaut :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. Remplacez la stratégie d’indexation par cet objet JSON modifié, puis **enregistrez** les modifications :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, sélectionnez le nœud de conteneur **products**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui trie les résultats par **categoryName** en ordre décroissant, puis par **prix** en ordre croissant :

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.price ASC
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats et les statistiques de la requête. Cette fois, étant donné que la requête est terminée, vous pouvez à nouveau passer en revue les frais d’unités de requête.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui trie les résultats par **catégorie** en ordre décroissant, puis par **nom** en ordre croissant, puis enfin par **prix** en ordre croissant :

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. Sélectionnez **Exécuter la requête**.

1. La requête devrait échouer avec l’erreur **La requête de tri n'a pas d'index composite correspondant à partir duquel elle peut être servie**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, développez le nœud de conteneur des **produits**, puis sélectionnez **Paramètres**.

1. Sous l’onglet **Paramètres**, accédez à la section **Stratégie d’indexation**.

1. Remplacez la stratégie d’indexation par cet objet JSON modifié, puis **enregistrez** les modifications :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, sélectionnez le nœud de conteneur **products**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui trie les résultats par **catégorie** en ordre décroissant, puis par **nom** en ordre croissant, puis enfin par **prix** en ordre croissant :

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats et les statistiques de la requête. Cette fois, étant donné que la requête est terminée, vous pouvez à nouveau passer en revue les frais d’unités de requête.

1. Fermez la fenêtre ou l’onglet de votre navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
