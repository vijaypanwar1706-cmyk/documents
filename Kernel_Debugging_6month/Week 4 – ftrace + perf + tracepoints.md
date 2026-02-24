`function_graph tracer` Linux kernel ka ek powerful **ftrace tracer** hai jo kisi function ke **entry aur exit dono** ko trace karta hai, aur saath hi function ke andar ka pura call flow (nested calls) tree format me dikhata hai.

Tum kernel deep dive kar rahe ho (NULL deref, oops, page fault etc.), isliye ye tracer debugging ke liye bahut powerful tool hai.

Main basic se advance tak step-by-step samjhaata hoon 👇

------

# 1️⃣ ftrace kya hai?

**ftrace (Function Tracer)** Linux kernel ka built-in tracing framework hai.

Location:

```
/sys/kernel/debug/tracing/
```

Ya new systems me:

```
/sys/kernel/tracing/
```

Iske andar alag-alag tracers available hote hain:

```
cat available_tracers
```

Typical output:

```
function
function_graph
nop
sched_switch
irqsoff
preemptoff
...
```

------

# 2️⃣ function vs function_graph

## 🔹 function tracer

Sirf function entry log karta hai:

Example:

```
vfs_read()
  ext4_file_read_iter()
    generic_file_read_iter()
```

Ye sirf entry dikhata hai, exit nahi.

------

## 🔹 function_graph tracer

Ye:

- Function entry
- Function exit
- Execution time
- Nested call structure
- Indentation tree format

Sab show karta hai.

Example output:

```
  0)               |  vfs_read() {
  0)   1.234 us    |    ext4_file_read_iter() {
  0)   2.456 us    |      generic_file_read_iter();
  0)   3.789 us    |    }
  0)   5.123 us    |  }
```

Isme:

- `{` → entry
- `}` → exit
- Time → function execution time

------

# 3️⃣ Ye kaam kaise karta hai internally?

Ye sab hota hai **mcount / fentry instrumentation** se.

## Step 1: Compilation time instrumentation

Kernel jab compile hota hai:

```
CONFIG_FUNCTION_GRAPH_TRACER=y
CONFIG_FUNCTION_TRACER=y
```

Compiler har function ke start me ek call insert karta hai:

Old style:

```
mcount()
```

New style (ARM64, x86 modern):

```
__fentry__()
```

------

## Step 2: Runtime patching (dynamic ftrace)

Kernel boot hone ke baad:

- Ye calls NOP me convert ho jaate hain
- Jab tracing enable karte ho:
  - NOP ko real call me patch kar diya jata hai

Isko bolte hain:

```
Dynamic ftrace
```

------

## Step 3: function_graph ka extra mechanism

Normal function tracer:

- Entry pe hook karta hai

Function_graph tracer:

- Entry pe hook
- Return address modify karta hai

Ye sabse important concept hai ⚠️

------

# 4️⃣ Return address modification (Core Concept)

Har function call me stack par return address store hota hai:

Example:

```
caller()
{
   callee();
}
```

Stack me:

```
| return address |
```

Function_graph tracer kya karta hai?

1. Entry pe:
   - Original return address save karta hai
   - Return address ko replace karta hai apne handler se
2. Jab function return hota hai:
   - Control tracer ke handler me jaata hai
   - Execution time calculate hota hai
   - Original return address restore hota hai

Isliye function_graph:

- Exit timing measure kar pata hai
- Pure call tree bana pata hai

Ye technique low-level stack manipulation use karti hai.

------

# 5️⃣ Basic usage (Raspberry Pi example)

Mount debugfs:

```
sudo mount -t debugfs none /sys/kernel/debug
cd /sys/kernel/debug/tracing
```

Enable tracer:

```
echo function_graph > current_tracer
```

Check:

```
cat current_tracer
```

------

# 6️⃣ Trace start/stop

Start:

```
echo 1 > tracing_on
```

Stop:

```
echo 0 > tracing_on
```

Read trace:

```
cat trace
```

Clear:

```
echo > trace
```

------

# 7️⃣ Important controls (Advanced tuning)

## 🔹 1. Filter specific function

```
echo vfs_read > set_graph_function
```

Sirf vfs_read aur uske children trace honge.

------

## 🔹 2. Exclude functions

```
echo schedule > set_graph_notrace
```

------

## 🔹 3. Limit depth

```
echo 3 > max_graph_depth
```

Call tree depth limit.

------

## 🔹 4. Show function duration threshold

```
echo 10000 > graph_duration
```

Sirf 10us se zyada wale functions show honge.

