---
lab:
  title: Paginer les résultats des requêtes multi-produits avec le SDK Azure Cosmos DB for NoSQL
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Paginer les résultats des requêtes multi-produits avec le SDK Azure Cosmos DB for NoSQL

Les requêtes Azure Cosmos DB ont généralement plusieurs pages de résultats. La pagination est effectuée automatiquement côté serveur quand Azure Cosmos DB ne peut pas retourner tous les résultats de requête en une seule exécution. Dans de nombreuses applications, vous souhaiterez écrire du code à l’aide du Kit de développement logiciel (SDK) pour traiter vos résultats de requête de manière performante.

Dans ce labo, vous allez créer un itérateur de flux qui peut être utilisé dans une boucle pour itérer sur l’ensemble de votre jeu de résultats.

## Préparer votre environnement de développement

Si vous n'avez pas encore cloné le référentiel de code de laboratoire pour le **DP-420** dans l'environnement dans lequel vous travaillez sur cet atelier, suivez ces étapes pour ce faire. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n'êtes pas déjà familier avec l'interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git : Clonez** pour cloner le dépôt GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **CTRL+SHIFT+P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Lors du provisionnement d’un compte Azure Cosmos DB pour la première fois, vous sélectionnerez les API que vous souhaitez que le compte prenne en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Une fois le provisionnement du compte Azure Cosmos DB pour NoSQL terminé, vous pouvez récupérer le point de terminaison et la clé et les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL à l’aide du SDK Azure pour .NET ou de tout autre SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au Portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une nouvelle **ressource de compte Azure Cosmos DB for NoSQL** avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou créer un groupe de ressources* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de laboratoire peuvent avoir des restrictions vous empêchant de créer un nouveau groupe de ressources. Si tel est le cas, utilisez le groupe de ressources pré-créé existant.

1. Attendez que la tâche de déploiement se termine avant de poursuivre cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Clés**.

1. Ce volet contient les informations de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du Kit de développement logiciel (SDK). Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur de **point de terminaison** plus tard dans cet exercice.

    1. Notez le champ **PRIMARY KEY**. Vous utiliserez cette valeur **clé** plus tard dans cet exercice.

1. Revenez à **Visual Studio Code**.

## Alimenter le compte Azure Cosmos DB for NoSQL avec des données

L'outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] déploie des exemples de données sur n'importe quel compte Azure Cosmos DB for NoSQL. L’outil est open source et disponible via NuGet. Vous allez installer cet outil dans Azure Cloud Shell, puis l’utiliser pour amorçage de votre base de données.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Installez l'outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161 ; Cette commande peut prendre quelques minutes. Cette commande affichera le message d'avertissement (*L'outil 'cosmicworks' est déjà installé') si vous avez déjà installé la dernière version de cet outil dans le passé.

1. Exécutez cosmicworks pour amorcer votre compte Azure Cosmos DB avec les options de ligne de commande suivantes :

    | **Option** | **Valeur** |
    | ---: | :--- |
    | **--endpoint** | *Valeur de point de terminaison que vous avez copiée précédemment dans ce labo* |
    | **--key** | *La valeur clé que vous avez traitée plus tôt dans cet atelier* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/** et que votre clé est : **fDR2ci9QgkdkvERTQ==**, la commande serait : ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Attendez que la commande **cosmicworks** ait fini de remplir le compte avec une base de données, un conteneur et des éléments.

1. Fermez le terminal intégré.

## Paginer via de petits jeux de résultats d’une requête SQL à l’aide du Kit de développement logiciel (SDK)

Lors du traitement des résultats de requête, vous devez vous assurer que votre code progresse dans toutes les pages de résultats et de vérifications pour voir si d’autres pages sont restantes avant d’effectuer des requêtes ultérieures.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **10-paginate-results-sdk**.

1. Ouvrez le fichier de code **product.cs**.

1. Observez la classe **Produit** et ses propriétés correspondantes. Plus précisément, cet atelier utilisera les propriétés **id**, **nom** et **prix**.

1. De retour dans le volet **Explorateur** de **Visual Studio Code**, ouvrez le fichier de code **script.cs**.

1. Mettez à jour la variable existante nommée **point de terminaison** avec sa valeur définie sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/**, l'instruction C# serait : **point de terminaison de chaîne = "https&shy;://dp420.documents.azure.com:443/";**.

1. Mettez à jour la variable existante nommée **clé** avec sa valeur définie sur la **clé** du compte Azure Cosmos DB que vous avez créé précédemment.

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

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

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
