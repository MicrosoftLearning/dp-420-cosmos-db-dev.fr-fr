---
lab:
  title: Configurer la stratégie d’indexation d’un conteneur Azure Cosmos DB for NoSQL avec le kit de développement logiciel (SDK)
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Configurer la stratégie d’indexation d’un conteneur Azure Cosmos DB for NoSQL avec le kit de développement logiciel (SDK)

Les stratégies d’indexation peuvent être gérées à partir de n’importe quel SDK Azure Cosmos DB. Le SDK .NET contient en particulier un ensemble de classes qui peuvent être utilisées pour concevoir et envoyer (push) une nouvelle stratégie d’indexation à un conteneur dans Azure Cosmos DB for NoSQL.

Dans ce labo, vous allez créer une stratégie d’indexation personnalisée pour un conteneur à l’aide du SDK .NET

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement utilisé, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git : Cloner** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple l’**API Mongo** ou l’**API NoSQL**). Une fois l’approvisionnement du compte Azure Cosmos DB for NoSQL effectué, vous pouvez récupérer le point de terminaison et la clé, puis les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant le kit Azure SDK pour .NET ou tout autre kit SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
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

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur **key** plus tard dans cet exercice.

1. Revenez à **Visual Studio Code**.

## Créer une stratégie d’indexation à l’aide du SDK .NET

Le SDK .NET contient une suite de classes liées à la classe parente [Microsoft.Azure.Cosmos.IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] pour générer des stratégies d’indexation dans le code.

1. Dans le volet **Explorateur**, accédez au dossier **12-custom-index-policy**.

1. Ouvrez le fichier de code **script.cs**.

1. Mettez à jour la variable existante nommée **endpoint** avec sa valeur définie sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est **https&shy;://dp420.documents.azure.com:443/**, l’instruction C# est **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Mettez à jour la variable existante nommée **key** avec sa valeur définie sur la **clé** du compte Azure Cosmos DB que vous avez créé précédemment.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est **fDR2ci9QgkdkvERTQ==**, l’instruction C# est **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Créez une variable de type [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] nommée **policy** à l’aide du constructeur vide par défaut :

    ```
    IndexingPolicy policy = new ();
    ```

1. Définissez la propriété [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode] de la variable **policy** sur une valeur de [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields] :

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. Ajoutez un nouvel objet de type [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] avec sa propriété [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] définie sur une valeur de **/*** à la propriété de collection [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] dans la variable **policy** :

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. Ajoutez un nouvel objet de type [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] avec sa propriété [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] définie sur la valeur de **/name/?** à la propriété de collection [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] de la variable **policy** :

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. Créez une variable de type [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] nommée **options** qui transmet les valeurs ``products`` et ``/categoryId`` en tant que paramètres de constructeur :

    ```
    ContainerProperties options = new ("products", "/categoryId");
    ```

1. Attribuez la variable **policy** à la propriété [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] de la variable **options** :

    ```
    options.IndexingPolicy = policy;
    ```

1. Appelez de manière asynchrone la méthode [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] de la variable **database** en transmettant la variable **options** en tant que paramètre de constructeur et en stockant le résultat dans une variable de type [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] nommée **container** :

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. Utilisez la méthode statique intégrée **Console.WriteLine** pour imprimer la propriété [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] de la classe Container avec un en-tête intitulé **Container Created** :

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. Une fois que vous avez terminé, votre fichier de code doit maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    IndexingPolicy policy = new ();
    policy.IndexingMode = IndexingMode.Consistent;
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );

    ContainerProperties options = new ("products", "/categoryId");
    options.IndexingPolicy = policy;

    Container container = await database.CreateContainerIfNotExistsAsync(options);
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. **Enregistrez** le fichier **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **12-custom-index-policy**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez et exécutez le projet avec la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Le script génère désormais le nom du nouveau conteneur :

    ```
    Container Created [products]
    ```

1. Fermez le terminal intégré.

1. Revenez à votre navigateur web.

## Observer une stratégie d’indexation créée par le SDK .NET à l’aide de l’Explorateur de données

Comme avec n’importe quelle autre stratégie d’indexation, vous pouvez utiliser l’Explorateur de données pour afficher les stratégies que vous avez envoyées à l’aide des SDK .NET. Vous allez à présent utiliser le portail pour passer en revue la stratégie que vous avez créée dans ce labo à partir du code.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **products** dans l’arborescence de l’**API NOSQL**.

1. Dans le nœud de conteneur **products** de l’arborescence de navigation de l’**API NOSQL**, sélectionnez **Mise à l’échelle et paramètres**.

1. Observez la stratégie d’indexation dans la section **Stratégie d’indexation** :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; Il s’agit de la représentation JSON de la stratégie d’indexation que vous avez créée à l’aide du SDK .NET dans ce labo.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
