Perfect.
Ab hum **LEVEL 4 – PRINTK & LOGGING** ko **“stethoscope” mindset** ke saath samjhenge.

Is level ke baad tum:

- Andha `printk()` spam **nahi** karoge
- Log **aise jagah** daaloge jahan se bug **khud bolne lage**

------

# 🔵 LEVEL 4 – PRINTK & LOGGING

> **“Stethoscope se heartbeat sunna”**

------

## 🧠 BIG MINDSET (Sabse pehle)

> ❌ Log likhna = debugging
> ✅ **Sahi log likhna = diagnosis**

Kernel log:

- Timing ko badal sakta hai
- Race condition chhupa sakta hai
- Performance gira sakta hai

Isliye **log bhi ek weapon hai**, use responsibly.

------

# 🟢 4.1 printk – Deep Understanding

> **“Basic tool, lekin dangerous”**

------

## 1️⃣ Log Levels (MOST IMPORTANT)

### Kernel log levels

| Level | Macro          | Matlab               |
| ----- | -------------- | -------------------- |
| 0     | `KERN_EMERG`   | System unusable      |
| 1     | `KERN_ALERT`   | Immediate action     |
| 2     | `KERN_CRIT`    | Critical             |
| 3     | `KERN_ERR`     | Error                |
| 4     | `KERN_WARNING` | Warning              |
| 5     | `KERN_NOTICE`  | Normal but important |
| 6     | `KERN_INFO`    | Informational        |
| 7     | `KERN_DEBUG`   | Debug                |

Example:

```c
printk(KERN_ERR "DMA failed\n");
```

------

### Debugger rule

> **Bug ke liye level choose karo, habit se nahi**

- Real error → `ERR`
- Suspicious → `WARN`
- Flow trace → `DEBUG`

------

## 2️⃣ printk Rate Limiting

### ❓ Problem

- Loop me printk
- IRQ me printk

Result:

- Log flood
- System slow
- Bug hide

------

### Solution

```c
printk_ratelimited(KERN_WARNING "IRQ storm\n");
```

🧠 Use when:

- Repeating error
- High-frequency path

------

## 3️⃣ printk vs pr_debug

### ❓ Difference

```c
printk(KERN_DEBUG "x=%d\n", x);
pr_debug("x=%d\n", x);
```

| printk             | pr_debug              |
| ------------------ | --------------------- |
| Always compiled    | Can be compiled out   |
| No dynamic control | Dynamic debug support |
| Risky              | Safe                  |

🧠 **Rule**

> Debug logs = `pr_debug()`
> Errors = `printk(KERN_ERR)`

------

# 🟡 4.2 Dynamic Debug

> **“Heartbeat sunna, bina surgery”**

------

## ❓ Dynamic debug kya hai?

Runtime pe:

- Debug log **ON/OFF**
- Reboot ki zarurat nahi

------

## Enable kaise kare?

### Step 1: Kernel config

```
CONFIG_DYNAMIC_DEBUG=y
```

------

### Step 2: Debugfs

```
mount -t debugfs none /sys/kernel/debug
```

------

### Step 3: Enable logs

Example:

```
echo 'file sound/soc/* +p' > /sys/kernel/debug/dynamic_debug/control
```

🧠 Matlab:

- Sirf ALSA code ka debug
- Baaki system silent

------

## Production-safe kyun?

- No printk spam
- No reboot
- Customer device par bhi possible

🧠 **Senior engineers ka favourite**

------

## Best practice

```c
pr_debug("probe start, dev=%p\n", dev);
```

Dynamic debug se control karo.

------

# 🔴 4.3 trace_printk

> **“High-end stethoscope (use carefully)”**

------

## ❓ trace_printk kya hai?

- printk jaisa
- Lekin **ftrace buffer** me jaata hai
- Console pe nahi

------

## Kab use kare?

### 1️⃣ Timing bugs

- Race condition
- Lock order
- Preemption issues

------

### 2️⃣ Non-intrusive tracing

- printk → slow
- trace_printk → minimal disruption

------

## Example

```c
trace_printk("lock acquired %d\n", cpu);
```

Output:

```
/sys/kernel/debug/tracing/trace
```

------

## WARNING ⚠️

> ❌ Production me mat chhodo
> ❌ Long term debugging ke liye nahi

Use:

- Short investigation
- Remove after fix

------

# 🧠 WHERE TO PUT LOGS (ART, NOT SCIENCE)

### ❌ Galat jagah

```c
while (1) {
    printk("loop\n");
}
```

------

### ✅ Sahi jagah

- Entry / exit points
- State change
- Error paths
- Before & after locks
- Allocation / free

------

## 🧠 LOGGING STRATEGY (GOLD)

1. Pehle **high-level flow**
2. Phir **conditions**
3. Phir **data**
4. Phir **timing**

> “Print karo **kyun**, sirf **kya** nahi”

------

# 🧠 FINAL GOLDEN RULES

- printk se bug **fix** nahi hota

- Galat printk se bug **hide** ho jaata hai

- Best log wo hota hai:

  > “Jise dekh kar next step obvious ho”

------

## ✅ LEVEL 4 COMPLETE

### Next best step:

👉 **LEVEL 5 – Memory debugging (KASAN, UAF, corruption)**
👉 ya **Live example: ek tricky bug ko sirf logging se expose karna**

Batao aage kya karein.