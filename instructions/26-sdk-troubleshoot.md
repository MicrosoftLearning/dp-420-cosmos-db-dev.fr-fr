---
lab:
  title: Résoudre les problèmes d’une application en utilisant le SDK Azure Cosmos DB for NoSQL
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Résoudre les problèmes d’une application en utilisant le SDK Azure Cosmos DB for NoSQL

Azure Cosmos DB offre un ensemble complet de codes de réponse qui nous aident à résoudre facilement les problèmes susceptibles de se produire avec nos différents types d’opérations. L’idée est de veiller à programmer une gestion appropriée des erreurs quand nous créons des applications pour Azure Cosmos DB.

Dans ce labo, nous créons un programme piloté par un menu qui nous permet d’insérer ou de supprimer un des deux documents. L’objectif principal de ce labo est de présenter certains codes de réponse les plus courants et de montrer comment les intégrer dans le code de gestion des erreurs de notre application.  Bien que nous codions la gestion des erreurs pour plusieurs codes de réponse, nous déclenchons seulement deux types de conditions différents.  Par ailleurs, la gestion des erreurs ne fait rien de complexe, selon le code de réponse, elle affiche un message à l’écran ou attend 10 secondes avant de réessayer l’opération.

## Préparer votre environnement de développement

Si vous n’avez pas encore cloné le référentiel de code du labo pour le cours **DP-420** dans l’environnement où vous travaillez sur ce labo, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git : Cloner** pour cloner le référentiel GitHub ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Une fois le compte Azure Cosmos DB for NoSQL provisionné, vous pouvez récupérer le point de terminaison et la clé. Utilisez le point de terminaison et la clé pour vous connecter programmatiquement au compte Azure Cosmos DB for NoSQL. Utilisez le point de terminaison et la clé sur les chaînes de connexion d’Azure SDK pour .NET ou un autre SDK.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au Portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Sélectionnez **+ Créer une ressource**, recherchez *Cosmos DB*, puis créez une ressource de compte **Azure Cosmos DB for NoSQL** avec les paramètres suivants, en conservant les valeurs par défaut de tous les autres paramètres :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Type de charge de travail** | **Formations** |
    | **Abonnement** | *Votre abonnement Azure existant* |
    | **Groupe de ressources** | *Sélectionner un groupe de ressources existant ou en créer un* |
    | **Nom du compte** | *Entrez un nom globalement unique* |
    | **Lieu** | *Choisissez une région disponible* |
    | **Mode de capacité** | *Débit approvisionné* |
    | **Appliquer la remise de niveau Gratuit** | *Ne pas appliquer* |

    > &#128221; Vos environnements de labo peuvent présenter des restrictions vous empêchant de créer un groupe de ressources. Si tel est le cas, utilisez le groupe de ressources existant précréé.

1. Attendez la fin de la tâche de déploiement avant de passer à cette tâche.

1. Accédez à la ressource de compte **Azure Cosmos DB** qui vient d’être créée, puis accédez au volet **Clés**.

1. Ce volet contient les détails de connexion et les informations d’identification nécessaires pour se connecter au compte à partir du kit SDK. Plus précisément :

    1. Notez le champ **URI**. Vous utilisez cette valeur de **point de terminaison** plus tard dans cet exercice.

    1. Notez le champ **CLÉ PRIMAIRE**. Vous utilisez cette valeur de **clé** plus tard dans cet exercice.

1. Réduisez la fenêtre de votre navigateur, mais ne la fermez pas. Nous revenons au portail Azure peu après avoir démarré une charge de travail en arrière-plan dans les étapes suivantes.


## Importer la bibliothèque Microsoft.Azure.Cosmos dans un script .NET

L’interface CLI .NET comprend une commande [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] pour importer des packages à partir d’un flux de package préconfiguré. Une installation .NET utilise NuGet comme flux de package par défaut.

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **26-sdk-troubleshoot**.

