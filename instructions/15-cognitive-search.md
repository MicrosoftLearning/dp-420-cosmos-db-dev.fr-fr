---
lab:
  title: "Rechercher dans des données en utilisant Recherche Azure AI et Azure Cosmos\_DB for NoSQL"
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Rechercher dans des données en utilisant Recherche Azure AI et Azure Cosmos DB for NoSQL

Recherche Azure AI combine un moteur de recherche en tant que service avec des fonctionnalités avec intégration en profondeur de capacités d’intelligence artificielle pour enrichir les informations dans l’index de recherche.

Dans ce labo, vous allez créer un index Recherche Azure AI qui indexe automatiquement les données dans un conteneur Azure Cosmos DB for NoSQL et enrichit les données en utilisant la fonctionnalité Translator d’Azure Cognitive Services.

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

1. Dans le menu de ressource, sélectionnez **Explorateur de données**.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre contextuelle **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *``cosmicworks``* |
    | **Approvisionner le débit** | activé |
    | **Débit de la base de données** | **Manuel** |
    | **RU/s de base de données requises** | ``1000`` |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **cosmicworks** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *``products``* |
    | **Clé de partition** | *``/category/name``* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks** , puis observez le nœud du conteneur **produits** dans la hiérarchie.

## Amorcez votre compte Azure Cosmos DB for NoSQL avec des exemples de données

Vous utiliserez un utilitaire en ligne de commande qui crée une base de données **cosmicworks** et un conteneur **produits**.

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal** et sélectionnez **Nouveau terminal**.

