# Kubernetes DaemonSet: From Basic to Advance Deep Dive 🛡️🖥️

To understand **DaemonSet** in Kubernetes, let's use a simple analogy: **A CCTV Camera on every floor of an apartment building.**

---

## 1. What is a DaemonSet? (The One-on-Every-Node) 🛡️

**Simple Meaning:** A DaemonSet ensures that exactly one Pod runs on **every node (worker machine)** in your cluster.

If you add 5 new nodes to the cluster, the DaemonSet automatically starts one pod on each of those 5 nodes. If a node is deleted, its pod is automatically removed.

---

## 2. Why was it created? (The Problem it Solves) ❓

Imagine you need to collect **Logs** from every node.
*   If you use a **ReplicaSet** and ask for "5 pods," K8s might run all 5 on a single node.
*   This would miss logs from the other nodes.
*   We needed a system that focuses on **location (node)** rather than count. Thus, the **DaemonSet** was created.

---

## 3. Real World Use Cases 🛠️

In production, DaemonSets are almost always used for these 3 tasks:
1.  **Logging:** e.g., `Fluentd` or `Logstash`. These collect logs from every node and send them to a central server.
2.  **Monitoring:** e.g., `Prometheus Node Exporter` or `Datadog agent`. These monitor every node's CPU/RAM health.
3.  **Networking:** e.g., `kube-proxy` or `Calico/Flannel`. These manage networking on every node.

---

## 4. DaemonSet vs. ReplicaSet 📊

| Feature | ReplicaSet / Deployment 👯‍♂️ | DaemonSet 🛡️ |
| :--- | :--- | :--- |
| **Focus** | **Quantity:** 10 pods needed, run them anywhere. | **Quality:** 1 pod needed on every node. |
| **Scaling** | You scale pods manually or via HPA. | It scales automatically with the cluster size (nodes). |
| **Use Case** | Apps (Frontend, Backend, APIs). | Infrastructure Tools (Logging, Monitoring). |

---

## 5. Advance Concept: Controlling Pods 🚀

You don't always want a DaemonSet to run on *every* node.
*   **NodeSelector:** You can say "Run only on nodes with the label `SSD=true`."
*   **Taints & Tolerations:** Some nodes are special (like Master nodes) and don't accept normal pods. To run a DaemonSet there, we must add "Tolerations."

---

## 6. YAML Code (Production Grade) 📄

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-logging
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd-bin
  template:
    metadata:
      labels:
        name: fluentd-bin
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-logging
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

---

## 7. Line-by-Line Explanation of the YAML 🔍

*   **`apiVersion: apps/v1`**: We use this API version for DaemonSets as well.
*   **`kind: DaemonSet`**: We are telling K8s that this is a DaemonSet.
*   **`namespace: kube-system`**: Most DaemonSets are cluster-level tools, so they are placed in `kube-system`.
*   **`selector`**: Defines which pods the DaemonSet should monitor.
*   **`tolerations`**: This is the most important part. It means, "Even if the Master node has a restriction (Taint), go ahead and run there anyway."
*   **`resources`**: Since it runs on every node, setting RAM/CPU limits is crucial so it doesn't starve the main apps.

---

## Interview Questions (Q&A) 🎤

**1. Q: Can we use the `replicas` field in a DaemonSet?**
*   **Ans:** No! DaemonSets do not have a `replicas` field because the count is determined by the number of Nodes in the cluster, not by a specific number you provide.

**2. Q: How does "RollingUpdate" work in a DaemonSet?**
*   **Ans:** When you change the image, it deletes and recreates pods one by one. This is controlled via `spec.updateStrategy.type: RollingUpdate`.

**3. Q: Can we restrict a DaemonSet to specific nodes?**
*   **Ans:** Yes, using `nodeSelector` or `nodeAffinity`. Example: running a monitoring agent only on "GPU nodes."

**4. Q: How does a DaemonSet know when a new Node joins the cluster?**
*   **Ans:** The DaemonSet Controller constantly "watches" the API Server. As soon as a new Node is added, the Controller creates a new Pod for that node.

**5. Q: What is the role of "Taints and Tolerations" in a DaemonSet?**
*   **Ans:** By default, K8s doesn't schedule pods on Master nodes. If we want logs from the Master node, we add a "Toleration" to the DaemonSet so it can run there.

**6. Q: Can we manually delete a DaemonSet pod?**
*   **Ans:** Yes, but the DaemonSet Controller will immediately notice it's missing and recreate it on that node.

**7. Q: What is the difference between DaemonSet and StatefulSet?**
*   **Ans:** A DaemonSet runs one copy on every node. A StatefulSet gives pods a unique identity (e.g., pod-0, pod-1) and permanent storage, but they don't necessarily run on every node.

**8. Q: What is the "OnDelete" update strategy?**
*   **Ans:** In this strategy, pods aren't automatically updated when the image changes. A pod is only updated with the new image after you manually delete it. This is used for sensitive tools.

**9. Q: Who is responsible for scheduling DaemonSet pods?**
*   **Ans:** Previously, the DaemonSet Controller did it, but now (since K8s 1.12+), the **default-scheduler** handles it using `NodeAffinity`.

**10. Q: If a node runs out of resources (RAM/CPU), will the DaemonSet pod still run?**
*   **Ans:** DaemonSet pods are vital. We set their `priorityClassName` to high (`system-node-critical`) so that K8s will shut down other normal pods before stopping a DaemonSet.

**That's a DaemonSet!** It ensures your infrastructure tools are always present on every machine. 🛡️
