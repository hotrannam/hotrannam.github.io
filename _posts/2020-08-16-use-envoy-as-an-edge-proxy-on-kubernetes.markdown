---
layout: post
title: Use Envoy as an edge proxy on Kubernetes
category: posts
---

In the first post of the Envoy series, I will go through how to use Envoy proxy as an API gateway for backend services which are deployed on Kubernetes.

First, I build a Docker image for Envoy proxy with a customized configuration. For the complete configuration, it can be found [here](https://github.com/hotrannam/k8s-dev/blob/master/edge-proxy/envoy.yaml).

```bash
FROM envoyproxy/envoy-dev:22c921a6318f07847afc61bc137a9e4833889b9d
COPY envoy.yaml /etc/envoy/envoy.yaml
```

In the listener, I bind Envoy proxy to port 30000.

```yaml
listeners:
- name: listener_0
  address:
    socket_address: { address: 0.0.0.0, port_value: 30000 }
```

Next, I config routes for handling requests that Envoy proxy receives. I use `http_connection_manager` for HTTP filter in this case.

```yaml
routes:
- match: { prefix: "/pong" }
  route: { cluster: pong-cluster }
- match: { prefix: "/ping" }
  route: { cluster: ping-cluster }
- match: { prefix: "/" }
  route: { cluster: ping-cluster }
```

There are 2 backend services - ping and pong - to handle requests. Based on the URL matching with the prefix, we will route requests to right clusters. A cluster is a collection of IP address (or domain name) and port of a backend service. Think multiple instances of a backend service.

```yaml
clusters:
- name: ping-cluster
  connect_timeout: 0.25s
  type: STRICT_DNS
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  load_assignment:
    cluster_name: ping-cluster
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: ping-svc.default.svc.cluster.local
              port_value: 8080
```

I use Kubernetes domain name `ping-svc.default.svc.cluster.local` for service address and cluster type `STRICT_DNS`. Envoy supports some load balancer types, I use round robin order for upstream selection for now.

Next, I define Kubernetes manifest for Envoy proxy.

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
```

```yaml
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

You will be noticed by `imagePullPolicy: Never` as I tell Kubernetes to use local Dockr images. You can get complete source code with instructions [here](https://github.com/hotrannam/k8s-dev).

