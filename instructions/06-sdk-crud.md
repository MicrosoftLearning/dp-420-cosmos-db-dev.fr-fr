---
lab:
  title: Créer et mettre à jour des documents avec le Kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Créer et mettre à jour des documents avec le Kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL

La classe [Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] comprend un ensemble de méthodes membres pour créer, récupérer, mettre à jour et supprimer des éléments dans un conteneur Azure Cosmos DB for NoSQL. Ensemble, ces méthodes permettent d’effectuer certaines des opérations « CRUD » les plus courantes sur divers éléments dans des conteneurs d’API NoSQL.

Dans ce labo, vous allez utiliser le kit SDK pour effectuer des opérations CRUD courantes sur un élément dans un conteneur Azure Cosmos DB for NoSQL.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement utilisé, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas déjà familiarisé avec l’interface Visual Studio Code, consultez la [documentation de prise en main][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git: Clone** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

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

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur de **clé** plus tard dans cet exercice.

1. Revenez à **Visual Studio Code**.

## Se connecter au compte Azure Cosmos DB for NoSQL à partir du kit de développement logiciel

À l’aide des informations d’identification du compte récemment créé, vous allez vous connecter aux classes du kit SDK et créer une base de données et une instance de conteneur. Ensuite, vous utiliserez l’Explorateur de données pour valider l’existence des instances dans le Portail Azure.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **06-sdk-crud**.

1. Ouvrez le menu contextuel du dossier **06-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **06-sdk-crud**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Générez le projet en utilisant la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

1. Fermez le terminal intégré.

1. Ouvrez le fichier de code **script.cs** dans le dossier **06-sdk-crud**.

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

1. Appelez de façon asynchrone la méthode CreateDatabaseIfNotExistsAsync de la variable **client** en passant le nom de la nouvelle base de données (**cosmicworks**) que vous souhaitez créer et en stockant le résultat dans une variable de type **Database** :

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Appelez de façon asynchrone la méthode **CreateContainerIfNotExistsAsync** de la variable **database** en passant le nom du nouveau conteneur (**products**), le chemin de la clé de partition (**/categoryId**) et le débit (**400**) que vous souhaitez créer dans la base de données **cosmicworks** et en stockant le résultat dans une variable de type **Container** :
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **06-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Fermez le terminal intégré.

1. Basculez vers la fenêtre de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

## Effectuer des opérations de création et de lecture de point sur des éléments avec le kit de développement logiciel

Vous allez maintenant utiliser l’ensemble de méthodes asynchrones dans la classe Microsoft.Azure.Cosmos.Container pour effectuer des opérations courantes sur des éléments au sein d’un conteneur d’API NoSQL. Ces opérations sont toutes effectuées à l’aide du modèle de programmation asynchrone de tâche en C#.

1. Revenez à **Visual Studio Code**. Ouvrez le fichier de code **product.cs** dans le dossier **06-sdk-crud**.

    > &#128221; Ne fermez pas l’éditeur du fichier **script.cs**.

1. Observez la classe **Product** dans ce fichier de code. Cette classe représente un élément de produit qui sera stocké et manipulé dans ce conteneur.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Créez un objet de type **Product** nommé **saddle** avec les propriétés suivantes :

    | Propriété | Valeur |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **p**rice | *45.99d* |
    | **balises** | *{ tan, new, crisp }* |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. Appelez de façon asynchrone la méthode générique [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] de la variable **container** en passant la variable **saddle** en tant que paramètre de méthode et en utilisant **Product** comme type générique :

    ```
    await container.CreateItemAsync<Product>(saddle);
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **06-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** :

    ```
    dotnet run
    ```

1. Fermez le terminal intégré.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Supprimez les lignes de code suivantes :

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. Créez une variable string nommée **id** avec la valeur **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** :

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Créez une variable string nommée **categoryId** avec la valeur **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Créez une variable de type [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] nommée **partitionKey** en passant la variable **categoryId** en tant que paramètre de constructeur :

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Appelez de façon asynchrone la méthode générique [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] de la variable **container** en passant les variables **id** et **partitionkey** en tant que paramètres de méthode, en utilisant **Product** comme type générique et en stockant le résultat dans une variable nommée **saddle** de type **Product** :

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Appelez la méthode statique **Console.WriteLine** pour afficher l’objet saddle à l’aide d’une chaîne de sortie mise en forme :

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **06-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Plus précisément, observez le texte de sortie mis en forme avec l’ID, le nom et le prix de l’élément.

1. Fermez le terminal intégré.

## Effectuer des opérations de mise à jour et de suppression de point avec le kit de développement logiciel

Dans le cadre de votre apprentissage du kit SDK, il n’est pas rare d’utiliser un compte SDK Azure Cosmos DB en ligne ou l’émulateur pour mettre à jour un élément, puis de basculer successivement entre l’Explorateur de données et votre IDE de choix pour effectuer une opération et vérifier si la modification a été appliquée. C’est exactement ce que vous allez faire ici en mettant à jour et en supprimant un élément à l’aide du kit SDK.

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Sélectionnez le seul élément dans le conteneur, puis observez les valeurs des propriétés **name** et **price** de l’élément.

    | **Propriété** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *Road Saddle* |
    | **Tarif** | *$45.99* |

    > &#128221; À ce stade, ces valeurs ne doivent pas avoir été modifiées depuis la création de l’élément. Vous allez modifier ces valeurs dans cet exercice.

1. Revenez à **Visual Studio Code**. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Supprimez les lignes de code suivantes :

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. Modifiez la variable **saddle** en définissant la valeur de la propriété price sur **32.55** :

    ```
    saddle.price = 32.55d;
    ```

1. Modifiez à nouveau la variable **saddle** en définissant la valeur de la propriété **name** sur **Road LL Saddle** :

    ```
    saddle.name = "Road LL Saddle";
    ```

1. Appelez de façon asynchrone la méthode générique [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] de la variable **container** en passant la variable **saddle** en tant que paramètre de méthode et en utilisant **Product** comme type générique :

    ```
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **06-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** :

    ```
    dotnet run
    ```

1. Fermez le terminal intégré.

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Sélectionnez le seul élément dans le conteneur, puis observez les valeurs des propriétés **name** et **price** de l’élément.

    | **Propriété** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *Road LL Saddle* |
    | **Tarif** | *$32.55* |

    > &#128221; À ce stade, ces valeurs doivent être différentes de celles que vous avez observées précédemment.

1. Revenez à **Visual Studio Code**. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Supprimez les lignes de code suivantes :

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. Appelez de façon asynchrone la méthode générique [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] de la variable **container** en passant les variables **id** et **partitionkey** en tant que paramètres de méthode et en utilisant **Product** comme type générique :

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **06-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** :

    ```
    dotnet run
    ```

1. Fermez le terminal intégré.

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Notez que la liste des éléments est désormais vide.

1. Fermez la fenêtre ou l’onglet de votre navigateur web.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
