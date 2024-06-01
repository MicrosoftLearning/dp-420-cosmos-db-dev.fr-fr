---
lab:
  title: Implémenter et utiliser des fonctions définies par l’utilisateur avec le SDK
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Implémenter et utiliser des fonctions définies par l’utilisateur avec le SDK

Le SDK .NET pour Azure Cosmos DB for NoSQL peut être utilisé pour gérer et appeler des constructions de programmation côté serveur directement à partir d’un conteneur. Quand vous préparez un nouveau conteneur, pensez à utiliser le SDK .NET pour publier des fonctions UDF directement sur un conteneur au lieu d’effectuer les tâches manuellement dans l’Explorateur de données.

Dans ce labo, vous créez une fonction UDF avec le SDK .NET, puis utilisez l’Explorateur de données pour vérifier que la fonction fonctionne correctement.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le dépôt de code du labo pour le cours **DP-420** dans l'environnement où vous travaillez, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n'êtes pas familiarisé avec l’interface de Visual Studio Code, consultez le [Guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git: Clone** pour cloner le dépôt GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans le dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl+Maj+P** pour ouvrir la palette de commandes.

1. Une fois le dépôt cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous provisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Dès que le provisionnement du compte Azure Cosmos DB for NoSQL est terminé, vous pouvez récupérer le point de terminaison et la clé, et les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant Azure SDK pour .NET ou un autre SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en laissant tous les autres paramètres avec leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent avoir des restrictions qui vous empêchent de créer un groupe de ressources. Si c’est le cas, utilisez le groupe de ressources pré-créé existant.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utilisez cette valeur de **point de terminaison** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utilisez cette valeur de **clé** plus tard dans cet exercice.

1. Sans fermer la fenêtre du navigateur, ouvrez **Visual Studio Code**.

## Alimenter le compte Azure Cosmos DB for NoSQL avec des données

L'outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] déploie des exemples de données sur un compte Azure Cosmos DB for NoSQL. L’outil est open source et disponible avec NuGet. Vous installez cet outil dans Azure Cloud Shell et l’utilisez pour l’amorçage de votre base de données.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Cette commande peut prendre quelques minutes. Cette commande affiche le message d’avertissement (*L’outil 'cosmicworks' est déjà installé) si vous avez déjà installé la dernière version de l’outil auparavant.

1. Exécutez cosmicworks pour amorcer votre compte Azure Cosmos DB avec les options de ligne de commande suivantes :

    | **Option** | **Valeur** |
    | ---: | :--- |
    | **--endpoint** | *Valeur de point de terminaison que vous avez copiée précédemment dans ce labo* |
    | **--key** | *Valeur de clé que vous avez copiée plus tôt dans ce labo* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/** et que votre clé est : **fDR2ci9QgkdkvERTQ==**, la commande est : ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Attendez que la commande **cosmicworks** ait fini de remplir le compte avec une base de données, un conteneur et des éléments.

1. Fermez le terminal intégré.

## Créer une fonction définie par l’utilisateur (UDF) en utilisant le SDK .NET

La classe [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] dans le SDK .NET comprend une propriété [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] utilisée pour effectuer des opérations CRUD sur des procédures stockées, des fonctions UDF et des déclencheurs directement à partir du SDK. Vous utilisez cette propriété pour créer une fonction UDF, puis poussez cette fonction UDF vers un conteneur Azure Cosmos DB for NoSQL. La fonction UDF que nous créons avec le SDK calcule le prix du produit avec les taxes, ce qui nous permet d’exécuter des requêtes SQL sur les produits en utilisant leur prix TTC.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **33-create-use-udf-sdk**.

1. Ouvrez le fichier de code **script.cs**.

1. Ajoutez un bloc using pour l’espace de noms [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts] :

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

1. Mettez à jour la variable existante nommée **endpoint** avec sa valeur définie sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/**, l’instruction C# est : **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Mettez à jour la variable existante nommée **key** avec sa valeur définie sur la **clé** du compte Azure Cosmos DB que vous avez créé précédemment.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est : **fDR2ci9QgkdkvERTQ==**, alors l’instruction C# est : **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Créez une variable de type [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] nommée props en utilisant le constructeur vide par défaut :

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. Définissez la propriété [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] de la variable **props** sur la valeur **tax** :

    ```
    props.Id = "tax";
    ```

1. Définissez la propriété [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] de la variable **props** sur la valeur **props.Body = "function tax(i) { return i * 1.25; }";** :

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. Passez un appel asynchrone à la méthode [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] de la variable **container** en passant la variable **props** comme paramètre et en enregistrant le résultat dans une variable nommée **udf** de type [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse] :

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. Utilisez la méthode statique intégrée **Console.WriteLine** pour imprimer la propriété [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] de la classe UserDefinedFunctionResponse avec un en-tête intitulé **Created UDF** :

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. Une fois que vous avez terminé, votre fichier de code doit maintenant inclure :
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **Enregistrez** le fichier **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **33-create-use-udf-sdk**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez et exécutez le projet avec la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```

1. Le script génère désormais le nom de la fonction UDF nouvellement créée :

    ```
    Created UDF [tax]
    ```

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

## Tester la fonction UDF avec l’Explorateur de données

Maintenant qu’une fonction UDF a été créée dans le conteneur Azure Cosmos DB, vous utilisez l’Explorateur de données pour vérifier qu’elle fonctionne comme prévu.

1. Revenez à votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **products** dans l’arborescence de l’**API NOSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Nouvelle requête SQL**.

1. Sous l’onglet de la requête, sélectionnez **Exécuter la requête** pour voir une requête standard qui sélectionne tous les éléments sans aucun filtre.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui renvoie tous les documents avec deux valeurs de prix projetées. La première valeur est la valeur de prix brute du conteneur, et la deuxième est la valeur de prix calculée par la fonction UDF :

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les documents et comparez leurs champs **price** et **priceWithTax**.

    > &#128221; Le champ **priceWithTax** doit avoir une valeur 25 % supérieure à celle du champ **price**.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
