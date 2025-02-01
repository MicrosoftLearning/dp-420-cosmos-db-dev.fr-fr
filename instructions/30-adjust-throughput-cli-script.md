---
lab:
  title: "Ajuster le débit approvisionné à l’aide d’un script Azure\_CLI"
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Ajuster le débit approvisionné à l’aide d’un script Azure CLI

Azure CLI est un ensemble de commandes que vous pouvez utiliser pour gérer différentes ressources dans Azure. Azure Cosmos DB dispose d’un groupe de commandes riche qui peut être utilisé pour gérer différents aspects d’un compte Azure Cosmos DB, quel que soit l’API sélectionnée.

Dans ce labo, vous allez créer un compte Azure Cosmos DB, une base de données et un conteneur avec Azure CLI. Vous allez ensuite apporter des ajustements au débit approvisionné à l’aide d’Azure CLI.

## Se connecter à Azure CLI

Avant d’utiliser Azure CLI, vous devez d’abord vérifier la version de l’interface CLI et vous connecter avec vos informations d’identification Azure.

1. Démarrez **Visual Studio Code**.

1. Ouvrez le menu **Terminal**, puis sélectionnez **Nouveau terminal** pour ouvrir une nouvelle instance de terminal.

1. Affichez la version de l’interface Azure CLI en utilisant la commande ci-après :

    ```
    az --version
    ```

1. Affichez les groupes de commandes Azure CLI les plus courants en utilisant la commande suivante :

    ```
    az --help
    ```

1. Installez les certificats tls/ssl avant de vous connecter à Azure :

    ```
    cd "C:\Program Files\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    ```

1. Commencez la procédure de connexion interactive de l’interface Azure CLI à l’aide de la commande suivante :

    ```
    az login
    ```

1. Azure CLI ouvre automatiquement une fenêtre ou un onglet de navigateur web. Dans l’instance de navigateur, connectez-vous à Azure CLI avec les informations d’identification Microsoft associées à votre abonnement.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

1. Regardez si votre fournisseur de labo a créé un groupe de ressources pour vous, et le cas échéant, enregistrez son nom parce que vous en aurez besoin dans la section suivante.

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    Cette commande peut retourner plusieurs noms de groupe de ressources.

1. (Facultatif) ***Si aucun groupe de ressources n’a été créé pour vous***, choisissez un nom de groupe de ressources et créez-le. *Notez que certains environnements de labo risquent d’être verrouillés et que vous aurez besoin d’un administrateur pour créer le groupe de ressources pour vous.*

    i. Choisissez le nom de l’emplacement le plus proche de vous dans cette liste.

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. Créez le groupe de ressources.  *Notez que certains environnements de labo risquent d’être verrouillés et que vous aurez besoin d’un administrateur pour créer le groupe de ressources pour vous.*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

## Créer un compte Azure Cosmos DB à l’aide d’Azure CLI

Le groupe de commandes **cosmosdb** contient des commandes de base pour créer et gérer des comptes Azure Cosmos DB à l’aide de l’interface CLI. Étant donné qu’un compte Azure Cosmos DB a un URI adressable, il est important de créer un nom global unique pour votre nouveau compte, même si vous le créez via un script.

1. Revenez à l’instance de terminal déjà ouverte dans **Visual Studio Code**.

1. Affichez les commandes Azure CLI les plus courantes liées à **Azure Cosmos DB** à l’aide de la commande suivante :

    ```
    az cosmosdb --help
    ```

1. Créez une variable nommée **suffix** avec la cmdlet PowerShell [Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] à l’aide de la commande suivante :

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; La cmdlet Get-Random génère un entier aléatoire compris entre 0 et 1 000 000. Cela est utile parce que nos services nécessitent un nom global unique.

1. Créez un autre nom de variable **accountName** à l’aide de la chaîne codée en dur **csms** et une substitution de variable pour injecter la valeur de la variable **$suffix** à l’aide de la commande suivante :

    ```
    $accountName="csms$suffix"
    ```

1. Créez un autre nom de variable **resourceGroup** en utilisant le nom du groupe de ressources que vous avez créé ou affiché précédemment dans ce labo à l’aide de la commande suivante :

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Par exemple, si votre groupe de ressources est nommé **dp420**, la commande est **$resourceGroup="dp420"**.

1. Utilisez la cmdlet **echo** pour écrire la valeur des variables **$accountName** et **$resourceGroup** dans la sortie du terminal à l’aide de la commande suivante :

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. Affichez les options pour **az cosmosdb create** à l’aide de la commande suivante :

    ```
    az cosmosdb create --help
    ```

