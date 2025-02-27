---
title: "07.2 - Générer des incorporations vectorielles avec Azure\_OpenAI et les stocker dans Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "07.2 - Générer des incorporations vectorielles avec Azure\_OpenAI et les stocker dans Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 11
parent: Python SDK labs
---

# Générer des incorporations vectorielles avec Azure OpenAI et les stocker dans Azure Cosmos DB for NoSQL

Azure OpenAI permet d’accéder à des modèles de langage avancés d’OpenAI, notamment les modèles `text-embedding-ada-002`, `text-embedding-3-small` et `text-embedding-3-large`. Grâce à l’un de ces modèles, vous pouvez générer des représentations vectorielles de données textuelles, qui peuvent être stockées dans un magasin vectoriel comme Azure Cosmos DB for NoSQL. Les recherches de similarités sont ainsi plus efficaces et précises, améliorant considérablement la capacité de Copilot à récupérer des informations pertinentes et à fournir des interactions contextuellement riches.

Dans ce labo, vous allez créer un Azure OpenAI Service et déployer un modèle d’incorporation. Vous allez ensuite utiliser du code Python pour créer des clients Azure OpenAI et Cosmos DB à l’aide de leurs kits de développement logiciel (SDK) Python respectifs pour générer des représentations vectorielles des descriptions de produits et les écrire dans votre base de données.

> &#128721; L’exercice précédent de ce module est un prérequis pour ce labo. Si vous avez toujours besoin d’effectuer cet exercice, veuillez le terminer avant de continuer, car il fournit l’infrastructure requise pour ce labo.

## Créer un Azure OpenAI Service

Azure OpenAI fournit un accès à l’API REST aux modèles de langage puissants d’OpenAI. Ces modèles peuvent être facilement adaptés à des tâches spécifiques, comme la génération de contenu, le résumé, la compréhension d’images, la recherche sémantique, le langage naturel et la traduction de code, entre autres.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

2. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

3. Sélectionnez **+ Créer une ressource**, recherchez *Azure OpenAI*, puis créez une ressource **Azure OpenAI** avec les paramètres suivants, en laissant tous les paramètres restants à leurs valeurs par défaut :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Région** | *Choisissez une région disponible qui prend en charge le modèle `text-embedding-3-small`* dans la [liste des régions compatibles](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings). |
    | **Nom** | *Entrez un nom globalement unique* |
    | **Niveau tarifaire** | *Choisissez Standard 0* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions qui vous empêchent de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

4. Attendez que la tâche de déploiement se termine avant de passer à la tâche suivante.

## Déployer un modèle d’incorporations

Pour utiliser Azure OpenAI afin de générer des incorporations, vous devez d’abord déployer une instance du modèle d’incorporation souhaité dans votre service.

1. Accédez à votre Azure OpenAI Service nouvellement créé dans le portail Azure (``portal.azure.com``).

2. Sur la page **Vue d’ensemble** d’Azure OpenAI Service, lancez **Azure AI Foundry** en sélectionnant le lien **Accéder au portail Azure AI Foundry** dans la barre d’outils.

3. Dans Azure AI Foundry, sélectionnez **Déploiements** dans le menu de gauche.

4. Sur la page **Déploiements de modèles**, sélectionnez **Déployer un modèle**, puis **Déployer le modèle de base** dans la liste déroulante.

5. Dans la liste des modèles, sélectionnez `text-embedding-3-small`.

    > &#128161; Vous pouvez filtrer la liste pour n’afficher que les modèles *Incorporations* l’aide du filtre des tâches d’inférence.

    > &#128221; Si vous ne voyez pas le modèle `text-embedding-3-small`, vous avez peut-être sélectionné une région Azure qui ne prend actuellement pas en charge ce modèle. Dans ce cas, vous pouvez utiliser le modèle `text-embedding-ada-002` pour ce labo. Les deux modèles génèrent des vecteurs avec 1 536 dimensions. Par conséquent, aucune modification ne doit être apportée à la stratégie de vecteur de conteneur que vous avez définie sur le conteneur `Products` dans Azure Cosmos DB.

6. Sélectionnez **Déployer** pour déployer le modèle.

7. Sur la page **Déploiements de modèles** dans Azure AI Foundry, notez le **Nom** du modèle de déploiement `text-embedding-3-small`, car vous en aurez besoin plus loin dans cet exercice.

