---
lab:
  title: "Configurer le débit pour Azure Cosmos\_DB for NoSQL avec le portail Azure"
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Configurer le débit pour Azure Cosmos DB for NoSQL avec le portail Azure

L’une des choses les plus importantes pour encapsuler votre tête consiste à configurer le débit dans Azure Cosmos DB for NoSQL. Pour créer un conteneur Azure Cosmos DB for NoSQL, vous devez d’abord créer un compte, puis une base de données; dans cet ordre.

Dans ce labo, vous allez approvisionner le débit à l’aide de différentes méthodes dans l’Explorateur de données. Vous allez approvisionner le débit manuellement ou à l’aide de la mise à l’échelle automatique, au niveau de la base de données et du conteneur.

## Créer un compte serverless

Commençons par créer un compte serverless. Il n’y a pas beaucoup à configurer ici, car tout est serverless. Lorsque nous créons notre base de données et notre conteneur, nous n’avons pas à provisionner le débit du tout. Vous verrez tout cela au fur et à mesure que nous allons créer ce compte.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Dans la catégorie **services Azure**, sélectionnez **Créer une ressource**, puis sélectionnez **Azure Cosmos DB**.

    > &#128161; Autrement, développez le menu **&#8801;**, sélectionnez **Tous les services**, dans la catégorie **Bases de données**, sélectionnez **Azure Cosmos DB**, puis sélectionnez **Créer**.

1. Dans le volet **Sélectionner l’API**, sélectionnez l’option **Créer** dans la section **Azure Cosmos DB for NoSQL** .

1. Dans le volet **Créer un compte Azure Cosmos DB**, observez l’onglet **Informations de base**.

1. Sous l’onglet **Informations de base**, entrez les valeurs suivantes pour chaque paramètre :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **Abonnement** | **Utilisez votre abonnement Azure existant.** *Toutes les ressources doivent appartenir à un groupe de ressources. Chaque groupe de ressources doit appartenir à un abonnement.* |
    | **Groupe de ressources** | **Utilisez un groupe de ressources existant ou créez-en un.** *Toutes les ressources doivent appartenir à un groupe de ressources.*|
    | **Nom du compte** |  **Entrez un nom globalement unique.** *Nom du compte globalement unique. Ce nom sera utilisé comme élément de l’adresse DNS pour les requêtes.  Le portail vérifiera le nom en temps réel.* |
    | **Lieu** | **Choisissez une région disponible.** *Sélectionnez la région géographique depuis laquelle votre base de données sera initialement hébergée.* |
    | **Mode de capacité** | Sélectionnez **serverless** |

1. Sélectionnez **Vérifier + créer** pour accéder à l’onglet **Vérifier + créer**, puis sélectionnez **Créer**.

    > &#128221; 10 à 15 minutes peuvent être nécessaires avant de pouvoir utiliser le compte Azure Cosmos DB for NoSQL.

1. Observez le volet **Déploiement**. Une fois le déploiement terminé, le volet se met à jour avec le message **Déploiement réussi**.

1. Toujours dan le volet **Déploiement**, sélectionnez **Accéder à la ressource**.

1. Dans le volet du **compte Azure Cosmos DB**, sélectionnez **Explorateur de données** dans le menu des ressources.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre contextuelle **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *`cosmicworks`* |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **cosmicworks** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utilisez la valeur existante* &vert; *cosmicworks* |
    | **ID de conteneur** | *`products`* |
    | **Clé de partition** | *`/categoryId`* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **cosmicworks**, puis observez le nœud de conteneur des **produits** dans la hiérarchie.

1. Retournez à l’**Accueil** du portail Azure.

## Créer un compte provisionné

Maintenant, nous allons créer un compte de débit approvisionné avec des options de configuration plus traditionnelles. Ce type de compte ouvre un monde d’options de configuration pour nous qui peut être un peu écrasant. Nous allons parcourir quelques exemples de paires de bases de données et de conteneurs qui sont possibles ici.

