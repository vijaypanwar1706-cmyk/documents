Theek hai 👍
Main **Hinglish me line-by-line meaning + deep explanation** dunga, aur jahan zarurat ho **real-life examples** bhi add karunga, taaki kernel/interrupt concept solid ho jaaye — especially tum jaise driver & kernel padhne waale ke liye.

------

## Kernel ka core responsibility – Hardware manage karna

**Original idea (Hinglish):**
Har operating system kernel ka ek **core kaam** hota hai machine se connected hardware ko manage karna — jaise hard drives, Blu-ray drives, keyboard, mouse, GPU (3D processors), Wi-Fi/Bluetooth radios, etc.

### Explanation

Kernel **hardware aur software ke beech ka middleman** hota hai.
Apps directly hardware se baat **nahi** karti, balki:

```
Application → Kernel → Device Driver → Hardware
```

Agar kernel ye kaam na kare, to:

- Har app apna driver likhegi ❌
- System unstable ho jaayega ❌

------

## Problem: CPU bahut fast, hardware slow

**Original idea (Hinglish):**
Processor hardware ke comparison me **orders of magnitude faster** hota hai. Isliye agar kernel har request ke baad hardware ka response wait kare, to CPU ka time waste hoga.

### Real-life example

Socho:

- CPU = Formula-1 car 🏎️
- Hard disk = Bullock cart 🐂

Agar F1 car ko har 10 meter pe bullock cart ka wait karna pade, to kya fayda?

Isi tarah:

```c
read_from_disk();
wait_till_disk_finishes(); // ❌ CPU waste
```

Ye **bad design** hai.

------

## Solution: Kernel ko free rehna chahiye

**Original idea (Hinglish):**
Hardware slow hota hai, isliye kernel ko request dene ke baad **free hona chahiye**, taaki wo doosra kaam kar sake. Jab hardware apna kaam complete kare, tab kernel usse deal kare.

### Matlab

Kernel bolta hai:

> “Maine kaam bol diya, tu khatam kare to mujhe bata dena.”

Aur kernel meanwhile:

- doosre process chalaata hai
- scheduling karta hai
- memory manage karta hai

------

## Polling – ek possible solution (but bad)

**Polling ka matlab:**
Kernel baar-baar hardware ka status check karta rahe:

```c
while (!device_ready)
    check_status();
```

### Problem with polling

- Hardware ready ho ya na ho → kernel baar-baar check karta rahe ❌
- CPU cycles waste ❌
- Power consumption zyada ❌

### Real-life example

Tum kisi friend se milne ja rahe ho aur har 10 sec phone karke pooch rahe ho:

> “Aa gaye kya?” 📞

Irritating + inefficient.

------

## Better solution: Interrupts 🔔

**Original idea (Hinglish):**
Hardware khud kernel ko signal kare jab usse attention chahiye. Is mechanism ko **interrupt** kehte hain.

### Real-life example (best one)

Polling =
Tum gate ke bahar khade ho aur baar-baar door knock kar rahe ho 🚪

Interrupt =
Doorbell 🔔
Tum apna kaam kar rahe ho, bell baji → turant respond.

------

## Interrupt kya hota hai?

**Definition (Hinglish):**
Interrupt ek **electrical signal** hota hai jo hardware device processor ko bhejta hai, bolne ke liye:

> “Oye CPU, mujhe dekh!”

### Example: Keyboard

- Tum key press karte ho
- Keyboard controller interrupt bhejta hai
- CPU OS ko bolta hai
- OS key read karta hai

Isliye typing **instant** lagti hai.

------

## Interrupt asynchronous hote hain

**Matlab:**
Interrupts **CPU clock ke saath sync me nahi hote**
→ kabhi bhi aa sakte hain

Isliye:

- Kernel kisi bhi time interrupt ho sakta hai
- Kernel ko hamesha ready rehna padta hai

------

## Interrupt controller ka role

Hardware directly CPU se baat nahi karta.

