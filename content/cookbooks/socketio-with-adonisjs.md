---
datetime: 2020-06-08
author: Virk
avatarUrl: https://res.cloudinary.com/adonis-js/image/upload/v1619103621/adonisjs-authors-avatars/DYO4KUru_400x400_shujhw.jpg
summary: Apprenez comment utiliser socket.io avec AdonisJS
---

[Socket.io](https://socket.io/) est une bibliothèque très populaire pour la communication bidirectionnelle et en temps réel. Dans ce guide, nous allons apprendre à utiliser socket.io avec AdonisJS.

La première étape consiste à installer le package depuis le registre des packages npm.

:::codegroup

```sh
// title: npm
npm i socket.io
```

```sh
// title: yarn
yarn add socket.io
```
:::

Ensuite, créons une classe de service responsable du démarrage du serveur socketio et nous fournissant une référence à celui-ci.

Le code du service peut être n'importe où dans votre code source. Je préfère le garder à l'intérieur du répertoire `./app/Services`.

```ts
// title: app/Services/Ws.ts
import { Server } from 'socket.io'
import AdonisServer from '@ioc:Adonis/Core/Server'

class Ws {
  public io: Server
  private booted = false

  public boot() {
    /**
     * Ignore multiple calls to the boot method
     */
    if (this.booted) {
      return
    }

    this.booted = true
    this.io = new Server(AdonisServer.instance!)
  }
}

export default new Ws()
```

Ensuite, créons un fichier `start/socket.ts` et collons-y le contenu suivant. Comme le fichier `routes`, nous utiliserons ce fichier pour écouter les connexions socket entrantes.

```ts
// title: start/socket.ts
import Ws from 'App/Services/Ws'
Ws.boot()

/**
 * Listen for incoming socket connections
 */
Ws.io.on('connection', (socket) => {
  socket.emit('news', { hello: 'world' })

  socket.on('my other event', (data) => {
    console.log(data)
  })
})
```

Enfin, importez le fichier créé ci-dessus dans le fichier `providers/AppProvider.ts` sous la méthode `ready`.

La méthode `ready` s'exécute après que le serveur HTTP AdonisJS soit prêt, et c'est à ce moment que nous devons établir la connexion socketio.

```ts
// title: providers/AppProvider.ts
import { ApplicationContract } from '@ioc:Adonis/Core/Application'

export default class AppProvider {
  constructor(protected app: ApplicationContract) {}

  public async ready() {
    if (this.app.environment === 'web') {
      await import('../start/socket')
    }
  }
}
```

C'est tout ce que vous devez faire pour configurer socket.io. Allons plus loin et testons également que nous pouvons établir une connexion à partir du navigateur.

## Configuration du client
Nous utiliserons la version CDN de socketio-client pour simplifier les choses. Ouvrons le fichier `resources/views/welcome.edge` et ajoutons les scripts suivants à la page.

```edge
// title: resources/views/welcome.edge

<body>
  <!-- Rest of markup -->

  // highlight-start
    <script src="https://cdn.socket.io/4.0.1/socket.io.min.js"></script>
    <script>
      const socket = io()
      socket.on('news', (data) => {
        console.log(data)
        socket.emit('my other event', { my: 'data' })
      })
    </script>
  // highlight-end
</body>
</html>
```

Commençons par démarrer le serveur de développement en exécutant `node ace serve --watch` et ouvrons [http://localhost:3333](http://localhost:3333) dans le navigateur pour tester l'intégration.

::video{url="https://res.cloudinary.com/adonis-js/video/upload/v1591543846/adonisjs.com/blog/socket-io_i4qe6n.mp4" controls}

## Diffusez de n'importe où
Depuis que nous avons abstrait la configuration socketio dans une classe de service, vous pouvez l'importer de n'importe où dans votre code pour diffuser des événements. Par exemple:

```ts
import Ws from 'App/Services/Ws'

class UsersController {
  public async store() {
    Ws.io.emit('new:user', { username: 'virk' })
  }
}
```

## Configurer CORS
La connexion socketio utilise directement le serveur HTTP sous-jacent de Node.js, et donc la configuration CORS d'AdonisJS ne fonctionnera pas avec elle.

Vous pouvez cependant configurer [cors avec socketio directement](https://socket.io/docs/v4/handling-cors/) comme ceci.

```ts

```ts
class Ws {
  public io: Server
  private booted = false

  public boot() {
    /**
     * Ignore multiple calls to the boot method
     */
    if (this.booted) {
      return
    }

    this.booted = true
    // highlight-start
    this.io = new Server(AdonisServer.instance!, {
      cors: {
        origin: '*'
      }
    })
    // highlight-end
  }
}
```
