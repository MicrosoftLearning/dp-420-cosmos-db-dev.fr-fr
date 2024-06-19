---
lab:
  title: Utiliser Azure Monitor pour analyser un compte Azure Cosmos DB for NoSQL
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Utiliser Azure Monitor pour analyser un compte Azure Cosmos DB for NoSQL

Azure Monitor est un service d’analyse de pile complet dans Azure qui fournit un ensemble complet de fonctionnalités pour surveiller les ressources Azure.  Azure Cosmos DB crée des données d’analyse à l’aide de Azure Monitor.  Azure Monitor capture les métriques et les données de télémétrie d’Cosmos DB.

Dans ce labo, vous allez exécuter une charge de travail simulée sur des conteneurs Azure Cosmos DB et analyser la façon dont cette charge de travail affecte le compte Azure Cosmos DB.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement où vous travaillez sur ce labo, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git : Cloner** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Une fois le compte Azure Cosmos DB for NoSQL approvisionné, vous pouvez récupérer le point de terminaison et la clé. Utilisez le point de terminaison et la clé pour vous connecter par programmation au compte Azure Cosmos DB for NoSQL. Utilisez le point de terminaison et la clé sur les chaînes de connexion du Kit de développement logiciel (SDK) Azure pour .NET ou un autre SDK.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au Portail Azure (``portal.azure.com``).

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
    | **Limiter la quantité totale de débit pouvant être approvisionnée sur ce compte** | *Décochez la case* |

    > &#128221; Vos environnements de labo peuvent avoir des restrictions qui vous empêchent de créer un nouveau groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Notez le champ **URI**. Vous allez utiliser cette valeur de **point de terminaison** plus tard dans cet exercice.

    1.Notez le champ **CLÉ PRINCIPALE**. Vous allez utiliser cette valeur de **clé** plus tard dans cet exercice.

1. Réduisez la fenêtre de votre navigateur, mais ne la fermez pas. Nous reviendrons au portail Azure peu après avoir démarré une charge de travail en arrière-plan dans les étapes suivantes.


## Importer les bibliothèques Microsoft.Azure.Cosmos et Newtonsoft.Json dans un script .NET

L’interface CLI .NET comprend une commande [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] pour importer des packages depuis un flux de package préconfiguré. Une installation .NET utilise NuGet comme flux de package par défaut.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **25-monitor**.

1. Ouvrez le menu contextuel du dossier **25-monitor**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande va ouvrir le terminal avec le répertoire de démarrage déjà défini sur le dossier **25-monitor**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] depuis NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Ajoutez le package [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] depuis NuGet en utilisant la commande suivante :

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

## Exécuter un script pour créer les conteneurs et la charge de travail

Nous sommes maintenant prêts à exécuter une charge de travail pour surveiller son utilisation du compte Azure Cosmos DB.  Le script que nous allons exécuter, en arrière-plan. Ce script va créer trois conteneurs et charger quelques données dans ces conteneurs. Le script va ensuite exécuter quelques requêtes SQL de manière aléatoire pour émuler plusieurs applications utilisateur ciblant le compte Azure Cosmos DB. 

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **25-monitor**.

1. Ouvrez le fichier de code **Program.cs**.

