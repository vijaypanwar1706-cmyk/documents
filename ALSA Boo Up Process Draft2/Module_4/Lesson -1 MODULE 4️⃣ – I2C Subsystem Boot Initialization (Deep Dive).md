Ab tak aap **Module 1, 2, 3** cover kar chuke ho:

1️⃣ Boot flow
2️⃣ Device Tree
3️⃣ Kernel initcalls

Ab boot process me next important cheez aati hai **I2C subsystem initialization**, kyunki **WM8960 codec I2C bus par connect hai**.

Isliye **Module 4 = I2C Subsystem Boot Initialization**.
Main ise **Raspberry Pi 4B + Kernel rpi-6.12.y** ke context me deep dive karta hoon (Hinglish me).

Aapke syllabus file me bhi ye step boot order me next hai.

------

# MODULE 4️⃣ – I2C Subsystem Boot Initialization (Deep Dive)

Goal:

Boot ke dauraan ye samajhna:

```
Kernel boot
   ↓
I2C core init
   ↓
BCM2835 I2C controller driver load
   ↓
I2C adapter register
   ↓
Device Tree ke devices detect
   ↓
WM8960 codec probe
```

------

# 4.1 I2C Subsystem Overview

Linux me **I2C subsystem 3 main parts me hota hai**

```
I2C Core
I2C Bus Driver (controller)
I2C Client Driver (device driver)
```

Example Pi4 case:

```
I2C Core        → drivers/i2c/i2c-core-base.c

I2C Controller  → drivers/i2c/busses/i2c-bcm2835.c

I2C Client      → sound/soc/codecs/wm8960.c
```

Architecture:

```
CPU
 ↓
I2C Controller (bcm2835)
 ↓
I2C Bus
 ↓
WM8960 Codec
```

------

# 4.2 Boot Time I2C Initialization

Kernel boot me ek stage par **I2C core initialize hota hai**.

File:

```
drivers/i2c/i2c-core-base.c
```

Important function:

```
i2c_init()
```

Registration:

```
subsys_initcall(i2c_init);
```

Matlab:

```
Boot phase → subsys_initcall
```

Execution flow:

```
start_kernel()
   ↓
do_basic_setup()
   ↓
do_initcalls()
   ↓
subsys_initcall
   ↓
i2c_init()
```

------

# 4.3 i2c_init() kya karta hai

Simplified view:

```
static int __init i2c_init(void)
{
    bus_register(&i2c_bus_type);
    i2c_adapter_class = class_create(...);
}
```

Ye do major kaam karta hai:

### 1️⃣ I2C bus type register

```
bus_register(&i2c_bus_type)
```

Ye **Linux device model** ko batata hai ki ek new bus type exist karta hai:

```
/sys/bus/i2c
```

------

### 2️⃣ I2C class create

```
class_create()
```

Ye create karta hai:

```
/sys/class/i2c-adapter
```

------

# 4.4 Raspberry Pi I2C Controller Driver

Ab next step:

**Hardware I2C controller driver load hota hai**

File:

```
drivers/i2c/busses/i2c-bcm2835.c
```

Yeh driver handle karta hai:

```
Broadcom BCM2711 I2C controller
```

Driver structure:

```
static struct platform_driver bcm2835_i2c_driver
```

Registration:

```
module_platform_driver(bcm2835_i2c_driver);
```

Iska matlab:

```
module_init(bcm2835_i2c_driver_init)
```

Boot me ye call hoga.

------

# 4.5 Platform Driver Matching

Driver match kaise hota hai?

Device Tree se.

DT me node hota hai:

```
&i2c1 {
    compatible = "brcm,bcm2835-i2c";
}
```

Driver me:

```
static const struct of_device_id bcm2835_i2c_of_match[]
```

Matching process:

```
DT compatible
     ↓
platform driver match
     ↓
probe() call
```

------

# 4.6 bcm2835_i2c_probe()

Ye sabse important function hai.

```
static int bcm2835_i2c_probe(struct platform_device *pdev)
```

Boot me jab driver match hota hai to kernel ye function call karta hai.

Inside probe:

### Step 1: Memory map

