---
layout: post
title: Use Envoy as an edge proxy on Kubernetes
category: posts
---

To start Minikube and hook up the Minikube Docker daemon for local Docker images once Minikube started.

```bash
minikube start --vm=true
eval $(minikube docker-env)
```

Ping and pong services are written in Node.js

```javascript
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

Dockerfile of Docker images ping-svc:dev and pong-svc:dev

```bash
FROM node:12
WORKDIR /usr/src/app
COPY index.js .
EXPOSE 8080
CMD [ "node", "index.js" ]
```


