# Ansible role - `k8s_argocd`

Deploy and configure Argo CD on Kubernetes clusters.

## Tags

| Tag      | Description                 |
|----------|-----------------------------|
| `apply`  | Deploy and configure ArgoCD |
| `delete` | Remove ArgoCD               |

## Variables

### Generic variables

| Variable                      | Type     | Required | Default                                   | Description                                                             |
|-------------------------------|----------|----------|-------------------------------------------|-------------------------------------------------------------------------|
| `k8s_argocd_namespace`        | `string` | No       | `argocd`                                  | ArgoCD namespace                                                        |
| `k8s_argocd_version`          | `string` | No       | `1.7`                                     | ArgoCD release (version)                                                |
| `k8s_argocd_install_manifest` | `string` | No       | `install-{{ k8s_argocd_version }}.yml.j2` | Local ArgoCD installation manifest                                      |
| `k8s_argocd_admin_password`   | `string` | No       | `""` (empty string)                       | New ArgoCD admin password (will be encrypted with `bcrypt`); See bellow |
| `k8s_argocd_server_insecure`  | `bool`   | No       | `no`                                      | Run server withour SSL; See bellow                                      |
| `k8s_argocd_custom_manifests` | `list`   | No       | `[]` (empty list)                         | Custom manifest to apply / delete                                       |

Some variables may look dangerous or inconsistant (e.g. `k8s_argocd_server_insecure`). Explaination:

* `k8s_argocd_admin_password`: This variable defaults to `""` (an empty string) and thus does not change the default password. You may want to bind this variable to a password from a vault.
* `k8s_argocd_server_insecure`: If you are using `Istio`, `Traefik` or another proxy mechanism, This variable let you terminate the SSL on this proxy instead of ArgoCD API server.
* `k8s_argocd_custom_manifests`: This role let you deploy *helpers* manifests, currently an Istio `VirtualService`.


### A note about `k8s_argocd_custom_manifests`

You may deploy ArgoCD with external, *helpers* manifests to ease its integration into your K8S cluster.

#### Supported helpers manifests

| Manifest               | Description                                      |
|------------------------|--------------------------------------------------|
| `istio-virtualservice` | A simple yet functional Istio's `VirtualService` |

#### Usage

```yaml
# Select the helpers
k8s_argocd_custom_manifests: [ "istiovirtualservice" ]

# Configure the helpers
k8s_argocd_istio_vs_name: "..."
# ...
```

### Custom variables - Istio

| Variable                       | Type     | Required | Default                            | Description                                     |
|--------------------------------|----------|----------|------------------------------------|-------------------------------------------------|
| `k8s_argocd_istio_vs_name`     | `string` | No       | `argocd-vs`                        | Istio's `VirtualService` name                   |
| `k8s_argocd_istio_vs_gateways` | `list`   | No       | `[ "istio-system/istio-gateway" ]` | Istio's `Gateway`                               |
| `k8s_argocd_istio_vs_hosts`    | `list`   | No       | `[ "argocd" ]`                     | List of accepted hosts (e.g. `argocd.corp.tld`) |
| `k8s_argocd_istio_vs_service`  | `string` | No       | `argocd-server`                    | ArgoCD server service name                      |

## Templates

| Template                      | Description                                                                    |
|-------------------------------|--------------------------------------------------------------------------------|
| `install-<version>.yml.j2`    | ArgoCD installation manifest; Version in format `<major>.<minor>` (e.g. `1.7`) |
| `istio-virtualservice.yml.j2` | Istio's `VirtualService` template for quick AgroCD integration in Istio        |
