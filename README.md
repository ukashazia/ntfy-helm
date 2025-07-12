# Ntfy Helm Chart

This Helm chart deploys the [ntfy](https://ntfy.sh/) notification service on a Kubernetes cluster. It supports multiple deployment options: Deployment, StatefulSet, or Pod, with a Service to expose the application.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.2.0+
- A ConfigMap named `ntfy` for configuration (optional, but enabled by default)

## Installation

1. Clone or download the chart to your local system:
   ```bash
   git clone <repository-url>
   cd ntfy-chart
   ```

2. Customize the `values.yaml` file to configure the deployment (see [Configuration](#configuration) below).

3. Install the chart:
   ```bash
   helm install ntfy ./ntfy-chart -f values.yaml
   ```

4. To upgrade an existing release:
   ```bash
   helm upgrade ntfy ./ntfy-chart -f values.yaml
   ```

5. To uninstall the chart:
   ```bash
   helm uninstall ntfy
   ```

## Chart Structure

The chart includes the following files:
- `Chart.yaml`: Chart metadata.
- `values.yaml`: Default configuration values.
- `templates/deployment.yaml`: Template for a Deployment (default).
- `templates/statefulset.yaml`: Template for a StatefulSet (disabled by default).
- `templates/pod.yaml`: Template for a single Pod (disabled by default).
- `templates/service.yaml`: Template for a Service to expose ntfy.
- `templates/_helpers.tpl`: Helper templates for labels and selectors.

## Configuration

The following table lists the configurable parameters in `values.yaml`:

| Key | Description | Default |
|-----|-------------|---------|
| `replicaCount` | Number of replicas for Deployment/StatefulSet | `1` |
| `image.repository` | Container image repository | `binwiederhier/ntfy` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | Image tag | `latest` |
| `resources.limits.cpu` | CPU limit for the container | `500m` |
| `resources.limits.memory` | Memory limit for the container | `128Mi` |
| `service.type` | Kubernetes Service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `configMap.enabled` | Enable ConfigMap volume mount | `true` |
| `configMap.name` | Name of the ConfigMap | `ntfy` |
| `deployment.enabled` | Enable Deployment | `true` |
| `statefulSet.enabled` | Enable StatefulSet | `false` |
| `statefulSet.cache.enabled` | Enable persistent cache volume | `true` |
| `statefulSet.cache.storage` | Storage size for cache volume | `1Gi` |
| `pod.enabled` | Enable single Pod | `false` |

### Example `values.yaml` for StatefulSet
To deploy ntfy as a StatefulSet with persistent storage (disabling Deployment):
```yaml
deployment:
  enabled: false
statefulSet:
  enabled: true
  cache:
    enabled: true
    storage: 1Gi
replicaCount: 1
image:
  repository: binwiederhier/ntfy
  pullPolicy: IfNotPresent
  tag: "latest"
resources:
  limits:
    cpu: "500m"
    memory: "128Mi"
service:
  type: ClusterIP
  port: 80
configMap:
  enabled: true
  name: ntfy
pod:
  enabled: false
```

## Usage Notes

- **Deployment vs. StatefulSet vs. Pod**:
  - Use `Deployment` for stateless deployments (default).
  - Use `StatefulSet` for stateful applications requiring persistent storage (e.g., cache.db).
  - Use `Pod` for single-instance, non-replicated deployments (e.g., testing).
  - Enable only one of `deployment.enabled`, `statefulSet.enabled`, or `pod.enabled` at a time to avoid conflicts.

- **ConfigMap**:
  - A ConfigMap named `ntfy` is expected for configuration files mounted at `/etc/ntfy`.
  - Create the ConfigMap before deploying the chart if `configMap.enabled` is `true`.

- **Accessing the Service**:
  - The Service exposes ntfy on port 80 by default.
  - Use `kubectl port-forward` or an Ingress to access the application externally.

## Example: Accessing ntfy
1. Port-forward the service:
   ```bash
   kubectl port-forward svc/ntfy 8080:80
   ```
2. Access ntfy at `http://localhost:8080`.

## Upgrading the Chart
To update the chart with new configurations:
```bash
helm upgrade ntfy ./ntfy-chart -f values.yaml
```

## Uninstallation
To remove the chart and all associated resources:
```bash
helm uninstall ntfy
```

## Notes
- The StatefulSet includes a PersistentVolumeClaim for cache storage (`/var/cache/ntfy/cache.db`) when enabled.
- Ensure the `ntfy` ConfigMap exists if `configMap.enabled` is `true`.
- Adjust resource limits and storage size based on your cluster capacity.

For more information about ntfy, visit the [official documentation](https://ntfy.sh/docs/).