## Déployer un modèle de saisie semi-automatique de conversation

En plus du modèle d’incorporation, vous aurez besoin d’un modèle de saisie semi-automatique de conversation pour votre Copilot. Vous allez utiliser le grand modèle de langage `gpt-4o` d’OpenAI pour générer des réponses à partir de votre Copilot.

1. Toujours sur la page **Déploiements de modèles** dans Azure AI Foundry, sélectionnez à nouveau le bouton **Déployer un modèle** et choisissez **Déployer le modèle de base** dans la liste déroulante.

2. Sélectionnez le modèle de saisie semi-automatique de conversation **gpt-4o** dans la liste.

3. Sélectionnez **Déployer** pour déployer le modèle.

4. Sur la page **Déploiements de modèles** dans Azure AI Foundry, notez le **Nom** du modèle de déploiement `gpt-4o`, car vous en aurez besoin plus loin dans cet exercice.

## Attribuer le rôle RBAC Utilisateur OpenAI Cognitive Services

Pour permettre à votre identité d’utilisateur d’interagir avec Azure OpenAI Service, vous pouvez attribuer le rôle **Utilisateur OpenAI Cognitive Services** à votre compte. Azure OpenAI Service prend en charge le contrôle d’accès en fonction du rôle Azure (Azure RBAC), un système d’autorisation pour la gestion des accès individuels aux ressources Azure. Avec Azure RBAC, vous attribuez différents niveaux d’autorisations à différents membres de l’équipe en fonction de leurs besoins pour un projet donné.

> &#128221; Le contrôle d’accès en fonction du rôle (RBAC) Entra ID pour l’authentification auprès de services Azure tels qu’Azure OpenAI renforce la sécurité grâce à des contrôles d’accès précis adaptés aux rôles d’utilisateur, ce qui réduit efficacement les risques d’accès non autorisés. La rationalisation de la gestion des accès sécurisée à l’aide du RBAC Entra ID rend une solution plus efficace et évolutive pour tirer parti des services Azure.

1. Dans le portail Azure (``portal.azure.com``), accédez à votre ressource Azure OpenAI.

2. Sélectionnez **Contrôle d’accès (IAM)** dans le volet de navigation de gauche.

3. Sélectionnez **Ajouter**, puis **Ajouter une attribution de rôle**.

4. Dans l’onglet **Rôle**, sélectionnez le rôle **Utilisateur OpenAI Cognitive Services**, puis **Suivant**.

5. Dans l’onglet **Membres**, choisissez d’autoriser l’accès à un utilisateur, un groupe ou un principal de service, puis sélectionnez **Sélectionner des membres**.

6. Dans la boîte de dialogue **Sélectionner des membres**, recherchez votre nom ou votre adresse e-mail et sélectionnez votre compte.

7. Dans l’onglet **Passer en revue + affecter**, sélectionnez **Passer en revue + affecter** pour affecter le rôle.

## Créer un environnement virtuel Python

Les environnements virtuels dans Python sont essentiels pour maintenir un espace de développement propre et organisé, ce qui permet aux projets individuels d’avoir leur propre ensemble de dépendances, isolé des autres. Cela empêche les conflits entre différents projets et garantit la cohérence dans votre workflow de développement. Grâce aux environnements virtuels, vous pouvez gérer facilement les versions de package, éviter les conflits de dépendances et maintenir la fluidité d’exécution de vos projets. Une meilleure pratique consiste à maintenir votre environnement de codage stable et fiable, ce qui rend votre processus de développement plus efficace et moins sujet aux problèmes.

1. À l’aide de Visual Studio Code, ouvrez le dossier dans lequel vous avez cloné le référentiel de code du labo pour le module d’apprentissage **Générer des Copilots avec Azure Cosmos DB**.

2. Dans Visual Studio Code, ouvrez une nouvelle fenêtre de terminal et remplacez les répertoires par le dossier `python/07-build-copilot`.

3. Créez un environnement virtuel nommé `.venv` en exécutant la commande suivante au prompt de terminal :

    ```bash
    python -m venv .venv 
    ```

    La commande ci-dessus crée un dossier `.venv` sous le dossier `07-build-copilot`, qui fournit un environnement Python dédié pour les exercices de ce labo.