Flow:

```
Device → Interrupt Controller → CPU → Kernel
```

Interrupt controller:

- Multiple devices ke interrupts ko manage karta hai
- CPU ko ek proper signal deta hai

### Simple example

Office reception desk 📞
Sab calls receptionist ke paas → wo boss ko forward karti hai.

------

## IRQ – Interrupt Request lines

Har interrupt ka ek **unique number** hota hai → IRQ number.

### PC example:

- IRQ 0 → Timer
- IRQ 1 → Keyboard

But:

- PCI devices me IRQ **dynamically assign** hote hain
- Modern systems me mapping flexible hoti hai

### Important baat

Kernel ko pata hota hai:

> “Kaunsa IRQ = kaunsa device”

Isliye kernel samajh jaata hai:

> “Ye interrupt keyboard ka hai ya network card ka?”

------

## Exceptions vs Interrupts

### Interrupt

- Hardware generate karta hai
- Asynchronous
- Example: keyboard, network card

### Exception

- CPU khud generate karta hai
- Synchronous
- Example:
  - Divide by zero
  - Page fault
  - Invalid instruction

### System call bhi exception hai

x86 me:

- System call = software interrupt
- Hardware nahi, **software** trigger karta hai

------

## Interrupt Handler / ISR

**Interrupt handler kya hota hai?**
Kernel ka wo function jo **interrupt aane par run hota hai**.

### Example

- Keyboard interrupt → keyboard ISR
- Timer interrupt → timer ISR

ISR device driver ka part hota hai.

------

## Interrupt context (atomic context)

Interrupt handler:

- Normal process context me nahi chalta
- **Interrupt context** me chalta hai

### Important restriction

❌ Sleep nahi kar sakta
❌ Blocking calls nahi
❌ schedule() nahi

Isliye ise **atomic context** bhi kehte hain.

------

## ISR fast kyun hona chahiye?

Interrupt:

- Kisi bhi time aa sakta hai
- Agar ISR slow hua → system sluggish

Isliye rule:

> **Interrupt handler = short & fast**

Minimum kaam:

- Interrupt acknowledge karo
- Hardware ko bolo: “Samajh gaya”

------

## Network card example (Top half vs Bottom half)

### Jab packet aata hai:

1. Network card interrupt bhejta hai
2. ISR (top half):
   - Interrupt acknowledge
   - Packet ko **main memory me copy**
   - Card ko ready karta hai

### Kyun copy jaldi?

- Network card ka buffer **bahut chhota**
- Delay hua → buffer overrun → packet drop ❌

Baaki ka heavy kaam:

- Protocol processing
- Application ko data dena
  ➡️ **Bottom half me hota hai**

------

## Top half vs Bottom half

| Top Half          | Bottom Half               |
| ----------------- | ------------------------- |
| ISR               | Deferred work             |
| Fast              | Slow allowed              |
| Interrupt context | Process / softirq context |
| Time-critical     | Non-critical              |

------

## request_irq() – Interrupt register karna

Driver interrupt handler register karta hai:

```c
int request_irq(
    unsigned int irq,
    irq_handler_t handler,
    unsigned long flags,
    const char *name,
    void *dev
);
```

### Parameters samjho:

1️⃣ **irq**
Interrupt number (fixed ya dynamically detected)

2️⃣ **handler**
ISR function pointer

3️⃣ **flags**
Important flags:

- `IRQF_SHARED` → shared interrupt
- `IRQF_DISABLED` → fast interrupt (rare)
- `IRQF_SAMPLE_RANDOM` → entropy ke liye

4️⃣ **name**
/proc/interrupts me dikhane ke liye

5️⃣ **dev**

- Shared IRQ ke liye **unique cookie**
- Usually device structure pointer

------

## request_irq() sleep kyun kar sakta hai?

Kyuki:

- /proc/irq entry banata hai
- `kmalloc()` call hota hai
- kmalloc **sleep kar sakta hai**

