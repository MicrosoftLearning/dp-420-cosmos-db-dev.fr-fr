---
lab:
  title: "07.3 - Générer un Copilot avec Python et Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Générer un Copilot avec Python et Azure Cosmos DB for NoSQL

En utilisant les fonctionnalités de programmation polyvalentes de Python et les fonctionnalités évolutives de base de données NoSQL et de recherche vectorielle d’Azure Cosmos DB, vous pouvez créer des Copilots IA puissants et efficaces, en rationalisant les workflows complexes.

Dans ce labo, vous allez générer un Copilot à l’aide de Python et d’Azure Cosmos DB for NoSQL, en créant une API back-end qui fournira des points de terminaison nécessaires pour interagir avec les services Azure (Azure OpenAI et Azure Cosmos DB) et une interface utilisateur front-end pour faciliter l’interaction utilisateur avec Copilot. Copilot servira d’assistant pour aider les utilisateurs de Cosmic Works à gérer et à trouver des produits liés au vélo. Plus précisément, Copilot permet aux utilisateurs d’appliquer et de supprimer des remises des catégories de produits, de rechercher des catégories de produits pour aider à informer les utilisateurs des types de produits disponibles et à utiliser la recherche vectorielle pour effectuer des recherches de similarités pour les produits.

![Diagramme d’architecture de Copilot de haut niveau montrant une interface utilisateur développée en Python à l’aide de Streamlit, d’une API back-end écrite en Python et d’interactions avec Azure Cosmos DB et Azure OpenAI.](media/07-copilot-high-level-architecture-diagram.png)

La séparation des fonctionnalités d’application en une interface utilisateur dédiée et une API back-end lors de la création d’un Copilot dans Python offre plusieurs avantages. Tout d’abord, elle améliore la modularité et la maintenabilité, ce qui vous permet de mettre à jour l’interface utilisateur ou le back-end indépendamment sans perturber l’autre. Streamlit fournit une interface intuitive et interactive qui simplifie les interactions utilisateur, tandis que FastAPI garantit une gestion des demandes et un traitement des données asynchrones hautes performances. Cette séparation favorise également la scalabilité, car différents composants peuvent être déployés sur plusieurs serveurs, ce qui optimise l’utilisation des ressources. En outre, elle permet de meilleures pratiques de sécurité, car l’API back-end peut gérer séparément les données sensibles et l’authentification, ce qui réduit le risque d’exposer des vulnérabilités dans la couche d’interface utilisateur. Cette approche aboutit à une application plus robuste, efficace et conviviale.

> &#128721; Les exercices précédents de ce module sont des prérequis pour ce labo. Si vous avez toujours besoin d’effectuer l’un de ces exercices, veuillez les terminer avant de continuer, car ils fournissent l’infrastructure et le code de démarrage nécessaires pour ce labo.

## Construire une API back-end

L’API back-end pour le copilote enrichit ses capacités à gérer des données complexes, fournit des informations en temps réel et se connecte en toute fluidité à divers services, ce qui rend les interactions plus dynamiques et plus informatives. Pour générer l’API pour votre Copilot, vous allez utiliser la bibliothèque Python FastAPI. FastAPI est un cadre web moderne et hautes performances conçu pour vous permettre de générer des API avec Python basées sur des indicateurs de type Python standard. En découplant Copilot du back-end à l’aide de cette approche, vous garantissez une plus grande flexibilité, maintenabilité et scalabilité, ce qui permet à Copilot d’évoluer indépendamment des modifications du back-end.

> &#128721; L’API back-end s’appuie sur le code que vous avez ajouté au fichier `main.py` dans le dossier `python/07-build-copilot/api/app` dans l’exercice précédent. Si vous n’avez pas encore terminé l’exercice précédent, veuillez le terminer avant de continuer.

1. À l’aide de Visual Studio Code, ouvrez le dossier dans lequel vous avez cloné le référentiel de code du labo pour le module d’apprentissage **Générer des Copilots avec Azure Cosmos DB**.

1. Dans le volet **Explorateur** de Visual Studio Code, accédez au dossier **python/07-build-copilot/api/app** et ouvrez le fichier `main.py` trouvé dans celui-ci.