4. Activez l’environnement virtuel en sélectionnant la commande appropriée pour votre système d’exploitation et votre interpréteur de commandes d’après le tableau ci-dessous et en l’exécutant au prompt du terminal.

    | Plateforme | Shell | Commande pour activer l’environnement virtuel |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | poisson | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

5. Installez les bibliothèques définies dans `requirements.txt` :

    ```bash
    pip install -r requirements.txt
    ```

    Le fichier `requirements.txt` contient un ensemble de bibliothèques Python que vous utiliserez tout au long de ce labo.

    | Bibliothèque | Version | Description |
    | ------- | ------- | ----------- |
    | `azure-cosmos` | 4.9.0 | Kit de développement logiciel (SDK) Azure Cosmos DB pour Python - Bibliothèque de client |
    | `azure-identity` | 1.19.0 | Kit de développement logiciel (SDK) Azure pour Python |
    | `fastapi` | 0.115.5 | Infrastructure web pour la génération d’API avec Python |
    | `openai` | 1.55.2 | Fournit l’accès à l’API REST Azure OpenAI à partir d’applications Python. |
    | `pydantic` | 2.10.2 | Validation des données à l’aide d’indicateurs de type Python. |
    | `requests` | 2.32.3 | Envoyer des requêtes HTTP. |
    | `streamlit` | 1.40.2 | Transforme les scripts Python en applications web interactives. |
    | `uvicorn` | 0.32.1 | Implémentation de serveur web ASGI pour Python. |
    | `httpx` | 0.27.2 | Client HTTP de nouvelle génération pour Python. |

## Ajouter une fonction Python pour vectoriser du texte

Le kit de développement logiciel (SDK) Python pour Azure OpenAI permet d’accéder à des classes synchrones et asynchrones qui peuvent être utilisées pour créer des incorporations pour des données textuelles. Cette fonctionnalité peut être encapsulée dans une fonction dans votre code Python.

1. Dans le volet **Explorateur** de Visual Studio Code, accédez au dossier `python/07-build-copilot/api/app` et ouvrez le fichier `main.py` situé dans celui-ci.

    > &#128221; Ce fichier servira de point d’entrée à une API Python back-end que vous allez générer dans l’exercice suivant. Dans cet exercice, vous allez fournir quelques fonctions asynchrones qui peuvent être utilisées pour importer des données avec des incorporations dans Azure Cosmos DB qui seront exploitées par l’API.

2. Pour utiliser le kit de développement logiciel (SDK) Azure OpenAI asynchrone pour Python, importez la bibliothèque en ajoutant le code suivant en haut du fichier `main.py` :

   ```python
   from openai import AsyncAzureOpenAI
   ```

3. Vous accédez de manière asynchrone à Azure OpenAI et Cosmos DB à l’aide de l’authentification Azure et des rôles RBAC Entra ID que vous avez précédemment attribués à votre identité d’utilisateur. Ajoutez la ligne suivante sous l’instruction d’importation `openai` en haut du fichier pour importer les classes requises à partir de la bibliothèque `azure-identity` :

   ```python
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   ```

    > &#128221; Pour vous assurer de pouvoir interagir en toute sécurité avec les services Azure à partir de votre API, vous allez utiliser le kit de développement logiciel (SDK) Azure Identity pour Python. Cette approche vous permet d’éviter de devoir stocker ou interagir avec des clés à partir du code, au lieu de tirer parti des rôles RBAC que vous avez attribués à votre compte pour accéder à Azure Cosmos DB et Azure OpenAI dans les exercices précédents.

4. Créez des variables pour stocker la version et le point de terminaison de l’API Azure OpenAI, en remplaçant le jeton `<AZURE_OPENAI_ENDPOINT>` par la valeur de point de terminaison de votre Azure OpenAI Service. Créez également une variable pour le nom de votre déploiement de modèle d’incorporation. Insérez le code suivant sous les instructions `import` dans le fichier :

   ```python
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   ```

    Si votre nom de déploiement de modèle d’incorporation est différent, mettez à jour la valeur attribuée à la variable en conséquence.

    > &#128161; La version d’API `2024-10-21` était la dernière version GA au moment de la rédaction. Vous pouvez l’utiliser ou une nouvelle version, si une est disponible. La documentation sur les spécifications d’API contient un [tableau avec les dernières versions d’API](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs).

    > &#128221; `EMBEDDING_DEPLOYMENT_NAME` correspond à la valeur de **Nom** que vous avez notée après le déploiement du modèle `text-embedding-3-small` dans Azure AI Foundry. Si vous devez vous y référer, lancez Azure AI Foundry, accédez à la page **Déploiements** et recherchez le déploiement dont le **Nom du modèle** est `text-embedding-3-small`. Copiez ensuite la valeur de champ **Nom** de cet élément. Si vous avez déployé le modèle `text-embedding-ada-002`, utilisez le nom de ce déploiement.

