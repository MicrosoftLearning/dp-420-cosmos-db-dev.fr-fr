---
lab:
  title: "03 - Créer et mettre à jour des documents avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Implement Azure Cosmos DB for NoSQL point operations
---

# Créer et mettre à jour des documents avec le Kit de développement logiciel (SDK) Azure Cosmos DB for NoSQL

La bibliothèque `azure-cosmos` inclut des méthodes pour créer, récupérer, mettre à jour et supprimer des éléments (CRUD) dans un conteneur Azure Cosmos DB for NoSQL. Ensemble, ces méthodes permettent d’effectuer certaines des opérations « CRUD » les plus courantes sur divers éléments dans des conteneurs d’API NoSQL.

Dans ce labo, vous allez utiliser le kit de développement logiciel (SDK) Python pour effectuer des opérations CRUD courantes sur un élément dans un conteneur Azure Cosmos DB for NoSQL.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des copilotes avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#install-the-azure-cosmos-library). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Installer la bibliothèque azure-cosmos

La bibliothèque **azure-cosmos** est disponible sur **PyPI** pour faciliter l’installation dans vos projets Python.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/03-sdk-crud**.

1. Ouvrez le menu contextuel du dossier **python/03-sdk-crud**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **python/03-sdk-crud**.

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
       asyncio.run(query_items_async())
   ```

1. Votre fichier **script.py** doit maintenant ressembler à ceci :

   ```python
   from azure.cosmos import PartitionKey
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

## Effectuer des opérations de création et de lecture de point sur des éléments avec le kit de développement logiciel

Vous allez maintenant utiliser l’ensemble de méthodes dans la classe **ContainerProxy** pour effectuer des opérations courantes sur des éléments dans un conteneur d’API NoSQL.

1. Revenez à **Visual Studio Code**. S’il n’est pas encore ouvert, ouvrez le fichier de code **script.py** dans le dossier **python/03-sdk-crud**.

1. Créez un élément de produit et attribuez-le à une variable nommée **selle** avec les propriétés suivantes :

    | Propriété | Valeur |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **p**rice | *45.99d* |
    | **balises** | *{ tan, new, crisp }* |

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
   ```

1. Appelez la méthode [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item) de la variable **container** en transmettant la variable **selle** en tant que paramètre de méthode :

   ```python
   await container.create_item(body=saddle)
   ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :
  
   ```python
   from azure.cosmos import PartitionKey
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
        
           saddle = {
               "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
               "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
               "name": "Road Saddle",
               "price": 45.99,
               "tags": ["tan", "new", "crisp"]
           }
            
           await container.create_item(body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Enregistrez** et réexécutez le script :

   ```bash
   python script.py
   ```

1. Observez le nouvel élément dans l’**Explorateur de données**.

1. Revenez à **Visual Studio Code**.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.py**.

1. Supprimez les lignes de code suivantes :

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
    
   await container.create_item(body=saddle)
   ```

1. Créez une variable de chaîne nommée **item_id** avec la valeur **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** :

   ```python
   item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
   ```

1. Créez une variable de chaîne nommée **partition_key** avec la valeur **9603ca6c-9e28-4a02-9194-51cdb7fea816** :

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Appelez la méthode [`read_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item) de la variable **container** en transmettant les variables **item_id** et **partitionKey** en tant que paramètres de méthode :

   ```python
   # Read item    
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
   ```

    > &#128161; La méthode `read_item` vous permet d’effectuer une opération de lecture de point sur un élément du conteneur. La méthode nécessite que les paramètres `item_id` et `partition_key` identifient l’élément à lire. Contrairement à l’exécution d’une requête à l’aide du langage de requête SQL de Cosmos DB pour rechercher un élément unique, la méthode `read_item` est plus efficace et économique pour récupérer un seul élément. Les lectures de points peuvent lire les données directement et n’ont pas besoin du moteur d’interrogation pour traiter la demande.

1. Affichez l’objet selle à l’aide d’une chaîne de sortie mise en forme :

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

   ```python
   from azure.cosmos import PartitionKey
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
       
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Enregistrez** et réexécutez le script :

   ```bash
   python script.py
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

1. Revenez à **Visual Studio Code**. Revenez à l’onglet de l’éditeur pour le fichier de code **script.py**.

1. Supprimez la ligne de code suivante :

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Modifiez la variable **saddle** en définissant la valeur de la propriété price sur **32.55** :

   ```python
   saddle["price"] = 32.55
   ```

1. Modifiez à nouveau la variable **saddle** en définissant la valeur de la propriété **name** sur **Road LL Saddle** :

   ```python
   saddle["name"] = "Road LL Saddle"
   ```

1. Appelez la méthode [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item) de la variable **container** en transmettant les variables **item_id** et **selle** en tant que paramètres de méthode :

   ```python
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Une fois que vous avez terminé, votre fichier de code devrait maintenant inclure :

   ```python
   from azure.cosmos import PartitionKey
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
        
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           saddle["price"] = 32.55
           saddle["name"] = "Road LL Saddle"
    
           await container.replace_item(item=item_id, body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **Enregistrez** et réexécutez le script :

   ```bash
   python script.py
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

1. Revenez à **Visual Studio Code**. Revenez à l’onglet de l’éditeur pour le fichier de code **script.py**.

1. Supprimez les lignes de code suivantes :

   ```python
   # Read item
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
    
   saddle["price"] = 32.55
   saddle["name"] = "Road LL Saddle"
    
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Appelez la méthode [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item) de la variable **container** en transmettant les variables **item_id** et **partitionKey** en tant que paramètres de méthode :

   ```python
   # Delete the item
   await container.delete_item(item=item_id, partition_key=partition_key)
   ```

1. Enregistrez et réexécutez le script :

   ```bash
   python script.py
   ```

1. Fermez le terminal intégré.

1. Revenez à la fenêtre ou à l’onglet de votre navigateur web.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis développez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**.

1. Sélectionnez le nœud **Items**. Notez que la liste des éléments est désormais vide.

1. Fermez la fenêtre ou l’onglet de votre navigateur web.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
