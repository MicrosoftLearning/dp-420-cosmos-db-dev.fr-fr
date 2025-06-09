---
lab:
  title: Traiter par lots plusieurs opérations de point avec le kit SDK Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Traiter par lots plusieurs opérations de point avec le kit SDK Azure Cosmos DB for NoSQL

Les classes [TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] et [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] sont ensemble la clé de la composition et de la décomposition des opérations en une seule étape logique. À l’aide de ces classes, vous pouvez écrire votre code pour effectuer plusieurs opérations, puis déterminer si elles ont été effectuées avec succès côté serveur.

Dans ce labo, vous utilisez le kit de développement logiciel (SDK) pour effectuer deux opérations à deux éléments dans lesquels vous tentez de créer deux éléments en tant qu’unité logique unique.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement utilisé, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas déjà familiarisé avec l’interface Visual Studio Code, consultez la [documentation de prise en main][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git: Clone** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL et configurer le projet SDK

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au Portail Azure (``portal.azure.com``).

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

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur **key** plus tard dans cet exercice.

1. Revenez à **Visual Studio Code**.

1. Dans le volet **Explorateur**, accédez au dossier **07-sdk-batch**.

1. Ouvrez le fichier de code **script.cs** dans le dossier **07-sdk-batch**.

    > &#128221; La bibliothèque **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** a déjà été préimportée à partir de NuGet.

1. Localisez la variable **string** nommée **endpoint**. Définissez sa valeur sur le **point de terminaison** du compte Azure Cosmos DB créé précédemment.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est **https&shy;://dp420.documents.azure.com:443/**, l’instruction C# est **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Localisez la variable **string** nommée **key**. Définissez sa valeur sur la **clé** du compte Azure Cosmos DB créé précédemment.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est **fDR2ci9QgkdkvERTQ==**, l’instruction C# est **string key = "fDR2ci9QgkdkvERTQ==";**.

1. **Enregistrez** le fichier de code **script.cs**.

1. Ouvrez le menu contextuel du dossier **07-sdk-batch**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **07-sdk-batch**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] depuis NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Générez le projet en utilisant la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

1. Fermez le terminal intégré.

## Création d’un lot transactionnel

Tout d’abord, nous allons créer un lot transactionnel simple qui crée deux produits fictifs. Ce lot insère une selle usée et un guidon rouillé dans le conteneur avec le même identificateur de catégorie « accessoires utilisés ». Les deux éléments ont la même clé de partition logique, ce qui garantit que nous aurons une opération de traitement par lots réussie.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Créez une variable **Product** nommée **saddle** avec un identificateur unique de **0120**, un nom de **Worn Saddle** et un identificateur de catégorie de **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Créez une variable **Product** nommée **handlebar** avec un identificateur unique **012A**, un nom de **Rusty Handlebar**et un identificateur de catégorie de **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Créez une variable de type **PartitionKey** nommée **partitionKey** en passant **9603ca6c-9e28-4a02-9194-51cdb7fea816** en tant que paramètre de constructeur :

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Appelez la méthode [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] de la variable **container** en passant la variable **partitionkey** en tant que paramètre de méthode et en utilisant la syntaxe Fluent pour appeler les méthodes génériques[CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] en passant les variables **saddle** et **handlebar** en tant qu’éléments à créer dans des opérations individuelles et stockez le résultat dans une variable nommée **batch** de type **TransactionalBatch** :

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. Dans une instruction using, appelez de façon asynchrone la méthode **ExecuteAsync** de la variable **batch** et stockez le résultat dans une variable de type **TransactionalBatchResponse** nommée **response** :

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Appelez la méthode statique **Console.WriteLine** pour générer la valeur de la propriété **StatusCode** de la variable **response** :

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Une fois que vous avez terminé, votre fichier de code doit maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **07-sdk-batch**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**  :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Le code d’état doit être **OK**.

1. Fermez le terminal intégré.

## Création d’un lot transactionnel errant

À présent, nous allons créer un lot transactionnel qui génère une erreur délibérée. Ce lot tente d’insérer deux éléments qui ont des clés de partition logique différentes. Nous allons créer une lampe stroboscopique qui clignote dans la catégorie « accessoires utilisés » et un nouveau casque dans la catégorie « accessoires qualités ». Par définition, il devrait s’agir d'une mauvaise requête et une erreur devrait être renvoyée lors de l’exécution de cette transaction.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Supprimez les lignes de code suivantes :

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Créez une variable **Product** nommée **light** avec un identificateur unique de **012B**, un nom de **Flickering Strobe Light** et un identificateur de catégorie de **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Créez une variable **Product** nommée **helmet** avec un identificateur unique de **012C**, un nom de **New Helmet** et un identificateur de catégorie de **0feee2e4-687a-4d69-b64e-be36afc33e74** :

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. Créez une variable de type **PartitionKey** nommée **partitionKey** en passant **9603ca6c-9e28-4a02-9194-51cdb7fea816** en tant que paramètre de constructeur :

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Appelez la méthode **CreateTransactionalBatch** de la variable **container** en passant la variable **partitionkey** en tant que paramètre de méthode et en utilisant la syntaxe Fluent pour appeler les méthodes génériques**CreateItem<>** en passant les variables **light** et **helmet** en tant qu’éléments à créer dans des opérations individuelles et stockez le résultat dans une variable nommée **batch** de type **TransactionalBatch** :

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. Dans une instruction using, appelez de façon asynchrone la méthode **ExecuteAsync** de la variable **batch** et stockez le résultat dans une variable de type **TransactionalBatchResponse** nommée **response** :

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Appelez la méthode statique **Console.WriteLine** pour générer la valeur de la propriété **StatusCode** de la variable **response** :

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Une fois que vous avez terminé, votre fichier de code doit maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **07-sdk-batch**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**  :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Le code d’état doit être **Demande incorrecte** ou **Conflit**. Cela s’est produit parce que tous les éléments de la transaction ne partage pas la même valeur de clé de partition que le lot transactionnel.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
