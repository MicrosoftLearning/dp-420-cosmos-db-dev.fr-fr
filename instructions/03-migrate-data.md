---
lab:
  title: Migrer les données existantes à l’aide d’Azure Data Factory
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Migrer les données existantes à l’aide d’Azure Data Factory

Dans Azure Data Factory, Azure Cosmos DB est pris en charge comme source d’ingestion de données et en tant que cible (récepteur) de sortie de données.

Dans ce labo, nous allons remplir Azure Cosmos DB à l’aide d’un utilitaire de ligne de commande utile, puis utiliser Azure Data Factory pour déplacer un sous-ensemble de données d’un conteneur vers un autre.

## Créer et amorcer votre compte Azure Cosmos DB pour NoSQL

Vous utilisez un utilitaire de ligne de commande qui crée une base de données **cosmicworks** et un conteneur **products** à **4 000** unités de requête par seconde (RU/s). Une fois créé, vous ajustez le débit à 400 RU/s.

Pour accompagner le conteneur products, vous créez manuellement un conteneur **flatproducts** qui sera la cible de la transformation ETL et de l’opération de chargement à la fin de ce labo.

1. Dans une nouvelle fenêtre ou un nouvel onglet de navigateur web, accédez au portail Microsoft Azure (``portal.azure.com``).

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
    | **Limiter la quantité totale de débit pouvant être approvisionnée sur ce compte** | *Décoché* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Remarquez le champ **PRIMARY CONNECTION STRING**. Vous utiliserez cette valeur de **chaîne de connexion** plus loin dans cet exercice.

1. Gardez l’onglet du navigateur ouvert, car nous y retournerons ultérieurement.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [Guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Dans **Visual Studio Code**, ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Installez l’outil de ligne de commande [cosmicworks][nuget.org/packages/cosmicworks] pour une utilisation globale sur votre machine.

    ```powershell
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

1. Revenez au navigateur web, ouvrez un nouvel onglet et accédez au portail Microsoft Azure (``portal.azure.com``).

1. Sélectionnez **Groupes de ressources**, puis sélectionnez le groupe de ressources que vous avez créé ou consulté précédemment dans ce labo, puis sélectionnez la ressource **Compte Azure Cosmos DB** que vous avez créée dans ce labo.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans **Explorateur de données**, développez le nœud de base de données **cosmicworks**, développez le nœud de conteneur **produits**, puis sélectionnez **Éléments**.

1. Observez et sélectionnez les différents éléments JSON dans le conteneur **produits**. Il s’agit des éléments créés par l’outil de ligne de commande utilisé dans les étapes précédentes.

1. Sélectionnez le nœud **Mise à l’échelle**. Sous l’onglet **Mise à l’échelle**, sélectionnez **Manuel**, mettez à jour le paramètre **débit requis** de **4 000 RU/s** à **400 RU/s**, puis **enregistrez** vos modifications**.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *`flatproducts`* |
    | **Clé de partition** | *`/category`* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur **flatproducts** dans la hiérarchie.

1. Revenez à la page **Accueil** du portail Microsoft Azure.

## Créer une ressource Azure Data Factory

Maintenant que les ressources Azure Cosmos DB pour NoSQL sont en place, vous créez une ressource Azure Data Factory et configurez tous les composants et connexions nécessaires pour effectuer un déplacement unique des données d’un conteneur d’API NoSQL vers un autre pour extraire des données, les transformer et les charger dans un autre conteneur d’API NoSQL.

1. Sélectionnez **+ Créer une ressource**, recherchez *Data Factory*, puis créez une ressource **Data Factory** avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources existant, ou créez un groupe de ressources* |
    | **Nom** | *Entrez un nom globalement unique* |
    | **Région** | *Choisissez une région disponible* |
    | **Version** | *V2* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions qui vous empêchent de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource **Data Factory** nouvellement créée, puis sélectionnez **Lancer Studio**.

    > &#128161; Vous pouvez également accéder à (``adf.azure.com/home``), sélectionner votre ressource Data Factory nouvellement créée, puis sélectionner l’icône d’accueil.

1. À partir de l’écran d’accueil. Sélectionnez l’option **Ingérer** pour lancer l’assistant rapide pour effectuer une opération de copie des données à grande échelle unique et passer à l’étape **Propriétés** de l’assistant.

1. À compter de l’étape **Propriétés** de l’assistant, dans la section **Type de tâche**, sélectionnez **Tâche de copie intégrée**.

1. Dans la section **Cadence des tâches ou planification des tâches**, sélectionnez **Exécuter une fois**, puis sélectionnez **Suivant** pour passer à l’étape **Source** de l’assistant.

1. Dans l’étape **Source** de l’assistant, dans la liste **Type de source**, sélectionnez **Azure Cosmos DB pour NoSQL**.

1. Dans la section **Connexion**, sélectionnez **+ Nouvelle connexion**.

1. Dans la fenêtre contextuelle **Nouvelle connexion (Azure Cosmos DB pour NoSQL)**, configurez la nouvelle connexion avec les valeurs suivantes, puis sélectionnez **Créer** :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *`CosmosSqlConn`* |
    | **Se connecter via le runtime d’intégration** | *AutoResolveIntegrationRuntime* |
    | **Méthode d’authentification** | *Clé de compte* &vert; *Chaîne de connexion* |
    | **Méthode de sélection du compte** | *À partir d’un abonnement Azure* |
    | **Abonnement Azure** | *Votre abonnement Azure existant* |
    | **Nom de compte Azure Cosmos DB** | *Votre nom de compte Azure Cosmos DB existant que vous avez choisi précédemment dans ce labo* |
    | **Nom de la base de données** | *cosmicworks* |

1. De retour dans la section **Magasin de données source**, dans la section **Tables sources**, sélectionnez **Utiliser la requête**.

1. Dans la liste **Nom de table**, sélectionnez **products**.

1. Dans l’**Éditeur de requête**, supprimez le contenu existant et entrez la requête suivante :

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. Sélectionnez **Aperçu des données** pour tester la validité de la requête. Sélectionnez **Suivant** pour passer à l’étape **Destination** de l’assistant.

1. Dans l’étape **Destination** de l’assistant, dans la liste **Type de destination**, sélectionnez **Azure Cosmos DB pour NoSQL**.

1. Dans la liste **Connexion**, sélectionnez **CosmosSqlConn**.

1. Dans la liste**Cible**, sélectionnez **flatproducts**, puis sélectionnez **Suivant** pour passer à l’étape **Paramètres** de l’assistant.

1. Dans l’étape **Paramètres** de l’assistant, dans le champ **Nom de la tâche**, entrez **`FlattenAndMoveData`**.

1. Laissez tous les champs restants à leurs valeurs vides par défaut, puis sélectionnez **Suivant** pour passer à l’étape finale de l’assistant.

1. Passez en revue le **Résumé** des étapes que vous avez sélectionnées dans l’assistant, puis sélectionnez **Suivant**.

1. Observez les différentes étapes du déploiement. Une fois le déploiement terminé, sélectionnez **Terminer**.

1. Revenez à l’onglet du navigateur avec votre **compte Azure Cosmos DB** et accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, sélectionnez le nœud de conteneur **flatproducts**, puis sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne tous les documents où le **name** est équivalent à **HL Headset** :

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        flatproducts p
    WHERE
        p.name = 'HL Headset'
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les résultats de la requête.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
