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

---

## ✅ 800x600 resolutions exposed

`libdvi/dvi_timing.c` already ships `dvi_timing_800x600p_60hz` (400 MHz bit
clock, VESA blanking) and `dvi_timing_800x600p_reduced_60hz` (354 MHz, CVT
reduced blanking), but neither was reachable from the `DVIresolution` enum.

Added at the END of the enum (existing indices unchanged) and mirrored in the
`dvispec[]` table with `VREG_VOLTAGE_1_30`:

| Enum value                   | Timing                              | Bit clock |
|------------------------------|-------------------------------------|-----------|
| `DVI_RES_800x600p60`         | `dvi_timing_800x600p_60hz`          | 400 MHz   |
| `DVI_RES_800x600p60_reduced` | `dvi_timing_800x600p_reduced_60hz`  | 354 MHz   |

Both are serious overclocks (the RP2040/RP2350 runs at the TMDS bit clock);
intended for 1-bit (`DVIGFX1`) use on RP2350. The reduced-blanking variant is
the gentler overclock but is not accepted by every monitor.

> ⚠️ **The `vreg` column of the `dvispec[]` table is dead code.**  The voltage
> `PicoDVI::begin()` applies comes from the constructor's `vreg_voltage v`
> parameter (default `VREG_VOLTAGE_1_20`), NOT from the table.  Callers using
> the 800x600 modes MUST pass `VREG_VOLTAGE_1_30` explicitly, e.g.
> `DVIGFX1(DVI_RES_800x600p60_reduced, false, cfg, VREG_VOLTAGE_1_30)` —
> otherwise `begin()` silently drops the core voltage back to 1.20 V while the
> caller may already be running overclocked, hanging the chip.  (This also
> explains why the table's 1.30 V entry for 1280x720p30 never took effect
> upstream.)  Verified on RP2350B hardware: 800x600 reduced-blanking runs
> stable at 354 MHz / 1.30 V with the QMI flash divider raised to 4.
