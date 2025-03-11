---
lab:
  title: Créer un conteneur Azure Cosmos DB for NoSQL en utilisant des modèles Azure Resource Manager
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Créer un conteneur Azure Cosmos DB for NoSQL en utilisant des modèles Azure Resource Manager

Les modèles Azure Resource Manager sont des fichiers JSON qui définissent de manière déclarative l’infrastructure que vous souhaitez déployer sur Azure. Les modèles Azure Resource Manager sont une solution courante d’infrastructure en tant que code pour déployer des services sur Azure. Bicep pousse le concept un peu plus loin en définissant un langage dédié plus facile à lire qui peut être utilisé pour créer des modèles JSON.

Dans ce labo, vous allez créer un compte, une base de données et un conteneur Azure Cosmos DB à l’aide d’un modèle Azure Resource Manager. Vous allez d’abord créer le modèle à partir de JSON brut, puis vous allez le créer à l’aide du langage dédié Bicep.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement utilisé, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git : Cloner** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer des ressources Azure Cosmos DB for NoSQL à l’aide de modèles Azure Resource Manager

Le fournisseur de ressources **Microsoft.DocumentDB** dans Azure Resource Manager permet de déployer des comptes, des bases de données et des conteneurs à l’aide de fichiers JSON. Bien que les fichiers soient complexes, ils suivent un format prévisible et peuvent être écrits à l’aide d’une extension Visual Studio Code.

> &#128161; Si vous êtes bloqué et que vous ne pouvez pas déterminer une erreur de syntaxe avec votre modèle, utilisez ce [modèle de solution Azure Resource Manager][github.com/arm-template-guide] comme guide.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **31-create-container-arm-template**.

1. Ouvrez le fichier **deploy.json**.

1. Observez le modèle Azure Resource Manager vide :

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. Dans le tableau **ressources**, ajoutez un nouvel objet JSON pour créer un compte Azure Cosmos DB :

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    L’objet est configuré avec les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Type de ressource** | *Microsoft.DocumentDB/databaseAccounts* |
    | **Version d’API** | *2021-05-15* |
    | **Nom du compte** | *csmsarm* &amp; *chaîne unique générée à partir du nom du compte*  |
    | **Lieu** | *Emplacement actuel du groupe de ressources* |
    | **Type d’offre de compte** | *Standard* |
    | **Emplacements** | *USA Ouest uniquement* |

1. Enregistrez le fichier **deploy.json**.

1. Ouvrez le menu contextuel du dossier **31-create-container-arm-template**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **31-create-container-arm-template**.

1. Installez les certificats tls/ssl avant de vous connecter à Azure :

    ```
    $CurrentDirectory=$pwd
    CD "C:\Program Files\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    CD $CurrentDirectory
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

1. Créez un nom de variable **resourceGroup** en utilisant le nom du groupe de ressources que vous avez créé ou affiché précédemment dans ce labo à l’aide de la commande suivante :

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Par exemple, si votre groupe de ressources est nommé **dp420**, la commande est **$resourceGroup="dp420"**.

1. Utilisez la cmdlet **echo** pour écrire la valeur de la variable **$resourceGroup** dans la sortie du terminal à l’aide de la commande suivante :

    ```
    echo $resourceGroup
    ```

1. Déployez le modèle Azure Resource Manager à l’aide de la commande [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] :

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Laissez le terminal intégré ouvert et revenez à l’éditeur pour le fichier **deploy.json**.

1. Dans le tableau **ressources**, ajoutez un autre objet JSON pour créer une base de données Azure Cosmos DB for NoSQL :

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    L’objet est configuré avec les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Type de ressource** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Version d’API** | *2021-05-15* |
    | **Nom du compte** | *csmsarm* &amp; *chaîne unique générée à partir du nom de compte* &amp; */cosmicworks*  |
    | **ID de ressource** | *cosmicworks* |
    | **Dépendances** | *databaseAccount créé précédemment dans le modèle* |

1. Enregistrez le fichier **deploy.json**.

1. Revenez au terminal intégré.

1. Déployez le modèle Azure Resource Manager à l’aide de la commande **az deployment group create** :

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Laissez le terminal intégré ouvert et revenez à l’éditeur pour le fichier **deploy.json**.

1. Dans le tableau **ressources**, ajoutez un autre objet JSON pour créer un conteneur Azure Cosmos DB for NoSQL :

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    L’objet est configuré avec les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Type de ressource** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **Version d’API** | *2021-05-15* |
    | **Nom du compte** | *csmsarm* &amp; *chaîne unique générée à partir du nom de compte* &amp; */cosmicworks/products*  |
    | **ID de ressource** | *produits* |
    | **Débit** | *400* |
    | **Clé de partition** | */categoryId* |
    | **Dépendances** | *Compte et base de données créés précédemment dans le modèle* |

1. Enregistrez le fichier **deploy.json**.

1. Revenez au terminal intégré.

1. Déployez le modèle Azure Resource Manager final à l’aide de la commande **az deployment group create** :

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Fermez le terminal intégré.

## Observer les ressources Azure Cosmos DB déployées

Une fois que vos ressources Azure Cosmos DB for NoSQL sont déployées, vous pouvez accéder aux ressources dans le portail Azure. À l’aide de l’Explorateur de données, vous allez vérifier que le compte, la base de données et le conteneur ont tous été déployés et configurés correctement.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **Groupes de ressources**, sélectionnez le groupe de ressources que vous avez créé ou affiché précédemment dans ce labo, puis sélectionnez la ressource **Compte Azure Cosmos DB** que vous avez créée dans ce labo avec le préfixe **csmsarm**.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NoSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Mise à l’échelle et paramètres**.

1. Observez les valeurs de la section **Mise à l'échelle**. Notez en particulier que l’option **Manuel** est sélectionnée dans la section **Débit** et que le débit approvisionné est défini sur **400** RU/s.

1. Observez les valeurs de la section **Paramètres**. Plus précisément, observez que la valeur de la **Clé de partition** est définie sur **/categoryId**.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

## Créer des ressources Azure Cosmos DB for NoSQL à l’aide de modèles Bicep

Bicep est un langage dédié efficace qui rend le déploiement de ressources Azure plus simple et plus facile que les modèles Azure Resource Manager. Vous allez déployer la même ressource à l’aide de Bicep et d’un autre nom pour illustrer la ou les différence\[s\].

> &#128161; Si vous êtes bloqué et que vous ne pouvez pas déterminer une erreur de syntaxe avec votre modèle, utilisez ce [modèle de solution Bicep][github.com/bicep-template-guide] comme guide.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **31-create-container-arm-template**.

1. Ouvrez le fichier **deploy.bicep** vide.

1. Dans le fichier, ajoutez un nouvel objet pour créer un compte Azure Cosmos DB :

    ```
    param location string = resourceGroup().location
    
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    L’objet est configuré avec les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Alias** | *Compte* |
    | **Nom** | *csmsarm* &amp; *chaîne unique générée à partir du nom du compte* |
    | **Type de ressource** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Version d’API** | *2021-05-15* |
    | **Lieu** | *Emplacement actuel du groupe de ressources* |
    | **Type d’offre de compte** | *Standard* |
    | **Emplacements** | *USA Ouest uniquement* |

