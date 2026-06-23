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
| ESP32 | 1 | Mikrokontroler utama (otak robot, RTOS dual-core) |
| Sensor IR | 4 | Deteksi garis putih batas arena (dohyo) |
| Sensor Ultrasonik (HC-SR04) | 3 | Deteksi posisi & jarak lawan |
| Motor DC Kuning (TT Motor) | 2 | Penggerak roda kiri & kanan |
| Driver Motor L298N | 1 | Kontrol kecepatan & arah motor (PWM via LEDC) |
| Sensor IMU MPU6050 | 1 | Deteksi kondisi deadlock saat dorong-dorongan |
| Baterai Li-Ion / LiPo | 1 set | Sumber daya |

## Arsitektur Sistem
[4x IR Sensor]            [3x Ultrasonic]
          |                          |
          v                          v
    +---------------------------------------+
    |              ESP32                    |
    |  (RTOS - State Machine - PID Steering)|
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
- **Depan Kiri & Depan Kanan**: deteksi garis saat menyerang ke depan (prioritas tertinggi)
- **Belakang Kiri & Belakang Kanan**: deteksi garis saat mundur / didorong lawan

### Sensor Ultrasonik (3 buah) - Deteksi Lawan
- **Depan Tengah**: deteksi lawan tepat di depan untuk serangan langsung (ATTACK / PUSH)
- **Depan Kiri (serong)**: deteksi lawan di sisi kiri (TRACK_LEFT - pivot kiri)
- **Depan Kanan (serong)**: deteksi lawan di sisi kanan (TRACK_RIGHT - pivot kanan)

### Sensor IMU MPU6050 - Anti-Deadlock
- Membaca **percepatan sumbu X (accelerometer)** via I2C raw register
- Jika lawan dekat (≤ 17 cm) **dan** percepatan mendekati nol selama 7 detik → trigger **manuver escape** (mundur + belok)

## Pin Mapping (ESP32)

### IR Sensor (Garis)
| Sensor | Pin ESP32 |
|--------|-----------|
| IR Kiri Depan | GPIO 33 |
| IR Kanan Depan | GPIO 32 |
| IR Kiri Belakang | GPIO 25 |
| IR Kanan Belakang | GPIO 26 |

### Ultrasonik HC-SR04 (Deteksi Lawan)
| Sensor | TRIG | ECHO |
|--------|------|------|
| Ultrasonik Kiri | GPIO 22 | GPIO 35 |
| Ultrasonik Tengah | GPIO 19 | GPIO 18 |
| Ultrasonik Kanan | GPIO 23 | GPIO 34 |

### Driver Motor L298N
| Fungsi | Pin ESP32 |
|--------|-----------|
| ENA (PWM Motor Kiri) | GPIO 27 |
| IN1 (Motor Kiri) | GPIO 15 |
| IN2 (Motor Kiri) | GPIO 14 |
| ENB (PWM Motor Kanan) | GPIO 2 |
| IN3 (Motor Kanan) | GPIO 13 |
| IN4 (Motor Kanan) | GPIO 4 |

### MPU6050 (I2C)
| Pin MPU6050 | Pin ESP32 |
|-------------|-----------|
| SDA | GPIO 21 |
| SCL | GPIO 5 |
| VCC | 3.3V |
| GND | GND |

> **Catatan**: PWM motor menggunakan LEDC (channel 0 untuk ENA, channel 1 untuk ENB) pada frekuensi 1 kHz, resolusi 8-bit.

## Kalibrasi Motor (Hasil Tuning)

Karena karakteristik motor DC kuning tidak sama persis kiri-kanan, PWM dikalibrasi agar robot bergerak lurus:

| Gerakan | Motor Kiri | Motor Kanan |
|---------|-----------|-------------|
| Maju lurus | 255 | 210 |
| Mundur lurus | 255 | 240 |

## Logika Kontrol (State Machine)
[SEARCH] ── lawan di samping ──► [TRACK_LEFT / TRACK_RIGHT]

│                                       │

│                                  sensor tengah lihat

│                                       │

│                                       v

└────── lawan di depan ──────────► [ATTACK]

│

jarak ≤ 12 cm

│

v

[PUSH]

│

7 detik diam

│

v

[ANTI_STUCK]
*Garis terdeteksi (semua state) ──► [AVOID_LINE]  (prioritas tertinggi)


| State | Deskripsi |
|-------|-----------|
| **SEARCH** | "Putar-lirik" - putar ~160ms lalu diam ~70ms supaya ultrasonik sempat baca |
| **TRACK_LEFT / RIGHT** | Pivot di tempat ke arah target sampai sensor tengah ikut melihat |
| **ATTACK** | Maju full PWM (255) saat lawan terlihat di depan |
| **PUSH** | Dorong full PWM saat lawan sangat dekat (≤ 12 cm) |
| **AVOID_LINE** | Stop → mundur/maju → belok menjauh dari garis tepi |
| **ANTI_STUCK** | Mundur + belok lebar saat MPU6050 mendeteksi deadlock |

## Fitur Utama Firmware

- **FreeRTOS multi-task** dual-core ESP32:
  - Core 1: Task IR (prio 5) + Task Robot State Machine (prio 4)
  - Core 0: Task Ultrasonik (prio 2) + Task MPU (prio 1)
- **PID Steering** untuk arah hadap target (Kp=5.5, Kd=1.8)
- **Target Lock System** mencegah robot ragu-ragu pindah-pindah target (lock 250ms)
- **Prioritas absolut** untuk deteksi garis — bahkan saat ANTI_STUCK, garis tetap memotong manuver

## Library yang Digunakan

- `Wire.h` — komunikasi I2C untuk MPU6050 (raw register, tanpa library tambahan)
- `math.h` — operasi `fabs()` untuk deteksi deadlock
- FreeRTOS (built-in ESP32) — multi-task scheduling

## Cara Menjalankan

1. Clone repository ini:
```bash
   git clone https://github.com/bryansasa/Sumo-Robot.git
```
2. Buka project di **Arduino IDE** atau **PlatformIO**
3. Pilih board **ESP32 Dev Module**
4. Sesuaikan wiring sesuai tabel pin mapping di atas
5. Upload kode ke ESP32
6. Buka Serial Monitor (115200 baud) untuk memantau state machine
7. Pasang baterai dan lakukan pengujian di arena dohyo

## Strategi Bertanding

- **Agresif**: target FRONT/CLOSE langsung override lock - maju full PWM tanpa transisi
- **Defensif terhadap garis**: IR memiliki prioritas tertinggi, dicek setiap 2ms
- **Anti-stuck**: MPU6050 + jarak depan mendeteksi dorong-dorongan mandek (timeout 7 detik)
- **Search hemat momentum**: putar-diam-putar supaya tidak kelewatan lawan & tidak kebawa keluar arena

## Tim Pengembang

- Bryan Gabriel Izaac Sasabone
- Muhammad Hafizh Odja
- Arillian Dylan Nugraha

## Lisensi

Project ini dibuat untuk keperluan edukasi dan kompetisi robotika.
