---
title: "07.4 - Implémenter RAG avec LangChain et la recherche vectorielle Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "07.4 - Implémenter RAG avec LangChain et la recherche vectorielle Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 13
parent: Python SDK labs
---

# Implémenter RAG avec LangChain et la recherche vectorielle Azure Cosmos DB for NoSQL

Les fonctionnalités d’orchestration de LangChain offrent une multitude d’avantages par rapport à l’implémentation directe de l’intégration LLM de votre Copilot à l’aide du client Azure OpenAI. LangChain permet une intégration plus transparente avec différentes sources de données, notamment Azure Cosmos DB, ce qui permet une recherche vectorielle efficace qui améliore le processus de récupération. LangChain offre des outils robustes pour la gestion et l’optimisation des workflows, ce qui facilite la création d’applications complexes avec des composants modulaires et réutilisables. Cette flexibilité simplifie non seulement le développement, mais garantit également la scalabilité et la maintenabilité.

Dans ce labo, vous allez améliorer votre Copilot en passant du point de terminaison de `/chat` de votre API à l’aide du client Azure OpenAI pour tirer parti des fonctionnalités puissantes d’orchestration de LangChain. Ce changement permettra une extraction de données plus efficace et des performances améliorées en intégrant la fonctionnalité de recherche vectorielle à Azure Cosmos DB for NoSQL. Que vous souhaitiez optimiser le processus de récupération des informations de votre application ou simplement explorer le potentiel de RAG, ce module vous guidera tout au long du processus transparent de conversion, en montrant comment LangChain peut simplifier et élever les fonctionnalités de votre application. Nous allons commencer ce parcours visant à libérer de nouvelles efficacités et informations avec LangChain et Azure Cosmos DB !

> &#128721; Les exercices précédents de ce module sont des prérequis pour ce labo. Si vous avez toujours besoin d’effectuer l’un de ces exercices, veuillez les terminer avant de continuer, car ils fournissent l’infrastructure et le code de démarrage nécessaires pour ce labo.

## Installer les bibliothèques LangChain

1. À l’aide de Visual Studio Code, ouvrez le dossier dans lequel vous avez cloné le référentiel de code du labo pour le module d’apprentissage **Générer des Copilots avec Azure Cosmos DB**.

2. Dans le volet **Explorateur** de Visual Studio Code, accédez au dossier **python/07-build-copilot** et ouvrez le fichier `requirements.txt` trouvé dans celui-ci.

3. Mettez à jour le fichier `requirements.txt` pour inclure les bibliothèques LangChain requises :

   ```ini
   langchain==0.3.9
   langchain-openai==0.2.11
   ```

4. Lancez une nouvelle fenêtre de terminal intégré dans Visual Studio Code et remplacez les répertoires par `python/07-build-copilot`.

5. Vérifiez que la fenêtre de terminal intégré s’exécute dans votre environnement virtuel Python en l’activant à l’aide de la commande appropriée pour votre système d’exploitation et votre interpréteur de commandes d’après le tableau suivant :

    | Plateforme | Shell | Commande pour activer l’environnement virtuel |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | poisson | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

6. Mettez à jour votre environnement virtuel avec les bibliothèques LangChain en exécutant la commande suivante au prompt de terminal intégré :

   ```bash
   pip install -r requirements.txt
   ```

7. Fermez le terminal intégré.

## Mettre à jour l’API back-end

Dans le labo précédent, vous avez exécuté un modèle RAG à l’aide du client Azure OpenAI et des données d’Azure Cosmos DB. Vous allez maintenant mettre à jour l’API back-end pour utiliser un agent LangChain avec des outils pour effectuer les mêmes actions.

L’utilisation de LangChain pour interagir avec les modèles de langage déployés dans votre Azure OpenAI Service est un peu plus simple du point de vue du code...

1. Supprimez l’instruction d’importation `from openai import AzureOpenAI` en haut du fichier `main.py`. Cette bibliothèque cliente n’est plus nécessaire, car toutes les interactions avec Azure OpenAI passeront par les classes fournies par LangChain.

