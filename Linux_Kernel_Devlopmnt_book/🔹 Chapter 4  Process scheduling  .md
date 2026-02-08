Chalo isko **step-by-step Hinglish me**, bilkul **real-life examples** ke saath samajhte hain. Ye chapter thoda heavy lagta hai, lekin concept clear ho gaya to **scheduler = pure OS ka dimaag** samajh aane lagta hai.

------

## 1️⃣ Process Scheduling kya hota hai?

Pichhle chapter me humne **process** padha tha —
👉 *running program ka OS-level form*.

Ab yahan baat ho rahi hai **Process Scheduler** ki.

### 🔹 Scheduler ka kaam kya hai?

Scheduler ek **kernel ka subsystem** hai jo decide karta hai:

- **Kaunsa process run kare?**
- **Kab run kare?**
- **Kitni der tak run kare?**

### 🧠 Simple words me:

CPU ek hi hai (ya limited CPUs hain),
lekin **process bahut saare**.

👉 Scheduler **CPU ka time baantta** hai sabke beech.

------

## 🧁 Real-life example

Socho:

- CPU = **1 ATM machine**
- Processes = **log line me khade**

Scheduler decide karega:

- Kaun pehle ATM use kare
- Kitni der tak kare
- Aur kab next ko chance mile

------

## 2️⃣ Multitasking kya hota hai?

### Multitasking OS ka matlab:

Ek se zyada process ko **ek hi time pe chalne ka illusion** dena.

### 🔹 Single-core machine:

- Ek time pe sirf **1 process actual me run**
- Baaki sab **very fast switching** ki wajah se chal rahe lagte hain

### 🔹 Multi-core machine:

- Alag-alag cores pe **actual parallel execution**
- 4 cores ⇒ 4 processes real me saath-saath

------

## 3️⃣ Process States (important)

System me bahut saare process memory me hote hain, lekin:

- 🟢 **Runnable** → run kar sakta hai
- 🔴 **Sleeping / Blocked** → kisi event ka wait
  (keyboard input, mouse click, network packet, timer)

👉 Isliye ho sakta hai:

- 200 processes memory me
- Sirf **1–2 runnable**

------

## 4️⃣ Multitasking ke types

### 1️⃣ Cooperative Multitasking ❌ (old & bad)

Isme:

- Process **khud decide karta** hai kab CPU chhodna hai
- Isko kehte hain **yielding**

#### 😬 Problems:

- Agar process **yield hi na kare** → system hang
- OS control me nahi hota

🧠 Example:

> Meeting me ek aadmi bolta hi ja raha hai, mic chhod hi nahi raha 😤

🕰️ Examples:

- Windows 3.1
- Mac OS 9

------

### 2️⃣ Preemptive Multitasking ✅ (Linux, Unix)

Isme:

- **Scheduler zabardasti CPU chheen leta hai**
- Process ko bina pooche rok diya jaata hai

👉 Isko kehte hain **Preemption**

🧠 Example:

> Teacher bolta hai:
> “Bas, time over. Agla student aao.”

Linux **hamesha preemptive** raha hai 👍

------

## 5️⃣ Timeslice kya hota hai?

### ⏱️ Timeslice:

- CPU ka ek **chhota fixed time**
- Jitni der process ko run karne mile

Example:

- Har process ko 5 ms CPU time

👉 Timeslice ensure karta hai:

- Koi bhi process **CPU pe kabza na kare**
- Fairness bani rahe

🧠 Example:

> Sab bachchon ko 1-1 minute bolne ka time

------

## 6️⃣ Linux aur Timeslice twist 😎

Most OS:

- Fixed / dynamic **timeslice** use karte hain

Linux ka **CFS (Completely Fair Scheduler)**:

- Traditional timeslice use hi **nahi karta**
- Instead **fairness** pe focus karta hai

👉 Matlab:

> “Jisne jitna kam CPU liya, usko pehle chance do”

(isko baad me deeply samjhenge)

------

## 7️⃣ Linux Scheduler ka Evolution

### 🔹 Old Scheduler (1991 – 2.4 kernel)

- Simple
- Easy to understand
- ❌ Zyada processes / CPUs pe slow

------

### 🔹 O(1) Scheduler (2.5 – 2.6)

O(1) matlab:

- Scheduler ka decision time **constant**
- 10 process ho ya 10,000 → same time

#### Features:

- Per-CPU runqueue
- Fast & scalable
- Servers ke liye 🔥

#### Problem:

- **Interactive apps** ke liye weak
  (mouse lag, UI slow feel)

------

### 🔹 Interactive Process kya hota hai?

- Jisse user directly interact kare
  - Browser
  - Terminal
  - Media player
  - GUI apps

👉 Desktop ke liye ye **most important** hote hain

------

### 🔹 CFS – Completely Fair Scheduler (2.6.23+)

- Inspired by **fair scheduling**