1. Ajoutez les lignes de code suivantes sous les instructions `import` existantes en haut du fichier `main.py` pour importer les bibliothèques qui seront utilisées pour effectuer des actions asynchrones à l’aide de FastAPI :

   ```python
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
   ```

1. Pour activer le point de terminaison `/chat` que vous allez créer pour recevoir des données dans le corps de la demande, vous transmettez du contenu via un objet `CompletionRequest` défini dans le module *modèles* de projets. Mettez à jour l’instruction d’importation `from models import Product` en haut du fichier pour inclure la classe `CompletionRequest` à partir du module `models`. L’instruction d’importation doit maintenant ressembler à ceci :

   ```python
   from models import Product, CompletionRequest
   ```

1. Vous aurez besoin du nom de déploiement du modèle de saisie semi-automatique de conversation que vous avez créé dans votre Azure OpenAI Service. Créez une variable en bas du bloc de variable de configuration Azure OpenAI pour fournir ceci :

   ```python
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
   ```

    Si votre nom de déploiement de saisie semi-automatique est différent, mettez à jour la valeur attribuée à la variable en conséquence.

1. Les kits de développement logiciel (SDK) Azure Cosmos DB et Identité fournissent des méthodes asynchrones pour l’utilisation de ces services. Chacune de ces classes est utilisée dans plusieurs fonctions de votre API. Vous allez donc créer des instances globales de chacune d’elles, ce qui permet au même client d’être partagé entre les méthodes. Insérez les déclarations de variables globales suivantes sous le bloc des variables de configuration Cosmos DB :

   ```python
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   ```

1. Supprimez les lignes de code suivantes du fichier, car les fonctionnalités fournies seront déplacées dans la fonction `lifespan` que vous définirez à l’étape suivante :

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

1. Pour créer des instances singleton des classes `CosmosClient` et des classes `DefaultAzureCredentail`, vous bénéficierez de l’objet `lifespan` dans FastAPI : cette méthode gère ces classes via le cycle de vie de l’application API. Insérez le code suivant pour définir la `lifespan` :

   ```python
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
   ```

   Dans FastAPI, les événements de durée de vie sont des opérations spéciales qui s’exécutent au début et à la fin du cycle de vie de l’application. Ces opérations s’exécutent avant que l’application commence à gérer les demandes et après son arrêt, ce qui les rend idéales pour initialiser et nettoyer les ressources utilisées dans l’ensemble de l’application et partagées entre les demandes. Cette approche garantit que la configuration nécessaire est effectuée avant que toutes les demandes soient traitées et que les ressources soient correctement gérées lors de l’arrêt.

1. Créez une instance de la classe FastAPI à l’aide du code suivant. Cela doit être inséré sous la fonction `lifespan` :

   ```python
   app = FastAPI(lifespan=lifespan)
   ```

   En appelant `FastAPI()`, vous initialisez une nouvelle instance de l’application FastAPI. Cette instance, appelée `app`, servira de point d’entrée principal pour votre application web. En transmettant la `lifespan`, vous attachez le gestionnaire d’événements de durée de vie à votre application.