1. Mettez à jour la variable existante nommée **endpoint** avec sa valeur définie sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/**, l’instruction C# sera : **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Mettez à jour la variable existante nommée **key** avec sa valeur définie sur la **clé** du compte Azure Cosmos DB que vous avez créé précédemment.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est : **fDR2ci9QgkdkvERTQ==**, l’instruction C# sera : **private static readonly string key = "fDR2ci9QgkdkvERTQ==";**.

1. Enregistrez le fichier **Program.cs**.

1. Revenez au *terminal intégré*.

1. Créez et exécutez le projet en utilisant la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```
    > &#128221; La première partie de ce script crée les trois conteneurs et y charge les données ; ceci doit prendre environ 2 minutes. Pour émuler certains événements de limitation de débit, le script définit ensuite le débit approvisionné sur 400 RU/s. Vous devez ensuite recevoir le message ***Creating simulated background workload, wait 5-10 minutes and go to the next step of the exercise.*** (Création d’une charge de travail en arrière-plan simulée, attendez 5 à 10 minutes et passez à l’étape suivante de l’exercice.). Étant donné que les ressources Azure chargent des données de supervision dans Azure Monitor de manière asynchrone, nous devons attendre un court instant pour commencer à obtenir des données de diagnostic dans les métriques et les insights Azure Monitor. Après 5 à 10 minutes, passez à l’étape suivante. Si vous le souhaitez, pour collecter des données de diagnostic supplémentaires, vous pouvez ne pas arrêter le script après 5 à 10 minutes et attendre simplement la fin du labo pour l’arrêter.

    > &#128221; Vous remarquerez quelques avertissements en jaune, car le compilateur détecte que le script exécute de nombreuses opérations de manière synchrone et n’attend pas de réponse des opérations. Vous pouvez ignorer ces avertissements, car c’est le comportement attendu pour l’exécution simultanée de plusieurs scripts SQL.

## Utiliser Azure Monitor pour analyser l’utilisation du compte Azure Cosmos DB

Dans cette partie de l’exercice, nous allons revenir au navigateur et passer en revue quelques-uns des rapports sur les insights et les métriques Azure Monitor.

### Rapports sur les métriques Azure Monitor

1. Revenez à la fenêtre de navigateur ouverte que nous avons réduite précédemment. Si vous l’avez fermée, ouvrez-en une nouvelle, puis accédez à votre page du compte Azure Cosmos DB sous portal.azure.com.

1. Dans le menu de gauche d’Azure Cosmos DB, sous *Surveillance*, sélectionnez **Métriques**. Vous remarquerez que les champs **Étendue** et **Espace de noms de métriques** sont préremplis avec les informations appropriées. Dans les étapes suivantes, nous allons examiner quelques options de **Métrique**, et les fonctionnalités *Ajouter un filtre* et *Appliquer un fractionnement*.

1. Par défaut, la section *Métriques* affiche les informations de diagnostic des dernières 24 heures. Nous devons être plus précis pour examiner les métriques pendant l’exécution de la charge de travail que nous avons créée à l’étape précédente. Dans le coin supérieur droit, sélectionnez le bouton intitulé ***Heure locale : Les dernières 24 heures (automatique)*** : nous obtenons ensuite une fenêtre montrant plusieurs options d’intervalle de temps avec une case d’option.  Choisissez la case d’option intitulée ***30 dernières minutes***, puis sélectionnez le bouton **Appliquer**. Si nécessaire, vous pouvez obtenir une granularité plus fine en choisissant la case d’option *Personnalisée*, et une date et une heure de début et de fin. 

1. Maintenant que nous avons un bon intervalle de temps pour nos graphiques de diagnostic, examinons quelques métriques. Nous allons commencer par une métrique courante. Dans le menu déroulant *Métrique*, choisissez **Nombre total d’unités de requête**. Par défaut, cette métrique est affichée en tant que somme totale des unités de requête (RU). Vous pouvez également changer le menu déroulant Agrégation en Moyenne ou en Maximum. Une fois que vous avez essayé ces deux agrégations, redéfinissez-le sur *Somme* pour les étapes suivantes.

1. Cette métrique nous donne une bonne idée du nombre d’unités de requête utilisées dans notre compte Azure Cosmos DB. Cependant, notre graphique actuel peut ne pas nous aider à identifier un problème quand nous avons plusieurs bases de données ou conteneurs dans notre compte. Nous allons changer cela : examinons notre consommation de RU par base de données. Dans le menu sous le titre du graphique, sélectionnez **Appliquer le fractionnement** ; sous le menu déroulant **Valeurs**, choisissez **DatabaseName**, puis cliquez n’importe où dans le graphique pour accepter les modifications. Un bouton **Fractionner par = DatabaseName** apparaît maintenant juste au-dessus du graphique. 

1. C’est beaucoup mieux : nous savons maintenant quelle est la base de données qui fait le plus gros du travail. Bien que ces informations soient intéressantes, nous n’avons aucune idée du conteneur qui fait tout le travail.  Sélectionnez le bouton **Fractionner par = DatabaseName** pour modifier la condition du fractionnement, puis choisissez **CollectionName** dans le menu déroulant *Valeurs*. Nous devrions maintenant avoir des données pour nos collections **customer** et **salesOrder**. Il n’y a qu’un problème avec ce graphique : la collection **salesOrder** existe dans deux bases de données, **database-v2** et **database-v3**. Cette valeur est donc une agrégation des noms de cette collection dans les deux bases de données.

1. Ce doit être facile à résoudre : sélectionnez le bouton **Ajouter un filtre** ; sous le menu déroulant *Propriété*, choisissez **DatabaseName** puis, sous *Valeurs*, choisissez **database-V3**.

1. Examinons deux autres métriques. Nous allons modifier le graphique existant, mais vous pouvez aussi créer un nouveau graphique si vous le souhaitez. Au-dessus du graphique, sélectionnez le bouton avec le *nom du compte Azure Cosmos DB* et l’étiquette **Nombre total d’unités de requête**. Choisissez **Nombre total de requêtes** dans le menu déroulant *Métrique* ; notez que la seule agrégation disponible est *Nombre*.

1. Deux filtres clés peuvent nous aider ici à résoudre différents types de problèmes. Ajoutons un filtre avec la propriété **StatusCode** (notez qu’un filtre similaire avec un type de détail différent serait **État**) ; pour *Valeurs*, choisissez **200** et **429**. Changez le fractionnement pour qu’il utilise StatusCode. Notez qu’il y a une énorme quantité de 429, qui sont des requêtes de limitation de débit, par rapport aux états 200, qui sont des requêtes réussies. Les exceptions 429 se sont produites en raison du fait que le script envoie des milliers de requêtes par seconde, alors que nous définissons le débit approvisionné sur 400 RU/s. *Ce grand nombre d’exceptions 429 par rapport aux requêtes réussies ne doit pas être considéré comme normal sur un environnement de production. Dans un environnement de production, les exceptions 429 ne doivent se produire que rarement dans un compte Azure Cosmos DB sain*.  Vous pouvez également utiliser les *Propriétés* **StatusCode** ou **État** de manière similaire pour résoudre les problèmes posés par **Nombre total d’unités de requête**.

1. Examinons le **Nombre total de requêtes**, mais en changeant le fractionnement en **OperationType**.  Cette propriété va nous aider à déterminer quelles opérations de lecture ou d’écriture effectuent la majeure partie du travail. Là encore, cette propriété peut être utilisée de façon similaire pour **Nombre total d’unités de requête**.

1. Comme nous l’avons fait avec le **Nombre total d’unités de requête**, essayez en choisissant différents filtres et options de fractionnement. 

1. La métrique finale que nous allons examiner dans cet exercice est la métrique **Consommation RU normalisée**. Changez votre fractionnement en **PartitionKeyRangeId**. Cette métrique nous aide à identifier la plage de clés de partition la plus utilisée. La métrique nous donne la tendance du débit pour une plage de clés de partition. Continuez et choisissez cette métrique dans le menu déroulant *Métrique*. Ce graphique doit maintenant nous montrer un système véritablement peu sain, atteignant une **Consommation RU normalisée** constante de 100 %.

> &#128221; Si vous souhaitez examiner plusieurs graphiques à la fois, cliquez sur l’option **+ Nouveau graphique** au-dessus du nom du graphique. 

> &#128221; Nous ne pouvons pas enregistrer directement nos métriques, mais vous pouvez créer ou utiliser un tableau de bord existant et y ajouter ce graphique en cliquant sur le bouton **Épingler au tableau de bord** dans le coin supérieur droit du graphique.  Cliquez sur le bouton et choisissez l’onglet **Créer un nouveau**, donnez-lui le nom *Labos DP-420*, puis cliquez sur **Créer et épingler**. Pour visualiser vos tableaux de bord privés, vous devez accéder au menu du portail dans le coin supérieur gauche, puis choisir Tableau de bord dans vos options de ressource Azure. Le tableau de bord peut prendre quelques minutes pour apparaître la première fois.

> &#128221; Une autre façon de partager votre graphique consiste à cliquer sur le menu déroulant Partager et à le télécharger en tant que fichier Excel ou à utiliser l’option Copier le lien.

### Rapports sur les insights Azure Monitor

Il peut être nécessaire de passer un peu de temps à peaufiner nos rapports de diagnostic des métriques Azure Monitor.  Les insights Cosmos DB fournissent une vue de l’ensemble des performances, des défaillances et de l’intégrité opérationnelle de vos ressources Azure Cosmos DB. Ces graphiques d’insights seront des graphiques prédéfinis similaires à ceux des métriques. Examinons certains d’entre eux.

1. Dans le menu de gauche d’Azure Cosmos DB, sous *Surveillance*, sélectionnez **Insights**. Vous remarquerez qu’il existe plusieurs onglets, qui vont de Vue d’ensemble à Options de gestion. Nous allons examiner quelques-uns de ces graphiques d’**insights**. Le premier onglet, Vue d’ensemble, fournit un résumé des graphiques les plus courants que vous pouvez utiliser. Par exemple, des graphiques tels que Nombre total de requêtes, Utilisation des données et des index, Exceptions 429 et Consommation RU normalisée.  Nous avons vu la plupart de ces graphiques dans la section précédente.

1. Notez qu’en haut des graphiques, nous choisir l’**Intervalle de temps** : sélectionnez donc *15* ou *30* minutes pour évaluer la charge de travail dans cet exercice.

1. Dans le coin supérieur droit de *chaque* graphique, vous remarquerez une option pour ***Ouvrir Metrics Explorer***. Commençons par sélectionner l’option **Ouvrir Metrics Explorer** pour le graphique **Nombre total de requêtes**. Notez que quand vous sélectionnez cette option, vous accédez aux rapports de métriques que nous avons examinés précédemment. L’avantage d’ouvrir Metrics Explorer est qu’une bonne partie du graphique a déjà été créée pour nous.

1. Revenons à la page Insights en sélectionnant **X** en haut à droite du graphique de métriques.

1. Sélectionnez l’onglet Débit. Ces graphiques permettent de mettre en évidence les problèmes de débit.  Observez attentivement le graphique **Consommation RU normalisée (%) par PartitionKeyRangeID**, qui peut être utilisé pour détecter les partitions les plus utilisées.

1. Sélectionnez l’onglet Requêtes. Ces graphiques sont très utiles pour analyser le nombre d’événements de limitation rencontrés par le compte (429 par rapport à 200) ou pour passer en revue le nombre de requêtes par type d’opérations.  

1. Sélectionnez l’onglet Stockage. Ces graphiques nous montrent à la fois la croissance de nos collections, et l’utilisation des données et des index.  

1. Sélectionnez l’onglet Système. Si votre application créait, supprimait ou interrogeait fréquemment les métadonnées des comptes, il est possible qu’il y ait des exceptions 429.  Ces graphiques nous aident à déterminer si cet accès fréquent aux métadonnées est la cause de nos exceptions 429. En outre, nous pouvons déterminer l’état de nos requêtes de métadonnées.  

### Rapports sur les insights Azure Monitor

1. Si le programme est toujours en cours d’exécution, revenez au terminal de commandes de Visual Studio Code.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[nuget.org/packages/Newtonsoft.Json/13.0.1]: https://www.nuget.org/packages/Newtonsoft.Json/13.0.1
