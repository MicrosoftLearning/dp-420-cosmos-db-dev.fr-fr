---
lab:
  title: Exécuter une requête avec le SDK Azure Cosmos DB for NoSQL
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Exécuter une requête avec le SDK Azure Cosmos DB for NoSQL

La dernière version du SDK .NET pour Azure Cosmos DB for NoSQL facilite plus que jamais l’interrogation d’un conteneur et l’itération asynchrone sur les jeux de résultats à l’aide des dernières meilleures pratiques et fonctionnalités de langage de C#.

Cette bibliothèque a des fonctionnalités spéciales pour faciliter l’interrogation d’Azure Cosmos DB à l’aide de [https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet].

Dans ce labo, vous allez utiliser un flux asynchrone pour effectuer une itération sur un jeu de résultats volumineux retourné par Azure Cosmos DB for NoSQL. Vous allez utiliser le SDK .NET pour l’interrogation et l’itération sur les résultats.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement utilisé, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted].

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

## Alimenter le compte Azure Cosmos DB for NoSQL avec des données

L’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] déploie des exemples de données sur n’importe quel compte Azure Cosmos DB for NoSQL. L’outil est open source et disponible avec NuGet. Vous installez cet outil dans Azure Cloud Shell et l’utilisez pour l’amorçage de votre base de données.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; L’exécution de cette commande peut prendre quelques minutes. Cette commande génère le message d’avertissement (*L’outil « cosmicworks » est déjà installé), si vous avez déjà installé la dernière version de cet outil.

1. Exécutez cosmicworks pour remplir initialement votre compte Azure Cosmos DB avec les options de ligne de commande suivantes :

    | **Option** | **Valeur** |
    | ---: | :--- |
    | **--endpoint** | *Valeur de point de terminaison que vous avez copiée plus tôt dans ce labo* |
    | **--key** | *Valeur de clé que vous avez copiée plus tôt dans ce labo* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Par exemple, si votre point de terminaison est **https&shy;://dp420.documents.azure.com:443/** et si votre clé est **fDR2ci9QgkdkvERTQ==**, la commande est ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Attendez que la commande **cosmicworks** ait fini de remplir le compte avec une base de données, un conteneur et des éléments.

1. Fermez le terminal intégré.

## Interroger les résultats d’une requête SQL à l’aide du kit de développement logiciel

Vous allez à présent utiliser un flux asynchrone pour créer une boucle foreach simple à comprendre sur les résultats paginés d’Azure Cosmos DB. En arrière-plan, le SDK gère l’itérateur de flux et vérifie que les requêtes suivantes sont appelées correctement.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **09-execute-query-sdk**.

1. Ouvrez le fichier de code **product.cs**.

1. Observez la classe **Product** et ses propriétés correspondantes. Plus précisément, ce labo utilisera les propriétés **id**, **name** et **price**.

1. De retour dans le volet **Explorateur** de **Visual Studio Code**, ouvrez le fichier de code **script.cs**.

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

1. Ajoutons du code supplémentaire à la fin du fichier **script.cs**. Créez une variable nommée **sql** de type *string* avec la valeur **SELECT * FROM products p** :

    ```
    string sql = "SELECT * FROM products p";
    ```

1. Créez une variable de type [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] en transmettant la variable **sql** en tant que paramètre au constructeur :

    ```
    QueryDefinition query = new (sql);
    ```

1. Créez une boucle **while** en appelant la méthode générique [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] de la classe [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] en transmettant la variable **query** en tant que paramètre, puis en itérant sur les résultats :

    ```
    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
    }
    ```

1. Dans la boucle **while**, lisez le résultat suivant de manière asynchrone, puis utilisez la méthode statique intégrée **Console.WriteLine** pour mettre en forme et imprimer les propriétés **id**, **name** et **price** de la variable **product** :

    ```
    FeedResponse<Product> response = await feed.ReadNextAsync();
    foreach (Product product in response)
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
        FeedResponse<Product> response = await feed.ReadNextAsync();
        foreach (Product product in response)
        {
            Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
        }
    }
    ```

1. **Enregistrez** le fichier **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **09-execute-query-sdk**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Créez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Le script génère à présent chaque produit dans le conteneur

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet]: https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