Isliye:
❌ Interrupt context me call mat karo
✔️ Process context me hi karo

------

## free_irq() – Interrupt unregister karna

Driver unload pe:

```c
free_irq(irq, dev);
```

- Shared IRQ me sirf wahi handler remove hota hai jiska `dev` match kare
- Last handler remove hone par IRQ disable hota hai

------

## Interrupt handler ka prototype

```c
static irqreturn_t handler(int irq, void *dev);
```

### Return values:

- `IRQ_HANDLED` → interrupt mera tha
- `IRQ_NONE` → mera nahi tha (shared IRQ case)

Kernel isse detect karta hai:

- Spurious interrupts

------

## Reentrancy ka tension nahi

Linux me:

- Same IRQ ek time pe dobara nahi aata
- Wo IRQ masked rehta hai

Isliye:
✔️ ISR ko reentrant hone ki zarurat nahi

------

## Shared interrupt handlers

Rules:
1️⃣ `IRQF_SHARED` flag mandatory
2️⃣ `dev` pointer **unique hona chahiye**
3️⃣ ISR ko check karna padega:

> “Kya interrupt mera device ka hai?”

Hardware me usually status register hota hai:

```c
if (!device_interrupt_pending())
    return IRQ_NONE;
```

------

## Bottom line (exam + interview ready)

- Interrupt = hardware ka “attention signal”
- Polling inefficient hai
- ISR fast hona chahiye
- Heavy kaam bottom half me
- request_irq() process context me
- Interrupt context me sleep ❌

------

Agar chaho to next step me main:

- **ISR ka real Linux code walk-through**
- **atomic context rules list**
- **softirq vs tasklet vs workqueue**
- **ALSA / network driver ISR mapping**

bol do 👍







Perfect 👍
Is section me **real Linux kernel ka interrupt handler** aata hai, isliye main isko **Hinglish + deep explanation + real-life analogy + code walkthrough** ke saath samjha raha hoon. Ye interview + actual driver understanding dono ke liye gold hai.

------

# A Real-Life Interrupt Handler (RTC example)

## RTC (Real-Time Clock) kya hota hai?

RTC ek **hardware device** hota hai jo:

- System date/time maintain karta hai
- Alarm set karta hai
- Periodic timer generate karta hai

⚠️ **Important:**
RTC **system timer se alag** hota hai.

### Real-life example

RTC = **Wall clock + alarm clock ⏰**
System timer = **CPU ka internal stopwatch**

Jab alarm bajta hai → ek **interrupt** generate hota hai
Bilkul waise jaise real clock ka alarm buzzer 🔔

------

## RTC interrupt ka use kyun hota hai?

- Time update complete hua
- Alarm trigger hua
- Periodic timer tick aaya

👉 In sab cases me RTC **interrupt bhejta hai**

------

## Driver load hone par kya hota hai?

Jab RTC driver load hota hai:

```c
rtc_init()
```

Is function ka ek kaam hota hai:
👉 **Interrupt handler register karna**

------

## RTC interrupt register karna (request_irq)

```c
if (request_irq(rtc_irq, rtc_interrupt,
                IRQF_SHARED,
                "rtc",
                (void *)&rtc_port)) {
    printk(KERN_ERR "rtc: cannot register IRQ %d\n", rtc_irq);
    return -EIO;
}
```

### Line-by-line samjho

### 1️⃣ `rtc_irq`

- RTC ka interrupt number
- PC par usually **IRQ 8**

### 2️⃣ `rtc_interrupt`

- Actual ISR (Interrupt Service Routine)

### 3️⃣ `IRQF_SHARED`

- RTC interrupt **shared hota hai**
- Isliye ye flag mandatory hai

### 4️⃣ `"rtc"`

- `/proc/interrupts` me name dikhega

### 5️⃣ `(void *)&rtc_port`

- **Unique dev cookie**
- Shared interrupt ke liye zaroori