1. Ouvrez le menu contextuel du dossier **26-sdk-troubleshoot**, puis sélectionnez **Ouvrir dans le terminal intégré** pour ouvrir une nouvelle instance de terminal.

    > &#128221; Cette commande ouvre le terminal avec le répertoire de démarrage déjà défini sur le dossier **26-sdk-troubleshoot**.

1. Ajoutez le package [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] à partir de NuGet en utilisant la commande suivante :

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

## Exécutez un script pour créer des options pilotées par un menu pour insérer et supprimer des documents.

Avant de pouvoir exécuter notre application, nous devons la connecter à notre compte Azure Cosmos DB. 

1. Dans **Visual Studio Code**, dans le volet **Explorateur**, accédez au dossier **26-sdk-troubleshoot**.

1. Ouvrez le fichier de code **Program.cs**.

1. Mettez à jour la variable existante nommée **endpoint** avec sa valeur définie sur le **point de terminaison** du compte Azure Cosmos DB que vous avez créé précédemment.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Par exemple, si votre point de terminaison est : **https&shy;://dp420.documents.azure.com:443/**, l’instruction C# sera : **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Mettez à jour la variable existante nommée **key** avec sa valeur définie sur la **clé** du compte Azure Cosmos DB que vous avez créé précédemment.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Par exemple, si votre clé est : **fDR2ci9QgkdkvERTQ==**, l’instruction C# est : **private static readonly string key = "fDR2ci9QgkdkvERTQ==";**.

1. Enregistrez le fichier.

1. Créez et exécutez le projet avec la commande [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] :

    ```
    dotnet run
    ```
    > &#128221; C’est un programme très simple.  Il affiche un menu avec cinq options, comme indiqué ci-dessous. Deux options pour insérer un document prédéfini, deux pour supprimer un document prédéfini et une pour quitter le programme.

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## Insertion et suppression de documents.