2. Supprimez les instructions d’importation suivantes en haut du fichier `main.py`, car elles ne seront plus nécessaires :

   ```python
   from openai import AsyncAzureOpenAI
   import json
   ```

### Mettre à jour le point de terminaison d’incorporation

1. Importez la classe `AzureOpenAIEmbeddings` à partir de la bibliothèque `langchain_openai` en ajoutant l’instruction d’importation suivante en haut du fichier `main.py` :

   ```python
   from langchain_openai import AzureOpenAIEmbeddings
   ```

2. Recherchez la méthode `generate_embeddings` dans le fichier et remplacez-la par ce qui suit, qui utilise la classe `AzureOpenAIEmbeddings` pour gérer les interactions avec Azure OpenAI :

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Use LangChain's Azure OpenAI Embeddings class
       azure_openai_embeddings = AzureOpenAIEmbeddings(
           azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
       return await azure_openai_embeddings.aembed_query(text)
   ```

    La classe `AzureOpenAIEmbeddings` fournit une interface permettant d’interagir avec l’API Incorporations Azure OpenAI, en retournant un objet de réponse simplifié ne contenant que le vecteur généré.

### Mettre à jour le point de terminaison de conversation

1. Mettez à jour l’instruction d’importation `lanchain_openai` pour ajouter la classe `AzureChatOpenAI` :

   ```python
   from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
   ```

1. Importez les objets LangChain supplémentaires suivants qui seront utilisés lors de la génération du point de terminaison `/chat` révisé :

   ```python
   from langchain.agents import AgentExecutor, create_openai_functions_agent
   from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
   from langchain_core.tools import StructuredTool
   ```

1. L’historique des conversations sera injecté différemment dans la conversation de Copilot à l’aide d’un agent LangChain. Supprimez donc immédiatement les lignes de code suivant la définition `system_prompt`. La ligne que vous devez supprimer est la suivante :

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{ "role": "system", "content": system_prompt }]

   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)

   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

1. À la place du code que vous venez de supprimer, définissez un objet `prompt` à l’aide de la classe `ChatPromptTemplate` de LangChain :

   ```python
   prompt = ChatPromptTemplate.from_messages(
       [
           ("system", system_prompt),
           MessagesPlaceholder("chat_history", optional=True),
           ("user", "{input}"),
           MessagesPlaceholder("agent_scratchpad")
       ]
   )
   ```

    Le `ChatPromptTemplate` est créé avec plusieurs composants dans un ordre spécifique. Voici comment ces éléments s’articulent :

    - **Message système** : utilise la fonction `system_prompt` pour attribuer un personnage à Copilot, en fournissant des instructions sur le comportement et l’interaction de l’assistant avec les utilisateurs.
    - **Historique des conversations** : permet d’incorporer l’`chat_history`, qui contient une liste de messages précédents de la conversation, dans le contexte dans lequel fonctionne le LLM.
    - **Entrée utilisateur** : message de l’utilisateur actuel.
    - **Bloc-notes de l’agent** : permet de prendre des notes ou d’effectuer des étapes intermédiaires de l’agent.

    Le prompt qui en résulte fournit une entrée structurée pour l’agent d’IA conversationnel, ce qui lui permet de générer une réponse en fonction du contexte donné.

1. Remplacez ensuite la définition de tableau `tools` par ce qui suit, qui utilise la classe `StructuredTool` de LangChain pour extraire les définitions de fonction dans le format approprié :

   ```python
   tools = [
       StructuredTool.from_function(coroutine=apply_discount),
       StructuredTool.from_function(coroutine=get_category_names),
       StructuredTool.from_function(coroutine=get_similar_products)
   ]
   ```

    La méthode `StructuredTool.from_function` dans LangChain crée un outil à partir d’une fonction donnée, en utilisant les paramètres d’entrée et de la description docstring de la fonction. Pour l’utiliser avec des méthodes asynchrones, vous transmettez le nom de la fonction au paramètre d’entrée `coroutine`.

    Dans Python, un docstring (ou chaîne de documentation) est un type spécial de chaîne utilisé pour documenter une fonction, une méthode, une classe ou un module. Il offre un moyen pratique d’associer la documentation au code Python et est généralement placé entre guillemets triples (""" ou '''). Les docstrings sont placés immédiatement après la définition de la fonction (ou méthode, classe ou module) qu’ils documentent.

    L’utilisation de cette fonction automatise la création des définitions de fonction JSON que vous deviez créer manuellement à l’aide du client Azure OpenAI, ce qui simplifie le processus d’appel de fonction.

1. Supprimez tout le code entre la définition de tableau `tools` que vous avez effectuée ci-dessus et l’instruction `return` à la fin de la fonction. À l’aide du client Azure OpenAI, vous devez effectuer deux appels au modèle de langage. Le premier pour lui permettre de déterminer la fonction appelée et si, le cas échéant, le prompt doit être approfondi, et le second pour demander la saisie semi-automatique d’une RAG. Entre les deux, vous devez utiliser du code pour inspecter la réponse du premier appel afin de déterminer si les appels de fonction étaient requis, puis écrire du code pour « gérer » l’appel de ces fonctions. Vous devez ensuite insérer la sortie de ces appels de fonction dans les messages envoyés au LLM, afin qu’il puisse disposer du prompt enrichi dans la perspective de la formulation d’une réponse de saisie semi-automatique. LangChain simplifie considérablement le processus d’appel d’un LLM à l’aide d’un modèle RAG, comme vous le verrez ci-dessous. Le code que vous devez supprimer est le suivant :

   ```python
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

   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )

   return final_response.choices[0].message.content
   ```

1. Juste en dessous de la définition de tableau `tools`, créez une référence à l’API Azure OpenAI à l’aide de la classe `AzureChatOpenAI` dans LangChain :

   ```python
   # Connect to Azure OpenAI API
   azure_openai = AzureChatOpenAI(
       azure_deployment=COMPLETION_DEPLOYMENT_NAME,
       azure_endpoint=AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
       api_version=AZURE_OPENAI_API_VERSION
   )
   ```

1. Pour permettre à votre agent LangChain d’interagir avec les fonctions que vous avez définies, vous allez créer un agent à l’aide de la méthode `create_openai_functions_agent`, à laquelle vous fournirez l’objet `AzureChatOpenAI`, le tableau `tools` et l’objet `ChatPromptTemplate` :

   ```python
   agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
   ```

    La fonction `create_openai_functions_agent` dans LangChain crée un agent qui peut appeler des fonctions externes pour effectuer des tâches à l’aide d’un modèle de langage et d’outils spécifiés. Cela permet d’intégrer différents services et fonctionnalités dans le workflow de l’agent, ce qui offre une meilleure flexibilité et des fonctionnalités améliorées.

1. Dans LangChain, la classe `AgentExecutor` est utilisée pour gérer le flux d’exécution des agents, comme celui que vous avez créé avec la méthode `create_openai_functions_agent`. Il gère le traitement des entrées, l’appel d’outils ou de modèles et la gestion des sorties. Utilisez le code ci-dessous pour créer un exécuteur d’agent pour votre agent :

   ```python
   agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
   ```

    `AgentExecutor` garantit que toutes les étapes requises pour générer une réponse sont exécutées dans le bon ordre. Il extrait les complexités de l’exécution des agents, en fournissant une couche supplémentaire de fonctionnalité et de structure, et facilite la génération, la gestion et la mise à l’échelle d’agents sophistiqués.

1. Vous allez utiliser la méthode `invoke` de l’exécuteur d’agent pour envoyer le message utilisateur entrant au LLM. Vous allez également inclure l’historique des conversations. Insérez le code suivant en dessous de la définition `agent_executor` :

   ```python
   completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
   ```

   Les jetons `input` et `chat_history` ont été définis dans l’objet de prompt créé à l’aide du `ChatPromptTemplate`. Avec la méthode `invoke`, ceux-ci seront injectés dans le prompt, ce qui permet au LLM d’utiliser ces informations lors de la création d’une réponse.

1. Pour finir, mettez à jour l’instruction return pour utiliser la `output` de l’objet de saisie semi-automatique de l’agent :

   ```python
   return completion["output"]
   ```

1. Enregistrez le fichier `main.py`. La fonction de point de terminaison `/chat` mise à jour doit maintenant ressembler à ceci :

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
       When asked to provide a list of products, you should:
           - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
       """
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", system_prompt),
               MessagesPlaceholder("chat_history", optional=True),
               ("user", "{input}"),
               MessagesPlaceholder("agent_scratchpad")
           ]
       )
    
       # Define function calling tools
       tools = [
           StructuredTool.from_function(apply_discount),
           StructuredTool.from_function(get_category_names),
           StructuredTool.from_function(get_similar_products)
       ]
    
       # Connect to Azure OpenAI API
       azure_openai = AzureChatOpenAI(
           azure_deployment=COMPLETION_DEPLOYMENT_NAME,
           azure_endpoint=AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
           api_version=AZURE_OPENAI_API_VERSION
       )
    
       agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
       agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
        
       completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
            
       return completion["output"]
   ```

