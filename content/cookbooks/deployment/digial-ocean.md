---
datetime: 2020-04-22
author: Virk
avatarUrl: https://res.cloudinary.com/adonis-js/image/upload/v1619103621/adonisjs-authors-avatars/DYO4KUru_400x400_shujhw.jpg
summary: Cookbook to deploy AdonisJS application to Digital ocean apps platform
---

Ce guide couvre les étapes du déploiement d'une application AdonisJS sur [Digital ocean apps platform](https://docs.digitalocean.com/products/app-platform/).

Le déploiement d'une application AdonisJS ne diffère pas du déploiement d'une application Node.js standard. Vous devez simplement garder quelques éléments à l'esprit :

- Vous construisez votre source TypeScript en JavaScript, avant de déployer l'application.
- Vous devrez démarrer le serveur à partir du dossier `build` et non de la racine du projet. Il en va de même pour l'exécution de migrations ou d'autres applications Node.js.

Vous pouvez construire votre projet pour la production en lançant la commande ace suivante. En savoir plus sur le [processus de construction TypeScript](../../guides/fundamentals/typescript-build-process.md)

```sh
node ace build --production

# OU utiliser le script npm
npm run build
```

## Configuration de l'application DO

Au moment de déployer votre application sur la plateforme DO apps, il vous sera demandé de fournir l'environnement. Vous pouvez consulter le fichier `.env` de développement pour les variables que vous devez définir.

- Définissez la variable d'environnement `PORT` identique au **port HTTP** que vous avez sélectionné dans les paramètres.
- Assurez-vous de générer la clé de l'application en lançant la commande `node ace generate:key` sur votre machine locale et définissez-la comme variable d'environnement `APP_KEY`.
- Sauvegardez la `APP_KEY` de manière sécurisée. Si vous perdez cette clé, toutes les données d'encryptage, les cookies et les sessions deviendront invalides.
- Assurez-vous de changer la **commande d'exécution** pour démarrer le serveur depuis le dossier de construction. `node build/server.js`.

![](https://res.cloudinary.com/adonis-js/image/upload/q_auto,f_auto/v1619105542/v5/do-start-screen.jpg)

## Utilisation de la base de données
Vous pouvez également ajouter la base de données en tant que composant de votre application et mettre à jour les variables d'environnement avec les informations d'identification de la base de données.

Nous trouvons que les variables d'environnement de la base de données de Digital ocean sont très génériques et nous recommandons de les réaffecter comme suit :

#### Variables env injectées dans l'océan numérique
```dotenv
HOSTNAME=
PORT=
USERNAME=
PASSWORD=
DATABASE=
```

#### Re-mapper les variables d'environnement pour les rendre plus spécifiques
Vous devez redéfinir les variables d'environnement pour qu'elles soient plus spécifiques. Par exemple

![](https://res.cloudinary.com/adonis-js/image/upload/q_auto,f_auto/v1619105542/v5/do-remmaped-env-vars.jpg)

### Exécution des migrations
Une fois que vous avez ajouté la base de données, vous devrez ajouter un nouveau composant de **"Job type "** et vous assurer de sélectionner **"Before every deploy "** comme cycle de vie.

Digital Ocean vous fera répéter le même processus d'ajout d'un nouveau composant, de re-sélection du référentiel et de redéfinition des variables d'environnement. Cependant, cette fois-ci, nous ajoutons un travail et non un service web.

Assurez-vous également de mettre à jour la **commande d'exécution** en `node build/ace make:migration --force`.

![](https://res.cloudinary.com/adonis-js/image/upload/q_auto,f_auto/v1619105809/v5/do-job-component.jpg)

## Gérer les fichiers téléchargés par les utilisateurs
Les applications Digital Ocean n'ont pas de stockage persistant et vous ne pouvez donc pas sauvegarder les fichiers téléchargés par les utilisateurs sur le système de fichiers du serveur. Il ne vous reste donc qu'une seule option : sauvegarder les fichiers téléchargés sur un service externe comme Digital ocean spaces.

Nous travaillons actuellement sur un module qui vous permet d'utiliser le système de fichiers local pendant le développement et de passer ensuite à un système de fichiers externe comme Digital ocean spaces pour la production. Tout cela sans changer une seule ligne de code.