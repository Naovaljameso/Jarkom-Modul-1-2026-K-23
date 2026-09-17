# Praktikum Modul 1 Komunikasi Data & Jaringan Komputer 2026

## Jarkom K-23

| Nama                | NRP        |
| ------------------- | ---------- |
| Naoval James Osamah | 5027251092 |
| Nur Rizki Syahbana  | 5027251095 |

## K-23 - _The Wired_

Dokumentasi praktikum ini memuat pembangunan jaringan **The Wired** menggunakan GNS3, konfigurasi routing dan akses internet, layanan FTP/Telnet/SSH, serta analisis paket menggunakan Wireshark.

> **Cakupan dokumentasi:** bukti yang tersedia pada folder `assets` mencakup pengerjaan nomor **1-13**. Nomor 14-20 memerlukan hasil analisis file PCAP dan belum memiliki bukti pada berkas yang tersedia, sehingga hasilnya tidak dibuat-buat dalam README ini.

## Daftar Isi

- [Lingkungan Praktikum](#lingkungan-praktikum)
- [Topologi dan Alokasi IP](#topologi-dan-alokasi-ip)
- [Ringkasan Hasil](#ringkasan-hasil)
- [Dokumentasi Pengerjaan](#dokumentasi-pengerjaan)
- [Status Nomor 14-20](#status-nomor-14-20)
- [Kesimpulan](#kesimpulan)

## Lingkungan Praktikum

| Komponen              | Keterangan                            |
| --------------------- | ------------------------------------- |
| Simulator jaringan    | GNS3                                  |
| Router                | Lain                                  |
| Client                | Alice, Mika, Chisa, Knights, dan Eiri |
| Analisis paket        | Wireshark                             |
| FTP server            | vsFTPd 3.0.5 pada Chisa               |
| Remote administration | Telnet dan OpenSSH                    |
| Prefix kelompok       | `10.75.x.x`                           |

Project GNS3 telah menyimpan konfigurasi interface dan script setup di dalam masing-masing node. Script utama yang tersedia adalah:

- `/root/setup_lain.sh`
- `/root/setup_alice.sh`
- `/root/setup_mika.sh`
- `/root/setup_chisa.sh`
- `/root/setup_knights.sh`
- `/root/setup_eiri.sh`
- `/root/setup_telnet.sh`
- `/root/setup_port80.sh`
- `/root/cek_status.sh`
- `/root/traffic_protocol7.sh`

## Topologi dan Alokasi IP

Router Lain memiliki satu interface ke NAT GNS3 dan tiga interface sebagai gateway untuk tiga subnet yang berbeda.

| Node    | Interface | IP address                         | Gateway         | Segmen       |
| ------- | --------- | ---------------------------------- | --------------- | ------------ |
| Lain    | `eth0`    | DHCP, teramati `192.168.122.96/24` | `192.168.122.1` | NAT/internet |
| Lain    | `eth1`    | `10.75.1.1/24`                     | -               | Switch 1     |
| Alice   | `eth0`    | `10.75.1.2/24`                     | `10.75.1.1`     | Switch 1     |
| Mika    | `eth0`    | `10.75.1.3/24`                     | `10.75.1.1`     | Switch 1     |
| Lain    | `eth2`    | `10.75.2.1/24`                     | -               | Switch 2     |
| Chisa   | `eth0`    | `10.75.2.2/24`                     | `10.75.2.1`     | Switch 2     |
| Lain    | `eth3`    | `10.75.3.1/24`                     | -               | Switch 3     |
| Knights | `eth0`    | `10.75.3.2/24`                     | `10.75.3.1`     | Switch 3     |
| Eiri    | `eth0`    | `10.75.3.3/24`                     | `10.75.3.1`     | Switch 3     |

![Topologi The Wired](./assets/01/01_Topologi.png)

## Ringkasan Hasil

## Ringkasan Hasil

| No. | Pengujian                        | Hasil                                                            |
| --: | -------------------------------- | ---------------------------------------------------------------- |
|   1 | Topologi dan IP statis           | Berhasil, tiga subnet dan lima client terkonfigurasi             |
|   2 | DHCP/NAT pada `eth0` router      | Berhasil, router memperoleh IP dan dapat mengakses `8.8.8.8`     |
|   3 | Routing antar-subnet             | Berhasil, komunikasi antarsegmen tanpa packet loss               |
|   4 | NAT masquerade dan DNS           | Berhasil pada seluruh client                                     |
|   5 | Persistensi setelah restart      | Interface, route, NAT, dan konektivitas tetap aktif              |
|   6 | Filter Wireshark `dns \|\| icmp` | 48 dari 52 paket ditampilkan                                     |
|   7 | Hak akses FTP                    | Alice read-write, Mika read-only, Eiri ditolak                   |
|   8 | Upload FTP dari Knights          | `STOR`, respons `226`, port PASV `17964`                         |
|   9 | Akses FTP Mika                   | Download berhasil, upload ditolak dengan `550 Permission denied` |
|  10 | Ping 77 paket                    | 0% packet loss; RTT `0.437/0.570/1.090 ms`                       |
|  11 | Analisis Telnet                  | Kredensial terlihat plaintext; data karakter berukuran 1 byte    |
|  12 | Pemindaian Netcat                | Port 22/80 terbuka, port 7777 tertutup                           |
|  13 | SSH public-key authentication    | Login tanpa password berhasil dan payload sesi terenkripsi       |

## Dokumentasi Pengerjaan

### 1. Pembuatan Topologi dan Konfigurasi Client

Topologi terdiri dari NAT, router Lain, tiga switch, dan lima client. Switch 1 menghubungkan Alice dan Mika, Switch 2 menghubungkan Chisa, sedangkan Switch 3 menghubungkan Knights dan Eiri. Seluruh node menggunakan prefix kelompok `10.75`.

<details>
<summary>Bukti konfigurasi topologi dan interface</summary>

![Topologi](./assets/01/01_Topologi.png)

![Konfigurasi router Lain](./assets/01/01_konfigurasi_routerLain.png)

![Konfigurasi Alice](./assets/01/01_config_Alice.png)

![Konfigurasi Mika](./assets/01/01_config_mika.png)

![Konfigurasi Chisa](./assets/01/01_config_chisa.png)

![Konfigurasi Knights](./assets/01/01_config_knights.png)

![Konfigurasi Eiri](./assets/01/01_config_eiri.png)

</details>

### 2. Koneksi Router ke Internet melalui DHCP

Interface `eth0` pada router Lain dikonfigurasi dengan DHCP. Hasil pengujian menunjukkan:

- `eth0` memperoleh `192.168.122.96/24`.
- Default route mengarah ke `192.168.122.1`.
- Ping ke `8.8.8.8` berhasil 4/4 paket dengan 0% packet loss.
- RTT pengujian adalah `19.174/19.845/20.554 ms` untuk min/avg/max.

```ini
auto eth0
iface eth0 inet dhcp
```

<details>
<summary>Bukti DHCP dan koneksi internet router</summary>

![Konfigurasi DHCP router](./assets/02/02_config_DHCP.png)

![Pengujian internet router](./assets/02/02_Router_Lain_DHCP_Internet.png)

</details>

### 3. Routing Antar-Subnet

IPv4 forwarding diaktifkan pada router Lain:

```sh
sysctl -w net.ipv4.ip_forward=1
```

Tabel routing memuat tiga jaringan lokal berikut:

```text
10.75.1.0/24 dev eth1 src 10.75.1.1
10.75.2.0/24 dev eth2 src 10.75.2.1
10.75.3.0/24 dev eth3 src 10.75.3.1
```

Pengujian representatif dari Alice, Chisa, dan Eiri menuju node-node di subnet lain seluruhnya menghasilkan 0% packet loss. Hal ini membuktikan router Lain dapat meneruskan paket di antara Switch 1, Switch 2, dan Switch 3.

<details>
<summary>Bukti routing antar-subnet</summary>

![Routing router Lain](./assets/03/03_Routing_Router_Lain.png)

![Pengujian routing Alice](./assets/03/03_Alice_Routing.png)

![Pengujian routing Chisa](./assets/03/03_Chisa_Routing.png)

![Pengujian routing Eiri](./assets/03/03_Eiri_Routing.png)

</details>

### 4. NAT Masquerade dan DNS Resolver

Router Lain menggunakan NAT masquerade pada trafik keluar melalui `eth0`. Rule forwarding mengizinkan trafik dari ketiga subnet menuju internet dan menerima trafik balasan dengan state `RELATED,ESTABLISHED`.

```sh
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth3 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Setiap client menggunakan resolver:

```text
nameserver 8.8.8.8
```

Ping ke `8.8.8.8` menguji konektivitas IP, sedangkan ping ke `google.com` sekaligus membuktikan resolusi DNS. Screenshot menunjukkan Alice, Mika, Chisa, Knights, dan Eiri berhasil melakukan kedua pengujian dengan 0% packet loss.

<details>
<summary>Bukti NAT, DNS, dan konektivitas seluruh client</summary>

![Konfigurasi NAT router](./assets/04/04_Config_NAT_Router_Lain.png)

![Rule NAT dan forwarding](./assets/04/04_NAT_Forward_Rules.png)

![DNS resolver](./assets/04/04_DNS_Resolver.png)

![Internet Alice](./assets/04/04_Alice_DNS.png)

![Internet Mika](./assets/04/04_Mika_DNS.png)

![Internet Chisa](./assets/04/04_Chisa_DNS.png)

![Internet Knights](./assets/04/04_Knights_DNS.png)

![Internet Eiri](./assets/04/04_Eiri_DNS.png)

</details>

### 5. Persistensi Konfigurasi Setelah Restart

Konfigurasi interface dan rule router disimpan dalam `/etc/network/interfaces`, sedangkan script verifikasi dibuat di `/root/cek_status.sh`.

```sh
#!/bin/sh

echo "=== STATUS INTERFACE ==="
ip -br a

echo
echo "=== STATUS NAT ==="
iptables -t nat -L -v -n
```

Setelah seluruh node direstart, router kembali memiliki alamat pada `eth0` hingga `eth3`, tabel NAT tetap memuat rule `MASQUERADE`, default route kembali tersedia, dan pengujian ke `8.8.8.8` serta `google.com` berhasil.

<details>
<summary>Bukti persistensi dan script cek status</summary>

![Script cek status](./assets/05/05_Script_Cek_Status.png)

![Cek status setelah restart](./assets/05/05_Cek_Status_Setelah_Restart.png)

![Verifikasi setelah restart](./assets/05/05_Verifikasi_Client_Setelah_Restart.png)

</details>

### 6. Packet Sniffing DNS dan ICMP pada Mika

Traffic generator pada Mika menghasilkan ping ke `8.8.8.8`, `1.1.1.1`, dan `its.ac.id`, serta query DNS untuk beberapa domain. Display filter yang digunakan adalah:

```wireshark
dns || icmp
```

Hasil capture memperlihatkan DNS query/response serta ICMP Echo Request/Reply. Dari total 52 paket pada capture, 48 paket lolos filter atau `92.3%` dari keseluruhan paket.

<details>
<summary>Bukti hasil filter DNS dan ICMP</summary>

![Filter DNS dan ICMP bagian 1](./assets/06/06_Wireshark_DNS_ICMP_Filter-1.png)

![Filter DNS dan ICMP bagian 2](./assets/06/06_Wireshark_DNS_ICMP_Filter-2.png)

</details>

### 7. FTP Server Chisa dan Kebijakan Hak Akses

vsFTPd berjalan pada TCP port 21 dengan shared folder `/var/wired/data`. Kebijakan yang diterapkan adalah:

| User    | Kebijakan             | Bukti                                                                      |
| ------- | --------------------- | -------------------------------------------------------------------------- |
| `alice` | Read-write            | Berhasil mengunggah `signal_alice.txt` berisi `Signal from Alice`          |
| `mika`  | Read-only             | Menjadi anggota grup `wiredread`; penolakan upload dibuktikan pada nomor 9 |
| `eiri`  | Blacklist/tanpa akses | Login ditolak dengan respons `530 Permission denied`                       |

Folder bersama dimiliki oleh `alice:wiredread` dengan permission `2750`, sehingga Alice dapat menulis dan anggota grup hanya dapat membaca/menelusuri folder.

<details>
<summary>Bukti konfigurasi dan kebijakan FTP</summary>

![Konfigurasi permission FTP Chisa](./assets/07/07_Config_Permission_FTP_Chisa.png)

![File hasil upload Alice](./assets/07/07_Alice_Upload_signal_alice.png)

![Login Eiri ditolak](./assets/07/07_Eiri_Login_Denied.png)

</details>

### 8. Upload Laporan Knights melalui FTP PASV

Knights terhubung ke FTP Server Chisa menggunakan akun Alice dan mengunggah `knights_report.txt`. Analisis sesi menunjukkan:

| Parameter             | Temuan                                        |
| --------------------- | --------------------------------------------- |
| Perintah upload       | `STOR knights_report.txt`                     |
| Respons awal transfer | `150 Ok to send data`                         |
| Respons sukses        | `226 Transfer complete`                       |
| Respons PASV          | `227 Entering Passive Mode (10,75,2,2,70,44)` |
| Port data PASV        | `70 × 256 + 44 = 17964`                       |
| Ukuran data           | 1112 bytes                                    |

Mode PASV membuat client membuka koneksi data menuju port dinamis server. Dua angka terakhir pada respons `227`, yaitu 70 dan 44, digunakan untuk menghitung port TCP data.

<details>
<summary>Bukti upload FTP dan file pada Chisa</summary>

![Analisis PASV STOR dan 226](./assets/08/08_FTP_PASV_STOR_226.png)

![Laporan Knights bagian 1](./assets/08/08_File_Knights_Report_Chisa-1.png)

![Laporan Knights bagian 2](./assets/08/08_File_Knights_Report_Chisa-2.png)

</details>

### 9. Akses FTP Read-Only oleh Mika

Mika berhasil mengunduh `protocol7_manifesto.txt` berukuran 1739 bytes. Ketika mencoba mengunggah `mika_test.txt`, server menolak perintah `STOR` dengan respons:

```text
550 Permission denied.
```

Hasil ini membuktikan akun Mika memiliki akses baca tetapi tidak memiliki akses tulis.

<details>
<summary>Bukti download dan penolakan upload Mika</summary>

![Download Protocol 7 oleh Mika](./assets/09/09_Mika_Download_Protocol7.png)

![Upload Mika ditolak](./assets/09/09_Mika_Upload_Denied.png)

</details>

### 10. Uji Latensi ICMP dari Knights ke Chisa

Perintah pengujian:

```sh
ping -c 77 -s 128 -i 0.3 10.75.2.2
```

| Parameter            | Hasil                        |
| -------------------- | ---------------------------- |
| Sumber               | Knights - `10.75.3.2`        |
| Tujuan               | Chisa - `10.75.2.2`          |
| Payload              | 128 bytes                    |
| Interval             | 0.3 detik                    |
| Echo Request         | ICMP Type 8, Code 0          |
| Echo Reply           | ICMP Type 0, Code 0          |
| Paket                | 77 dikirim, 77 diterima      |
| Packet loss          | 0%                           |
| RTT min/avg/max/mdev | `0.437/0.570/1.090/0.118 ms` |

Pada Wireshark, ukuran frame yang terlihat adalah 170 bytes. Output ping menampilkan 136 bytes, yaitu gabungan 128 bytes payload dan 8 bytes header ICMP. Seluruh 77 request mendapat reply sehingga koneksi stabil dan latensinya rendah.

<details>
<summary>Bukti ICMP Type/Code dan hasil ping</summary>

![ICMP Echo Request](./assets/10/10_ICMP_Echo_Request.png)

![ICMP Echo Reply](./assets/10/10_ICMP_Echo_Reply.png)

![Hasil ping 77 paket](./assets/10/10_Ping_77_RTT.png)

</details>

### 11. Kelemahan Telnet: Kredensial Plaintext

Login dilakukan dari Eiri (`10.75.3.3`) menuju layanan Telnet Chisa (`10.75.2.2:23`) menggunakan:

```text
Username: phantom_user
Password: wired_ghost
```

Fitur **Follow TCP Stream** dapat menampilkan username dan password karena Telnet tidak mengenkripsi isi sesi. Filter berikut memperlihatkan input client yang dikirim sebagai TCP payload berukuran satu byte:

```wireshark
tcp.stream == 1 && ip.src == 10.75.3.3 && tcp.dstport == 23 && tcp.len == 1
```

Setiap karakter muncul dalam segmen terpisah karena sesi terminal Telnet bekerja secara interaktif/character-at-a-time: input segera dikirim tanpa menunggu satu baris lengkap. Pada capture terlihat segmen `PSH, ACK` dengan `Len: 1`. TCP dapat menggabungkan data pada kondisi lain, tetapi pada sesi ini tiap input memang tertangkap sebagai satu karakter per segmen.

<details>
<summary>Bukti kredensial plaintext dan paket per karakter</summary>

![Follow TCP Stream Telnet](./assets/11/11_Telnet_Follow_TCP_Stream.png)

![Paket karakter Telnet](./assets/11/11_Telnet_Character_Packets.png)

</details>

### 12. Pemindaian Port Knights dengan Netcat

Alice memindai node Knights (`10.75.3.2`) menggunakan Netcat:

```sh
nc -vz -w 2 10.75.3.2 22
nc -vz -w 2 10.75.3.2 80
nc -vz -w 2 10.75.3.2 7777
```

| Port | Layanan | Hasil Netcat                  | Respons TCP |
| ---: | ------- | ----------------------------- | ----------- |
|   22 | SSH     | Terbuka                       | `SYN, ACK`  |
|   80 | HTTP    | Terbuka                       | `SYN, ACK`  |
| 7777 | Rahasia | Tertutup / connection refused | `RST, ACK`  |

Port terbuka membalas paket SYN dengan SYN-ACK karena ada layanan yang mendengarkan. Port tertutup langsung membalas RST-ACK untuk menolak koneksi.

<details>
<summary>Bukti Netcat dan analisis TCP flags</summary>

![Pemindaian port dengan Netcat](./assets/12/12_Netcat_Port_Scan.png)

![Perbandingan SYN-ACK dan RST-ACK](./assets/12/12_TCP_SYNACK_RSTACK.png)

</details>

### 13. SSH Tanpa Password dengan Public Key Authentication

OpenSSH server dikonfigurasi pada Knights, sedangkan pasangan kunci Ed25519 dibuat pada Mika untuk user `mika_admin`. Konfigurasi penting pada server:

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

Mika (`10.75.1.3`) berhasil login sebagai `mika_admin` ke Knights (`10.75.3.2`) tanpa memasukkan password. Capture Wireshark memperlihatkan:

- Client dan server melakukan **Protocol Version Exchange** dengan `SSH-2.0-OpenSSH_10.2`.
- Client dan server bertukar paket **Key Exchange Init**.
- Dilanjutkan PQ/T hybrid key exchange, `New Keys`, dan paket terenkripsi.

Berbeda dari Telnet, SSH melakukan pertukaran kunci untuk membentuk session key. Proses autentikasi dan data aplikasi setelah key exchange dienkripsi, sehingga username, kredensial, serta perintah tidak dapat dibaca sebagai teks terbuka dari packet capture.

<details>
<summary>Bukti login SSH dan proses pertukaran kunci</summary>

![Login SSH tanpa password](./assets/13/13_SSH_Login_No_Password.png)

![SSH Protocol Version Exchange](./assets/13/13_SSH_Protocol_Version_Exchange.png)

![SSH Key Exchange](./assets/13/13_SSH_Key_Exchange.png)

</details>

## Nomor 14-20

aszqx# NOMOR 14 — Analisis `wired_bruteforce.pcapng`

## Tujuan

Mencari:

1. IP penyerang
2. IP target
3. Port yang diserang
4. Password user `lain_admin`
5. Web server software dan versinya pada response header

## Langkah Pengerjaan

### 1. Buka file PCAP

Buka Wireshark:

```text
File → Open
```

Pilih:

```text
wired_bruteforce.pcapng
```

### 2. Cari trafik HTTP

Pada kolom Display Filter:

```text
http
```

Tekan **Enter**.

### 3. Cari request login

Gunakan filter:

```text
http.request.method == "POST"
```

Cari request yang berhubungan dengan login.

### 4. Tentukan IP penyerang

Klik paket login dan lihat bagian:

```text
Internet Protocol Version 4
```

Perhatikan:

```text
Source
Destination
```

IP pada `Source` dicatat sebagai IP penyerang.

```text
IP Penyerang : [hasil dari PCAP]
```

### 5. Tentukan IP target dan port

Pada paket yang sama lihat:

```text
Destination
TCP Destination Port
```

Catat:

```text
IP Target : [hasil dari PCAP]
Port Target : [hasil dari PCAP]
```

### 6. Cari password `lain_admin`

Klik kanan paket login:

```text
Follow → HTTP Stream
```

Cari username:

```text
lain_admin
```

Kemudian cari password pada request tersebut.

Catat:

```text
Username : lain_admin
Password : [hasil dari PCAP]
```

### 7. Cari web server software dan versi

Cari HTTP response dari server.

Buka:

```text
HTTP Response
→ Response Headers
```

Cari field:

```text
Server:
```

Catat:

```text
Web Server : [hasil dari PCAP]
Versi : [hasil dari PCAP]
```

## Validasi Socket

```bash
nc [IP_Group] 3401
```

Masukkan jawaban sesuai format yang diminta socket.

## Dokumentasi

Screenshot yang disarankan:

- Paket login yang menunjukkan Source, Destination, dan port.
- HTTP Stream yang menunjukkan username dan password.
- Response header yang menunjukkan `Server`.
- Hasil validasi socket.

---

# NOMOR 15 — Analisis `wired_usb_hid.pcap`

## Tujuan

Mencari:

1. Vendor ID
2. Product ID
3. USB device address
4. Pesan rahasia dari keystroke

## Langkah Pengerjaan

### 1. Buka file PCAP

```text
File → Open → wired_usb_hid.pcap
```

### 2. Tampilkan trafik USB

Gunakan Display Filter:

```text
usb
```

### 3. Cari Device Descriptor

Cari paket yang berhubungan dengan:

```text
Device Descriptor
```

Buka bagian USB dan cari:

```text
idVendor
idProduct
```

Catat:

```text
Vendor ID : [hasil dari PCAP]
Product ID : [hasil dari PCAP]
```

### 4. Cari USB Device Address

Masih pada detail USB, cari alamat device.

```text
USB Device Address : [hasil dari PCAP]
```

### 5. Cari paket HID keyboard

Cari paket yang berisi data keyboard/HID.

Perhatikan field seperti:

```text
USB HID Data
usb.capdata
```

### 6. Rekonstruksi keystroke

Periksa paket keyboard secara berurutan dan cocokkan kode tombol dengan karakter yang diketik sampai membentuk pesan.

Catat:

```text
Pesan Rahasia : [hasil decoding]
```

## Validasi Socket

```bash
nc [IP_Group] 3402
```

Masukkan:

```text
Vendor ID
Product ID
USB Device Address
Pesan Rahasia
```

sesuai urutan yang diminta socket.

## Dokumentasi

Screenshot:

- Device Descriptor.
- Vendor ID dan Product ID.
- USB device address.
- Paket HID keyboard.
- Hasil validasi socket.

---

# NOMOR 16 — Analisis `wired_ftp_theft.pcap`

## Tujuan

Mencari:

1. IP server FTP penyerang
2. Banner software FTP
3. Username login
4. Password login
5. Ukuran file `knights_payload.exe`

## Langkah Pengerjaan

### 1. Buka file PCAP

```text
File → Open → wired_ftp_theft.pcap
```

### 2. Filter FTP

```text
ftp
```

### 3. Cari banner server

Cari response awal server, biasanya response dengan kode:

```text
220
```

Catat:

```text
FTP Server IP : [hasil dari PCAP]
FTP Banner : [hasil dari PCAP]
```

### 4. Cari username

Cari command:

```text
USER
```

Catat:

```text
Username : [hasil dari PCAP]
```

### 5. Cari password

Cari command:

```text
PASS
```

Catat:

```text
Password : [hasil dari PCAP]
```

### 6. Cari file malware

Cari command:

```text
RETR knights_payload.exe
```

`RETR` menunjukkan proses download file.

### 7. Cari ukuran file

Periksa informasi ukuran file pada trafik FTP.

Catat:

```text
File : knights_payload.exe
Size : [jumlah bytes]
```

## Validasi Socket

```bash
nc [IP_Group] 3403
```

Masukkan jawaban sesuai format socket.

## Dokumentasi

Screenshot:

- Banner FTP.
- Command `USER`.
- Command `PASS`.
- `RETR knights_payload.exe`.
- Informasi ukuran file.
- Hasil validasi socket.

---

# NOMOR 17 — Analisis `wired_http_c2.pcap`

## Tujuan

Mencari:

1. Host/domain tempat malware diunduh
2. IP server penyerang
3. Nama file executable malware
4. HTTP status code

## Langkah Pengerjaan

### 1. Buka file PCAP

```text
File → Open → wired_http_c2.pcap
```

### 2. Filter HTTP

```text
http
```

### 3. Cari HTTP Request

Cari request yang melakukan download.

Buka:

```text
Hypertext Transfer Protocol
```

Cari:

```text
Host:
```

Catat:

```text
Host/Domain : [hasil dari PCAP]
```

### 4. Tentukan IP server penyerang

Lihat:

```text
Destination
```

Catat:

```text
IP Server Penyerang : [hasil dari PCAP]
```

### 5. Tentukan nama file executable

Periksa:

```text
Request URI
```

Contoh bentuk:

```text
GET /files/nama.exe HTTP/1.1
```

Ambil nama executable dari URI.

```text
Executable : [hasil dari PCAP]
```

### 6. Cari HTTP Status Code

Cari response dari server dan perhatikan:

```text
HTTP/1.1 xxx
```

Catat:

```text
Status Code : [hasil dari PCAP]
```

## Validasi Socket

```bash
nc [IP_Group] 3404
```

Masukkan jawaban sesuai format socket.

## Dokumentasi

Screenshot:

- HTTP request.
- `Host`.
- Destination IP.
- Request URI.
- HTTP status code.
- Hasil validasi socket.

---

# NOMOR 18 — Analisis `wired_smb_transfer.pcapng`

## Tujuan

Mencari:

1. Protokol jaringan yang dieksploitasi
2. IP pengirim
3. IP penerima
4. Folder tujuan penyimpanan malware
5. Nama file executable malware

## Langkah Pengerjaan

### 1. Buka file PCAP

```text
File → Open → wired_smb_transfer.pcapng
```

### 2. Filter SMB

Coba:

```text
smb
```

Jika capture menggunakan SMB2:

```text
smb2
```

### 3. Tentukan protokol

Lihat nama protokol pada Packet List/Packet Details.

Catat:

```text
Protocol : [hasil dari PCAP]
```

### 4. Tentukan IP pengirim dan penerima

Lihat:

```text
Source
Destination
```

Catat:

```text
IP Pengirim : [hasil dari PCAP]
IP Penerima : [hasil dari PCAP]
```

### 5. Cari folder tujuan

Perhatikan paket yang berhubungan dengan:

```text
Tree Connect
Create Request
```

Cari path/folder tempat file akan disimpan.

Catat:

```text
Folder Tujuan : [hasil dari PCAP]
```

### 6. Cari nama file executable

Pada `Create Request` atau data transfer, cari file dengan ekstensi:

```text
.exe
```

Catat:

```text
Nama Malware : [hasil dari PCAP]
```

## Validasi Socket

```bash
nc [IP_Group] 3405
```

Masukkan jawaban sesuai urutan yang diminta socket.

## Dokumentasi

Screenshot:

- Paket SMB/SMB2.
- Source dan Destination IP.
- Folder/path tujuan.
- Nama file executable.
- Hasil validasi socket.

---

# NOMOR 19 — Analisis `wired_smtp_threat.pcap`

## Tujuan

Mencari:

1. Alamat email korban
2. Password korban yang diklaim bocor
3. Jenis malware
4. Batas waktu dalam hari
5. MailClientID

## Langkah Pengerjaan

### 1. Buka file PCAP

```text
File → Open → wired_smtp_threat.pcap
```

### 2. Filter SMTP

```text
smtp
```

### 3. Gunakan Follow TCP Stream

Klik kanan paket SMTP:

```text
Follow → TCP Stream
```

### 4. Cari email korban

Cari command:

```text
RCPT TO:
```

Alamat setelah command tersebut adalah alamat email korban.

```text
Email Korban : [hasil dari PCAP]
```

### 5. Cari password yang diklaim bocor

Setelah:

```text
DATA
```

akan terlihat isi email.

Cari bagian yang menyebut password.

```text
Password : [hasil dari PCAP]
```

### 6. Cari jenis malware

Masih pada isi email, cari penyebutan jenis malware.

```text
Jenis Malware : [hasil dari PCAP]
```

### 7. Cari batas waktu

Cari kalimat yang menyebut deadline/batas waktu.

Catat jumlah hari:

```text
Deadline : [jumlah hari]
```

### 8. Cari MailClientID

Cari teks:

```text
MailClientID
```

Catat:

```text
MailClientID : [hasil dari PCAP]
```

## Validasi Socket

```bash
nc [IP_Group] 3406
```

Masukkan seluruh jawaban sesuai format socket.

## Dokumentasi

Screenshot:

- `RCPT TO`.
- Isi email pada TCP Stream.
- Password.
- Jenis malware.
- Deadline.
- MailClientID.
- Hasil validasi socket.

---

# NOMOR 20 — Analisis `wired_tls_decrypt.pcapng`

## File yang digunakan

```text
wired_tls_decrypt.pcapng
keyslogfile.txt
```

## Tujuan

Mencari:

1. Versi TLS yang dinegosiasikan
2. SNI/domain
3. IP server HTTPS penyerang
4. User-Agent
5. HTTP request method
6. HTTP request path

## Langkah Pengerjaan

### 1. Masukkan keylog ke Wireshark

Buka:

```text
Edit → Preferences
```

Kemudian:

```text
Protocols → TLS
```

Cari:

```text
(Pre)-Master-Secret log filename
```

Arahkan ke:

```text
keyslogfile.txt
```

### 2. Buka PCAP

```text
File → Open → wired_tls_decrypt.pcapng
```

### 3. Cari TLS Handshake

Gunakan:

```text
tls
```

Cari:

```text
Client Hello
Server Hello
```

### 4. Cari versi TLS

Klik `Server Hello`.

Cari versi protokol TLS yang digunakan.

```text
TLS Version : [hasil dari PCAP]
```

### 5. Cari SNI

Klik `Client Hello`.

Buka:

```text
Extensions
→ Server Name
```

Catat:

```text
SNI : [hasil dari PCAP]
```

### 6. Cari IP server HTTPS

Perhatikan:

```text
Destination
```

pada koneksi HTTPS tersebut.

Catat:

```text
IP Server HTTPS : [hasil dari PCAP]
```

### 7. Cari HTTP hasil dekripsi

Setelah keylog berhasil digunakan, cari trafik HTTP yang sudah dapat dibaca.

Gunakan:

```text
http
```

### 8. Cari User-Agent

Pada HTTP request cari:

```text
User-Agent:
```

Catat:

```text
User-Agent : [hasil dari PCAP]
```

### 9. Cari HTTP Method

Lihat request line.

Contoh:

```text
GET /xxxx HTTP/1.1
```

atau:

```text
POST /xxxx HTTP/1.1
```

Catat:

```text
Method : [GET/POST/dll.]
```

### 10. Cari request path

Pada request line, ambil bagian setelah method.

Contoh:

```text
GET /malware/payload.exe HTTP/1.1
```

Maka:

```text
Path : /malware/payload.exe
```

## Kesimpulan

Implementasi nomor 1-13 berhasil membangun tiga subnet yang dirutekan oleh Lain, memberikan akses internet mandiri kepada seluruh client melalui NAT dan DNS, mempertahankan konfigurasi setelah restart, serta menjalankan layanan FTP, Telnet, dan SSH sesuai skenario. Analisis Wireshark juga menunjukkan perbedaan keamanan yang jelas: Telnet mengekspos kredensial dalam plaintext, sedangkan SSH melindungi proses autentikasi dan data sesi menggunakan enkripsi.