- Idea:

  > CPU ko aise baanto jaise sab ek hi time pe chal rahe ho

- Desktop + Server dono ke liye balanced

👉 Aaj ka Linux **CFS use karta hai**

------

## 8️⃣ Scheduler Policy kya hoti hai?

### Policy = Scheduler ka behavior

Matlab:

- Kis type ke process ko priority mile
- Kab preempt kare
- Kaun wait kare

🧠 Policy system ka **“feel” decide karti hai**:

- Desktop smooth?
- Server fast?
- UI responsive?

------

## 9️⃣ I/O-Bound vs CPU-Bound Processes 🔥

### 1️⃣ I/O-Bound Process

- Zyada time **wait** karta hai
- Short bursts me CPU use karta hai

Examples:

- GUI apps
- Browser
- Terminal
- Mouse / keyboard input

🧠 Example:

> Customer service agent jo zyada time customer ka wait karta hai

------

### 2️⃣ CPU-Bound Process

- Lagataar **calculation**
- Rarely block hota hai

Examples:

- `ssh-keygen`
- Video encoding
- MATLAB
- Infinite loop 😈

🧠 Example:

> Factory machine jo bina rukke kaam karti rahe

------

## 🔄 Mixed Behavior (real world)

Bahut saare process **mix** hote hain:

- Word processor:
  - Normally I/O-bound (typing)
  - Kabhi-kabhi CPU spike (spell check)
- X Window Server:
  - I/O bhi
  - CPU bhi

------

## 🔟 Scheduler ke 2 conflicting goals

Scheduler ko **balance** banana hota hai:

### ⚡ Low Latency (fast response)

- User click kare → turant response

### 🚀 High Throughput (max work)

- CPU idle na rahe
- Zyada kaam complete ho

👉 Linux **I/O-bound ko thoda favor** karta hai
kyunki desktop experience smooth chahiye

लेकिन:

- CPU-bound processes ko ignore nahi karta
- Fairness maintain karta hai

------

## 🔚 One-line Summary

> **Process Scheduler CPU ka traffic police hai**
> jo decide karta hai kaun chalega, kab chalega, aur kitni der tak —
> taaki system fast bhi rahe aur fair bhi ⚖️

------

Agar chaho to next step me:

- **CFS ka internals (vruntime, red-black tree)**
- **Runqueue & context switch**
- **Real kernel structures**

bolna 👍







Perfect, ye **Process Priority + CFS** ka core chapter hai.
Main isko **Hinglish + real-life + contrast ke saath** explain kar raha hoon, taaki **CFS dimag me permanently baith jaaye**.

------

# 🔥 Process Priority (Linux way)

## 1️⃣ Priority-based Scheduling kya hota hai?

Basic idea (generic OS concept):

- **High priority process → pehle chale**
- **Low priority process → baad me chale**
- Same priority → **round-robin** (ek ke baad ek)

Kuch OS me:

- High priority process ko **longer timeslice** bhi milta hai

👉 User ya system dono priority set kar sakte hain.

⚠️ **Important:**
Linux exactly aise kaam nahi karta, especially **CFS ke baad**.

------

## 2️⃣ Linux me Priority ke 2 alag world 🌍🌍

Linux kernel **do alag priority ranges** implement karta hai:

------

## 🟢 (A) Nice Value — Normal Processes

### Nice value range:

```
-20  (highest priority)
  0  (default)
+19  (lowest priority)
```

👉 **Bigger nice value = lower priority**
👉 Matlab tum CPU ke saath “nice” ban rahe ho 😄

### Example:

- nice = 0 → normal
- nice = 10 → background
- nice = -10 → important task

### Linux me nice ka matlab:

❌ “Itna ms ka timeslice milega”
✅ “CPU ka **kitna hissa** milega”

📌 **Key point (very important):**

> Linux me nice value **absolute time control nahi karta**,
> balki **proportion of CPU** control karta hai.

------

### Nice values dekhna:

```bash
ps -el
```

Column: **NI**

------

## 🔴 (B) Real-Time Priority — RT Processes

### Range:

```
0 – 99
```

👉 Yahan:

- **Higher number = higher priority**
- RT process **hamesha normal process se upar**

Nice aur RT priority:
❌ overlap nahi karti
👉 **disjoint spaces**

### RT priority dekhna:

```bash
ps -eo state,uid,pid,ppid,rtprio,time,comm
```

- RTPRIO = `-` → normal process

📌 RT scheduling = POSIX standard (POSIX.1b)

------

## 3️⃣ Timeslice kya hota hai? ⏱️

### Timeslice:

> Kitni der tak process CPU pe chalega
> before being preempted

Scheduler ko decide karna hota hai:

- Default timeslice kitna ho?

------

## 4️⃣ Timeslice ka dilemma 😵‍💫

### ⛔ Long timeslice problem:

