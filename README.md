# Sistem Monitoring & Kontrol Kelembaban Tanah & pH + ESP32-CAM Telegram Bot
  Proyek ini merupakan integrasi antara Arduino (ATMega/Uno/Nano/Mega) sebagai sistem kontrol utama dan ESP32-CAM sebagai modul pemantauan visual serta pengirim notifikasi ke Telegram bot. Sistem ini mampu:

## 📡 Fitur Utama Sistem
### 🟢 Modul Arduino
    - Pembacaan:
      - Sensor Soil Moisture
      - Sensor pH Tanah (DMS)
      - Sensor PIR
    - Pengaturan Setpoint:
      - Tanah basah / kering (potensiometer)
      - pH Asam / Basa (potensiometer)
    - Kontrol:
      - Relay Pompa Air
      - Relay Asam / Basa
      - Buzzer Alarm
      - Servo (bergerak saat ada gerakan PIR)
      - LCD 16x2 menampilkan status

### 📷 Modul ESP32-CAM
    - Mengambil foto ketika diminta via Telegram
      - Mengirim:
        - Foto terkini
        - Status tanah (basah/normal/kering)
        - Status pH (asam/normal/basa)
        - Angka pH dan kelembaban
    - Menerima request user melalui Telegram
    - Berkomunikasi dengan Arduino via TX-RX
    - Fitur Flash LED untuk pencahayaan

## 📨 Format Data Serial
### 📤 Arduino → ESP32-CAM
    #1!     → status tanah basah
    #2!     → status tanah normal
    #3!     → status tanah kering
    #11!    → pH asam
    #22!    → pH normal
    #33!    → pH basa
    #PHVALUE@      → nilai pH
    #MOISTURE&     → nilai kelembaban
### 📥 ESP32-CAM → Arduino
    #1?     → request status tanah + pH + nilai

## 📱 Command Telegram
Perintah	Fungsi
- /start	menampilkan menu
- /photo	ambil foto dari ESP32-CAM
- /status	kirim status tanah & pH
- /help	menampilkan bantuan



