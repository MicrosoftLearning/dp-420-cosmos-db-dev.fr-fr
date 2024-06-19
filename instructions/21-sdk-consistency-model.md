---
lab:
  title: Configurer des modèles de cohérence dans le portail et le kit SDK Azure Cosmos DB for NoSQL
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Configurer des modèles de cohérence dans le portail et le kit SDK Azure Cosmos DB for NoSQL

Le niveau de cohérence par défaut pour les nouveaux comptes Azure Cosmos DB for NoSQL est la cohérence de session. Ce paramètre par défaut peut être modifié pour toutes les demandes futures. Au niveau d'une requête individuelle, vous pouvez aller plus loin et assouplir le niveau de cohérence pour cette requête spécifique.

Dans ce laboratoire, nous allons configurer le niveau de cohérence par défaut pour un compte Azure Cosmos DB for NoSQL, puis configurer un niveau de cohérence pour une opération individuelle à l'aide du SDK.

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
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Distribution globale** &vert; **Géo-redondance** | *Enable* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Répliquer des données globalement**.

1. Dans le volet **Répliquer des données globalement**, ajoutez deux régions de lecture supplémentaires au compte, puis **Enregistrer** vos modifications.

    > &#128221; Dans quelques étapes, vous serez invité à changer le niveau de cohérence en Fort, mais notez que la cohérence forte pour les comptes avec des régions couvrant plus de 5 000 miles (8 000 kilomètres) est bloquée par défaut en raison d’une latence d’écriture élevée. Veillez à choisir des régions plus proches les unes des autres.  Dans un environnement de production, pour activer cette fonctionnalité, contactez le support technique.

1. Attendez que la tâche de réplication se termine avant de poursuivre.

    > &#128221; Cette opération peut prendre environ 5 à 10 minutes et accéder au volet **Cohérence par défaut**.

1. Dans le panneau des ressources, accédez au volet **Cohérence par défaut**.

1. Dans le **Cohérence par défaut**, sélectionnez l’option **Fort**, puis **Enregistrez** vos modifications.

1. Attendez que la modification du niveau de cohérence par défaut persiste avant de poursuivre cette tâche.

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

À l’aide des informations d’identification du compte récemment créé, vous allez vous connecter aux classes du kit SDK et créer une base de données et une instance de conteneur. Ensuite, vous utiliserez l'explorateur de données pour valider l'existence des instances dans le Portail Microsoft Azure.

1. Dans le volet **Explorateur**, accédez au dossier **21-sdk-consistency-model**.

1. Ouvrez le menu contextuel du dossier **21-sdk-consistency-model**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **21-sdk-consistency-model**.

1. Générez le projet à l’aide de la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

    > &#128221; Vous pourriez voir un avertissement du compilateur indiquant que les variables **point de terminaison** et **key** sont actuellement inutilisées. Vous pouvez ignorer cet avertissement en toute sécurité, car vous utiliserez ces variables dans cette tâche.

1. Fermez le terminal intégré.

1. Ouvrez le fichier de code **product.cs**.

1. Observez l’enregistrement **Produit** et ses propriétés correspondantes. Plus précisément, cet atelier utilisera les propriétés **id**, **nom** et **categoryId**.

1. De retour dans le volet **Explorateur** de **Visual Studio Code**, ouvrez le fichier de code **script.cs**.

    > &#128221; La **Bibliothèque [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** a déjà été pré-importée depuis NuGet.

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

## Configurer le niveau de cohérence pour une opération ponctuelle

La classe **ItemRequestOptions** contient des propriétés de configuration par requête. Cette classe permet d'assouplir le niveau de cohérence, qui passe de la valeur par défaut actuelle de forte à éventuelle.

1. Créez une variable de chaîne nommée **id** avec la valeur **7d9273d9-5d91-404c-bb2d-126abb6e4833** :

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. Créez une variable de chaîne nommée **categoryId** avec la valeur **78d204a2-7d64-4f4a-ac29-9bfc437ae959** :

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. Créez une variable de type **PartitionKey** nommée **partitionKey** en passant la variable **categoryId** en tant que paramètre de constructeur :

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Appelez de façon asynchrone la méthode **ReadItemAsync\<\>** générique de la variable **conteneur**en passant les variables **id** et ** partitionkey** en tant que paramètres de méthode, en utilisant **Product** comme type générique et en stockant le résultat dans une variable nommée ** réponse**de type **ItemResponse\<Product\>**  :

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Appelez la méthode statique **Console.WriteLine** pour imprimer les frais de requête à l’aide d’une chaîne de sortie mise en forme :

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **21-sdk-consistency-model**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**  :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Le montant de la requête (en RU) doit être imprimé sur la console.

    > &#128221; Les frais de requête actuels doivent être de **2 RU**. Cela s'explique par le fait que la cohérence forte exige une lecture à partir d'au moins deux répliques pour s'assurer qu'il s'agit de la dernière écriture.

1. Fermez le terminal intégré.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.cs**.

1. Supprimez les lignes de code suivantes :

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Créez une variable nommée **options** de type [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] définissant la propriété [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] sur la valeur d’énumération [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel] :

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. Appelez de façon asynchrone la méthode **ReadItemAsync\<\>** générique de la variable **conteneur**en passant les variables **id**, ** partitionKey** et **options** en tant que paramètres de méthode, en utilisant **Product** comme type générique et en stockant le résultat dans une variable nommée **réponse** de type **ItemResponse\<Product\>**  :

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. Appelez la méthode statique **Console.WriteLine** pour imprimer les frais de requête à l’aide d’une chaîne de sortie mise en forme :

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **21-sdk-consistency-model**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**  :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Le montant de la requête (en RU) doit être imprimé sur la console.

    > &#128221; Les frais de requête actuels doivent être de **1 RU**. Cela s'explique par le fait que la cohérence éventuelle ne nécessite qu'une lecture à partir d'une seule réplique.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