5. Utilisez le kit de développement logiciel (SDK) Azure Identity pour la classe `DefaultAzureCredential` de Python afin de créer des informations d’identification asynchrones pour accéder à Azure OpenAI et Azure Cosmos DB à l’aide de l’authentification RBAC Microsoft Entra ID en insérant le code suivant sous les déclarations de variables :

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

6. Pour gérer la création d’incorporations, insérez ce qui suit, qui ajoute une fonction pour générer des incorporations à l’aide d’un client Azure OpenAI :

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

    La création du client Azure OpenAI n’a pas besoin de la valeur `api_key`, car elle récupère un jeton du porteur à l’aide de la classe `get_bearer_token_provider` du kit de développement logiciel (SDK) Azure Identity.

7. Le fichier `main.py` doit maintenant être semblable à ce qui suit :

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

8. Enregistrez le fichier `main.py`.

## Tester la fonction d’incorporation

Pour vous assurer que la fonction `generate_embeddings` dans le fichier `main.py` fonctionne correctement, vous allez ajouter quelques lignes de code en bas du fichier pour qu’elle puisse être exécutée directement. Ces lignes vous permettent d’exécuter la fonction `generate_embeddings` à partir de la ligne de commande, en transmettant le texte à incorporer.

1. Ajoutez un bloc **main guard** contenant un appel à `generate_embeddings` en bas du fichier `main.py` :

   ```python
   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

    > &#128221; Le bloc `if __name__ == "__main__":` est généralement appelé **main guard** ou **entry point** dans Python. Il garantit que certains codes ne sont exécutés que lorsque le script est exécuté directement, et non lorsqu’il est importé en tant que module dans un autre script. Cette pratique permet d’organiser le code et de le rendre plus réutilisable et modulaire.

2. Enregistrez le fichier `main.py`, qui doit maintenant se présenter ainsi :

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

3. Dans Visual Studio Code, ouvrez une nouvelle fenêtre de terminal intégré.

4. Avant d’exécuter l’API, qui envoie des demandes à Azure OpenAI, vous devez vous connecter à Azure à l’aide de la commande `az login`. Dans la fenêtre de terminal, exécutez :

   ```bash
   az login
   ```

5. Terminez le processus de connexion dans votre navigateur.

6. Au prompt de terminal, remplacez les répertoires par `python/07-build-copilot`.

7. Vérifiez que la fenêtre de terminal intégré est en cours d’exécution dans votre environnement virtuel Python en activant votre environnement virtuel à l’aide d’une commande du tableau ci-dessous et en sélectionnant la commande appropriée pour votre système d’exploitation et votre interpréteur de commandes.

    | Plateforme | Shell | Commande pour activer l’environnement virtuel |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | poisson | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

8. Au prompt de terminal, remplacez les répertoires par `api/app`, puis exécutez la commande suivante :

   ```python
   python main.py "Hello, world!"
   ```

9. Examinez la sortie dans la fenêtre de terminal. Vous devez voir un tableau de nombre à virgule flottante, qui est la représentation vectorielle de la chaîne « Hello, world! » chaîne. Elle doit ressembler à la sortie abrégée suivante :

   ```bash
   [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
   ```

## Générer une fonction pour écrire des données dans Azure Cosmos DB

À l’aide du kit de développement logiciel (SDK) Azure Cosmos DB pour Python, vous pouvez créer une fonction qui permet de faire un upsert de documents dans votre base de données. Une opération upsert met à jour un enregistrement si une correspondance est trouvée et insère un nouvel enregistrement si ce n’est pas le cas.

1. Revenez au fichier `main.py` ouvert dans Visual Studio Code et importez la classe `CosmosClient` asynchrone à partir du kit de développement logiciel (SDK) Azure Cosmos DB pour Python en insérant la ligne suivante juste en dessous des instructions `import` déjà présentes dans le fichier :

   ```python
   from azure.cosmos.aio import CosmosClient
   ```

2. Ajoutez une autre instruction d’importation pour référencer la  classe `Product` à partir du module *modèles* dans le dossier `api/app`. La classe `Product` définit la forme des produits dans le jeu de données Cosmic Works.

   ```python
   from models import Product
   ```

3. Créez un groupe de variables contenant des valeurs de configuration associées à Azure Cosmos DB et ajoutez-les au fichier `main.py` sous les variables Azure OpenAI que vous avez insérées précédemment. Veillez à remplacer le jeton `<AZURE_COSMOSDB_ENDPOINT>` par le point de terminaison de votre compte Azure Cosmos DB.

   ```python
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
   ```

4. Ajoutez une fonction nommée `upsert_product` pour l’upsert (mettre à jour ou insérer) de documents dans Cosmos DB, en insérant le code suivant sous la fonction `generate_embeddings` dans le fichier `main.py` :

   ```python
   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
   ```

5. Enregistrez le fichier `main.py`, qui doit maintenant se présenter ainsi :

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"

   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
    
   if __name__ == "__main__":
       import sys
       print(generate_embeddings(sys.argv[1]))
   ```

## Vectoriser des exemples de données

Vous êtes maintenant prêt à tester les fonctions `generate_embeddings` et `upsert_document` ensemble. Pour ce faire, vous allez remplacer le bloc de protection principale `if __name__ == "__main__"` avec du code qui télécharge un exemple de fichier de données contenant des informations de produits Cosmic Works depuis GitHub, puis vectorise le champ `description` de chaque produit et fait un upsert des documents dans le conteneur `Products` dans votre base de données Azure Cosmos DB.

> &#128221; Cette approche est utilisée pour illustrer les techniques de génération avec Azure OpenAI et le stockage d’incorporations dans Azure Cosmos DB. Dans un scénario du monde réel, cependant, une approche plus robuste, comme l’utilisation d’une fonction Azure déclenchée par le flux de modification Azure Cosmos DB, serait plus appropriée pour la gestion de l’ajout d’incorporations à des documents existants et nouveaux.

1. Dans le fichier `main.py`, remplacez le bloc de code `if __name__ == "__main__":` par ce qui suit :

   ```python
   if __name__ == "__main__":
       import asyncio
       from models import Product
       import requests
    
       async def main():
           product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
           products = [Product(**data) for data in requests.get(product_raw_data).json()]
    
           # Call the generate_embeddings function, passing in an argument from the command line.    
           for product in products:
               print(f"Generating embeddings for product: {product.name}", end="\r")
               product.embedding = await generate_embeddings(product.description)
               await upsert_product(product.model_dump())
    
           print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
           # Close open credential sessions
           await credential.close()
    
       asyncio.run(main())
   ```

2. Dans l’invite de terminal intégré ouverte dans Visual Studio Code, réexécutez le fichier `main.py` à l’aide de la commande :

   ```python
   python main.py
   ```

3. Attendez que l’exécution du code se termine, indiquée par un message précisant que tous les produits avec des descriptions vectorisées ont fait l’objet d’un upsert dans le conteneur Cosmos DB. Le processus de vectorisation et d’upsert des données peut prendre jusqu’à 10 à 15 minutes pour les 295 enregistrements du jeu de données des produits. Si tous les produits ne sont pas insérés, vous pouvez réexécuter `main.py` à l’aide de la commande ci-dessus pour ajouter les produits restants.

## Passer en revue les exemples de données ayant fait l’objet d’un upsert dans Cosmos DB

1. Revenez au portail Azure (``portal.azure.com``) et accédez à votre compte Azure Cosmos DB.

2. Sélectionnez **Explorateur de données** dans le menu de navigation de gauche.

3. Développez la base de données **CosmicWorks** et le contenu **Produits**, puis sélectionnez **Éléments** sous le conteneur.

4. Sélectionnez plusieurs documents aléatoires dans le conteneur et vérifiez que le champ `embedding` est rempli avec un tableau de vecteurs.
