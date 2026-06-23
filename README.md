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
