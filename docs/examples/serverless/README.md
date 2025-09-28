# Serverless Configuration and Documentation

## Overview

This document provides a detailed guide on configuring serverless environments using Kubernetes, with a focus on integrating Prometheus for monitoring and KEDA for scaling. The configuration aims to ensure efficient resource utilization and seamless scaling of applications.

## Prometheus Configuration

Prometheus is used for monitoring and alerting. To enable cross-namespace ServiceMonitor discovery, use `namespaceSelector`. In Prometheus, define `serviceMonitorSelector` to associate with ServiceMonitors.

### ServiceMonitor Example

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-monitor
  namespace: example-namespace
  labels:
    app.kubernetes.io/name: servicemonitor
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      environment: production
  endpoints:
    - port: http
      path: /metrics
      scheme: http
```

### Important Notes

- Ensure that the `namespaceSelector` is set to allow cross-namespace monitoring.
- Label your services appropriately to be discovered by Prometheus.

## KEDA Configuration

KEDA (Kubernetes Event-driven Autoscaling) is used for scaling applications based on custom metrics. It can be integrated with Prometheus to trigger scaling actions.

### ScaledObject Example

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: example-scaler
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-deployment
  pollingInterval: 30  # Set a longer polling interval to avoid conflicts
  cooldownPeriod: 300
  minReplicaCount: 0
  maxReplicaCount: 10
  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://prometheus-operated.example-namespace.svc.cluster.local:9090
      metricName: custom_metric
      query: custom_metric_query
      threshold: "1"
```

### Important Notes

- Ensure that the `serverAddress` points to the correct Prometheus service.
- Adjust `pollingInterval` and `cooldownPeriod` to optimize scaling behavior and avoid conflicts with other scaling mechanisms.

## Integration with Activator

Consider integrating the serverless configuration with an activator for scale-from-zero scenarios. The activator can be implemented using a controller pattern or as a standalone goroutine.

### Controller Runtime Framework

Using the Controller Runtime framework can simplify the development of Kubernetes controllers. It provides abstractions for managing resources and handling events.

#### Key Components

1. **Controller**: Monitors resource states and triggers actions to align actual and desired states.
2. **Reconcile Function**: Core logic for transitioning resource states.
3. **Manager**: Manages the lifecycle of controllers and shared resources.
4. **Client**: Interface for interacting with the Kubernetes API.
5. **Scheme**: Registry for resource types.
6. **Event Source and Handler**: Define event sources and handling logic.

### Example Reconcile Function

```go
func (r *ExampleReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // Fetch the resource
    var resource corev1.Resource
    if err := r.Get(ctx, req.NamespacedName, &resource); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }

    // Implement reconciliation logic
    return r.reconcileResource(ctx, &resource)
}
```

## Conclusion

This configuration guide provides a comprehensive approach to setting up a serverless environment with Kubernetes, Prometheus, and KEDA. By following these guidelines, you can ensure efficient scaling and monitoring of your applications.