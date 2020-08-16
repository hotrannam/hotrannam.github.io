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

>
<div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%"><span style="color: #008800; font-weight: bold">const</span> http <span style="color: #333333">=</span> require(<span style="background-color: #fff0f0">&#39;http&#39;</span>);
>
<span style="color: #008800; font-weight: bold">const</span> port <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">8080</span>;
>
<span style="color: #008800; font-weight: bold">const</span> server <span style="color: #333333">=</span> http.createServer((req, res) <span style="color: #333333">=&gt;</span> {
  res.statusCode <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">200</span>;
  res.setHeader(<span style="background-color: #fff0f0">&#39;Content-Type&#39;</span>, <span style="background-color: #fff0f0">&#39;text/plain&#39;</span>);
  res.end(<span style="background-color: #fff0f0">&#39;ping me ...&#39;</span>);
});
>
server.listen(port, () <span style="color: #333333">=&gt;</span> {
  console.log(<span style="color: #FF0000; background-color: #FFAAAA">`</span>Server running at ${port}<span style="color: #FF0000; background-color: #FFAAAA">`</span>);
});
</pre></div>
>

>
<div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">FROM node:12

WORKDIR /usr/src/app

COPY index.js .

EXPOSE 8080
>
CMD <span style="color: #333333">[</span> <span style="background-color: #fff0f0">&quot;node&quot;</span>, <span style="background-color: #fff0f0">&quot;index.js&quot;</span> <span style="color: #333333">]</span>
</pre></div>
>

We will build local Docker images with names ping-svc:dev and pong-svc:dev.