- UI laggy
- Mouse / keyboard response slow
- “System heavy lag raha hai”

### ⛔ Short timeslice problem:

- Zyada **context switching**
- CPU ka time waste
- Cache thandi ho jaati hai

------

### I/O-bound vs CPU-bound conflict (phir se)

- **I/O-bound**:
  - Short CPU burst
  - Frequently run hona chahiye
- **CPU-bound**:
  - Long continuous CPU chahiye
  - Cache warm rehni chahiye

👉 Ek hi timeslice sabke liye = **suboptimal**

------

## 5️⃣ Linux CFS ka revolution 🚀

Linux CFS bolta hai:

> ❌ “Main timeslice assign nahi karunga”
> ✅ “Main CPU ka **fair proportion** dunga”

------

## 6️⃣ CFS me CPU ka hisaab kaise hota hai?

### Core concept:

- Har process ko CPU ka **share** milta hai
- Ye depend karta hai:
  - System load
  - Nice value (weight)

------

### Nice value = weight ⚖️

- High nice (+ve) → **deflationary weight**
- Low nice (-ve) → **inflationary weight**

🧠 Simple:

> Nice value = “CPU ke liye bargaining power”

------

## 7️⃣ Preemption in CFS (super important)

Linux **preemptive** hai.

Jab koi process runnable hota hai:

- CFS check karta hai:

  > “Isne CPU **kitna kam** use kiya hai?”

### Rule:

- Jo process **least CPU use** kiya hai
  → **wo turant chalega**

👉 Even agar current process chal raha ho
👉 **Preempt kar diya jaayega**

------

## 8️⃣ Real-life scenario (text editor vs video encoder) 🎯

### 📝 Text Editor

- I/O-bound
- Mostly **sleeping**
- Jab key press aaye → **instant response expected**

### 🎥 Video Encoder

- CPU-bound
- 100% CPU kha sakta hai
- 0.5 sec late hua → user ko farak nahi

------

## 9️⃣ Traditional OS approach

Usually:

- Editor → high priority + big timeslice
- Encoder → low priority + small timeslice

⚠️ Problem:

- Heuristic based
- Gaming possible
- Complex tuning

------

## 🔥 Linux CFS approach (genius)

### Assume:

- 2 runnable processes
- Same nice value

👉 CPU share:

```
Editor: 50%
Encoder: 50%
```

------

### Reality:

- Editor **50% use hi nahi karta**
- Wo mostly **sleep** karta hai

👉 Encoder ko mil jaata hai:

- 80–90% CPU practically
- Encoding fast complete 👍

------

### Crucial moment: editor wakes up ⏰

User ne key press kiya…

CFS dekhta hai:

- Editor ne **bahut kam CPU** liya hai
- Encoder ne **bahut zyada CPU** liya hai

👉 CFS bolta hai:

> “Fairness restore karo!”

🔥 Encoder **preempt**
🔥 Editor **immediately run**

Editor:

- Key handle karta hai
- Phir wapas sleep 😴

👉 Result:

- Editor = buttery smooth
- Encoder = maximum throughput

💯 Best of both worlds

------

## 10️⃣ Scheduler Classes (Linux internals)

Linux scheduler **modular** hai.

### Scheduler Classes:

- Har class ek algorithm
- Har class ka **priority**

Kernel:

- Sab classes ko priority order me check karta hai
- Pehli class jisme runnable process mile → winner

------

### CFS ka role:

- Normal processes ke liye
- Linux name: `SCHED_NORMAL`
- POSIX name: `SCHED_OTHER`

📁 Code:

```
kernel/sched_fair.c
```

------

## 11️⃣ Old Unix Scheduler problems 😬

Traditional Unix model:

- Priority + timeslice

### ❌ Problem 1: Bad switching behavior

Example:

- nice 0 → 100 ms
- nice 20 → 5 ms

2 low-priority processes:

- 5 ms + 5 ms
- Context switch every **10 ms**

2 normal processes:

- 100 ms + 100 ms
- Switch every **200 ms**

👉 Background tasks = zyada switching 😖
(ulta hona chahiye)

------

### ❌ Problem 2: Nice value inconsistency

nice 0 → 1:

- 100 → 95 ms (small change)

nice 18 → 19:

- 10 → 5 ms (**50% cut**)

👉 “nice by 1” ka effect **predictable nahi**

------

### ❌ Problem 3: Timer tick dependency

- Timeslice = multiple of timer tick
- Tick = 1 ms ya 10 ms

Problems:

- Minimum timeslice limit
- Inconsistent mapping
- Kernel complexity

------

### ❌ Problem 4: Wake-up priority boost

- Interactive ke liye boost dena
- Lekin:
  - Sleep/wake pattern se scheduler ko **cheat** kiya ja sakta hai

------

## 12️⃣ CFS ka radical idea 💡

> ❌ Absolute timeslice mat do
> ✅ CPU ka **proportion** do

