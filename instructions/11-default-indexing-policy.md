---
lab:
  title: Examiner la stratégie d’index par défaut pour un conteneur Azure Cosmos DB for NoSQL avec le portail
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Examiner la stratégie d’index par défaut pour un conteneur Azure Cosmos DB for NoSQL avec le portail

Chaque conteneur dans Azure Cosmos DB a une stratégie d’indexation qui dirige le service sur la façon d’indexer des éléments dans le conteneur. Par défaut, cette stratégie d’indexation indexe chaque propriété de chaque élément. La stratégie d’indexation par défaut vous permet de commencer à utiliser Azure Cosmos DB de façon simple et rapide, car vous n’avez pas à penser à l’indexation, aux performances et à la gestion au début d’un projet.

Dans ce labo, vous allez observer et manipuler la stratégie d’indexation par défaut pour quelques conteneurs à l’aide de l’Explorateur de données.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge différentes API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Dès que l’approvisionnement du compte Azure Cosmos DB for NoSQL est terminé, vous pouvez récupérer le point de terminaison et la clé, et les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant le kit Azure SDK pour .NET ou un autre SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en laissant tous les autres paramètres avec leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent avoir des restrictions qui vous empêchent de créer un groupe de ressources. Si c’est le cas, utilisez le groupe de ressources pré-créé existant.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée, puis au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus loin dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur **key** plus tard dans cet exercice.


## Alimenter le compte Azure Cosmos DB for NoSQL avec des données

L'outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] déploie des exemples de données sur n'importe quel compte Azure Cosmos DB for NoSQL. L’outil est open source et disponible avec NuGet. Vous installez cet outil dans Azure Cloud Shell et l’utilisez pour l’amorçage de votre base de données.

1. Démarrez **Visual Studio Code**.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Si vous n'êtes pas familiarisé avec l’interface de Visual Studio Code, consultez le [Guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre ordinateur.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```
  
    > &#128161; Cette commande peut prendre quelques minutes. Cette commande affiche le message d’avertissement (*L’outil 'cosmicworks' est déjà installé) si vous aviez déjà installé la dernière version de l’outil.

1. Exécutez cosmicworks pour amorcer votre compte Azure Cosmos DB avec les options de ligne de commande suivantes :

    | **Option** | **Valeur** |
    | ---: | :--- |
    | **--endpoint** | *Valeur de point de terminaison que vous avez copiée précédemment dans ce labo* |
    | **--key** | *Valeur de clé que vous avez copiée plus tôt dans ce labo* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/** et que votre clé est : **fDR2ci9QgkdkvERTQ==**, la commande est : ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Attendez que la commande **cosmicworks** ait fini de remplir le compte avec une base de données, un conteneur et des éléments.

1. Fermez le terminal intégré.

## Afficher et manipuler la stratégie d’indexation par défaut

Lorsqu’un conteneur est créé avec du code, le portail ou un outil, la stratégie d’indexation est définie sur une valeur par défaut intelligente si vous n’en spécifiez aucune. Vous observez cette stratégie d’indexation par défaut et apportez une modification à la stratégie.

1. Revenez à votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne tous les documents où le **nom** équivaut à **HL Headset** :

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats de la requête.

1. Sous l’onglet **Requête**, sélectionnez **Statistiques de requête**.

1. Observez la valeur du champ **Frais de requête** dans la section **Statistiques de requête**.

    > &#128221; Tous les chemins sont actuellement indexés, donc cette requête doit être relativement efficace.

1. Dans le nœud de conteneur **products** de l’arborescence de navigation de l’**API NOSQL**, sélectionnez **Mise à l’échelle et paramètres**.

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

1. Créez une requête SQL qui retourne tous les documents où le **nom** équivaut à **HL Headset** :

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
