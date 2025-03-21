---
lab:
  title: Configurer Azure Cosmos DB
  module: Setup
---

# Configurer Azure Cosmos DB

Dans cet exercice, vous allez créer un compte Azure Cosmos DB for NoSQL que vous allez utiliser tout au long des modules du labo et accorder à votre identité de l’utilisateur l’accès à la gestion des données dans le compte en lui attribuant le rôle **Contributeur aux données intégrées Cosmos DB**. Cela vous permettra d’utiliser l’authentification Azure pour accéder à la base de données à partir de votre code de labo et d’éviter de devoir stocker et gérer des clés.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Lorsque vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que vous souhaitez que le compte prenne en charge. Une fois l’approvisionnement du compte Azure Cosmos DB for NoSQL effectué, vous pouvez récupérer le point de terminaison et la clé, puis les utiliser pour vous connecter au compte Azure Cosmos DB for NoSQL en utilisant le kit de développement logiciel (SDK) Azure pour Python ou tout autre SDK de votre choix.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Sans serveur* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Copiez le champ **URI** et enregistrez-le dans un éditeur de texte pour plus tard. Vous utiliserez cette valeur de **point de terminaison** plus tard dans cet exercice.

1. Laissez l’onglet du navigateur ouvert pour l’étape suivante.

## Fournir à votre identité d’utilisateur le rôle RBAC Contributeur aux données intégrées Cosmos DB

Pour la dernière tâche de cet exercice, vous allez accorder à votre identité de l’utilisateur Microsoft Entra ID l’accès à la gestion des données dans votre compte Azure Cosmos DB for NoSQL en lui attribuant le rôle RBAC **Contributeur aux données intégrées Cosmos DB**. Cela vous permettra d’utiliser l’authentification Azure pour accéder à la base de données à partir de votre code et d’éviter de devoir stocker et gérer des clés.

> &#128221; L’utilisation du contrôle d’accès en fonction du rôle (RBAC) de Microsoft Entra ID pour l’authentification auprès des services Azure comme Azure Cosmos DB présente plusieurs avantages majeurs par rapport aux méthodes basées sur des clés. Le RBAC Entra ID renforce la sécurité grâce à des contrôles d’accès précis adaptés aux rôles d’utilisateur, ce qui réduit efficacement les risques d’accès non autorisés. Il simplifie également la gestion des utilisateurs, ce qui permet aux administrateurs d’attribuer et de modifier dynamiquement des autorisations sans devoir distribuer et gérer des clés de chiffrement. Cette approche améliore également la conformité et l’audit en s’alignant sur les stratégies organisationnelles et en facilitant la surveillance et l’examen complets des accès. En rationalisant la gestion des accès sécurisée, le RBAC Entra ID rend une solution plus efficace et évolutive pour tirer parti des services Azure.

1. Dans la barre d’outils du [portail Azure](https://portal.azure.com), ouvrez Cloud Shell.

    ![L’icône Cloud Shell est mise en surbrillance dans la barre d’outils du portail Azure.](media/azure-portal-toolbar-cloud-shell.png)

1. Au prompt Cloud Shell, vérifiez que votre abonnement à l’exercice est utilisé pour les commandes suivantes en exécutant `az account set -s <SUBSCRIPTION_ID>`, en remplaçant le jeton d’espace réservé `<SUBSCRIPTION_ID>` par l’ID de l’abonnement que vous utilisez pour cet exercice.

1. Copiez la sortie de la commande ci-dessus à utiliser en tant que jeton `<PRINCIPAL_OBJECT_ID>` dans la commande `az cosmosdb sql role assignment create` ci-dessous.

1. Vous allez ensuite récupérer l’ID de définition du rôle **Contributeur aux données intégrées Cosmos DB**. Exécutez la commande suivante, en veillant à remplacer les jetons `<RESOURCE_GROUP_NAME>` et `<COSMOS_DB_ACCOUNT_NAME>`.

    ```bash
    az cosmosdb sql role definition list --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>"
    ```

    Passez en revue la sortie et recherchez la définition de rôle nommée **Contributeur de données intégré Cosmos DB**. La sortie contient l’identificateur unique de la définition de rôle dans la propriété `name`. Enregistrez cette valeur, car vous devrez l’utiliser dans l’étape d’attribution plus loin dans l’étape suivante.

1. Vous êtes maintenant prêt à vous attribuer la définition du rôle **Contributeur aux données intégrées Cosmos DB**. Entrez la commande suivante lorsque vous y êtes invité, en veillant à remplacer les jetons `<RESOURCE_GROUP_NAME>` et `<COSMOS_DB_ACCOUNT_NAME>`.

    > &#128221; Dans la commande ci-dessous, l’`role-definition-id` est défini sur `00000000-0000-0000-0000-000000000002`, qui est la valeur par défaut pour la définition du rôle **Contributeur aux données intégrées Cosmos DB**. Si la valeur que vous avez récupérée de la commande `az cosmosdb sql role definition list` est différente, remplacez la valeur dans la commande ci-dessous avant de l’exécuter. La commande `az ad signed-in-user show` récupère l’ID d’objet de l’utilisateur Entra ID connecté.

    ```bash
    az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
    ```

1. Une fois la commande exécutée, vous serez en mesure d’exécuter du code localement pour insérer des données stockées dans votre base de données NoSQL Cosmos DB.

1. Fermez Cloud Shell.