------

## 🔹 5. Trace specific PID

```
echo <pid> > set_ftrace_pid
```

------

# 8️⃣ Output Format samjho

Example:

```
  1)               |  do_sys_open() {
  1)   5.678 us    |    getname_flags();
  1)   2.345 us    |    path_openat();
  1) +12.345 us    |  }
```

Explanation:

| Symbol | Meaning        |
| ------ | -------------- |
| `{`    | Function entry |
| `}`    | Exit           |
| us     | Microseconds   |
| +      | Over threshold |
| 1)     | CPU number     |

------

# 9️⃣ SMP System me behavior

Har CPU ka apna:

```
per_cpu ring buffer
```

Isliye output me:

```
0)
1)
2)
```

CPU number dikhega.

------

# 🔟 Internals Deep Dive

Key files:

```
kernel/trace/trace_functions_graph.c
kernel/trace/ftrace.c
```

Important structures:

```
struct ftrace_graph_ret
struct ftrace_graph_ent
```

Important functions:

- ftrace_graph_entry()
- ftrace_graph_return()

------

# 1️⃣1️⃣ Performance Impact

function_graph:

- Normal function tracer se heavy hota hai
- Stack manipulation karta hai
- Deep call chains me slow ho sakta hai

Production me continuously use nahi karte.

------

# 1️⃣2️⃣ Real Debug Use Cases

## 🔹 1. Slow syscall debugging

Example:

- read slow hai
- kaunsa internal function slow hai?

## 🔹 2. Driver debugging

Tum audio driver ya kernel driver padh rahe ho:

- probe() ka complete call tree dekh sakte ho
- interrupt handler ka flow dekh sakte ho

## 🔹 3. Boot time analysis

Boot slow hai?

- initcalls trace karo

------

# 1️⃣3️⃣ function_graph vs perf vs eBPF

| Tool           | Use case                     |
| -------------- | ---------------------------- |
| function_graph | Deep function call tracing   |
| perf           | Sampling profiler            |
| eBPF           | Dynamic programmable tracing |

function_graph deterministic hota hai (har call capture karta hai)

------

# 1️⃣4️⃣ ARM64 Specific Notes (Raspberry Pi 4 / Cortex-A72)

ARM64 me:

- fentry patching hota hai
- Return address x30 (LR register) se manage hota hai
- Stack frame based tracking hoti hai

Agar:

```
CONFIG_FRAME_POINTER=y
```

enabled ho to tracing aur reliable hoti hai.

------

# 1️⃣5️⃣ Common Problems

### ❌ Empty trace

Check:

```
CONFIG_FUNCTION_GRAPH_TRACER
```

### ❌ Kernel panic

Agar:

- Stack corruption
- Custom module ne LR modify kiya

------

# 1️⃣6️⃣ Ultra Advanced Concepts

## 🔹 Recursion protection

Tracer khud ko trace nahi karta.
Nahi to infinite recursion ho jaayegi.

Use:

```
FTRACE_NOTRACE
```

------

## 🔹 IRQ / NMI context

Function_graph limited context me hi safe hota hai.

------

## 🔹 Return stack overflow

Deep recursion me:

```
ftrace_graph_ret_stack
```

Overflow ho sakta hai.

------

# 1️⃣7️⃣ Practical Lab (Tumhare liye)

Raspberry Pi me try karo:

1. ssh enable karo
2. mount debugfs
3. echo function_graph
4. sirf ek syscall trace karo:

```
echo do_sys_open > set_graph_function
```

1. kisi file ko open karo:

```
cat /etc/passwd
```

1. trace read karo

Tumhe pura VFS call chain dikhega.

------

# 🎯 Final Summary

function_graph tracer:

- Kernel level deterministic function call tracer
- Entry + exit + timing show karta hai
- Return address patching use karta hai
- Deep debugging ke liye powerful tool hai
- Production ke liye heavy ho sakta hai

------

Agar tum chaho to next step me main:

- 📌 Iska internal call flow diagram bana sakta hoon
- 📌 Stack level memory layout samjha sakta hoon
- 📌 Ya ek driver debugging real example kara sakta hoon
- 📌 Ya function_graph + KASAN + page fault combined debugging sikha sakta hoon

Batao next kis direction me deep dive karein?











snd_pcm_open par trace kaise lagaaye ? 



Perfect 👍 tumne sahi observation kiya.

Error kyun aaya:

```bash
echo snd_pcm_open > set_graph_function
write error: Invalid argument
```

👉 Kyunki `snd_pcm_open` exact symbol name nahi hai kernel me.

