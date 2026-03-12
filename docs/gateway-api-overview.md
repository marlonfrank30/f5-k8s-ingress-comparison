# 🌐 Gateway API Overview — Why It Matters for F5 Solutions

The Kubernetes Gateway API is the CNCF-standardised successor to the Ingress API. Understanding it is essential for evaluating NGF and BNK.

---

## What is the Gateway API?

The Gateway API is a set of Kubernetes CRDs developed by the SIG-Network community as the official next generation of Kubernetes traffic management. It is:

- **Role-oriented** — designed for three distinct personas
- **Expressive** — supports L4 and L7, multi-protocol, multi-cluster
- **Portable** — conformant implementations are interchangeable
- **Extensible** — policy attachments allow vendor extensions without core changes

---

## Core Resources

```
┌────────────────────────────────────────────────────────┐
│  GatewayClass                                          │
│  (Who manages this class of Gateway?)                  │
│  → Managed by: Infrastructure Provider                 │
└───────────────────────┬────────────────────────────────┘
                        │
┌───────────────────────▼────────────────────────────────┐
│  Gateway                                               │
│  (What listens? On which ports and protocols?)         │
│  → Managed by: Cluster Operator                        │
└──────────┬──────────────────────────┬──────────────────┘
           │                          │
┌──────────▼──────────┐  ┌───────────▼──────────────────┐
│  HTTPRoute          │  │  GRPCRoute / TLSRoute /       │
│  (L7 HTTP routing)  │  │  TCPRoute / UDPRoute          │
│  → App Developer    │  │  → App Developer              │
└─────────────────────┘  └──────────────────────────────┘
```

---

## The Three-Role Model

This is one of the most important design decisions in the Gateway API — and it directly maps to how NGF and BNK are architected.

| Role | Who | Manages |
|---|---|---|
| **Infrastructure Provider** | Cloud/platform team | `GatewayClass` — defines the implementation |
| **Cluster Operator** | Platform/SRE team | `Gateway` — defines listeners, ports, TLS certs |
| **Application Developer** | Dev team | `HTTPRoute`, `GRPCRoute` etc. — defines routing rules |

This separation means:
- Application developers **cannot** change listener ports or TLS certificates
- Cluster operators **cannot** change the underlying gateway implementation
- Each role operates in their own Kubernetes namespace with appropriate RBAC

---

## Route Resources

### HTTPRoute
The primary L7 routing resource. Supports host, path, header, query parameter, and method-based matching.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
spec:
  parentRefs:
  - name: prod-gateway
    namespace: nginx-gateway
  hostnames:
  - myapp.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api/v2
    - headers:
      - name: X-Region
        value: us-east
    filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
        - name: X-Forwarded-For
          value: "%REQ(X-Forwarded-For)%"
    backendRefs:
    - name: api-service-v2
      port: 8080
      weight: 90
    - name: api-service-v2-canary
      port: 8080
      weight: 10
```

### GRPCRoute
Native gRPC routing — no annotation workarounds needed.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GRPCRoute
metadata:
  name: grpc-route
spec:
  parentRefs:
  - name: prod-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - method:
        type: Exact
        service: com.example.UserService
        method: GetUser
    backendRefs:
    - name: user-service
      port: 9090
```

### TLSRoute
For TLS passthrough (SNI-based routing without termination).

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TLSRoute
metadata:
  name: tls-passthrough
spec:
  parentRefs:
  - name: prod-gateway
    sectionName: tls-passthrough-listener
  rules:
  - backendRefs:
    - name: my-tls-backend
      port: 443
```

---

## Cross-Namespace Routing

A critical feature for multi-tenant platforms — `ReferenceGrant` allows resources in one namespace to reference resources in another.

```yaml
# In the backend namespace — grants permission
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-from-app-ns
  namespace: backend-services
spec:
  from:
  - group: gateway.networking.k8s.io
    kind: HTTPRoute
    namespace: app-team-a
  to:
  - group: ""
    kind: Service
```

---

## Policy Attachment

Policy Attachment is the extension mechanism. It allows implementations to add capabilities (WAF, rate limiting, auth) without modifying core Gateway API resources.

```yaml
# NGF / BNK — rate limit policy attached to a route
apiVersion: gateway.nginx.org/v1alpha1
kind: ObservabilityPolicy
metadata:
  name: trace-all
spec:
  targetRefs:
  - group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: my-app-route
  tracing:
    strategy: ratio
    ratio: 100
```

---

## Gateway API Conformance

Both NGF and BNK are conformant with Gateway API. This means:

- You can verify conformance with `gwctl`
- HTTPRoute definitions written for NGF will work on any other conformant implementation
- You are not locked in to F5-specific annotations

To verify conformance:
```bash
# Run the official conformance test suite
go run sigs.k8s.io/gateway-api/conformance/... \
  --gateway-class=nginx \
  --supported-features=HTTPRoute,GRPCRoute,TLSRoute
```

---

## Ingress API vs Gateway API

| Aspect | Ingress API | Gateway API |
|---|---|---|
| CNCF Status | Stable (frozen) | Active development |
| Role model | Single (admin) | Three roles |
| Protocol support | HTTP/HTTPS only | HTTP, HTTPS, TCP, UDP, gRPC, TLS |
| L7 features | Annotations (fragile) | Native spec fields |
| Portability | Implementation-specific annotations | Conformance-tested portability |
| Multi-tenancy | Limited | First-class |
| Cross-namespace | Not native | ReferenceGrant |
| Policy extension | Annotations | Policy Attachment |
| Future | Maintenance only | Active SIG-Network investment |

---

## Migration Tool: ingress2gateway

The `ingress2gateway` tool converts Ingress resources and annotations to Gateway API resources. It supports multiple providers including NGINX.

```bash
# Install
go install sigs.k8s.io/ingress2gateway@latest

# Convert
ingress2gateway print \
  --input-file ./ingress-resources.yaml \
  --providers nginx \
  --output-format gateway-api

# Preview what would change
ingress2gateway diff \
  --providers nginx \
  --namespace production
```

The tool handles:
- Host and path rules → HTTPRoute `matches`
- TLS → Gateway `listeners` and Secret references
- Canary annotations → HTTPRoute `weight`
- Rewrite annotations → HTTPRoute `filters`
- Rate limit annotations → RateLimit policy (where supported)
