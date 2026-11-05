# Panduan Lengkap Instalasi & Setup FPGA Development (Vivado 2019.1 & EBAZ4205 Zynq-7000) di Ubuntu 24.04 LTS

Dokumen ini berisi panduan komprehensif dari awal (*clean install*) hingga siap pakai untuk pengembangan FPGA berbasis **Xilinx Zynq-7000 SoC (XC7Z010-1CLG400)** seperti board **EBAZ4205** dan board kontroler miner sejenis (misalnya **Antminer S9 Control Board**), menggunakan **Xilinx Vivado 2019.1 + Xilinx SDK** di lingkungan **Ubuntu 24.04 LTS (Noble Numbat)**.

---

## Daftar Isi
1. [Daftar Link Sumber Resmi & Komunitas](#1-daftar-link-sumber-resmi--komunitas)
2. [Spesifikasi Hardware EBAZ4205 Zynq-7000](#2-spesifikasi-hardware-ebaz4205-zynq-7000)
3. [Alasan Memilih Vivado 2019.1](#3-alasan-memilih-vivado-20191)
4. [Persiapan Sistem & Penyesuaian Ubuntu 24.04 LTS](#4-persiapan-sistem--penyesuaian-ubuntu-2404-lts)
5. [Trik Download Cepat & Resume Anti-Gagal (aria2c)](#5-trik-download-cepat--resume-anti-gagal-aria2c)
6. [Langkah-Langkah Instalasi Vivado 2019.1 WebPACK](#6-langkah-langkah-instalasi-vivado-20191-webpack)
7. [Instalasi Driver Kabel JTAG & Udev Rules](#7-instalasi-driver-kabel-jtag--udev-rules)
8. [Pemasangan Board Definition Files EBAZ4205](#8-pemasangan-board-definition-files-ebaz4205)
9. [Konfigurasi Environment & Shortcut Vivado](#9-konfigurasi-environment--shortcut-vivado)
10. [Pinout & Master XDC Constraint File](#10-pinout--master-xdc-constraint-file)
11. [Contoh Proyek Pertama: Verilog Blink LED](#11-contoh-proyek-pertama-verilog-blink-led)
12. [Catatan Modifikasi Fisik Hardware EBAZ4205](#12-catatan-modifikasi-fisik-hardware-ebaz4205)
13. [Troubleshooting & Solusi Masalah Umum (FAQ)](#13-troubleshooting--solusi-masalah-umum-faq)

---

## 1. Daftar Link Sumber Resmi & Komunitas

Simpan daftar tautan penting berikut sebagai rujukan utama:

### A. Portal Download Resmi Xilinx / AMD
* **Halaman Arsip Download Vivado 2019.1**:  
  [AMD/Xilinx Vivado 2019.1 Download Archive](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html)  
  *Pilih bagian: `Vivado™ Design Suite - HLx Editions - 2019.1 - May 29, 2019`*
* **File Installer Offline Rekomendasi (Single-File Tarball ~21.4 GB)**:  
  Nama file: `Xilinx_Vivado_SDK_2019.1_0524_1430.tar.gz`
* **File Web Installer (Ringan ~70 MB)**:  
  Nama file: `Xilinx_Unified_2019.1_0524_1430_Lin64.bin`

### B. Dokumentasi Resmi AMD / Xilinx
* **UG585 - Zynq-7000 SoC Technical Reference Manual (TRM)**:  
  [https://docs.amd.com/v/u/en-US/ug585-Zynq-7000-TRM](https://docs.amd.com/v/u/en-US/ug585-Zynq-7000-TRM)
* **UG973 - Vivado Design Suite User Guide: Release Notes, Installation, and Licensing**:  
  [https://docs.amd.com/r/2019.1-English/ug973-vivado-release-notes-install-license](https://docs.amd.com/r/2019.1-English/ug973-vivado-release-notes-install-license)
* **UG895 - Vivado Design Suite User Guide: System-Level Design Entry**:  
  [https://docs.amd.com/r/2019.1-English/ug895-vivado-system-level-design-entry](https://docs.amd.com/r/2019.1-English/ug895-vivado-system-level-design-entry)

### C. Repositori Komunitas EBAZ4205 & Antminer S9
* **xjtuecho/EBAZ4205 (Referensi Skematik, Altium, KiCad, & Modifikasi)**:  
  [https://github.com/xjtuecho/EBAZ4205](https://github.com/xjtuecho/EBAZ4205)
* **XyleMora/EBAZ4205 (Board Files XML untuk Vivado)**:  
  [https://github.com/XyleMora/EBAZ4205](https://github.com/XyleMora/EBAZ4205)
* **tomorrow56/EBAZ4205_tutorial (Tutorial Lengkap & Merged XDC)**:  
  [https://github.com/tomorrow56/EBAZ4205_tutorial](https://github.com/tomorrow56/EBAZ4205_tutorial)
* **Muhammad-Yunus/EBAZ4205_PROJECT (Desain Vivado, PYNQ, & Addon PCB)**:  
  [https://github.com/Muhammad-Yunus/EBAZ4205_PROJECT](https://github.com/Muhammad-Yunus/EBAZ4205_PROJECT)
* **xjtuecho/Antminer_s9_zynq (Referensi Board Zynq C5 Antminer S9)**:  
  [https://github.com/xjtuecho/Antminer_s9_zynq](https://github.com/xjtuecho/Antminer_s9_zynq)

### D. Paket Library Legacy Debian / Ubuntu
* **`libtinfo5` amd64**:  
  [http://mirrors.kernel.org/ubuntu/pool/universe/n/ncurses/libtinfo5_6.4-2_amd64.deb](http://mirrors.kernel.org/ubuntu/pool/universe/n/ncurses/libtinfo5_6.4-2_amd64.deb)
* **`libncurses5` amd64**:  
  [http://mirrors.kernel.org/ubuntu/pool/universe/n/ncurses/libncurses5_6.4-2_amd64.deb](http://mirrors.kernel.org/ubuntu/pool/universe/n/ncurses/libncurses5_6.4-2_amd64.deb)

---

## 2. Spesifikasi Hardware EBAZ4205 Zynq-7000

Board EBAZ4205 awalnya diproduksi sebagai papan kontroler mesin miner *Ebang Ebit E9+*, yang kemudian dialihfungsikan oleh komunitas embedded & FPGA menjadi dev board Zynq berbiaya sangat terjangkau:

* **SoC FPGA**: Xilinx Zynq-7000 `XC7Z010-1CLG400`
  * **PS (Processing System)**: Dual-core ARM Cortex-A9 MPCore @ 667 MHz
  * **PL (Programmable Logic)**: Setara Artix-7 FPGA (28.000 Logic Cells, 240 KB Block RAM, 80 DSP Slices)
* **RAM**: 256 MB DDR3 SDRAM (Micron `MT41K128M16 JT-125`, 16-bit wide bus)
* **Flash On-Board**: 128 MB SLC NAND Flash (Spansion S34ML01G100TFI00)
* **Storage Eksternal**: Slot MicroSD Card (dapat diaktifkan via modifikasi boot resistor)
* **Ethernet**: 10/100 Mbps Ethernet PHY (ICPlus IP101G atau RTL8211E) terhubung melalui EMIO PS
* **Clock Oscillator**: 33.333333 MHz kristal aktif terhubung ke pin FPGA `N18`
* **Indikator & Tombol On-board**:
  * LED Hijau (LED6): Pin `W13` (*Active LOW*)
  * LED Merah (LED5): Pin `W14` (*Active LOW*)
  * Push Button (S2): Pin `V13`
* **Expansion Headers (3x 20-Pin, 2.0 mm Pitch)**:
  * **DATA1**: I/O Bank 34 (LVCMOS33)
  * **DATA2**: I/O Bank 35 (LVCMOS33)
  * **DATA3**: I/O Bank 34 & 35 (LVCMOS33)
* **UART Header (J7)**:
  * Terhubung ke Zynq PS UART1 (MIO 24 = TX, MIO 25 = RX) pada logic level 3.3V.

---

## 3. Alasan Memilih Vivado 2019.1

Banyak pengguna pemula bingung memilih antara Vivado versi terbaru (2020.x, 2022.x, 2024.x) vs Vivado 2019.1. Untuk keluarga **Zynq-7000** (khususnya board modifikasi seperti EBAZ4205):

1. **Xilinx SDK Bawaan (Eclipse-Based)**:  
   Versi **2019.1 adalah rilis resmi terakhir** yang menyertakan **Xilinx SDK**. Mulai versi 2019.2 ke atas, Xilinx SDK dihapus dan digantikan oleh *Vitis Unified Platform*. Vitis jauh lebih kompleks, jauh lebih berat, dan sering mengalami masalah saat meng-handle BSP baremetal custom board non-standar.
2. **Kompabilitas Proyek Open-Source Komunitas**:  
   Seluruh repository populer EBAZ4205 (seperti `xjtuecho`, `tomorrow56`, u-boot-xlnx 2019.1, PetaLinux 2019.1) dibangun menggunakan format Vivado 2019.1 (`.hdf` hardware description file). Format `.xsa` di Vitis/Vivado baru memecah kompatibilitas BSP lama ini.
3. **Gratis Tanpa Lisensi Tambahan (WebPACK)**:  
   Chip Zynq `XC7Z010` didukung penuh pada edisi **Vivado HL WebPACK** tanpa perlu membeli lisensi atau memasang license file berbayar.
4. **Ukuran Lebih Hemat**:  
   Installer Vivado 2019.1 WebPACK (~21 GB arsip, ~30 GB terinstal untuk Zynq saja) jauh lebih ringan dibandingkan Vivado ML terbaru yang membutuhkan ruang disk hingga >100 GB.

---

## 4. Persiapan Sistem & Penyesuaian Ubuntu 24.04 LTS

Ubuntu 24.04 LTS (Noble Numbat) dirilis jauh setelah Vivado 2019.1 (yang aslinya dirancang untuk Ubuntu 16.04/18.04). Oleh karena itu, diperlukan 4 penyesuaian wajib sebelum instalasi:

### A. Alokasi Disk Space & Symlink Direktori Instalasi
Installer Vivado secara default ingin memasang ke direktori `/tools/Xilinx` atau `/opt/Xilinx`. Pada laptop dengan partisi root (`/`) terbatas (misalnya di bawah 50 GB), proses ekstraksi akan gagal di tengah jalan akibat disk penuh.

**Solusi:** Arahkan `/tools/Xilinx` ke partisi `/home` (yang memiliki ruang lega) menggunakan *symbolic link*:
```bash
# Buat direktori di home
mkdir -p "$HOME/Xilinx"

# Buat parent directory /tools dan symlink ke home
sudo mkdir -p /tools
sudo rm -rf /tools/Xilinx
sudo ln -s "$HOME/Xilinx" /tools/Xilinx
```

### B. Dependensi Paket Sistem & Build Tools
Pasang perkakas kompilasi, pustaka grafis GTK/OpenGL, emulator serial, dan utilitas pendukung:
```bash
sudo apt update
sudo apt install -y \
    build-essential \
    gcc \
    g++ \
    make \
    gawk \
    libgl1 \
    libglu1-mesa \
    libxi6 \
    libxrender1 \
    libxtst6 \
    libcanberra-gtk-module \
    libcanberra-gtk3-module \
    libjpeg62 \
    zlib1g \
    zlib1g-dev \
    net-tools \
    fxload \
    libtinfo6 \
    libncurses6 \
    libncursesw6 \
    picocom \
    minicom \
    openocd \
    aria2 \
    python3 \
    python3-pip \
    python3-serial
```

### C. Penanganan Masalah Library Deprecated (`libtinfo5` & `libncurses5`)
Vivado 2019.1 dan komponen Java/ELF loader-nya membutuhkan `libtinfo.so.5` dan `libncurses.so.5`. Repositori Ubuntu 24.04 hanya menyertakan versi 6, sehingga jika tidak ditangani, installer atau Vivado akan crash/gagal membuka GUI.

**Cara Memasang Library Versi 5:**
```bash
# Unduh paket deb legacy
cd /tmp
wget http://mirrors.kernel.org/ubuntu/pool/universe/n/ncurses/libtinfo5_6.4-2_amd64.deb
wget http://mirrors.kernel.org/ubuntu/pool/universe/n/ncurses/libncurses5_6.4-2_amd64.deb

# Pasang paket
sudo dpkg -i libtinfo5_6.4-2_amd64.deb libncurses5_6.4-2_amd64.deb || sudo apt install -f -y

# Bersihkan file temporer
rm -f libtinfo5_6.4-2_amd64.deb libncurses5_6.4-2_amd64.deb
```

*Jika karena alasan tertentu paket deb di atas tidak dapat diinstal, buat symlink fallback berikut:*
```bash
sudo ln -sf /usr/lib/x86_64-linux-gnu/libtinfo.so.6 /usr/lib/x86_64-linux-gnu/libtinfo.so.5
sudo ln -sf /usr/lib/x86_64-linux-gnu/libncurses.so.6 /usr/lib/x86_64-linux-gnu/libncurses.so.5
```

### D. Mengubah Shell Default `/bin/sh` ke `bash`
Secara default di Ubuntu, `/bin/sh` adalah symlink ke `dash`. Skrip automasi build, Makefiles, dan generator IP core Xilinx Vivado menggunakan sintaks Bash khusus (*bashisms*). Menjalankannya dengan `dash` akan menimbulkan error acak saat sintesis atau kompilasi BSP SDK.

Jalankan perintah ini:
```bash
sudo ln -sf bash /bin/sh
```
Verifikasi:
```bash
ls -l /bin/sh
# Output harus menunjukkan: /bin/sh -> bash
```

---

## 5. Trik Download Cepat & Resume Anti-Gagal (aria2c)

Ukuran installer Vivado 2019.1 Single-File adalah **~21.4 GB**. Jika mengunduh langsung melalui web browser (Brave / Chrome / Firefox), unduhan rentan mengalami:
- Kecepatan dibatasi (*throttling*) oleh browser single-thread.
- Gagal di tengah jalan jika jaringan Wi-Fi/koneksi sempat terputus.

Gunakan **`aria2c`** untuk mengunduh dengan **16 koneksi paralel simultan** dan fitur **resume otomatis**:

### Prosedur Mendapatkan Link Download AMD:
1. Buka browser dan kunjungi:  
   [AMD Vivado 2019.1 Archive](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html)
2. Klik bagian: **`Vivado™ Design Suite - HLx Editions - 2019.1 - May 29, 2019`**.
3. Klik tombol **Download** pada baris:  
   `Vivado HLx 2019.1: All OS installer Single-File Download` (~21.4 GB).
4. Login menggunakan akun AMD/Xilinx Anda, isi formulir verifikasi alamat ekspor (*Export Compliance Verification*), lalu klik **Submit / Download**.
5. Saat jendela popup browser "Save As" muncul, klik **Save**.
6. Segera tekan **`Ctrl + J`** di browser untuk membuka tab **Downloads**.
7. Klik tombol **Pause** atau **Cancel** pada file yang mulai terdownload tersebut (agar browser tidak berebut bandwidth).
8. **Klik kanan** pada nama file tersebut $\rightarrow$ pilih **"Copy link address"** (Salin alamat tautan).

### Eksekusi Download di Terminal:
Jalankan perintah berikut di terminal (ganti `LINK_URL_HASIL_COPY` dengan link yang Anda salin; pastikan diapit tanda petik dua `"..."`):

```bash
aria2c -c -x 16 -s 16 -k 1M -d ~/Downloads "LINK_URL_HASIL_COPY"
```

> [!TIP]
> **Anti-Gagal**: Jika laptop dimatikan, koneksi putus, atau kuota habis, Anda tidak perlu mengulang dari 0%. Cukup jalankan perintah yang sama persis di terminal, dan `aria2c` akan secara otomatis melanjutkan (*resume*) dari persentase terakhir.

---

## 6. Langkah-Langkah Instalasi Vivado 2019.1 WebPACK

Setelah proses unduhan file `Xilinx_Vivado_SDK_2019.1_0524_1430.tar.gz` selesai 100% di folder `~/Downloads`:

### A. Ekstraksi File Installer
```bash
cd ~/Downloads
tar -xzf Xilinx_Vivado_SDK_2019.1_0524_1430.tar.gz
cd Xilinx_Vivado_SDK_2019.1_0524_1430
```

### B. Menjalankan Setup dengan Locale Aman
Vivado berbasis antarmuka grafis Java yang dapat bermasalah jika lingkungan sistem menggunakan format locale yang tidak dikenali:
```bash
export LC_ALL=C.UTF-8
export LANG=C.UTF-8
chmod +x xsetup
./xsetup
```

### C. Opsi Pengaturan pada Wizard Instalasi GUI

Ikuti langkah-langkah wizard berikut dengan cermat untuk menghemat ruang disk hingga puluhan Gigabyte:

1. **Welcome**: Klik **Next**.
2. **Accept License Agreements**: Centang ketiga kotak persetujuan (*I Agree*), lalu klik **Next**.
3. **Select Product to Install**:
   * Pilih opsi: **Vivado HL Design Edition** atau **Vivado HL WebPACK**.
   * *(Catatan: Keduanya sama-sama gratis untuk chip Zynq XC7Z010, namun WebPACK adalah profil default tanpa lisensi berbayar)*.
   * Klik **Next**.
4. **Customize Installation (PENTING — HEMAT DISK SPACE)**:
   * [x] **Vivado Design Suite**: Centang.
   * [x] **Software Development Kit (SDK)**: **WAJIB DICENTANG** (ini adalah IDE untuk program C/C++ ARM Cortex-A9).
   * [x] **DocNav**: Opsional (boleh dihilangkan centangnya jika ingin hemat 1 GB).
   * **Devices Tree**:
     * [ ] Hilangkan centang pada: **Production Devices -> UltraScale+**
     * [ ] Hilangkan centang pada: **Production Devices -> UltraScale**
     * [ ] Hilangkan centang pada: **Production Devices -> Kintex-7**
     * [ ] Hilangkan centang pada: **Production Devices -> Virtex-7**
     * [x] **BIARKAN TERCENTANG HANYA**: **Production Devices -> 7 Series -> Zynq-7000**.
     *(Menghilangkan keluarga FPGA besar di atas akan menghemat sekitar 25–30 GB kapasitas penyimpanan!)*
   * Klik **Next**.
5. **Select Destination Directory**:
   * Installation Directory: Isi dengan `/tools/Xilinx` (atau `/home/maruf/Xilinx`).
   * *(Karena `/tools/Xilinx` sudah kita symlink ke `/home/maruf/Xilinx`, seluruh file akan disimpan dengan aman di partisi `/home`)*.
   * Klik **Next**.
6. **Installation Summary**: Periksa ringkasan, lalu klik **Install**.
7. Tunggu hingga proses instalasi dan verifikasi checksum selesai (biasanya memakan waktu 15–30 menit tergantung performa SSD).

---

## 7. Instalasi Driver Kabel JTAG & Udev Rules

Agar Vivado Hardware Manager dan Xilinx SDK dapat memprogram FPGA dan men-debug prosesor ARM melalui kabel JTAG (seperti Digilent JTAG-HS2/HS3, Platform Cable USB clone, atau antarmuka FT2232H/FT232H) tanpa membutuhkan akses root/sudo:

### A. Pasang Driver Kabel Resmi Xilinx
```bash
cd /tools/Xilinx/Vivado/2019.1/data/xinstall/cntl
sudo ./install_drivers.sh
```

### B. Pasang Udev Rules Tambahan untuk JTAG & Serial Console
Buat file udev rules khusus di sistem:
```bash
sudo tee /etc/udev/rules.d/52-xilinx-ftdi-digilent.rules > /dev/null << 'EOF'
# Digilent JTAG cables (JTAG-HS1, HS2, HS3, SMT2, etc.)
ATTRS{idVendor}=="1443", MODE="0666", GROUP="dialout"
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6010", MODE="0666", GROUP="dialout"

# FTDI FT232H / FT2232H / FT4232H (Custom OpenOCD / Digilent compatible)
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6014", MODE="0666", GROUP="dialout"
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6011", MODE="0666", GROUP="dialout"
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", MODE="0666", GROUP="dialout"

# Xilinx Platform Cable USB (DLC9, DLC10, and clones)
ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0008", MODE="0666"
ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0007", MODE="0666"
ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0009", MODE="0666"
ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="000d", MODE="0666"
ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0013", MODE="0666"
ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0015", MODE="0666"

# Cypress FX2 LP (used by unprogrammed Platform Cable USB clones)
ATTRS{idVendor}=="04b4", ATTRS{idProduct}=="8613", MODE="0666"

# USB UART Adapters (CP2102, CH340, PL2303, FTDI) for Serial Terminal Console
KERNEL=="ttyUSB[0-9]*", MODE="0666", GROUP="dialout"
KERNEL=="ttyACM[0-9]*", MODE="0666", GROUP="dialout"
EOF
```

### C. Masukkan User ke Group `dialout` & `plugdev`
```bash
sudo usermod -aG dialout,plugdev $USER
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## 8. Pemasangan Board Definition Files EBAZ4205

Secara bawaan (*out of the box*), Vivado hanya mengenali papan resmi Xilinx (seperti ZC702, ZedBoard, KC705). Board EBAZ4205 adalah perangkat *third-party*, sehingga kita perlu menambahkan folder definisinya agar saat membuat Block Design (Zynq PS), timing DDR3, UART, Flash, dan clock terkonfigurasi secara otomatis dalam 1 klik.

### Struktur File Board Definition EBAZ4205:
Folder `ebaz4205/1.0/` terdiri dari:
- `board.xml`: Mendefinisikan nama board, part number `xc7z010clg400-1`, dan komponen interface.
- `preset.xml`: Mengonfigurasi parameter internal Zynq Processing System (Zynq PS7):
  - RAM DDR3: Micron `MT41K128M16 JT-125` (16-bit bus).
  - UART1: MIO pin 24..25.
  - Storage: NAND Flash MIO 0..14 dan SD0 MIO 40..45.
  - Ethernet: ENET0 via EMIO.
- `part0_pins.xml`: Pemetaan pin fisik untuk LED (LED5 & LED6).
- `ebaz4205.jpg`: Ikon preview visual board di antarmuka Vivado.

### Perintah Pemasangan Board Files:
Jalankan perintah ini untuk menyalin file definisi board ke direktori instalasi Vivado:

```bash
# Buat folder board di Vivado
mkdir -p /tools/Xilinx/Vivado/2019.1/data/boards/board_files/ebaz4205/1.0

# Salin dari repositori XyleMora (atau workspace lokal)
cp -r ~/.gemini/antigravity-ide/scratch/ebaz4205-workspace/board_files/ebaz4205/1.0/* \
      /tools/Xilinx/Vivado/2019.1/data/boards/board_files/ebaz4205/1.0/
```

*Jika Anda mengunduh langsung dari GitHub:*
```bash
git clone --depth 1 https://github.com/XyleMora/EBAZ4205.git /tmp/XyleMora_EBAZ
mkdir -p /tools/Xilinx/Vivado/2019.1/data/boards/board_files/ebaz4205/1.0
cp -r "/tmp/XyleMora_EBAZ/Documents/Board files/ebaz4205/1.0/"* /tools/Xilinx/Vivado/2019.1/data/boards/board_files/ebaz4205/1.0/
rm -rf /tmp/XyleMora_EBAZ
```

Setelah langkah ini selesai, setiap kali Anda membuat proyek baru di Vivado dan memilih tab **Boards**, papan **"EBAZ4205 Development Board"** akan langsung muncul dalam daftar pilihan.

---

## 9. Konfigurasi Environment & Shortcut Vivado

Untuk menjalankan Vivado dan Xilinx SDK dari terminal manapun tanpa harus mengetik path lengkap:

### A. Tambahkan Alias ke `~/.bashrc`
Buka file `~/.bashrc`:
```bash
echo '' >> ~/.bashrc
echo '# Xilinx Vivado & SDK 2019.1 Environment' >> ~/.bashrc
echo 'alias vivado_env="source /tools/Xilinx/Vivado/2019.1/settings64.sh && export LC_ALL=C.UTF-8 && export LANG=C.UTF-8"' >> ~/.bashrc
echo 'alias vivado="vivado_env && vivado"' >> ~/.bashrc
echo 'alias xsdk="vivado_env && xsdk"' >> ~/.bashrc
```
Muat ulang konfigurasi:
```bash
source ~/.bashrc
```

### B. Membuat Desktop Launcher (Menu Aplikasi Ubuntu)
Buat file shortcut agar Vivado bisa dibuka langsung dari App Grid Ubuntu:
```bash
mkdir -p ~/.local/share/applications
cat << 'EOF' > ~/.local/share/applications/vivado2019.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=Vivado 2019.1
Comment=Xilinx Vivado Design Suite
Exec=bash -c "source /tools/Xilinx/Vivado/2019.1/settings64.sh && export LC_ALL=C.UTF-8 && vivado"
Icon=/tools/Xilinx/Vivado/2019.1/doc/images/vivado_logo.png
Terminal=false
Categories=Development;Electronics;
StartupWMClass=Vivado
EOF

chmod +x ~/.local/share/applications/vivado2019.desktop
update-desktop-database ~/.local/share/applications/
```

---

## 10. Pinout & Master XDC Constraint File

Simpan file constraint berikut sebagai referensi pemetaan pin I/O saat membuat project Vivado untuk EBAZ4205:

File: `ebaz4205_master.xdc`
```tcl
# ==============================================================================
# Master Constraint File (XDC) for EBAZ4205 (Xilinx Zynq XC7Z010-1CLG400)
# ==============================================================================

# ------------------------------------------------------------------------------
# 1. On-Board Oscillator Clock (33.333 MHz)
# ------------------------------------------------------------------------------
set_property PACKAGE_PIN N18 [get_ports sys_clk]
set_property IOSTANDARD LVCMOS33 [get_ports sys_clk]
create_clock -period 30.000 -name sys_clk_pin -waveform {0.000 15.000} -add [get_ports sys_clk]

# ------------------------------------------------------------------------------
# 2. On-Board User LEDs (Active LOW: 0 = Menyala, 1 = Mati)
# ------------------------------------------------------------------------------
# LED Hijau (LED6)
set_property PACKAGE_PIN W13 [get_ports led_green]
set_property IOSTANDARD LVCMOS33 [get_ports led_green]

# LED Merah (LED5)
set_property PACKAGE_PIN W14 [get_ports led_red]
set_property IOSTANDARD LVCMOS33 [get_ports led_red]

# ------------------------------------------------------------------------------
# 3. On-Board Push Button (S2)
# ------------------------------------------------------------------------------
set_property PACKAGE_PIN V13 [get_ports btn_s2]
set_property IOSTANDARD LVCMOS33 [get_ports btn_s2]

# ------------------------------------------------------------------------------
# 4. Expansion Header DATA1 (Bank 34, LVCMOS33)
# ------------------------------------------------------------------------------
# set_property -dict {PACKAGE_PIN A20 IOSTANDARD LVCMOS33} [get_ports {data1[3]}]
# set_property -dict {PACKAGE_PIN H16 IOSTANDARD LVCMOS33} [get_ports {data1[4]}]
# set_property -dict {PACKAGE_PIN B19 IOSTANDARD LVCMOS33} [get_ports {data1[5]}]
# set_property -dict {PACKAGE_PIN B20 IOSTANDARD LVCMOS33} [get_ports {data1[6]}]
# set_property -dict {PACKAGE_PIN C20 IOSTANDARD LVCMOS33} [get_ports {data1[7]}]
# set_property -dict {PACKAGE_PIN H17 IOSTANDARD LVCMOS33} [get_ports {data1[8]}]
# set_property -dict {PACKAGE_PIN D20 IOSTANDARD LVCMOS33} [get_ports {data1[9]}]
# set_property -dict {PACKAGE_PIN D18 IOSTANDARD LVCMOS33} [get_ports {data1[10]}]
# set_property -dict {PACKAGE_PIN H18 IOSTANDARD LVCMOS33} [get_ports {data1[11]}]
# set_property -dict {PACKAGE_PIN D19 IOSTANDARD LVCMOS33} [get_ports {data1[12]}]
# set_property -dict {PACKAGE_PIN F20 IOSTANDARD LVCMOS33} [get_ports {data1[13]}]
# set_property -dict {PACKAGE_PIN E19 IOSTANDARD LVCMOS33} [get_ports {data1[14]}]
# set_property -dict {PACKAGE_PIN F19 IOSTANDARD LVCMOS33} [get_ports {data1[15]}]
# set_property -dict {PACKAGE_PIN K17 IOSTANDARD LVCMOS33} [get_ports {data1[16]}]

# ------------------------------------------------------------------------------
# 5. Expansion Header DATA2 (Bank 35, LVCMOS33)
# ------------------------------------------------------------------------------
# set_property -dict {PACKAGE_PIN G20 IOSTANDARD LVCMOS33} [get_ports {data2[3]}]
# set_property -dict {PACKAGE_PIN J18 IOSTANDARD LVCMOS33} [get_ports {data2[4]}]
# set_property -dict {PACKAGE_PIN G19 IOSTANDARD LVCMOS33} [get_ports {data2[5]}]
# set_property -dict {PACKAGE_PIN H20 IOSTANDARD LVCMOS33} [get_ports {data2[6]}]
# set_property -dict {PACKAGE_PIN J19 IOSTANDARD LVCMOS33} [get_ports {data2[7]}]
# set_property -dict {PACKAGE_PIN K18 IOSTANDARD LVCMOS33} [get_ports {data2[8]}]
# set_property -dict {PACKAGE_PIN K19 IOSTANDARD LVCMOS33} [get_ports {data2[9]}]
# set_property -dict {PACKAGE_PIN J20 IOSTANDARD LVCMOS33} [get_ports {data2[10]}]
# set_property -dict {PACKAGE_PIN L16 IOSTANDARD LVCMOS33} [get_ports {data2[11]}]
# set_property -dict {PACKAGE_PIN L19 IOSTANDARD LVCMOS33} [get_ports {data2[12]}]
# set_property -dict {PACKAGE_PIN M19 IOSTANDARD LVCMOS33} [get_ports {data2[13]}]
# set_property -dict {PACKAGE_PIN L20 IOSTANDARD LVCMOS33} [get_ports {data2[14]}]
# set_property -dict {PACKAGE_PIN M20 IOSTANDARD LVCMOS33} [get_ports {data2[15]}]
# set_property -dict {PACKAGE_PIN L17 IOSTANDARD LVCMOS33} [get_ports {data2[16]}]

# ------------------------------------------------------------------------------
# 6. Expansion Header DATA3 (Bank 34 & 35, LVCMOS33)
# ------------------------------------------------------------------------------
# set_property -dict {PACKAGE_PIN M17 IOSTANDARD LVCMOS33} [get_ports {data3[3]}]
# set_property -dict {PACKAGE_PIN M18 IOSTANDARD LVCMOS33} [get_ports {data3[4]}]
# set_property -dict {PACKAGE_PIN N19 IOSTANDARD LVCMOS33} [get_ports {data3[5]}]
# set_property -dict {PACKAGE_PIN N20 IOSTANDARD LVCMOS33} [get_ports {data3[6]}]
# set_property -dict {PACKAGE_PIN P18 IOSTANDARD LVCMOS33} [get_ports {data3[7]}]
# set_property -dict {PACKAGE_PIN R18 IOSTANDARD LVCMOS33} [get_ports {data3[9]}]
# set_property -dict {PACKAGE_PIN R19 IOSTANDARD LVCMOS33} [get_ports {data3[10]}]
# set_property -dict {PACKAGE_PIN P20 IOSTANDARD LVCMOS33} [get_ports {data3[11]}]
# set_property -dict {PACKAGE_PIN N17 IOSTANDARD LVCMOS33} [get_ports {data3[12]}]
# set_property -dict {PACKAGE_PIN U20 IOSTANDARD LVCMOS33} [get_ports {data3[13]}]
# set_property -dict {PACKAGE_PIN T20 IOSTANDARD LVCMOS33} [get_ports {data3[14]}]
# set_property -dict {PACKAGE_PIN T19 IOSTANDARD LVCMOS33} [get_ports {data3[15]}]
# set_property -dict {PACKAGE_PIN V20 IOSTANDARD LVCMOS33} [get_ports {data3[16]}]
# set_property -dict {PACKAGE_PIN U19 IOSTANDARD LVCMOS33} [get_ports {data3[17]}]
# set_property -dict {PACKAGE_PIN P19 IOSTANDARD LVCMOS33} [get_ports {data3[18]}]
```

---

## 11. Contoh Proyek Pertama: Verilog Blink LED

Untuk menguji bahwa Vivado, synthesizer, implementasi, dan programmer JTAG berfungsi sempurna, buat modul Verilog sederhana berikut:

File: `ebaz4205_blink_top.v`
```verilog
`timescale 1ns / 1ps
// ==============================================================================
// EBAZ4205 Verilog LED Blink & Button Test
// FPGA: Zynq-7000 (XC7Z010-1CLG400)
// Clock: 33.333 MHz pada PIN N18
// LED Hijau (LED6): W13 (Active LOW)
// LED Merah (LED5): W14 (Active LOW)
// Tombol S2: V13
// ==============================================================================

module ebaz4205_blink_top (
    input  wire sys_clk,    // 33.333 MHz clock dari pin N18
    input  wire btn_s2,     // Push button S2 dari pin V13
    output wire led_green,  // LED6 (Hijau) pada pin W13
    output wire led_red     // LED5 (Merah) pada pin W14
);

    // 33.333 MHz -> 1 detik = 33.333.333 siklus clock
    // Toggle setiap 0.5 detik -> 16.666.666 siklus clock
    localparam CNT_HALF_SEC = 32'd16_666_666;

    reg [31:0] counter = 32'd0;
    reg        blink_state = 1'b0;

    always @(posedge sys_clk) begin
        if (counter >= CNT_HALF_SEC - 1) begin
            counter     <= 32'd0;
            blink_state <= ~blink_state;
        end else begin
            counter     <= counter + 1'b1;
        end
    end

    // Catatan: LED pada EBAZ4205 adalah Active LOW:
    // Logika 0 = LED MENYALA, Logika 1 = LED MATI
    assign led_green = blink_state ? 1'b0 : 1'b1;
    assign led_red   = btn_s2 ? ~blink_state : 1'b0;

endmodule
```

### Langkah Uji di Vivado:
1. Buka Vivado $\rightarrow$ **Create Project** $\rightarrow$ beri nama proyek.
2. Di bagian **Default Part**:
   - Jika sudah memasang board files: Klik tab **Boards** $\rightarrow$ pilih **EBAZ4205 Development Board**.
   - Atau via Parts tab: Pilih part **`xc7z010clg400-1`**.
3. Tambahkan source file `ebaz4205_blink_top.v` dan constraint file `ebaz4205_master.xdc`.
4. Klik **Generate Bitstream** di panel kiri bawah (*Flow Navigator*).
5. Sambungkan programmer JTAG ke header JTAG board EBAZ4205.
6. Buka **Open Hardware Manager** $\rightarrow$ **Auto Connect**.
7. Klik kanan pada `xc7z010_1` $\rightarrow$ **Program Device** $\rightarrow$ pilih file `.bit` yang baru selesai di-build.
8. **Hasil**: LED hijau pada board akan berkedip stabil 1 Hz!

---

## 12. Catatan Modifikasi Fisik Hardware EBAZ4205

Karena EBAZ4205 aslinya dirancang untuk mesin miner, beberapa modifikasi perangkat keras umum diperlukan:

1. **Modifikasi Power Supply (Dioda D24)**:
   - Konektor power asli board dirancang untuk tegangan **12V DC**.
   - Sebagian besar developer menyuplai daya **5V DC** melalui pin 1 & 2 pada header ekspansi `DATA3`.
   - **Perhatian**: Di dekat konektor power terdapat dioda schottky **`D24`**. Drop tegangan melalui dioda ini (~0.4V) dapat menyebabkan regulator 3.3V, 1.8V, dan 1.0V kekurangan tegangan input jika suplai 5V drop di bawah 4.6V.  
     *Solusi umum*: Jumper/bypass kedua kaki dioda D24 jika Anda selalu menggunakan suplai 5V DC murni.
2. **Modifikasi Boot Mode Resistor (SD Card Boot)**:
   - Secara pabrik, EBAZ4205 diatur untuk boot dari chip NAND Flash (`boot mode = NAND`).
   - Agar board dapat mem-boot sistem Linux atau FSBL dari kartu MicroSD, ubah posisi resistor boot strap:
     - Lepas resistor SMD **`R2584`** (resistor 10k ohm yang menghubungkan pin boot ke GND).
     - Pasang resistor tersebut ke posisi **`R2585`** (menghubungkan pin boot ke 3.3V VCC).
     - *(Alternatif: Pasang switch toggle kecil pada kedua posisi resistor tersebut untuk memilih NAND boot vs SD boot)*.
3. **Koneksi Serial UART Console (Header J7)**:
   - Header J7 terdiri dari 4 pin:
     - Pin 1: GND
     - Pin 2: UART TX (terhubung ke Zynq MIO 24)
     - Pin 3: UART RX (terhubung ke Zynq MIO 25)
     - Pin 4: VCC 3.3V (jangan sambungkan ke USB-UART jika USB-UART sudah punya daya sendiri)
   - Sambungkan adapter USB-to-UART (CP2102 / CH340 / FTDI):
     - GND adapter $\rightarrow$ GND J7
     - RX adapter $\rightarrow$ TX J7 (Pin 2)
     - TX adapter $\rightarrow$ RX J7 (Pin 3)
   - Buka serial console di Linux dengan baudrate **115200**:
     ```bash
     picocom -b 115200 /dev/ttyUSB0
     ```

---

## 13. Troubleshooting & Solusi Masalah Umum (FAQ)

### 1. Error: `libtinfo.so.5: cannot open shared object file: No such file or directory`
* **Penyebab**: Komponen Vivado mencari shared library terminal versi 5 yang tidak ada secara default di Ubuntu 24.04.
* **Solusi**: Pastikan Anda sudah menginstal paket `libtinfo5_6.4-2_amd64.deb` atau membuat symlink:
  ```bash
  sudo ln -sf /usr/lib/x86_64-linux-gnu/libtinfo.so.6 /usr/lib/x86_64-linux-gnu/libtinfo.so.5
  sudo ln -sf /usr/lib/x86_64-linux-gnu/libncurses.so.6 /usr/lib/x86_64-linux-gnu/libncurses.so.5
  ```

### 2. Error: GUI Vivado membeku (*hang/freeze*) atau teks tidak terbaca
* **Penyebab**: Glitch rendering Java Swing pada display server Wayland atau locale sistem.
* **Solusi**: Selalu jalankan dengan locale C / UTF-8 dan fallback driver X11 jika menggunakan Wayland:
  ```bash
  export LC_ALL=C.UTF-8
  export LANG=C.UTF-8
  export _JAVA_AWT_WM_NONREPARENTING=1
  vivado
  ```

### 3. Hardware Manager: Cable Driver Not Found / Permission Denied
* **Penyebab**: User akun Anda belum masuk ke group `dialout`, atau aturan udev belum dimuat ulang.
* **Solusi**:
  ```bash
  sudo usermod -aG dialout,plugdev $USER
  sudo udevadm control --reload-rules
  sudo udevadm trigger
  ```
  *Lakukan logout sesi Ubuntu atau restart PC agar hak akses group diperbarui secara penuh.*

### 4. Error saat Export Hardware ke Xilinx SDK
* **Solusi**: Pastikan selalu mencentang opsi **"Include bitstream"** pada jendela dialog `File -> Export -> Export Hardware`. Jika bitstream tidak disertakan, Anda tidak dapat memprogram logika PL FPGA dari dalam Xilinx SDK.

---
*Dokumen ini disusun untuk Maruf (ThinkPad T470p) — Antigravity FPGA Development Environment.*