1. Installez l’outil de ligne de commande [cosmiqueworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre ordinateur.

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

1. Fermez **Visual Studio Code**.

## Créer une ressource Recherche Azure AI

Avant de poursuivre cet exercice, vous devez d’abord créer une nouvelle instance Recherche Azure AI.

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au Portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Recherche AI*, puis créez une nouvelle ressource de compte **Recherche Azure AI** avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources existant, ou créez un groupe de ressources* |
    | **Nom** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |

    > &#128221 ; vos environnements lab peuvent avoir des restrictions vous empêchant de créer un nouveau groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource de compte **Recherche Azure AI** nouvellement créée.

## Générer un indexeur et un index pour les données Azure Cosmos DB for NoSQL

Vous allez créer un indexeur qui indexe un sous-ensemble de données dans un conteneur Azure Cosmos DB for NoSQL spécifique toutes les heures.

1. Depuis le panneau de la ressource **Recherche IA**, sélectionnez **Importer des données**.

1. À l’étape **Se connecter à vos données** de l’Assistant **Importer des données**, dans la liste **Sources de données**, sélectionnez **Azure Cosmos DB**.

1. Configurez la source de données avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Nom de la source des données** | *``products-cosmossql-source``* |
    | **Chaîne de connexion** | ***chaîne de connexion** du compte Azure Cosmos DB for NoSQL créé précédemment* |
    | **Sauvegarde de la base de données** | *cosmicworks* |
    | **Collection** | *products* |

1. Dans le champ de **requête**, entrez la requête SQL suivante pour créer une vue matérialisée d’un sous-ensemble de vos données dans le conteneur :

    ```sql
    SELECT 
        p.id, 
        p.category, 
        p.name, 
        p.price,
        p._ts
    FROM 
        products p 
    WHERE 
        p._ts > @HighWaterMark 
    ORDER BY 
        p._ts
    ```

1. Cochez la case **Résultats de la requête triés par _ts**.

    > &#128221; Cette case à cocher permet à Recherche Azure AI de savoir que la requête trie les résultats par le champ **_ts**. Ce type de tri permet un suivi de progression incrémentiel. Si l’indexeur échoue, il peut récupérer directement depuis la même valeur **_ts**, car les résultats sont classés par le timestamp.

1. Sélectionnez **Suivant : Ajouter des compétences cognitives**.

1. Sélectionnez **Passer à : Personnaliser l’index cible**.

1. À l’étape **Personnaliser l’index cible** de l’Assistant, configurez l’index avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Nom de l'index** | *``products-index``* |
    | **Clé** | *id* |

1. Dans la table de champs, configurez les options **Récupérable**, **Filtrable**, **Triable**, **À choix multiples** et **Interrogeable** pour chaque champ en utilisant le tableau suivant :

    | **Champ** | **Récupérable** | **Filtrable** | **Triable** | **À choix multiples** | **Possibilité de recherche** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | **category** | &#10004; | &#10004; | &#10004; | &#10004; | |
    | **name** | &#10004; | &#10004; | &#10004; | | &#10004; (Français - Microsoft) |
    | **p**rice | &#10004; | &#10004; | &#10004; | &#10004; | |

1. Sélectionnez **Suivant : Créer un indexeur**.

1. À l’étape **Créer un indexeur** de l’Assistant, configurez l’indexeur avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *``products-cosmosdb-indexer``* |
    | **Planification** | *Toutes les heures* |

1. Sélectionnez **Envoyer** pour créer la source de données, l’index et l’indexeur.

    > &#128221; Vous devrez peut-être ignorer une fenêtre contextuelle d’enquête après avoir créé votre premier indexeur.

1. Dans le panneau des ressources **Recherche IA**, accédez à l’onglet **Indexeurs** pour observer le résultat de votre première opération d’indexation.

1. Attendez que l’indexeur **products-cosmosdb-indexer** ait un état **Réussite** avant de poursuivre cette tâche.

    > &#128221; Vous devrez peut-être utiliser l’option **Actualiser** pour mettre à jour le panneau s’il ne le fait pas automatiquement.

1. Accédez à l’onglet **Index** puis sélectionnez l’index **products-index**.

## Valider l’index avec des exemples de requêtes de recherche

Maintenant que votre vue matérialisée des données Azure Cosmos DB for NoSQL se trouve dans l’index de recherche, vous pouvez effectuer quelques requêtes de base qui tirent parti des fonctionnalités de Recherche Azure AI.

> &#128221; Ce labo n’est pas destiné à enseigner la syntaxe Recherche Azure AI. Ces requêtes ont été organisées pour présenter certaines des fonctionnalités disponibles dans l’index de recherche et le moteur.

1. Dans l’onglet **Explorateur de recherche**, sélectionnez la liste déroulante **Vue**, puis sélectionnez la **Vue JSON**.

1. Notez, dans l’**Éditeur de requête JSON**, la syntaxe de la requête de recherche JSON par défaut qui retourne tous les résultats possibles à l’aide d’un opérateur **\*** (générique).

   ```json
    {
      "search": "*",
      "count": true
    }
   ```

1. Sélectionnez le bouton **Rechercher** pour effectuer la recherche.

1. Notez que cette requête de recherche retourne tous les résultats possibles et inclut également un champ de métadonnées qui indique le nombre total de résultats, même s’ils ne sont pas tous inclus sur la même page.

1. Dans l’**Éditeur de requête JSON**, entrez la requête suivante, puis sélectionnez **Rechercher** :

    ```json
    {
        "search": "touring 3000"
    }
    ```

1. Notez que cette requête de recherche retourne des résultats qui contiennent soit le terme **touring** soit le terme **3000**, en donnant un score plus élevé aux résultats qui contiennent les deux termes. Les résultats sont ensuite triés en ordre décroissant par le champ **@search.score**.

1. Dans l’**Éditeur de requête JSON**, entrez la requête suivante, puis sélectionnez **Rechercher** :

    ```json
    {
        "search": "blue"
        , "count": true
        , "top": 6
    }
    ```

1. Remarquez que cette requête de recherche ne retourne qu’un ensemble de six résultats à la fois, même s’il existe plus de correspondances côté serveur.

1. Dans l’**Éditeur de requête JSON**, entrez la requête suivante, puis sélectionnez **Rechercher** :

    ```json
    {
        "search": "mountain"
        , "count": true
        , "top": 25
        , "skip": 50
    }
    ```

1. Remarquez que cette requête de recherche ignore les 50 premiers résultats et retourne un ensemble de 25 résultats. S’il s’agissait d’une vue paginée dans une application côté client, vous pourriez en déduire qu’il s’agirait de la troisième « page » des résultats.

1. Dans l’**Éditeur de requête JSON**, entrez la requête suivante, puis sélectionnez **Rechercher** :

    ```json
    {
        "search": "touring"
        , "count": true
        , "filter": "price lt 500"
    }
    ```

1. Remarquez que cette requête de recherche ne retourne que les résultats où la valeur du champ de prix numérique est inférieure à 500.

1. Dans l’**Éditeur de requête JSON**, entrez la requête suivante, puis sélectionnez **Rechercher** :

    ```json
    {
        "search": "road"
        , "count": true
        , "top": 15
        , "facets": ["price,interval:500"]
    }
    ```

1. Remarquez que cette requête de recherche retourne une collection de données à facettes qui indique le nombre d’éléments appartenant à chaque catégorie, même s’ils ne sont pas tous présents dans la page des résultats actuelle. Dans cet exemple, les éléments correspondants sont répartis en catégories de prix numériques avec des intervalles de 500. Cela est généralement utilisé pour remplir des filtres et des aides à la navigation dans les applications côté client.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
