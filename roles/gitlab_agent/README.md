
# Token creation
Follow these steps in GitLab to create a token:
* Gitlab-Repository -> Operate -> Kubernetes clusters
* Connect a cluster
* Select a Name
* Token is shown (`kas_address` can also be seen in the provided sample helm values)


# Sample Config

```yaml
gitlab_agent_instances:
  - kas_address: "wss://gitlab.example.com//-/kubernetes-agent/"
    token: "glagent-asdf"
    release_name: "agent-1"
    namespace: "gitlab-agent-test"
    rbac_create: true
  - kas_address: "wss://gitlab.example.com//-/kubernetes-agent/"
    token: "glagent-123asdf"
    release_name: "agent-2"
    namespace: "gitlab-agent-test"
    rbac_create: true

```
