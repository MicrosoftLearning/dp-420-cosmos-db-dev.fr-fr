---
lab:
  title: Configurer le SDK Azure Cosmos DB for NoSQL pour le développement hors connexion
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Configurer le SDK Azure Cosmos DB for NoSQL pour le développement hors connexion

L’émulateur Azure Cosmos DB est un outil local qui émule le service Azure Cosmos DB à des fins de développement et de test. L’émulateur prend en charge NoSQL et peut être utilisé à la place du service cloud lors du développement de code à l’aide du kit Azure SDK pour .NET.

Dans ce labo, vous allez vous connecter à l’émulateur Azure Cosmos DB à partir du kit Azure SDK pour .NET.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le dépôt de code du labo pour le cours **DP-420** dans l’environnement où vous travaillez, suivez ces étapes. Sinon, ouvrez le dossier cloné précédemment dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas déjà familiarisé avec l’interface Visual Studio Code, consultez la [Documentation de prise en main][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git: Clone** pour cloner le dépôt GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans le dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl+Maj+P** pour ouvrir la palette de commandes.

1. Une fois le dépôt cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Démarrer l’émulateur Azure Cosmos DB

Votre environnement doit déjà avoir l’émulateur préinstallé. Dans le cas contraire, reportez-vous aux [Instructions d’installation][docs.microsoft.com/azure/cosmos-db/local-emulator] pour installer l’émulateur Azure Cosmos DB. Une fois l’émulateur démarré, vous pouvez récupérer la chaîne de connexion et l’utiliser pour vous connecter à l’émulateur à l’aide du kit Azure SDK pour .NET ou d’un autre SDK de votre choix.

1. Démarrez **l’émulateur Azure Cosmos DB**.

    > &#128221; Vous pouvez être invité à accorder à l’administrateur l’accès pour démarrer l’émulateur. Dans l’environnement du labo, le compte **Administrateur** a le même mot de passe que le compte **Stagiaire**.

    > &#128161; L’émulateur Azure Cosmos DB est épinglé à la fois à la barre des tâches Windows et au menu Démarrer. ***Si l’émulateur ne démarre pas à partir des icônes épinglées, essayez de l’ouvrir en double-cliquant sur le fichier*** **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe********. Notez que l’émulateur prend 20 à 30 secondes pour démarrer.

1. Attendez que l’émulateur ouvre automatiquement votre navigateur par défaut et accédez à la page d’accueil **localhost:8081/_explorer/index.html**.

1. Dans la page d’accueil **Émulateur Azure Cosmos DB**, accédez au volet **Démarrage rapide**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Plus précisément :

    > &#128221; Remarquez le champ **Chaîne de connexion principale**. Vous utiliserez cette valeur de **chaîne de connexion** plus loin dans cet exercice.

1. Accédez au volet **Explorateur**.

1. Dans l’**Explorateur de données**, remarquez qu’il n’existe aucun nœud dans l’arborescence de navigation de l’**API NoSQL**.

1. Laissez cet onglet ouvert et basculez vers **Visual Studio Code**.

## Se connecter à l’émulateur à partir du Kit de développement logiciel (SDK)

La bibliothèque **Microsoft.Azure.Cosmos** a déjà été préinstallée dans le script .NET que vous allez utiliser dans cet exercice. De plus, une partie du code réutilisable a déjà été écrite pour vous faire gagner du temps. Vous devez mettre à jour la valeur de la chaîne de connexion réutilisable et écrire quelques lignes de code pour terminer le script.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **05-sdk-offline**.

1. Ouvrez le fichier de code **script.cs** dans le dossier **05-sdk-offline**.

1. Mettez à jour la variable existante nommée **connectionString** avec sa valeur définie sur la **chaîne de connexion** de l’émulateur Azure Cosmos DB.
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; L’URI de l’émulateur est généralement ***localhost:[port]*** en utilisant SSL avec le port par défaut défini sur **8081**.

    > &#128221; *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* est la clé par défaut pour toutes les installations de l’émulateur. Cette clé peut être modifiée à l’aide des options de ligne de commande.

1. Appelez de manière asynchrone la méthode [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] de la variable **client** en passant le nom de la nouvelle base de données (**cosmoworks**) que vous souhaitez créer dans l’émulateur et en stockant le résultat dans une variable de type [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] :

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Utilisez la méthode statique intégrée **Console.WriteLine** pour imprimer la propriété [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] de la classe Database avec un en-tête intitulé **New Database** :

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **05-sdk-offline**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **05-sdk-offline**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] depuis NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Créez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Fermez le terminal intégré.

## Afficher les modifications dans l’émulateur

Maintenant que vous avez créé une base de données dans l’émulateur Azure Cosmos DB, vous allez utiliser l’**Explorateur de données** en ligne pour observer la nouvelle base de données de l’API NoSQL dans l’émulateur.

1. Revenez à votre navigateur.

1. Dans la page d’accueil **Émulateur Azure Cosmos DB**, accédez au volet **Explorateur**.

1. Dans l’**Explorateur de données**, actualisez l’**API NoSQL** pour observer le nouveau nœud de base de données **cosmicworks** dans l’arborescence de navigation.

1. Revenez à **Visual Studio Code**.

## Créer et afficher un nouveau conteneur

La création d’un conteneur est similaire au modèle utilisé pour créer une base de données. Le code que vous découvrez ici est pertinent que vous créiez ou non des ressources dans le cloud ou dans l’émulateur, vous avez juste besoin de modifier la chaîne de connexion. Vous allez ensuite développer le fichier de script pour créer un conteneur avec la base de données.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **05-sdk-offline**.

1. Ouvrez une nouvelle fois le fichier de code **script.cs** dans le dossier **05-sdk-offline**.

1. Appelez de façon asynchrone la méthode [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] de la variable **database** en passant le nom du nouveau conteneur (**products**), le chemin de la clé de partition (**/categoryId**) et le débit (**400**) que vous souhaitez créer dans la base de données **cosmicworks** et en stockant le résultat dans une variable de type [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] :

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. Utilisez la méthode statique intégrée **Console.WriteLine** pour imprimer la propriété [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] de la classe Container avec un en-tête intitulé **New Container** :

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **05-sdk-offline**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

1. Basculez vers votre navigateur.

1. Dans la page d’accueil **Émulateur Azure Cosmos DB**, accédez au volet **Explorateur**.

1. Dans l’**Explorateur de données**, actualisez l’**API SQL** pour observer le nouveau nœud de conteneur **products** dans le nœud de base de données **cosmicworks**.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

## Arrêter l’émulateur Azure Cosmos DB

Il est important d’arrêter l’émulateur lorsque vous avez terminé de l’utiliser, car il peut utiliser des ressources système dans votre environnement. Utilisez l’icône de la barre d’état système pour arrêter l’émulateur et toutes les instances en cours d’exécution.

 Accédez à l’icône de l’émulateur dans la barre d’état système Windows, ouvrez le menu contextuel, puis sélectionnez **Quitter** pour arrêter l’émulateur.

> &#128221; Toutes les instances de l’émulateur peuvent mettre une minute à se fermer.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
