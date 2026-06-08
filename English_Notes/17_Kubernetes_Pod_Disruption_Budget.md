# Kubernetes Pod Disruption Budget (PDB): The Safety Net 🛡️🚧

In Kubernetes, nodes sometimes go down for maintenance (upgrades, patching). When this happens, pods are evicted. **Pod Disruption Budget (PDB)** is your way of telling Kubernetes: "Hey, at any given time, I need at least X number of pods running for my app to stay healthy."

---

## 1. What is PDB? (The Party Analogy) 🥳

Think of a **Party** where you need at least **3 Security Guards** at all times to keep things safe.
If one guard needs a break (maintenance), they can only go if there are still 3 guards left. If only 3 are working, no one is allowed to leave until a replacement arrives.

In K8s, a **PDB** limits the number of Pods of a replicated application that are down simultaneously from voluntary disruptions.

---

## 2. Voluntary vs. Involuntary Disruptions 🌪️🛠️

To understand PDB, you must know the two types of "bad things" that happen to pods:

*   **Involuntary Disruptions (Unavoidable):**
    *   Hardware failure of the physical machine.
    *   Kernel panic.
    *   Cluster panic.
*   **Voluntary Disruptions (Planned):**
    *   `kubectl drain` a node for maintenance.
    *   Updating a deployment's pod template.
    *   Cluster autoscaler scaling down a node.

**Note:** PDB **ONLY** protects against **Voluntary Disruptions**. It cannot stop a hardware crash!

---

## 3. Why do we need PDB? 🤔

1.  **Availability:** Ensures your application always has enough replicas to handle traffic.
2.  **Quorum Protection:** For apps like Etcd or ZooKeeper, losing too many nodes at once can break the whole system (split-brain).
3.  **Controlled Upgrades:** It prevents the cluster administrator from accidentally killing too many pods while upgrading nodes.

---

## 4. How to Define PDB? (The Two Ways) 📏

You can specify your "budget" in two ways:

*   **minAvailable:** The minimum number of pods that must be up. (e.g., "I need at least 3 pods").
*   **maxUnavailable:** The maximum number of pods that can be down. (e.g., "Only 1 pod can be down at a time").

---

## 5. YAML Example (Production Grade) 📄

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  # Method 1: Minimum pods that must stay alive
  minAvailable: 2 
  
  # OR Method 2: Maximum pods that can be killed (Use only one)
  # maxUnavailable: 1 

  # Which pods does this apply to?
  selector:
    matchLabels:
      app: my-web-server
```

*Note: You can also use percentages, like `minAvailable: 50%`.*

---

## 6. What happens during a `kubectl drain`? 🚿

When an admin runs `drain`:
1.  K8s checks if there is a PDB for the pods on that node.
2.  If killing a pod violates the PDB (e.g., `minAvailable` is 2 and only 2 are running), the `drain` command will **fail** or hang.
3.  The admin must wait for a new pod to start on another node before the old one can be killed.

---

## 7. Advanced: Common Pitfalls ⚠️

1.  **Setting minAvailable = Total Replicas:** If you have 3 replicas and set `minAvailable: 3`, you can **never** drain a node. The `drain` will wait forever.
2.  **PDB for Single Replica Apps:** Don't use PDB for apps with `replicas: 1` unless you are okay with `drain` failing.
3.  **Selector Mismatch:** Always ensure the `selector` in your PDB matches the `labels` in your Deployment/StatefulSet.

---

## 🚀 Top 10 Production-Grade Interview Questions

1.  **What is a Pod Disruption Budget?**
    *   *Ans:* It defines the minimum number of pods that must be available during planned (voluntary) disruptions.

2.  **Does PDB protect against a Node crashing?**
    *   *Ans:* No. PDB only protects against voluntary disruptions like node draining or autoscaling.

3.  **What is the difference between `minAvailable` and `maxUnavailable`?**
    *   *Ans:* `minAvailable` sets the floor for healthy pods, while `maxUnavailable` sets the ceiling for pods that can be evicted.

4.  **What happens if a `kubectl drain` violates a PDB?**
    *   *Ans:* The eviction request is rejected, and the `drain` command will keep retrying or fail until the budget is met.

5.  **Can we use percentages in PDB?**
    *   *Ans:* Yes, e.g., `minAvailable: 80%` or `maxUnavailable: 10%`.

6.  **Why is PDB critical for Quorum-based applications?**
    *   *Ans:* Because losing a majority of nodes (quorum) can cause the entire database/service to fail. PDB ensures we never drop below that majority.

7.  **What happens if you set `maxUnavailable: 0`?**
    *   *Ans:* Kubernetes will not allow any pods to be voluntarily evicted. This effectively blocks node maintenance for those pods.

8.  **Can we have multiple PDBs for the same set of pods?**
    *   *Ans:* No, only one PDB can apply to a pod. If multiple PDBs match, the behavior is undefined (usually the first one created wins).

9.  **How do you check the status of a PDB?**
    *   *Ans:* Use `kubectl get pdb`. It shows `ALLOWED DISRUPTIONS`. If this is 0, no more pods can be killed.

10. **Does PDB work with Pods not managed by a Controller (naked pods)?**
    *   *Ans:* Yes, as long as the PDB selector matches the pod labels, but it's best practice to use it with Deployments/StatefulSets.
