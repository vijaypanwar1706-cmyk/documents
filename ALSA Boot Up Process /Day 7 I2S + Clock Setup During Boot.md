Excellent 🔥
अब हम **सबसे ज़्यादा confusing लेकिन decisive phase** पर हैं:

> **“Sound card दिख रही है, clocks भी दिख रहे हैं… फिर भी आवाज़ क्यों नहीं?”**

यही सवाल **PHASE-6 (I2S + Clock Setup)** clear करता है।

मैं इसे भी **basic → kernel-internal → hardware timing → advanced debug** तक ले जा रहा हूँ
(Raspberry Pi 4 + BCM2835 I2S + WM8960 context)

------

# 🔊 PHASE 6: I2S + Clock Setup During Boot

**(Very Basics → Full Advanced)**

------

## 0️⃣ One-line intuition (सबसे पहले समझो)

> **Boot के time clocks “exist” करते हैं
> लेकिन audio start होने पर ही “flow” करते हैं**

Clock visible ≠ Clock active

------

## 1️⃣ BCM2835 I2S init — kab hota hai?

📍 Driver:

```
sound/soc/bcm/bcm2835-i2s.c
```

### Boot order recap:

```
ALSA core (Phase 3)
   ↓
ASoC core (Phase 4)
   ↓
BCM2835 I2S probe  ← (Phase 6 starts here)
   ↓
WM8960 codec
   ↓
Machine driver
```

------

## 2️⃣ I2S probe ke time kya hota hai? (VERY IMPORTANT)

### In `bcm2835_i2s_probe()`:

✔️ Registers map hote hain
✔️ DMA channels request hote hain
✔️ Clock handles get hote hain

❌ **Clock enable nahi hota**

------

### Code-level idea:

```c
i2s->clk = devm_clk_get(dev, NULL);
```

📌 Clock **reference** milta hai
लेकिन:

```c
clk_prepare_enable() ❌
```

abhi call nahi hota

------

## 3️⃣ I2S clocks kaun-kaun se hote hain?

### 🎵 Audio clocks breakdown

| Clock     | Role                           |
| --------- | ------------------------------ |
| **MCLK**  | Master clock (codec reference) |
| **BCLK**  | Bit clock (data bits)          |
| **LRCLK** | Left/Right channel sync        |

------

### Hardware relationship:

```
MCLK
  ↓
BCLK = sample_rate × bits × channels
  ↓
LRCLK = sample_rate
```

------

## 4️⃣ Boot ke time clocks “dikhte” kyun hain?

### Check:

```bash
ls /sys/kernel/debug/clk/
```

Aapko milta hai:

```
i2s_clk
pcm_clk
```

👉 Iska matlab:

- Clock tree **registered** hai
- Clock provider ready hai

❌ Matlab:

- Clock run kar raha hai → ❌

------

## 5️⃣ Clock dependency order (CRITICAL RULE)

### Golden rule:

> **Codec clock tab enable hota hai
> jab CPU I2S clock ready ho**

------

### Order:

```
CPU I2S clock
   ↓
MCLK output
   ↓
Codec sysclk
   ↓
DAC/ADC enable
```

Agar:

- Codec pehle clock maange
- CPU abhi idle ho

👉 Codec **silent** rahega

------

## 6️⃣ Actual clock enable kab hota hai? (MOST IMPORTANT)

### ❗ Clock enable hota hai only when:

```text
PCM stream STARTS
```

### Flow:

```
aplay / tinyplay
   ↓
snd_pcm_open()
   ↓
hw_params()
   ↓
snd_soc_dai_set_sysclk()
   ↓
clk_prepare_enable()
```

📌 **Boot time pe nahi**

------

## 7️⃣ `snd_soc_dai_set_sysclk()` ka role

Machine driver me ya codec driver me:

```c
snd_soc_dai_set_sysclk(codec_dai, WM8960_SYSCLK_MCLK, freq, SND_SOC_CLOCK_IN);
```

👉 Ye sirf **relationship define** karta hai
❌ Clock start nahi karta

------

## 8️⃣ DAPM + clocks (Advanced but essential)

DAPM bolta hai:

> “Audio path active nahi?
> Power off everything”

So:

- Speaker off
- DAC off
- Codec clock gated

------

## 9️⃣ Boot ke baad WM8960 + I2S state table

| Component     | State       |
| ------------- | ----------- |
| I2S registers | Initialized |
| Clock tree    | Present     |
| Clock running | ❌           |
| Codec power   | Standby     |
| Audio output  | ❌           |

👉 **Perfectly normal state**

------

## 🔟 Why “clocks visible but no sound”? (EXAM-LEVEL ANSWER)

> Boot ke time:
>
> - Clock providers register ho jaate hain
> - Clock consumers bind ho jaate hain
> - But ALSA PCM start nahi hua hota
> - Isliye clocks gated rehte hain

------

## 1️⃣1️⃣ Real hardware debug (VERY PRACTICAL)

### Check runtime clock:

```bash
cat /sys/kernel/debug/clk/clk_summary | grep i2s
```

Playback ke time:

```
enable_cnt > 0
```

Idle me:

```
enable_cnt = 0
```

------

## 1️⃣2️⃣ Typical beginner confusion (IMPORTANT)

❌ “Boot pe clock chalu hona chahiye”
✅ **Audio-driven clocking model**

Linux me:

> **No audio → no clock**

------

## 1️⃣3️⃣ Mental model (lock this in brain 🧠)

```
Boot:
  I2S + Codec = READY BUT IDLE

Playback:
  PCM open
     ↓
  Clocks ON
     ↓
  Audio flows 🎶
```

------

## 🏁 Final Outcome (PHASE-6 Mastery)

Ab aap confidently explain kar sakte ho:

✔️ I2S probe pe clocks kyun idle hote hain
✔️ Clock dependency order
✔️ MCLK/BCLK/LRCLK ka real enable point
✔️ “Sound card hai but awaaz nahi” ka exact reason
✔️ Kernel-level debug approach

------

### 🔜 Next phase

👉 **PHASE-7: Sound Card Registration & /dev/snd/**
(Ab ALSA user-visible world start hota hai)

Agar ready ho, bolo:
**“PHASE-7 start karo”** 🔥