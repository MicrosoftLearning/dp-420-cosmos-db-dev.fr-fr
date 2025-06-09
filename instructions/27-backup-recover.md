---
lab:
  title: Récupérer une base de données ou un conteneur à partir d’un point de récupération
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Récupérer une base de données ou un conteneur à partir d’un point de récupération 

Azure effectue automatiquement des sauvegardes chiffrées de vos données. Ces sauvegardes sont effectuées en deux modes, les modes de sauvegarde **Périodique** et **Continue**.

Dans ce labo, vous effectuez **une sauvegarde** et **des restaurations** à l’aide du mode de sauvegarde continue. Tout d’abord, vous créez un compte Azure Cosmos DB. Vous créez ensuite deux conteneurs et y ajouter quelques documents. Ensuite, vous mettez à jour quelques documents dans ces conteneurs. Enfin, vous créez des restaurations du compte à un point avant chaque suppression.

## Créer un compte Azure Cosmos DB for NoSQL

Azure Cosmos DB est un service de base de données NoSQL basé sur le cloud qui prend en charge plusieurs API. Quand vous approvisionnez un compte Azure Cosmos DB pour la première fois, vous sélectionnez les API que le compte doit prendre en charge (par exemple, l’**API Mongo** ou l’**API NoSQL**). Une fois le compte Azure Cosmos DB for NoSQL approvisionné, vous pouvez récupérer le point de terminaison et la clé. Utilisez le point de terminaison et la clé pour vous connecter par programmation au compte Azure Cosmos DB for NoSQL. Utilisez le point de terminaison et la clé sur les chaînes de connexion du Kit de développement logiciel (SDK) Azure pour .NET ou un autre SDK.

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
    | ONGLET **Distribution mondiale** | Désactiver les écritures multirégions |

    > &#128221; Notez que vous pouvez activer le mode **Continu** (7 jours) pendant la création du compte Azure Cosmos DB, en le sélectionnant sous l’onglet **Stratégie de sauvegarde**. Dans ce laboratoire, vous avez le choix d’activer cette fonctionnalité lors de la création du compte ou après la création du compte dans la section facultative ci-dessous. **Toutefois, l’activation de la fonctionnalité <ins>*après*</ins> la création du compte *peut prendre plus de 5 minutes*.**

    > &#128221; Notez que *[les comptes d’écriture multirégions ne sont actuellement pas pris en charge pour les sauvegardes continues][/azure/cosmos-db/continuous-backup-restore-introduction]*.

    > &#128221 ; vos environnements lab peuvent avoir des restrictions vous empêchant de créer un nouveau groupe de ressources. Si tel est le cas, utilisez le groupe de ressources pré-créé existant.

## Ajouter une base de données et deux conteneurs au compte

Nous allons créer une base de données et quelques conteneurs.

1. Dans le portail Microsoft Azure, accédez à la page de votre compte Azure Cosmos DB.

1. Sous **Explorateur de données**, ajoutez un nouveau conteneur avec les paramètres suivants

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **ID de base de données** | *Créez un nom* : *`Sales`* |
    | **Partager le débit entre les conteneurs** | *Ne pas sélectionner* |
    | **ID de conteneur** | *`customer`* |
    | **Clé de partition** | *`/id`* |
    | **Débit de conteneur (400 - RU/s illimité)** | Débit *manuel* : *400*|

1. Sous **Explorateur de données**, ajoutez un nouveau conteneur avec les paramètres suivants

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **ID de base de données** | *Utilisez un nom existant* : *Ventes* |
    | **ID de conteneur** | *`salesOrder`* |
    | **Clé de partition** | *`/id`* |
    | **Débit de conteneur (400 - RU/s illimité)** | Débit *manuel* : *400*|

## Ajouter des éléments aux conteneurs

Ajoutons des documents à ces conteneurs.

1. Dans le portail Microsoft Azure, accédez à la page de votre compte Azure Cosmos DB.

1. Sous **Explorateur de données**, ajoutez les deux documents suivants au conteneur **customer**.

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. Sous **Explorateur de données**, ajoutez les trois documents suivants au conteneur **salesOrder**.

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## Modifiez le mode de sauvegarde par défaut en continu (facultatif si la fonctionnalité n’est pas activée pendant la création du compte)

*Si vous n’avez pas activé la fonctionnalité pendant la création du compte Azure Cosmos DB, vous devez le faire maintenant.*  La modification du mode de sauvegarde est simple, tout ce qui est nécessaire consiste à modifier un paramètre sur **Activé**. Nous allons le modifier.

1. Dans le portail Microsoft Azure, accédez à la page de votre compte Azure Cosmos DB.

1. Dans la section **Paramètres**, sélectionnez **Sauvegarder et restaurer**.