📌 Agar ye fail hota hai → driver load fail

------

## Ab actual interrupt handler (ISR)

```c
static irqreturn_t rtc_interrupt(int irq, void *dev)
```

👉 Jab bhi RTC interrupt aayega
👉 Ye function **turant execute hoga**

------

## ISR ke andar kya ho raha hai?

### Step 1: Spinlock – kyun?

```c
spin_lock(&rtc_lock);
```

### Kyun zaroori?

- SMP system (multiple CPU cores)
- Multiple CPUs ek saath RTC data access kar sakte hain
- Data corruption se bachne ke liye lock

📌 Interrupt context me:

- **mutex ❌**
- **sleep ❌**
- sirf **spinlock ✔️**

------

## rtc_irq_data update

```c
rtc_irq_data += 0x100;
rtc_irq_data &= ~0xff;
rtc_irq_data |= (CMOS_READ(RTC_INTR_FLAGS) & 0xF0);
```

### Iska matlab (simple terms me):

`rtc_irq_data` ek `unsigned long` hai jisme:

- **Upper bytes** → interrupt count
- **Lower byte** → interrupt type/status

### Real-life analogy

Socho ek register:

- Upper part = “Kitni baar alarm baja”
- Lower part = “Alarm / periodic / update?”

------

## RTC status register read karna

```c
CMOS_READ(RTC_INTR_FLAGS)
```

👉 Hardware se directly puchha ja raha hai:

> “Kaunsa type ka interrupt tha?”

------

## Periodic timer update

```c
if (rtc_status & RTC_TIMER_ON)
    mod_timer(&rtc_irq_timer,
              jiffies + HZ/rtc_freq + 2*HZ/100);
```

### Meaning:

Agar RTC periodic timer ON hai:

- Next timer event schedule karo

📌 `mod_timer()`:

- Kernel timer update karta hai
- ISR me allowed hai (sleep nahi karta)

------

## First spinlock unlock

```c
spin_unlock(&rtc_lock);
```

👉 Ab RTC internal data safe hai

------

## Callback function execution

```c
spin_lock(&rtc_task_lock);
if (rtc_callback)
    rtc_callback->func(rtc_callback->private_data);
spin_unlock(&rtc_task_lock);
```

### Ye kya hai?

RTC driver:

- Users ko allow karta hai
- Ek **callback function register** karne ke liye
- Har RTC interrupt pe ye callback chalega

### Real-life example

Alarm bajte hi:

- Tumhara phone vibration + ringtone + screen on
- Ye sab **callbacks** jaise hain

------

## Waiting processes ko jagana

```c
wake_up_interruptible(&rtc_wait);
```

👉 Jo process RTC event ka wait kar raha tha
👉 Ab usko jagao

### Example

User program:

```c
read(/dev/rtc);
```

Wo sleep me tha → interrupt aaya → wake up

------

## Async notification (SIGIO)

```c
kill_fasync(&rtc_async_queue, SIGIO, POLL_IN);
```

👉 Async I/O users ko signal bhejna
👉 “RTC event aaya hai”

------

## ISR ka end

```c
return IRQ_HANDLED;
```

👉 Kernel ko bola:

> “Haan, interrupt maine handle kiya”

⚠️ RTC driver:

- Shared interrupt support karta hai
- Lekin **spurious interrupt detect nahi kar sakta**
- Isliye **hamesha IRQ_HANDLED return karta hai**

------

# Interrupt Context (bahut important)

## Process context vs Interrupt context

### Process context

- current process hota hai
- sleep allowed
- schedule allowed

### Interrupt context

- ❌ koi process nahi
- ❌ sleep nahi
- ❌ blocking nahi

```c
current   // meaningless (sirf interrupted task dikhaata hai)
```

👉 Isliye:

- `kmalloc(GFP_KERNEL)` ❌
- `mutex_lock()` ❌
- `schedule()` ❌

------

## ISR hamesha fast kyun?

