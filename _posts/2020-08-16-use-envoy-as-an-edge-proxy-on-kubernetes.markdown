---
layout: post
title: Use Envoy as an edge proxy on Kubernetes
category: posts
---

This tutorial demostrates how to use Envoy proxy as an API gateway for ping and pong services. All is deployed to Kubernetes or Minikube.

We need to build Docker image for Envoy proxy with a configuration. The full configuration of Envoy proxy can be found [here](https://github.com/hotrannam/k8s-dev/blob/master/edge-proxy/envoy.yaml)


```bash
FROM envoyproxy/envoy-dev:22c921a6318f07847afc61bc137a9e4833889b9d
COPY envoy.yaml /etc/envoy/envoy.yaml
```

We tell Envoy proxy to listen to a port, in this case 30000.

```yaml
listeners:
- name: listener_0
  address:
    socket_address: { address: 0.0.0.0, port_value: 30000 }
```

We use `http_connection_manager` for HTTP filter. Next, we config routes to handle requests that Envoy proxy receives.

```yaml
routes:
- match: { prefix: "/pong" }
  route: { cluster: pong-svc }
- match: { prefix: "/ping" }
  route: { cluster: ping-svc }
- match: { prefix: "/" }
  route: { cluster: ping-svc }
```

As mentioned early, we have 2 services ping and pong. Based on the URL matching with the prefix, we will route requests to right clusters. Now, let's talk about cluster.

```yaml
clusters:
- name: ping-svc
  connect_timeout: 0.25s
  type: STRICT_DNS
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  load_assignment:
    cluster_name: ping-svc
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: ping-svc.default.svc.cluster.local
              port_value: 8080
```

A cluster is a collection of address with IP and port that are the backend for a service, in this case we use domain name in Kubernetes instead - `ping-svc.default.svc.cluster.local`.

Envoy supports some load balancer types, we use round robin order for upstream selection for now.

Next, we define Kubernetes manifest for Envoy proxy.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: edge-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: edge-proxy
  template:
    metadata:
      labels:
        app: edge-proxy
    spec:
      containers:
      - name: edge-proxy
        image: edge-proxy:dev
        imagePullPolicy: Never
        ports:
        - containerPort: 30000
          protocol: TCP
apiVersion: v1
kind: Service
metadata:
  name: edge-proxy
spec:
  type: NodePort
  selector:
    app: edge-proxy
  ports:
    - name: edge-proxy
      protocol: TCP
      port: 30000
      targetPort: 30000
```

In this tutorial, we build local Docker images and config Kubernetes to use them as you will be noticed with `imagePullPolicy: Never`. You can follow instruction [here](https://github.com/hotrannam/k8s-dev/blob/master/README.md) and grab full source code at this GitHub [repo](https://github.com/hotrannam/k8s-dev)

This is the first post in the series of Envoy, I will write more posts on other features of Envoy in context of deploying to Kubenetes.
