# common

![Version: 1.2.9](https://img.shields.io/badge/Version-1.2.9-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.2.9](https://img.shields.io/badge/AppVersion-1.2.9-informational?style=flat-square)

A comprehensive, production-ready Helm chart for deploying applications on Kubernetes

**Homepage:** <https://github.com/wasilak/common-chart>

## Overview

The **Common Chart** is a comprehensive, production-ready Helm chart for deploying applications on Kubernetes. It provides a flexible, battle-tested foundation for deploying Deployments, StatefulSets, and DaemonSets with sensible defaults and extensive customization options.

## Features

- **Multiple Workload Types**: Support for Deployment, StatefulSet, and DaemonSet configurations
- **Security First**: Pod Security Context, Container Security Context, and RBAC support
- **Storage Management**: Persistent volumes, ConfigMaps, and EmptyDir volumes
- **Advanced Networking**: Ingress support with domain redirect middleware for Traefik
- **Observability**: ServiceMonitor integration for Prometheus and distributed tracing (Jaeger, Datadog)
- **Authentication**: Forward auth with Authelia support
- **Resource Management**: CPU/Memory limits and requests with Pod Disruption Budgets
- **Health Checks**: Startup, Liveness, and Readiness probes
- **Environment Configuration**: Flexible environment variables and secrets management
- **Container Lifecycle**: Pre-stop and post-start hooks
- **Sidecar Support**: Logging sidecars and custom sidecars
- **HTTPS Ready**: Built-in HTTPS redirect support

## Installation

### Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- kubectl configured to access your cluster

### Quick Start

Add the Helm repository and install the chart:

```bash
# Add repository (when OCI publishing is complete)
helm repo add common-chart oci://ghcr.io/wasilak/common-chart

# Install the chart
helm install my-app common-chart/common \
  --set image.repository=my-image \
  --set image.tag=v1.0.0
```

### Using Custom Values

Create a `values.yaml` file with your custom configuration:

```yaml
image:
  repository: my-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent

deployment:
  replicaCount: 3

service:
  type: LoadBalancer

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

ingress:
  hosts:
    - host: my-app.example.com
      paths:
        - path: /
          pathType: Prefix
```

Then install with your custom values:

```bash
helm install my-app common-chart/common -f values.yaml
```

## Configuration Examples

### Enable High Availability

```yaml
deployment:
  replicaCount: 3
  strategy: RollingUpdate

podDisruptionBudget:
  enabled: true
  minAvailable: 1

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - my-app
          topologyKey: kubernetes.io/hostname
```

### Enable Resource Limits

```yaml
resources:
  limits:
    cpu: "1000m"
    memory: "1Gi"
  requests:
    cpu: "500m"
    memory: "512Mi"
```

### Enable RBAC and ServiceAccount

```yaml
rbac:
  create: true
  rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]

serviceAccount:
  create: true
  name: my-app
```

### Configure Storage

```yaml
storage:
  persistence:
    - name: data
      enabled: true
      size: 10Gi
      storageClassName: fast-ssd
      accessModes:
        - ReadWriteOnce

volumeMounts:
  - name: data
    mountPath: /data
```

### Enable Health Checks

```yaml
livenessProbeEnabled: true
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 3

readinessProbeEnabled: true
readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3
```

## Access Instructions

### ClusterIP Service (Default)

For ClusterIP service access, use port-forward:

```bash
kubectl port-forward svc/my-app 8080:80
# Access the application at http://localhost:8080
```

### NodePort Service

For NodePort service access, get the node IP and port:

```bash
kubectl get svc my-app -o jsonpath='{.status.nodePort}'
# Access at http://<node-ip>:<nodePort>
```

### LoadBalancer Service

For LoadBalancer service access, get the external IP:

```bash
kubectl get svc my-app -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
# Access at http://<external-ip>
```

### Ingress

For Ingress access, ensure the hostname is configured and accessible:

```bash
kubectl get ingress my-app
# Access at https://my-app.example.com (or configured hostname)
```

## Upgrading

To upgrade an existing release:

```bash
helm upgrade my-app common-chart/common -f values.yaml
```

## Uninstalling

To uninstall the chart:

```bash
helm uninstall my-app
```

## RBAC

This chart can optionally create RBAC resources (ServiceAccount, ClusterRole, ClusterRoleBinding) to grant the application necessary permissions.

To enable RBAC:

```yaml
rbac:
  create: true
  rules:
    - apiGroups: [""]
      resources: ["pods", "services"]
      verbs: ["get", "list", "watch"]
```

## Troubleshooting

### Pod not starting

Check pod status and logs:

```bash
kubectl get pods -l app=my-app
kubectl logs -f deployment/my-app
kubectl describe pod <pod-name>
```

### Service not accessible

Verify service configuration:

```bash
kubectl get svc my-app
kubectl get endpoints my-app
```

### Health check failures

Check probe configuration and application logs:

```bash
kubectl logs -f deployment/my-app
kubectl describe pod <pod-name>
```

### Image pull errors

Verify image repository and pull secrets:

```yaml
image:
  repository: your-registry/image
  pullSecrets:
    - name: registry-credentials
```

### Persistent volume issues

Check PVC and storage class:

```bash
kubectl get pvc
kubectl get storageclass
kubectl describe pvc <pvc-name>
```

## Support

For issues, feature requests, or questions:

- **GitHub Issues**: [wasilak/common-chart](https://github.com/wasilak/common-chart/issues)
- **GitHub Discussions**: [wasilak/common-chart](https://github.com/wasilak/common-chart/discussions)

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| wasilak | <piotr.m.boruc@gmail.com> |  |

## Source Code

* <https://github.com/wasilak/common-chart>

## Chart Values

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| type | string | `"deployment"` |  |
| deployment.replicaCount | int | `1` |  |
| deployment.strategy | string | `"RollingUpdate"` |  |
| statefulset.replicaCount | int | `1` |  |
| statefulset.podManagementPolicy | string | `"OrderedReady"` |  |
| statefulset.updateStrategy.type | string | `"RollingUpdate"` |  |
| statefulset.updateStrategy.rollingUpdate.partition | int | `0` |  |
| statefulset.volumeClaimTemplates | list | `[]` |  |
| daemonset.updateStrategy.type | string | `"RollingUpdate"` |  |
| daemonset.updateStrategy.rollingUpdate.maxUnavailable | int | `1` |  |
| domainRedirect | list | `[]` |  |
| customSubdomain | string | `""` |  |
| labels | object | `{}` |  |
| podLabels | object | `{}` |  |
| annotations | object | `{}` |  |
| podAnnotations | object | `{}` |  |
| nodeSelector | object | `{}` |  |
| tolerations | list | `[]` |  |
| affinity | object | `{}` |  |
| resources | object | `{}` |  |
| env | list | `[]` |  |
| envFrom | list | `[]` |  |
| storage.persistence | list | `[]` |  |
| volumeMounts | list | `[]` |  |
| volumes.emptyDir | list | `[]` |  |
| volumes.hostPath | list | `[]` |  |
| volumes.configMap | list | `[]` |  |
| volumes.secret | list | `[]` |  |
| initContainers | list | `[]` |  |
| lifecycle.preStop | object | `{}` |  |
| lifecycle.postStart | object | `{}` |  |
| sidecars | list | `[]` |  |
| configMaps | list | `[]` |  |
| podDisruptionBudget.enabled | bool | `false` |  |
| podDisruptionBudget.minAvailable | string | `nil` |  |
| podDisruptionBudget.maxUnavailable | int | `1` |  |
| rbac.create | bool | `false` |  |
| rbac.rules | list | `[]` |  |
| hostNetwork | bool | `false` |  |
| enableServiceLinks | bool | `true` |  |
| dnsPolicy | string | `"ClusterFirst"` |  |
| podSecurityContext.runAsUser | int | `1000` |  |
| podSecurityContext.runAsGroup | int | `3000` |  |
| podSecurityContext.fsGroup | int | `2000` |  |
| podSecurityContext.fsGroupChangePolicy | string | `"OnRootMismatch"` |  |
| containerSecurityContext.allowPrivilegeEscalation | bool | `false` |  |
| containerSecurityContext.readOnlyRootFilesystem | bool | `true` |  |
| containerSecurityContext.privileged | bool | `false` |  |
| containerSecurityContext.runAsNonRoot | bool | `true` |  |
| containerSecurityContext.runAsUser | int | `1000` |  |
| containerSecurityContext.runAsGroup | int | `3000` |  |
| containerSecurityContext.capabilities.drop[0] | string | `"ALL"` |  |
| containerSecurityContext.seccompProfile.type | string | `"RuntimeDefault"` |  |
| image.registry | string | `"docker.io"` |  |
| image.repository | string | `""` |  |
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.tag | string | `""` |  |
| image.digest | string | `""` |  |
| image.pullSecrets | list | `[]` |  |
| serviceAccount.create | bool | `false` |  |
| serviceAccount.name | string | `""` |  |
| serviceAccount.annotations | object | `{}` |  |
| service.type | string | `"ClusterIP"` |  |
| service.externalTrafficPolicy | string | `"Cluster"` |  |
| ports[0].name | string | `"http"` |  |
| ports[0].containerPort | int | `2000` |  |
| ports[0].servicePort | int | `2000` |  |
| ports[0].protocol | string | `"TCP"` |  |
| ports[0].ingress[0].host | string | `"example.com"` |  |
| ports[0].ingress[0].paths[0].path | string | `"/"` |  |
| ports[0].ingress[0].paths[0].pathType | string | `"Prefix"` |  |
| ports[0].scrape.enabled | bool | `false` |  |
| ports[0].scrape.path | string | `"/metrics"` |  |
| command | list | `[]` |  |
| args | list | `[]` |  |
| startupProbeEnabled | bool | `false` |  |
| startupProbe.httpGet.path | string | `"/health"` |  |
| startupProbe.httpGet.port | string | `"http"` |  |
| startupProbe.initialDelaySeconds | int | `0` |  |
| startupProbe.periodSeconds | int | `10` |  |
| startupProbe.timeoutSeconds | int | `1` |  |
| startupProbe.failureThreshold | int | `30` |  |
| startupProbe.successThreshold | int | `1` |  |
| livenessProbeEnabled | bool | `true` |  |
| livenessProbe.httpGet.path | string | `"/health"` |  |
| livenessProbe.httpGet.port | string | `"http"` |  |
| livenessProbe.initialDelaySeconds | int | `10` |  |
| livenessProbe.periodSeconds | int | `10` |  |
| livenessProbe.timeoutSeconds | int | `2` |  |
| livenessProbe.failureThreshold | int | `3` |  |
| livenessProbe.successThreshold | int | `1` |  |
| readinessProbeEnabled | bool | `true` |  |
| readinessProbe.httpGet.path | string | `"/health"` |  |
| readinessProbe.httpGet.port | string | `"http"` |  |
| readinessProbe.initialDelaySeconds | int | `5` |  |
| readinessProbe.periodSeconds | int | `5` |  |
| readinessProbe.timeoutSeconds | int | `2` |  |
| readinessProbe.failureThreshold | int | `3` |  |
| readinessProbe.successThreshold | int | `1` |  |
| auth.enabled | bool | `false` |  |
| auth.type | string | `"forwardauth-authelia"` |  |
| auth.oidc.secretName | string | `"external-secret"` |  |
| auth.url | string | `"http://authelia-otteryak-foo.kube-system.svc.cluster.local:9091/api/authz/forward-auth"` |  |
| https.enabled | bool | `true` |  |
| https.redirect | bool | `true` |  |
| externalSecret.enabled | bool | `false` |  |
| serviceMonitor.enabled | bool | `false` |  |
| serviceMonitor.namespace | string | `"observability"` |  |
| tracing.enabled | bool | `false` |  |
| tracing.jaeger.agentHost | string | `"jaeger-agent.observability.svc.cluster.local"` |  |
| tracing.jaeger.agentPort | int | `6831` |  |
| tracing.datadog.enabled | bool | `false` |  |
| tracing.datadog.agentHost | string | `"datadog-agent.datadog.svc.cluster.local"` |  |
| tracing.datadog.agentPort | int | `8126` |  |
| logging.sidecars | list | `[]` |  |