### Result:

- **Constant fairness**
- **Variable switching rate**

------

## 13️⃣ Fair Scheduling (Perfect Multitasking dream)

Ideal system (imaginary):

- n processes
- Har process = **1/n CPU**
- Sab ek saath chalte hain (virtually)

Example:

- 2 process → 50% + 50%
- 10 ms me dono ne 5–5 ms kaam kiya

👉 Isko kehte hain **perfect multitasking**

------

## 14️⃣ CFS ka practical approximation

Reality:

- Infinite switching impossible
- Switching cost hota hai

CFS kya karta hai:

- Least CPU-used process ko next run

- Timeslice = function of:

  ```
  (process weight) / (total weight)
  ```

------

## 15️⃣ Targeted Latency 🎯

CFS ka ek concept:

- **Targeted latency**

Example:

```
Targeted latency = 20 ms
```

- 2 processes → 10 ms each
- 4 processes → 5 ms each
- 20 processes → 1 ms each

------

## 16️⃣ Minimum Granularity 🧱

Problem:

- Zyada process → too small timeslice

Solution:

- **Minimum granularity = 1 ms**

👉 Isse:

- Switching cost control me rehta hai
- Thodi unfairness accept ki jaati hai (by design)

------

## 17️⃣ Nice values ka real effect (CFS magic)

Example:

- nice 0
- nice 5

👉 nice 5 ≈ **1/3 penalty**

Target latency = 20 ms:

- nice 0 → 15 ms
- nice 5 → 5 ms

Now:

- nice 10 & nice 15
  👉 STILL **15 ms & 5 ms**

🔥 Key insight:

> **Absolute nice value irrelevant**
> **Relative difference matters**

------

## 🧠 Final Big Picture Summary

- Old Unix:
  - Fixed timeslice
  - Priority mapping
  - Inconsistent fairness
- Linux CFS:
  - No timeslice thinking
  - CPU share thinking
  - Fairness first
  - Interactive apps naturally win
  - CPU-bound apps still efficient

👉 **CFS = fairness-driven scheduler**

------

Agar next step chaho to:

- **vruntime (virtual runtime)**
- **Red-Black Tree usage**
- **sched_entity, task_struct**
- **Real-time scheduler (SCHED_FIFO, SCHED_RR)**

Bas bolo 👍



Great, ab hum **CFS ka actual engine room** dekh rahe hain 🔧
Main isko **Hinglish + intuition + real-life analogies** ke saath explain karunga, taaki tumhe **code padhte waqt bhi picture clear rahe**.

------

# 🧠 Linux Scheduling Implementation (CFS Internals)

CFS ka logic samajh chuke ho (fair share, vruntime, no timeslice).
Ab sawal hai:

👉 **Kernel me ye sab actual me kaise hota hai?**

CFS implementation mainly **4 parts** me samajh aata hai:

1️⃣ Time Accounting
2️⃣ Process Selection
3️⃣ Scheduler Entry Point
4️⃣ Sleeping & Waking Up

Is message me hum **Time Accounting + Process Selection** cover kar rahe hain (yahi core hai).

------

## 1️⃣ Time Accounting – “Kaun kitni der chala?”

Har scheduler ko ye pata hona chahiye:

> “Kaunsa process kitna CPU le chuka hai?”

### Old Unix way ❌

- Har process ke paas **timeslice**
- Har timer tick pe timeslice decrement
- Zero hua → preempt

### CFS way ✅

- ❌ No timeslice
- ✅ **Virtual Runtime (vruntime)**

------

## 2️⃣ Scheduler Entity – CFS ka data holder 📦

CFS har process ko directly track nahi karta,
wo ek **scheduler entity** track karta hai.

### Structure:

```c
struct sched_entity {
    struct load_weight load;
    struct rb_node run_node;
    struct list_head group_node;
    unsigned int on_rq;

    u64 exec_start;
    u64 sum_exec_runtime;
    u64 vruntime;

    u64 prev_sum_exec_runtime;
    u64 last_wakeup;
    u64 avg_overlap;
    u64 nr_migrations;
    u64 start_runtime;
    u64 avg_wakeup;
};
```

👉 Ye structure **task_struct ke andar embedded hota hai**:

```c
task_struct -> se
```

### 🧠 Real-life analogy

Socho:

- `task_struct` = employee file
- `sched_entity` = **attendance + salary usage register**

------

## 3️⃣ vruntime – CFS ka heart ❤️

### vruntime kya hai?

> **Actual runtime**, lekin **weighted / normalized**

- Unit: **nanoseconds**
- Timer tick se **independent**
- Fairness measure

### Ideal world (perfect multitasking):

- Sab processes ka vruntime **same**
- Sabne equal CPU liya

### Real world:

- CPU ek hi hai
- Processes ek-ek karke chalte hain

👉 CFS bolta hai:

> “Jo process jitna zyada chal chuka,
> uska vruntime utna zyada”

Aur:

> “Jo kam chala hai, usko priority do”

------

## 4️⃣ update_curr() – Runtime ka hisaab ✍️

Jab bhi:

- Timer tick aata hai
- Process sleep karta hai
- Process wake hota hai

👉 `update_curr()` call hota hai

### Simplified flow:

```c
delta_exec = now - curr->exec_start;
```

👉 Matlab:

> “Last time se ab tak kitni der CPU pe tha?”

Agar delta_exec = 0 → kuch nahi hua

------

### Actual kaam: `__update_curr()`

```c
delta_exec_weighted = calc_delta_fair(delta_exec, curr);
curr->vruntime += delta_exec_weighted;
```

🧠 Important:

- `calc_delta_fair()` **nice value ka weight apply** karta hai
- High priority → vruntime **slow grow**
- Low priority → vruntime **fast grow**

### Analogy 🎯

- Rich employee (high priority): salary slowly cut hoti hai
- Intern (low priority): salary jaldi exhaust hoti hai

------

## 5️⃣ Bottom line of Time Accounting

- CFS continuously track karta hai:
  - Actual runtime
  - Weighted runtime (vruntime)
- vruntime hi decide karega:
  👉 **next kaun chalega**

------

# 🔥 Process Selection – “Ab kaun chalega?”

### CFS ka golden rule:

> **Run the process with the smallest vruntime**

Bas.
Ye hi poora scheduling logic hai 😄

------

## 6️⃣ Runnable processes ko kaise store kiya jaata hai?

CFS uses **Red-Black Tree (rbtree)** 🌳

### Key:

- **vruntime**

### Tree me:

- Har runnable process = ek node
- Ordered by vruntime

### Why rbtree?

- Insert → O(log N)
- Delete → O(log N)
- Smallest element → easy

------

## 7️⃣ Leftmost node = next process 🏃‍♂️

Binary search tree me:

- **Leftmost node = smallest key**

👉 Yahan:

- Smallest key = **smallest vruntime**

### Function:

```c
__pick_next_entity()
struct rb_node *left = cfs_rq->rb_leftmost;
return rb_entry(left, struct sched_entity, run_node);
```

🧠 Optimization:

- Leftmost node **cache** karke rakha jaata hai
- Tree traverse bhi nahi karna padta

------

### Agar tree empty?

- Koi runnable process nahi
- 👉 **idle task** chalega

------

## 8️⃣ Process tree me kaise add hota hai? (enqueue)

Jab:

- Process wake hota hai
- fork() hota hai
- migration hoti hai

👉 `enqueue_entity()`

### Steps (high level):

1. vruntime normalize karo
2. Current task ka runtime update
3. Stats update
4. **Red-black tree me insert**

------

### Actual insertion: `__enqueue_entity()`

Logic:

- Tree root se start
- Compare vruntime (key)
- Chhota → left
- Bada → right

Agar:

- Sirf left-left-left gaye
  👉 Ye **new leftmost node** hai

Then:

```c
cfs_rq->rb_leftmost = &se->run_node;
```

🧠 Analogy:

- School me line sorted by **least attendance**
- Naya student agar sabse kam attendance wala ho → line ke aage

------

## 9️⃣ Process tree se kaise nikalta hai? (dequeue)

Jab:

- Process sleep karta hai
- Process exit karta hai

👉 `dequeue_entity()`

Steps:

1. Current runtime update
2. Stats clean
3. Tree se remove

------

### Actual removal: `__dequeue_entity()`

```c
rb_erase(&se->run_node, &cfs_rq->tasks_timeline);
```

Agar:

- Ye process leftmost tha

👉 Next node ko leftmost bana do:

```c
cfs_rq->rb_leftmost = rb_next(&se->run_node);
```

🧠 Analogy:

- Queue ka pehla banda chala gaya
- Next banda automatically front pe

------

## 🔑 Key Takeaways (Interview + Kernel Dev)

- CFS **timeslice use nahi karta**
- Fairness = **vruntime**
- Scheduler decision = **smallest vruntime**
- Runnable tasks = **red-black tree**
- Leftmost node = **next task**
- Nice value = **weight**, not time

------

## 🧠 Mental Model (yaad rakhne ke liye)

> **CFS = CPU ka ledger**

- Jo jitna zyada kharch kar chuka
- Wo utna peeche
- Jo kam kharch kiya
- Wo aage

------

Agar next chaho to hum:

- **Sleeping & Waking logic (place_entity, wakeup preemption)**
- **sched_min_granularity & latency tuning**
- **Real-time scheduler internals**
- **Code walk-through with diagrams**

bolna 👍





Perfect, tumne **Linux Device Drivers / Scheduler chapter ka core part** utha liya hai 👍
Ab main is poore text ko **step-by-step Hinglish me**, **code ke peeche ka dimaag** samjhaata hoon — jaise kernel developer sochta hai.