1. Créez un compte Azure Cosmos DB à l’aide des variables prédéfinies et de la commande suivante :

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. Attendez que la commande **create** termine l’exécution et retourne la sortie avant de poursuivre ce labo.

    > &#128161; La commande **create** peut prendre entre deux et douze minutes, en moyenne.

## Créer des ressources Azure Cosmos DB for NoSQL en utilisant Azure CLI

Le groupe de commandes **cosmosdb sql** contient des commandes pour la gestion des ressources propres à l’API NoSQL pour Azure Cosmos DB. Vous pouvez toujours utiliser l’indicateur **--help** pour passer en revue les options de ces groupes de commandes.

1. Revenez à l’instance de terminal déjà ouverte dans **Visual Studio Code**.

1. Affichez les groupes de commandes Azure CLI les plus courants liés à **Azure Cosmos DB for NoSQL** à l’aide de la commande suivante :

    ```
    az cosmosdb sql --help
    ```

1. Affichez les commandes Azure CLI pour la gestion des bases de données **Azure Cosmos DB for NoSQL** à l’aide de la commande suivante :

    ```
    az cosmosdb sql database --help
    ```

1. Créez une base de données Azure Cosmos DB à l’aide des variables prédéfinies, du nom de base de données **cosmicworks** et de la commande suivante :

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Attendez que la commande **create** termine l’exécution et retourne la sortie avant de poursuivre ce labo.

1. Affichez les commandes Azure CLI pour la gestion des conteneurs **Azure Cosmos DB for NoSQL** à l’aide de la commande suivante :

    ```
    az cosmosdb sql container --help
    ```

1. Créez un conteneur Azure Cosmos DB à l’aide des variables prédéfinies, du nom de base de données **cosmicworks**, du nom de conteneur **products** et de la commande suivante :

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Attendez que la commande **create** termine l’exécution et retourne la sortie avant de poursuivre ce labo.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **Groupes de ressources**, sélectionnez le groupe de ressources que vous avez créé ou affiché précédemment dans ce labo, puis sélectionnez la ressource **Compte Azure Cosmos DB** que vous avez créée dans ce labo avec le préfixe **csms**.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NoSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Mise à l’échelle et paramètres**.

1. Observez les valeurs dans l’onglet **Mise à l’échelle**. Notez en particulier que l’option **Manuel** est sélectionnée dans la section **Débit** et que le débit approvisionné est défini sur **400** RU/s.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

## Ajuster le débit d’un conteneur existant à l’aide d’Azure CLI

L’interface Azure CLI peut être utilisée pour migrer un conteneur d’un approvisionnement manuel à un approvisionnement mis à l’échelle automatiquement du débit. Si le conteneur utilise le débit mis à l’échelle automatiquement, l’interface CLI peut être utilisée pour ajuster dynamiquement la valeur de débit maximale autorisée.

1. Revenez à l’instance de terminal déjà ouverte dans **Visual Studio Code**.

1. Affichez les commandes Azure CLI pour la gestion du débit des conteneurs **Azure Cosmos DB for NoSQL** à l’aide de la commande suivante :

    ```
    az cosmosdb sql container throughput --help
    ```

1. Migrez le débit du conteneur **products** de l’approvisionnement manuel vers l’approvisionnement mis à l’échelle automatiquement à l’aide de la commande suivante :

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Attendez que la commande **migrate** termine l’exécution et retourne la sortie avant de poursuivre ce labo.

1. Interrogez le conteneur **products** pour déterminer la valeur de débit minimale possible à l’aide de la commande suivante :

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Mettez à jour le débit mis à l’échelle automatiquement maximal du conteneur **products** de la valeur par défaut actuelle de **1 000** vers la nouvelle valeur de **5 000** à l’aide de la commande suivante :

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Attendez que la commande **update** termine l’exécution et retourne la sortie avant de poursuivre ce labo.

1. Fermez **Visual Studio Code**.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **Groupes de ressources**, sélectionnez le groupe de ressources que vous avez créé ou affiché précédemment dans ce labo, puis sélectionnez la ressource **Compte Azure Cosmos DB** que vous avez créée dans ce labo avec le préfixe **csms**.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NoSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Mise à l’échelle et paramètres**.

1. Observez les valeurs dans l’onglet **Mise à l’échelle**. Notez en particulier que l’option **Mise à l’échelle automatique** est sélectionnée dans la section **Débit** et que le débit approvisionné est défini sur **5 000** RU/s.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