1. Dans la catégorie **services Azure**, sélectionnez **Créer une ressource**, puis sélectionnez **Azure Cosmos DB**.

    > &#128161; Autrement, développez le menu **&#8801;**, sélectionnez **Tous les services**, dans la catégorie **Bases de données**, sélectionnez **Azure Cosmos DB**, puis sélectionnez **Créer**.

1. Dans le volet **Sélectionner l’API**, sélectionnez l’option **Créer** dans la section **Azure Cosmos DB for NoSQL** .

1. Dans le volet **Créer un compte Azure Cosmos DB**, observez l’onglet **Informations de base**.

1. Sous l’onglet **Informations de base**, entrez les valeurs suivantes pour chaque paramètre :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **Abonnement** | **Utilisez votre abonnement Azure existant.** *Toutes les ressources doivent appartenir à un groupe de ressources. Chaque groupe de ressources doit appartenir à un abonnement.* |
    | **Groupe de ressources** | **Utilisez un groupe de ressources existant ou créez-en un.** *Toutes les ressources doivent appartenir à un groupe de ressources.*|
    | **Nom du compte** |  **Entrez un nom globalement unique.** *Nom du compte globalement unique. Ce nom sera utilisé comme élément de l’adresse DNS pour les requêtes.  Le portail vérifiera le nom en temps réel.* |
    | **Lieu** | **Choisissez une région disponible.** *Sélectionnez la région géographique depuis laquelle votre base de données sera initialement hébergée.* |
    | **Mode de capacité** | **Débit approvisionné** |
    | **Appliquer la remise de niveau Gratuit** | **Ne pas appliquer** |
    | **Limiter la quantité totale de débit pouvant être approvisionnée sur ce compte** | **Décoché** |

1. Sélectionnez **Vérifier + créer** pour accéder à l’onglet **Vérifier + créer**, puis sélectionnez **Créer**.

    > &#128221; 10 à 15 minutes peuvent être nécessaires avant de pouvoir utiliser le compte Azure Cosmos DB for NoSQL.

1. Observez le volet **Déploiement**. Une fois le déploiement terminé, le volet se met à jour avec le message **Déploiement réussi**.

1. Toujours dan le volet **Déploiement**, sélectionnez **Accéder à la ressource**.

1. Dans le volet du **compte Azure Cosmos DB**, sélectionnez **Explorateur de données** dans le menu des ressources.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre contextuelle **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *`nothroughputdb`* |
    | **Approvisionner le débit** | *Décoché* |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **nothroughputdb** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utiliser des * &vert; *nothroughputdb* existants |
    | **ID de conteneur** | *`requiredthroughputcontainer`* |
    | **Clé de partition** | *`/primarykey`* |
    | **Débit du conteneur** | *Manuel* |
    | **RU/s** | *`400`* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **nothroughputdb**, puis observez le nœud de conteneur des **produits** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, développez **Nouveau conteneur**, puis sélectionnez **Nouvelle base de données**.

1. Dans la fenêtre contextuelle **Nouvelle base de données**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *`manualthroughputdb`* |
    | **Approvisionner le débit** | *Activée* |
    | **Débit de la base de données** | *Manuel* |
    | **RU/s** | *`400`* |

1. De retour dans le volet **Explorateur de données**, observez le nœud de base de données **manualthroughputdb** dans la hiérarchie.

1. Dans le volet **Explorateur de données**, sélectionnez **Nouveau conteneur**.

1. Dans la fenêtre contextuelle **Nouveau conteneur**, entrez les valeurs suivantes pour chaque paramètre, puis sélectionnez **OK** :

    | **Paramètre** | **Valeur** |
    | --: | :-- |
    | **ID de base de données** | *Utiliser le manuel * &vert; *manuelthroughputdb* existant |
    | **ID de conteneur** | *`childcontainer`* |
    | **Clé de partition** | *`/primarykey`* |
    | **Provisionner un débit dédié pour ce conteneur** | *Activée* |
    | **Débit du conteneur** | *Manuel* |
    | **RU/s** | *`1000`* |

1. De retour dans le volet **Explorateur de données**, développez le nœud de base de données **manualthroughputdb**, puis observez le nœud de conteneur des **produits** dans la hiérarchie.
