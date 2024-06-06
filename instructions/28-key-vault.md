---
lab:
  title: Stocker des clés de compte Azure Cosmos DB for NoSQL dans Azure Key Vault
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Stocker des clés de compte Azure Cosmos DB for NoSQL dans Azure Key Vault

Pour ajouter un code de connexion à un compte Azure Cosmos DB à votre application, il suffit de fournir l’URI et les clés du compte. Ces informations de sécurité peuvent parfois être codées en dur dans le code de l’application. Toutefois, si votre application est déployée sur Azure App Service, vous pouvez enregistrer les informations de connexion chiffrées dans Azure Key Vault.

Dans ce labo, nous allons chiffrer et stocker la chaîne de connexion à un compte Azure Cosmos DB dans Azure Key Vault. Nous créerons ensuite une application web Azure App Service qui récupérera ces informations d’identification à partir d’Azure Key Vault. L’application utilisera ces informations d’identification et se connectera au compte Azure Cosmos DB. L’application créera ensuite des documents dans les conteneurs du compte Azure Cosmos DB et retournera son état à une page web.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement où vous travaillez sur ce labo, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Ouvrez la palette de commandes et exécutez **Git: Clone** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl+Maj+P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, sélectionnez ***FERMER*** pour fermer *Visual Studio Code*. Nous l’ouvrirons plus tard en pointant directement vers le dossier **28-key-vault**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Une fois le compte Azure Cosmos DB for NoSQL approvisionné, vous pouvez récupérer le point de terminaison et la clé. Utilisez le point de terminaison et la clé pour vous connecter par programmation au compte Azure Cosmos DB for NoSQL. Utilisez le point de terminaison et la clé sur les chaînes de connexion du Kit de développement logiciel (SDK) Azure pour .NET ou un autre SDK.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au Portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du SDK. Notez le champ **CHAÎNE DE CONNEXION PRINCIPALE**. Vous utiliserez la valeur de cette **chaîne de connexion** plus loin dans cet exercice.

## Créer un coffre de clés Azure et stocker les informations d’identification du compte Azure Cosmos DB en tant que secret

Avant de créer notre application web, nous allons sécuriser la chaîne de connexion au compte Azure Cosmos DB en la copiant dans un *secret* chiffré par *Azure Key Vault*. Nous allons le faire maintenant.

1. Sous un nouvel onglet du navigateur, accédez au Portail Azure et ouvrez la page **Coffres de clés**.

1. Ajoutez un coffre en sélectionnant le bouton ***+ Créer***, remplissez le coffre avec les paramètres suivants, *en conservant les valeurs par défaut de tous les autres paramètres*, puis créez le coffre :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du coffre de clés** | *Entrez un nom globalement unique* |
    | **Région** | *Choisissez une région disponible* |
    | **Configuration de l’accès/Modèle d’autorisation** | *Stratégie d’accès au coffre* |
    | **Configuration de l’accès/Stratégies d’accès** | *Cocher la case du nom d’utilisateur actuel* |

    > &#128221; Dans un environnement de production, notez que vous sélectionnerez probablement le contrôle RBAC au lieu de la stratégie d’accès au coffre, et que votre administrateur vous attribuera probablement le rôle RBAC approprié pour limiter votre accès au coffre de clés.

1. Accédez au coffre une fois celui-ci créé.

1. Sous la section *Objets*, sélectionnez **Secrets**.

1. Sélectionnez **+ Générer/Importer** pour chiffrer notre chaîne de connexion d’informations d’identification. Renseignez les valeurs du *secret* avec les paramètres suivants, *en conservant les valeurs par défaut de tous les autres paramètres*, puis créez le secret :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Options de chargement** | *Manuel* |
    | **Nom** | *Nom utilisé pour étiqueter votre secret* |
    | **Valeur** | *Ce champ est le plus important. Copiez ici la valeur du champ CHAÎNE DE CONNEXION PRINCIPALE issu de la section des clés de votre compte Azure Cosmos DB. Cette valeur sera convertie en secret.* |
    | **Activé** | *Oui* |
 
1. Votre nouveau secret doit désormais apparaître sous Secrets. Nous devons obtenir l’*identificateur de secret* que nous ajouterons au code de notre application web. Sélectionnez le **secret** que vous avez créé.

1. Azure Key Vault vous permet de créer plusieurs versions de votre secret. Toutefois, dans ce labo, nous n’avons besoin que d’une seule version. Sélectionnez **Version actuelle**.

1. Notez la valeur du champ **Identificateur de secret**. Nous utiliserons cette valeur dans le code de notre application pour obtenir le secret du coffre de clés.  Notez que cette valeur est une URL. Il nous reste encore une étape à effectuer pour que ce secret fonctionne correctement, mais nous nous en occuperons un peu plus tard.

## Créer une application web Azure App Service

Nous allons créer une application web qui se connectera au compte Azure Cosmos DB et qui créera des conteneurs et des documents. Nous ne coderons pas en dur les *informations d’identification* Azure Cosmos DB dans cette application. Au lieu de cela, nous coderons en dur l’**Identificateur de secret** du coffre de clés. Nous verrons que cet identificateur est inutile sans les droits appropriés attribués à l’application web sur la couche Azure. Commençons à coder.



1. Ouvrez **Visual Studio Code**.  Ouvrez le dossier **28-key-vault**. Pour cela, sélectionnez Fichier->Ouvrir le dossier, puis accédez au dossier **28-key-vault**.

    > &#128221; Notez que vous ne devez voir que le dossier **28-key-vault** et ses fichiers et sous-dossiers dans l’arborescence de l’**Explorateur**. Si vous pouvez voir l’ensemble du référentiel GitHub que nous avons cloné précédemment, ***fermez Visual Studio Code*** et rouvrez-le directement dans le dossier **28-key-vault**.  L’application web ne fonctionnera pas correctement si ce répertoire n’est pas votre répertoire racine de projets. Vérifiez donc que vous ne pouvez voir que le dossier **28-key-vault** et ses fichiers et sous-dossiers dans l’arborescence de l’**Explorateur**.

1. Ouvrez le menu contextuel du dossier **28-key-vault**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **28-key-vault**.

1. Créons à présent un interpréteur de commandes d’application web MVC. Nous allons remplacer quelques-uns des fichiers générés dans un instant. Exécutez la commande suivante pour créer l’application web :

    ```
    dotnet new mvc
    ```


    > &#128221;Cette commande a créé l’interpréteur de commandes d’une application web et a donc ajouté plusieurs fichiers et répertoires. Nous avons déjà quelques fichiers avec tout le code dont nous avons besoin. 

1. Remplacez les fichiers **.\Controllers\HomeController.cs** et **.\Views\Home\Index.cshtml** par leurs fichiers respectifs du répertoire **.\KeyvaultFiles**.

1. Une fois les fichiers remplacés, sélectionnez ***SUPPRIMER*** pour supprimer le répertoire **.\KeyvaultFiles**.

## Importer les bibliothèques manquantes dans le script .NET

