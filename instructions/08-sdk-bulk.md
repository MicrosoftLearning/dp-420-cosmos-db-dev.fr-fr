---
lab:
  title: Déplacer plusieurs documents en bloc avec le Kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Déplacer plusieurs documents en bloc avec le Kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL

Le moyen le plus simple d’apprendre à effectuer une opération en bloc consiste à tenter d’envoyer (push) de nombreux documents à un compte Azure Cosmos DB for NoSQL dans le cloud. Pour cela, vous pouvez utiliser les fonctionnalités en bloc du kit SDK en vous aidant un peu de l’espace de noms [System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks].

Dans ce labo, vous allez utiliser la bibliothèque [Bogus][nuget.org/packages/bogus/33.1.1] de NuGet pour générer des données fictives et les placer dans un compte Azure Cosmos DB.

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

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur de **clé** plus tard dans cet exercice.

1. Toujours dans la ressource de compte **Azure Cosmos DB** récemment créée, accédez au volet **Explorateur de données**.

1. Dans **Explorateur de données**, sélectionnez **Nouveau conteneur**, puis créez un conteneur avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **ID de base de données** | *Créer nouveau* &vert; *`cosmicworks`* |
    | **Partager le débit entre les conteneurs** | *Ne pas sélectionner* |
    | **ID de conteneur** | *`products`* |
    | **Clé de partition** | *`/categoryId`* |
    | **Débit du conteneur** | *Mise à l’échelle automatique* &vert; *`4000`* |

1. Revenez à **Visual Studio Code**.

1. Dans le volet **Explorateur**, accédez au dossier **08-sdk-bulk**.

1. Ouvrez le fichier de code **script.cs** dans le dossier **08-sdk-bulk**.

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

1. Ouvrez le menu contextuel du dossier **08-sdk-bulk**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **08-sdk-bulk**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] à partir de NuGet à l’aide de la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Générez le projet à l’aide de la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

1. Fermez le terminal intégré.

## Insérer en bloc vingt-cinq mille documents

Allons-y franchement et essayons d’insérer un grand nombre de documents pour voir comment cela fonctionne. Dans nos tests internes, cela peut prendre environ 1 à 2 minutes si la machine virtuelle du labo et le compte Azure Cosmos DB for NoSQL sont relativement proches les uns des autres sur le plan géographique.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Créez une instance de la classe [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] nommée **options** en définissant la propriété **AllowBulkExecution** sur la valeur **true** :

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. Créez une instance de la classe **CosmosClient** nommée **client** en passant les variables **endpoint**, **key** et **options** en tant que paramètres de constructeur :

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. Utilisez la méthode [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] de la variable **client** pour récupérer le conteneur existant en utilisant le nom de la base de données (*cosmicworks*) et le nom du conteneur (*products*) :

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Utilisez cet exemple de code spécial pour générer **25 000** produits fictifs à l’aide de la classe **Faker** de la bibliothèque Bogus importée à partir de NuGet.

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; La bibliothèque [Bogus][nuget.org/packages/bogus/33.1.1] est une bibliothèque open source permettant de concevoir des données fictives afin de tester des applications avec une interface utilisateur. Elle est idéale pour apprendre à développer des applications d’importation/exportation en bloc.

1. Créez une liste (**List<>**) générique de type **Task** nommée **concurrentTasks** :

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. Créez une boucle foreach qui sera itérée sur la liste des produits générée précédemment dans cette application :

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. Dans la boucle foreach, créez une tâche (**Task**) pour insérer de manière asynchrone un produit dans Azure Cosmos DB for NoSQL, en veillant à spécifier explicitement la clé de partition et à ajouter la tâche à la liste des tâches nommée **concurrentTasks** :

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. Après la boucle foreach, attendez de façon asynchrone le résultat de **Task.WhenAll** sur la variable **concurrentTasks** :

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. Utilisez la méthode statique **Console.WriteLine** intégrée pour afficher le message statique **Bulk tasks complete**(Tâches en bloc terminées) sur la console :

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **08-sdk-bulk**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** :

    ```
    dotnet run
    ```

1. L’application doit s’exécuter en mode silencieux. Après environ une à deux minutes, elle doit se terminer en mode silencieux.

1. Fermez le terminal intégré.

## Observer les résultats

Maintenant que vous avez envoyé 25 000 éléments à Azure Cosmos DB, examinons l’Explorateur de données.

1. Revenez au navigateur web et accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Développez le nœud **products**, puis sélectionnez le nœud **Items**. Observez la liste d’éléments dans votre conteneur.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NoSQL**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne le nombre de documents créés à l’aide de l’opération en bloc :

    ```
    SELECT COUNT(1) FROM items
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez le nombre d’éléments dans votre conteneur.

1. Fermez la fenêtre ou l’onglet de votre navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