Interrupt:

- Kisi bhi code ko beech me rok deta hai
- Kabhi user code
- Kabhi kernel code
- Kabhi doosra interrupt handler

Isliye:

> **ISR = minimum work**

Heavy kaam → **bottom half**

------

## Kernel stack & Interrupt stack

### Pehle kya hota tha?

- ISR process ke kernel stack ko use karta tha
- Stack size:
  - 32-bit → 8KB
  - 64-bit → 16KB

### Problem

- Stack overflow ka risk
- Memory waste

------

## 2.6 ke baad kya change hua?

- Kernel stack → **4KB**
- Har CPU ke liye:
  - **Dedicated interrupt stack**
  - Size = 1 page

### Benefit

- ISR ko poora page milta hai
- Overall memory efficient

📌 Lekin rule same:

> **ISR me stack usage minimum rakho**

------

# Interrupt ka full journey (hardware → kernel)

### Flow samjho:

```
Hardware
   ↓
Interrupt Controller
   ↓
Processor pin
   ↓
Assembly entry code
   ↓
do_IRQ()
   ↓
handle_IRQ_event()
   ↓
ISR(s)
   ↓
ret_from_intr()
   ↓
Resume execution
```

------

## do_IRQ() ka kaam

```c
unsigned int do_IRQ(struct pt_regs regs)
```

### Kya karta hai?

- Registers save karta hai
- IRQ number nikalta hai
- Interrupt acknowledge karta hai
- Line disable karta hai
- Handler call karta hai

------

## handle_IRQ_event()

👉 Yahin **actual ISR execution hota hai**

### Important steps:

1. Agar `IRQF_DISABLED` nahi hai → interrupts enable
2. Sab handlers ko call karo (shared case)
3. `IRQ_HANDLED / IRQ_NONE` check
4. Randomness add (agar flag set ho)
5. Interrupts disable karke return

------

## ret_from_intr()

Assembly code:

- Check: `need_resched`
- Agar:
  - User space me return + reschedule needed → `schedule()`
  - Kernel space + `preempt_count == 0` → `schedule()`

Phir:

- Registers restore
- Execution resume

------

# Exam / Interview summary (must remember)

- RTC interrupt = alarm clock
- ISR runs in **interrupt context**
- Interrupt context me:
  - sleep ❌
  - mutex ❌
  - blocking ❌
- ISR fast hona chahiye
- Heavy work → bottom half
- Shared interrupt:
  - `IRQF_SHARED`
  - unique `dev`
  - device-specific check

------

Agar chaho to next step me main:

- **RTC ISR ko simplify karke pseudo-code**
- **ISR vs threaded IRQ**
- **Hard IRQ vs Soft IRQ**
- **Interview questions from this chapter**

bas bolo 👍



Bilkul 👍
Is poore section ko main **Hinglish me translate + deeply explain + real-life examples** ke saath samjha raha hoon. Ye part **interrupt debugging, driver dev aur interviews** ke liye bahut important hai.

------

# `/proc/interrupts` kya hota hai?

## Procfs (Process File System)

**procfs ek virtual filesystem hota hai**
👉 Ye disk par real files nahi hoti
👉 Ye **sirf kernel memory me exist karta hai**
👉 Usually `/proc` par mounted hota hai

### Simple language me

`/proc` = **kernel ka dashboard 📊**

Jab tum:

```bash
cat /proc/interrupts
```

karte ho
to kernel ek function chalata hai jo **live interrupt data generate** karta hai.

------

## `/proc/interrupts` ka kaam

Ye file:

- System me **kaun-kaun se interrupts aa rahe hain**
- **kitni baar** aa chuke hain
- **kaunse device se**
- **kaunse CPU par**

ye sab batati hai.

------

## Sample Output (Uniprocessor PC)

