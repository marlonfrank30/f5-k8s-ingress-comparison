# 🔐 Security Comparison — F5 Kubernetes Ingress Solutions

This document covers the security capabilities of each solution in depth: WAF, mTLS, authentication, firewalling, and Zero Trust.

---

## Security Capability Matrix

| Capability | NIC | ingress-nginx | NGF | BNK | CIS |
|---|---|---|---|---|---|
| **TLS Termination** | ✅ | ✅ | ✅ | ✅ Hardware | ✅ Hardware |
| **mTLS (client + backend)** | ✅ Policy CRD | ✅ Annotations | ✅ BackendTLSPolicy | ✅ | ✅ C3D |
| **TLS Passthrough** | ✅ TransportServer | ⚠️ Annotation | ✅ TLSRoute | ✅ TLSRoute | ✅ TLSProfile |
| **SSL Re-encryption** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Hardware SSL Offload** | ❌ | ❌ | ❌ | ✅ DPU | ✅ FPGA/ASIC |
| **WAF** | 💰 App Protect | ⚠️ ModSecurity | 💰 F5 WAF | ✅ F5 Next WAF | ✅ Advanced WAF |
| **Bot Defense** | 💰 App Protect | ❌ | 💰 F5 WAF | ✅ | ✅ Advanced WAF |
| **API Security** | 💰 App Protect | ❌ | 💰 | ✅ | ✅ |
| **OWASP Top 10** | 💰 | ⚠️ Limited | 💰 | ✅ | ✅ |
| **IPS/IDS** | ❌ | ❌ | ❌ | ✅ F5-IPSD | ✅ AFM |
| **Stateful Firewall** | ❌ | ❌ | ❌ | ✅ F5BigFwPolicy | ✅ AFM |
| **DDoS Protection** | ✅ F5 stack | ⚠️ Basic | ✅ F5 stack | ✅ Full | ✅ Full |
| **IP Allow/Deny** | ✅ Policy CRD | ✅ Annotation | ✅ | ✅ F5BigFwPolicy | ✅ Policy + AFM |
| **Rate Limiting** | ✅ Policy CRD | ✅ Annotation | ✅ RateLimit policy | ✅ | ✅ Policy CRD |
| **JWT Validation** | ✅ OSS + Plus | ⚠️ External | 🔄 Roadmap | ✅ | ✅ APM + iRules |
| **OIDC / OAuth** | ✅ v5.3+ | ⚠️ External auth-url | 🔄 Roadmap | ✅ | ✅ APM full |
| **MFA / SSO** | ⚠️ Via OIDC | ❌ | 🔄 | ⚠️ Via OIDC | ✅ APM native |
| **Basic Auth** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **RBAC** | ✅ K8s RBAC | ✅ K8s RBAC | ✅ Enhanced 3-role | ✅ K8s + GW API | ✅ K8s + BIG-IP |
| **Zero Trust** | ✅ Policy CRD | ⚠️ Limited | ✅ Role model | ✅ | ✅ APM identity |
| **CORS** | ✅ | ✅ | ✅ Native HTTPRoute | ✅ | ✅ iRules |
| **Secret Management** | ✅ K8s Secrets | ✅ K8s Secrets | ✅ K8s Secrets | ✅ K8s Secrets | ✅ K8s + BIG-IP certs |

---

## WAF Deep-Dive

### NIC — NGINX App Protect WAF (Commercial)

NGINX App Protect WAF is available as an add-on to NGINX Plus. It is based on F5's Advanced WAF engine adapted for NGINX.

**Capabilities:**
- OWASP Top 10 protection
- gRPC payload inspection and schema validation
- OpenAPI schema-based positive security model
- Bot defense and signature-based detection
- L7 DoS protection
- CVE-based signature database with automatic updates

**Configuration:**
```yaml
apiVersion: k8s.nginx.org/v1
kind: Policy
metadata:
  name: waf-policy
spec:
  waf:
    enable: true
    apPolicy: "default/dataguard-alarm"
    securityLog:
      enable: true
      apLogConf: "default/logconf"
      logDest: "syslog:server=syslog-svc.default:514"
```

