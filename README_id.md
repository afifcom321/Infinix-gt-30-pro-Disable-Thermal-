# GT30Pro Thermal Kill (v1.1-KSU)

Modul **KernelSU** untuk melakukan *deep bypass* terhadap sistem thermal throttling bawaan pada **Infinix GT 30 Pro** (chipset MediaTek). Modul ini mematikan pembatasan suhu di level kernel/DVFS agar CPU, GPU, dan APU/NPU dapat bekerja terus di frekuensi maksimum, tanpa mematikan service `vendor.thermal-mediatek` (agar sistem tidak mendeteksi anomali thermal).

> **Status pengujian:** hanya diuji dengan **KernelSU** pada **Infinix GT 30 Pro**. **Belum diuji** di Magisk maupun perangkat/chipset MediaTek lain.

---

## ⚠️ Peringatan Sebelum Install

Modul ini **menonaktifkan hampir seluruh mekanisme proteksi panas** perangkat:

- Semua *thermal zone* kernel di-disable dan trip point-nya dinaikkan ke 200 °C.
- Semua *cooling device* dipaksa ke state 0 (tidak pernah aktif menurunkan performa).
- DVFS thermal ceiling MediaTek dihapus.
- Battery throttle (pembatasan performa saat baterai rendah/panas) dimatikan.
- CPU, GPU, dan APU/NPU dikunci ke frekuensi maksimum secara terus-menerus, dijaga oleh *watchdog* yang berjalan tiap 5 detik.

Akibatnya, perangkat **berpotensi menjadi sangat panas**, baterai dapat **terdegradasi lebih cepat**, dan penggunaan dalam waktu lama/beban berat (gaming, benchmark berulang) **berisiko merusak komponen hardware secara permanen**. Modul ini ditujukan untuk pengujian/benchmark singkat, bukan untuk pemakaian harian.

Karena mekanisme thermal protection sengaja dilumpuhkan, **selalu pantau suhu perangkat secara manual** saat modul ini aktif, dan hentikan pemakaian (uninstall modul / reboot) jika perangkat terasa panas berlebih.

---

## Disclaimer Tanggung Jawab

Modul ini dibagikan **apa adanya (as-is)**, tanpa jaminan dalam bentuk apa pun, untuk tujuan eksperimen/benchmark pada Infinix GT 30 Pro dengan KernelSU.

- Belum ada pengujian di perangkat atau kombinasi root method lain.
- Segala risiko pemakaian — termasuk namun tidak terbatas pada bootloop, softbrick, kerusakan baterai, atau kerusakan hardware lain — **sepenuhnya menjadi tanggung jawab pengguna**.
- Author (**Afif Sakti**) tidak bertanggung jawab atas kerusakan perangkat yang timbul dari penggunaan modul ini, khususnya pada perangkat selain Infinix GT 30 Pro yang belum pernah diuji.

Gunakan hanya jika Anda memahami risikonya dan tahu cara memulihkan perangkat (misalnya lewat recovery mode atau flash tool resmi) apabila terjadi bootloop.

---

## Fitur / Apa yang Dilakukan Modul Ini

1. **Stop service thermal non-esensial** — menghentikan `android.hardware.thermal-service.mediatek`, `thermal_core`, `thermald`, dan proses lain yang mengandung kata "thermal", **kecuali** `vendor.thermal-mediatek` yang sengaja dibiarkan tetap berjalan.
2. **Disable seluruh thermal zone kernel** — mode di-set `disabled`, semua trip point dinaikkan ke 200000 (200 °C), policy diubah ke `user_space`.
3. **Force cooling device ke state 0** — memastikan tidak ada throttling aktif dari sisi cooling device.
4. **CPU frequency lock** — governor semua core diset ke `performance`, `scaling_min_freq` dan `scaling_max_freq` dikunci ke `cpuinfo_max_freq`.
5. **GPU frequency lock** — governor GPU Mali diset ke `performance` dengan `min_freq` dikunci ke `max_freq`, power policy `always_on`.
6. **APU/NPU unlock** — frekuensi APU/MDLA (AI engine) dikunci ke maksimum untuk memperbaiki skor CPU AI/benchmark.
7. **Battery throttle disable** — menonaktifkan power capping berbasis level baterai (`persist.vendor.battery.perf.disable`, dsb).
8. **Scheduler tuning** — `schedtune.boost` diset tinggi untuk top-app/foreground, `sched_autogroup` dan `cpu_dma_latency` dinonaktifkan, CPU boost diaktifkan.
9. **Hapus DVFS thermal ceiling MediaTek** lewat berbagai path debugfs/procfs.
10. **Watchdog "aman"** — proses background yang tiap 5 detik mengecek apakah `android.hardware.thermal-service.mediatek` menyala kembali (lalu dimatikan ulang) dan me-reset cooling device yang kembali aktif, **tanpa** mematikan `vendor.thermal-mediatek`.

