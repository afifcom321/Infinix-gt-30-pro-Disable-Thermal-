# GT30Pro Thermal Kill (v1.1-KSU)

A **KernelSU** module that performs a deep bypass of the built-in thermal throttling system on the **Infinix GT 30 Pro** (MediaTek chipset). It disables temperature limiting at the kernel/DVFS level so the CPU, GPU, and APU/NPU can keep running at maximum frequency, without stopping the `vendor.thermal-mediatek` service (so the system doesn't flag a thermal anomaly).

> **Testing status:** tested only with **KernelSU** on the **Infinix GT 30 Pro**. **Not tested** on Magisk or on other MediaTek devices/chipsets.

---

## ⚠️ Warning Before You Install

This module **disables almost all of the device's thermal protection mechanisms**:

- All kernel thermal zones are disabled, with trip points raised to 200 °C.
- All cooling devices are forced to state 0 (throttling never activates).
- MediaTek's DVFS thermal ceiling is removed.
- Battery throttle (performance limiting when the battery is low/hot) is disabled.
- CPU, GPU, and APU/NPU are locked to maximum frequency continuously, enforced by a watchdog that runs every 5 seconds.

As a result, the device **can get very hot**, the battery **may degrade faster**, and extended/heavy use (gaming, repeated benchmarking) **risks permanent hardware damage**. This module is meant for short testing/benchmark sessions, not daily use.

Because thermal protection is deliberately disabled, **always monitor device temperature manually** while this module is active, and stop using it (uninstall / reboot) if the device gets excessively hot.

---

## Liability Disclaimer

This module is provided **as-is**, without any warranty, for experimentation/benchmarking purposes on the Infinix GT 30 Pro with KernelSU.

- It has not been tested on other devices or root method combinations.
- All risks of use — including but not limited to bootloops, soft-bricking, battery damage, or other hardware damage — are **entirely the user's responsibility**.
- The author (**Afif Sakti**) is not responsible for any device damage resulting from the use of this module, especially on devices other than the Infinix GT 30 Pro, which have not been tested.

Only use this module if you understand the risks and know how to recover your device (e.g. via recovery mode or an official flashing tool) in case of a bootloop.

---

## What This Module Does

1. **Stops non-essential thermal services** — kills `android.hardware.thermal-service.mediatek`, `thermal_core`, `thermald`, and any other process containing "thermal" in its name, **except** `vendor.thermal-mediatek`, which is deliberately left running.
2. **Disables all kernel thermal zones** — mode set to `disabled`, all trip points raised to 200000 (200 °C), policy switched to `user_space`.
3. **Forces cooling devices to state 0** — ensures no cooling-device-side throttling is ever active.
4. **CPU frequency lock** — governor on every core set to `performance`, with `scaling_min_freq` and `scaling_max_freq` locked to `cpuinfo_max_freq`.
5. **GPU frequency lock** — Mali GPU governor set to `performance`, `min_freq` locked to `max_freq`, power policy set to `always_on`.
6. **APU/NPU unlock** — APU/MDLA (AI engine) frequency locked to maximum to improve AI/benchmark scores.
7. **Battery throttle disable** — disables battery-level-based power capping (`persist.vendor.battery.perf.disable`, etc.).
8. **Scheduler tuning** — raises `schedtune.boost` for top-app/foreground, disables `sched_autogroup` and `cpu_dma_latency`, enables CPU boost.
9. **Removes MediaTek's DVFS thermal ceiling** via several debugfs/procfs paths.
10. **"Safe" watchdog** — a background loop that checks every 5 seconds whether `android.hardware.thermal-service.mediatek` has restarted (and stops it again) and resets any cooling device that becomes active again, **without** touching `vendor.thermal-mediatek`.

Activity logs are saved to:
```
/data/local/tmp/gt30pro_thermal.log
```

---

## Module File Structure

| File | Purpose |
|---|---|
| `module.prop` | Module metadata (id, name, version, author, description) shown in KernelSU/Magisk Manager |
| `system.prop` | Build-time properties: disables MTK Thermal 2.0, DVFS bypass, APU/NPU thermal disable, battery throttle disable |
| `customize.sh` | Install-time script — prints analysis info & instructions in the KernelSU Manager screen, copies module files |
| `post-fs-data.sh` | Runs very early (before full boot) — disables thermal zones via sysfs as early as possible |
| `service.sh` | Runs after boot completes — performs the full bypass (CPU/GPU/APU lock, battery throttle disable, scheduler tuning) and starts the watchdog |

---

## Requirements

- Infinix GT 30 Pro (MediaTek chipset)
- Unlocked bootloader
- Root access via **KernelSU**
- Terminal/ADB access recommended, for checking logs or emergency uninstall



## Installation

1. Download the module zip from this repo's [Releases] page.
2. Open **KernelSU Manager**.
3. Go to **Module → Install from storage**, then select the module zip.
4. Wait for installation to finish, then **reboot** the device.
5. After boot, check the log at `/data/local/tmp/gt30pro_thermal.log` to confirm the module is running correctly.

## Benchmark Tips (from the module author's notes)

- Charge the battery to 80% or higher before benchmarking.
- Or keep the charger connected during the benchmark.
- Let the device cool down for about 5 minutes before testing.
- Close all background apps before running AnTuTu or any other benchmark.

## Post-Install Verification

Check the log via terminal/ADB:
```bash
cat /data/local/tmp/gt30pro_thermal.log
```

At the end of the log you should see:
- `vendor.thermal-mediatek` → **running** (intentionally left running)
- `android.hardware.thermal-service.mediatek` → **stopped**
- CPU0 and CPU7 frequencies close to the chipset's maximum frequency

## Uninstalling

1. Open **KernelSU Manager → Modules**.
2. Find **GT 30 Pro Thermal Kill**, then select **Remove/Uninstall**.
3. Reboot the device.

**If you experience a bootloop:**
- Boot into Recovery/Safe Mode if possible, then disable the module via KernelSU Manager.
- If ADB access is still available, remove the module folder manually:
  ```bash
  adb shell
  su
  rm -rf /data/adb/modules/GT30Pro_ThermalKill
  reboot
  ```
- As a last resort, reflash the official Infinix GT 30 Pro firmware using SP Flash Tool (MediaTek).

---

## Compatibility

| Device | Root Method | Status |
|---|---|---|
| Infinix GT 30 Pro | KernelSU | ✅ Tested |
| Infinix GT 30 Pro | Magisk | ❌ Not tested |
| Other MediaTek devices | KernelSU/Magisk | ❌ Not tested, not recommended |

---

## Author

**Afif Sakti**

## License

Not yet specified — add a `LICENSE` file to your preference (e.g. MIT) before publishing to GitHub if you want to explicitly set terms for use/redistribution.
