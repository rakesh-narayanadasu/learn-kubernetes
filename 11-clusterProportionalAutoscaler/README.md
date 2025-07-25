**Cluster Proportional Autoscaler (CPA)**

Pod-level Autoscaling for System Components

What it does:
* Automatically adjusts the replica count of system or infrastructure pods (like CoreDNS, kube-proxy, metrics-server) based on the size of the cluster.
* Commonly used to scale system components proportionally to the number of nodes or cores in the cluster.
* Useful for things that serve the whole cluster, like DNS resolution (CoreDNS).

Use Case:
* Ensures system services don’t become a bottleneck as the cluster grows.
* You don't want a fixed number of DNS pods regardless of 10 vs. 1000 nodes.

Scenario:

Let’s say your cluster is growing due to high demand (CA is adding nodes). More workloads = more DNS queries. If you don’t increase CoreDNS pods, latency increases. CPA automatically scales CoreDNS replicas based on the number of nodes (e.g., 1 pod per 10 nodes), keeping DNS performance consistent.

**Cluster Autoscaler (CA)-Node-level Autoscaling**

What it does:
* Automatically adjusts the number of nodes in your cluster based on pending pods.
* If pods can’t be scheduled due to insufficient resources (CPU, memory), CA will add nodes.
* If nodes are underutilized and their pods can be moved elsewhere, CA can remove nodes.

Use Case:
* You're running workloads that vary over time like batch jobs or web traffic that spikes during the day.
* Helps reduce cost by scaling down when workloads are low.

Scenario:

Imagine you’re running an e-commerce site. During Black Friday, thousands of users come online, and new pods (web frontends, payment processors, etc.) need to be scheduled. The CA detects pending pods and adds nodes. After the sale ends, traffic drops, and CA removes unused nodes to save cost.

**When to Use Both?**

In a production-grade cluster, you often use both together:
* CA ensures you have enough nodes to run your workloads.
* CPA ensures that system components scale to support those workloads efficiently.

**CPA Scaling Modes**

The Cluster Proportional Autoscaler (CPA) supports two main scaling modes for determining how many replicas of a component (like CoreDNS) to run as the cluster grows:

1. Linear Mode
* Scales replicas linearly based on a formula.
```replicas = base + (nodes * nodesPerReplica)```
* You can configure scaling based on either:
    * Number of nodes
    * Number of cores

Note: It gives smooth, continuous scaling, ideal when resource usage scales evenly with cluster size.

2. Ladder Mode (Step Function Mode)
* You define a replica count for different cluster sizes.
* CPA chooses the right replica count based on which range the current number of nodes falls into.

Note: This gives step-wise scaling, which can be easier to control and 
tune precisely for known infrastructure sizes.

Which One Should You Use?
* Use Linear mode if your workloads scale predictably and gradually with cluster size.
* Use Ladder mode if you want explicit control over when and how many replicas are deployed at specific cluster sizes.

