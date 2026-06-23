# Sumo-Robot

Robot Sumo adalah robot otonom yang bertanding dalam arena berbentuk lingkaran (dohyo). Robot dilengkapi dengan sensor untuk mendeteksi lawan dan garis batas arena, motor penggerak untuk manuver, serta sistem kontrol berbasis mikrokontroler untuk mengeksekusi strategi pertandingan secara mandiri tanpa intervensi manusia setelah pertandingan dimulai.

## Tujuan Proyek

- Mengaplikasikan konsep robotika, elektronika, dan pemrograman embedded
- Mengembangkan kemampuan desain mekanik, integrasi sensor, dan logika kontrol
- Membangun robot yang kompetitif sesuai standar pertandingan robot sumo
- Melatih kerjasama tim dalam proyek engineering

## Spesifikasi Hardware

| Komponen | Jumlah | Fungsi |
|----------|--------|--------|
| ESP32 | 1 | Mikrokontroler utama (otak robot) |
| Sensor IR | 4 | Deteksi garis putih batas arena (dohyo) |
| Sensor Ultrasonik (HC-SR04) | 3 | Deteksi posisi & jarak lawan |
| Motor DC Kuning (TT Motor) | 2 | Penggerak roda kiri & kanan |
| Driver Motor L298N | 1 | Kontrol kecepatan & arah motor |
| Sensor IMU MPU6050 | 1 | Deteksi kondisi deadlock (stuck/terbalik) |
| Baterai Li-Ion / LiPo | 1 set | Sumber daya |

## Arsitektur Sistem
[4x IR Sensor]            [3x Ultrasonic]
          |                          |
          v                          v
    +---------------------------------------+
    |              ESP32                    |
    |  (Strategi & Pengambilan Keputusan)   |
    +---------------------------------------+
          ^                          |
          |                          v
    [MPU6050 IMU]               [L298N Driver]
                                     |
                                +----+----+
                                v         v
                          [Motor DC]  [Motor DC]
                            Kiri        Kanan


## Konfigurasi Sensor

### Sensor IR (4 buah) - Deteksi Garis
- **Depan Kiri & Depan Kanan**: deteksi garis saat menyerang ke depan
- **Belakang Kiri & Belakang Kanan**: deteksi garis saat mundur / didorong lawan

### Sensor Ultrasonik (3 buah) - Deteksi Lawan
- **Depan Tengah**: deteksi lawan tepat di depan untuk serangan langsung
- **Depan Kiri (serong)**: deteksi lawan di sisi kiri
- **Depan Kanan (serong)**: deteksi lawan di sisi kanan

### Sensor IMU MPU6050 - Anti-Deadlock
- Mendeteksi kondisi robot yang **stuck** (saling dorong tanpa pergerakan)
- Membaca **percepatan (accelerometer)** dan **rotasi (gyroscope)**
- Jika nilai pergerakan mendekati nol dalam waktu tertentu padahal motor aktif → trigger **manuver escape**

## Logika Kontrol (State Machine)
1. **SEARCH** — robot berputar mencari lawan menggunakan ultrasonik
2. **ATTACK** — maju penuh ke arah lawan terdekat
3. **AVOID_LINE** — mundur & berbelok jika IR mendeteksi garis batas
4. **ESCAPE** — manuver khusus (mundur + putar) saat MPU6050 mendeteksi deadlock

## Pin Mapping (ESP32)

| Komponen | Pin ESP32 |
|----------|-----------|
| IR Depan Kiri | GPIO 34 |
| IR Depan Kanan | GPIO 35 |
| IR Belakang Kiri | GPIO 32 |
| IR Belakang Kanan | GPIO 33 |
| Ultrasonik Depan (Trig/Echo) | GPIO 13 / 12 |
| Ultrasonik Kiri (Trig/Echo) | GPIO 14 / 27 |
| Ultrasonik Kanan (Trig/Echo) | GPIO 26 / 25 |
| L298N IN1, IN2, IN3, IN4 | GPIO 16, 17, 18, 19 |
| L298N ENA, ENB (PWM) | GPIO 4, 5 |
| MPU6050 SDA, SCL | GPIO 21, 22 |

> *Pin mapping dapat disesuaikan dengan kebutuhan wiring.*

## Library yang Digunakan

- `Wire.h` — komunikasi I2C untuk MPU6050
- `MPU6050.h` / `Adafruit_MPU6050.h` — pembacaan IMU
- `NewPing.h` — pembacaan sensor ultrasonik
- `Arduino.h` — fungsi dasar

## Cara Menjalankan

1. Clone repository ini:
```bash
   git clone https://github.com/bryansasa/Sumo-Robot.git
```
2. Buka project di **Arduino IDE** atau **PlatformIO**
3. Install library yang dibutuhkan
4. Pilih board **ESP32 Dev Module**
5. Upload kode ke ESP32
6. Pasang baterai dan lakukan pengujian di arena dohyo

## Strategi Bertanding

- **Agresif**: prioritaskan serangan ke arah lawan terdekat
- **Defensif terhadap garis**: IR memiliki prioritas tertinggi (jangan keluar arena)
- **Anti-stuck**: MPU6050 mencegah robot terjebak dalam dorongan statis dengan lawan

## Tim Pengembang

- Bryan Sasabone
- *(tambahkan anggota tim lainnya)*

## Lisensi

Project ini dibuat untuk keperluan edukasi dan kompetisi robotika.
