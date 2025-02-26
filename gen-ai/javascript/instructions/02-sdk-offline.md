---
title: "02 - Configurer le kit de développement logiciel (SDK) JavaScript Azure\_Cosmos\_DB pour le développement hors connexion"
lab:
  title: "02 - Configurer le kit de développement logiciel (SDK) JavaScript Azure\_Cosmos\_DB pour le développement hors connexion"
  module: Configure the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 5
parent: JavaScript SDK labs
---

# Configurer le kit de développement logiciel (SDK) JavaScript Azure Cosmos DB pour le développement hors connexion

L’émulateur Azure Cosmos DB est un outil local qui émule le service Azure Cosmos DB à des fins de développement et de test. L’émulateur prend en charge l’API NoSQL et peut être utilisé à la place du service cloud lors du développement de code à l’aide du kit de développement logiciel (SDK) Azure pour JavaScript.

Dans ce labo, vous allez vous connecter à l’émulateur Azure Cosmos DB à partir du kit de développement logiciel (SDK) Azure pour JavaScript.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des Copilots avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Démarrer l’émulateur Azure Cosmos DB

Si vous utilisez un environnement de labo hébergé, l’émulateur doit déjà y être installé. Dans le cas contraire, reportez-vous aux [Instructions d’installation](https://docs.microsoft.com/azure/cosmos-db/local-emulator) pour installer l’émulateur Azure Cosmos DB. Une fois l’émulateur démarré, vous pouvez récupérer la chaîne de connexion et l’utiliser pour vous connecter à l’émulateur à l’aide du kit de développement logiciel (SDK) Azure pour JavaScript.

> &#128161; Vous pouvez éventuellement installer le [nouvel émulateur Azure Cosmos DB basé sur Linux (en préversion)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux) qui est disponible en tant que conteneur Docker. Il prend en charge l’exécution sur une grande variété de processeurs et de systèmes d’exploitation.

1. Démarrez **l’émulateur Azure Cosmos DB**.

    > 💡 Si vous utilisez Windows, l’émulateur Azure Cosmos DB est épinglé à la fois à la barre des tâches Windows et au menu Démarrer. S’il ne démarre pas à partir des icônes épinglées, essayez de l’ouvrir en double-cliquant sur le fichier **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe**.

1. Attendez que l’émulateur ouvre automatiquement votre navigateur par défaut et accédez à la page de destination **https://localhost:8081/_explorer/index.html**.

1. Dans le volet **Démarrage rapide**, notez la **Chaîne de connexion principale**. Vous utiliserez cette chaîne de connexion ultérieurement.

> &#128221; Parfois, la page de destination ne se charge pas correctement, même si l’émulateur est en cours d’exécution. Si cela se produit, vous pouvez utiliser la chaîne de connexion connue pour vous connecter à l’émulateur. La chaîne de connexion connue est la suivante : `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## Importer la bibliothèque @azure/cosmos

La bibliothèque **@azure/cosmos** est disponible sur **npm** pour faciliter l’installation dans vos projets JavaScript.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/02-sdk-offline**.

1. Ouvrez le menu contextuel du dossier **javascript/02-sdk-offline**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > 💡 Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **javascript/02-sdk-offline**.

1. Initialisez un nouveau projet Node.js :

    ```bash
    npm init -y
    ```

1. Installez le package [@azure/cosmos][npmjs.com/package/@azure/cosmos] à l’aide de la commande suivante :

    ```bash
    npm install @azure/cosmos
    ```

## Se connecter à l’émulateur à partir du kit de développement logiciel (SDK) JavaScript

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **javascript/02-sdk-offline**.

1. Ouvrez le fichier JavaScript vierge nommé **script.js**.

1. Ajoutez le code suivant pour vous connecter à l’émulateur, créer une base de données et afficher son ID :

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    
    // Connection string for the Azure Cosmos DB Emulator
    const endpoint = "https://127.0.0.1:8081/";
    const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    // Initialize the Cosmos client
    const client = new CosmosClient({ endpoint, key });
    
    async function main() {
        // Create a database
        const databaseName = "cosmicworks";
        const { database } = await client.databases.createIfNotExists({ id: databaseName });
    
        // Print the database ID
        console.log(`New Database: Id: ${database.id}`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Enregistrez** le fichier **script.js**.

## Exécuter le script

1. Utilisez la même fenêtre de terminal dans **Visual Studio Code** que celle que vous avez utilisée pour installer la bibliothèque pour ce labo. Si vous l’avez fermée, ouvrez le menu contextuel du dossier **javascript/02-sdk-offline**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Exécutez le script avec la commande `node` :

    ```bash
    node script.js
    ```

1. Le script crée une base de données nommée `cosmicworks` dans l’émulateur. Vous devez obtenir une sortie similaire à la suivante :

    ```text
    New Database: Id: cosmicworks
    ```

## Créer et afficher un nouveau conteneur

Vous pouvez étendre le script pour créer un conteneur dans la base de données.

### Code mis à jour

1. Modifiez le fichier `script.js` pour **remplacer** la ligne suivante en bas du fichier (`main().catch((error) => console.error(error));`) afin de créer un conteneur :

```javascript
async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

Le fichier `script.js` doit maintenant se présenter comme ceci :

```javascript
const { CosmosClient } = require("@azure/cosmos");
process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

// Connection string for the Azure Cosmos DB Emulator
const endpoint = "https://127.0.0.1:8081/";
const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";

// Initialize the Cosmos client
const client = new CosmosClient({ endpoint, key });

async function main() {
    // Create a database
    const databaseName = "cosmicworks";
    const { database } = await client.databases.createIfNotExists({ id: databaseName });

    // Print the database ID
    console.log(`New Database: Id: ${database.id}`);
}

async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

### Exécuter le script mis à jour

1. Exécutez le script mis à jour à l’aide de la commande suivante :

    ```bash
    node script.js
    ```

1. Le script crée un conteneur nommé `products` dans l’émulateur. Vous devez obtenir une sortie similaire à la suivante :

    ```text
    New Database: Id: cosmicworks
    New Container: Id: products
    ```

### Vérifier les résultats

1. Basculez vers le navigateur dans lequel l’Explorateur de données de l’émulateur est ouvert.

1. Actualisez l’**API NoSQL** pour observer la nouvelle base de données **cosmicworks** et le conteneur **produits**.

## Arrêter l’émulateur Azure Cosmos DB

Il est important d’arrêter l’émulateur lorsque vous avez terminé de l’utiliser pour libérer des ressources système. Suivez les étapes ci-dessous en fonction de votre système d’exploitation :

### Sur macOS ou Linux :

Si vous avez démarré l’émulateur dans une fenêtre de terminal, procédez comme suit :

1. Recherchez la fenêtre de terminal où l’émulateur est en cours d’exécution.

1. Appuyez sur `Ctrl + C` pour terminer le processus de l’émulateur.

Sinon, si vous devez arrêter le processus de l’émulateur manuellement :

1. Ouvrez une nouvelle fenêtre de terminal.

1. Utilisez la commande suivante pour rechercher le processeur de l’émulateur :

    ```bash
    ps aux | grep CosmosDB.Emulator
    ```

Identifiez le **PID** (ID de processus) du processus de l’émulateur dans la sortie. Utilisez la commande kill pour terminer le processus de l’émulateur :

```bash
kill <PID>
```

### Sur Windows :

1. Recherchez l’icône de l’émulateur Azure Cosmos DB dans la barre d’état système Windows (près de l’horloge dans la barre des tâches).

1. Cliquez avec le bouton droit sur l’icône de l’émulateur pour ouvrir le menu contextuel.

1. Sélectionnez **Quitter** pour arrêter l’émulateur.

> 💡 Quitter l’ensemble des instances de l’émulateur peut prendre une minute.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
