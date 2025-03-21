---
lab:
  title: "01 - Se connecter à Azure\_Cosmos\_DB\_for\_NoSQL avec le kit de développement logiciel (SDK)"
  module: Use the Azure Cosmos DB for NoSQL SDK
---

# Se connecter à Azure Cosmos DB for NoSQL avec le kit SDK

Le kit de développement logiciel (SDK) Azure pour Python est une suite de bibliothèques clientes offrant une interface de développement cohérente pour interagir avec de nombreux services Azure. Les bibliothèques clientes sont des packages que vous utiliseriez pour consommer ces ressources et interagir avec elles.

Dans ce labo, vous allez vous connecter à un compte Azure Cosmos DB for NoSQL à l’aide du kit de développement logiciel (SDK) Azure pour Python.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des copilotes avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#install-the-azure-cosmos-library). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Installer la bibliothèque azure-cosmos

La bibliothèque **azure-cosmos** est disponible sur **PyPI** pour faciliter l’installation dans vos projets Python.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/01-sdk-connect**.

1. Ouvrez le menu contextuel du dossier **python/01-sdk-connect**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **python/01-sdk-connect**.

1. Créez et activez un environnement virtuel pour gérer les dépendances :

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Installez le package [azure-cosmos][pypi.org/project/azure-cosmos] à l’aide de la commande suivante :

   ```bash
   pip install azure-cosmos
   ```

1. Installez la bibliothèque [azure-identity][pypi.org/project/azure-identity], qui nous permet d’utiliser l’authentification Azure pour se connecter à l’espace de travail Azure Cosmos DB, à l’aide de la commande suivante :

   ```bash
   pip install azure-identity
   ```

1. Fermez le terminal intégré.

## Utiliser la bibliothèque azure-cosmos

Une fois la bibliothèque Azure Cosmos DB du kit de développement logiciel (SDK) Azure pour Python importée, vous pouvez utiliser immédiatement ses classes pour vous connecter à un compte Azure Cosmos DB for NoSQL. La classe **CosmosClient** est la classe principale utilisée pour établir la connexion initiale à un compte Azure Cosmos DB for NoSQL.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **python/01-sdk-connect**.

1. Ouvrez le fichier Python vierge nommé **script.py**.

1. Ajoutez l’instruction `import` suivante pour importer les classes **CosmosClient** et **DefaultAzureCredential** :

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential
   ```

1. Ajoutez les variables nommées **endpoint** et **credential**, puis définissez la valeur **endpoint** sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment. La variable **credential** doit être définie sur une nouvelle instance de la classe **DefaultAzureCredential** :

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Par exemple, si votre point de terminaison est **https://dp420.documents.azure.com:443/**, l’instruction est **endpoint = "https://dp420.documents.azure.com:443/"**.

1. Ajoutez une nouvelle variable nommée **client** et initialisez-la en tant que nouvelle instance de la classe **CosmosClient** à l’aide des variables **endpoint** et **credential** :

   ```python
   client = CosmosClient(endpoint, credential=credential)
   ```

1. Ajoutez une fonction nommée **main** pour lire et afficher les propriétés du compte :

   ```python
   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
   ```

1. Votre fichier **script.py** doit maintenant ressembler à ceci :

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   client = CosmosClient(endpoint, credential=credential)

   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
    ```

1. **Enregistrez** le fichier **script.py**.

## Tester le script

Le code Python pour vous connecter au compte Azure Cosmos DB for NoSQL étant désormais terminé, vous pouvez tester le script. Ce script affiche le niveau de cohérence par défaut et le nom de la première région accessible en écriture. La valeur d’emplacement que vous avez spécifiée lorsque vous avez créé le compte doit s’afficher à la suite de l’exécution de ce script.

1. Dans **Visual Studio Code**, ouvrez le menu contextuel du dossier **python/01-sdk-connect**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Avant d’exécuter le script, vous devez vous connecter à Azure à l’aide de la commande `az login`. Dans la fenêtre de terminal, exécutez :

   ```bash
   az login
   ```

1. Exécutez le script avec la commande `python` :

   ```bash
   python script.py
   ```

1. Le script génère désormais le niveau de cohérence par défaut et la première région accessible en écriture. Par exemple, si le niveau de cohérence par défaut du compte est **Session** et que la première région accessible en écriture est **USA Est**, le script affiche :

   ```text
   Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
   Primary Region: East US
   ```

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
