---
lab:
  title: Se connecter à Azure Cosmos DB for NoSQL avec le kit SDK
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Se connecter à Azure Cosmos DB for NoSQL avec le kit SDK

Le kit de développement logiciel Azure SDK pour .NET est une suite de bibliothèques offrant une interface de développement cohérente pour interagir avec de nombreux services Azure. Le kit Azure SDK pour .NET est conforme à la spécification .NET Standard 2.0, ce qui garantit que vous pouvez l’utiliser dans les applications .NET Framework (version 4.6.1 ou ultérieure), .NET Core (version 2.1 ou ultérieure) et .NET (version 5 ou ultérieure).

Dans ce labo, vous allez vous connecter à un compte Azure Cosmos DB for NoSQL à l’aide du kit Azure SDK pour .NET.

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
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |
    | **Limiter la quantité totale de débit pouvant être approvisionnée sur ce compte** | *Décoché* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utiliserez cette valeur de **clé** plus tard dans cet exercice.

1. Gardez l’onglet du navigateur ouvert, car nous y retournerons ultérieurement.

## Afficher la bibliothèque Microsoft.Azure.Cosmos sur NuGet

Le site web NuGet contient un index pouvant faire l’objet d’une recherche des packages que vous pouvez importer dans vos applications .NET. Pour importer un package en préversion tel que **Microsoft.Azure.Cosmos**, vous pouvez utiliser le site web de NuGet pour obtenir les versions et commandes appropriées afin d’importer le package dans vos applications.

1. Sous le nouvel onglet du navigateur, accédez au site web de NuGet (``nuget.org``).

1. Passez en revue la description de NuGet, le gestionnaire de package pour .NET et ses fonctionnalités.

1. Recherchez la bibliothèque **Microsoft.Azure.Cosmos** sur NuGet.org.

1. Sélectionnez l’onglet **CLI .NET** et observez la commande requise pour importer la dernière version de cette bibliothèque dans un projet .NET.

    > &#128161; Il est inutile d’enregistrer cette commande. Vous utiliserez une version spécifique de la bibliothèque plus loin dans cet exercice.

1. Fermez la fenêtre ou l’onglet de votre navigateur web.

## Importer la bibliothèque Microsoft.Azure.Cosmos dans un projet .NET

L’interface CLI .NET comprend une commande [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] pour importer des packages à partir d’un flux de package préconfiguré. Une installation .NET utilise NuGet comme flux de package par défaut.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **04-sdk-connect**.

1. Ouvrez le menu contextuel du dossier **04-sdk-connect**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **04-sdk-connect**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.*
    ```

1. Fermez le terminal intégré.

## Utiliser la bibliothèque Microsoft.Azure.Cosmos

Une fois la bibliothèque Azure Cosmos DB du kit Azure SDK pour .NET importée, vous pouvez immédiatement utiliser ses classes dans l’espace de noms [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] pour vous connecter à un compte Azure Cosmos DB for NoSQL. La classe [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] est la classe principale utilisée pour établir la connexion initiale à un compte Azure Cosmos DB for NoSQL.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **04-sdk-connect**.

1. Ouvrez le fichier de code **script.cs** vide.

1. Ajoutez des blocs using pour les espaces de noms intégrés **System** et **System.Linq** :

    ```
    using System;
    using System.Linq;
    ```

1. Ajoutez un bloc using pour l’espace de noms [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] :

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. Ajoutez une variable **string** nommée **endpoint** avec pour valeur le **point de terminaison** du compte Azure Cosmos DB créé précédemment.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est **https&shy;://dp420.documents.azure.com:443/**, l’instruction C# est **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Ajoutez une variable **string** nommée **key** avec pour valeur la **clé** du compte Azure Cosmos DB créé précédemment.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est **fDR2ci9QgkdkvERTQ==**, l’instruction C# est **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Ajoutez une nouvelle variable nommée **client** de type [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] à l’aide des variables **endpoint** et **key** dans le constructeur :
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. Ajoutez une nouvelle variable nommée **account** de type [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] en utilisant le résultat asynchrone de l’appel de la méthode [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] de la variable **client** :

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. Utilisez la méthode statique intégrée **Console.WriteLine** pour afficher la propriété [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] de la classe AccountProperties avec un en-tête intitulé **Account Name** :

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. Utilisez la méthode statique intégrée **Console.WriteLine** pour interroger la propriété [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] de la classe AccountProperties, puis afficher la propriété [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] du premier résultat avec un en-tête intitulé **Primary Region** :

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **Enregistrez** le fichier de code **script.cs**.

## Tester le script

Le code .NET pour vous connecter au compte Azure Cosmos DB for NoSQL étant désormais terminé, vous pouvez tester le script. Ce script affiche le nom du compte et le nom de la première région accessible en écriture. La valeur d’emplacement que vous avez spécifiée lorsque vous avez créé le compte doit s’afficher à la suite de l’exécution de ce script.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **04-sdk-connect**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Le script génère désormais le nom du compte et la première région accessible en écriture. Par exemple, si vous avez nommé le compte **dp420** et que la première région accessible en écriture était **USA Ouest 2**, le script génère :

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
