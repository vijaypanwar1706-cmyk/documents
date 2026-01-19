Excellent.
Ab hum **LEVEL 10 – CONFIG & BUILD-LEVEL DEBUGGING** ko **“operation se pehle ka diagnostic setup”** jaise samjhenge.

Yeh level sikhaata hai:

> ❌ Har time sab DEBUG on
> ✅ **Right time, right CONFIG**

------

# 🔵 LEVEL 10 – CONFIG & BUILD-LEVEL DEBUGGING

> **“Galat tools ke saath surgery mat karo”**

------

## 🧠 BIG MINDSET (MOST IMPORTANT)

> ❌ Kernel crash ho raha hai → aur DEBUG off hai
> = **andhere me operation**

Kernel CONFIGs:

- Bug **pakad bhi sakte hain**
- Bug **chhupa bhi sakte hain**
- Timing **badal bhi sakte hain**

Isliye CONFIG bhi ek **debug tool** hai.

------

# 🟢 10.1 Debug CONFIGs (Core Tools)

------

## 1️⃣ `CONFIG_DEBUG_KERNEL`

> **“Master switch”**

### Kya karta hai?

- Baaki debug options enable hone deta hai
- WARN, sanity checks active

🧠 **Rule**

> Agar debugging kar rahe ho → **ye ON hona chahiye**

------

## 2️⃣ `CONFIG_DEBUG_LIST`

> **“Linked list ka lie detector”**

### Kya detect karta hai?

- Double add
- Double delete
- Corrupt list pointers

### Typical crash before:

- Random oops
- Panic in unrelated code

### After enable:

```
list_del corruption. prev->next is LIST_POISON1
```

🧠 **Driver dev ke liye MUST**

------

## 3️⃣ `CONFIG_DEBUG_ATOMIC_SLEEP`

> **“Atomic context ka police”**

### Kya detect karta hai?

- sleep in spinlock
- mutex in IRQ
- GFP_KERNEL in atomic context

Message:

```
BUG: sleeping function called from invalid context
```

🧠 **Most common beginner mistakes yahin pakde jaate hain**

------

## 4️⃣ `CONFIG_SCHED_DEBUG`

> **“Scheduler ECG”**

### Kya milta hai?

- Runqueue state
- Task info
- Preemption status

Use when:

- Freeze
- CPU starvation
- Hung task

🧠 **Freeze bina panic → ye ON karo**

------

# 🟡 10.2 Wrong CONFIG Bugs

> **“Bug tha, par dikha hi nahi”**

------

## 1️⃣ Debug OFF → Crash Miss

### Example

- Use-after-free
- Race condition

Without debug:

- Random crash
- Weird behavior

With debug:

```
refcount_t: underflow
```

🧠 **Bug pehle bhi tha, ab visible hua**

------

## 2️⃣ Debug kernel ≠ Production kernel

### Debug kernel

- Heavy checks
- Slow
- More logs
- Bugs earlier exposed

### Production kernel

- Fast
- Silent
- Bugs delayed

🧠 **Golden rule**

> Bug debug kernel me aaye → production me bhi hai
> Bas chhupa hua

------

## 3️⃣ Debug CONFIG timing bug hide kar sakta hai

### Reality

- Debug code slow karta hai
- Race disappear ho sakti hai

🧠 Isliye:

- Kabhi debug OFF karke bhi test karo

------

# 🧠 WHICH CONFIG WHEN? (MOST IMPORTANT)

### Crash / Panic

- `DEBUG_KERNEL`
- `DEBUG_LIST`
- `DEBUG_ATOMIC_SLEEP`

------

### Freeze / Hang

- `SCHED_DEBUG`
- `LOCKDEP`
- `RCU_STALL_DETECTOR`

------

### Memory issues

- `KASAN`
- `SLUB_DEBUG`
- `PAGE_POISONING`

------

### Race conditions

- `KCSAN`
- `LOCKDEP`

------

# 🧠 BUILD-LEVEL DEBUGGING TIPS

- Same `.config` ko version control me rakho
- Working vs broken config diff karo
- One change at a time

🧠 **CONFIG bisect = powerful technique**

------

# 🧠 ONE-LINE GOLDEN TRUTH

> **Kernel debugging me aadha bug code me hota hai**
> **aur aadha CONFIG me**

------

## ✅ LEVEL 10 COMPLETE

Next logical steps:

👉 **LEVEL 11 – Static analysis (bug runtime se pehle pakadna)**
👉 ya **Real case: ek bug jo sirf debug CONFIG me dikha**

Batao aage kaunsa level chahiye.