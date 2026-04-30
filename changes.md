# 🛠️ Changes in this Fork

> Fork based on [PicoDVI - Adafruit Fork](https://github.com/adafruit/PicoDVI)

---

## ✅ RP2350 Compatibility Fix

### ⚠️ Context

In the PicoDVI library, the **RP2040-PiZero** config uses different pins than the **RP2350-PiZero** board.
The RP2350B exposes GPIO pins with indices ≥ 32, which requires the use of 64-bit mask variants.

---

### 📝 Modified File

**[`src/libdvi/dvi_serialiser.pio.h`](src/libdvi/dvi_serialiser.pio.h)** — line 80

---

### 🔴 Original Code

```c
pio_sm_set_pins_with_mask(pio, sm, 2u << data_pins, 3u << data_pins);
pio_sm_set_pindirs_with_mask(pio, sm, ~0u, 3u << data_pins);
```

### 🟢 Replacement Code

```c
pio_sm_set_pins_with_mask64(pio, sm, 2ull << data_pins, 3ull << data_pins);
pio_sm_set_pindirs_with_mask64(pio, sm, ~0ull, 3ull << data_pins);
```

---

### 💡 Reason

The `_with_mask64` variants support GPIO pin indices ≥ 32, which are required for full **RP2350B** compatibility.
Without this change, the 32-bit shift produces an incorrect mask and the DVI data pins are not initialized properly.