------

# 🔁 Scheduler Entry Point — `schedule()`

## 🔑 Big Picture

`schedule()` **kernel ka main gate** hai scheduler ke liye.

👉 Jab bhi kernel ko lagta hai:

- current task ne CPU chhodna chahiye
- ya koi aur task zyada deserving hai

to kernel **schedule()** call karta hai.

📍 Location:

```
kernel/sched.c
```

------

## 🧠 schedule() actually kya karta hai?

- Ye **scheduler-agnostic** hai
- Matlab:
  - CFS ho
  - RT ho
  - Deadline ho
    → schedule() kisi ek ko favor nahi karta

👉 Ye sirf itna bolta hai:

> “Highest priority scheduler class batao, tum kaunsa task chalaoge?”

------

# 🎯 `pick_next_task()` — Real decision yahin hota hai

`schedule()` internally call karta hai:

```c
pick_next_task(rq);
```

Yahin decide hota hai:

> **“Next CPU pe kaun chalega?”**

------

## 🔥 Code Walkthrough (Intuition ke saath)

### Function signature

```c
static inline struct task_struct *
pick_next_task(struct rq *rq)
```

- `rq` = runqueue (per-CPU runnable tasks)

------

## 🚀 Optimization Hack (VERY IMPORTANT)

```c
if (likely(rq->nr_running == rq->cfs.nr_running)) {
    p = fair_sched_class.pick_next_task(rq);
    if (likely(p))
        return p;
}
```

### 🧠 Iska matlab kya?

- Agar:
  - total runnable tasks == CFS runnable tasks
    → matlab **sirf normal processes hi chal rahe hain**

📌 Real life:

- 99% time system me sirf normal apps hi hoti hain
- RT / deadline tasks rare hote hain

👉 To kernel bolta hai:

> “Loop chalane ki kya zarurat?
> Direct CFS se hi next task le lo.”

💥 Performance optimization (fast path)

------

## 🧩 Scheduler Classes Priority Order

Scheduler classes linked list ki tarah hoti hain:

```
stop_sched_class
↓
dl_sched_class   (deadline)
↓
rt_sched_class   (real time)
↓
fair_sched_class (CFS)
↓
idle_sched_class
```

------

## 🔄 Core Loop — Class by Class Check

```c
class = sched_class_highest;
for (;;) {
    p = class->pick_next_task(rq);
    if (p)
        return p;
    class = class->next;
}
```

### Kaam kya ho raha hai?

- Highest priority class se start

- Har class se poocha jaata hai:

  > “Tumhare paas koi runnable task hai?”

- Jo pehla **non-NULL task** de → wahi select

⚠️ Idle class hamesha kuch na kuch return karta hai
(isliye loop infinite hai but safe)

------

## 🧠 CFS ka role yahan

CFS ka `pick_next_task()`:

```c
fair_sched_class.pick_next_task()
```

👉 Ye internally call karta hai:

```
pick_next_entity()
    ↓
__pick_next_entity()
```

📌 Jo hum pehle discuss kar chuke:

- Red-Black tree
- Leftmost node
- Smallest vruntime

------

# 😴 Sleeping and Waking Up

## ❓ Task sleep kyun karta hai?

- File I/O wait
- Keyboard input
- Network packet
- Timer expire
- Lock / semaphore contention

👉 Sleep ka matlab:
❌ runnable nahi
❌ scheduler consider nahi karega

------

## 🛑 Sleeping State kyun zaruri hai?

Agar sleeping state na ho:

- Scheduler us task ko select karta rahega
- Task busy-loop karega
- CPU waste 💀

------

## 🧾 Sleep process ka exact flow

### Jab task sleep karta hai:

1. Task state set karta hai:
   - `TASK_INTERRUPTIBLE`
   - ya `TASK_UNINTERRUPTIBLE`
2. Wait queue me add hota hai
3. CFS red-black tree se remove hota hai
4. `schedule()` call karta hai

------

### Jab task wake hota hai:

1. State → `TASK_RUNNING`
2. Wait queue se remove
3. CFS rbtree me wapas add
4. Scheduler consider karega

------

## ⚖️ TASK_INTERRUPTIBLE vs TASK_UNINTERRUPTIBLE

| State           | Signal ka effect   |
| --------------- | ------------------ |
| INTERRUPTIBLE   | Signal aaya → wake |
| UNINTERRUPTIBLE | Signal ignore      |

📌 Disk I/O aksar **UNINTERRUPTIBLE** hota hai
(isliye load average badhta hai)

------

# 🧵 Wait Queues — Sleep ka backbone

Wait queue =
➡️ **event ka waiting room**

### Type:

```c
wait_queue_head_t
```

------

## 🧨 Simple sleep API kyun galat hai?

Old style:

```c
if (!condition)
    sleep();
```

❌ Race condition:

- Condition true ho gayi
- Task phir bhi so gaya
- Forever sleep 😱

------

# ✅ Correct Pattern (VERY IMPORTANT FOR DRIVER DEV)

```c
DEFINE_WAIT(wait);
add_wait_queue(q, &wait);

while (!condition) {
    prepare_to_wait(q, &wait, TASK_INTERRUPTIBLE);

    if (signal_pending(current))
        handle_signal();

    schedule();
}

finish_wait(q, &wait);
```

------

## 🔍 Step-by-step logic

1️⃣ `DEFINE_WAIT`
→ wait queue entry banaya

2️⃣ `add_wait_queue()`
→ event ke waiting list me add

3️⃣ `prepare_to_wait()`
→

- task state = sleeping
- race conditions handle

4️⃣ Signal aaya?
→ spurious wakeup ho sakta hai

5️⃣ Wake hone ke baad:
→ condition **phir se check**

6️⃣ Condition true?
→ `finish_wait()`
→ state = RUNNING
→ wait queue se remove

💡 **while loop mandatory hai**
(never trust single wakeup)

------

# 📌 inotify_read() example — Real kernel code

Ye function:

- userspace read() karta hai
- event na ho → sleep
- event aaya → wake + copy

Key difference:

- condition check loop ke andar
- kyunki lock lena padta hai

📌 Pattern SAME hai:

- prepare_to_wait
- check condition
- schedule
- finish_wait

------

# 🧠 Kernel Developer Mental Model

> **Sleep = remove from scheduler universe**
> **Wake = re-enter scheduling competition**

------

## 🎯 Interview-Ready Summary

- `schedule()` = generic entry point
- `pick_next_task()` = real decision maker
- Scheduler classes checked by priority
- CFS optimized fast path
- Sleeping tasks:
  - wait queue
  - not runnable
- Always sleep in **while loop**
- Spurious wakeups are NORMAL

------

Agar next chaho to hum:

- **wakeup preemption (place_entity)**
- **why waking task may preempt current**
- **TASK_UNINTERRUPTIBLE & load average**
- **Driver wait queue patterns**

bolna 🔥



Badhiya 👌
Ye **scheduler ka advanced + interview-favorite part** hai. Main isko **Hinglish me**, **flow + real life examples + kernel logic** ke saath explain kar raha hoon. Thoda lamba hoga, par dimag me permanently baith jaayega.

------

# 🔔 Waking Up (Sleep se wapas CPU tak)

## ❓ Jab event ho jaata hai, task kaise uthta hai?

👉 Kernel **wake_up()** call karta hai.

Example:

- Disk se data aa gaya
- Keyboard key press hua
- Network packet aa gaya

📌 Jo code event create karta hai (VFS, driver, interrupt handler), **wahi wake_up()** call karta hai.

------

## 🔄 wake_up() → try_to_wake_up()

### Actual kaam yahan hota hai 👇

`try_to_wake_up(task)` ye steps karta hai:

1️⃣ **task->state = TASK_RUNNING**
→ task ab runnable hai

2️⃣ **enqueue_task()**
→ CFS ke red-black tree me wapas add

3️⃣ **Priority check**

```c
if (new_task_priority > current_task_priority)
    need_resched = 1;
```

👉 Matlab:

> “Agar uthne wala task zyada important hai, to current task ko side karo”

------

### 🧠 Real Life Example

Tum office me kaam kar rahe ho
Boss ka urgent call aata hai

👉 Tum current kaam chhod ke boss ka call lete ho
= **need_resched set**

------

## ⚠️ Spurious Wakeups (FAKE ALARM)

Important line:

> Just because task wake hua ≠ event hua

### Example:

- Signal aa gaya
- Timeout
- Interrupt

👉 Isliye **sleep hamesha while loop me hota hai**

❌ Galat:

```c
if (!data)
    sleep();
```

✅ Sahi:

```c
while (!data)
    sleep();
```

------

## 🔁 Scheduler States Flow (Figure 4.1 simplified)

### States:

- `TASK_RUNNING` → runnable
- `TASK_INTERRUPTIBLE` → sleeping (signal se uth sakta)
- `TASK_UNINTERRUPTIBLE` → sleeping (signal ignore)

------

### Flow samjho:

1️⃣ Task running hai
2️⃣ `_add_wait_queue()`

- state = TASK_INTERRUPTIBLE
- `schedule()` call
- runqueue se remove

3️⃣ Event hua
4️⃣ `try_to_wake_up()`

- state = TASK_RUNNING
- runqueue me add
- `need_resched = 1`

5️⃣ Scheduler decide karega:

- abhi chale ya baad me

------

# 🔄 Context Switching (Actual switch)

Scheduler bolta hai:

> “Next task mil gaya, ab switch karo”

Ye kaam karta hai:

```c
context_switch()
```

------

## context_switch() ke 2 MAIN KAAM

