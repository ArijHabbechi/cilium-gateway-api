# cilium-gateway-api
Curated Cilium Gateway API examples demonstrating HTTP routing, HTTPS termination, TLS passthrough, and weighted traffic splitting.
## Overview
This repository is a minimal, focused reference for using Cilium as a Kubernetes Gateway API controller.  
It contains only:

-   Gateway API manifests (HTTPRoute, HTTPS termination, TLSRoute passthrough, traffic splitting)
-   A single README with the essential commands, checks, and conceptual notes

The goal is to understand Gateway API semantics with Cilium, not application deployment.

----------

:warning:  **Before applying anything : Cilium must be running as the dataplane and gateway controller.**

#### Required features: 
-   kubeProxyReplacement=true
-   gatewayAPI.enabled=true
-   Gateway API CRDs installed

#### Gateway API CRDs  
kubectl get crd gateways.gateway.networking.k8s.io  
kubectl get crd httproutes.gateway.networking.k8s.io  
kubectl get crd tlsroutes.gateway.networking.k8s.io  
  
#### Cilium health + config  
```bash 
cilium status --wait  
cilium config view | grep enable-gateway-api  
```
#### GatewayClass provided by Cilium  
```kubectl get gatewayclass```

If the GatewayClass cilium is present and Accepted, the cluster is ready.


## Routing models covered in this repo
Gateway API decomposes traffic management into clear, role-oriented APIs.  
Each folder in manifests/ demonstrates one routing model :


### 1) HTTP routing (HTTPRoute)
-   Layer 7 HTTP routing
-   Path, header, query, and method-based matching
-   Replaces complex Ingress + annotations

####  Traffic flow

Client → Gateway (HTTP) → Service (HTTP)

####  Used for
-   REST APIs
-   Path-based routing (/api, /details)
-   Header or query-based routing
-   Multi-rule L7 policies
----------

###  2) HTTPS routing with TLS termination

-   TLS is terminated at the Gateway
-   Gateway presents a certificate and decrypts traffic
-   Backend services receive plain HTTP

#####  Traffic flow

Client → Gateway (HTTPS) → Service (HTTP)

#####  Used for
-   Centralized certificate management
-   HTTPS enforcement at the edge
-   Hostname-based routing (bookinfo.cilium.rocks)
----------

###  3) TLSRoute (TLS passthrough / SNI routing)
-   Gateway does not terminate TLS
-   TLS remains encrypted end-to-end
-   Routing is done using SNI during TLS handshake

#####  Traffic flow
Client → Gateway (HTTPS) → Service (HTTPS)

#####  Used for
-   End-to-end encryption
-   Legacy TLS services
-   When the Gateway must not inspect HTTP traffic

----------

###  4) Traffic splitting (weighted routing)
-   Native weighted load balancing at L7
-   No service mesh, no sidecars, no extra tooling

#####  Traffic flow
Client → Gateway → Service A (50%)  
 Service B (50%)

#####  Used for
-   Canary releases
-   Blue/green deployments
-   A/B testing

----------
#### Operational notes

-   Cilium runs the L7 proxy (Envoy) per node, not per pod
-   eBPF accelerates packet handling and shortens the datapath

Gateway API cleanly separates:
-   **Infrastructure ownership (Gateway / GatewayClass)**
-   **Application routing (Routes)**

This makes routing portable across controllers and avoids Ingress-specific behavior.

----------

###  Applying and observing
```bash
kubectl apply -f manifests/<routing-type>/  
kubectl get gateway  
kubectl get httproute  
kubectl get tlsroute
```

:smile: **Bonus** : To retrieve the Gateway IP : ```kubectl get gateway <name> -o jsonpath='{.status.addresses[0].value}'```

## Apply the Bookinfo application

You can deploy the same application used in the tests directly from the Istio project:

```kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/bookinfo/platform/kube/bookinfo.yaml```

Verify that all pods are running: ```kubectl get pods```

*All services are exposed internally (ClusterIP). External access is provided only via the Gateway API resources defined in this repository.*

