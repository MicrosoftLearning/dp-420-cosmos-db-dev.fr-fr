---
lab:
  title: Configurer l’environnement lab
  module: Setup
---

# Configurer un environnement de labo local

Dans l’idéal, vous devez suivre ces labos dans un environnement de labo hébergé. Si vous souhaitez les terminer sur votre propre ordinateur, vous pouvez le faire en installant le logiciel suivant. Vous pouvez rencontrer des dialogues et un comportement inattendus lors de l’utilisation de votre propre environnement. En raison de la large gamme de configurations locales possibles, l’équipe de formation ne peut pas prendre en charge les problèmes que vous pouvez rencontrer dans votre propre environnement.

## Outils de ligne de commande Azure

1. [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) ou [Azure Cloud Shell](https://shell.azure.com) - Installez-les si vous souhaitez exécuter des commandes via l’interface CLI au lieu du portail Azure.

## Node.JS

1. Téléchargez et installez Node.js v18.0.0 ou version ultérieure à partir de [nodejs.org/en/download].

1. Téléchargez et installez NPM v10.2.3 ou version ultérieure à partir de [npmjs.com/get-npm].

Méthode recommandée pour installer la dernière version de NPM et node.js sur Windows :

- Installer NVM à partir de [github.com/coreybutler/nvm-windows]
- Exécuter nvm install latest
- Exécuter nvm list (pour afficher les versions de NPM/node.js disponibles)
- Exécuter nvm use latest (pour utiliser la dernière version disponible)

### Git

1. Téléchargez et installez à partir de [git-scm.com/downloads].

    - Utilisez les options par défaut dans le programme d’installation.

### Visual Studio Code (et extensions)

1. Téléchargez et installez à partir de [code.visualstudio.com/download].

    - Utilisez les options par défaut dans le programme d’installation.

1. Après l’installation, démarrez Visual Studio Code.

### Émulateur Azure Cosmos DB

1. Téléchargez et installez à partir de [docs.microsoft.com/azure/cosmos-db/local-emulator].
    - Utilisez les options par défaut dans le programme d’installation.

### Cloner le référentiel de labo

Si vous n’avez pas encore cloné le référentiel de code du labo pour **Générer des copilotes avec Azure Cosmos DB** dans l’environnement où vous travaillez sur ce labo, suivez ces étapes. Sinon, ouvrez le dossier précédemment cloné dans **Visual Studio Code**.

1. Démarrez **Visual Studio Code**.

    > &#128221; Si vous n’êtes pas encore familiarisé avec l’interface de Visual Studio Code, consultez le [guide de démarrage de Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Ouvrez la palette de commandes et exécutez **Git : Cloner** pour cloner le référentiel GitHub ``https://github.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs`` dans un dossier local de votre choix.

    > &#128161; Vous pouvez utiliser le raccourci clavier **Ctrl + Maj + P** pour ouvrir la palette de commandes.

1. Une fois le référentiel cloné, ouvrez le dossier local que vous avez sélectionné dans **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks

[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[code.visualstudio.com/download]: https://code.visualstudio.com/download
[git-scm.com/downloads]: https://git-scm.com/downloads
[nodejs.org/en/download]: https://nodejs.org/en/download
[npmjs.com/get-npm]: https://npmjs.com/get-npm
[github.com/coreybutler/nvm-windows]: https://github.com/coreybutler/nvm-windows
