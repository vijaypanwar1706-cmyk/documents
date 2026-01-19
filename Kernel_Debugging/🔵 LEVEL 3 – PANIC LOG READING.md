Excellent.
Ab hum **LEVEL 3 – PANIC LOG READING** ko **bilkul ECG report padhne** jaise samjhenge.

Is level ke baad tum **panic log dekh kar ghabraoge nahi**, balki sochoge:

> “Achha… yahan se kahani shuru hoti hai.”

------

# 🔵 LEVEL 3 – PANIC LOG READING

> **“ECG report padhna seekho”**

------

## 🧠 BIG MINDSET (Sabse pehle ye baithao)

> ❌ Panic log = random scary text
> ✅ Panic log = **kernel ki last autobiography**

Kernel crash ke last 1–2 ms me **sab kuch likh deta hai**
Tumhara kaam = **usko sahi order me padhna**

------

# 🟢 3.1 Panic Log ka Structure

> **“Body report ka format”**

Ek typical panic / oops log me hamesha ye parts hote hain
(order kabhi-kabhi thoda badal sakta hai):

------

## 1️⃣ CPU Number

### Example:

```
CPU: 2 PID: 134 Comm: alsactl Tainted: G O
```

### Matlab:

- **CPU: 2** → kaunsa core crash ke waqt chal raha tha

🧠 Debug hint:

> Agar bug **sirf CPU >0 par aata hai**
> → SMP / per-cpu / race bug

------

## 2️⃣ PID / Task Information

### Example:

```
PID: 134 Comm: alsactl
```

### Matlab:

- Kaunsa process context me crash hua
- IRQ context nahi hai (wo alag dikhega)

🧠 Important:

> Driver crash ≠ user app ka bug
> User app sirf **trigger** hota hai

------

## 3️⃣ Call Trace (SABSE IMPORTANT)

### Example:

```
Call Trace:
 snd_soc_probe+0x34/0x120
 platform_drv_probe+0x50/0xa0
 really_probe+0x1c0/0x3a0
 driver_probe_device+0x90/0x120
```

### Matlab:

- **Last function → pehle wali functions**
- Stack ka ulta order

🧠 Golden rule:

> **Top function = jahan crash hua**
> **Neeche = kaun wahan le kar aaya**

------

## 4️⃣ Register Dump

### Example:

```
RIP: snd_soc_probe+0x34/0x120
RAX: 0000000000000000
RBX: ffff888012345000
```

(ARM64 me: `PC`, `x0-x30`)

------

### Debug power

- `x0 / RDI` → first argument
- NULL value → NULL deref
- Weird value → corruption

🧠 Advanced skill:

> Register dekh kar pata lagana
> **kaunsa pointer galat tha**

------

## 5️⃣ Stack Dump

### Example:

```
Stack:
 ffff88801234abc0 ffffffff81012340
 ffff88801234abd0 ffffffff81045678
```

### Matlab:

- Raw stack memory
- Deep corruption ke liye

🧠 Beginners ignore karte hain
Experts yahin se truth nikaalte hain

------

# 🟣 3.2 Call Trace Decoding

> **“Function names ke peeche ka sach”**

------

## 1️⃣ RIP / PC ka matlab

### RIP (x86) / PC (ARM64)

- Current instruction pointer
- **Exactly wahi instruction** jahan CPU gira

Example:

```
PC is at snd_soc_probe+0x34/0x120
```

🧠 Matlab:

- Function start se `0x34` bytes andar
- Total size `0x120`

------

### Debug trick

> Agar crash hamesha **same offset** par ho
> → deterministic bug
> Agar offset badalta rahe
> → memory corruption

------

## 2️⃣ Inline Functions (Invisible killers)

### ❓ Inline function kya hota hai?

Compiler ne function ko **merge kar diya**

Result:

- Call trace me function dikhta hi nahi
- Lekin bug wahi hota hai

🧠 Debugger mindset:

> “Call trace me nahi dikh raha
> iska matlab ye nahi ki yeh involve nahi”

------

## 3️⃣ ORC Unwinder (Kernel 6.x)

### ❓ ORC kya karta hai?

- Reliable stack unwinding
- Frame pointer ke bina bhi

Old problem:

- Corrupt stack = fake call trace

New kernel:

- ORC = **zyada trustworthy trace**

🧠 Lekin:

> Agar memory corrupt ho
> ORC bhi confused ho sakta hai

------

# 🟠 3.3 Address → Source Line

> **“Address ko code me badalna”**

Ab tumhe address mil gaya
Ab usko **actual source line** banana hai

------

## 1️⃣ addr2line

### Use:

```
addr2line -e vmlinux ffffffff81012340
```

### Output:

```
drivers/sound/soc/core.c:245
```

🧠 Fast, simple, effective

------

## 2️⃣ gdb + vmlinux (Doctor level)

### Use cases:

- Variable inspect
- Structure dump
- Register mapping

Example:

```
(gdb) list *snd_soc_probe+0x34
```

🧠 Yeh tab use karo jab:

> Panic repeat ho raha ho
> Root cause gehra ho

------

## 3️⃣ objdump

### Use:

- Assembly view
- Instruction-level crash

Example:

```
objdump -d vmlinux | less
```

🧠 Kab kaam aata hai?

> Jab C code sahi lage
> par instruction galat ho

------

# 🧠 HOW TO READ A PANIC LOG (STEP-BY-STEP)

### Step 1

- Panic vs Oops?

### Step 2

- CPU number
- PID / context

### Step 3

- Top of call trace

### Step 4

- Registers (NULL? garbage?)

### Step 5

- Address area (Level 1 memory anatomy yaad)

------

# 🧠 FINAL GOLDEN LINE

> **Panic log random nahi hota**
> Wo ek **sequence of events** hota hai
> Tumhe sirf **uski language samajhni hai**

------

## ✅ LEVEL 3 COMPLETE

### Next natural jump:

👉 **LEVEL 4 – printk & logging (stethoscope)**
👉 ya **Real panic log le kar full dissection (ECG → diagnosis)**

Batao next kya chahiye.