```
           CPU0
0:   3602371  XT-PIC  timer
1:      3048  XT-PIC  i8042
2:         0  XT-PIC  cascade
4:   2689466  XT-PIC  uhci-hcd, eth0
5:         0  XT-PIC  EMU10K1
12:    85077  XT-PIC  uhci-hcd
15:    24571  XT-PIC  aic7xxx
NMI:        0
LOC:  3602236
ERR:        0
```

------

## Column-by-column samjho

### 🟦 First Column – Interrupt Line (IRQ number)

```
0, 1, 2, 4, 5, 12, 15
```

👉 Ye IRQ numbers hain
👉 Jin IRQ par **handler registered hai wahi dikhenge**

Agar koi IRQ list me nahi hai →
❌ uspe koi handler nahi laga

------

### 🟦 Second Column – Interrupt Count

Example:

```
0: 3602371
```

👉 IRQ 0 (timer) **36 lakh baar fire ho chuka hai**

Real-life example:

- Doorbell ka counter
- Kitni baar bell baji

📌 EMU10K1 (sound card) ka count `0` hai
→ Matlab system boot ke baad sound card use hi nahi hua

------

### 🟦 CPU Columns

```
CPU0
```

👉 Har CPU ke liye alag column hota hai
👉 Ye system **single-core** hai isliye sirf CPU0

Multi-core me:

```
CPU0 CPU1 CPU2 CPU3
```

------

### 🟦 Interrupt Controller

```
XT-PIC
```

👉 Ye batata hai **kaunsa interrupt controller handle kar raha hai**

- `XT-PIC` → old PC programmable interrupt controller
- `IO-APIC-level / IO-APIC-edge` → modern systems

📌 Aaj ke systems me mostly **IO-APIC** hota hai

------

### 🟦 Last Column – Device Name

```
timer
i8042
uhci-hcd, eth0
```

👉 Ye naam **request_irq() ke `name` parameter se aata hai**

```c
request_irq(irq, handler, flags, "rtc", dev);
```

Agar interrupt **shared** hai:

```
uhci-hcd, eth0
```

👉 Dono devices same IRQ share kar rahe hain

------

## `/proc/interrupts` ka use kyun important hai?

### Debugging

- Kya interrupt aa raha hai ya nahi?
- Interrupt storm ho raha hai?
- Kaunsa device zyada interrupts generate kar raha hai?

### Real-life debugging example

Agar network slow hai:

```bash
watch -n1 cat /proc/interrupts
```

Aur eth0 ka counter badh hi nahi raha
→ Problem driver / IRQ routing me hai

------

## procfs ka code kaha hota hai?

👉 Mostly:

```
fs/proc/
```

👉 `/proc/interrupts` ka function:

```
show_interrupts()
```

⚠️ Ye **architecture-dependent** hota hai
(x86, ARM, etc ke liye alag)

------

# Interrupt Control (Interrupt ko control karna)

Kernel ke paas **interrupt ko control karne ke tools** hote hain:

- Disable / Enable interrupts
- Sirf ek IRQ band karna
- Check karna: interrupt context me ho ya nahi

------

## Interrupt control kyun chahiye?

Main reason = **Synchronization**

### Problem

- Interrupt handler data access kar raha hai
- Same time process bhi wahi data access kar raha hai

### Solution

- Interrupt temporarily disable
- Lock lagao

📌 Important:

- Interrupt disable = ❌ SMP protection nahi
- Dusra CPU phir bhi access kar sakta hai
- Isliye **spinlock + interrupt disable** combo use hota hai

------

# Local Interrupt Disable / Enable

```c
local_irq_disable();
/* interrupts band */
local_irq_enable();
```

### Matlab

- Sirf **current CPU** ke interrupts band
- Dusre CPUs unaffected

### x86 me

- `cli` → disable
- `sti` → enable

------

## DANGER ⚠️ – local_irq_disable() ka issue

Agar interrupts pehle se disabled the:

```c
local_irq_disable();
/* ... */
local_irq_enable();
```