## Démarrer l’API et les applications d’interface utilisateur

1. Pour démarrer l’API, ouvrez une nouvelle fenêtre de terminal dans Visual Studio Code.

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

1. Ouvrez une nouvelle fenêtre de terminal intégré, remplacez les répertoires par `python/07-build-copilot` pour activer votre environnement Python, puis remplacez les répertoires par le dossier `ui` et exécutez ce qui suit pour démarrer votre application d’interface utilisateur :

   ```bash
   python -m streamlit run index.py
   ```

1. Si l’interface utilisateur ne s’ouvre pas automatiquement dans une fenêtre du navigateur, lancez un nouvel onglet ou une nouvelle fenêtre de navigateur et accédez à <http://localhost:8501> pour ouvrir l’interface utilisateur.

## Tester le copilote

1. Avant d’envoyer des messages dans l’interface utilisateur, revenez à Visual Studio Code et sélectionnez la fenêtre de terminal intégré associée à l’application API. Dans cette fenêtre, vous verrez le sortie « détaillée » générée par l’exécuteur d’agent LangChain, qui fournit des informations sur la façon dont LangChain gère les demandes que vous envoyez. Faites attention à la sortie de cette fenêtre lorsque vous envoyez les demandes ci-dessous, en vérifiant à nouveau après chaque appel.

1. Au prompt de conversation dans l’interface utilisateur, entrez « Appliquer la remise » et envoyez le message.

    Vous devez recevoir une réponse vous demandant le pourcentage de remise que vous souhaitez appliquer, et pour quelle catégorie de produit.

1. Répondez « Gants ».

    Vous recevrez une réponse vous demandant le pourcentage de remise que vous souhaitez appliquer à la catégorie « Gants ».

1. Envoyez le message « 25 % ».

    Vous devez obtenir une réponse similaire à « Une remise de 25 % a bien été appliquée à tous les produits de la catégorie « Gants » ».

1. Demandez à Copilot : « montre-moi tous les gants ».

    Dans la réponse, vous devriez voir une liste de tous les gants dans la base de données, dont le prix inclura une remise de 25 %.

1. Pour finir, posez la question suivante : « Quels sont les meilleurs gants pour faire de l’équitation par temps froid ? » pour effectuer une recherche vectorielle. Cela implique un appel de fonction à la méthode `get_similar_items`, qui appelle ensuite la méthode `generate_embeddings` que vous avez mise à jour pour utiliser une implémentation LangChain et la fonction `vector_search`.

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.