1. Extrayez ensuite les points de terminaison de votre API. La méthode `api_status` est attachée à l’URL racine de votre API et sert de message d’état pour montrer que l’API est correctement opérationnelle. Vous allez générer le point de terminaison `/chat`  plus loin dans cet exercice. Insérez le code suivant sous le code pour créer le client, la base de données et le conteneur Cosmos DB :

   ```python
   @app.get("/")
   async def api_status():
       """Display a status message for the API"""
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

1. Remplacez le bloc de protection principal en bas du fichier pour démarrer le serveur web `uvicorn` ASGI (Asynchronous Server Gateway Interface) lorsque le fichier est exécuté à partir de la ligne de commande :

   ```python
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Enregistrez le fichier `main.py`. Il doit maintenant ressembler à ce qui suit, y compris les méthodes `generate_embeddings` et `upsert_product` que vous avez ajoutées dans l’exercice précédent :

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product, CompletionRequest
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
    
   app = FastAPI(lifespan=lifespan)
    
   @app.get("/")
   async def api_status():
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """ Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
    
   async def generate_embeddings(text: str):
       # Create Azure OpenAI client
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
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Pour tester rapidement votre API, ouvrez une nouvelle fenêtre de terminal intégré dans Visual Studio Code.

1. Vérifiez que vous êtes connecté à Azure à l’aide de la commande `az login`. Exécutez la commande suivante au prompt de terminal :

   ```bash
   az login
   ```

1. Terminez le processus de connexion dans votre navigateur.

1. Remplacez les répertoires par `python/07-build-copilot` au prompt de terminal.

1. Vérifiez que la fenêtre de terminal intégré s’exécute dans votre environnement virtuel Python en l’activant à l’aide d’une commande du tableau ci-dessous et en sélectionnant la commande appropriée pour votre système d’exploitation et votre interpréteur de commandes.

    | Plateforme | Shell | Commande pour activer l’environnement virtuel |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | poisson | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

1. Au prompt de terminal, remplacez les répertoires par `api/app`, puis exécutez la commande suivante pour exécuter l’application web FastAPI :

   ```bash
   uvicorn main:app
   ```

1. Si elle ne s’ouvre pas automatiquement, lancez une nouvelle fenêtre de navigateur web ou un nouvel onglet et accédez à <http://127.0.0.1:8000>.

    Le message `{"status":"ready"}` dans la fenêtre du navigateur indique que votre API est en cours d’exécution.

1. Accédez à l’interface utilisateur Swagger de l’API en ajoutant `/docs` à la fin de l’URL : <http://127.0.0.1:8000/docs>.

    > &#128221; L’interface utilisateur Swagger est une interface web interactive permettant d’explorer et de tester les points de terminaison d’API générés à partir de spécifications OpenAPI. Elle permet aux développeurs et aux utilisateurs de visualiser, d’interagir et de déboguer des appels d’API en temps réel, améliorant ainsi la facilité d’utilisation et la documentation.

1. Revenez à Visual Studio Code et arrêtez l’application API en appuyant sur **Ctrl+C** dans la fenêtre de terminal intégré associée.

## Incorporer des données de produit à partir d’Azure Cosmos DB

En tirant parti des données d’Azure Cosmos DB, Copilot peut simplifier les workflows complexes et aider les utilisateurs à effectuer efficacement les tâches. Copilot peut mettre à jour les enregistrements et récupérer des valeurs de recherche en temps réel, ce qui garantit des informations précises et opportunes. Cette fonctionnalité permet à Copilot de fournir des interactions avancées, d’améliorer la capacité des utilisateurs à naviguer rapidement et précisément et à effectuer des tâches.

Les fonctions permettront au Copilot de gestion des produits d’appliquer des remises aux produits d’une catégorie. Ces fonctions seront le mécanisme par lequel Copilot récupère et interagit avec les données de produit Cosmic Works à partir d’Azure Cosmos DB.

1. Copilot utilisera une fonction asynchrone nommée `apply_discount` pour ajouter et supprimer des remises et des prix de vente sur les produits dans une catégorie spécifiée. Insérez le code de fonction suivant sous la fonction `upsert_product` en bas du fichier `main.py` :

   ```python
   async def apply_discount(discount: float, product_category: str) -> str:
       """Apply a discount to products in the specified category."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
           """,
           parameters = [
               {"name": "@product_category", "value": product_category}
           ]
       )
    
       # Apply the discount to the products
       async for item in query_results:
           item['discount'] = discount
           item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
           await container.upsert_item(item)
    
       return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
   ```

    Cette fonction effectue une recherche dans Azure Cosmos DB pour extraire tous les produits d’une catégorie et appliquer la remise demandée à ces produits. Elle calcule également le prix de vente de l’article à l’aide de la remise spécifiée et l’insère dans la base de données.

2. Vous allez ensuite ajouter une deuxième fonction nommée `get_category_names`, que le copilote appellera pour l’aider à savoir quelles catégories de produits sont disponibles lors de l’application ou de la suppression de remises des produits. Ajoutez la méthode ci-dessous sous la fonction `apply_discount` dans le fichier :

   ```python
   async def get_category_names() -> list:
       """Retrieve the names of all product categories."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
       # Get distinct product categories
       query_results = container.query_items(
           query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
       )
       categories = []
       async for category in query_results:
           categories.append(category)
       return list(categories)
   ```

    La fonction `get_category_names` interroge le conteneur `Products` pour récupérer une liste de noms de catégories distincts dans la base de données.

3. Enregistrez le fichier `main.py`.

## Implémenter le point de terminaison de conversation

Le point de terminaison `/chat` de l’API back-end sert d’interface via laquelle l’interface utilisateur front-end interagit avec les modèles Azure OpenAI et les données de produit Cosmic Works internes. Ce point de terminaison sert de pont de communication, ce qui permet d’envoyer l’entrée d’interface utilisateur au service Azure OpenAI, qui la traite ensuite à l’aide de modèles de langage sophistiqués. Les résultats sont ensuite retournés au front-end, ce qui permet des conversations intelligentes en temps réel. En tirant parti de cette configuration, les développeurs peuvent garantir une expérience utilisateur transparente et réactive, tandis que le back-end gère la tâche complexe de traitement du langage naturel et la génération de réponses appropriées. Cette approche prend également en charge la scalabilité et la maintenabilité en découplant le front-end de l’infrastructure IA sous-jacente.

1. Recherchez le stub du point de terminaison `/chat` que vous avez ajouté précédemment dans le fichier `main.py`.

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

    La fonction accepte `CompletionRequest` en tant que paramètre. L’utilisation d’une classe pour le paramètre d’entrée permet de transmettre plusieurs propriétés dans le point de terminaison d’API dans le corps de la demande. La classe `CompletionRequest` est définie dans le module *modèles* et inclut les propriétés de message utilisateur, d’historique des conversations et d’historique maximal. L’historique des conversations permet à Copilot de référencer les aspects précédents de la conversation avec l’utilisateur, afin qu’il ait toujours connaissance du contexte de l’ensemble de la discussion. La propriété `max_history` vous permet de définir le nombre de messages d’historique à transmettre dans le contexte du LLM. Cela vous permet de contrôler les utilisations des jetons pour votre prompt et d’éviter les limites TPM sur les demandes.

2. Pour commencer, supprimez la ligne `raise NotImplementedError("The chat endpoint is not implemented yet.")` de la fonction lorsque vous commencez le processus d’implémentation du point de terminaison.

3. La première chose que vous allez faire dans la méthode de point de terminaison de conversation est de fournir un prompt système. Ce prompt définit le « personnage » de Copilot, dictant comment Copilot doit interagir avec les utilisateurs, répondre aux questions et tirer parti des fonctions disponibles pour effectuer des actions.

   ```python
   # Define the system prompt that contains the assistant's persona.
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   """
   ```

4. Créez ensuite un tableau de messages à envoyer au LLM, en ajoutant le prompt système, tous les messages dans l’historique des conversations et le message utilisateur entrant. Ce code doit se trouver directement sous la déclaration de prompt système dans la fonction :

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{"role": "system", "content": system_prompt }]
    
   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)
    
   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

    La propriété `messages` encapsule l’historique de la conversation en cours. Elle inclut l’ensemble de la séquence d’entrées utilisateur et des réponses de l’IA, ce qui aide le modèle à maintenir le contexte. En référençant cet historique, l’IA peut générer des réponses cohérentes et contextuellement pertinentes, ce qui garantit que les interactions restent fluides et dynamiques. Cette propriété est essentielle pour permettre à l’IA de comprendre le flux et les nuances de la conversation au fur et à mesure qu’elle progresse.

5. Pour permettre à Copilot d’utiliser les fonctions que vous avez définies ci-dessus pour interagir avec les données d’Azure Cosmos DB, vous devez définir une collection d’« outils ». Le LLM appellera ces outils dans le cadre de son exécution. Azure OpenAI utilise des définitions de fonction pour permettre des interactions structurées entre l’IA et différents outils ou API. Lorsqu’une fonction est définie, elle décrit les opérations qu’elle peut effectuer, les paramètres nécessaires et toutes les entrées requises. Pour créer un tableau d’`tools`, fournissez le code suivant contenant les définitions de fonction pour les méthodes `apply_discount` et `get_category_names` que vous avez définies précédemment :

   ```python
   # Define function calling tools
   tools = [
       {
           "type": "function",
           "function": {
               "name": "apply_discount",
               "description": "Apply a discount to products in the specified category",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "discount": {"type": "number", "description": "The percent discount to apply."},
                       "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                   },
                   "required": ["discount", "product_category"]
               }
           }
       },
       {
           "type": "function",
           "function": {
               "name": "get_category_names",
               "description": "Retrieves the names of all product categories"
           }
       }
   ]
   ```

    En utilisant des définitions de fonction, Azure OpenAI garantit que les interactions entre l’IA et les systèmes externes sont bien organisées, sécurisées et efficaces. Cette approche structurée permet à l’IA d’effectuer des tâches complexes de manière transparente et fiable, améliorant ainsi ses fonctionnalités globales et son expérience utilisateur.

6. Créez un client Azure OpenAI asynchrone pour effectuer des demandes à votre modèle de saisie semi-automatique de conversation :

   ```python
   # Create Azure OpenAI client
   aoai_client = AsyncAzureOpenAI(
       api_version = AZURE_OPENAI_API_VERSION,
       azure_endpoint = AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
   )
   ```

7. Le point de terminaison de conversation effectue deux appels à Azure OpenAI pour tirer parti de l’appel de fonction. Le premier fournit l’accès du client Azure OpenAI aux outils :

   ```python
   # First API call, providing the model to the defined functions
   response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages,
       tools = tools,
       tool_choice = "auto"
   )
    
   # Process the model's response and add it to the conversation history
   response_message = response.choices[0].message
   messages.append(response_message)
   ```

8. La réponse à ce premier appel contient des informations du LLM sur les outils ou fonctions qu’il a déterminés comme étant nécessaires pour répondre à la demande. Vous devez inclure du code pour traiter les sorties de l’appel de fonction, les insérer dans l’historique des conversations afin que le LLM puisse les utiliser pour formuler une réponse sur les données contenues dans ces sorties :

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

    L’appel de fonction dans Azure OpenAI permet l’intégration transparente d’API ou d’outils externes directement dans la sortie de votre modèle. Lorsque le modèle détecte une demande pertinente, il construit un objet JSON avec les paramètres nécessaires, que vous exécutez ensuite. Le résultat est retourné au modèle, ce qui lui permet de fournir une réponse finale complète enrichie avec des données externes.

9. Pour terminer la demande avec les données enrichies d’Azure Cosmos DB, vous devez envoyer une deuxième demande à Azure OpenAI pour générer une saisie semi-automatique :

   ```python
   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )
   ```

10. Pour finir, retournez la réponse de saisie semi-automatique à l’interface utilisateur :

   ```python
   return final_response.choices[0].message.content
   ```

11. Enregistrez le fichier `main.py`. La méthode `generate_chat_completion` du point de terminaison `/chat` doit ressembler à ceci :

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
   ```

## Générer une interface utilisateur de conversation simple

L’interface utilisateur Streamlit fournit une interface permettant aux utilisateurs d’interagir avec votre Copilot.

1. L’interface utilisateur sera définie à l’aide du fichier `index.py` situé dans le dossier `python/07-build-copilot/ui`.

2. Ouvrez le fichier `index.py` et ajoutez les instructions d’importation suivantes au début du fichier pour démarrer :

   ```python
   import streamlit as st
   import requests
   ```

3. Configurez la page Streamlit définie dans le fichier `index.py` en ajoutant la ligne suivante sous les instructions `import` :

   ```python
   st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
   ```

4. L’interface utilisateur interagit avec l’API back-end à l’aide de la bibliothèque `requests` pour effectuer des appels au point de terminaison `/chat` que vous avez défini sur l’API. Vous pouvez encapsuler l’appel d’API dans une méthode qui attend le message utilisateur actuel et une liste de messages de l’historique des conversations.

   ```python
   async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
       """Send a message to the Copilot chat endpoint."""
       try:
           api_endpoint = "http://localhost:8000"
           request = {"message": message, "chat_history": chat_history}
           response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
           return response.json()
       except Exception as e:
           st.error(f"An error occurred: {e}")
           return""
   ```

5. Définissez la fonction `main`, qui est le point d’entrée des appels dans l’application.

   ```python
   async def main():
       """Main function for the Cosmic Works Product Management Copilot UI."""
    
       st.write(
           """
           # Cosmic Works Product Management Copilot
        
           Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
        
           **Ask the copilot to apply or remove a discount on a category of products or to find products.**
           """
       )
    
       # Add a messages collection to the session state to maintain the chat history.
       if "messages" not in st.session_state:
           st.session_state.messages = []
    
       # Display message from the history on app rerun.
       for message in st.session_state.messages:
           with st.chat_message(message["role"]):
               st.markdown(message["content"])
    
       # React to user input
       if prompt := st.chat_input("What can I help you with today?"):
           with st. spinner("Awaiting the copilot's response to your message..."):
               # Display user message in chat message container
               with st.chat_message("user"):
                   st.markdown(prompt)
                
               # Send the user message to the copilot API
               response = await send_message_to_copilot(prompt, st.session_state.messages)
    
               # Display assistant response in chat message container
               with st.chat_message("assistant"):
                   st.markdown(response)
                
               # Add the current user message and assistant response messages to the chat history
               st.session_state.messages.append({"role": "user", "content": prompt})
               st.session_state.messages.append({"role": "assistant", "content": response})
   ```

6. Pour fini, ajoutez un bloc de **protection principal** à la fin du fichier :

   ```python
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

7. Enregistrez le fichier `index.py`.

## Tester Copilot via l’interface utilisateur

1. Revenez à la fenêtre de terminal intégré que vous avez ouverte dans Visual Studio Code pour le projet d’API et entrez ce qui suit pour démarrer l’application API :

   ```bash
   uvicorn main:app
   ```

2. Ouvrez une nouvelle fenêtre de terminal intégré, remplacez les répertoires par `python/07-build-copilot` pour activer votre environnement Python, puis remplacez les répertoires par le dossier `ui` et exécutez ce qui suit pour démarrer votre application d’interface utilisateur :

   ```bash
   python -m streamlit run index.py
   ```

3. Si l’interface utilisateur ne s’ouvre pas automatiquement dans une fenêtre du navigateur, lancez un nouvel onglet ou une nouvelle fenêtre de navigateur et accédez à <http://localhost:8501> pour ouvrir l’interface utilisateur.

4. Au prompt conversation de l’interface utilisateur, entrez « Appliquer la remise » et envoyez le message.

    Étant donné que vous deviez fournir à Copilot plus de détails pour agir, la réponse doit être une demande d’informations supplémentaires, comme fournir le pourcentage de remise que vous souhaitez appliquer et la catégorie de produits auxquels la remise doit être appliquée.

5. Pour comprendre les catégories disponibles, demandez à Copilot de vous fournir une liste de catégories de produits.

    Copilot effectue un appel de fonction à l’aide de la fonction `get_category_names` et enrichit les messages de conversation avec ces catégories pour pouvoir répondre en conséquence.

6. Vous pouvez également demander un ensemble plus spécifique de catégories, comme « Me fournir une liste de catégories liées aux vêtements ».

7. Demandez ensuite à Copilot d’appliquer une remise de 15 % à tous les produits vestimentaires.

8. Vous pouvez vérifier que la remise tarifaire a bien été appliquée en ouvrant votre compte Azure Cosmos DB dans le portail Azure, en sélectionnant l’**Explorateur de données** et en exécutant une requête sur le conteneur `Products` pour afficher tous les produits de la catégorie « vêtements », par exemple :

   ```sql
   SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
   WHERE CONTAINS(LOWER(c.category_name), "clothing")
   ```

    Notez que chaque élément dans les résultats de la requête a une valeur de `discount` de `0.15`, et que le `sale_price` doit être inférieur de 15 % au `price` d’origine.

9. Revenez à Visual Studio Code et arrêtez l’application API en appuyant sur **Ctrl+C** dans la fenêtre de terminal exécutant l’application. Vous pouvez laisser l’interface utilisateur en cours d’exécution.

## Intégrer la recherche vectorielle

Jusqu’à présent, vous avez donné à Copilot la capacité à réaliser des actions pour appliquer des remises aux produits, mais il n’a toujours pas connaissance des produits stockés dans la base de données. Dans cette tâche, vous allez ajouter des fonctionnalités de recherche vectorielle qui vous permettront de demander des produits avec certaines qualités et de trouver des produits similaires dans la base de données.

1. Revenez au fichier `main.py` dans le dossier `api/app` et fournissez une méthode pour effectuer des recherches vectorielles sur le conteneur `Products` dans votre compte Azure Cosmos DB. Vous pouvez insérer cette méthode sous les fonctions existantes en bas du fichier.

   ```python
   async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
       """Search for similar product vectors in Azure Cosmos DB"""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
           FROM Products p
           WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
           ORDER BY VectorDistance(p.embedding, @query_embedding)
           """,
           parameters = [
               {"name": "@query_embedding", "value": query_embedding},
               {"name": "@num_results", "value": num_results},
               {"name": "@similarity_score", "value": similarity_score}
           ]
       )
       similar_products = []
       async for result in query_results:
           similar_products.append(result)
       formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
       return formatted_results
   ```

2. Créez ensuite une méthode nommée `get_similar_products` qui servira de fonction utilisée par le LLM pour effectuer des recherches vectorielles sur votre base de données :

   ```python
   async def get_similar_products(message: str, num_results: int):
       """Retrieve similar products based on a user message."""
       # Vectorize the message
       embedding = await generate_embeddings(message)
       # Perform vector search against products in Cosmos DB
       similar_products = await vector_search(embedding, num_results=num_results)
       return similar_products
   ```

    La fonction `get_similar_products` effectue des appels asynchrones à la fonction `vector_search` que vous avez définie ci-dessus, ainsi qu’à la fonction `generate_embeddings` que vous avez créée dans l’exercice précédent. Les incorporations sont générées sur le message utilisateur entrant pour lui permettre de comparer les vecteurs stockés dans la base de données à l’aide de la fonction `VectorDistance` intégrée dans Cosmos DB.

3. Pour autoriser le LLM à utiliser les nouvelles fonctions, vous devez mettre à jour le tableau `tools` que vous avez créé précédemment, en ajoutant une définition de fonction pour la méthode `get_similar_products` :

   ```json
   {
       "type": "function",
       "function": {
           "name": "get_similar_products",
           "description": "Retrieve similar products based on a user message.",
           "parameters": {
               "type": "object",
               "properties": {
                   "message": {"type": "string", "description": "The user's message looking for similar products"},
                   "num_results": {"type": "integer", "description": "The number of similar products to return"}
               },
               "required": ["message"]
           }
       }
   }
   ```

4. Vous devez également ajouter du code pour gérer la sortie de la nouvelle fonction. Ajoutez la condition `elif` suivante au bloc de code qui gère les sorties d’appel de fonction :

   ```python
   elif call.function.name == "get_similar_products":
       func_response = await get_similar_products(**json.loads(call.function.arguments))
       messages.append(
           {
               "role": "tool",
               "tool_call_id": call.id,
               "name": call.function.name,
               "content": json.dumps(func_response)
           }
       )
   ```

    Le bloc terminé doit maintenant ressemble à ceci :

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
           elif call.function.name == "get_similar_products":
               func_response = await get_similar_products(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

5. Pour finir, vous devez mettre à jour la définition de prompt système pour fournir des instructions sur la façon d’effectuer des recherches vectorielles. Insérez ce qui suit en bas de l’`system_prompt` :

   ```plaintext
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   ```

    Le prompt système mis à jour ressemblera à ceci :

   ```python
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   """
   ```

6. Enregistrez le fichier `main.py`.

## Tester la fonctionnalité de recherche vectorielle

1. Redémarrez l’application API en exécutant ce qui suit dans la fenêtre de terminal intégré ouverte pour cette application dans Visual Studio Code :

   ```bash
   uvicorn main:app
   ```

2. L’interface utilisateur doit toujours être en cours d’exécution. Toutefois, si vous l’avez arrêtée, revenez à la fenêtre de terminal intégré correspondante et exécutez :

   ```bash
   python -m streamlit run index.py
   ```

3. Revenez à la fenêtre du navigateur exécutant l’interface utilisateur et, au prompt de conversation, entrez les éléments suivants :

   ```bash
   Tell me about the mountain bikes in stock
   ```

    Cette question retourne quelques produits qui correspondent à votre recherche.

4. Essayez quelques autres recherches, telles que « Montre-moi les pédales durables », « Fournis une liste de 5 maillots élégants » et « Donne-moi des détails sur tous les gants d’équitation par temps chaud ».

    Pour les deux dernières requêtes, notez que les produits contiennent la remise de 15 % et le prix de vente que vous avez appliqués précédemment.