---

### NGF — F5 WAF (Commercial, NGINX One / NGINX Plus)

Same underlying F5 WAF engine, configured via Policy attachment resources in the Gateway API model.

---

### BNK — F5 BIG-IP Next WAF

Full F5 Advanced WAF engine running in-cluster alongside the TMM data plane.

**Additional BNK-specific capabilities:**
- **F5-IPSD (IPS):** SNORT 3-compatible deep packet inspection. Signature-based IDS/IPS for L3–L7 threats.
- **F5BigFwPolicy:** Stateful L3/L4 ACL enforcement with per-rule logging.
- **Per-subscriber visibility:** Billing and compliance telemetry at subscriber level.

```yaml
apiVersion: f5.f5bigip.bnk.io/v1alpha1
kind: BNKSecPolicy
metadata:
  name: app-security-policy
spec:
  wafPolicy:
    enable: true
    mode: blocking
    owasp: true
  rateLimitPolicy:
    requestsPerSecond: 1000
  ipFilter:
    denyList:
    - 192.168.100.0/24
```

---

### CIS — F5 Advanced WAF

Full F5 Advanced WAF exposed through the CIS Policy CRD. This is the same WAF engine available on BIG-IP hardware appliances and BIG-IP VE.

```yaml
apiVersion: cis.f5.com/v1
kind: Policy
metadata:
  name: waf-policy
spec:
  l7Policies:
    waf: /Common/my-asm-policy
  l3Policies:
    ip:
      allow:
        - 10.0.0.0/8
      deny:
        - 0.0.0.0/0
  iRules:
    insecure: /Common/my-irule
```

---

## mTLS Deep-Dive

### NIC — Policy CRD egressMTLS / ingressMTLS

NIC supports mTLS in both directions: verifying client certificates on inbound connections, and presenting client certificates when connecting to upstream services.

```yaml
apiVersion: k8s.nginx.org/v1
kind: Policy
metadata:
  name: mtls-policy
spec:
  ingressMTLS:
    clientCertSecret: client-ca-secret
    verifyClient: "on"
    verifyDepth: 2
  egressMTLS:
    tlsSecret: upstream-client-cert
    verifyServer: true
    serverName: true
    trustedCertSecret: upstream-ca-secret
```

### NGF — BackendTLSPolicy

NGF uses the standard Gateway API `BackendTLSPolicy` resource for backend mTLS.

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha3
kind: BackendTLSPolicy
metadata:
  name: backend-mtls
spec:
  targetRefs:
  - group: ""
    kind: Service
    name: my-backend-service
  validation:
    caCertificateRefs:
    - name: backend-ca-cert
      group: ""
      kind: Secret
    hostname: my-backend.internal.example.com
```

### CIS — C3D (Client Certificate Constrained Delegation)

CIS uses BIG-IP's C3D feature to delegate client certificate validation to BIG-IP while re-issuing certificates to backend services — enabling mTLS without requiring all backend pods to manage their own certificates.

---

## Zero Trust Architecture

### NIC + NGF — Policy-Enforced Zero Trust

NIC and NGF implement zero trust through layered Policy resources:
- Authentication: JWT / OIDC at the ingress point
- Authorization: IP allowlists, rate limiting
- Encryption: mTLS end-to-end
- Separation: Namespace-level isolation (NGF: ReferenceGrant)

### BNK — Zero-Trust Network Architecture

BNK enforces single ingress/egress points per tenant with full traffic inspection:
- F5BigFwPolicy as the default-deny stateful firewall
- BNKSecPolicy for per-tenant WAF, auth, and rate limiting
- F5-IPSD for deep packet inspection on all traffic
- DPU-level isolation in shared-server deployments

### CIS — Identity-Aware Zero Trust via BIG-IP APM

CIS exposes BIG-IP APM at the ingress layer for identity-aware access control:
- Per-user / per-group authorization policies
- OIDC federation with enterprise IdPs (Azure AD, Okta, Ping)
- MFA enforcement before traffic reaches Kubernetes services
- Session management and SSO across multiple applications
