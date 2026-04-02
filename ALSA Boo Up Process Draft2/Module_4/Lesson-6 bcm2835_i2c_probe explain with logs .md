bcm2835_i2c_probe





static int bcm2835_i2c_probe(struct platform_device *pdev)
{
    struct bcm2835_i2c_dev *i2c_dev;
    int ret;
    struct i2c_adapter *adap;
    struct clk *mclk;
    u32 bus_clk_rate;

    printk(KERN_INFO "vijayp: [%s] ENTER bcm2835_i2c_probe\n", __func__);
    
    /* Allocate driver private structure */
    i2c_dev = devm_kzalloc(&pdev->dev, sizeof(*i2c_dev), GFP_KERNEL);
    if (!i2c_dev) {
        printk(KERN_ERR "vijayp: kzalloc failed\n");
        return -ENOMEM;
    }
    printk(KERN_INFO "vijayp: memory allocated for bcm2835_i2c_dev\n");
    
    platform_set_drvdata(pdev, i2c_dev);
    i2c_dev->dev = &pdev->dev;
    
    init_completion(&i2c_dev->completion);
    printk(KERN_INFO "vijayp: completion initialized\n");
    
    /* Map I2C registers */
    i2c_dev->regs = devm_platform_get_and_ioremap_resource(pdev, 0, NULL);
    if (IS_ERR(i2c_dev->regs)) {
        printk(KERN_ERR "vijayp: ioremap resource failed\n");
        return PTR_ERR(i2c_dev->regs);
    }
    printk(KERN_INFO "vijayp: I2C registers mapped\n");
    
    /* Get parent clock */
    mclk = devm_clk_get(&pdev->dev, NULL);
    if (IS_ERR(mclk)) {
        printk(KERN_ERR "vijayp: clock get failed\n");
        return dev_err_probe(&pdev->dev, PTR_ERR(mclk),
                             "Could not get clock\n");
    }
    printk(KERN_INFO "vijayp: parent clock acquired\n");
    
    /* Register I2C bus clock */
    i2c_dev->bus_clk = bcm2835_i2c_register_div(&pdev->dev, mclk, i2c_dev);
    if (IS_ERR(i2c_dev->bus_clk)) {
        printk(KERN_ERR "vijayp: clock register failed\n");
        return dev_err_probe(&pdev->dev, PTR_ERR(i2c_dev->bus_clk),
                             "Could not register clock\n");
    }
    printk(KERN_INFO "vijayp: bus clock registered\n");
    
    /* Read clock-frequency from device tree */
    ret = of_property_read_u32(pdev->dev.of_node, "clock-frequency",
                               &bus_clk_rate);
    if (ret < 0) {
        printk(KERN_WARNING "vijayp: clock-frequency property not found, using default\n");
        bus_clk_rate = I2C_MAX_STANDARD_MODE_FREQ;
    } else {
        printk(KERN_INFO "vijayp: clock-frequency = %u\n", bus_clk_rate);
    }
    
    /* Set bus clock rate */
    ret = clk_set_rate_exclusive(i2c_dev->bus_clk, bus_clk_rate);
    if (ret < 0) {
        printk(KERN_ERR "vijayp: clock rate set failed\n");
        return dev_err_probe(&pdev->dev, ret,
                             "Could not set clock frequency\n");
    }
    printk(KERN_INFO "vijayp: clock rate configured\n");
    
    /* Enable clock */
    ret = clk_prepare_enable(i2c_dev->bus_clk);
    if (ret) {
        printk(KERN_ERR "vijayp: clock enable failed\n");
        goto err_put_exclusive_rate;
    }
    printk(KERN_INFO "vijayp: clock enabled\n");
    
    /* Get IRQ */
    i2c_dev->irq = platform_get_irq(pdev, 0);
    if (i2c_dev->irq < 0) {
        printk(KERN_ERR "vijayp: IRQ get failed\n");
        ret = i2c_dev->irq;
        goto err_disable_unprepare_clk;
    }
    printk(KERN_INFO "vijayp: IRQ number = %d\n", i2c_dev->irq);
    
    /* Register IRQ handler */
    ret = request_irq(i2c_dev->irq, bcm2835_i2c_isr, IRQF_SHARED,
                      dev_name(&pdev->dev), i2c_dev);
    if (ret) {
        printk(KERN_ERR "vijayp: request_irq failed\n");
        goto err_disable_unprepare_clk;
    }
    printk(KERN_INFO "vijayp: IRQ handler registered\n");
    
    /* Setup I2C adapter */
    adap = &i2c_dev->adapter;
    i2c_set_adapdata(adap, i2c_dev);
    adap->owner = THIS_MODULE;
    adap->class = I2C_CLASS_DEPRECATED;
    
    snprintf(adap->name, sizeof(adap->name), "bcm2835 (%s)",
             of_node_full_name(pdev->dev.of_node));
    
    adap->algo = &bcm2835_i2c_algo;
    adap->dev.parent = &pdev->dev;
    adap->dev.of_node = pdev->dev.of_node;
    adap->quirks = of_device_get_match_data(&pdev->dev);
    
    printk(KERN_INFO "vijayp: I2C adapter configured\n");
    
    /* Disable clock stretching timeout */
    bcm2835_i2c_writel(i2c_dev, BCM2835_I2C_CLKT, 0);
    bcm2835_i2c_writel(i2c_dev, BCM2835_I2C_C, 0);
    
    printk(KERN_INFO "vijayp: hardware registers initialized\n");
    
    /* Register adapter with I2C core */
    ret = i2c_add_adapter(adap);
    if (ret) {
        printk(KERN_ERR "vijayp: i2c_add_adapter failed\n");
        goto err_free_irq;
    }
    
    printk(KERN_INFO "vijayp: I2C adapter registered successfully\n");
    printk(KERN_INFO "vijayp: EXIT bcm2835_i2c_probe\n");
    
    return 0;

err_free_irq:
    free_irq(i2c_dev->irq, i2c_dev);

err_disable_unprepare_clk:
    clk_disable_unprepare(i2c_dev->bus_clk);

err_put_exclusive_rate:
    clk_rate_exclusive_put(i2c_dev->bus_clk);

    printk(KERN_ERR "vijayp: bcm2835_i2c_probe failed ret=%d\n", ret);
    
    return ret;
}











