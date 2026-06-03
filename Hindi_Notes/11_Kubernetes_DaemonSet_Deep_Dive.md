# Kubernetes DaemonSet: Basic se Advance Deep Dive 🛡️🖥️

Kubernetes me **DaemonSet** ko samajhne ke liye ek simple example lete hain: **Ek Apartment Building ke har Floor par ek CCTV Camera.**

---

## 1. DaemonSet Kya Hai? (The One-on-Every-Node) 🛡️

**Simple Meaning:** DaemonSet ye ensure karta hai ki aapke cluster ke **har ek node (worker machine)** par exactly ek Pod chale.

Agar aap cluster me 5 naye nodes add karte ho, toh DaemonSet apne aap un 5 nodes par bhi ek-ek pod start kar dega. Agar koi node delete hota hai, toh wahan se pod apne aap hat jayega.

---

## 2. Kyu aaya? (The Problem it Solves) ❓

Socho aapko har node ke **Logs** (kaam ka record) collect karne hain. 
*   Agar aap **ReplicaSet** use karoge aur bologe "5 pods chalao", toh ho sakta hai K8s saare 5 pods ek hi node par chala de.
*   Isse baaki nodes ke logs miss ho jayenge.
*   Humein ek aisa system chahiye tha jo ginti par nahi, balki **location (node)** par focus kare. Isliye **DaemonSet** banaya gaya.

---

## 3. Real World Use Cases (Kahan use hota hai?) 🛠️

Production me DaemonSet in 3 kamo ke liye hamesha use hota hai:
1.  **Logging (Logs batone wala):** Jaise `Fluentd` ya `Logstash`. Ye har node se logs uthakar central server par bhejte hain.
2.  **Monitoring (Nazar rakhne wala):** Jaise `Prometheus Node Exporter` ya `Datadog agent`. Ye har node ka CPU/RAM health check karte hain.
3.  **Networking (Rasta dikhane wala):** Jaise `kube-proxy` ya `Calico/Flannel`. Ye har node ki networking manage karte hain.

---

## 4. DaemonSet vs ReplicaSet (Sabse bada antar) 📊

| Feature | ReplicaSet / Deployment 👯‍♂️ | DaemonSet 🛡️ |
| :--- | :--- | :--- |
| **Focus** | **Quantity (Ginti):** 10 pods chahiye, chahe kahin bhi chalein. | **Quality (Location):** Har node par 1 pod chahiye hi chahiye. |
| **Scaling** | Aap manual ya HPA se pods badhate ho. | Ye cluster size (nodes) ke sath apne aap scale hota hai. |
| **Use Case** | Apps (Frontend, Backend, APIs). | Background Tools (Logging, Monitoring). |

---

## 5. Advance Concept: Pods ko kaise control karein? 🚀

Hamesha zaroori nahi ki har node par DaemonSet chale.
*   **NodeSelector:** Aap bol sakte ho "Sirf un nodes par chalao jinpar `SSD=true` ka label hai."
*   **Taints & Tolerations:** Kuch nodes special hote hain (jaise Master nodes) jahan normal pods nahi ja sakte. DaemonSet ko wahan chalane ke liye humein "Tolerations" add karni padti hain.

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

## 7. YAML ki har Line ka Matlab (Line-by-Line Explanation) 🔍

*   **`apiVersion: apps/v1`**: DaemonSet ke liye bhi hum yahi API version use karte hain.
*   **`kind: DaemonSet`**: Hum K8s ko bata rahe hain ki ye ek DaemonSet hai.
*   **`namespace: kube-system`**: Zyadatar DaemonSets cluster-level tools hote hain, isliye inhe `kube-system` me rakha jata hai.
*   **`selector`**: Ye batata hai ki DaemonSet ko kin pods par nazar rakhni hai.
*   **`tolerations`**: Ye sabse important part hai. Iska matlab hai "Bhai, agar Master node par koi panga (Taint) hai, toh bhi wahan chale jana (Tolerate kar lena)."
*   **`resources`**: Kyunki ye har node par chalega, isliye iski RAM/CPU limits set karna bahut zaroori hai taaki ye main app ke resources na kha jaye.

---

## Interview Questions (Q&A) 🎤 (Unique & Production Grade)

**1. Q: Kya hum DaemonSet me `replicas` field use kar sakte hain?**
*   **Ans:** Nahi! DaemonSet me `replicas` field nahi hoti kyunki iska count cluster ke Nodes ki ginti se decide hota hai, aapki di hui ginti se nahi.

**2. Q: "RollingUpdate" DaemonSet me kaise kaam karta hai?**
*   **Ans:** Jab aap DaemonSet ka image badalte hain, toh ye ek-ek karke purane pods ko delete karta hai aur naye pods banata hai. Isse hum `spec.updateStrategy.type: RollingUpdate` se control karte hain.

**3. Q: Kya hum DaemonSet ko sirf specific nodes par restrict kar sakte hain?**
*   **Ans:** Haan, hum `nodeSelector` ya `nodeAffinity` use kar sakte hain. Example: Sirf "GPU nodes" par monitoring agent chalana.

**4. Q: Agar naya Node cluster me join hota hai, toh DaemonSet ko kaise pata chalta hai?**
*   **Ans:** DaemonSet Controller hamesha API Server ko "watch" karta hai. Jaise hi naya Node add hota hai, Controller us node ke liye ek naya Pod create kar deta hai.

**5. Q: "Taints and Tolerations" ka DaemonSet me kya role hai?**
*   **Ans:** Default me K8s Master node par pods schedule nahi karta. Par agar humein Master node ke bhi logs chahiye, toh hum DaemonSet me "Toleration" add karte hain taaki wo Master node par bhi chal sake.

**6. Q: Kya hum ek DaemonSet pod ko manually delete kar sakte hain?**
*   **Ans:** Kar toh sakte hain, par turant DaemonSet Controller dekhega ki us node par pod missing hai, aur wo ek naya pod wahan phir se bana dega.

**7. Q: DaemonSet vs StatefulSet me kya antar hai?**
*   **Ans:** DaemonSet har node par ek copy chalata hai. StatefulSet pods ko ek unique identity (e.g., pod-0, pod-1) aur permanent storage deta hai, par wo har node par chale aisa zaroori nahi.

**8. Q: "OnDelete" update strategy kya hoti hai?**
*   **Ans:** Is strategy me image badalne par pods apne aap update nahi hote. Jab aap kisi pod ko manually delete karoge, tab naya pod nayi image ke saath aayega. Ye sensitive tools ke liye use hota hai.

**9. Q: DaemonSet pods ko schedule karne ka zimmedar kaun hai?**
*   **Ans:** Pehle DaemonSet Controller khud pods ko node assign karta tha, par ab (K8s 1.12+) **default-scheduler** hi ise handle karta hai `NodeAffinity` use karke.

**10. Q: Agar node par resources (RAM/CPU) khatam ho jayein, toh kya DaemonSet pod chalega?**
*   **Ans:** DaemonSet pods bahut important hote hain. Hum inka `priorityClassName` high rakhte hain (`system-node-critical`) taaki agar resources kam hon, toh K8s dusre normal pods ko band kar de par DaemonSet ko chalne de.