1. Enregistrez le fichier **deploy.bicep**.

1. Ouvrez le menu contextuel du dossier **31-create-container-arm-template**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

1. Créez un nom de variable **resourceGroup** en utilisant le nom du groupe de ressources que vous avez créé ou affiché précédemment dans ce labo à l’aide de la commande suivante :

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Par exemple, si votre groupe de ressources est nommé **dp420**, la commande est **$resourceGroup="dp420"**.

1. Déployez le modèle Bicep à l’aide de la commande **az deployment group create** :

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Laissez le terminal intégré ouvert et revenez à l’éditeur pour le fichier **deploy.bicep**.

1. Dans le fichier, ajoutez un autre objet pour créer une base de données Azure Cosmos DB :

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    L’objet est configuré avec les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Parent** | *Compte créé précédemment dans le modèle* |
    | **Alias** | *Sauvegarde de la base de données* |
    | **Nom** | *cosmicworks*  |
    | **Type de ressource** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **Version d’API** | *2021-05-15* |
    | **ID de ressource** | *cosmicworks* |

1. Enregistrez le fichier **deploy.bicep**.

1. Revenez au terminal intégré.

1. Déployez le modèle Bicep à l’aide de la commande **az deployment group create** :

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Laissez le terminal intégré ouvert et revenez à l’éditeur pour le fichier **deploy.bicep**.

1. Dans le fichier, ajoutez un autre objet pour créer un conteneur Azure Cosmos DB :

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    L’objet est configuré avec les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Parent** | *Base de données créée précédemment dans le modèle* |
    | **Alias** | *Conteneur* |
    | **Nom** | *produits*  |
    | **ID de ressource** | *produits* |
    | **Débit** | *400* |
    | **Chemin d’accès de clé de partition** | */categoryId* |

1. Enregistrez le fichier **deploy.bicep**.

1. Revenez au terminal intégré.

1. Déployez le modèle Bicep final à l’aide de la commande **az deployment group create** :

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Fermez le terminal intégré.

1. Fermez **Visual Studio Code**.

## Observer les résultats du déploiement du modèle Bicep

Les déploiements Bicep peuvent être validés en utilisant la plupart des mêmes techniques que les déploiements Azure Resource Manager. Non seulement vous allez vérifier que votre compte, votre base de données et votre conteneur ont été correctement déployés, mais vous allez également afficher l’historique des déploiements sur les six déploiements.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **groupes de ressources**, puis sélectionnez le groupe de ressources que vous avez créé ou consulté précédemment dans ce labo.

1. Dans le groupe de ressources, accédez au volet **Déploiements**.

1. Observez les six déploiements à partir des modèles Azure Resource Manager et des fichiers Bicep.

1. Toujours dans le groupe de ressources, accédez au volet **Vue d’ensemble**.

1. Toujours dans le groupe de ressources, sélectionnez la ressource du **compte Azure Cosmos DB** que vous avez créée dans ce labo avec le préfixe **csmsbicep**.

1. Dans la ressource de compte **Azure Cosmos DB**, accédez au volet **Explorateur de données**.

1. Dans l’**Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nouveau nœud de conteneur **products** dans l’arborescence de navigation de l’**API NoSQL**.

1. Sélectionnez le nœud de conteneur **products** dans l’arborescence de navigation de l’**API NOSQL**, puis sélectionnez **Mise à l’échelle et paramètres**.

1. Observez les valeurs de la section **Mise à l'échelle**. Notez en particulier que l’option **Manuel** est sélectionnée dans la section **Débit** et que le débit approvisionné est défini sur **400** RU/s.

1. Observez les valeurs de la section **Paramètres**. Plus précisément, observez que la valeur de la **Clé de partition** est définie sur **/categoryId**.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
