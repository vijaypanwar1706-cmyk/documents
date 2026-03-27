आपके syllabus के **MODULE 3 – Kernel Driver Core Initialization** को अब हम **deep dive** करते हैं। यह kernel boot के दौरान बहुत critical concept है क्योंकि इसी से decide होता है कि **ALSA, I2C, I2S और WM8960 driver किस order में initialize होंगे**।

आपके uploaded syllabus में भी यही phase kernel boot के अंदर आता है ।

------

# MODULE 3️⃣ – Kernel Driver Core Initialization (Deep Dive)

Kernel boot होने के बाद drivers random तरीके से load नहीं होते।
Linux kernel में **initcall framework** होता है जो initialization को **levels में divide करता है**।

Simplified boot order:

```
start_kernel()

   ↓
setup_arch()

   ↓
rest_init()

   ↓
kernel_init()

   ↓
do_basic_setup()

   ↓
do_initcalls()
```

सबसे important function:

```
do_initcalls()
```

यही सारे **initcall levels को execute करता है।**

------

# 3.1 Initcall Levels (Actual Order)

Kernel में initialization इस order में होता है:

```
pure_initcall
core_initcall
postcore_initcall
arch_initcall
subsys_initcall
fs_initcall
device_initcall
late_initcall
```

लेकिन driver learning में हम अक्सर इन 4 को focus करते हैं:

```
pure_initcall
core_initcall
subsys_initcall
module_init
```

अब इनको deeply समझते हैं।

------

# 1️⃣ pure_initcall

यह **सबसे early initialization stage** है।

Purpose:

```
very low level kernel setup
```

Examples

- scheduler basics
- memory subsystems

Audio drivers यहाँ initialize नहीं होते।

------

# 2️⃣ core_initcall

यह stage **kernel core subsystems** के लिए है।

Examples

```
driver model
bus infrastructure
device framework
```

यहाँ register होते हैं:

```
driver core
device model
```

Important point:

👉 **Platform driver framework यहाँ ready होता है**

Example:

```
driver_init()
```

------

# 3️⃣ subsys_initcall

यह stage **major kernel subsystems के लिए है।**

Examples

- I2C
- SPI
- ALSA
- Networking
- USB

यहाँ register होते हैं:

```
I2C subsystem
ALSA core
ASoC framework
```

Example:

```
subsys_initcall(i2c_init);
subsys_initcall(snd_soc_init);
```

Meaning:

```
I2C bus
ALSA framework
```

इस stage पर ready हो जाते हैं।

------

# 4️⃣ module_init

यह driver level initialization है।

Example:

```
module_init(wm8960_modinit)
```

या

```
module_i2c_driver(wm8960_i2c_driver)
```

यह **device driver registration** के लिए होता है।

Examples:

```
WM8960 codec driver
I2S controller driver
Machine driver
```

------

# Important Kernel Macro Mapping

Kernel macros actually expand to initcalls.

Example:

```
module_init(x)
```

internally becomes

```
device_initcall(x)
```

Meaning:

```
Driver level initialization
```

------

# Real Example – WM8960 Driver

File:

```
sound/soc/codecs/wm8960.c
```

Driver registration:

```
module_i2c_driver(wm8960_i2c_driver);
```

Internally expands to:

```
module_init()
```

Meaning:

```
device_initcall level
```

So WM8960 driver register होता है **device_initcall stage पर**।

------

# Real Example – I2C Core

File:

```
drivers/i2c/i2c-core-base.c
subsys_initcall(i2c_init);
```

Meaning:

```
I2C subsystem initialize
```

------

# Real Example – ALSA Core

File:

```
sound/core/init.c
subsys_initcall(snd_init);
```

Meaning:

```
ALSA core initialize
```

------

# Real Example – ASoC Framework

File:

```
sound/soc/soc-core.c
subsys_initcall(snd_soc_init);
```

Meaning:

```
ASoC framework initialize
```

------

# Complete Boot Order for Your Audio Stack

Raspberry Pi + WM8960 case:

```
1️⃣ core_initcall
   driver core ready

2️⃣ subsys_initcall
   I2C subsystem init
   ALSA core init
   ASoC framework init

3️⃣ device_initcall
   bcm2835 I2C controller driver register
   bcm2835 I2S driver register
   wm8960 codec driver register
   simple-audio-card machine driver

4️⃣ driver matching
   DT compatible match
   probe() called
```

------

# Final Boot Timeline (Your Case)

```
Kernel Boot

↓

driver core init

↓

I2C subsystem init

↓

ALSA core init

↓

ASoC framework init

↓

I2C controller driver probe

↓

WM8960 codec probe

↓

simple-audio-card probe

↓

snd_soc_register_card()

↓

/proc/asound/cards created
```

------

# How to See This Practically (Very Important)

Raspberry Pi में check करें:

```
dmesg | grep initcall
```

या enable करें:

```
CONFIG_INITCALL_DEBUG
```

फिर boot log में दिखेगा:

```
initcall i2c_init+0x0/0x100 returned 0
initcall snd_soc_init+0x0/0x200 returned 0
initcall wm8960_modinit+0x0/0x80 returned 0
```

यह आपको exact order दिखा देगा।

------

# Kernel Source Files You Must Read (Module 3)

Very important:

```
init/main.c
```

Functions:

```
start_kernel()
do_initcalls()
do_one_initcall()
```

------

# Mental Model (Very Important)

यह sequence हमेशा याद रखें:

```
Boot
 ↓
Driver Framework
 ↓
Subsystems (I2C / ALSA)
 ↓
Drivers Register
 ↓
Device Tree Match
 ↓
Probe()
 ↓
Sound Card Register
```

------

✅ अगर आप चाहें तो मैं अगला step और deep बना सकता हूँ:

**MODULE 3.2**

```
start_kernel() → do_initcalls() → driver registration
```

और मैं आपको **actual kernel call stack diagram (boot → ALSA card creation)** भी बना सकता हूँ — जो kernel driver developers use करते हैं।