---
layout: post
title: Use Envoy as an edge proxy on Kubernetes
category: posts
---

We will use Minikube for this tutorial. To start Minikube

><iframe
  src="https://carbon.now.sh/embed?bg=rgba(171%2C%20184%2C%20195%2C%201)&t=seti&wt=none&l=auto&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=minikube%2520start%2520--vm%253Dtrue"
  style="width: 353px; height: 204px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

and then hook up the Minikube Docker daemon for local Docker images once Minikube started.

><iframe
  src="https://carbon.now.sh/embed?bg=rgba(171%2C%20184%2C%20195%2C%201)&t=seti&wt=none&l=auto&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=eval%2520%2524(minikube%2520docker-env)"
  style="width: 378px; height: 204px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

The following are Node.js service and Dockerfile.

{% highlight javascript %}
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
{% endhighlight %}

and Dockerfile

{% highlight bash %}
FROM node:12

WORKDIR /usr/src/app

COPY index.js .

EXPOSE 8080

CMD [ "node", "index.js" ]
{% endhighlight %}

We will build local Docker images with names ping-svc:dev and pong-svc:dev.