आपके logs दिखाते हैं कि **Raspberry Pi 4B का I2C controller driver (`bcm2835_i2c_probe`) सफलतापूर्वक initialize हो गया है**।
मैं हर line को **simple Hinglish में step-by-step explain** करता हूँ।

------

# Overall summary

Time stamp:

```
[7.33 sec]
```

मतलब:

```
Kernel boot के लगभग 7.3 सेकंड बाद
I2C controller driver initialize हुआ
```

अब हर step समझते हैं।

------

# 1️⃣ Probe function start

```
[7.333425] ENTER bcm2835_i2c_probe
```

मतलब:

```
Device Tree match हो गया
↓
Kernel ने bcm2835_i2c_probe() call कर दिया
```

यहीं से **I2C controller initialization शुरू होता है**।

------

# 2️⃣ Driver private memory allocate

```
memory allocated for bcm2835_i2c_dev
```

यह code चला:

```
devm_kzalloc()
```

मतलब:

```
Driver ने अपना internal data structure बनाया
```

इसमें store होगा:

```
register address
IRQ
clock
adapter info
```

------

# 3️⃣ Completion object initialize

```
completion initialized
```

यह code:

```
init_completion(&i2c_dev->completion)
```

मतलब:

```
I2C transfer complete होने का signal handle करने के लिए
completion mechanism setup हुआ
```

Kernel में **synchronization mechanism** है।

------

# 4️⃣ Hardware registers map

```
I2C registers mapped
```

यह code:

```
devm_platform_get_and_ioremap_resource()
```

मतलब:

```
Physical I2C hardware registers
↓
Kernel virtual address space में map
```

अब driver hardware registers access कर सकता है।

Example register:

