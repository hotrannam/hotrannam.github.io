---
layout: post
title: Use Envoy as an edge proxy on Kubernetes
category: posts
---

We will use `Minikube` for this tutorial. To start Minikube

```minikube start --vm=true```

and then hook up the Minikube Docker daemon for local Docker images once Minikube started.

```eval $(minikube docker-env)```

The following are Node.js service and Dockerfile.

```js
const http = require('http');

const port = 8080;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('ping me ...');
});

server.listen(port, () => {
  console.log(`Server running at ${port}`);
});
```

```
FROM node:12

WORKDIR /usr/src/app

COPY index.js .

EXPOSE 8080

CMD [ "node", "index.js" ]
```

We will build local Docker images with names ping-svc:dev and pong-svc:dev.


