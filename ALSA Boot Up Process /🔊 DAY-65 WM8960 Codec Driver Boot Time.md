Excellent 👍
अब हम **real hardware-level audio** में enter कर रहे हैं —
**PHASE-5 (WM8960 Codec Driver Boot Time)** वो phase है जहाँ **chip सच में “जागती” है**।

मैं इसे **bilkul same style** में समझा रहा हूँ:
👉 *zero-level intuition → kernel probe flow → register-level → advanced debug*
(Raspberry Pi 4 + I2C + WM8960 focus)

------

# 🔊 PHASE 5: WM8960 Codec Driver Boot Time

**(Basic → Full Advanced, Boot-Time Deep Dive)**

------

## 0️⃣ Big Picture (30-second intuition)

Boot ke pehle phases me:

- ALSA ready (PHASE-3)
- ASoC framework ready (PHASE-4)
- CPU DAI + Machine driver waiting

अब:

> **WM8960 codec driver probe hota hai**
> → chip detect hoti hai
> → registers init hote hain
> → codec ASoC ke andar “usable component” ban jaata hai

❗ *Abhi bhi audio play nahi hota*
(PCM open hone par hi clocks & paths enable honge)

------

## 1️⃣ WM8960 driver kernel me kaha hota hai?

📍 Path:

```
sound/soc/codecs/wm8960.c
```

📌 Driver type:

- **I2C based ASoC codec driver**
- Uses **regmap** abstraction

------

## 2️⃣ Codec probe kab aur kyu hota hai?

### Trigger:

- Device Tree me entry milti hai:

```dts
wm8960: codec@1a {
    compatible = "wlf,wm8960";
    reg = <0x1a>;
};
```

### Boot flow:

```
I2C core init
   |
   |--> I2C bus scan
           |
           |--> wm8960 matched
                   |
                   |--> wm8960_probe()
```

👉 **PHASE-5 starts here**

------

## 3️⃣ `wm8960_probe()` — Codec ka entry point

### Function signature:

```c
static int wm8960_probe(struct i2c_client *i2c)
```

### Is function ka kaam:

> “Main verify karunga ki WM8960 chip sach me present hai,
> phir usko ALSA-ASoC world me register karunga”

------

## 4️⃣ Step-by-step inside `wm8960_probe()` (VERY IMPORTANT)

------

### 🥇 Step 1: Memory + private data

```c
struct wm8960_priv *wm8960;
wm8960 = devm_kzalloc(...);
```

📌 Ye structure store karta hai:

- regmap handle
- clock state
- sysclk
- bias level

------

### 🥈 Step 2: Regmap initialization (MOST IMPORTANT)

#### ❓ Regmap kya hota hai?

> Regmap = **register access abstraction**

WM8960 ke registers:

- 9-bit wide
- I2C over slow bus

#### Code:

```c
wm8960->regmap = devm_regmap_init_i2c(i2c, &wm8960_regmap);
```

#### Regmap config defines:

- register width
- readable/writable registers
- cache strategy

📌 Benefit:

- Cache
- Debug
- Power-safe register access

------

### 🥉 Step 3: I2C communication test

Driver internally:

- Writes to reset register
- Reads back values

Agar fail:

```
-ENODEV
```

👉 Codec **probe abort**

------

## 5️⃣ Codec Reset Sequence (Hardware awakening)

### 🔄 Reset register

WM8960 register:

```
R15 = RESET
```

Code:

```c
regmap_write(regmap, WM8960_RESET, 0);
```

### Hardware effect:

- All registers → default state
- Power blocks off
- Audio paths disabled

📌 **Ye step mandatory hai**
warna chip unpredictable hogi

------

## 6️⃣ Default Register Programming (Silent but CRITICAL)

Reset ke baad driver kuch **safe defaults** program karta hai:

### Examples:

- Bias control
- VMID enable
- Clocking defaults
- ADC/DAC power down (initially)

📌 Ye sab:

- **Sound produce nahi karta**
- Sirf stable base create karta hai

------

## 7️⃣ `snd_soc_component_register()` — Codec enters ASoC

### Code:

```c
snd_soc_component_register(&i2c->dev,
                           &soc_component_dev_wm8960,
                           &wm8960_dai, 1);
```

### Ye function kya karta hai?

1️⃣ Codec ko **ASoC component** banata hai
2️⃣ DAI expose karta hai (`wm8960-hifi`)
3️⃣ Mixer controls register karta hai
4️⃣ DAPM widgets register karta hai

👉 Ab codec **bind hone ke liye ready** hai

------

## 8️⃣ DAPM graph creation (Advanced but IMPORTANT)

WM8960 driver define karta hai:

- Inputs
- Outputs
- Mixers
- Power widgets

📍 File me:

```c
static const struct snd_soc_dapm_widget wm8960_dapm_widgets[];
```

DAPM ensures:

- Speaker tabhi on jab audio route active ho
- Power saving

------

## 9️⃣ Boot ke baad WM8960 state kya hoti hai?

| Item                 | State   |
| -------------------- | ------- |
| Codec detected       | ✅       |
| I2C working          | ✅       |
| Registers programmed | ✅       |
| DAC/ADC              | ❌ (off) |
| Clocks               | ❌ (off) |
| Audio output         | ❌       |

👉 Ye design hai, bug nahi

------

## 🔟 dmesg me kya dikhna chahiye?

```bash
dmesg | grep -i wm8960
```

Typical:

```
wm8960 1-001a: WM8960 audio codec
```

Agar ye line nahi:

- I2C issue
- DT compatible mismatch
- Power issue

------

## 1️⃣1️⃣ Real-world failure cases (Advanced Debug)

### ❌ I2C detect fail

- Wrong address
- Pull-ups missing

### ❌ Regmap init fail

- Wrong reg width
- Bus unstable

### ❌ No sound but probe ok

- DAPM path closed
- Machine driver routing missing

------

## 1️⃣2️⃣ Mental model (VERY IMPORTANT)

```
I2C detect
   |
wm8960_probe()
   |
regmap init
   |
reset
   |
default registers
   |
snd_soc_component_register()
   |
Codec READY (but silent)
```

------

## 🏁 Final Outcome (PHASE-5 Mastery)

Ab aap confidently explain kar sakte ho:

✔️ WM8960 **kab aur kaise detect hota hai**
✔️ `wm8960_probe()` ka har logical step
✔️ Reset + default programming ka role
✔️ Codec boot ke baad **silent kyun hota hai**
✔️ Real hardware debug approach

------

### 🔜 Next phase

👉 **PHASE-6: I2S + Clock Setup During Boot**
(yahin se “sound card hai par awaaz nahi” ka reason clear hota hai)

Bolo: **“PHASE-6 start karo”** 🔥