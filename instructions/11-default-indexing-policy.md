---
lab:
  title: Examiner la stratégie d’index par défaut pour un conteneur Azure Cosmos DB for NoSQL avec le portail
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Examiner la stratégie d’index par défaut pour un conteneur Azure Cosmos DB for NoSQL avec le portail

Chaque conteneur dans Azure Cosmos DB a une stratégie d’indexation qui dirige le service sur la façon d’indexer des éléments dans le conteneur. Par défaut, cette stratégie d’indexation indexe chaque propriété de chaque élément. La stratégie d’indexation par défaut vous permet de commencer à utiliser Azure Cosmos DB de façon simple et rapide, car vous n’avez pas à penser à l’indexation, aux performances et à la gestion au début d’un projet.

Dans ce labo, vous allez observer et manipuler la stratégie d’indexation par défaut pour quelques conteneurs à l’aide de l’Explorateur de données.

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

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Remarquez le champ **PRIMARY CONNECTION STRING**. Vous utiliserez cette valeur de **chaîne de connexion** plus loin dans cet exercice.

## Alimenter le compte Azure Cosmos DB for NoSQL avec des données

L’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] déploie des exemples de données sur n’importe quel compte Azure Cosmos DB for NoSQL. L’outil est open source et disponible avec NuGet. Vous installez cet outil dans Azure Cloud Shell et l’utilisez pour l’amorçage de votre base de données.

1. Démarrez **Visual Studio Code**.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Si vous n'êtes pas familiarisé avec l’interface de Visual Studio Code, consultez le [Guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre ordinateur.

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

## Afficher et manipuler la stratégie d’indexation par défaut

Lorsqu’un conteneur est créé avec du code, le portail ou un outil, la stratégie d’indexation est définie sur une valeur par défaut intelligente si vous n’en spécifiez aucune. Vous observez cette stratégie d’indexation par défaut et apportez une modification à la stratégie.

1. Revenez à votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne tous les documents où le **name** est équivalent à **HL Headset** :

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats de la requête.

1. Sous l’onglet **Requête**, sélectionnez **Statistiques de requête**.

1. Observez la valeur du champ **Frais de requête** dans la section **Statistiques de requête**.

    > &#128221; Tous les chemins sont actuellement indexés, donc cette requête doit être relativement efficace.

1. Dans le nœud de conteneur **products** de l’arborescence de navigation de l’**API NOSQL**, sélectionnez **Paramètres**.

1. Observez la stratégie d’indexation par défaut dans la section **Stratégie d’indexation** :

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

    > &#128221; Cette stratégie par défaut indexe tous les chemins possibles à l’exception de **_etag**.

1. Dans l’éditeur, remplacez le contenu de la stratégie d’indexation pour indexer uniquement le chemin **/price** :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. Sélectionnez **Enregistrer** pour appliquer vos modifications.

1. Sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne tous les documents où le **name** est équivalent à **HL Headset** :

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats de la requête.

1. Sous l’onglet **Requête**, sélectionnez **Statistiques de requête**.

1. Observez la valeur du champ **Frais de requête** dans la section **Statistiques de requête**.

    > &#128221; Maintenant que la propriété **name** n’est pas indexée, les frais de requête ont augmenté.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne tous les documents où le **prix** est supérieur à **3 000 USD** :

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats de la requête.

1. Sous l’onglet **Requête**, sélectionnez **Statistiques de requête**.

1. Observez la valeur du champ **Frais de requête** dans la section **Statistiques de requête**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
