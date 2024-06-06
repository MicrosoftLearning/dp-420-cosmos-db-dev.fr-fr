---
lab:
  title: Traiter les événements de flux de modification à l’aide du SDK Azure Cosmos DB for NoSQL
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Traiter les événements de flux de modification à l’aide du SDK Azure Cosmos DB for NoSQL

Le flux de modification Azure Cosmos DB for NoSQL est la clé pour créer des applications supplémentaires pilotées par les événements de la plateforme. Le SDK .NET pour Azure Cosmos DB for NoSQL est fourni avec une suite de classes pour créer vos applications qui s’intègrent au flux de modification et écoutent les notifications sur les opérations au sein de vos conteneurs.

Dans ce labo, vous allez utiliser la fonctionnalité de processeur de flux de modification dans le SDK .NET pour créer une application qui est avertie de l’exécution d’une opération de création ou de mise à jour sur un élément du conteneur spécifié.

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
    | **Mode de capacité** | *Sans serveur* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions qui vous empêchent de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur de **point de terminaison** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur de **clé** plus tard dans cet exercice.

1. Dans le menu de ressource, sélectionnez **Explorateur de données**.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre contextuelle **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *``cosmicworks``* |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **cosmicworks** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *``products``* |
    | **Clé de partition** | *``/categoryId``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **products** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez une nouvelle fois **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *``productslease``* |
    | **Clé de partition** | *``/partitionKey``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **productslease** dans la hiérarchie.

1. Revenez à **Visual Studio Code**.

## Implémenter le processeur de flux de modification dans le SDK .NET

La classe **Microsoft.Azure.Cosmos.Container** est fournie avec une série de méthodes pour créer facilement le processeur de flux de modification. Pour commencer, vous avez besoin d’une référence à votre conteneur surveillé, votre conteneur à bail et un délégué en C\# (pour gérer chaque lot de modifications).

1. Dans le volet **Explorateur**, accédez au dossier **13-change-feed**.

1. Ouvrez le fichier de code **product.cs**.

1. Observez la classe **Produit** et ses propriétés correspondantes. Plus précisément, ce labo utilisera les propriétés **id** et **name**.

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

1. Utilisez la méthode **GetContainer** de la variable **client** pour récupérer le conteneur existant en utilisant le nom de la base de données (*cosmicworks*) et le nom du conteneur (*products*). Stockez ensuite le résultat dans une variable nommée **sourceContainer** de type **Container** :

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. Utilisez la méthode **GetContainer** de la variable **client** pour récupérer le conteneur existant en utilisant le nom de la base de données (*cosmicworks*) et le nom du conteneur (*productslease*). Stockez ensuite le résultat dans une variable nommée **leaseContainer** de type **Container** :

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. Créez une variable déléguée nommée **handleChanges** de type [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] à l’aide d’une fonction anonyme asynchrone vide qui a deux paramètres d’entrée :

    1. Un paramètre nommé **changes** de type **IReadOnlyCollection\<Product\>**.
    
    1. Un paramètre nommé **cancellationToken** de type **CancellationToken**.

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. Dans la fonction anonyme, utilisez la méthode statique intégrée **console.WriteLine** pour imprimer la chaîne brute **START\tTraitement du lot de modifications...** :

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. Toujours dans la fonction anonyme, créez une boucle foreach qui itère sur la variable **changes** à l’aide de la variable **product** pour représenter une instance de type **Product** :

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. Dans la boucle foreach de la fonction anonyme, utilisez la méthode statique asynchrone intégrée **Console.WriteLineAsync** pour imprimer les propriétés **id** et **name** de la variable **product** :

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. En dehors de la boucle foreach et de la fonction anonyme, créez une variable nommée **builder** qui stocke le résultat de l’appel de [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] sur la variable **sourceContainer** à l’aide des paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. Appelez la méthode [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] avec le paramètre **consoleApp**, la méthode [WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] avec le paramètre **leaseContainer** et la méthode [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] sur la variable **Builder** stockant le résultat dans une variable nommée **processor** de type [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor] :

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. Appelez de façon asynchrone la méthode [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync] de la variable **processor** :

    ```
    await processor.StartAsync();
    ```

1. Utilisez les méthodes statiques intégrées **Console.WriteLine** et **Console.ReadKey** pour imprimer la sortie dans la console et que l’application attende l’activation d’une touche :

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. Appelez de façon asynchrone la méthode [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] de la variable **processor** :

    ```
    await processor.StopAsync();
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **Enregistrez** le fichier **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **13-change-feed**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Laissez **Visual Studio Code** et le terminal ouverts.

    > &#128221; Vous allez utiliser un autre outil pour générer des éléments dans votre conteneur Azure Cosmos DB for NoSQL. Une fois les éléments générés, retournez à ce terminal pour observer la sortie. Ne fermez pas prématurément le terminal.

## Remplir initialement votre compte Azure Cosmos DB for NoSQL avec des exemples de données

Vous allez vous servir d’un utilitaire en ligne de commande qui crée une base de données **cosmicworks** et un conteneur **products**. L’outil crée ensuite un ensemble d’éléments que vous allez observer à l’aide du processeur de flux de modification qui s’exécute dans votre fenêtre de terminal.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Terminal divisé** pour ouvrir un nouveau terminal placé côte à côte de votre instance existante.

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre ordinateur.

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

1. Observez la sortie du terminal de votre application .NET. Le terminal génère un message **Opération détectée** pour chaque modification qui lui a été envoyée à l’aide du flux de modification.

1. Fermez les deux terminaux intégrés.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