Log aktivitas modul disimpan di:
```
/data/local/tmp/gt30pro_thermal.log
```

---

## Struktur File Modul

| File | Fungsi |
|---|---|
| `module.prop` | Metadata modul (id, nama, versi, author, deskripsi) untuk KernelSU/Magisk Manager |
| `system.prop` | Property build-time: disable MTK Thermal 2.0, DVFS bypass, APU/NPU thermal disable, battery throttle disable |
| `customize.sh` | Script saat instalasi — menampilkan info analisis & instruksi di layar KernelSU Manager, menyalin file modul |
| `post-fs-data.sh` | Dijalankan sangat awal (sebelum boot penuh) — disable thermal zone lewat sysfs sedini mungkin |
| `service.sh` | Dijalankan setelah boot selesai — menjalankan seluruh bypass (CPU/GPU/APU lock, battery throttle disable, scheduler tuning) dan watchdog |

---

## Requirement

- Infinix GT 30 Pro (chipset MediaTek)
- Bootloader unlocked
- Root akses via **KernelSU**
- Akses ke terminal/ADB dianjurkan, untuk keperluan cek log atau uninstall darurat

## Urutan Instalasi yang Disarankan

Sesuai catatan pada `customize.sh`, urutan loading modul mengikuti urutan nama di KernelSU Manager. Disarankan flash dengan urutan berikut:

1. `GT30Pro Performance v4.1`
2. `GT30Pro GPU Fix v1.0`
3. `GT30Pro Thermal Kill v1.1` (modul ini)

## Cara Install

1. Download file zip modul dari halaman [Releases] repo ini.
2. Buka **KernelSU Manager**.
3. Pilih **Module → Install from storage**, lalu pilih file zip modul.
4. Tunggu proses instalasi selesai, lalu **reboot** perangkat.
5. Setelah boot selesai, cek log di `/data/local/tmp/gt30pro_thermal.log` untuk memastikan modul berjalan dengan benar.

## Tips Benchmark (dari catatan pembuat modul)

- Charge baterai ke 80% atau lebih sebelum benchmark.
- Atau sambungkan charger selama proses benchmark berlangsung.
- Dinginkan perangkat sekitar 5 menit sebelum mulai test.
- Tutup semua aplikasi latar belakang sebelum menjalankan AnTuTu atau benchmark lain.

## Verifikasi Setelah Install

Cek log lewat terminal/ADB:
```bash
cat /data/local/tmp/gt30pro_thermal.log
```

Yang seharusnya terlihat di bagian akhir log:
- `vendor.thermal-mediatek` → **running** (memang sengaja dipertahankan)
- `android.hardware.thermal-service.mediatek` → **stopped**
- Frekuensi CPU0 dan CPU7 mendekati frekuensi maksimum chipset

## Cara Uninstall

1. Buka **KernelSU Manager → Modules**.
2. Cari **GT 30 Pro Thermal Kill**, lalu pilih **Remove/Uninstall**.
3. Reboot perangkat.

**Jika terjadi bootloop:**
- Masuk ke Recovery/Safe Mode jika memungkinkan, lalu nonaktifkan modul lewat KernelSU Manager.
- Jika masih bisa akses ADB, hapus folder modul secara manual:
  ```bash
  adb shell
  su
  rm -rf /data/adb/modules/GT30Pro_ThermalKill
  reboot
  ```
- Jika perangkat tidak bisa diakses sama sekali, opsi terakhir adalah flash ulang firmware resmi Infinix GT 30 Pro melalui SP Flash Tool (MediaTek).

---

## Kompatibilitas

| Perangkat | Root Method | Status |
|---|---|---|
| Infinix GT 30 Pro | KernelSU | ✅ Sudah diuji |
| Infinix GT 30 Pro | Magisk | ❌ Belum diuji |
| Perangkat MediaTek lain | KernelSU/Magisk | ❌ Belum diuji, tidak disarankan |

---

## Author

**Afif Sakti**

## Lisensi

Belum ditentukan — tambahkan file `LICENSE` sesuai preferensi Anda (mis. MIT) sebelum publish ke GitHub jika ingin mengatur izin penggunaan/modifikasi ulang secara eksplisit.
