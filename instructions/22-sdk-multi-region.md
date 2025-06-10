---
lab:
  title: "Se connecter à un compte d’écriture multirégion avec le SDK Azure Cosmos\_DB for NoSQL"
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Se connecter à un compte d’écriture multirégion avec le SDK Azure Cosmos DB for NoSQL

La classe **CosmosClientBuilder** est une classe Fluent conçue pour générer le client SDK visant à vous connecter à votre conteneur et à effectuer des opérations. À l’aide du générateur, vous pouvez configurer une région d’application préférée pour les opérations d’écriture si votre compte Azure Cosmos DB for NoSQL est déjà configuré pour les écritures multirégions.

Dans ce labo, vous allez configurer un compte Azure Cosmos DB for NoSQL avec plusieurs régions et activer les écritures multirégions. Vous allez ensuite utiliser le SDK pour effectuer des opérations dans une région spécifique.

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

1. Accédez à la ressource du compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Répliquer les données globalement**.

1. Dans le volet **Répliquer les données globalement**, ajoutez au moins une région supplémentaire au compte.

1. Toujours dans le volet **Répliquer les données globalement**, activez **Écritures multirégions**, puis **Enregistrez** vos modifications.

1. Attendez que la tâche de réplication se termine avant de poursuivre.

    > &#128221; Cette opération peut prendre de 5 à 10 minutes environ.

1. Notez au moins l’une des régions supplémentaires que vous avez créées. Vous l’utiliserez plus loin dans cet exercice.

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

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **products** dans la hiérarchie.

1. Dans le panneau des ressources, accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur **key** plus tard dans cet exercice.

1. Revenez à **Visual Studio Code**.

## Se connecter au compte Azure Cosmos DB for NoSQL à partir du kit de développement logiciel

À l’aide des informations d’identification du compte récemment créé, vous allez vous connecter aux classes du kit SDK et créer une base de données et une instance de conteneur. Ensuite, utilisez l’Explorateur de données pour confirmer l’existence des instances dans le portail Azure.

1. Dans le volet **Explorateur**, accédez au dossier **22-sdk-multi-region**.

1. Ouvrez le menu contextuel du dossier **22-sdk-multi-region**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **22-sdk-multi-region**.

1. Générez le projet à l’aide de la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

    > &#128221; Vous pourriez voir un avertissement du compilateur indiquant que les variables **point de terminaison** et **key** sont actuellement inutilisées. Vous pouvez ignorer cet avertissement en toute sécurité, car vous utiliserez ces variables dans cette tâche.

1. Fermez le terminal intégré.

1. Ouvrez le fichier de code **product.cs**.

1. Observez l’enregistrement **Produit** et ses propriétés correspondantes. Plus précisément, cet atelier utilisera les propriétés **id**, **nom** et **categoryId**.

1. De retour dans le volet **Explorateur** de **Visual Studio Code**, ouvrez le fichier de code **script.cs**.

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

## Configurer la région d’écriture pour le SDK

La méthode Fluent **WithApplicationRegion** est utilisée pour configurer la région préférée pour les opérations suivantes à l’aide de la classe builder.

1. Créez une nouvelle instance de la classe [CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder] nommée **builder** en passant les variables **endpoint** et **key** en tant que paramètres de constructeur :

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. Créez une variable nommée **region** de type **chaîne** avec le nom de la région supplémentaire que vous avez créée précédemment dans le labo. Par exemple, si vous avez créé votre compte Azure Cosmos DB for NoSQL dans la région **USA Est**, puis ajouté **Brésil Sud**, votre variable de chaîne contient :

    ```
    string region = "Brazil South"; 
    ```

    > &#128161; Vous pouvez également utiliser la classe statique [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] qui inclut des propriétés de chaîne intégrées pour différentes régions Azure.

1. Appelez la méthode [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion] avec un paramètre de **région** et la méthode [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] couramment sur la variable **builder** stockant le résultat dans une variable nommée **client** de type **CosmosClient** qui est encapsulée dans une instruction using :

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. Utilisez la méthode **GetContainer** de la variable **client** pour récupérer le conteneur existant en utilisant le nom de la base de données (*cosmicworks*) et le nom du conteneur (*products*) :

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Créez deux variables **string** nommées **id** et **categoryId** en générant une nouvelle valeur **Guid**, puis en stockant le résultat sous forme de chaîne :

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. Créez une variable nommée **item** de type **Product** en passant la variable **id**, la valeur de chaîne **Polished Bike Frame** et la variable **categoryId** en tant que paramètres de constructeur :

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. Appelez de manière asynchrone la méthode **CreateItemAsync\<\>** de la variable **container** en passant la variable **item** en tant que paramètre et en stockant le résultat dans une variable nommée **response** :

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. Appelez la méthode statique **Console.WriteLine** pour afficher le code d’état HTTP de la réponse et les frais de requête (en unités de requête) :

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";    

    CosmosClientBuilder builder = new (endpoint, key);            
    
    string region = "West Europe";
    
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);
    
    var response = await container.CreateItemAsync<Product>(item);
    
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **22-sdk-multi-region**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**  :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. Le code d’état HTTP et les frais de requête (en RU) devraient être affichés dans la console.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
