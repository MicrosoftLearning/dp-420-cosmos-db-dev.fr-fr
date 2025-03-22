---
lab:
  title: "04 - Traiter par lots plusieurs opérations de point avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# Traiter par lots plusieurs opérations de point avec le kit SDK Azure Cosmos DB for NoSQL

Le kit de développement logiciel (SDK) Python `azure-cosmos` fournit la méthode `execute_item_batch` permettant d’exécuter plusieurs opérations de point dans une même étape logique. Cela permet aux développeurs de regrouper efficacement plusieurs opérations et de déterminer si elles se sont terminées avec succès côté serveur.

Dans ce labo, vous allez utiliser le kit de développement logiciel (SDK) Python pour effectuer des opérations de traitement par lots à deux éléments qui illustrent les lots transactionnels réussis et errants.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des copilotes avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#install-the-azure-cosmos-library). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Installer la bibliothèque azure-cosmos

La bibliothèque **azure-cosmos** est disponible sur **PyPI** pour faciliter l’installation dans vos projets Python.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/04-sdk-batch**.

1. Ouvrez le menu contextuel du dossier **python/04-sdk-batch**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **python/04-sdk-batch**.

1. Créez et activez un environnement virtuel pour gérer les dépendances :

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Installez le package [azure-cosmos][pypi.org/project/azure-cosmos] à l’aide de la commande suivante :

   ```bash
   pip install azure-cosmos
   ```

1. Étant donné que nous utilisons la version asynchrone du kit de développement logiciel (SDK), nous devons également installer la bibliothèque `asyncio` :

   ```bash
   pip install asyncio
   ```

1. La version asynchrone du kit de développement logiciel (SDK) a également besoin de la bibliothèque `aiohttp`. Installez-le en utilisant la commande suivante :

   ```bash
   pip install aiohttp
   ```

1. Installez la bibliothèque [azure-identity][pypi.org/project/azure-identity], qui nous permet d’utiliser l’authentification Azure pour se connecter à l’espace de travail Azure Cosmos DB, à l’aide de la commande suivante :

   ```bash
   pip install azure-identity
   ```

## Utiliser la bibliothèque azure-cosmos

À l’aide des informations d’identification du compte récemment créé, vous allez vous connecter aux classes du kit SDK et créer une base de données et une instance de conteneur. Ensuite, vous utiliserez l’Explorateur de données pour valider l’existence des instances dans le Portail Azure.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/03-sdk-crud**.

1. Ouvrez le fichier Python vierge nommé **script.py**.

1. Ajoutez l’instruction `import` suivante pour importer la classe **PartitionKey** :

   ```python
   from azure.cosmos import PartitionKey
   ```

1. Ajoutez les instructions `import` suivantes pour importer la classe **CosmosClient** asynchrone, la classe **DefaultAzureCredential** et la bibliothèque **asyncio** :

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio
   ```

1. Ajoutez les variables nommées **endpoint** et **credential**, puis définissez la valeur **endpoint** sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment. La variable **credential** doit être définie sur une nouvelle instance de la classe **DefaultAzureCredential** :

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Par exemple, si votre point de terminaison est **https://dp420.documents.azure.com:443/**, l’instruction est **endpoint = "https://dp420.documents.azure.com:443/"**.

1. Toutes les interactions avec Cosmos DB commencent par une instance de `CosmosClient`. Pour utiliser le client asynchrone, nous devons utiliser les mots clés async/await, qui ne peuvent être utilisés que dans les méthodes asynchrones. Créez une méthode asynchrone nommée **main** et ajoutez le code suivant pour créer une instance de la classe **CosmosClient** asynchrone à l’aide des variables **endpoint** et **credential** :

   ```python
   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
   ```

    > &#128161; Étant donné que nous utilisons le client **CosmosClient** asynchrone, vous devez également le préparer et le fermer pour pouvoir l’utiliser correctement. Nous vous recommandons d’utiliser les mots clés `async with` comme illustré dans le code ci-dessus pour démarrer vos clients. Ces mots clés créent un gestionnaire de contexte qui prépare, initialise et nettoie automatiquement le client, afin que vous n’ayez pas à le faire.

1. Ajoutez le code suivant pour créer une base de données et un conteneur s’ils n’existent pas déjà :

   ```python
   # Create database
   database = await client.create_database_if_not_exists(id="cosmicworks")
    
   # Create container
   container = await database.create_container_if_not_exists(
       id="products",
       partition_key=PartitionKey(path="/categoryId"),
       offer_throughput=400
   )
   ```

1. Sous la méthode `main`, ajoutez le code suivant pour exécuter la méthode `main` à l’aide de la bibliothèque `asyncio` :

   ```python
   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. Votre fichier **script.py** doit maintenant ressembler à ceci :

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Enregistrez** le fichier **script.py**.

