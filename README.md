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
        from: All # this is way too permissive for production use
    name: http
    port: 80
    protocol: HTTP
  - allowedRoutes:
      kinds:
      - group: gateway.networking.k8s.io
        kind: HTTPRoute
      namespaces:
        from: All # this is way too permissive for production use
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

create load balancer resources
```
kubectl apply -f load-balancer-resources/
```

### call the service using grpcurl

```
grpcurl frontend.endpoints.e2m-private-test-01.cloud.goog:443 whereami.Whereami.GetPayload
grpcurl -proto whereami.proto frontend.endpoints.e2m-private-test-01.cloud.goog:443 whereami.Whereami.GetPayload
```

output:
```
$ grpcurl -proto whereami.proto frontend.endpoints.e2m-private-test-01.cloud.goog:443 whereami.Whereami.GetPayload
{
  "cluster_name": "edge-to-mesh-01",
  "metadata": "grpc-frontend",
  "node_name": "gk3-edge-to-mesh-01-nap-1c08r7is-99e76d24-6f6c",
  "pod_ip": "10.54.0.174",
  "pod_name": "whereami-grpc-75b6c67769-rmqkp",
  "pod_name_emoji": "🏊‍♀",
  "pod_namespace": "grpc-h2c",
  "pod_service_account": "whereami-grpc",
  "project_id": "e2m-private-test-01",
  "timestamp": "2025-08-29T05:11:55",
  "zone": "us-central1-f",
  "gce_instance_id": "4739827861112604717",
  "gce_service_account": "e2m-private-test-01.svc.id.goog"
}
```