❌ Ye blindly interrupts ON kar dega
Chahe pehle OFF hi kyun na ho

------

## Safe method – save & restore

```c
unsigned long flags;

local_irq_save(flags);
/* interrupts disabled */

local_irq_restore(flags);
/* original state restored */
```

### Kyun better hai?

- Pehle ka state yaad rakhta hai
- Wapas **same state** me restore karta hai

📌 Rule:

- `flags` **unsigned long** hona chahiye
- save aur restore **same function** me hone chahiye

------

## Ye functions kaha use ho sakte hain?

✔ Interrupt context
✔ Process context

------

# No More Global `cli()` 😌

Pehle kernel me:

- `cli()` → poore system ke interrupts band
- `sti()` → poore system ke interrupts ON

### Problems

- Global lock jaisa behave karta tha
- SMP performance kharab
- Driver writers lazy ho jaate the 😄

------

## Aaj kya use hota hai?

- `local_irq_disable()`
- `spinlock`
- Fine-grained locking

### Fayde

- Fast
- Clean
- Scalable
- SMP friendly

------

# Specific Interrupt Line Disable karna

Kabhi-kabhi:
👉 Sirf **ek device ka interrupt** band karna hota hai

Example:

- Device reconfigure karte time

------

## Available APIs

```c
disable_irq(irq);
disable_irq_nosync(irq);
enable_irq(irq);
synchronize_irq(irq);
```

------

### `disable_irq(irq)`

- IRQ line band
- Jab tak **current handler finish na ho** wait karta hai

### `disable_irq_nosync(irq)`

- IRQ band
- Wait nahi karta

### `synchronize_irq(irq)`

- Sirf wait karta hai
- Disable nahi karta

### `enable_irq(irq)`

- IRQ wapas enable

------

## Nesting ka rule

```c
disable_irq(irq);
disable_irq(irq);

enable_irq(irq);  // still disabled
enable_irq(irq);  // now enabled
```

------

## Shared IRQ ke saath danger ⚠️

Agar IRQ shared hai:

- Ek driver IRQ disable karega
- Saare devices ke interrupts band ho jayenge

📌 Isliye:

- PCI devices ❌ use nahi karte
- Mostly legacy ISA devices me use hota hai

------

# Interrupt System Status Check

## Interrupt enabled hai ya nahi?

```c
irqs_disabled()
```

- non-zero → disabled
- zero → enabled

------

## Context check macros

```c
in_interrupt()
in_irq()
```

### `in_interrupt()`

- ISR ya bottom half → TRUE

### `in_irq()`

- Sirf hard IRQ handler → TRUE

### Process context check

```c
if (!in_interrupt()) {
    // safe to sleep
}
```

------

# Summary Table (Yaad rakhne layak)

| Function           | Kaam                      |
| ------------------ | ------------------------- |
| local_irq_disable  | Local CPU interrupts band |
| local_irq_enable   | Local CPU interrupts ON   |
| local_irq_save     | State save + disable      |
| local_irq_restore  | State restore             |
| disable_irq        | IRQ line band + wait      |
| disable_irq_nosync | IRQ band, no wait         |
| enable_irq         | IRQ enable                |
| irqs_disabled      | Interrupt state           |
| in_interrupt       | Interrupt context?        |
| in_irq             | Hard IRQ context?         |

------

# Final Conclusion (Big Picture)

- Interrupt = hardware ka **attention signal**
- `/proc/interrupts` = live interrupt monitor
- Interrupt fast hone chahiye
- Synchronization ke liye:
  - spinlock
  - local_irq_save/restore
- Global `cli()` obsolete
- Modern kernel = fine-grained locking
- Heavy kaam → bottom half

------

Agar chaho, next me main:

- `/proc/interrupts` ko **live analyze karna sikhaun**
- **HZ se uptime calculate** karna
- **Interview Q&A from this chapter**
- **Bottom halves (softirq, tasklet, workqueue)** start karun

Bas bolo 👍