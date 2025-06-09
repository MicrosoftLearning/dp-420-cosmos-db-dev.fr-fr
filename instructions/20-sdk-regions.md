---
lab:
  title: Se connecter à différentes régions avec le kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Se connecter à différentes régions avec le kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL

Lorsque vous activez la géoredondance pour un compte Azure Cosmos DB for NoSQL, vous pouvez ensuite utiliser le kit de développement logiciel (SDK) pour lire des données depuis les régions dans tout ordre que vous configurez. Cette technique est bénéfique lorsque vous distribuez vos demandes de lecture dans toutes vos régions de lecture disponibles.

Dans ce labo, vous allez configurer la classe CosmosClient pour vous connecter à des régions de lecture dans un ordre de secours que vous configurez manuellement.

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

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Répliquer des données globalement**.

1. Dans le volet **Répliquer des données globalement**, ajoutez deux régions de lecture supplémentaires au compte, puis **Enregistrer** vos modifications.

1. Attendez que la tâche de réplication se termine avant de poursuivre.

    > &#128221; Cette opération peut prendre de 5 à 10 minutes environ.

1. Enregistrez les noms de la région **Write** (primaire) et des deux régions **Read**. Vous utiliserez ces noms de région plus loin dans cet exercice.

    > &#128221; Par exemple, si votre région primaire est **Europe Nord** et que vos deux régions secondaires de lecture sont **USA Est 2** et **Afrique du Sud Nord**, vous enregistrerez ces trois noms en l’état.

1. Dans le panneau des ressources, accédez au volet **Explorateur de données**.

1. Dans le volet **Explorateur de données** , sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Créer nouveau* &vert; *``cosmicworks``* |
    | **Partager le débit entre les conteneurs** | *Ne pas sélectionner* |
    | **ID de conteneur** | *``products``* |
    | **Clé de partition** | *``/categoryId``* |
    | **Débit de conteneur** | *Manuel* &vert; *400* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks** , puis observez le nœud de conteneur des **produits** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks** , développez le nœud de conteneur des **produits**, puis sélectionnez **Éléments**.

1. Toujours dans le volet **Explorateur de données**, sélectionnez **Nouvel élément** depuis la barre de commandes. Dans l’éditeur, remplacez l’élément JSON d’espace réservé par le contenu suivant :

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Sélectionnez **Enregistrer** depuis la barre de commandes pour ajouter l’élément JSON :

1. Dans l’onglet **Éléments** , observez le nouvel élément dans le volet **Éléments**.

1. Dans le panneau des ressources, accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur **key** plus tard dans cet exercice.

1. Revenez à **Visual Studio Code**.

## Se connecter au compte Azure Cosmos DB for NoSQL à partir du kit de développement logiciel

À l’aide des informations d’identification du compte nouvellement créé, vous vous connecterez aux classes du kit de développement logiciel (SDK) et accéderez à la base de données et à l’instance de conteneur depuis une autre région.

1. Dans le volet **Explorateur**, accédez au dossier **20-sdk-regions**.

1. Ouvrez le menu contextuel du dossier **20-sdk-regions**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **20-sdk-regions** .

1. Générez le projet à l’aide de la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

    > &#128221; Vous pourriez voir un avertissement du compilateur indiquant que les variables **point de terminaison** et **key** sont actuellement inutilisées. Vous pouvez ignorer cet avertissement en toute sécurité, car vous utiliserez ces variables dans cette tâche.

1. Fermez le terminal intégré.

1. Ouvrez le fichier de code **script.cs** dans le dossier **20-sdk-regions**.

    > &#128221; La bibliothèque **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0]** a déjà été préimportée à partir de NuGet.

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

## Configurer le kit de développement logiciel (SDK) .NET avec une liste de régions préférées

La classe **CosmosClientOptions** inclut une propriété pour configurer la liste des régions à laquelle vous souhaitez vous connecter avec le kit de développement logiciel (SDK). La liste est triée par priorité de basculement, et tentera de se connecter à chaque région dans l’ordre que vous configurez.

1. Créez une variable de type générique **List\<string\>** qui contient une liste des régions que vous avez configurées avec votre compte, en commençant par la troisième région et en terminant par la première région (primaire). Par exemple, si vous avez créé votre compte Azure Cosmos DB for NoSQL dans la région **USA Ouest**, puis ajouté **Afrique du Sud Nord** et enfin ajouté **Asie Est**, alors votre variable de liste contiendrait :

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; Vous pouvez également utiliser la classe statique [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] qui inclut des propriétés de chaîne intégrées pour différentes régions Azure.

1. Créez une nouvelle instance de la classe **CosmosClientOptions** nommée **options** avec la propriété [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] définie sur la variable **regions** :

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    ```

1. Créez une nouvelle instance de la classe **CosmosClient** nommée **client** en transmettant les variables **endpoint**, **key** et **options** en tant que paramètres de constructeur :

    ```
    using CosmosClient client = new (endpoint, key, options); 
    ```

1. Utilisez la méthode **GetContainer** de la variable **client** pour récupérer le conteneur existant en utilisant le nom de la base de données (*cosmicworks*) et le nom du conteneur (*products*) :

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Utilisez la méthode [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] de la variable **container** pour récupérer un élément spécifique depuis le serveur et stocker le résultat dans une variable nommée **response** du type [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] pouvant accepter la valeur Null :

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. Appelez la méthode statique **Console.WriteLine** pour imprimer l’identificateur d’élément actif et les données de diagnostic JSON :

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. Une fois que vous avez terminé, votre fichier de code doit maintenant inclure :
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **20-sdk-regions**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Le nom du conteneur et les données de diagnostic JSON doivent s’imprimer dans la sortie de la console.

1. Examinez les données de diagnostic JSON. Recherchez une propriété nommée **HttpResponseStats** et une propriété enfant nommée **RequestUri**. La valeur de cette propriété doit être un URI qui inclut le nom et la région que vous avez configurés précédemment dans ce labo.

    > &#128221; Par exemple, si le nom de votre compte est : **dp420** et que la première région que vous avez configurée est **Asie Est**, alors la valeur de la propriété JSON serait : **dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products**.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
