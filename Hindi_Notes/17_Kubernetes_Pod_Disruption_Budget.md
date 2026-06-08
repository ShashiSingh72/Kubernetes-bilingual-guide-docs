# Kubernetes Pod Disruption Budget (PDB): Aapka Safety Net 🛡️🚧

Kubernetes me kabhi-kabhi nodes ko maintenance ke liye band karna padta hai (upgrades ya patching ke liye). Jab aisa hota hai, toh pods ko waha se hataya (evict kiya) jata hai. **Pod Disruption Budget (PDB)** aapka wo tareeka hai jisse aap K8s ko bolte ho: "Bhai, mere app ko sahi chalne ke liye kam se kam X pods hamesha zinda rehne chahiye."

---

## 1. PDB Kya Hai? (Security Guard ki Analogy) 🥳

Sochiye ek **Party** ho rahi hai jisme hamesha **3 Security Guards** ka hona zaroori hai.
Agar kisi guard ko break par jana hai (maintenance), toh wo tabhi ja sakta hai jab peeche 3 guards bache hon. Agar pehle se hi sirf 3 hain, toh koi nahi ja sakta jab tak koi replacement na aa jaye.

K8s me, **PDB** voluntary disruptions ke waqt ek saath kitne pods band ho sakte hain, uski limit set karta hai.

---

## 2. Voluntary vs. Involuntary Disruptions 🌪️🛠️

PDB samajhne ke liye ye do cheezein janna bahut zaroori hai:

*   **Involuntary Disruptions (Jo humare haath me nahi hain):**
    *   Hardware failure (Machine kharab ho gayi).
    *   Kernel panic.
    *   Cluster panic.
*   **Voluntary Disruptions (Jo hum jaan-boojh kar karte hain):**
    *   `kubectl drain` karna node maintenance ke liye.
    *   Deployment update karna.
    *   Cluster autoscaler ka node delete karna.

**Dhyan dein:** PDB sirf **Voluntary Disruptions** se bachata hai. Agar node ka hardware crash ho gaya, toh PDB kuch nahi kar sakta!

---

## 3. PDB ki Zaroorat Kyu Hai? 🤔

1.  **High Availability:** Ye ensure karta hai ki aapka app hamesha traffic handle karne ke liye taiyaar rahe.
2.  **Quorum Protection:** Etcd ya ZooKeeper jaise apps ke liye, agar ek saath bahut saare nodes gir gaye toh poora system band ho sakta hai. PDB isse rokta hai.
3.  **Safe Upgrades:** Cluster admin galti se ek saath saare pods delete na kar de, ye uska dhyan rakhta hai.

---

## 4. PDB Kaise Define Karein? (Do Tareeke) 📏

Aap apna "budget" do tareekon se set kar sakte hain:

*   **minAvailable:** Kam se kam itne pods up rehne chahiye. (e.g., "Mujhe 3 pods hamesha chahiye").
*   **maxUnavailable:** Zyada se zyada itne pods band ho sakte hain. (e.g., "Sirf 1 pod ek baar me band ho sakta hai").

---

## 5. YAML Example (Production Grade) 📄

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  # Tareeka 1: Kam se kam itne zinda rehne chahiye
  minAvailable: 2 
  
  # YA Tareeka 2: Zyada se zyada itne mar sakte hain (Dono me se ek hi use karein)
  # maxUnavailable: 1 

  # Ye PDB kin pods par lagega?
  selector:
    matchLabels:
      app: my-web-server
```

*Note: Aap percentage bhi use kar sakte hain, jaise `minAvailable: 50%`.*

---

## 6. `kubectl drain` ke waqt kya hota hai? 🚿

Jab admin `drain` command chalata hai:
1.  K8s check karta hai ki kya us node ke pods par koi PDB laga hai.
2.  Agar pod delete karne se PDB violate ho raha hai (e.g., `minAvailable` 2 hai aur sirf 2 hi bache hain), toh `drain` command **fail** ho jayegi ya ruk jayegi.
3.  Admin ko wait karna padega jab tak doosre node par naya pod start na ho jaye.

---

## 7. Advanced: Common Pitfalls ⚠️

1.  **minAvailable = Total Replicas:** Agar aapne 3 pods rakhe hain aur `minAvailable` bhi 3 kar diya, toh aap **kabhi bhi** node drain nahi kar payenge. `drain` hamesha fail hoga.
2.  **Single Replica Apps:** Jo app sirf 1 replica par chal rahe hain, unke liye PDB use karna mushkil hota hai kyuki drain karne par wo hamesha 0 ho jayenge.
3.  **Selector Check:** Hamesha check karein ki PDB ka `selector` aur Deployment ke `labels` match kar rahe hon.

---

## 🚀 Top 10 Production-Grade Interview Questions

1.  **Pod Disruption Budget (PDB) kya hota hai?**
    *   *Ans:* Ye ek policy hai jo batati hai ki voluntary maintenance ke waqt kam se kam kitne pods available rehne chahiye.

2.  **Kya PDB hardware failure se bachata hai?**
    *   *Ans:* Nahi. PDB sirf voluntary (planned) disruptions se bachata hai, involuntary (unplanned) crashes se nahi.

3.  **`minAvailable` aur `maxUnavailable` me kya antar hai?**
    *   *Ans:* `minAvailable` floor set karta hai (kitne up rahein), aur `maxUnavailable` ceiling set karta hai (kitne down ho sakein).

4.  **Agar `kubectl drain` PDB rules tod raha ho toh kya hoga?**
    *   *Ans:* Eviction request reject ho jayegi aur `drain` command wait karega ya fail ho jayega.

5.  **PDB me percentage use kar sakte hain?**
    *   *Ans:* Haan, jaise `minAvailable: 80%`.

6.  **Quorum-based apps ke liye PDB kyu zaroori hai?**
    *   *Ans:* Kyuki agar quorum (majority) loss hua toh database fail ho sakta hai. PDB ensure karta hai ki majority hamesha bani rahe.

7.  **`maxUnavailable: 0` set karne se kya hoga?**
    *   *Ans:* K8s kisi bhi pod ko voluntarily delete nahi karne dega. Maintenance ke waqt ye node ko drain hone se rok dega.

8.  **Kya ek Pod par do PDB lag sakte hain?**
    *   *Ans:* Nahi, ek pod par ek hi PDB lagna chahiye. Agar multiple lagenge toh behavior ajeeb ho sakta hai.

9.  **PDB ka status kaise check karein?**
    *   *Ans:* `kubectl get pdb`. Isme `ALLOWED DISRUPTIONS` column check karein.

10. **Kya PDB bina controller (naked pods) ke kaam karta hai?**
    *   *Ans:* Haan, agar labels match karte hain toh kaam karega, par hamesha Deployment/StatefulSet ke saath use karna best practice hai.
