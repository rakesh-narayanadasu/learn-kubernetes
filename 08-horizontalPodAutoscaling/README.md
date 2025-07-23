**Why Autoscale in K8S?**



* Improved application availability
* Efficient resource utilization
* Elasticity
* Fault tolerance and recovery
* Seamless load management



**Horizontal Pod Autoscaler**

HPA in Kubernetes is used to automatically scale the number of pods in a Deployment, StatefulSet, or ReplicaSet based on CPU/memory usage or custom metrics. Metrics server must be installed for HPA to work.
```
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=5
```
* Maintains 1 to 5 pods.
* If average CPU usage > 50%, scale up pods.
* If average CPU usage < 50%, scale down pods.

**How It Works (Step-by-Step)**
1. HPA controller wakes up every 15 seconds.
2. It queries the metrics server (via API server) for the average CPU/memory usage across all pods.
3. It compares actual usage to the target (e.g., 50% CPU).
4. Calculates the desired number of replicas using a formula:
```
desiredReplicas = currentReplicas × ( currentMetric / targetMetric )
```
5. If needed, it updates the deployment/replica set with the new replica count.
6. Kube-controller-manager starts or terminates pods accordingly.

**Metrics in K8S**

1. Native Metrics
* Also called: Resource metrics.
* Built-in support in HPA.
* Provided by metrics-server.

Examples:
CPU usage and Memory usage

```
# Use Case: Scale when CPU > 70%
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70

```

2. Custom Metrics
* Application-specific metrics (not built-in).
* Collected from Prometheus or other monitoring tools.
* Requires Custom Metrics API (e.g., via Prometheus Adapter).

Examples:
Request count, Queue length and Latency

You define the metric per pod or per workload.
```
# Use Case: Scale based on request rate per pod
metrics:
- type: Pods
  pods:
    metric:
      name: http_requests_per_second
    target:
      type: AverageValue
      averageValue: 100

```

3. External Metrics
* Metrics that are not tied to pods.
* Also uses External Metrics API (e.g., Prometheus Adapter).

Examples:
SQS queue length (AWS), Kafka lag, and Weather API

Useful for scaling based on external systems.
```
# Use Case: Scale if external queue size > 1000
metrics:
- type: External
  external:
    metric:
      name: external_queue_length
    target:
      type: Value
      value: 1000
```

**Multiple Metrics**

Using multiple metrics in Kubernetes provides granular control, better performance, and resilient scaling. It allows the platform to make smarter, more context-aware decisions rather than relying on a single signal like CPU.

Avoid multiple metrics when:
* Their correlation with the app's actual load is weak
* THe relationship is unclear or hard to define
* Adapter reliability is in question
* Lack of capacity to monitor and maintain adapters effectively
* Metrics are collected at different intervals

**HPA scaling policy**

In Kubernetes, the Horizontal Pod Autoscaler (HPA) scaling policy determines how quickly and by how much the number of pods is adjusted in response to metric changes.

The behavior field in the HPA spec lets you configure scaling direction, rate limits, and policies.
It has two subfields:
* ```scaleUp```: Policies when increasing replicas
* ```scaleDown```: Policies when decreasing replicas

Scaling Policies: How They Work
Each scaling direction (scaleUp or scaleDown) can define:
* ```stabilizationWindowSeconds```: How long to wait before applying a scaling decision.
* ```selectPolicy```: How to choose among multiple defined policies (Max, Min, Disabled).
* ```policies```: A list of rules limiting scaling velocity.

```
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
        - type: Percent
          value: 100
          periodSeconds: 60
      selectionPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
      selectionPolicy: Max
```

```scaleUp```:
* Immediate reaction (```stabilizationWindowSeconds: 0```)
* Can scale up by:
  * Max 2 pods per minute OR
  * Max 100% increase per minute
* ```selectPolicy: Max``` chooses the more aggressive option

```scaleDown```:
* Wait 5 minutes before applying a scale-down decision
* Reduce replicas by max 50% per minute
* Prevents flapping or aggressive scale-ins

**Default Behavior (If ```behavior``` is omitted)**

If you don’t specify ```behavior```, Kubernetes uses conservative defaults:
* No scaling faster than 4 pods per 60 seconds
* ```scaleDown``` has a 5-minute stabilization window
* ```selectPolicy``` defaults to Max
