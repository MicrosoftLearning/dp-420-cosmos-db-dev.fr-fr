---
lab:
  title: Créer une procédure stockée avec le portail Azure
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Créer une procédure stockée avec le portail Azure

Les procédures stockées sont l’une des façons dont vous pouvez exécuter la logique métier côté serveur dans Azure Cosmos DB. Avec une procédure stockée, vous pouvez effectuer des opérations CRUD (créer, lire, mettre à jour, supprimer) de base avec un conteneur sur plusieurs documents dans une seule étendue transactionnelle.

Dans ce laboratoire, vous allez créer une procédure stockée qui crée un document dans votre conteneur. Vous allez ensuite utiliser une requête SQL pour valider les résultats de la procédure stockée.

## Créer une procédure stockée

Les procédures stockées sont créées en JavaScript intégré au langage et prennent en charge l’exécution d’opérations CRUD de base à l’intérieur du moteur de base de données. L'exécution de JavaScript à l'intérieur du moteur de base de données est rendue possible grâce au Kit de développement logiciel (SDK) JavaScript côté serveur pour Azure Cosmos DB et à une série de méthodes d’assistance.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionnez un groupe de ressources ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent avoir des restrictions qui vous empêchent de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources précréé existant.

1. Attendez que la tâche de déploiement se termine avant de poursuivre.

1. Accédez à la ressource de compte **Azure Cosmos DB** nouvellement créée et accédez au volet **Explorateur de données**.

1. Dans **Explorateur de données**, sélectionnez **Nouveau conteneur**, puis créez un conteneur avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **ID de base de données** | *Créer nouveau* &vert; *``cosmicworks``* |
    | **Partager le débit entre les conteneurs** | *Sélectionnez cette option* |
    | **Débit de la base de données** | *Manuel* &vert; *400* |
    | **ID de conteneur** | *``products``* |
    | **Indexation** | *Automatique* |
    | **Clé de partition** | *``/categoryId``* |

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis sélectionnez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez **Nouvelle procédure stockée**.

1. Dans le champ **ID de la procédure stockée**, entrez la valeur **createDoc**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une fonction JavaScript nommée **createDoc** sans paramètres d’entrée :

    ```
    function createDoc() {
        
    }
    ```

1. Dans la fonction **createDoc**, appelez la méthode [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] intégrée et stockez le résultat dans une variable nommée **context** :

    ```
    var context = getContext();
    ```

1. Appelez la méthode [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] de l’objet de contexte et stockez le résultat dans une variable nommée **container** :

    ```
    var container = context.getCollection();
    ```

1. Créez un objet nommé **doc** avec deux propriétés :

    | **Propriété** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *premier document* |
    | **ID de la catégorie** | *démo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. Appelez la méthode **createDocument** de l’objet conteneur en passant en paramètres le résultat de l’appel de la méthode **getSelfLink** de l’objet conteneur et le nouveau document :

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. Une fois que vous avez terminé, votre code de procédure stockée doit maintenant inclure :

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. Sélectionnez **Enregistrer** pour conserver les modifications apportées à la procédure stockée.

1. Sélectionnez **Exécuter**, puis exécutez la procédure stockée à l’aide des paramètres d’entrée suivants :

    | **Paramètre** | **Clé** | **Valeur** |
    | ---: | :--- | :--- |
    | **Valeur de clé de partition** | *Chaîne* | *démo* |

1. Observez le résultat vide. Bien que la procédure stockée s’exécute correctement, le code JavaScript n’a jamais retourné une réponse lisible par un humain.

## Implémenter les meilleures pratiques pour une procédure stockée

Bien que la procédure stockée créée précédemment dans ce laboratoire dispose de fonctionnalités de base, elle manque également de certaines techniques courantes de gestion des erreurs courantes qui doivent être implémentées dans toutes les procédures stockées. Tout d’abord, la procédure stockée suppose qu’elle aura toujours le temps d’effectuer l’opération et ne vérifie pas la valeur renvoyée de la méthode **createDocument** pour s’assurer qu’elle a suffisamment de temps. Ensuite, la procédure stockée part du principe que tous les documents sont correctement insérés sans vérifier ou lever des messages d’erreur potentiels. Enfin, la procédure stockée ne retourne pas le document nouvellement créé comme réponse HTTP pour la requête qui a appelé la procédure stockée à l’origine. Vous allez apporter ces trois modifications à la procédure stockée pour implémenter les meilleures pratiques courantes.

1. Revenez à l’éditeur pour la procédure stockée **createDoc**.

1. Recherchez la ligne 1 dans le code qui définit la fonction **createDoc** :

    ```
    function createDoc() {
    ```

    et mettez à jour la ligne de code pour inclure un paramètre nommé **title** :

    ```
    function createDoc(title) {
    ```

1. Recherchez la ligne 5 dans le code qui définit la propriété **name** de l’objet **doc** :

    ```
    name: 'first document',
    ```

    et mettez à jour la ligne de code pour utiliser la valeur du paramètre **title** :

    ```
    name: title,
    ```

1. Recherchez la ligne 8 dans le code qui appelle la méthode **createDocument** :

    ```
    container.createDocument(
    ```

    et mettez à jour la ligne de code pour stocker le résultat de l’appel de méthode dans une variable nommée **accepted**

    ```
    var accepted = container.createDocument(
    ```

1. Ajoutez une nouvelle ligne de code après l’appel de méthode **createDocument** pour vérifier la valeur de la variable **accepted** et renvoyer la méthode si elle n’est pas vraie :

    ```
    if (!accepted) return;
    ```

1. Enfin, ajoutez un troisième paramètre à l’appel de la méthode **createDocument**, qui est une fonction prenant deux paramètres nommés **error** et **newDoc**, vérifiant si l’erreur est null, puis assignant newDoc au corps de la réponse de la procédure stockée :

    ```
    ,
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. Une fois que vous avez terminé, votre code de procédure stockée doit maintenant inclure :

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. Sélectionnez **Mettre à jour** pour conserver les modifications apportées à la procédure stockée.

1. Sélectionnez **Exécuter**, puis exécutez la procédure stockée à l’aide des paramètres d’entrée suivants :

    | **Paramètre** | **Clé** | **Valeur** |
    | ---: | :--- | :--- |
    | **Valeur de clé de partition** | *Chaîne* | *démo* |
    | **Paramètres d'entrée** | *Chaîne* | *deuxième document* |

1. Observez le résultat JSON. Une fois la procédure stockée exécutée avec succès, le document nouvellement créé a été retourné en réponse à la requête HTTP d’origine.

## Interroger des documents

Pour conclure, vous utiliserez l’Explorateur de données pour émettre une requête SQL qui renverra les deux documents créés dans ce labo.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez **Nouvelle requête SQL**.

1. Supprimez le contenu de la zone de l’éditeur.

1. Créez une requête SQL qui retourne tous les documents où le **categoryId** équivaut à **demo** :

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. Sélectionnez **Exécuter la requête**.

1. Observez les deux documents que vous avez créés dans ce labo en tant que résultats de l’exécution de cette requête.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