1. Avant d’exécuter le script, vous devez vous connecter à Azure à l’aide de la commande `az login`. Dans la fenêtre de terminal, exécutez :

   ```bash
   az login
   ```

1. Exécutez le script pour créer la base de données et le conteneur :

   ```bash
   python script.py
   ```

1. Basculez vers la fenêtre de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

## Création d’un lot transactionnel

Tout d’abord, nous allons créer un lot transactionnel simple qui crée deux produits fictifs. Ce lot insère une selle usée et un guidon rouillé dans le conteneur avec le même identificateur de catégorie « accessoires utilisés ». Les deux éléments ont la même clé de partition logique, ce qui garantit que nous aurons une opération de traitement par lots réussie.

1. Revenez à **Visual Studio Code**. S’il n’est pas encore ouvert, ouvrez le fichier de code **script.py** dans le dossier **python/04-sdk-batch**.

1. Créez deux dictionnaires représentant des produits : une **selle usée** et un **guidon rouillé**. Les deux éléments partagent la même valeur de clé de partition **"9603ca6c-9e28-4a02-9194-51cdb7fea816"**.

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   ```

1. Définissez la valeur de clé de partition.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Créez un lot contenant les deux éléments.

   ```python
   batch = [saddle, handlebar]
   ```

1. Exécutez le lot à l’aide de la méthode `execute_item_batch` de l’objet `container` et affichez la réponse pour chaque élément du lot.

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           saddle = ("create", (
               {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           handlebar = ("create", (
               {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [saddle, handlebar]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Enregistrez** et réexécutez le script :

   ```bash
   python script.py
   ```

1. La sortie doit indiquer un code d’état réussi pour chaque opération.

## Création d’un lot transactionnel errant

À présent, nous allons créer un lot transactionnel qui génère une erreur délibérée. Ce lot tente d’insérer deux éléments qui ont des clés de partition logique différentes. Nous allons créer une lampe stroboscopique qui clignote dans la catégorie « accessoires utilisés » et un nouveau casque dans la catégorie « accessoires qualités ». Par définition, il devrait s’agir d'une mauvaise requête et une erreur devrait être renvoyée lors de l’exécution de cette transaction.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.py**.

1. Supprimez les lignes de code suivantes :

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

   batch = [saddle, handlebar]
   ```

1. Modifiez le script pour créer une **lampe stroboscopique** et un **nouveau casque** avec différentes valeurs de clé de partition.

   ```python
   light = ("create", (
       {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   helmet = ("create", (
       {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
   ))
   ```

1. Définissez la valeur de clé de partition pour le lot.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Créez un lot contenant les deux éléments.

   ```python
   batch = [light, helmet]
   ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

   ```python
   from azure.cosmos import exceptions, PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

           light = ("create", (
               {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           helmet = ("create", (
               {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [light, helmet]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **Enregistrez** et réexécutez le script :

   ```bash
   python script.py
   ```

1. Observez la sortie du terminal. Le code d’état sur le deuxième élément (le « Nouveau casque ») doit être **400** pour **Demande incorrecte**. Cela s’est produit parce que tous les éléments de la transaction ne partage pas la même valeur de clé de partition que le lot transactionnel.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
