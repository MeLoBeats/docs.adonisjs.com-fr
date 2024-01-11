---
datetime: 2020-04-22
author: Virk
avatarUrl: https://res.cloudinary.com/adonis-js/image/upload/v1619103621/adonisjs-authors-avatars/DYO4KUru_400x400_shujhw.jpg
summary: Cookbook to deploy AdonisJS application to Heroku
---

Ce guide couvre les étapes à suivre pour déployer une application AdonisJS sur [Heroku](https://devcenter.heroku.com/articles/deploying-nodejs).

Le déploiement d'une application AdonisJS ne diffère pas du déploiement d'une application Node.js standard. Vous devez simplement garder quelques points à l'esprit :

- Vous devez compiler votre code TypeScript en JavaScript avant de déployer l'application.
- Vous devrez démarrer le serveur à partir du dossier `build` et non de la racine du projet. Il en va de même pour l'exécution des migrations ou de toute autre application Node.js.

Vous pouvez compiler votre projet pour la production en exécutant la commande ace suivante. En savoir plus sur le [processus de compilation TypeScript](../../guides/fundamentals/typescript-build-process.md)

```sh
node ace build --production

# OR use the npm script
npm run build
```

## Ajout du Procfile
Avant de pousser votre code vers Heroku pour le déploiement, assurez-vous de créer un [Procfile](https://devcenter.heroku.com/articles/procfile#deploying-to-heroku) à la racine de votre application.

Ce fichier indique à Heroku d'exécuter toujours les migrations lors de la release et de démarrer le serveur à partir du dossier `build`.

```text
web: node build/server.js
release: node build/ace migration:run --force
```

## Définition des variables d'environnement
Vous devez également définir les variables d'environnement dans le tableau de bord Heroku. Vous pouvez consulter le fichier `.env` de développement pour connaître les variables que vous devez définir avec Heroku.

- Ne définissez pas la variable d'environnement `PORT`. Heroku la définira automatiquement pour vous.
- Assurez-vous de générer la clé de l'application en exécutant la commande `node ace generate:key` et de la définir comme variable d'environnement `APP_KEY`.
- Sauvegardez la clé `APP_KEY` de manière sécurisée. Si vous perdez cette clé, toutes les données de chiffrement, les cookies et les sessions deviendront invalides.

![](https://res.cloudinary.com/adonis-js/image/upload/f_auto,q_auto/v1619085409/v5/heroku-env-vars.jpg)

## Il est temps de déployer
Vous pouvez maintenant pousser votre code vers Heroku en lançant la commande `git push heroku master`. Heroku effectuera les étapes suivantes pour vous.

- Il détectera votre application comme étant une application Node.js et utilisera le buildpack `heroku/nodejs` pour la construire et la déployer.
- Il détectera le script `build` dans le fichier `package.json` et construira votre code TypeScript en JavaScript. **Vous devez toujours exécuter le code JavaScript en production**.
- Après la construction, il va [prune](https://docs.npmjs.com/cli/v7/commands/npm-prune) les dépendances de développement.
- Exécute le script `release` défini dans le `Procfile`.
- Exécute le script `web` défini dans le `Procfile`.

## Utilisation de la base de données
Vous pouvez utiliser les modules complémentaires Heroku pour ajouter une base de données à votre application. Assurez-vous simplement de définir les variables d'environnement nécessaires pour qu'AdonisJS puisse se connecter à votre base de données.

Encore une fois, vous pouvez consulter le `.env` de développement pour voir le nom des variables d'environnement que vous utilisez pour la connexion à la base de données.

:::note

Si vous utilisez PostgreSQL et que vous obtenez l'erreur `pg_hba.conf`. Assurez-vous alors d'activer les certificats SSL pour votre application. Si vous ne pouvez pas activer SSL, vous devez mettre à jour la connexion à la base de données pour autoriser les connexions non-SSL.

```ts
// title: config/database.js
pg: {
  client: 'pg',
  connection: {
    // ....
    ssl: {
      rejectUnauthorized: false
    }
  }
}
```

:::

## Gestion des fichiers téléchargés par les utilisateurs
Heroku ne dispose pas de [stockage persistant](https://help.heroku.com/K1PPS2WM/why-are-my-file-uploads-missing-deleted), et vous ne pouvez pas sauvegarder les fichiers téléchargés par l'utilisateur sur le système de fichiers du serveur. Il ne vous reste donc qu'une seule option : sauvegarder les fichiers téléchargés sur un service externe tel que S3.

Nous travaillons actuellement sur un module qui vous permet d'utiliser le système de fichiers local pendant le développement et de passer ensuite à un système de fichiers externe comme S3 pour la production. Tout cela sans changer une seule ligne de code.