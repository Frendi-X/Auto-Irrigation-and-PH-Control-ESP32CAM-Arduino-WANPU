# Auto-Irrigation-and-PH-Control-ESP32CAM-Arduino-WANPU
  Sistem ini dirancang sebagai solusi otomatisasi untuk pemantauan kondisi tanah (kelembaban & pH) sekaligus kontrol pompa cairan untuk menstabilkan kadar air dan tingkat keasaman. Selain itu, sistem juga
  menyediakan monitoring visual melalui ESP32-CAM yang terhubung dengan Telegram Bot. Integrasi ini memungkinkan pengguna memantau kondisi tanah dari jarak jauh, menerima notifikasi, meminta foto terbaru,
  serta mengontrol perangkat secara otomatis tanpa intervensi manual. [OMI - MAN 1 Ponorogo]

---

# 🧩 Fitur Utama Sistem

## 🔹 Sistem Kontrol Berbasis Arduino

Arduino bertugas sebagai pusat pengendali fisik:

* Membaca seluruh sensor (pH, moisture, PIR).
* Mengolah nilai ADC menjadi kategori status.
* Mengaktifkan pompa berdasarkan status tanah.
* Mengaktifkan pompa Asam/Basa berdasarkan status pH.
* Menggerakkan servo & buzzer saat ada gerakan.
* Menampilkan data pada LCD 16x2 I2C.
* Mengirim data ke ESP32-CAM secara rutin (interval 300–800 ms).

### Struktur logika Arduino:

1. **Pembacaan Sensor**

   * Soil moisture: analog (0–1023)
   * Sensor DMS pH: analog → dikonversi ke angka pH
   * PIR: HIGH/LOW
2. **Penentuan Status**
   **Kelembaban:**

   * Basah → moisture > 800
   * Normal → 450–800
   * Kering → < 450

   **pH:**

   * Asam → pH < 6
   * Normal → 6–7.5
   * Basa → >7.5
3. **Kontrol Pompa**
   Sesuai kategori yang dihitung.
4. **Pengendalian Alarm PIR**

   * Servo bergerak kiri-kanan 2x
   * Buzzer beep 2x
5. **Pengiriman Data Serial ke ESP32-CAM**
   4 jenis data dikirim dalam serial pattern.
6. **LCD Update hanya jika nilai berubah** (mengurangi flicker).

---

# 📸 Sistem ESP32-CAM + Telegram Bot

ESP32-CAM berfungsi sebagai sistem monitoring IoT dan antarmuka pengguna.

## Tugas utama ESP32-CAM:

* Menghubungkan ke WiFi.
* Menjalankan **Telegram Bot API**.
* Menerima data dari Arduino & menyimpannya sebagai variabel global.
* Mengirim foto bila user mengirim `/photo`.
* Menyusun status lengkap bila user mengirim `/status`.
* Menerima data realtime dari Arduino untuk notifikasi otomatis (opsional).

### Perintah Telegram:

| Perintah  | Fungsi                                                           |
| --------- | ---------------------------------------------------------------- |
| `/start`  | Menampilkan menu awal                                            |
| `/photo`  | Mengambil foto camera dan mengirim ke Telegram                   |
| `/status` | Menampilkan status pH, kelembaban, pompa aktif, dan nilai sensor |

---

# 🔗 Mekanisme Komunikasi Arduino ↔ ESP32-CAM

## Protokol komunikasi dibuat **singkat, tidak ambigu, dan low-latency**.

### ➤ Format Request dari ESP → Arduino

ESP meminta paket data menggunakan format:

```
#1?
```

Artinya: kirim semua status.

### ➤ Format Data dari Arduino → ESP32

Arduino mengirim 4 kategori data:

#### 1. Status Kelembaban Tanah

```
#1!  → Tanah Basah
#2!  → Tanah Normal
#3!  → Tanah Kering
```

#### 2. Status pH Tanah

```
#11! → pH Asam
#22! → pH Normal
#33! → pH Basa
```

#### 3. Nilai pH (numerik)

Contoh:

```
#6.82@
#7.30@
```

#### 4. Nilai Moisture (ADC atau %)

Contoh:

```
#712&
#389&
```

ESP32-CAM melakukan parsing karakter pertama & penutup:

