Excellent.
Ab hum **LEVEL 2 – CRASH TYPES** ko **doctor-level diagnosis** ki tarah samjhenge.

Is level ke baad tum **panic log se pehle hi** andaza laga paoge:

> “Ye turant wali bimari hai ya body me zehar pehle se tha.”

------

# 🟡 LEVEL 2 – CRASH TYPES

> **“Bimari pehchaan = aadha ilaaj”**

------

## 🟥 BIG PICTURE (Sabse pehle ye socho)

Kernel crash **2 tarah ke hote hain**:

1. **Immediate (Acute attack)**
   → Galti hui → turant gir gaya
2. **Delayed / Silent (Slow poison)**
   → Galti pehle hui → effect baad me dikha

🧠 **Debugger rule #1**

> Crash time ≠ Bug time

------

# 🔴 2.1 Immediate Crashes

> **“Heart attack on the spot”**

Ye crashes **jaise hi galat instruction chala**, turant aate hain.

------

## 1️⃣ Kernel Panic

### ❓ Kernel panic kya hota hai?

Kernel khud decide karta hai:

> “Aage chalna unsafe hai, STOP.”

------

### Common messages

```
Kernel panic - not syncing: Fatal exception
Kernel panic - not syncing: Attempted to kill init
```

------

### Kyun hota hai?

- Unrecoverable exception
- init process mar gaya
- Stack corruption
- Inconsistent kernel state

🧠 **Important**

> Panic hamesha bug nahi hota
> Kabhi-kabhi **damage itna zyada hota hai** ki kernel ruk jaata hai

------

## 2️⃣ Kernel Oops

### ❓ Oops kya hota hai?

- Serious error
- Kernel **continue karne ki koshish karta hai**
- Lekin internal state corrupt ho jaati hai

------

### Typical message

```
Oops: 0000000096000004
```

------

### Symptoms

- System chal raha hota hai
- Kuch time baad:
  - random crash
  - weird behavior
  - reboot

🧠 **Golden rule**

> Ek Oops ke baad kernel “trustworthy” nahi hota

------

## 3️⃣ BUG / BUG_ON

### ❓ BUG kya hota hai?

Kernel ka **self-check assertion**

```c
BUG_ON(x == NULL);
```

Agar condition true:
💥 **Intentional crash**

------

### Kyun use hota hai?

- “Ye condition kabhi honi hi nahi chahiye”
- Agar hui → kernel already unsafe

🧠 BUG =

> “Doctor ne khud patient ko OT me rok diya”

------

## 4️⃣ NULL Pointer Dereference

### ❓ Sabse common immediate crash

```c
struct foo *f = NULL;
f->x = 10;
```

------

### Panic log me kaise dikhega?

```
Unable to handle kernel NULL pointer dereference at 0000000000000000
```

------

### Debug hint

| Address         | Matlab        |
| --------------- | ------------- |
| `0x0`           | Pure NULL     |
| `0x10`          | NULL + offset |
| `0xffff8880...` | Freed pointer |

🧠 **Immediate crash = line-by-line bug**

------

# 🟠 2.2 Delayed / Soft Failures

> **“Dheere dheere patient jam ho raha hai”**

Ye crashes **turant nahi aate**.

------

## 1️⃣ Soft Lockup

### ❓ Soft lockup kya hota hai?

- CPU **too long ek hi task** me busy
- Scheduler ko chance nahi milta

------

### Message

```
BUG: soft lockup - CPU#1 stuck for 22s
```

------

### Common reasons

- Infinite loop
- Preemption disabled
- Heavy work in interrupt context

🧠 **Soft lockup = logic bug**

------

## 2️⃣ Hard Lockup

### ❓ Hard lockup kya hota hai?

- CPU bilkul respond nahi kar raha
- Interrupts bhi nahi

------

### Difference

| Soft        | Hard         |
| ----------- | ------------ |
| CPU busy    | CPU dead     |
| Logs aate   | Logs mushkil |
| Recoverable | Mostly fatal |

🧠 Often hardware / spinlock bug

------

## 3️⃣ Hung Task

### ❓ Hung task kya hota hai?

Process:

- Waiting state me
- Bahut zyada time se

------

### Message

```
INFO: task xyz blocked for more than 120 seconds
```

------

### Causes

- Deadlock
- Mutex never unlocked
- IO wait forever

🧠 **User-visible symptom**

------

## 4️⃣ RCU Stall

### ❓ RCU stall kya hota hai?

RCU expects:

> “Har CPU kabhi na kabhi quiescent state me jaaye”

Agar nahi gaya:

```
RCU Stall detected
```

------

### Root causes

- Infinite loop with preemption off
- IRQ disabled too long
- Broken RCU usage

🧠 **RCU stall = serious kernel misuse**

------

# ⚫ 2.3 Silent Killers

> **“Zeher jo baad me maarta hai”**

Ye sabse **dangerous** category hai.

------

## 1️⃣ Memory Corruption

### ❓ Kya hota hai?

- Kisi ne memory overwrite kar di
- Victim kuch aur code

------

### Symptoms

- Crash random jagah
- Different call trace every time
- Unreproducible

🧠 **Crash location ≠ bug location**

------

## 2️⃣ Use-After-Free

### ❓ Classic killer

```c
kfree(p);
p->x = 10;   // 💣
```

------

### Kyun dangerous?

- Memory kisi aur ko mil jaati hai
- Data valid lagta hai
- Crash later

🧠 **Delayed poison**

------

## 3️⃣ Race Condition

### ❓ Race kya hoti hai?

- 2 CPUs
- Same data
- No proper locking

------

### Symptoms

- Bug sirf SMP me
- Bug disappear with printk
- Very hard to reproduce

🧠 **Heisenbug**

------

## 4️⃣ Refcount Bugs

### ❓ Reference counting galat ho gaya

- Underflow → free too early
- Overflow → leak

------

### Typical message

```
refcount_t: underflow; use-after-free
```

🧠 **Modern kernels isko aggressively detect karte hain**

------

# 🧠 FINAL DIAGNOSIS FRAMEWORK (MUST REMEMBER)

## Step 1: Crash timing

- Turant? → Immediate
- Baad me? → Silent / delayed

## Step 2: Pattern

- Same place? → logic bug
- Random? → memory corruption

## Step 3: CPU / SMP dependency

- Single core OK?
- Multi core crash?

------

## 🧠 ONE-LINE GOLDEN RULE

> **Crash turant dikha** → code galat
> **Crash baad me dikha** → memory / race
> **Crash random** → corruption / hardware

------

## ✅ LEVEL 2 COMPLETE

Next best progression:

👉 **LEVEL 3 – Panic log reading (ECG padhna)**
👉 **Ya ek real panic log lekar classify karen: immediate vs silent**

Bolo kaunsa chahiye.