Mikrotik 

OPTIMALISASI MIKROTIK: METHODE BUDI FASTTRACK-BURST

Konsep Penggabungan FastTrack, Interface List, dan Queue Tree + Burst untuk Koneksi Instan dan CPU Ringan

Penulis: BUDI 
Tahun Rilis: 2026

Target Implementasi: Jaringan RT/RW Net, Hotspot Cafe, dan LAN Kantor (Skala Kecil - Menengah)

1. LATAR BELAKANG & MASALAH KLASIK
Banyak teknisi MikroTik menghadapi dilema besar:

Jika FastTrack Aktif, penggunaan CPU router sangat hemat (adem), namun manajemen bandwidth lewat Queue Tree menjadi lumpuh (bocor).
Jika FastTrack Dimatikan, Queue Tree bekerja dengan baik, namun router mengalami micro-delay (respon telat/berputar di awal) saat client membuka koneksi baru karena beban pemrosesan paket kecil (DNS, SYN/ACK) yang tinggi di CPU. 
Fitur Burst sekalipun sering kali terasa telat merespons akibat antrean pemrosesan ini.

2. SOLUSI & KONSEP CERDAS METODE

Metode ini memecahkan masalah di atas dengan konsep 
"Jalur Bypass Cerdas Berbasis Interface List".
Trafik Pembuka / Sisa (Modem ISP ⇄ Router): 
Diberikan jalur ekspres lewat FastTrack. Ini membebaskan CPU dari beban membaca paket-paket kecil pembuka koneksi. 
Hasilnya, koneksi langsung "tersedia" seketika (instan) bagi client.
Trafik Utama Pelanggan (Hotspot & LAN): Dikecualikan dari FastTrack menggunakan fitur Interface List. 
Paket data yang mulai membesar akan langsung ditangkap oleh Mangle secara akurat, lalu dialirkan ke Queue Tree + Burst.
Pengguna mendapatkan sensasi internet yang sangat responsif di detik pertama, lalu bandwidth tetap terkontrol secara adil di detik berikutnya tanpa membuat router kepanasan.

3. PANDUAN IMPLEMENTASI (KODE SCRIPT)
Salin dan tempel (copy-paste) script di bawah ini ke dalam New Terminal MikroTik Anda.

Langkah A: 
Pembuatan Interface List (Pengelompokan Port Lokal)Script ini menyatukan Port Hotspot dan Port LAN ke dalam satu wadah bernama LOKAL agar mudah dikecualikan.

#========================================================# 

METODE BUDI FASTTRACK-BURST: 

TAHAP 1 (INTERFACE LIST)
 

#========================================================#

/interface list

add name=LOKAL


/interface list member

add interface=ether3-hotspot list=LOKAL

add interface=ether4-lan list=LOKAL

Use code with caution.
(Catatan: Ubah ether3-hotspot dan ether4-lan sesuai nama interface asli di router Anda).

Langkah B: 
Konfigurasi Firewall Filter (Jalur Cepat Selektif)
Script ini mengaktifkan FastTrack untuk semua jalur, KECUALI port yang berada di dalam daftar LOKAL.

# ======================================================# 

METODE [BUDI] FASTTRACK-BURST: 

TAHAP 2 (FIREWALL FILTER)

# PENTING: TARIK KEDUA RULE INI KE URUTAN PALING ATAS (INDEX 0 & 1)

# ======================================================#

/ip firewall filter

add chain=forward action=fasttrack-connection connection-state=established,related \
    
in-interface-list=!LOKAL out-interface-list=!LOKAL \
    
comment="BUDI Method: FastTrack selain Port Hotspot & LAN"

add chain=forward action=accept connection-state=established,related \
    
comment="BUDI Method: Accept koneksi LOKAL untuk Queue Tree"

Use code with caution.


Langkah C: 
Integrasi Mangle & Queue Tree + Burst
Pastikan rule Mangle Anda tetap berjalan normal untuk menandai (packet-mark) aktivitas Browsing, Download, Game, dan Upload. Di dalam menu Queue Tree, 
pastikan Anda mengaktifkan parameter Burst (Burst Limit, Burst Threshold, Burst Time) pada Child Queue untuk mengunci kesempurnaan metode ini.


4. KESIMPULAN & MANFAAT METODE

Dengan menerapkan Metode BUDI FastTrack-Burst, jaringan Anda akan mendapatkan keuntungan berupa:

Zero Latency Delay: Tidak ada lagi jeda tunggu (loading berputar) saat client pertama kali mengklik sebuah tautan atau membuka aplikasi.

CPU Saver: Konsumsi CPU MikroTik tetap berada di angka yang aman karena sebagian beban kerja sudah dipindahkan ke chip FastTrack.



Bandwidth Management 100% Akurat: Queue Tree dan pembagian Burst tetap berjalan mutlak tanpa kebocoran kuota.