### 1️⃣ switch_mm()

📍 `<asm/mmu_context.h>`

- Virtual memory change:
  - Old process ka page table ❌
  - New process ka page table ✅

🧠 Matlab:

> Har process ka apna illusion hota hai memory ka

------

### 2️⃣ switch_to()

📍 `<asm/system.h>`

Ye **CPU state switch** karta hai:

- Registers save/restore
- Stack pointer
- Architecture specific state

📌 Ye part **CPU dependent** hota hai

------

## 🎬 Movie Analogy

- switch_mm() = scene change
- switch_to() = actor change

------

# 🚨 need_resched — Scheduler ka Alarm Bell

Kernel har time `schedule()` call nahi karta
Isliye ek **flag** rakha hai:

```c
need_resched
```

👉 Matlab:

> “Jab safe ho, scheduler chala dena”

------

## need_resched kab set hota hai?

1️⃣ **scheduler_tick()**

- time slice khatam
- ya fairness disturb

2️⃣ **try_to_wake_up()**

- higher priority task utha

------

## need_resched kab check hota hai?

- User-space return pe
- Interrupt return pe
- Kernel preemption points pe

------

## ❓ need_resched global kyun nahi?

📌 Per-process hota hai

Reason:

- `current` pointer hamesha cache-hot hota
- Global variable slow hota

Evolution:

- Pre 2.2 → global
- 2.2/2.4 → task_struct int
- 2.6+ → thread_info bit

------

# 👤 User Preemption (Safe Zone)

User-space me:

- No locks
- No kernel data structures

👉 Isliye **sabse safe preemption**

------

## User preemption hota hai jab:

✔️ System call se return
✔️ Interrupt se return

Flow:

```
return_to_user()
    ↓
check need_resched
    ↓
schedule()
```

------

# 🧠 Kernel Preemption (Linux ka SUPERPOWER)

Linux = **Fully Preemptive Kernel** 🔥

Dusre Unix:
❌ kernel me enter → jab tak bahar nahi, koi preemption nahi

Linux:
✅ kernel ke andar bhi preempt ho sakta hai

------

## ❓ Kernel me kab safe hota hai?

Rule:

> **Agar lock nahi pakda → preempt safe**

------

## 🔢 preempt_count

Har task ke paas:

```c
preempt_count
```

- Lock acquire → ++
- Lock release → --

📌 `preempt_count == 0` → preemptible

------

## Kernel preemption flow

Interrupt return pe:

```c
if (need_resched && preempt_count == 0)
    schedule();
```

Agar lock held:
❌ preempt nahi hoga

Jaise hi lock chhuta:

- unlock code check karega
- agar need_resched set → schedule()

------

## Kernel preemption kab hota hai?

✔️ Interrupt se kernel-space return
✔️ Lock release ke baad
✔️ schedule() explicitly call
✔️ Kernel task block kare

------

# ⏱️ Real-Time Scheduling (RT)

Normal tasks:

```
SCHED_NORMAL → CFS
```

RT tasks:

```
SCHED_FIFO
SCHED_RR
```

📍 Implemented in:

```
kernel/sched_rt.c
```

------

## 🚦 SCHED_FIFO

- No timeslice ❌
- Jab tak:
  - block kare
  - yield kare

⚠️ Infinite run possible

Example:

- Industrial control
- Audio pipeline

------

## 🔄 SCHED_RR

- FIFO + timeslice ✅
- Same priority RT tasks round-robin
- Lower priority kabhi preempt nahi kar sakta

------

## ⚖️ RT Priority

- Range: **0 – 99**
- Static (no dynamic change)

Nice values:

- Mapped to **100 – 139**

------

# 🧰 Scheduler System Calls (User control)

Important calls:

- `nice()` → priority change
- `sched_setscheduler()` → policy
- `sched_setparam()` → RT priority
- `sched_setaffinity()` → CPU binding
- `sched_yield()` → voluntarily CPU chhodna

------

## 🎯 Processor Affinity

👉 “Ye task sirf in CPUs pe chalega”

Stored in:

```c
task_struct.cpus_allowed
```

Use case:

- Cache locality
- Real-time
- Performance tuning

------

# 🤲 Yielding CPU — sched_yield()

Matlab:

> “Mujhe baad me chalne do”

Effect:

- Running → expired queue
- Priority list ke end me

⚠️ Aajkal **sched_yield() expensive** hai
Soch samajh ke use karo

------

# 🧠 Final Mental Model (Golden)

- wake_up ≠ event confirmed
- Always sleep in while loop
- need_resched = polite interrupt
- Kernel preemption = lock based
- RT tasks = dangerous but powerful

------

Agar chaho next:

- **CFS wakeup preemption**
- **Latency vs throughput**
- **Load average & TASK_UNINTERRUPTIBLE**
- **Interview Q&A on scheduler**

Bas bolo 👍