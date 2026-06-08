# Kubernetes Health Probes: Basic se Advance tak 🩺🚀

Production me aap sirf ye umeed nahi kar sakte ki aapka app sahi chal raha hoga. Aapko ek system chahiye jo automatic check kare ki app crash toh nahi hua, ya hang toh nahi ho gaya. Iske liye Kubernetes use karta hai **Health Probes**.

---

## 1. Health Probes Kya Hai? (Doctor ki Analogy) 👨‍⚕️

Sochiye Kubernetes ek **Doctor** hai aur aapka Pod ek **Patient**.
Doctor patient ke shikayat karne ka intezaar nahi karta, wo thodi-thodi der me pulse aur breathing check karta rehta hai taaki sab theek rahe.

Technical term me, **Probe** ek periodic check hai jo `kubelet` container par karta hai.

---

## 2. Probes ki Zaroorat Kyu Hai? 🤔

1.  **Self-Healing:** Agar aapka app hang ho gaya (Deadlock), toh K8s ko bahar se dekh kar pata nahi chalega. Probes K8s ko batate hain ki app "zombie" ban gaya hai aur use **restart** kar dena chahiye.
2.  **Zero Downtime:** Agar app ko database load karne me 30 seconds lagte hain, toh aap nahi chahenge ki traffic turant waha jaye. Probes ensure karte hain ki traffic tabhi jaye jab app poori tarah ready ho.
3.  **Stability:** Ye kharab pods ko users tak pahunchne se rokta hai.

---

## 3. Probes ke Teen Types 🛡️

### A. Liveness Probe (Kya tum zinda ho?) 💓
*   **Kaam:** Check karna ki container chal raha hai ya nahi.
*   **Action:** Agar ye fail hua, toh Kubernetes container ko **restart** kar dega.
*   **Kab use karein:** Jab app chal toh raha ho par kaam na kar raha ho (jaise infinite loop me fans jana).

### B. Readiness Probe (Kya tum kaam ke liye taiyaar ho?) 🚦
*   **Kaam:** Check karna ki app traffic handle karne ke liye ready hai ya nahi.
*   **Action:** Agar ye fail hua, toh Kubernetes Pod ka IP **Service se hata dega**. Traffic aana band ho jayega, par pod restart nahi hoga.
*   **Kab use karein:** App startup ke time, jab wo files load kar raha ho ya cache warm up kar raha ho.

### C. Startup Probe (Kya tum abhi uth rahe ho?) ☕
*   **Kaam:** Slow-starting apps ko handle karna.
*   **Action:** Jab tak ye success nahi hota, Liveness aur Readiness probes ko disable rakhta hai.
*   **Kab use karein:** Aise purane (legacy) apps jo start hone me 5-10 minute lete hain. Ye Liveness probe ko pod kill karne se rokta hai jab tak startup khatam na ho jaye.

---

## 4. Kubernetes Kaise Check Karta Hai? (Mechanisms) ⚙️

K8s ke paas pulse check karne ke 4 tareeke hain:

1.  **HTTP GET:** K8s ek endpoint (e.g., `/healthz`) par request bhejta hai. Agar response 200-399 ke beech hai, toh sab theek hai.
2.  **Exec Command:** Container ke andar ek command run ki jati hai (e.g., `cat /tmp/healthy`). Agar exit code 0 aaya, toh healthy hai.
3.  **TCP Socket:** K8s ek specific port par connection kholne ki koshish karta hai. Connect hua toh healthy.
4.  **gRPC:** (Advanced) K8s gRPC Health Checking Protocol use karta hai.

---

## 5. Configuration Parameters (Setting Sahi Karna) 🔧

Inhe tune karna bahut zaroori hai:

*   **initialDelaySeconds:** Pehla check karne se pehle kitne second rukan hai.
*   **periodSeconds:** Kitni der me check karna hai (e.g., har 10 second me).
*   **timeoutSeconds:** Response ka kitni der intezaar karna hai.
*   **failureThreshold:** Kitni baar fail hone par K8s action le (Restart ya Service se hatana).
*   **successThreshold:** Kitni baar pass hone par use healthy mana jaye (aksar 1).

---

## 6. YAML Example (Production Grade) 📄

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-probe-demo
spec:
  containers:
  - name: my-app
    image: my-app:v1
    ports:
    - containerPort: 8080

    # 1. Startup Probe: Slow startup ke liye
    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30
      periodSeconds: 10 # App ko 300 seconds milenge start hone ke liye

    # 2. Liveness Probe: Hang hone par restart karega
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 15

    # 3. Readiness Probe: Jab tak ready na ho traffic mat bhejo
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
```

---

## 7. Advanced: Best Practices 💡

1.  **Alag Endpoints Rakhein:** Liveness (`/live`) aur Readiness (`/ready`) ke liye alag endpoints use karein. Liveness check halka hona chahiye, jabki Readiness me DB connection check kar sakte hain.
2.  **Over-probe mat karein:** Bahut jaldi-jaldi check karne se app par load badh sakta hai. `periodSeconds` dhyan se set karein.
3.  **Liveness me external dependency mat daalein:** Liveness ko kabhi database par depend mat karein. Agar DB down hua, toh saare pods restart hone lagenge (CrashLoopBackOff).
4. **Legacy apps ke liye Startup Probes best hain:** Ye sabse safe tareeka hai slow apps ko handle karne ka.

---

## 🚀 Top 10 Production-Grade Interview Questions

1.  **Liveness aur Readiness probe me kya antar hai?**
    *   *Ans:* Liveness probe fail hone par container ko **restart** karta hai. Readiness probe fail hone par pod ko Service traffic se hata deta hai, par restart nahi karta.

2.  **Agar Liveness probe fail ho jaye toh kya hoga?**
    *   *Ans:* `kubelet` us container ko kill kar dega aur restart policy ke hisaab se naya container start karega.

3.  **Liveness probe me database check kyu nahi dalna chahiye?**
    *   *Ans:* Kyuki agar database down hua toh aapke saare pods restart hone lagenge (CrashLoopBackOff), jabki app me koi kharabi nahi hai. Isse Readiness me use karein.

4.  **`initialDelaySeconds` ka kya use hai?**
    *   *Ans:* Ye app ko start hone ka thoda waqt deta hai pehla check shuru karne se pehle, taaki app faltu me restart na ho.

5.  **Startup Probes slow apps ki kaise madad karte hain?**
    *   *Ans:* Ye slow apps ko lamba startup time dete hain aur tab tak Liveness/Readiness ko rok kar rakhte hain jab tak app ek baar successfully start na ho jaye.

6.  **Default `failureThreshold` kitna hota hai?**
    *   *Ans:* Iski default value 3 hoti hai.

7.  **Kya ek Pod me ek se zyada probes ho sakte hain?**
    *   *Ans:* Haan, ek container me Liveness, Readiness aur Startup teeno ek saath configure kiye ja sakte hain.

8.  **`periodSeconds` aur `timeoutSeconds` me kya farak hai?**
    *   *Ans:* `periodSeconds` matlab kitni der me check karna hai. `timeoutSeconds` matlab ek check ke response ka kitni der wait karna hai.

9.  **Liveness/Startup probes ke liye `successThreshold` kya hona chahiye?**
    *   *Ans:* Inke liye ye hamesha exact 1 hona chahiye. Readiness ke liye ye 1 se zyada ho sakta hai.

10. **Fail hote hue probe ko kaise debug karein?**
    *   *Ans:* `kubectl describe pod <pod-name>` command run karein aur "Events" section check karein. Waha probe fail hone ka message dikh jayega.

