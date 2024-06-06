---
lab:
  title: Traiter des données Azure Cosmos DB for NoSQL en utilisant Azure Functions
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Traiter des données Azure Cosmos DB for NoSQL en utilisant Azure Functions

Le déclencheur Azure Cosmos DB pour Azure Functions est implémenté à l’aide d’un processeur de flux de modification. Grâce à ces connaissances, vous pouvez créer des fonctions qui répondent aux opérations de création et de mise à jour dans votre conteneur Azure Cosmos DB for NoSQL. Si vous avez implémenté manuellement un processeur de flux de modification, la configuration d’Azure Functions est similaire.

Dans ce labo, vous allez effectuer les tâches suivantes :

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud, qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple l’**API Mongo** ou l’**API NoSQL**). Une fois l’approvisionnement du compte Azure Cosmos DB for NoSQL effectué, vous pouvez récupérer le point de terminaison et la clé, et les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant le kit Azure SDK pour .NET ou tout autre kit SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources existant, ou créez un groupe de ressources* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Sans serveur* |

    > &#128221; Vos environnements lab peuvent présenter des restrictions qui vous empêchent de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur de **point de terminaison** plus tard dans cet exercice.

    1. Notez le champ **PRIMARY KEY**. Vous utiliserez cette valeur de **clé** plus tard dans cet exercice.

1. Dans le menu de ressource, sélectionnez **Explorateur de données**.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre indépendante **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *``cosmicworks``* |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **cosmicworks** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre indépendante **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *``products``* |
    | **Clé de partition** | *``/categoryId``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **products** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez une nouvelle fois **Nouveau conteneur**.

1. Dans la fenêtre indépendante **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *``productslease``* |
    | **Clé de partition** | *``/id``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **productslease** dans la hiérarchie.

1. Retournez à l’**Accueil** du portail Azure.

## Créer une ressource Application Insights

Pour pouvoir créer l’*application Azure Functions*, vous devez activer au préalable une ressource *Azure Application Insights* afin d’effectuer le monitoring des fonctions de l’application. Application Insights a d’abord besoin d’un *espace de travail Log Analytics*.

1. Dans la zone de recherche, recherchez **Espaces de travail Log Analytics**.

1. Sélectionnez **+Créer** pour créer un espace de travail *Log Analytics*.

1. Dans la boîte de dialogue **Espace de travail Log Analytics**, entrez les valeurs suivantes pour chaque paramètre, sélectionnez **Vérifier + créer**, puis **Créer** :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources existant, ou créez un groupe de ressources* |
    | **Nom** | *``lab14laworkspace``* |
    | **Lieu** | *Choisissez une région disponible* |

1. Une fois votre *espace de travail Log Analytics* créé, dans la zone de recherche, recherchez **Application Insights**.

1. Sélectionnez **+Créer** pour créer une ressource *Application Insights*.

1. Dans la boîte de dialogue **Application Insights**, entrez les valeurs suivantes pour chaque paramètre, sélectionnez **Vérifier + créer**, puis **Créer** :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement (les deux entrées)** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources existant, ou créez un groupe de ressources* |
    | **Nom** | *``lab14appinsight``* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Espace de travail Log Analytics** | *lab14laworkspace* |

Vous devez désormais pouvoir effectuer un monitoring de vos fonctions d’application.

## Créer une application de fonction Azure et une fonction déclenchée par Azure Cosmos DB

Pour pouvoir commencer à écrire du code, vous devez créer au préalable la ressource Azure Functions et ses ressources dépendantes (Application Insights, Stockage) à l’aide de l’Assistant Création.

1. Sélectionnez **+ Créer une ressource**, recherchez *Functions*, puis créez une ressource de compte **Application de fonction** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources existant, ou créez un groupe de ressources* |
    | **Nom** | *Entrez un nom globalement unique* |
    | **Publier** | *Code* |
    | **Pile d’exécution** | *.NET* |
    | **Version** | *6 (LTS) In-process* |
    | **Région** | *Choisissez une région disponible* |
    | **Compte de stockage** | *Créer un compte de stockage* |

    > &#128221; Vos environnements lab peuvent présenter des restrictions qui vous empêchent de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Functions** qui vient d’être créée, puis accédez au volet **Fonctions**.

1. Dans le volet **Fonctions**, sélectionnez **+ Créer**.