```
BCM2835_I2C_C
BCM2835_I2C_S
BCM2835_I2C_DLEN
```

------

# 5️⃣ Parent clock मिला

```
parent clock acquired
```

यह code:

```
devm_clk_get()
```

मतलब:

```
I2C controller को clock signal चाहिए
```

Raspberry Pi SoC में हर peripheral clock से चलता है।

------

# 6️⃣ I2C bus clock create हुआ

```
bus clock registered
```

यह code:

```
bcm2835_i2c_register_div()
```

मतलब:

```
Parent clock से I2C bus clock derive किया गया
```

यानी:

```
CPU clock
↓
I2C bus clock
```

------

# 7️⃣ Device Tree से frequency पढ़ी

```
clock-frequency = 100000
```

Device Tree में लिखा होता है:

```
clock-frequency = <100000>;
```

मतलब:

```
I2C bus speed = 100 kHz
```

यह standard mode है।

Modes:

```
Standard  → 100 kHz
Fast      → 400 kHz
Fast+     → 1 MHz
```

------

# 8️⃣ Clock rate set

```
clock rate configured
```

यह code:

```
clk_set_rate_exclusive()
```

मतलब:

```
I2C controller का clock 100kHz पर set कर दिया
```

------

# 9️⃣ Clock enable

```
clock enabled
```

यह code:

```
clk_prepare_enable()
```

मतलब:

```
अब hardware को actual clock signal मिलना शुरू हो गया
```

अब controller **operate कर सकता है**।

------

# 🔟 IRQ number मिला

```
IRQ number = 41
```

यह code:

```
platform_get_irq()
```

मतलब:

```
I2C controller interrupt number = 41
```

जब I2C transfer complete होगा:

```
hardware interrupt generate करेगा
```

------

# 11️⃣ IRQ handler register

```
IRQ handler registered
```

यह code:

```
request_irq()
```

मतलब:

```
Kernel को बता दिया:

अगर interrupt 41 आए
तो यह function चलाओ
bcm2835_i2c_isr()
```

------

# 12️⃣ I2C adapter setup

```
I2C adapter configured
```

यह code:

```
struct i2c_adapter setup
```

मतलब:

```
Linux kernel को बताया गया
यह एक I2C bus controller है
```

------

# 13️⃣ Hardware registers initialize

```
hardware registers initialized
```

यह code:

```
bcm2835_i2c_writel()
```

मतलब:

```
I2C controller registers reset / initialize किए गए
```

Example:

```
timeout disable
control register reset
```

------

# 14️⃣ I2C adapter register

```
I2C adapter registered successfully
```

यह **सबसे important line है**।

यह code:

```
i2c_add_adapter()
```

मतलब:

```
Linux kernel में नई I2C bus register हो गई
```

अब:

```
/dev/i2c-1
```

device available हो जाता है।

और:

```
I2C devices probe हो सकते हैं
```

------

# 15️⃣ Probe finish

```
EXIT bcm2835_i2c_probe
```

मतलब:

```
I2C controller successfully initialize हो गया
```

------

# Final execution flow

आपके log का complete flow:

```
Device Tree match
        ↓
bcm2835_i2c_probe()
        ↓
memory allocate
        ↓
register map
        ↓
clock setup
        ↓
IRQ setup
        ↓
I2C adapter setup
        ↓
i2c_add_adapter()
        ↓
I2C bus ready
```

------

# Important observation

आपका I2C controller start हो गया है:

```
bus speed  = 100kHz
IRQ        = 41
adapter    = registered
```

मतलब:

```
अब I2C devices communicate कर सकते हैं
```

जैसे:

```
WM8960 codec
EEPROM
RTC
Sensor
```

------

💡 अगर चाहें तो मैं आपको **next step भी दिखा सकता हूँ**:

```
i2c_add_adapter()
↓
I2C device driver probe
```

यानी:

```
I2C controller start
↓
I2C devices detect
↓
device driver probe
```

यह **ALSA audio codec initialization समझने में बहुत important step है** (जो आप Raspberry Pi में सीख रहे हैं)।