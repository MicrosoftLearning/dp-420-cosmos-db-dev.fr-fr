---
title: "03 - Créer et mettre à jour des documents avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "03 - Créer et mettre à jour des documents avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: JavaScript SDK labs
---

# Créer et mettre à jour des documents avec le Kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL

La bibliothèque `@azure/cosmos` inclut des méthodes pour créer, récupérer, mettre à jour et supprimer des éléments (CRUD) dans un conteneur Azure Cosmos DB for NoSQL. Ensemble, ces méthodes permettent d’effectuer certaines des opérations « CRUD » les plus courantes sur divers éléments dans des conteneurs d’API NoSQL.

Dans ce labo, vous allez utiliser le kit de développement logiciel (SDK) JavaScript pour effectuer des opérations CRUD courantes sur un élément dans un conteneur Azure Cosmos DB for NoSQL.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des Copilots avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#import-the-azurecosmos-library). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Importer la bibliothèque @azure/cosmos

La bibliothèque **@azure/cosmos** est disponible sur **npm** pour faciliter l’installation dans vos projets JavaScript.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/03-sdk-crud**.

1. Ouvrez le menu contextuel du dossier **javascript/03-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **javascript/03-sdk-crud**.

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

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/03-sdk-crud**.

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
    const { CosmosClient } = require("@azure/cosmos");
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

## Effectuer des opérations de création et de lecture de point sur des éléments avec le kit de développement logiciel

Vous allez maintenant utiliser l’ensemble de méthodes dans la classe **Container** pour effectuer des opérations courantes sur des éléments dans un conteneur d’API NoSQL.

1. Revenez à **Visual Studio Code**. S’il n’est pas encore ouvert, ouvrez le fichier de code **script.js** dans le dossier **javascript/03-sdk-crud**.

1. Créez un élément de produit et attribuez-le à une variable nommée **selle** avec les propriétés suivantes. Veillez à ajouter le code suivant dans la fonction `main` :

    | Propriété | Valeur |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **p**rice | *45.99d* |
    | **balises** | *{ tan, new, crisp }* |

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };
    ```

1. Appelez la méthode [`create`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-create) de la classe **items ** du conteneur, en transmettant la variable **selle** en tant que paramètre de méthode :

    ```javascript
    const { resource: item } = await container
        .items.create(saddle);
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const saddle = {
            id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
            categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
            name: "Road Saddle",
            price: 45.99,
            tags: ["tan", "new", "crisp"]
        };
    
        const { resource: item } = await container
                .items.create(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Enregistrez** et réexécutez le script :

    ```bash
    node script.js
    ```

1. Observez le nouvel élément dans l’**Explorateur de données**.

1. Revenez à **Visual Studio Code**.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.js**.

1. Supprimez les lignes de code suivantes :

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };

    const { resource: item } = await container
            .items.create(saddle);
    ```

1. Créez une variable de chaîne nommée **item_id** avec la valeur **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** :

    ```javascript
    const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Créez une variable de chaîne nommée **partition_key** avec la valeur **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Appelez la méthode [`read`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-read) de la classe **item** du conteneur, en transmettant les variables **itemId** et **partitionKey** en tant que paramètres de méthode :

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();
    ```

    > &#128161; La méthode `read` vous permet d’effectuer une opération de lecture de point sur un élément du conteneur. La méthode nécessite que les paramètres `itemId` et `partitionKey` identifient l’élément à lire. Contrairement à l’exécution d’une requête à l’aide du langage de requête SQL de Cosmos DB pour rechercher un élément unique, la méthode `read` est plus efficace et économique pour récupérer un seul élément. Les lectures de points peuvent lire les données directement et n’ont pas besoin du moteur d’interrogation pour traiter la demande.

1. Affichez l’objet selle à l’aide d’une chaîne de sortie mise en forme :

    ```javascript
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();
    
        console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Enregistrez** et réexécutez le script :

    ```bash
    node script.js
    ```

1. Observez la sortie du terminal. Plus précisément, observez le texte de sortie mis en forme avec l’ID, le nom et le prix de l’élément.

## Effectuer des opérations de mise à jour et de suppression de point avec le kit de développement logiciel

Dans le cadre de votre apprentissage du kit de développement logiciel (SDK), il n’est pas rare d’utiliser un compte Azure Cosmos DB en ligne ou l’émulateur pour mettre à jour un élément, puis de basculer successivement entre l’Explorateur de données et l’IDE de votre choix pour effectuer une opération et vérifier si la modification a été appliquée. C’est exactement ce que vous allez faire ici en mettant à jour et en supprimant un élément à l’aide du kit SDK.

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Sélectionnez le seul élément dans le conteneur, puis observez les valeurs des propriétés **name** et **price** de l’élément.

    | **Propriété** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *Road Saddle* |
    | **Tarif** | *$45.99* |

    > &#128221; À ce stade, ces valeurs ne doivent pas avoir été modifiées depuis la création de l’élément. Vous allez modifier ces valeurs dans cet exercice.

1. Revenez à **Visual Studio Code**. Revenez à l’onglet de l’éditeur pour le fichier de code **script.js**.

1. Supprimez la ligne de code suivante :

    ```javascript
    console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    ```

1. Modifiez la variable **saddle** en définissant la valeur de la propriété price sur **32.55** :

    ```javascript
    // Update the item
    saddle.price = 32.55;
    ```

1. Modifiez à nouveau la variable **saddle** en définissant la valeur de la propriété **name** sur **Road LL Saddle** :

    ```javascript
    saddle.name = "Road LL Saddle";
    ```

1. Appelez la méthode [`replace`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-replace) de la classe **item** du conteneur, en transmettant la variable **selle** en tant que paramètre de méthode :

    ```javascript
    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();

        // Update the item
        saddle.price = 32.55;
        saddle.name = "Road LL Saddle";
    
        await container.item(saddle.id, partitionKey).replace(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Enregistrez** et réexécutez le script :

    ```bash
    node script.js
    ```

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Sélectionnez le seul élément dans le conteneur, puis observez les valeurs des propriétés **name** et **price** de l’élément.

    | **Propriété** | **Valeur** |
    | ---: | :--- |
    | **Nom** | *Road LL Saddle* |
    | **Tarif** | *$32.55* |

    > &#128221; À ce stade, ces valeurs doivent être différentes de celles que vous avez observées précédemment.

1. Revenez à **Visual Studio Code**. Revenez à l’onglet de l’éditeur pour le fichier de code **script.js**.

1. Supprimez les lignes de code suivantes :

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();

    // Update the item
    saddle.price = 32.55;
    saddle.name = "Road LL Saddle";

    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Appelez la méthode [`delete`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-delete) de la classe **item** du conteneur, en transmettant les variables **itemId** et **partitionKey** en tant que paramètres de méthode :

    ```javascript
    // Delete the item
    await container.item(itemId, partitionKey).delete();
    ```

1. Enregistrez et réexécutez le script :

    ```bash
    node script.js
    ```

1. Fermez le terminal intégré.

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Notez que la liste des éléments est désormais vide.

1. Fermez la fenêtre ou l’onglet de votre navigateur web.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