Tumhare `available_filter_functions` me ye dikh raha hai:

```
snd_pcm_open_substream [snd_pcm]
snd_pcm_open.part.0 [snd_pcm]
```

Iska matlab samjho 👇

------

# 🔎 1️⃣ `snd_pcm_open.part.0` kya hai?

Ye GCC ka optimized symbol hai.

Modern GCC jab function ko:

- partially inline karta hai
- ya function splitting karta hai
- ya optimization me break karta hai

To symbol ban jata hai:

```
function_name.part.N
```

Isliye actual traceable symbol:

```
snd_pcm_open.part.0
```

Not:

```
snd_pcm_open
```

------

# ✅ 2️⃣ Tumne sahi set kiya

Ye command bilkul correct hai:

```bash
echo snd_pcm_open.part.0 > set_graph_function
```

Aur confirm bhi ho gaya:

```bash
cat set_graph_function
snd_pcm_open.part.0 [snd_pcm]
```

✔️ Ab trace properly lagega.

------

# 🎯 3️⃣ Ab kya karna hai

Ab sirf trigger karo:

```bash
echo > trace
echo 1 > tracing_on

aplay test.wav

echo 0 > tracing_on
cat trace
```

Ya agar sirf open test karna hai:

```bash
aplay -l
```

------

# 🧠 4️⃣ Better Debug Setup (Recommended)

Since PCM open me usually ye chain hoti hai:

```
snd_pcm_open.part.0
  snd_pcm_open_substream
  snd_pcm_attach_substream
  driver_open_callback
```

To better trace:

```bash
echo snd_pcm_open.part.0 > set_graph_function
echo snd_pcm_open_substream >> set_graph_function
```

------

# 🔬 5️⃣ Agar output empty aaye to

Possible reasons:

### 🔹 1. Wrong process trace ho raha hai

Specific PID set karo:

```bash
pidof aplay
echo <PID> > set_ftrace_pid
```

------

### 🔹 2. ALSA open ho hi nahi raha

Confirm karo:

```bash
ls -l /dev/snd/
```

------

### 🔹 3. Module name check karo

Tumhare output me:

```
[snd_pcm]
```

Matlab function module ke andar hai.

Check:

```bash
lsmod | grep snd_pcm
```

Agar module loaded nahi hai → trace nahi milega.

------

# 🔥 6️⃣ Pro Level: Module-specific filtering

Since ye module function hai:

Better way:

```bash
echo 'snd_pcm_open.part.0 [snd_pcm]' > set_graph_function
```

Bracket ke saath exact name bhi use kar sakte ho.

------

# 🧬 7️⃣ Deep Insight (Important)

Tumne jo dekha:

```
snd_pcm_open.part.0
```

Ye indicate karta hai:

Kernel compiled with optimization (likely -O2)

Agar tum debugging kernel banaoge:

```
CONFIG_CC_OPTIMIZE_FOR_DEBUGGING=y
```

To function splitting kam hoga
Aur plain `snd_pcm_open` symbol milega.

------

# 🎧 8️⃣ Audio Driver Deep Debug Path

Agar Raspberry Pi pe ho (likely bcm2835):

Actual hardware open function ho sakta hai:

```
bcm2835_pcm_open
```

Check karo:

```bash
cat available_filter_functions | grep pcm_open
```

Better hardware debug ke liye driver-level open trace karo.

------

# 🧨 9️⃣ Ab ek powerful test karo

Sab pcm open related functions trace karo:

```bash
echo > set_graph_function
cat available_filter_functions | grep pcm_open | cut -d' ' -f1 > set_graph_function
```

Phir:

```bash
echo 1 > tracing_on
aplay test.wav
echo 0 > tracing_on
cat trace
```

Ab tumhe pura open chain dikhega 🔥

------

# 🎯 Summary

✔️ Error aaya kyunki exact symbol `snd_pcm_open` nahi hai
✔️ Correct symbol: `snd_pcm_open.part.0`
✔️ Module functions ke liye exact name use karna padta hai
✔️ Optimization ke wajah se `.part.N` suffix aata hai

------

Agar tum chaho to next step me:

- 🎧 ALSA PCM open ka full internal flow diagram banaata hoon
- 🔬 Driver callback tak ka call stack explain karta hoon
- 🧠 Ya interrupt + DMA start tak ka tracing setup karaata hoon
- 🧨 Ya tumhare Raspberry Pi ke specific sound driver ka deep reverse path nikaalte hain

Ab tum kis level pe jaana chahte ho?