---
lab:
  title: Activer les fournisseurs de ressources
  module: Setup
---

# Activer les fournisseurs de ressources Azure

Certains fournisseurs de ressources doivent être inscrits dans votre abonnement Azure. Suivez ces étapes pour vous assurer qu’ils sont inscrits.

1. Dans une nouvelle fenêtre ou un nouvel onglet du navigateur web, accédez au portail Azure (``portal.azure.com``).

1. Connectez-vous au portail en utilisant les informations d’identification Microsoft associées à votre abonnement.

1. Dans la page **Accueil**, sélectionnez **Abonnements**.

    > &#128161 ; Alternativement; développez le menu **&#8801;**, sélectionnez **Tous les services**, puis, dans la catégorie **Tout**, sélectionnez **Abonnements**.

1. Sélectionnez votre abonnement Azure.

    > &#128221; Si vous avez plusieurs abonnements, sélectionnez celui que vous avez créé en échangeant votre Pass Azure.

1. Dans le panneau de votre abonnement, dans la section **Paramètres**, sélectionnez **Fournisseurs de ressources**.

1. Dans la liste des fournisseurs de ressources, vérifiez que les fournisseurs suivants sont inscrits :
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; Si un fournisseur n’est pas inscrit, sélectionnez ce fournisseur, puis sélectionnez **Inscrire**.

1. Fermez votre fenêtre ou votre onglet de navigateur web.

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