L’interface CLI .NET comprend une commande [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] pour importer des packages à partir d’un flux de package préconfiguré. Une installation .NET utilise NuGet comme flux de package par défaut.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Ajoutez le package [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. Ajoutez le package [Microsoft.Azure.KeyVault][nuget.org/packages/Microsoft.Azure.KeyVault] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. Ajoutez le package [Microsoft.Azure.Services.AppAuthentication][nuget.org/packages/Microsoft.Azure.Services.AppAuthentication] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.6.2
    ```

## Ajout de l’identificateur de secret à votre application web

1. Dans Visual Studio, ouvrez le fichier `.\Controllers\HomeControler.cs`.

1. La fonction définie par l’utilisateur **GetKeyVaultSecret** obtient le secret du compte Azure Cosmos DB. La fonction, qui commence à la *ligne 98*, doit ressembler au script ci-dessous.

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. Passons en revue les appels importants effectués par cette fonction.

    - À la *ligne 100*, nous définissons le jeton de l’application web actuelle. Ce jeton est fourni à Azure Key Vault pour identifier l’application qui tente d’accéder au coffre. 
    - Dans les *lignes 104-105*, nous préparons le *client Key Vault* qui se connectera à Azure Key Vault. Notez que nous envoyons le jeton de l’application web en tant que paramètre. 
    - Dans les *lignes 107-108*, nous fournissons au client Key Vault l’adresse URL de notre **identificateur de secret**, qui retourne le secret stocké dans ce coffre de clés. 

1.  Avant de pouvoir déployer notre application web, nous devons toujours envoyer l’URL de l’**identificateur de secret**.  À la *ligne 107*, remplacez la chaîne ***<Key Vault Secret Identifier>*** par l’URL de l’**identificateur de secret** que nous avons enregistrée dans la section des *secrets* et enregistrez le fichier.

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## (Facultatif) Installer l’extension Azure App Service

Dans Visual Studio, si vous affichez la palette de commandes (**Ctrl+Maj+P**) et que la recherche de commandes Azure App Service ne retourne rien, nous devons installer l’extension.

1. Dans le menu de gauche de Visual Studio Code, sélectionnez l’option **Extensions**.

1. Dans la barre de recherche, recherchez Azure App Service et sélectionnez-le.

1. Sélectionnez le bouton Installer pour procéder à son installation.

1. Fermez l’onglet **Extensions** et revenez à votre code.

## Déployer votre application sur Azure App Service

Le reste du code est simple : obtenir la chaîne de connexion, se connecter à Azure Cosmos DB et ajouter des documents. L’application doit également nous fournir des commentaires sur les éventuels problèmes. Nous ne devrions plus avoir besoin d’apporter de modifications une fois l’application déployée. C’est parti. 

> &#128221; La plupart des étapes ci-dessous sont exécutées dans la palette de commandes (**Ctrl+Maj+P**), dans la partie supérieure de votre écran Visual Studio. 

1. Dans Visual Studio Code, ouvrez la palette de commandes, recherchez ***Azure App Service : Créer une application web... (avancé)***.

1. Sélectionnez ***Connexion à Azure...***. Cette option ouvre une fenêtre dans le navigateur web, suit le processus de connexion, ferme le navigateur une fois l’opération terminée et revient à Visual Studio Code.

1. (Facultatif) Si vous êtes invité à indiquer votre abonnement, sélectionnez votre abonnement.

1. Entrez un nom globalement unique pour votre application web.

1. Sélectionnez un groupe de ressources existant ou créez-en un si nécessaire.

1. Sélectionnez **.NET 6 (LTS)**.

1. Sélectionnez **Windows**.

1. Sélectionnez un Emplacement disponible.

1. Sélectionnez **+ Créer un plan App Service**.

1. Acceptez le nom par défaut du plan App Service (il doit être identique au nom de votre application web) ou choisissez un nouveau nom.

1. Sélectionnez **Gratuit (F1) Essayer Azure gratuitement**.

1. Pour la ressource Application Insights, sélectionnez **Ignorer pour le moment**.

1. Le déploiement doit maintenant s’exécuter avec une barre d’état dans le coin inférieur droit. 

1. Lorsque vous y êtes invité, sélectionnez **Déployer**.

1. Sélectionnez **Parcourir**. Vous devriez vous trouver dans le dossier **28-key-vault**. Sélectionnez ce dossier.

1. Une fenêtre contextuelle avec le message **La configuration requise pour le déploiement est manquante dans « 28-key-vault »** doit s’afficher. Sélectionnez le bouton Ajouter une configuration.  Cette option crée le dossier `.vscode` manquant.

    > &#128221; Remarque très importante : si cette fenêtre contextuelle n’apparaît pas lors de votre premier déploiement de l’application, le chargement sur Azure App Service ne contient pas tous les fichiers. Le déploiement réussit, mais le site web retourne toujours le message *Vous n’êtes pas autorisé à afficher ce répertoire ou cette page*. Le problème vient probablement du fait que Visual Studio Code a été ouvert sur le référentiel cloné GitHub à la place du dossier **28-key-vault**.

1. Sélectionnez **Oui** lorsque vous êtes invité à toujours déployer sur cet espace de travail.

1. Sélectionnez **Parcourir le site web** lorsque vous y êtes invité.  Vous pouvez également ouvrir un navigateur et accéder à **`https://<yourwebappname>.azurewebsites.net`**. Dans les deux cas, nous avons un problème. Vous voyez un message défini par l’utilisateur sur notre page web. Vous devriez voir le message **Key Vault n’était pas accessible** avec un message d’erreur étendu. Nous allons arranger ça.

## Autoriser notre application à utiliser une identité managée

La première chose que nous devons faire est d’autoriser notre application d’utiliser une identité managée. L’utilisation d’une identité managée permet à notre application d’utiliser des services Azure comme Azure Key Vault.

1. Ouvrez votre navigateur et connectez-vous au Portail Azure.

1. Ouvrez la page **App Services**. Le nom de votre application web doit être répertorié. Sélectionnez-le.

1. Sous la section *Paramètres*, sélectionnez **Identité**.

1. Sous État, sélectionnez **Activé** et **Enregistrer**.  Sélectionnez **Oui** si vous êtes invité à activer l’*Identité managée attribuée*.

1. Réessayons notre application web.  Dans votre navigateur, accédez à **`https://<yourwebappname>.azurewebsites.net`**.

1. Il y a encore un problème. Alors que le premier message est un message défini par l’utilisateur et envoyé par notre programme, le deuxième message est généré par le système. Le deuxième message signifie que nous sommes autorisés à nous connecter au coffre de clés, mais que nous ne sommes pas autorisés à afficher le secret à l’intérieur du coffre.  Pour résoudre ce problème, définissons un dernier paramètre.

## Octroi d’une stratégie d’accès aux secrets Key Vault à notre application web

L’objectif de départ de ce labo était d’empêcher le codage en dur de nos comptes Azure Cosmos DB dans nos applications. Toutefois, nous avons codé en dur l’URL de notre **identificateur de secret** que tout le monde peut voir. Alors, comment pouvons-nous sécuriser nos informations d’identification ? La bonne nouvelle est que l’identificateur de secret en lui-même est inutile. L’**identificateur de secret** vous permet uniquement d’arriver à la porte du coffre de clés Azure, mais c’est le coffre qui décide qui entre et qui n’entre pas. Cela signifie que nous devons créer une stratégie d’accès au coffre de clés pour notre application afin qu’elle puisse voir les secrets dans ce coffre. Examinons cette solution.

1. (Facultatif) Avant de créer la stratégie, passons en revue le contenu actuel de notre base de données Azure Cosmos DB.  Dans le Portail Azure, accédez à votre compte Azure Cosmos DB. Existe-t-il une base de données **GlobalCustomers** ? Si ce n’est pas le cas, elle est créée lors de l’exécution réussie de l’application web. Si c’est le cas, passez en revue le nombre d’éléments de la base de données. L’exécution réussie de l’application web ajoute d’autres éléments.

1. Dans le Portail Azure, accédez au coffre de clés créé précédemment.

1. Sous la section *Paramètres*, sélectionnez **Configuration de l’accès**.

1. Vérifiez que **Stratégie d’accès au coffre** est sélectionnée, puis sélectionnez **Accéder aux stratégies d’accès**.

1. Sélectionnez **+ Créer**.

1. Sous l’onglet **Autorisations**, cochez la case **Obtenir** pour les **Autorisations de clé** et **Autorisations du secret**, puis sélectionnez **Suivant**.

1. Dans l’onglet **Principal**, dans la zone de recherche, entrez le nom que vous avez donné à votre App Service, sélectionnez-le dans la liste, puis sélectionnez **Suivant**.

1. Dans l’onglet **Application (facultatif)**, sélectionnez **Suivant**.
    
1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.

1. Réessayons notre application web.  Dans votre navigateur, accédez à **`https://<yourwebappname>.azurewebsites.net`**.

1. Opération réussie. Notre page web doit indiquer que nous avons inséré de nouveaux éléments dans le conteneur du client. Nous pouvons également voir le secret réel affiché.

    > &#128221; Dans un environnement de production, **n’affichez jamais** le secret. Nous l’avons fait ici uniquement à titre d’illustration.


1. Accédez à votre compte Azure Cosmos DB et vérifiez que vous disposez d’une nouvelle base de données **GlobalCustomers** contenant des données ou, si la base de données existait déjà, qu’elle contient désormais plus d’éléments.

Nous venons d’utiliser Azure Key Vault pour protéger les clés de votre compte Azure Cosmos DB.
