# kubernetes_roles

Manages Kubernetes RBAC (roles, role bindings, service accounts).

## Subtasks

The role runs in three phases. Each can be toggled independently via a boolean variable (all default to `true`):

| Variable | Task | Description |
|---|---|---|
| `update_roles` | `update_roles.yml` | Create/remove ClusterRole resources |
| `update_serviceaccounts` | `update_serviceaccounts.yml` | Create ServiceAccounts and token secrets |
| `update_role_bindings` | `update_role_bindings.yml` | Create/delete RoleBindings (Kyverno and native) |

To skip a phase: `-e update_kubeconfigs=false`.

## Required Variables

| Variable | Type | Description |
|---|---|---|
| `kubeconfig` | string | Path to the admin kubeconfig used to authenticate with the cluster |
| `present_users` | list | List of users whose service accounts, tokens, and bindings to manage |
| `kubernetes_roles` | list | List of ClusterRole definitions to reconcile |
| `permissions` | dict | Maps user keys to their Kubernetes permission configs |

## Variable Templates

### `kubernetes_roles`

```yaml
kubernetes_roles:
  - name: "cluster-reader"
    hosts: "cluster-a:cluster-b"
    labels:
      team: "platform"
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list", "watch"]
```

### `permissions`

```yaml
permissions:
  my_user:
    kubernetes:
      - hosts: "cluster-a"
        role: "cluster-reader"
        namespaces: ["default", "app"]        # Optional — triggers Kyverno ClusterPolicy
        namespace_excludes: ["kube-system"]    # Optional
```

### `present_users`

```yaml
present_users:
  - name: "my_user"
    serviceaccount_name: "my-user-sa"           # Optional, defaults to `name`
    permissions: "{{ permissions.my_user }}"
```

## Optional Variables

| Variable | Type | Default | Description |
|---|---|---|---|
| `proxy_url` | string | *unset* | Proxy URL for cluster access |
| `proxy_headers` | dict | *unset* | Extra proxy headers |
| `generate_new_token` | bool | `false` | Force rotation of existing service account tokens |
| `serviceaccount_namespace` | string | *required* | Namespace for the service account |

## Example Usage

```yaml
---
- name: Configure Kubernetes RBAC
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - kubernetes_roles
  vars:
    kubeconfig: "/path/to/admin-kubeconfig"
    serviceaccount_namespace: "default"
    update_kubeconfigs: false
    kubernetes_roles:
      - name: "readonly"
        hosts: "prod-cluster"
        rules:
          - apiGroups: [""]
            resources: ["pods", "services"]
            verbs: ["get", "list", "watch"]
    present_users:
      - name: "monitoring"
        serviceaccount_name: "monitoring-sa"
        permissions: "{{ permissions.monitoring }}"
    permissions:
      monitoring:
        kubernetes:
          - hosts: "prod-cluster"
            role: "readonly"
```
