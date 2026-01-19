Perfect. Ab hum **LEVEL 1 – KERNEL ANATOMY** ko
**bilkul zero se lekar “panic address dekh kar dimaag me picture ban jaaye”**
is level tak le jaayenge.

Is level ke baad tum **kernel ko ek body ki tarah visualize** kar paoge.

------

# 🟢 LEVEL 1 – KERNEL ANATOMY

> **“Body samajhna, taaki chot kahan lagi hai pata chale”**

------

# 🔵 1.1 Kernel Memory Anatomy

> **Kernel RAM ko kaise dekhta hai**

------

## 1️⃣ Kernel Virtual Address Space (Foundation)

### ❓ Virtual address space kya hota hai?

Kernel **direct physical RAM** use nahi karta
Wo **virtual addresses** use karta hai.

👉 Har pointer jo kernel me hota hai:

```
0xffff888012345678
```

ye **virtual address** hota hai

------

### User space vs Kernel space

| Space        | Typical range (ARM64 / x86_64) |
| ------------ | ------------------------------ |
| User space   | 0x0000_0000_0000_0000 –        |
| Kernel space | 0xffff_8000_0000_0000 –        |

🧠 **Rule:**

> Agar panic address `0xffff...` se start ho
> to wo **kernel memory** hai

------

### Debugger mindset

> Panic log me address dekhte hi
> pehla sawal:
> **“Ye kernel ka kaunsa area hai?”**

------

## 2️⃣ Linear Map (Direct Mapping)

### ❓ Linear map kya hota hai?

Kernel RAM ka **direct mapping**:

```
Physical RAM  ──────▶  Kernel virtual address
```

Formula (simplified):

```
kernel_va = phys_addr + PAGE_OFFSET
```

------

### Characteristics

- **Contiguous**
- Fast
- Mostly used for:
  - Page cache
  - Slab pages
  - Normal allocations

------

### Example

```
ffff888000000000  <-- Linear map start
```

Agar panic address:

```
ffff88801234abcd
```

🧠 Turant bolo:

> “Ye **normal RAM** ka access hai
> kmalloc / slab related ho sakta hai”

------

### Common bugs here

- Use-after-free
- Slab corruption
- DMA overwrite

------

## 3️⃣ vmalloc Area (Non-contiguous mapping)

### ❓ vmalloc kyun chahiye?

Kabhi-kabhi kernel ko:

- **Large memory**
- **Contiguous virtual**, but
- **Physical pages scattered**

chahiye

👉 Tab use hota hai **vmalloc**

------

### Characteristics

- Virtual contiguous
- Physical non-contiguous
- Slower than linear map
- Used for:
  - Huge buffers
  - Debug tools
  - Some drivers

------

### Address pattern

```
ffffc90000000000  <-- vmalloc region (example)
```

Agar panic address yahan ka ho:

🧠 Debugger bole:

> “Ye **vmalloc memory** hai
> free / access mismatch ho sakta hai”

------

### Typical vmalloc bugs

- vmalloc memory ko `kfree()` kar diya
- vmalloc pointer ko DMA me use kar liya

------

## 4️⃣ Module Memory

### ❓ Kernel modules kahan rehte hain?

Loadable modules (`.ko`)
**separate memory region** me load hote hain

------

### Sections

- Module `.text`
- Module `.data`
- Module `.bss`

Address pattern:

```
ffffffffc0000000  <-- module region (example)
```

------

### Panic example

```
PC is at snd_soc_probe+0x34/0x120 [snd_soc_core]
```

🧠 Samajh jao:

> “Crash **module ke code** me hai
> core kernel nahi”

------

### Debug power

- Faulty driver ko isolate kar paoge
- “Kernel bug” vs “Driver bug” clear

------

## 5️⃣ Per-CPU Memory

### ❓ Per-CPU memory kya hoti hai?

Har CPU ke liye **alag memory copy**

Used for:

- Stats
- Counters
- Fast paths
- Lockless data

------

### Address look

- Usually **strange offsets**
- Different CPU → different address

------

### Common bugs

- Wrong CPU access
- Preemption ke bina per-cpu access
- Pointer store kar ke baad me use

🧠 Symptom:

> Bug sirf SMP / multi-core me aata hai

------

# 🔴 1.1 Debug Summary (VERY IMPORTANT)

| Address area    | Matlab           |
| --------------- | ---------------- |
| `ffff8880...`   | Linear map (RAM) |
| `ffffc9...`     | vmalloc          |
| `ffffffffc0...` | Module           |
| Per-cpu         | SMP-related bug  |

👉 **Panic address = first diagnosis**

------

# 🟣 1.2 Kernel Binary Anatomy

> **Kernel ka sharir, disk par aur RAM me**

------

## 1️⃣ vmlinux vs Image

### 🔹 vmlinux

- ELF file
- Full symbols
- Debugging ke liye

Used for:

- gdb
- addr2line
- crash utility

🧠 **Runtime nahi**, diagnosis ke liye

------

### 🔹 Image / bzImage

- Compressed
- Bootable
- No symbols

Used by:

- Bootloader

🧠 **Boot karta hai, debug nahi**

------

## 2️⃣ System.map

### ❓ System.map kya hai?

- Symbol → address mapping
- Build time generated

Example:

```
ffffffff81012340 T start_kernel
```

🧠 Panic address ko function name me badalne ke kaam aata hai

------

## 3️⃣ kallsyms

### ❓ kallsyms kya karta hai?

Kernel ke andar hi:

- Address → symbol translation

Isliye panic log me:

```
PC is at do_page_fault+0x20/0x80
```

Aata hai

🧠 Agar kallsyms off:

> Panic log almost useless ho jaata hai

------

## 4️⃣ Kernel Sections (`.text .data .bss __init`)

------

### 🔹 `.text`

- Executable code
- Crash here → instruction fault

🧠 “Code bug”

------

### 🔹 `.data`

- Initialized global variables

Bug:

- Corrupted global state

------

### 🔹 `.bss`

- Uninitialized globals
- Zero-filled

Bug:

- Unexpected zero
- Overwrite

------

### 🔹 `__init`

- Boot-time code
- Boot ke baad **free ho jaata hai**

🔥 **Classic bug**:

```c
__init int foo(void) { ... }

ptr = foo;   // ❌ after init
```

👉 Random crash later

------

## 🧠 FINAL SUPERPOWER (GOAL ACHIEVED)

Ab tum:

- Panic address dekh kar bata sakte ho:
  - RAM?
  - vmalloc?
  - Module?
- Function kaunsa section me tha
- Boot-time bug ya runtime bug

------

## 🔚 LEVEL 1 COMPLETE

### Next logical jump:

👉 **LEVEL 2 – Crash Types (panic, oops, lockup, stall)**
ya
👉 **Live example: ek panic log lekar “address → area → root cause”**

Batao next kya chahiye.