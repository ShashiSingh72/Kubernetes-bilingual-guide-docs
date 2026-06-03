# Kubernetes Production Errors & Interview Prep 🛠️🎓

When working with Kubernetes, some errors occur frequently, and in interviews, they are often subjected to a "Post-mortem" (detailed investigation). The biggest star here is: **CrashLoopBackOff.**

---

## 1. CrashLoopBackOff: The Most Frequently Asked Question 🔄

**Simple Meaning:** A Pod starts, crashes, K8s restarts it, it crashes again... and this loop continues.

**Main Causes:**
1.  **Code Error:** A bug in the app is preventing it from running.
2.  **Missing Config:** The app needs an `.env` file or ConfigMap to run, but it can't find it.
3.  **Database Connection:** The app tries to connect to the DB on startup, fails, and crashes.
4.  **Port Conflict:** The required port is already taken by something else.

**How to Debug? (Steps):**
*   Step 1: `kubectl logs <pod-name>` (This is where you'll find the real cause).
*   Step 2: `kubectl describe pod <pod-name>` (Check Events to see what K8s is saying).

---

## 2. Other Famous Production Errors 🛑

### A. ImagePullBackOff / ErrImagePull
*   **Meaning:** K8s cannot download the image.
*   **Cause:** Incorrect image name, wrong tag (e.g., `v10` instead of `v1`), or wrong DockerHub password.

### B. OOMKilled (Out of Memory Killed) 😵
*   **Meaning:** The Pod consumed more RAM than its "Boundary" (Limit).
*   **Cause:** Memory leak in the app or the limit was set too low.
*   **Solution:** Increase the RAM `limits` in the YAML.

### C. Pod stays in "Pending" State ⏳
*   **Meaning:** The Pod is waiting in line but can't find a "Room" (Node) to live in.
*   **Cause:** No space in the cluster (No CPU/RAM available), or "Taints/Tolerations" are blocking the Pod.

---

## 3. The "Debugging Masterclass" Flow 🩺
If asked in an interview, "The app isn't working, what will you do?", give this answer:

1.  **Check Pod Status:** `kubectl get pods` (See the current status).
2.  **Describe Pod:** `kubectl describe pod <name>` (Check events: was it scheduled on a Node?).
3.  **Check Logs:** `kubectl logs <name>` (What error occurred inside the app?).
4.  **Check Events:** `kubectl get events --sort-by=.metadata.creationTimestamp` (What went wrong in the cluster?).
5.  **Inside the Pod:** `kubectl exec -it <name> -- sh` (Enter the pod and check if files exist).

---

## 4. Advance Interview Questions (Pro Level) 🧠

**Q1: What happens when you run `kubectl apply -f deployment.yaml`?**
*   **Ans:** API Server receives the request -> stores it in `etcd` -> Controller Manager notices the Deployment -> creates a ReplicaSet -> Scheduler finds a free Node -> Kubelet tells the container runtime to start the Pod.

**Q2: What is the difference between Liveness and Readiness Probes?**
*   **Ans:** Liveness checks if the Pod is "Alive" (if not, restart it). Readiness checks if the Pod is "Ready" to take traffic (if not, remove it from the Service).

**Q3: How to update an application with zero downtime?**
*   **Ans:** Use the **RollingUpdate** strategy in the Deployment.

---

## 5. Summary Checklist for Interviews:

| Error | Quick Fix |
| :--- | :--- |
| **CrashLoopBackOff** | Check Logs (`kubectl logs`) |
| **ImagePullBackOff** | Check Image name/Registry credentials |
| **OOMKilled** | Increase RAM limits in YAML |
| **Pending** | Check Node resources or Taints |
| **CreateContainerConfigError** | Missing ConfigMap or Secret |

**That's Troubleshooting!** Master these errors, and you've conquered 50% of Kubernetes. 🚀
