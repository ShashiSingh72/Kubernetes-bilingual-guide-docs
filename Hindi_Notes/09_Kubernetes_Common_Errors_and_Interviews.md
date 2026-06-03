# Kubernetes Production Errors & Interview Prep 🛠️🎓

Kubernetes me kaam karte waqt kuch errors aise hote hain jo hamesha aate hain, aur interviews me inka "Post-mortem" (gehraai se pucha jata hai). Isme sabse bada hero hai: **CrashLoopBackOff.**

---

## 1. CrashLoopBackOff: Sabse Zyada Pucha Jane Wala Sawaal 🔄

**Simple Meaning:** Pod start hota hai, crash ho jata hai, phir K8s use restart karta hai, wo phir crash ho jata hai... aur ye loop chalta rehta hai.

**Kyu aata hai? (Main Causes):**
1.  **Code Error:** App ke andar koi bug hai jo use chalne hi nahi de raha.
2.  **Missing Config:** App ko chalne ke liye `.env` file ya ConfigMap chahiye tha, par wo nahi mila.
3.  **Database Connection:** App start hote hi DB se connect karne ki koshish karta hai, DB nahi mila toh crash ho gaya.
4.  **Port Conflict:** Wo port pehle se hi kisi aur ne pakda hua hai.

**Debug Kaise Karein? (Steps):**
*   Step 1: `kubectl logs <pod-name>` (Yahan aapko asli wajah milegi).
*   Step 2: `kubectl describe pod <pod-name>` (Events dekho ki K8s kya bol raha hai).

---

## 2. Other Famous Production Errors 🛑

### A. ImagePullBackOff / ErrImagePull
*   **Meaning:** K8s image download nahi kar pa raha.
*   **Wajah:** Image ka naam galat hai, tag galat hai (`v1` ki jagah `v10` likh diya), ya Registry (DockerHub) ka password galat hai.

### B. OOMKilled (Out of Memory Killed) 😵
*   **Meaning:** Pod ne apni "Aukat" (Limit) se zyada RAM kha li.
*   **Wajah:** App me memory leak hai ya aapne limit bahut kam set ki hai.
*   **Solution:** `limits` me RAM badhao.

### C. Pod stays in "Pending" State ⏳
*   **Meaning:** Pod line me laga hai par use koi "Room" (Node) nahi mil raha.
*   **Wajah:** Cluster me jagah nahi hai (No CPU/RAM), ya aapne "Taints/Tolerations" lagaye hain jo Pod ko wahan jaane nahi de rahe.

---

## 3. The "Debugging Masterclass" Flow 🩺
Interview me agar pucha jaye "App nahi chal raha, kya karoge?", toh ye answer dena:

1.  **Check Pod Status:** `kubectl get pods` (Dekho Pod ka status kya hai).
2.  **Describe Pod:** `kubectl describe pod <name>` (Events dekho, kya wo Node pe schedule hua?).
3.  **Check Logs:** `kubectl logs <name>` (App ke andar kya error aaya?).
4.  **Check Events:** `kubectl get events --sort-by=.metadata.creationTimestamp` (Cluster me kya gadbad hui).
5.  **Inside the Pod:** `kubectl exec -it <name> -- sh` (Pod ke andar jaakar check karo file hai ya nahi).

---

## 4. Advance Interview Questions (Pro Level) 🧠

**Q1: What happens when you run `kubectl apply -f deployment.yaml`?**
*   **Ans:** API Server request leta hai -> `etcd` me store karta hai -> Controller Manager dekho Deployment ko -> ReplicaSet banata hai -> Scheduler dekhta hai kaun sa Node khali hai -> Kubelet container runtime ko bolkar Pod start karwata hai.

**Q2: Liveness vs Readiness Probe me kya antar hai?**
*   **Ans:** Liveness check karta hai Pod "Zinda" hai ya nahi (Nahi toh restart kar do). Readiness check karta hai Pod "Traffic" lene ke liye taiyaar hai ya nahi (Nahi toh Service se hata do).

**Q3: How to update an application with zero downtime?**
*   **Ans:** Use **RollingUpdate** strategy in Deployment.

---

## 5. Summary Checklist for Interviews:

| Error | Quick Fix |
| :--- | :--- |
| **CrashLoopBackOff** | Check Logs (`kubectl logs`) |
| **ImagePullBackOff** | Check Image name/Registry credentials |
| **OOMKilled** | Increase RAM limits in YAML |
| **Pending** | Check Node resources or Taints |
| **CreateContainerConfigError** | Missing ConfigMap or Secret |

**Bas yahi hai Troubleshooting!** In errors ko samajh liya toh aapne 50% Kubernetes jeet liya. 🚀
