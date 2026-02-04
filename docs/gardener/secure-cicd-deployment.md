# Secure CI/CD Deployment with OIDC

Deploy to your Gardener cluster without storing secrets. This guide uses OIDC (OpenID Connect) tokens instead of static kubeconfig files.

## Why use OIDC instead of static kubeconfig?

Traditional CI/CD stores a kubeconfig secret that grants cluster access. This has problems:

| Problem | Static Kubeconfig | OIDC Tokens |
|---------|-------------------|-------------|
| **Lifetime** | Never expires | Expires in 1 hour |
| **If leaked** | Attacker has permanent access | Token expires quickly |
| **Scope** | Works from anywhere | Only works from your CI/CD |
| **Rotation** | Manual | Automatic every pipeline |
| **Audit** | "service-account did X" | "project-123/main-branch did X" |

With OIDC, your CI/CD provider (GitLab, GitHub, etc.) acts as an identity provider. Kubernetes validates the identity and grants short-lived access.

## How it works

```
CI/CD Provider                    Kubernetes
(GitLab/GitHub)                   (Gardener Shoot)

1. Pipeline starts
   ↓
2. CI generates signed JWT token
   (contains: project ID, branch, user)
   ↓
3. kubectl sends token ──────────► 4. Validates signature
                                      (using CI provider's public keys)
                                   ↓
                                   5. Checks token claims
                                      (is this from allowed project?)
                                   ↓
                                   6. Maps to username
                                      e.g., "gitlab:41:18:branch:main"
                                   ↓
                                   7. Checks RBAC permissions
                                   ↓
8. Success/Denied ◄────────────────
```

## Setup Overview

You need three things:

1. **Authentication ConfigMap** (Garden cluster) - Tells the Shoot how to validate tokens
2. **RBAC** (Shoot cluster) - Grants permissions to specific projects/branches
3. **CI/CD Pipeline** - Requests OIDC token and uses it

## Step 1: Authentication ConfigMap

Apply to the **Garden cluster**. This configures your Shoot to trust your CI/CD provider.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cicd-oidc-auth-config
  namespace: garden-<your-project>
data:
  config.yaml: |
    apiVersion: apiserver.config.k8s.io/v1beta1
    kind: AuthenticationConfiguration
    jwt:
    - issuer:
        url: <issuer-url>
        audiences:
        - <your-kubernetes-api-url>
        audienceMatchPolicy: MatchAny
      claimMappings:
        username:
          expression: <username-expression>
      claimValidationRules:
      - expression: <validation-expression>
        message: "validation failed"
```

### Provider-specific settings

**GitLab (self-hosted)**:
```yaml
issuer:
  url: https://gitlab.example.com
claimMappings:
  username:
    expression: "'gitlab:' + string(claims.namespace_id) + ':' + string(claims.project_id) + ':' + claims.ref_type + ':' + claims.ref"
claimValidationRules:
- expression: 'claims.namespace_id == "<your-group-id>"'
```

**GitHub**:
```yaml
issuer:
  url: https://token.actions.githubusercontent.com
claimMappings:
  username:
    expression: "'github:' + claims.repository_owner_id + ':' + claims.repository_id + ':' + claims.sub"
claimValidationRules:
- expression: 'claims.repository_owner_id == "<your-org-id>"'
```

## Step 2: Enable on Shoot

Add to your `shoot.yaml`:

```yaml
spec:
  kubernetes:
    kubeAPIServer:
      structuredAuthentication:
        configMapName: cicd-oidc-auth-config
```

## Step 3: RBAC in Shoot Cluster

Apply to the **Shoot cluster**. Grants permissions to specific CI/CD identities.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: cicd-deployer
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch", "create", "patch", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-deployer-binding
  namespace: default
subjects:
  - kind: User
    name: "<mapped-username>"  # e.g., gitlab:41:18:branch:main
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: cicd-deployer
  apiGroup: rbac.authorization.k8s.io
```

## Step 4: CI/CD Pipeline

**GitLab CI** (`.gitlab-ci.yml`):
```yaml
deploy:
  image: bitnami/kubectl:latest
  id_tokens:
    KUBE_TOKEN:
      aud: https://api.<shoot>.<project>.gardener.leaf.cloud
  script:
    - kubectl config set-cluster shoot --server=${KUBE_API_SERVER} --insecure-skip-tls-verify=true
    - kubectl config set-credentials ci --token=${KUBE_TOKEN}
    - kubectl config set-context default --cluster=shoot --user=ci
    - kubectl config use-context default
    - kubectl apply -f deployment.yaml
```

**GitHub Actions**:
```yaml
jobs:
  deploy:
    permissions:
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - name: Get OIDC token
        run: |
          TOKEN=$(curl -s -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
            "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=https://api.<shoot>.<project>.gardener.leaf.cloud" | jq -r '.value')
          echo "KUBE_TOKEN=$TOKEN" >> $GITHUB_ENV
      - name: Deploy
        run: |
          kubectl config set-cluster shoot --server=$KUBE_API_SERVER --insecure-skip-tls-verify=true
          kubectl config set-credentials ci --token=$KUBE_TOKEN
          kubectl config set-context default --cluster=shoot --user=ci
          kubectl config use-context default
          kubectl apply -f deployment.yaml
```

## Security controls

### Restrict by branch
Only allow deployments from `main`:
```yaml
name: "gitlab:41:18:branch:main"
```

### Restrict by namespace
Role and RoleBinding are namespace-scoped. Create them only in namespaces where deployment is allowed.

### Restrict by project
The `claimValidationRules` in the ConfigMap can enforce which projects are allowed:
```yaml
- expression: 'claims.project_id == "18"'
  message: "only project 18 is allowed"
```

## Debugging

To see what's in your OIDC token, decode it in your pipeline:

```bash
echo "$KUBE_TOKEN" | cut -d. -f2 | tr '_-' '/+' | base64 -d | jq .
```

This shows the claims that Kubernetes uses for validation and username mapping.

## References

- [Gardener Structured Authentication](https://gardener.cloud/docs/gardener/shoot/shoot-structured-authentication/)
- [GitLab CI/CD OIDC](https://docs.gitlab.com/ee/ci/secrets/id_token_authentication.html)
- [GitHub Actions OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
