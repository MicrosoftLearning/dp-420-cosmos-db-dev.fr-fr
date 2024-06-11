---
lab:
  title: Optimiser une stratégie d’index d’un conteneur Azure Cosmos DB for NoSQL pour les opérations courantes
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimiser une stratégie d’index d’un conteneur Azure Cosmos DB for NoSQL pour les opérations courantes

Pour les charges de travail nécessitant beaucoup d’écritures ou les charges de travail avec des objets JSON volumineux, il peut être avantageux d’optimiser la stratégie d’indexation pour n’indexer que les propriétés que vous utiliserez dans vos requêtes.

Dans ce labo, nous allons utiliser une application .NET de test pour insérer un élément JSON volumineux dans un conteneur Azure Cosmos DB pour NoSQL à l’aide de la stratégie d’indexation par défaut, puis en utilisant une stratégie d’indexation qui a été légèrement ajustée.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code labo pour **DP-420** à l’environnement dans lequel vous travaillez sur ce labo, procédez comme suit. Sinon, ouvrez le dossier cloné précédemment dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas déjà familiarisé avec l’interface Visual Studio Code, consultez la [Documentation de prise en main][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git: Clone** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl+Maj+P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge différentes API. Lors du provisionnement d’un compte Azure Cosmos DB pour la première fois, vous sélectionnerez les API que vous souhaitez que le compte prenne en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Une fois que le compte Azure Cosmos DB for NoSQL a terminé l’approvisionnement, vous pouvez récupérer le point de terminaison et la clé et les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant le kit de développement logiciel (SDK) Azure pour .NET, ou tout autre Kit de développement logiciel (SDK) de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au Portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une nouvelle ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un nouveau* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Sans serveur* |

    > &#128221 ; vos environnements lab peuvent avoir des restrictions vous empêchant de créer un nouveau groupe de ressources. Si tel est le cas, utilisez le groupe de ressources pré-créé existant.

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Explorateur de données**.

1. Dans le volet **Explorateur de données** , sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Créer nouveau* &vert; *``cosmicworks``* |
    | **ID de conteneur** | *``products``* |
    | **Clé de partition** | *``/categoryId``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks** , puis observez le nœud de conteneur des **produits** dans la hiérarchie.

1. Dans le panneau des ressources, accédez au volet **Clés**.

1. Ce volet contient les informations de connexion et les informations d’identification nécessaires pour se connecter au compte depuis le kit de développement logiciel (SDK). Plus précisément :

    1. Notez le champ **URI**. Vous utiliserez cette valeur **endpoint** plus loin dans cet exercice.

    1. Remarquez le champ **PRIMARY KEY**. Vous utiliserez cette valeur **key** plus loin dans cet exercice.

1. Revenez à **Visual Studio Code**.

## Exécuter l’application .NET de test à l’aide de la stratégie d’indexation par défaut

Ce labo dispose d’une application .NET de test prédéfinie qui prend un objet JSON volumineux et crée un élément dans le conteneur Azure Cosmos DB pour NoSQL. Une fois l’opération d’écriture unique terminée, l’application génère l’identificateur unique de l’élément et les frais en RU dans la fenêtre de console.

1. Dans le volet **Explorateur**, accédez au dossier **23-index-optimization**.

1. Ouvrez le menu contextuel du dossier **23-index-optimization**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **23-index-optimization**.

1. Générez le projet à l’aide de la commande [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] :

    ```
    dotnet build
    ```

    > &#128221; Vous pourriez voir un avertissement du compilateur indiquant que les variables **endpoint** et **key** sont actuellement inutilisées. Vous pouvez ignorer cet avertissement en toute sécurité, car vous utiliserez ces variables dans cette tâche.

1. Fermez le terminal intégré.

1. Ouvrez le fichier de code **script.cs**.

1. Localisez la variable **string** nommée **endpoint**. Définissez sa valeur sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/**, alors l'instruction C# serait : **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Recherchez la variable **string** nommée **key**. Définissez sa valeur sur la **clé** du compte Azure Cosmos DB que vous avez créé précédemment.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est : **fDR2ci9QgkdkvERTQ==**, alors l'instruction C# serait : **string key = "fDR2ci9QgkdkvERTQ==";**.

1. **Enregistrez** le fichier de code **script.cs**.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **23-index-optimization**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Générez et exécutez le projet en utilisant la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**  :

    ```
    dotnet run
    ```

1. Observez la sortie du terminal. L’identificateur unique de l’élément et les frais de requête de l’opération (en unités de requête) doivent être imprimés dans la console.

1. Générez et exécutez le projet au moins deux fois à l’aide de la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observez les frais en RU dans la sortie de la console :

    ```
    dotnet run
    ```

1. Laissez le terminal intégré ouvert.

    > &#128221 ; Vous allez réutiliser ce terminal plus loin dans cet exercice. Il est important de laisser le terminal ouvert afin de pouvoir comparer les frais en RU d’origine et mis à jour.

## Mettre à jour la stratégie d’indexation et réexécuter l’application .NET

Ce scénario de laboratoire suppose que nos futures requêtes se concentrent principalement sur les propriétés name et categoryName. Pour optimiser notre élément JSON volumineux, vous excluez tous les autres champs de l’index en créant une stratégie d’indexation qui commence par exclure tous les chemins d’accès. Ensuite, la stratégie inclut de manière sélective des chemins d’accès spécifiques.

1. Revenez à votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks** , développez le nœud de conteneur des **produits**, puis sélectionnez **Paramètres**.

1. Sous l’onglet **Paramètres**, accédez à la section **Stratégie d’indexation**.

1. Observez la stratégie d’indexation par défaut :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. Remplacez la stratégie d’indexation par cet objet JSON modifié, puis **enregistrez** les modifications :

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

1. Revenez à **Visual Studio Code**. Revenez au terminal ouvert.

1. Générez et exécutez le projet au moins deux fois à l’aide de la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observez les nouveaux frais en RU dans la sortie de la console, qui doivent être nettement inférieurs aux frais d’origine. Étant donné que vous n’indexez pas toutes les propriétés d’élément, le coût de vos écritures est considérablement inférieur lors de la mise à jour de l’index. Cela peut toutefois vous coûter considérablement si vos lectures doivent interroger sur les propriétés qui ne sont pas indexées.  

    ```
    dotnet run
    ```

    > &#128221; Si vous ne voyez pas de frais en RU mis à jour, vous devrez peut-être attendre quelques minutes.

1. Revenez à votre navigateur web.

    > &#128221 ; Si la page **Stratégie d’indexation** n’est pas ouverte, accédez à l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, développez le nœud conteneur des **produits**, sélectionnez **Paramètres** et accédez à la section **Stratégie d’indexation**.

1. Remplacez la stratégie d’indexation par cet objet JSON modifié, puis **enregistrez** les modifications :

    ```
    {
      "indexingMode": "none"
    }
    ```

1. Fermez votre fenêtre ou votre onglet de navigateur web.

1. Revenez à **Visual Studio Code**. Revenez au terminal ouvert.

1. Générez et exécutez le projet au moins deux fois à l’aide de la commande **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**. Observez les nouveaux frais en RU dans la sortie de la console, qui doivent être beaucoup moins élevés que les frais d’origine.  Comment cela est-il possible ? Étant donné que ce script mesure les RU lorsque vous écrivez l’élément, en choisissant d’avoir aucun index, il n’existe aucune surcharge de gestion de cet index. Le revers de la médaille est que si vos écritures génèrent moins de RU, vos lectures sont très coûteuses.

    ```
    dotnet run
    ```

    > &#128221; Si vous ne voyez pas de frais en RU mis à jour, vous devrez peut-être attendre quelques minutes.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
