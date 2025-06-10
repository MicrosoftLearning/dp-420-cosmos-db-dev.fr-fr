---
lab:
  title: Paginer les résultats des requêtes multi-produits avec le SDK Azure Cosmos DB for NoSQL
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Paginer les résultats des requêtes multi-produits avec le SDK Azure Cosmos DB for NoSQL

Les requêtes Azure Cosmos DB ont généralement plusieurs pages de résultats. La pagination est effectuée automatiquement côté serveur quand Azure Cosmos DB ne peut pas retourner tous les résultats de requête en une seule exécution. Dans de nombreuses applications, vous souhaiterez écrire du code à l’aide du Kit de développement logiciel (SDK) pour traiter vos résultats de requête de manière performante.

Dans ce labo, vous allez créer un itérateur de flux qui peut être utilisé dans une boucle pour itérer sur l’ensemble de votre jeu de résultats.

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

1. Revenez à **Visual Studio Code**.

## Alimenter le compte Azure Cosmos DB for NoSQL avec des données

L’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] déploie des exemples de données sur n’importe quel compte Azure Cosmos DB for NoSQL. L’outil est open source et disponible avec NuGet. Vous installez cet outil dans Azure Cloud Shell et l’utilisez pour l’amorçage de votre base de données.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

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

## Paginer via de petits jeux de résultats d’une requête SQL à l’aide du Kit de développement logiciel (SDK)

Lors du traitement des résultats de requête, vous devez vous assurer que votre code progresse dans toutes les pages de résultats et de vérifications pour voir si d’autres pages sont restantes avant d’effectuer des requêtes ultérieures.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **10-paginate-results-sdk**.

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

    > &#128221; Par exemple, si votre clé est : **fDR2ci9QgkdkvERTQ==**, alors l'instruction C# serait : **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Créez une nouvelle variable nommée **sql** de type *string* avec une valeur de **SELECT p.id, p.name, p.price FROM products p** :

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. Créez une nouvelle variable de type [QueryDefinition][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition] en passant la variable **sql** en paramètre au constructeur :

    ```
    QueryDefinition query = new (sql);
    ```

1. Créez une nouvelle variable de type [QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] nommée **options** à l'aide du constructeur vide par défaut :

    ```
    QueryRequestOptions options = new ();
    ```

1. Définissez la propriété [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] de la variable **options** sur une valeur de **50** :

    ```
    options.MaxItemCount = 50;
    ```

1. Créez une nouvelle variable nommée **itérateur** de type [FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1] en appelant la méthode générique [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] de la classe [Récipient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] en passant les variables de **requête** et d'**options** en tant que paramètres :

    ```
    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);
    ```

1. Créez un boucle **pendant** qui vérifie la propriété [HasMoreResults][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults] de la variable d’**itérateur** :

    ```
    while (iterator.HasMoreResults)
    {
        
    }
    ```

1. Dans le boucle **pendant que**, appelez de manière asynchrone la méthode [ReadNextAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync] de la variable d’**itérateur** stockant le résultat dans une variable nommée **produits** de type générique [FeedResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1] à l’aide de la classe **Produit** :

    ```
    FeedResponse<Product> products = await iterator.ReadNextAsync();
    ```

1. Toujours dans la boucle **pendant**, créez une nouvelle boucle **foreach** en itérant sur la variable **produits** en utilisant la variable **produit** pour représenter une instance de type **Produit** :

    ```
    foreach (Product product in products)
    {

    }
    ```

1. Dans la boucle **foreach**, utilisez la méthode statique intégrée **Console.WriteLine** pour formater et imprimer les propriétés **id**, **nom** et **prix** de la variable **produit** :

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. De retour dans la boucle **pendant**, utilisez la méthode statique intégrée **Console.WriteLine** pour imprimer le message. *Appuyez sur n'importe quelle touche pour obtenir plus de résultats* :

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. Toujours dans la boucle **pendant**, utilisez la méthode statique intégrée **CConsole.ReadKey** pour écouter la prochaine saisie de touche :

    ```
    Console.ReadKey();
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    QueryDefinition query = new (sql);

    QueryRequestOptions options = new ();
    options.MaxItemCount = 50;

    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);

    while (iterator.HasMoreResults)
    {
        FeedResponse<Product> products = await iterator.ReadNextAsync();
        foreach (Product product in products)
        {
            Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
        }

        Console.WriteLine("Press any key for next page of results");
        Console.ReadKey();        
    }
    ```

1. **Enregistrer** le fichier **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **10-paginate-results-sdk**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez et exécutez le projet à l'aide de la commande [exécution dotnet][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Le script génère désormais le premier ensemble de 50 éléments qui correspondent à la requête. Appuyez sur n’importe quelle touche pour obtenir le jeu suivant de 50 éléments jusqu’à ce que la requête ait itéré sur tous les éléments correspondants.

    > &#128161; La requête correspondra à des centaines d’éléments dans le conteneur de produits.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
