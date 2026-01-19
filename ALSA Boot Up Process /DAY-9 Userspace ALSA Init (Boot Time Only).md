Perfect 👍
अब हम **kernel se bahar nikal kar userspace** me ja rahe hain —
**PHASE-8** वो phase है जहाँ log me sab “ready” dikh raha hota hai,
volume controls aa jaate hain… **लेकिन आवाज़ अब भी नहीं आती** 😄

Main isko bhi **basic → system-level → exact boot flow → advanced debug** style me explain kar raha hoon
(Raspberry Pi + ALSA + WM8960 context)

------

# 🔊 PHASE 8: Userspace ALSA Init (Boot Time Only)

**(Basics → Full Advanced Explanation)**

------

## 0️⃣ One-line intuition (सबसे पहले)

> **PHASE-8 ka kaam audio bajana nahi,
> balki audio system ko “last known usable state” me lana hai**

❌ Playback start nahi
✅ Mixer / volume / routes restore

------

## 1️⃣ Kernel vs Userspace boundary (IMPORTANT)

### Ab tak kya hua:

- Kernel ne sound card register kar di (PHASE-7)
- `/dev/snd/*` nodes ban gaye

### Ab kya hota hai:

- Userspace ko signal milta hai:

  > “New sound device arrived”

👉 Yahin se **udev** ka role start hota hai

------

## 2️⃣ `udev` kya karta hai? (Basic but critical)

**udev = device manager**

Jab kernel:

```text
/dev/snd/controlC0
```

create karta hai,
tab uevent generate hota hai:

```
add /devices/.../sound/card0
```

👉 udev rule match karta hai

------

## 3️⃣ ALSA related udev rules kaha hote hain?

### Common paths:

```
/lib/udev/rules.d/
 ├── 50-udev-default.rules
 ├── 60-persistent-alsa.rules
 ├── 90-alsa-restore.rules   ⭐
```

(Exact numbering distro ke hisaab se thoda change ho sakta hai)

------

## 4️⃣ `90-alsa-restore.rules` — PHASE-8 ka HERO ⭐

Rule ka simplified version:

```text
SUBSYSTEM=="sound", ACTION=="add", \
  RUN+="/usr/sbin/alsactl restore %k"
```

### Matlab:

- Jab sound device add ho
- `alsactl restore` chalao

------

## 5️⃣ `alsactl restore` kya karta hai?

### ❓ Question:

“Restore from where?”

### 📍 State file:

```
/var/lib/alsa/asound.state
```

Is file me stored hota hai:

- Mixer volumes
- Switch states
- Mute/unmute
- DAPM route selections

👉 Ye file **last shutdown** pe save hoti hai

------

## 6️⃣ Boot-time mixer defaults kaise apply hote hain?

### Flow:

```
snd_card_register()
   ↓
/dev/snd created
   ↓
udev event
   ↓
alsactl restore
   ↓
Mixer state applied
```

📌 **No PCM open**
📌 **No playback**

------

## 7️⃣ Important clarity: alsactl kya kya nahi karta ❌

| Expectation           | Reality |
| --------------------- | ------- |
| Clock enable kare     | ❌       |
| I2S start kare        | ❌       |
| PCM stream open kare  | ❌       |
| Speaker power on kare | ❌       |

👉 Ye sab **runtime events** hain (PHASE-6)

------

## 8️⃣ Mixer defaults kab lagte hain?

### Timeline:

- Card register ke **immediately baad**
- User login se pehle
- Desktop start hone se pehle

Isliye:

> “Boot complete hote hi volume correct dikhta hai”

------

## 9️⃣ Practical check (Hands-on)

### Check restore rule:

```bash
udevadm test /sys/class/sound/card0
```

### Check state file:

```bash
alsactl dump
```

------

## 🔟 Common real-world issues (Advanced Debug)

### ❌ Volume zero after reboot

- `alsactl store` kabhi run nahi hua
- State file missing

Fix:

```bash
alsamixer
alsactl store
```

------

### ❌ alsactl restore fails silently

- Permission issue
- Wrong card index

Debug:

```bash
alsactl restore -v
```

------

## 1️⃣1️⃣ Mental model (Lock this forever 🧠)

```
Kernel:
  "Sound card bana di"

Userspace:
  "Last time jaise volume tha, waisa kar do"
```

------

## 1️⃣2️⃣ Why audio still silent after PHASE-8?

### Because:

- PCM open nahi hua
- Clocks gated hain
- DAPM paths off hain

👉 Audio tab aayega jab:

```bash
aplay test.wav
```

------

## 🏁 Final Outcome (PHASE-8 Mastery)

Ab aap confidently explain kar sakte ho:

✔️ udev ka role in ALSA boot
✔️ `alsactl restore` **exact kya karta hai**
✔️ Mixer defaults kab apply hote hain
✔️ Boot ke baad bhi audio silent kyun hota hai
✔️ Kernel vs userspace responsibility boundary

------

### 🔜 Last phase (Final polish)

👉 **PHASE-9: Boot Log Tracing & Debugging**
(jahan hum poora audio boot ko ms-level par trace karenge)

Bolo: **“PHASE-9 start karo”** 🔥