1. Sélectionnez **1** et **Entrée** pour insérer le premier document. Le programme insère le premier document et renvoie le message suivant.

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. Sélectionnez encore **1** et **Entrée** pour insérer le premier document. Cette fois, le programme se bloque avec une exception. En examinant la pile d’erreurs, nous pouvons déterminer la raison de l’échec du programme. D’après le message extrait de la pile d’erreurs, nous avons une exception non prise en charge « Conflit (409) »

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. Comme nous insérons un document, nous devons passer en revue la liste des [codes d’état de création de document][/rest/api/cosmos-db/create-a-document#status-codes] courants renvoyés quand un document est créé. La description de ce code est : *L’ID fourni pour le nouveau document a été pris par un document existant*. C’est sans surprise, car nous venons d’exécuter l’option de menu pour créer le même document il y a quelques instants.

1. Plus loin dans la pile, nous pouvons voir que cette exception a été appelée à partir de la ligne 100, à son tour appelée à partir de la ligne 64.

    ```
    at Program.CreateDocument1(Container Customer) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 100   
   at Program.CompleteTaskOnCosmosDB(String consoleinputcharacter, Container container) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 64
    ```

1. D’après la ligne 100, comme prévu, l’erreur a été provoquée par l’opération *CreateItemAsync*. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. Par ailleurs, en examinant les lignes 100 à 103, il est évident que ce code n’a pas de gestion des erreurs. Nous devons y remédier. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. Nous devons déterminer ce que doit faire notre code de gestion des erreurs. En examinant les [codes d’état de création de document][/rest/api/cosmos-db/create-a-document#status-codes], nous pouvons choisir de créer un code de gestion des erreurs pour chaque code d’état possible de cette opération.  Dans ce labo, nous prenons en compte seulement les codes de cette liste, c’est-à-dire les codes d’état 403 et 409.  Tous les autres codes d’état renvoyés affichent simplement le message d’erreur système.

    > &#128221; Notez que bien que nous codions une tâche de gestion des erreurs pour une exception 403, dans ce labo, nous ne générons pas d’exception 403.

1. Ajoutons la gestion des erreurs pour la fonction nommée **CompleteTaskOnCosmosDB**. Recherchez la boucle **while** dans la fonction **Main** sur la ligne **45** et enveloppez les appels de **CompleteTaskOnCosmosDB** avec un code de gestion des erreurs. Nous remplaçons l’instruction **CompleteTaskOnCosmosDB** de la ligne **47** par le code ci-dessous.  La première chose à noter dans ce nouveau code est que, sur l’instruction **catch** nous capturons une exception avec le type de classe **CosmosException**.  Cette classe inclut la propriété **StatusCode**, qui renvoie le code d’état de complétion de la demande à partir du service Azure Cosmos DB. La propriété **StatusCode** est de type **System.Net.HttpStatusCode**, nous pouvons utiliser cette valeur et la comparer aux noms de champs du [Code d’état HTTP][dotnet/api/system.net.httpstatuscode] .NET.  

    ```C#
        try
        {
            await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. Enregistrez le fichier et, comme nous avons été bloqués et que nous devons réexécuter notre programme de menu, exécutez la commande :

    ```
    dotnet run
    ```
 
1. Sélectionnez encore **1** et **Entrée** pour insérer le premier document. Cette fois, nous ne sommes pas bloqués et nous obtenons un message plus convivial décrivant ce qui s’est passé.

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    ```

1. Ce code a ajouté la gestion des erreurs pour les exceptions *403* et *409*, ajoutons maintenant du code pour certains types d’exception de communication courants. Il existe trois types d’exception de communication courants : *429*, *503* et *408* (trop de demandes, service indisponible et expiration de la demande, respectivement). Vers la ligne *66*, il doit maintenant y avoir une instruction **default**. Ajoutez le code ci-dessous juste après l’instruction **break;** et juste avant l’instruction **default**.  Le code vérifie si nous avons une de ces exceptions de communication et, le cas échéant, attendez 10 secondes, puis réessayez d’insérer le document.  Voyons le code :

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; Notez que bien que nous codions une tâche indiquant ce qu’il faut faire en cas d’exception 429, 503 ou 408, dans ce labo, nous ne générons pas d’erreur avec ce type d’exception.

1. Notre fonction **Main** doit ressembler à ceci :

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                 try
                 {
                     await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                 }
                 catch (CosmosException e)
                 {
                     switch (e.StatusCode.ToString())
                     {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                     }
                }
                

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. Notez que la fonction **CreateDocument2** est également corrigée par les changements ci-dessus.

1. Enfin, les fonctions **DeleteDocument1** et **DeleteDocument2** ont également besoin que le code suivant soit remplacé par le code approprié de gestion des erreurs similaire à la fonction **CreateDocument1**. La seule différence avec ces fonctions, hormis utiliser **DeleteItemAsync** au lieu de **CreateItemAsync**, est que les [codes d’état de suppression][/rest/api/cosmos-db/delete-a-document] sont différents des codes d’état d’insertion. Pour les suppressions, nous regardons seulement le code d’état **404**, qui désigne un document introuvable. Mettons à jour la gestion des erreurs de l’appel de fonction **CompleteTaskOnCosmosDB** avec une instruction case supplémentaire.  Dans la fonction **Main**, le code suivant doit être ajouté au-dessus de l’instruction case **default** :

    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1. Enregistrez le fichier.

1. Dès que vous avez fini de corriger toutes les fonctions, testez plusieurs fois toutes les options de menu pour vérifier que votre application renvoie un message en cas d’exception et qu’elle ne se bloque pas.  Si votre application se bloque, corrigez les erreurs et réexécutez simplement la commande :

    ```
    dotnet run
    ```


1. Ne regardez pas, mais dès que vous avez terminé, les codes de votre fonction `Main` doivent ressembler à ceci.

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## Conclusion

Même les développeurs les plus juniors savent qu’une gestion des erreurs appropriée doit être ajoutée à l’ensemble du code. Bien que la gestion des erreurs dans ce code soit simple, vous connaissez maintenant les bases des composants d’exception Azure Cosmos DB pour créer des solutions de gestion des erreurs robustes dans votre code.


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

