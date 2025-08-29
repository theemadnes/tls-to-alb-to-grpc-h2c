# TLS-TO-ALB-TO-GRPC-H2C

Assumes you already have a Google Cloud application load balancer created via Gateway API. Example used here:
```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: external-http
  namespace: asm-ingress
spec:
  addresses:
  - type: NamedAddress
    value: mcg-ip # existing static IP
  gatewayClassName: gke-l7-global-external-managed-mc # see https://cloud.google.com/kubernetes-engine/docs/how-to/gatewayclass-capabilities
  listeners:
  - allowedRoutes:
      namespaces:
        from: Same
    name: http
    port: 80
    protocol: HTTP
  - allowedRoutes:
      kinds:
      - group: gateway.networking.k8s.io
        kind: HTTPRoute
      namespaces:
        from: Same
    name: https
    port: 443
    protocol: HTTPS
```

> Note that the above `gatewayClass` is multicluster (`-mc`) so the `targetRef` fields will be slightly different when using single-cluster `gatewayClass`es.

### setup

create namespace
```
kubectl create ns grpc-h2c
```

create whereami resources
```
kubectl apply -f whereami-grpc/
```
