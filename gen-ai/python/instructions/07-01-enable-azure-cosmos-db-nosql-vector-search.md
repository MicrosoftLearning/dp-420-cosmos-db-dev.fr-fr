---
title: "07.1 - Activer la recherche vectorielle pour Azure\_Cosmos\_DB\_for\_NoSQL"
lab:
  title: "07.1 - Activer la recherche vectorielle pour Azure\_Cosmos\_DB\_for\_NoSQL"
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 10
parent: Python SDK labs
---

# Activer la recherche vectorielle pour Azure Cosmos DB for NoSQL

Azure Cosmos DB for NoSQL fournit une fonctionnalité efficace d’indexation et de recherche vectorielle fiable conçue pour stocker et interroger efficacement et avec précision des vecteurs de grande dimension à n’importe quelle échelle. Pour bénéficier de cette fonctionnalité, vous devez autoriser votre compte à utiliser la fonctionnalité *Recherche vectorielle pour l’API NoSQL*.

Dans ce labo, vous allez créer un compte Azure Cosmos DB for NoSQL et activer la fonctionnalité Recherche vectorielle sur celui-ci afin de préparer une base de données à utiliser en tant que magasin de vecteurs.

## Préparer votre environnement de développement

Si vous n’avez pas déjà cloné le référentiel de code du labo pour **Générer des Copilots avec Azure Cosmos DB** et configuré votre environnement local, consultez les instructions dans [Configurer un environnement de labo local](00-setup-lab-environment.md) pour ce faire.

## Créer un compte Azure Cosmos DB for NoSQL

Si vous avez déjà créé un compte Azure Cosmos DB for NoSQL pour les labos **Générer des Copilots avec Azure Cosmos DB** sur ce site, vous pouvez l’utiliser pour ce labo et passer à la [section suivante](#enable-vector-search-for-nosql-api). Dans le cas contraire, consultez les instructions dans [Configurer Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) pour créer un compte Azure Cosmos DB for NoSQL que vous utiliserez tout au long des modules du labo et accordez à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**.

## Activer la recherche vectorielle pour l’API NoSQL

Dans cette tâche, vous allez activer la fonctionnalité *Recherche vectorielle pour l’API NoSQL* dans votre compte Azure Cosmos DB à l’aide d’Azure CLI.

1. Dans la barre d’outils du [portail Azure](https://portal.azure.com), ouvrez Cloud Shell.

    ![L’icône Cloud Shell est mise en surbrillance dans la barre d’outils du portail Azure.](media/07-azure-portal-toolbar-cloud-shell.png)

2. Au prompt Cloud Shell, vérifiez que votre abonnement à l’exercice est utilisé pour les commandes suivantes en exécutant `az account set -s <SUBSCRIPTION_ID>`, en remplaçant le jeton d’espace réservé `<SUBSCRIPTION_ID>` par l’ID de l’abonnement que vous utilisez pour cet exercice.

3. Activez la fonctionnalité *Recherche vectorielle pour l’API NoSQL* en exécutant la commande suivante à partir d’Azure Cloud Shell, en remplaçant les jetons `<COSMOS_DB_ACCOUNT_NAME>` et `<RESOURCE_GROUP_NAME>` par le nom de votre groupe de ressources et le nom du compte Azure Cosmos DB, respectivement.

     ```bash
     az cosmosdb update \
       --resource-group <RESOURCE_GROUP_NAME> \
       --name <COSMOS_DB_ACCOUNT_NAME> \
       --capabilities EnableNoSQLVectorSearch
     ```

4. Attendez que la commande s’exécute correctement avant de quitter Cloud Shell.

5. Fermez Cloud Shell.

## Créer une base de données et un conteneur pour les vecteurs d’hébergement

1. Sélectionnez **Explorateur de données** dans le menu de gauche de votre compte Azure Cosmos DB dans le [portail Azure](https://portal.azure.com), puis sélectionnez **Nouveau conteneur**.

2. Dans la boîte de dialogue **Nouveau conteneur** :
   1. Sous **ID de base de données**, sélectionnez **Créer** et entrez « CosmicWorks » dans le champ d’ID de base de données.
   2. Dans la zone **ID de conteneur**, entrez le nom « Produits ».
   3. Attribuez « /category_id » comme **Clé de partition**.

      ![Capture d’écran des paramètres Nouveau conteneur spécifiés ci-dessus entrés dans la boîte de dialogue.](media/07-azure-cosmos-db-new-container.png)

   4. Faites défiler jusqu’au bas de la boîte de dialogue **Nouveau conteneur**, développez **Stratégie de vecteur de conteneur**, puis sélectionnez **Ajouter une incorporation vectorielle**.

   5. Dans la section des paramètres **Stratégie de vecteur de conteneur**, définissez les éléments suivants :

      | Paramètre | Valeur |
      | ------- | ----- |
      | **Chemin d’accès** | Entrez */embedding*. |
      | **Type de données** | Sélectionnez *float32*. |
      | **Fonction de distance** | Sélectionnez *cosinus*. |
      | **Dimensions** | Entrez *1536* pour correspondre au nombre de dimensions produites par le modèle `text-embedding-3-small`  d’OpenAI. |
      | **Type d’index** | Sélectionnez *diskANN*. |
      | **Taille d’octets de quantification** | Laissez ce champ vide. |
      | **Indexation de la taille de la liste de recherche** | Acceptez la valeur par défaut *100*. |

      ![Capture d’écran de la stratégie de vecteur de conteneur spécifiée ci-dessus entrée dans la boîte de dialogue Nouveau conteneur.](media/07-azure-cosmos-db-container-vector-policy.png)

   6. Sélectionnez **OK** pour créer la base de données et le conteneur.

   7. Attendez que le conteneur soit créé avant de continuer. La préparation du conteneur peut prendre plusieurs minutes.
