---
title: "06- Paginer les résultats des requêtes multi-produits avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "06- Paginer les résultats des requêtes multi-produits avec le kit de développement logiciel (SDK) Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Author complex queries with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 9
parent: Python SDK labs
---

# Paginer les résultats des requêtes multi-produits avec le SDK Azure Cosmos DB for NoSQL

Les requêtes Azure Cosmos DB ont généralement plusieurs pages de résultats. La pagination est effectuée automatiquement côté serveur quand Azure Cosmos DB ne peut pas retourner tous les résultats de requête en une seule exécution. Dans de nombreuses applications, vous souhaiterez écrire du code à l’aide du Kit de développement logiciel (SDK) pour traiter vos résultats de requête de manière performante.

Dans ce labo, vous allez créer un itérateur de flux qui peut être utilisé dans une boucle pour itérer sur l’ensemble de votre jeu de résultats.

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

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/06-sdk-pagination**.

1. Ouvrez le menu contextuel du dossier **python/06-sdk-pagination**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **python/06-sdk-pagination**.

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

## Paginer via de petits jeux de résultats d’une requête SQL à l’aide du Kit de développement logiciel (SDK)

Lors du traitement des résultats de requête, vous devez vous assurer que votre code progresse dans toutes les pages de résultats et de vérifications pour voir si d’autres pages sont restantes avant d’effectuer des requêtes ultérieures.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/06-sdk-pagination**.

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

1. Créez une variable nommée **sql** de type *string* avec la valeur **SELECT * FROM products WHERE products.price > 500** :

   ```python
   sql = "SELECT * FROM products WHERE products.price > 500"
   ```

1. Appelez la méthode [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items) avec la variable `sql` en tant que paramètre au constructeur. Définissez `max_item_count` sur `50` pour limiter le nombre d’éléments retournés dans chaque page.

   ```python
   iterator = container.query_items(
       query=sql,
       max_item_count=50  # Set maximum items per page
   )
   ```

1. Créez une boucle **for** asynchrone qui appelle la méthode [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page) de manière asynchrone sur l’objet d’itérateur. Cette méthode retourne une page de résultats chaque fois qu’elle est appelée.

   ```python
   async for page in iterator.by_page():
   ```

1. Dans la boucle **for** asynchrone, itérez sur les résultats paginés de manière asynchrone et affichez l’`id`, le `name` et le `price` de chaque élément.

   ```python
   async for product in page:
       print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")
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
           # Get database and container clients
           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products WHERE products.price > 500"
        
           iterator = container.query_items(
               query=sql,
               max_item_count=50  # Set maximum items per page
           )
        
           async for page in iterator.by_page():
               async for product in page:
                   print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")

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

1. Le script génère désormais des pages de 50 éléments à la fois.

    > &#128161; La requête correspondra à des centaines d’éléments dans le conteneur de produits.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