```
devm_ioremap_resource()
```

Controller registers map ho jaate hain.

Example registers:

```
C register
S register
DLEN register
```

------

### Step 2: Clock enable

```
clk_prepare_enable()
```

I2C controller clock start hota hai.

------

### Step 3: Adapter create

```
struct i2c_adapter adap;
```

Ye represent karta hai:

```
I2C bus
```

Example:

```
i2c-1
```

------

### Step 4: Adapter register

```
i2c_add_adapter()
```

Iske baad system me bus visible ho jati hai.

Check:

```
/sys/class/i2c-adapter/
```

Output example:

```
i2c-0
i2c-1
```

------

# 4.7 Device Tree se I2C Devices Detect

Ab kernel check karta hai:

```
Is bus par kaunse devices defined hain?
```

Device Tree me:

```
&i2c1 {
    wm8960: codec@1a {
        compatible = "wlf,wm8960";
        reg = <0x1a>;
    };
};
```

Meaning:

```
I2C address = 0x1A
device = WM8960
```

Kernel create karta hai:

```
i2c_client
```

Structure:

```
struct i2c_client
```

Example:

```
name = wm8960
addr = 0x1a
```

------

# 4.8 I2C Driver Match (WM8960)

Ab kernel search karta hai:

```
Kaunsa driver compatible = "wlf,wm8960" support karta hai?
```

Driver:

```
sound/soc/codecs/wm8960.c
```

Driver structure:

```
static struct i2c_driver wm8960_i2c_driver
```

Matching:

```
.of_match_table
```

------

# 4.9 WM8960 Probe Trigger

Final flow:

```
i2c bus ready
    ↓
device tree device detected
    ↓
wm8960 driver match
    ↓
wm8960_i2c_probe()
```

Ye step **Module 8 me deep study karenge**.

------

# 4.10 Complete Boot Flow (Module 4 Perspective)

Real boot timeline:

```
start_kernel
   ↓
do_basic_setup
   ↓
subsys_initcall
   ↓
i2c_init()

   ↓
bcm2835_i2c_driver_init
   ↓
platform_driver_register

   ↓
Device Tree match
   ↓
bcm2835_i2c_probe()

   ↓
i2c_add_adapter()

   ↓
I2C bus ready

   ↓
WM8960 device detected
```

------

# 4.11 Raspberry Pi me Practical Verification

Aap apne Pi4 me check kar sakte ho:

### I2C buses

```
ls /sys/class/i2c-adapter/
```

Example:

```
i2c-0
i2c-1
```

------

### Detect codec

```
sudo i2cdetect -y 1
```

Output me dikhega:

```
1a
```

Matlab:

```
WM8960 detected
```

------

### Boot logs

```
dmesg | grep i2c
```

Example:

```
bcm2835_i2c fe804000.i2c: BSC controller registered
```

------

# 4.12 Kernel Source Reading Roadmap

Agar aap real deep study karna chahte ho to is order me code padho:

### Step 1

```
drivers/i2c/i2c-core-base.c
```

Function:

```
i2c_init()
```

------

### Step 2

```
drivers/i2c/busses/i2c-bcm2835.c
```

Function:

```
bcm2835_i2c_probe()
```

------

### Step 3

```
drivers/i2c/i2c-core-of.c
```

Function:

```
of_i2c_register_devices()
```

Ye Device Tree devices detect karta hai.

------

# 🔥 Next Module (Very Important)

Next module hoga:

# MODULE 5️⃣ – I2S Controller Initialization

File:

```
sound/soc/bcm/bcm2835-i2s.c
```

Ye initialize karega:

```
CPU DAI (Digital Audio Interface)
```

Without this:

```
WM8960 audio work nahi karega
```

------

Agar chaho to main next message me **Module 4 ka complete kernel call graph** bhi bana sakta hoon jisme:

```
i2c_init()
 → platform_driver_register()
 → bcm2835_i2c_probe()
 → i2c_add_adapter()
 → of_i2c_register_devices()
```

Aur saath me **exact boot sequence diagram** bhi bana dunga jo ALSA audio driver study ke liye bahut powerful mental model deta hai.