1. Dans la fenêtre indépendante **Créer une fonction**, créez une fonction avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Environnement de développement** | *Développer dans le portail* |
    | **Sélectionner un modèle** | *Déclencheur Azure Cosmos DB* |
    | **Nouvelle fonction** | *``ItemsListener``* |
    | **Connexion au compte Cosmos DB** | *Sélectionnez Nouveau* &vert; *Sélectionnez Compte Azure Cosmos DB* &vert; *Sélectionnez le compte Azure Cosmos DB que vous avez créé* |
    | **Nom de la base de données** | *``cosmicworks``* |
    | **Nom de la collection** | *``products``* |
    | **Nom de collection pour les baux** | *``productslease``* |
    | **Créer une collection de baux si elle n’existe pas** | *Aucun* |

## Implémenter du code de fonction dans .NET

La fonction que vous avez créée est un script C# qui est modifié dans le portail. Vous allez à présent utiliser le portail pour écrire une fonction courte permettant de générer l’identificateur unique d’un élément inséré ou mis à jour dans le conteneur.

1. Dans le volet **ItemsListener** &vert; **Fonction**, accédez au volet **Code + test**.

1. Dans l’éditeur du script **run.csx**, supprimez le contenu de la zone de l’éditeur.

1. Dans la zone de l’éditeur, référencez la bibliothèque **Microsoft.Azure.DocumentDB.Core** :

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. Ajoutez des blocs using pour les espaces de noms **System**, **System.Collections.Generic** et [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] :

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. Créez une méthode statique nommée **Run** qui comporte deux paramètres :

    1. Paramètre nommé **input** de type **IReadOnlyList\<\>** avec le type générique [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document].

    1. Paramètre nommé **log** de type **ILogger**.

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. Dans la méthode **Run**, appelez la méthode **LogInformation** de la variable **log** en passant une chaîne qui calcule le nombre d’éléments dans le lot actuel :

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. Toujours dans la méthode **Run**, créez une boucle foreach qui itère sur la variable **input** à l’aide de la variable **item** pour représenter une instance de type **Document** :

    ```
    foreach(Document item in input)
    {
    }
    ```

1. Dans la boucle foreach de la méthode **Run**, appelez la méthode **LogInformation** de la variable **log** en passant une chaîne qui affiche la propriété [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] de la variable **item** :

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. Une fois que vous avez fini, votre fichier de code doit inclure :
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. Développez la section **Journaux** pour vous connecter aux journaux en streaming de la fonction actuelle.

    > &#128161; La connexion au service de journaux en streaming peut prendre quelques secondes. Un message s’affiche dans la sortie des journaux une fois que vous êtes connecté.

1. **Enregistrez** le code de fonction actuel.

1. Observez le résultat de la compilation du code C#. En principe, vous devez voir le message **Compilation réussie** à la fin de la sortie des journaux.

    > &#128221; Vous verrez peut-être des messages d’avertissement dans la sortie des journaux. Ces avertissements n’ont pas d’impact sur ce labo.

1. **Agrandissez** la section des journaux pour développer la fenêtre Sortie, et remplir l’espace maximal disponible.

    > &#128221; Vous allez utiliser un autre outil pour générer des éléments dans votre conteneur Azure Cosmos DB for NoSQL. Une fois les éléments générés, retournez à cette fenêtre de navigateur pour observer la sortie. Ne fermez pas la fenêtre de navigateur trop tôt.

## Remplir initialement votre compte Azure Cosmos DB for NoSQL avec des exemples de données

Vous allez vous servir d’un utilitaire en ligne de commande, qui crée une base de données **cosmicworks** et un conteneur **products**. L’outil crée ensuite un ensemble d’éléments que vous allez observer à l’aide du processeur de flux de modification qui s’exécute dans votre fenêtre de terminal.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [Guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Installez l’outil en ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

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

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/** et si votre clé est : **fDR2ci9QgkdkvERTQ==**, la commande est : ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Attendez que la commande **cosmicworks** ait fini de remplir le compte avec une base de données, un conteneur et des éléments.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

1. Retournez à la fenêtre ou à l’onglet de navigateur ouvert avec la section de journaux Azure Functions développée.

1. Observez la sortie des journaux de votre fonction. Le terminal génère un message **Opération détectée** pour chaque changement qui lui a été envoyé à l’aide du flux de modification. Les opérations sont regroupées par lots d’environ 100 opérations.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
