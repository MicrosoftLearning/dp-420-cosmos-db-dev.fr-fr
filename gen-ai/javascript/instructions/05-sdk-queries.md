---
lab:
  title: "05 - Exécuter une requête avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Query the Azure Cosmos DB for NoSQL
---

# Exécuter une requête avec le SDK Azure Cosmos DB for NoSQL

La dernière version du kit de développement logiciel (SDK) JavaScript pour Azure Cosmos DB for NoSQL simplifie l’interrogation d’un conteneur et l’itération sur les jeux de résultats à l’aide des fonctionnalités modernes de JavaScript.

La bibliothèque `@azure/cosmos` dispose de fonctionnalités intégrées pour rendre l’interrogation d’Azure Cosmos DB efficace et simple.

Dans ce labo, vous allez utiliser un itérateur pour traiter un jeu de résultats volumineux retourné par Azure Cosmos DB for NoSQL. Vous allez utiliser le kit de développement logiciel (SDK) JavaScript pour l’interrogation et l’itération sur les résultats.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des copilotes avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#create-azure-cosmos-db-database-and-container-with-sample-data). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Créer une base de données et un conteneur Azure Cosmos DB avec des exemples de données

Si vous avez déjà créé une base de données Azure Cosmos DB nommée **cosmoworks-full** et un conteneur dans celle-ci nommé **produits**, qui est préchargé avec des exemples de données, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#import-the-azurecosmos-library). Sinon, suivez les étapes ci-dessous pour créer un exemple de base de données et de conteneur.

<details markdown=1>
<summary markdown="span"><strong>Cliquez pour développer/réduire les étapes de création de base de données et de conteneur avec des exemples de données.</strong></summary>

1. Dans la ressource de compte **Azure Cosmos DB** nouvellement créée, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, sélectionnez **Lancer le démarrage rapide** sur la page d’accueil.

1. Dans le formulaire **Nouveau conteneur**, entrez les valeurs suivantes :

    - **ID de base de données** : `cosmicworks-full`
    - **ID de conteneur** : `products`
    - **Clé de partition** : `/categoryId`
    - **Magasin analytique** : `Off`

1. Sélectionnez **OK** pour créer le conteneur. Ce processus prend une minute ou deux pendant qu’il crée les ressources et précharge le conteneur avec des exemples de données de produit.

1. Gardez l’onglet du navigateur ouvert, car nous y retournerons ultérieurement.

1. Revenez à **Visual Studio Code**.

</details>

## Importer la bibliothèque @azure/cosmos

La bibliothèque **@azure/cosmos** est disponible sur **npm** pour faciliter l’installation dans vos projets JavaScript.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/05-sdk-queries**.

1. Ouvrez le menu contextuel du dossier **javascript/05-sdk-queries**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **javascript/05-sdk-queries**.

1. Initialisez un nouveau projet Node.js :

    ```bash
    npm init -y
    ```

1. Installez le package [@azure/cosmos][npmjs.com/package/@azure/cosmos] à l’aide de la commande suivante :

    ```bash
    npm install @azure/cosmos
    ```

1. Installez la bibliothèque [@azure/identity][npmjs.com/package/@azure/identity], qui nous permet d’utiliser l’authentification Azure pour se connecter à l’espace de travail Azure Cosmos DB, à l’aide de la commande suivante :

    ```bash
    npm install @azure/identity
    ```

## Interroger les résultats d’une requête SQL à l’aide du kit de développement logiciel

À l’aide des informations d’identification du compte nouvellement créé, vous allez vous connecter aux classes du kit de développement logiciel (SDK) et vous connecter à la base de données et au conteneur que vous avez provisionnés dans une étape précédente, puis effectuer une itération sur les résultats d’une requête SQL à l’aide du kit de développement logiciel (SDK).

Vous allez à présent utiliser un itérateur pour créer une boucle simple à comprendre sur les résultats paginés d’Azure Cosmos DB. En arrière-plan, le kit de développement logiciel (SDK) gère l’itérateur de flux et vérifie que les demandes suivantes sont appelées correctement.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/05-sdk-queries**.

1. Ouvrez le fichier JavaScript vide nommé **script.js**.

1. Ajoutez les instructions `require` suivantes pour importer les bibliothèques **@azure/cosmos** et **@azure/identity** :

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. Ajoutez les variables nommées **endpoint** et **credential**, puis définissez la valeur **endpoint** sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment. La variable **credential** doit être définie sur une nouvelle instance de la classe **DefaultAzureCredential** :

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; Par exemple, si votre point de terminaison est **https://dp420.documents.azure.com:443/**, l’instruction est **const endpoint = "https://dp420.documents.azure.com:443/";**.

1. Ajoutez une nouvelle variable nommée **client** et initialisez-la en tant que nouvelle instance de la classe **CosmosClient** à l’aide des variables **endpoint** et **credential** :

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. Créez une méthode nommée **queryContainer** et du code pour exécuter cette méthode lorsque vous exécutez le script. Vous allez ajouter le code pour interroger le conteneur dans cette méthode :

    ```javascript
    async function queryContainer() {
        // Query the container
    }

    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. Dans la méthode **queryContainer**, ajoutez le code suivant pour vous connecter à la base de données et au conteneur que vous avez créés précédemment :

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. Créez une variable de chaîne de requête nommée `sql` avec la valeur `SELECT * FROM products p`.

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. Appelez la méthode [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) avec la variable `sql` en tant que paramètre au constructeur. Le paramètre `enableCrossPartitionQuery`, lorsqu’il est défini sur `true`, permet d’envoyer plusieurs demandes pour exécuter la requête dans le service Azure Cosmos DB. Plusieurs demandes sont nécessaires si la requête n’est pas limitée à une seule valeur de clé de partition.

    ```javascript
    const iterator = container.items.query(
        query,
        { enableCrossPartitionQuery: true }
    );
    ```

1. Itérez sur les résultats paginés et affichez l’`id`, le `name` et le `price` de chaque élément :

    ```javascript
    while (iterator.hasMoreResults()) {
        const { resources } = await iterator.fetchNext();
        for (const item of resources) {
            console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
        }
    }
    ```

1. Votre fichier **script.js** doit maintenant ressembler à ceci :

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function queryContainer() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products p";
    
        const iterator = container.items.query(
            query,
            { enableCrossPartitionQuery: true }
        );
        
        while (iterator.hasMoreResults()) {
            const { resources } = await iterator.fetchNext();
            for (const item of resources) {
                console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
            }
        }
    }
    
    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. **Enregistrez** le fichier **script.js**.

1. Avant d’exécuter le script, vous devez vous connecter à Azure à l’aide de la commande `az login`. Dans la fenêtre de terminal, exécutez :

    ```bash
    az login
    ```

1. Exécutez le script pour créer la base de données et le conteneur :

    ```bash
    node script.js
    ```

1. Le script génère à présent chaque produit dans le conteneur.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
