**Why Do We Need Vertical Pod Autoscaling (VPA)?**

Vertical Pod Autoscaler (VPA) automatically adjusts the CPU and memory (RAM) requests and limits for pods based on their actual usage over time.

This is useful because:
* Manually setting resources is hard and error-prone.
* Overprovisioning wastes resources.
* Underprovisioning causes performance issues or OOM kills.

**Scenario: E-commerce Application**

You have a product catalog microservice running in Kubernetes:
* During normal hours, it needs 100m CPU and 256Mi memory.
* During peak sales (Black Friday), it spikes to 500m CPU and 1Gi memory.
* You initially request 500m CPU and 1Gi memory to be safe but this is wasted 90% of the time.

With Only HPA:

* Traffic increases → HPA adds pods → handles load.
* But pods are overprovisioned → still wasteful.

With Only VPA:

* Single pod becomes bigger → but can’t handle more traffic alone → slow performance.

With HPA + VPA:

* During peak hours:
    * HPA increases pods from 3 → 6
    * VPA raises pod size from 100m/256Mi → 500m/1Gi

* During low hours:
    * HPA reduces to 2 pods
    * VPA lowers size to save memory

**VPA Modes**

Initial: VPA assigns resources only at Pod creation, without changes during its lifetime.

Auto: VPA assigns and updates resources during the Pod's lifetime, with options for eviction and rescheduling.

Off: VPA doesn't change Pod resources but sets recommended resources, useful for a "dry run"

**VPA Resource Recommendation Values**

In Kubernetes Vertical Pod Autoscaler (VPA), the terms ```lowerBound```, ```target```, ```uncappedTarget```, and ```upperBound``` refer to different resource recommendation values (typically CPU and memory) provided by the VPA for containers in a pod.

1. ```lowerBound```

    Definition: The minimum recommended resource request.

    Purpose: Ensures containers have enough resources to avoid under-provisioning.

    Calculated As: A safety margin below the target, often based on observed usage patterns and policy constraints.

    Use Case: Prevents the VPA from setting resource requests too low, which could lead to OOM (Out of Memory) errors or CPU throttling.

2. ```target```

    Definition: The ideal resource request for the container based on observed usage.

    Purpose: Represents the best guess for right-sizing a container to balance resource efficiency and application performance.

    Calculated As: Based on historical resource usage metrics collected by the VPA Recommender.

    Use Case: If no capping is applied, this is what the VPA would ideally set as the resource request.

3. ```uncappedTarget```

    Definition: The same as target, before applying any caps.

    Purpose: Shows the raw recommendation from VPA without enforcement of minAllowed or maxAllowed constraints.

    Use Case: Helps diagnose if the VPA's target has been limited by admin-defined resource boundaries.

4. ```upperBound```

    Definition: The maximum recommended resource request.

    Purpose: Prevents over-provisioning by capping how much CPU/memory a container should request.

    Calculated As: A safety margin above the target, considering historical peaks, spikes, and configured constraints.
    
    Use Case: Avoids allocating unnecessary resources which would reduce cluster efficiency.


```lowerBound```:	Safe minimum recommendation	

```target```:	Adjusted optimal recommendation	

```uncappedTarget```:	Raw optimal recommendation	

```upperBound```:	Safe maximum recommendation

**HPA vs VPA**

**HPA**

* Scales the number of pod replicas horizontally
* For handling increased load by adding more pods
* Supports CPU and memory metrics natively, custom metrics with Prometheus Adapter
* May over-provision resources, increasing costs
* Scaling **stateless** applications that handle varying traffic patterns

**VPA**

* Adjusts CPU and memory limits per pod vertically
* For optimizing resource requests per pod to prevent over/underutilization
* Supports CPU and memory metrics, does not support custom metrics
* Can cause pod disruptions during recreation, may lead to pending pods if resource limits exceed availability
* Scaling resource constrained **stateful** appplications for efficient resource usage