* `!` → status kategori
* `@` → nilai pH
* `&` → nilai kelembaban

Parsing dilakukan character-by-character untuk menghindari corrupt buffer.

---

# ⚙ Struktur Hardware Lengkap

## Arduino (Input/Output)

| Pin | Fungsi                               |
| --- | ------------------------------------ |
| A0  | Sensor pH (DMS)                      |
| A1  | Setpoint pH (potensiometer opsional) |
| A2  | Soil Moisture                        |
| 5   | PIR                                  |
| 7   | Relay Pompa Air                      |
| 8   | Relay Pompa Asam                     |
| 9   | Relay Pompa Basa                     |
| 2   | Servo                                |
| 3   | LED indikator                        |
| 4   | Buzzer                               |
| 13  | LED status pH                        |
| 10  | RX SoftwareSerial                    |
| 11  | TX SoftwareSerial                    |

## ESP32-CAM

| Pin | Fungsi  |
| --- | ------- |
| U0R | UART RX |
| U0T | UART TX |

**Catatan penting:**
Gunakan **level shifter 5V → 3.3V** atau resistor divider pada jalur TX Arduino → RX ESP32.

---

# 🧠 Alur Kerja Sistem (Flow System)

## 💧 A. Alur Monitoring Tanah (Arduino)

1. Arduino membaca soil moisture.
2. Konversi nilai menjadi kategori.
3. Tampilkan ke LCD.
4. Kirim status ke ESP.
5. Jika “Kering” → Pompa Air ON
6. Jika “Basah/Normal” → Pompa Air OFF

## ⚗ B. Alur Kontrol pH (Arduino)

1. Arduino membaca nilai analog pH.
2. Konversi ke bentuk pH aktual.
3. Tentukan status:

   * Asam → Pompa Basa ON
   * Basa → Pompa Asam ON
   * Normal → kedua pompa OFF
4. Kirim status + nilai numerik ke ESP.

## 👁‍🗨 C. Alur Kamera (ESP32-CAM)

1. Menunggu perintah via Telegram.
2. Jika `/photo`:

   * Ambil gambar → kirim ke user.
3. Jika `/status`:

   * Susun data dari Arduino → kirim format teks lengkap.

## 🔒 D. Alur Keamanan PIR

1. PIR mendeteksi gerakan.
2. Servo bergerak 0° → 180° → 0°.
3. Buzzer beep pendek 2x.
4. Mengirim flag ke ESP (opsional).

---

# 🛡 Sistem Notifikasi (opsional)

Jika diaktifkan, ESP32-CAM dapat mengirim notifikasi otomatis:

* Tanah kering berkepanjangan
* pH terlalu asam atau terlalu basa
* PIR mendeteksi gerakan
* Pompa menyala terus (overrun alert)

Format pesan Telegram dapat berupa teks + foto.

---

# 8. 📊 Format Status Telegram

Contoh output `/status`:

```
📡 STATUS TANAH SAAT INI

🟫 Kelembaban : Kering (389)
🌡 pH Tanah   : Basa (7.82)

💧 Pompa Air  : ON
⚗ Pompa Basa : OFF
⚗ Pompa Asam : ON

📸 Gunakan /photo untuk melihat kamera
```

---

# 📌 Kelebihan Sistem

* Realtime, akurat, dan stabil.
* Hemat daya & cocok untuk greenhouse atau pertanian otomatis.
* Protokol serial anti-error.
* Bisa dikembangkan untuk IoT full (MQTT/Cloud).

---

# 📦 Pengembangan Lanjutan yang Direkomendasikan

* Menambah fitur **Auto-Fertilizer Pump**.
* Mengirim grafik 24 jam ke Telegram.
* Menambah datalogger ke SD Card (ESP32-CAM).
* Menggunakan sensor pH digital (SEN0161).
* Integrasi Web Dashboard ESP32.

---

# 📝 Kesimpulan

Dokumen ini memuat penjelasan teknis lengkap mengenai cara kerja sistem monitoring & kontrol tanah yang menggabungkan Arduino dan ESP32-CAM berbasis Telegram Bot. Sistem dirancang agar stabil, mudah dikembangkan, dan efisien untuk pertanian modern atau penelitian.