1. Sélectionnez **Modifier** en regard du **mode Stratégie de sauvegarde**, dans l’écran, sélectionnez l’option **Continu (7 jours),** puis sélectionnez **Enregistrer**. ***L’activation de cette fonctionnalité peut prendre plus de cinq minutes***.

    > &#128221; Notez que *[les comptes d’écriture multirégions ne sont actuellement pas pris en charge pour les sauvegardes continues][/azure/cosmos-db/continuous-backup-restore-introduction]*. Si vous n’avez pas désactivé les écritures multirégions lorsque vous avez créé votre compte Azure Cosmos DB, vous devez le faire maintenant ou l’activation de la fonctionnalité de sauvegarde continue échoue.  Vous pouvez désactiver les écritures multirégions dans la section **Répliquer les données globalement** *Paramètres*.

## Supprimer l’un des documents salesOrder

1. Sous **Explorateur de données**, exécutez la requête suivante pour obtenir la date et l’heure actuelles. Copiez cet horodatage dans le Bloc-notes. Cet horodatage doit être au format UTC.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. Sous **Explorateur de données**, recherchez le document **salesOrder** avec l’**ID** `0019092E-BD25-48F5-8050-7051B2655BC5`. Supprimez le document, vérifiez que le document n’est plus présent.

## Restaurer la base de données au point précédant la suppression du document salesOrder

1. Dans le portail Microsoft Azure, accédez à la page de votre compte Azure Cosmos DB.

1. Dans la section *Paramètres*, sélectionnez **Restauration à un point dans le temps**. Utilisez les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Point de restauration (UTC)** | Convertissez la date et l’heure de manière appropriée. L’heure doit être au format AM/PM|
    | **Lieu** | *Sélectionner un emplacement disponible* |
    | **Sélectionner les ressources que vous souhaitez restaurer** | *Base de données/conteneurs sélectionnés* |
    | **Restaurer la ressource** | *salesOrder* |
    | **Restaurer le compte cible** | *choisir un* ***nouveau*** *nom de compte Azure Cosmos DB* |

    > &#128221; Pour les restaurations Azure Cosmos DB, vous ne restaurez ***jamais*** sur le compte *existant* et vous devrez toujours créer un compte Azure Cosmos DB.

    > &#128221; Bien que vous ayez pu choisir de restaurer l’ensemble de la base de données ou même le compte entier, dans un environnement de production réel, les bases de données peuvent être énormes. Dans de nombreux scénarios, il peut être plus rapide de restaurer simplement les conteneurs ou les bases de données nécessaires.

1. Cette restauration peut prendre 15 minutes ou plus, accédez à la section suivante et laissez cette restauration en cours d’exécution en arrière-plan.

## Supprimer le conteneur customer

1. Sous **Explorateur de données**, exécutez la requête suivante pour obtenir la date et l’heure actuelles. Copiez cet horodatage dans le Bloc-notes.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. Supprimez le conteneur **customer**.

## Restaurer la base de données au point précédant la suppression du document salesOrder

1. Dans le portail Microsoft Azure, accédez à la page de votre compte Azure Cosmos DB.

1. Dans la section *Paramètres*, sélectionnez **Restauration à un point dans le temps**. Utilisez les paramètres suivants :

    | **Paramètre** | **Valeur** |
    | ---: | :--- |
    | **Lieu** | *Sélectionner un emplacement disponible* |
    | **Point de restauration (UTC)** | Convertissez la date et l’heure de manière appropriée. L’heure doit être au format AM/PM|
    | **Sélectionner les ressources que vous souhaitez restaurer** | *Base de données/conteneurs sélectionnés* |
    | **Restaurer la ressource** | *`customer`* |
    | **Restaurer le compte cible** | *choisir un* ***nouveau*** *nom de compte Azure Cosmos DB* |

    > &#128221; Pour les restaurations Azure Cosmos DB, vous ne restaurez ***jamais*** sur le compte *existant* et vous devrez toujours créer un compte Azure Cosmos DB.

    > &#128221; Bien que vous ayez pu choisir de restaurer l’ensemble de la base de données ou même le compte entier, dans un environnement de production réel, les bases de données peuvent être énormes. Dans de nombreux scénarios, il peut être plus rapide de restaurer simplement les conteneurs ou les bases de données nécessaires.

1. Cette restauration peut prendre 15 minutes ou plus, accédez à la section suivante et laissez cette restauration en cours d’exécution en arrière-plan.

## Passer en revue les données restaurées

Les restaurations peuvent prendre beaucoup de temps en fonction de la taille de la base de données et d’autres facteurs. Une fois les restaurations de compte Azure Cosmos DB terminées :

1. Pour notre première restauration, assurez-vous que le troisième document a été récupéré.

1. Pour la deuxième restauration, nous devrions avoir restauré la table client.

## Nettoyage

1. Supprimez les deux nouveaux comptes Azure Cosmos DB créés par les restaurations de compte.

1. Supprimez la base de données Sales et, si nécessaire, supprimez le compte Azure Cosmos DB d’origine.

[/azure/cosmos-db/continuous-backup-restore-introduction]:https://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction

