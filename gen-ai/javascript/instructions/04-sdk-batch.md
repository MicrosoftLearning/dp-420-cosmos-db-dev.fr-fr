---
title: "04 - Traiter par lots plusieurs opérations de point avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "04 - Traiter par lots plusieurs opérations de point avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 7
parent: JavaScript SDK labs
---

# Traiter par lots plusieurs opérations de point avec le kit SDK Azure Cosmos DB for NoSQL

La classe `TransactionalBatch` du kit de développement logiciel (SDK) JavaScript pour Azure Cosmos DB fournit des fonctionnalités permettant de composer et d’exécuter des opérations par lots dans la même clé de partition logique. À l’aide de cette fonctionnalité, vous pouvez effectuer plusieurs opérations dans une seule transaction et vérifier que toutes ou aucune des opérations ne sont terminées.

Dans ce labo, vous utilisez le kit de développement logiciel (SDK) JavaScript pour effectuer deux opérations à deux éléments dans lesquels vous tentez de créer deux éléments en tant qu’unité logique unique.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des Copilots avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#import-the-azurecosmos-library). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Importer la bibliothèque @azure/cosmos

La bibliothèque **@azure/cosmos** est disponible sur **npm** pour faciliter l’installation dans vos projets JavaScript.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/04-sdk-batch**.

1. Ouvrez le menu contextuel du dossier **javascript/04-sdk-batch**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **javascript/04-sdk-batch**.

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

## Utiliser la bibliothèque @azure/cosmos

Une fois la bibliothèque Azure Cosmos DB du kit de développement logiciel (SDK) Azure pour JavaScript importée, vous pouvez utiliser immédiatement ses classes pour vous connecter à un compte Azure Cosmos DB for NoSQL. La classe **CosmosClient** est la classe principale utilisée pour établir la connexion initiale à un compte Azure Cosmos DB for NoSQL.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/04-sdk-batch**.

1. Ouvrez le fichier JavaScript vide nommé **script.js**.

1. Ajoutez les instructions `require` suivantes pour importer les bibliothèques **@azure/cosmos** et **@azure/identity** :

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
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

1. Ajoutez le code suivant pour créer une base de données et un conteneur s’ils n’existent pas déjà :

    ```javascript
    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. Votre fichier **script.js** doit maintenant ressembler à ceci :

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
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

1. Basculez vers la fenêtre de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

## Création d’un lot transactionnel

Tout d’abord, nous allons créer un lot transactionnel simple qui ajoute deux produits fictifs au conteneur. Ce lot insère une selle usée et un guidon rouillé dans le conteneur avec le même identificateur de catégorie « accessoires utilisés ». Les deux éléments ont la même clé de partition logique, ce qui garantit une opération de traitement par lots réussie.

1. Revenez à **Visual Studio Code**. S’il n’est pas encore ouvert, ouvrez le fichier de code **script.js** dans le dossier **javascript/04-sdk-batch**.

1. Définissez les deux éléments de produit à insérer dans le lot transactionnel :

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    ```

1. Créez un lot transactionnel pour la même clé de partition logique et ajoutez les éléments :

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = container.items.batch(partitionKey)
        .create(saddle)
        .create(handlebar);
    ```

1. Exécutez le lot et affichez l’état de l’opération :

    ```javascript
    const response = await batch.execute();
    console.log(`Status: ${response.statusCode}`);
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: saddle },
            { operationType: BulkOperationType.Create, resourceBody: handlebar },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Enregistrez** et réexécutez le script :

    ```bash
    node script.js
    ```

1. La sortie doit indiquer un code d’état réussi pour chaque opération.

## Création d’un lot transactionnel errant

À présent, nous allons créer un lot transactionnel qui génère une erreur délibérée. Ce lot tente d’insérer deux éléments qui ont des clés de partition logique différentes. Nous allons créer une lampe stroboscopique qui clignote dans la catégorie « accessoires utilisés » et un nouveau casque dans la catégorie « accessoires qualités ». Par définition, il devrait s’agir d'une mauvaise requête et une erreur devrait être renvoyée lors de l’exécution de cette transaction.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.js**.

1. Supprimez les lignes de code suivantes :

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };

    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: saddle },
        { operationType: BulkOperationType.Create, resourceBody: handlebar },
    ];
    ```

1. Modifiez le script pour créer une **lampe stroboscopique** et un **nouveau casque** avec différentes valeurs de clé de partition.

    ```javascript
    const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    ```

1. Créez une variable de chaîne nommée **partition_key** avec la valeur **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Créez un lot avec les éléments **lampe** et **casque** :

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: light },
        { operationType: BulkOperationType.Create, resourceBody: helmet },
    ];
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: light },
            { operationType: BulkOperationType.Create, resourceBody: helmet },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Enregistrez** et réexécutez le script :

    ```bash
    node script.js
    ```

1. Observez la sortie du terminal. Le code d’état des éléments doit être **424** pour **Échec de la dépendance** ou **400** pour **Demande incorrecte**. Cela s’est produit parce que tous les éléments de la transaction ne partage pas la même valeur de clé de partition que le lot transactionnel.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
