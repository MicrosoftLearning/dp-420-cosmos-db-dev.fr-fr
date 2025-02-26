---
title: "05 - Exécuter une requête avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "05 - Exécuter une requête avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: Python SDK labs
---

# Exécuter une requête avec le SDK Azure Cosmos DB for NoSQL

La dernière version du kit de développement logiciel (SDK) Python pour Azure Cosmos DB for NoSQL simplifie l’interrogation d’un conteneur et l’itération sur les jeux de résultats à l’aide des fonctionnalités modernes de Python.

La bibliothèque `azure-cosmos` dispose de fonctionnalités intégrées pour rendre l’interrogation d’Azure Cosmos DB efficace et simple.

Dans ce labo, vous allez utiliser un itérateur pour traiter un jeu de résultats volumineux retourné par Azure Cosmos DB for NoSQL. Vous allez utiliser le kit de développement logiciel (SDK) Python pour l’interrogation et l’itération sur les résultats.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des Copilots avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des copilotes avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#create-azure-cosmos-db-database-and-container-with-sample-data). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Créer une base de données et un conteneur Azure Cosmos DB avec des exemples de données

Si vous avez déjà créé une base de données Azure Cosmos DB nommée **cosmoworks-full** et un conteneur dans celle-ci nommé **produits**, qui est préchargé avec des exemples de données, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#install-the-azure-cosmos-library). Sinon, suivez les étapes ci-dessous pour créer un exemple de base de données et de conteneur.

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

## Installer la bibliothèque azure-cosmos

La bibliothèque **azure-cosmos** est disponible sur **PyPI** pour faciliter l’installation dans vos projets Python.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/05-sdk-queries**.

1. Ouvrez le menu contextuel du dossier **python/05-sdk-queries**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **python/05-sdk-queries**.

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

## Interroger les résultats d’une requête SQL à l’aide du kit de développement logiciel

À l’aide des informations d’identification du compte nouvellement créé, vous allez vous connecter aux classes du kit de développement logiciel (SDK) et vous connecter à la base de données et au conteneur que vous avez provisionnés dans une étape précédente, puis effectuer une itération sur les résultats d’une requête SQL à l’aide du kit de développement logiciel (SDK).

Vous allez à présent utiliser un itérateur pour créer une boucle simple à comprendre sur les résultats paginés d’Azure Cosmos DB. En arrière-plan, le kit de développement logiciel (SDK) gère l’itérateur de flux et vérifie que les demandes suivantes sont appelées correctement.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/05-sdk-queries**.

1. Ouvrez le fichier Python vierge nommé **script.py**.

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

1. Ajoutez le code suivant pour vous connecter à la base de données et au conteneur que vous avez créés précédemment :

   ```python
   database = client.get_database_client("cosmicworks-full")
   container = database.get_container_client("products")
   ```

1. Créez une variable de chaîne de requête nommée `sql` avec la valeur `SELECT * FROM products p`.

   ```python
   sql = "SELECT * FROM products p"
   ```

1. Appelez la méthode [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) avec la variable `sql` en tant que paramètre au constructeur.

   ```python
   result_iterator = container.query_items(
       query=sql
   )
   ```

1. La méthode **query_items** a retourné un itérateur asynchrone que nous stockons dans une variable nommée `result_iterator`. Cela signifie que chaque objet de l’itérateur est un objet attendu et qu’il ne contient pas encore le résultat de la requête. Ajoutez le code ci-dessous pour créer une boucle **for** asynchrone afin d’attendre chaque résultat de la requête lorsque vous effectuez une itération sur l’itérateur asynchrone et affichez l’`id`, le `name` et le `price` de chaque élément.

   ```python
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Sous la méthode `main`, ajoutez le code suivant pour exécuter la méthode `main` à l’aide de la bibliothèque `asyncio` :

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. Votre fichier **script.py** doit maintenant ressembler à ceci :

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           result_iterator = container.query_items(
               query=sql
           )
            
           # Perform the query asynchronously
           async for item in result_iterator:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

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

1. Le script génère à présent chaque produit dans le conteneur.

## Effectuer une requête dans une partition logique

Dans la section précédente, vous avez interrogé tous les éléments du conteneur. Par défaut, le client **CosmosClient** asynchrone effectue des requêtes entre partitions. C’est la raison pour laquelle la requête que vous avez exécutée (`"SELECT * FROM products p"`) a entraîné l’analyse de toutes les partitions du conteneur par le moteur de requête. Comme meilleure pratique, vous devez toujours interroger dans une partition logique pour éviter les requêtes entre partitions. En définitive, cela vous permet d’économiser de l’argent et d’améliorer les performances.

Dans cette section, vous allez effectuer une requête dans une partition logique en incluant la clé de partition dans la requête.

1. Revenez à l’onglet de l’éditeur pour le fichier de code **script.py**.

1. Supprimez les lignes de code suivantes :

   ```python
   result_iterator = container.query_items(
       query=sql
   )
    
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Modifiez le script pour créer une variable **partition_key** afin de stocker la valeur d’ID de catégorie des maillots. Ajoutez la variable **partition_key** en tant que paramètre à la méthode **query_items**. Cela garantit que la requête est exécutée dans la partition logique pour la catégorie des maillots.

   ```python
   partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

   result_iterator = container.query_items(
       query=sql,
       partition_key=partition_key
   )
   ```

1. Dans la section précédente, vous avez exécuté une boucle « for » asynchrone directement sur l’itérateur asynchrone (`async for item in result_iterator:`). Cette fois-ci, vous allez créer de manière asynchrone une liste complète des résultats de la requête réels. Ce code effectue la même action que l’exemple de boucle « for » que vous avez utilisé précédemment. Ajoutez les lignes de code suivantes pour créer une liste de résultats et afficher les résultats :

   ```python
   item_list = [item async for item in result_iterator]

   for item in item_list:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Votre fichier **script.py** doit maintenant ressembler à ceci :

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

           result_iterator = container.query_items(
               query=sql,
               partition_key=partition_key
           )
    
           # Perform the query asynchronously
           item_list = [item async for item in result_iterator]
    
           for item in item_list:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Enregistrez** le fichier **script.py**.

1. Exécutez le script pour créer la base de données et le conteneur :

   ```bash
   python script.py
   ```

1. Le script génère désormais chaque produit dans la catégorie des maillots, exécutant ainsi efficacement une requête dans la partition.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
