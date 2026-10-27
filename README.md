# CyberDefenders RetailBreach — Blue Team Investigation

> **Jenis proyek:** Blue Team / SOC Analyst / PCAP Analysis  
> **Platform:** CyberDefenders — RetailBreach  
> **Tools:** Wireshark, CyberChef (URL Decode), browser-based lab  
> **Status:** Selesai — 7/7 pertanyaan terpecahkan

## Ringkasan

Proyek ini mendokumentasikan investigasi file PCAP pada skenario RetailBreach. Analisis berfokus pada identifikasi IP penyerang, enumerasi direktori, injeksi Cross-Site Scripting (XSS), indikasi pencurian cookie sesi, serta eksploitasi Local File Inclusion (LFI)/path traversal untuk membaca `/etc/passwd`.

Seluruh analisis dilakukan pada artefak PCAP yang disediakan di lab. Ini adalah latihan analisis defensif dalam lingkungan terotorisasi, bukan pengujian terhadap sistem publik.

## Tujuan investigasi

- Mengidentifikasi host penyerang, server web, dan host administrator.
- Menemukan aktivitas directory brute-force dari pola respons HTTP 404 dan User-Agent.
- Merekonstruksi payload XSS dari permintaan HTTP POST.
- Menentukan waktu kunjungan administrator ke halaman yang terinfeksi dalam UTC.
- Mengidentifikasi indikasi penggunaan cookie sesi yang dicuri.
- Menelusuri eksploitasi path traversal/LFI dan file sistem yang ditargetkan.

## Temuan utama

| Temuan | Hasil |
|---|---|
| IP penyerang | `111.224.180.128` |
| Web server | `73.124.17.52` |
| IP administrator | `135.143.142.5` |
| Tool directory enumeration | `gobuster/3.6` (User-Agent) |
| Halaman XSS | `/reviews.php` |
| Waktu kunjungan admin relevan (UTC) | `2024-03-29 12:09` (presisi jawaban lab: menit) |
| Skrip yang dieksploitasi | `log_viewer.php` (terlihat pada URI `/admin/log_viewer.php`) |
| File sistem yang ditargetkan | `/etc/passwd` |

**Catatan keamanan:** nilai token sesi tidak dicantumkan dalam dokumentasi publik. Screenshot yang berpotensi menampilkan token telah disunting untuk menyamarkan nilainya.

## Kronologi ringkas

1. **Enumerasi direktori:** banyak permintaan ke jalur yang tidak ditemukan menghasilkan HTTP 404. HTTP Stream menunjukkan `User-Agent: gobuster/3.6`.
2. **Injeksi XSS:** permintaan `POST /reviews.php` berisi parameter `review` dengan JavaScript yang mengirim `document.cookie` ke host penyerang.
3. **Kunjungan administrator:** permintaan administrator ke `/reviews.php` yang relevan tercatat sekitar `2024-03-29 12:09 UTC`.
4. **Penggunaan sesi:** permintaan selanjutnya dari IP penyerang menyertakan cookie sesi yang diperoleh dari serangan. Token sengaja tidak disalin ke README.
5. **Eksploitasi LFI/path traversal:** penyerang mengakses `/admin/log_viewer.php` dengan parameter `file` berisi traversal direktori untuk membaca `/etc/passwd`.

## Analisis teknis dan filter Wireshark

### 1. Identifikasi percakapan IP

Buka **Statistics → Conversations → IPv4**. Percakapan utama yang terlihat pada bukti menghubungkan `111.224.180.128` dan `73.124.17.52`.

### 2. Identifikasi directory brute-force

```wireshark
http.response.code == 404
```

Ikuti aliran HTTP yang relevan (**Follow → HTTP Stream**) dan periksa `User-Agent`. Pada bukti lab, nilainya adalah `gobuster/3.6`.

### 3. Temukan permintaan XSS

```wireshark
ip.src == 111.224.180.128 && http contains "script"
```

Periksa permintaan `POST /reviews.php` dan decode nilai parameter `review` bila masih URL-encoded. Payload yang diamati menggunakan `fetch()` untuk mengirim `document.cookie` ke IP penyerang. Payload lengkap tidak diulang di README agar dokumentasi berfokus pada indikator dan analisis defensif.

### 4. Tentukan waktu kunjungan administrator

```wireshark
ip.src != 111.224.180.128 && http.request.uri contains "reviews.php"
```

Ubah tampilan waktu ke **View → Time Display Format → UTC Date and Time of Day**. Bandingkan dua permintaan ke `/reviews.php` dan gunakan permintaan setelah halaman terinfeksi. Jawaban lab dibulatkan ke menit: `2024-03-29 12:09` UTC.

### 5. Telusuri permintaan yang memakai sesi curian

```wireshark
ip.src == 111.224.180.128 && http
```

Cari permintaan ke endpoint admin dan korelasikan dengan cookie sesi pada HTTP Stream. Jangan mempublikasikan nilai token sesi dalam repository.

### 6. Identifikasi LFI/path traversal

```wireshark
ip.src == 111.224.180.128 && http && frame contains "passwd"
```

Pada HTTP Stream, permintaan yang relevan menuju `/admin/log_viewer.php` dan parameter `file` menggunakan pola `../` untuk mencoba membaca `/etc/passwd`.

## IOC / artefak observasi

Lihat [`data/iocs.csv`](data/iocs.csv). IP dan URI di bawah adalah indikator dari skenario lab; jangan menganggapnya sebagai indikator ancaman global tanpa validasi tambahan.

## Pemetaan MITRE ATT&CK (perkiraan berdasarkan perilaku yang diamati)

Lihat [`reports/mitre-attack-mapping.md`](reports/mitre-attack-mapping.md). Pemetaan merupakan interpretasi analis dan bukan penetapan resmi oleh CyberDefenders.

## Rekomendasi deteksi dan respons

- **Web logs:** buat alert untuk lonjakan HTTP 404 dari satu sumber, permintaan berulang ke banyak direktori, dan User-Agent pemindai seperti `gobuster`.
- **XSS:** validasi dan encode output berdasarkan konteks; jangan memasukkan input pengguna langsung ke HTML; gunakan Content Security Policy sebagai pertahanan tambahan.
- **Cookie sesi:** aktifkan `HttpOnly`, `Secure`, dan `SameSite` sesuai kebutuhan; gunakan HTTPS; rotasi/invalidate sesi setelah indikasi kompromi dan paksa autentikasi ulang.
- **LFI/path traversal:** gunakan allowlist nama file, normalisasi path dan pastikan akses tetap berada dalam direktori yang diizinkan; jangan menerima path arbitrer dari pengguna.
- **Respons insiden:** pertahankan PCAP dan log, korelasikan waktu dengan zona waktu yang konsisten, identifikasi akun/sesi terdampak, rotasi kredensial/sesi, dan periksa akses file serta aktivitas pasca-eksploitasi.

## Struktur repository

```text
RetailBreach-Blue-Team-Portfolio/
├── README.md
├── data/
│   └── iocs.csv
├── evidence/
│   ├── 01-ip-conversation.png
│   ├── 02-directory-enumeration-gobuster.png
│   ├── 03-xss-post-request.png
│   ├── 04-xss-url-decoding.png
│   ├── 05-admin-page-timestamp.png
│   ├── 06-admin-cookie-redacted.png
│   ├── 07-attacker-session-requests.png
│   └── 08-lfi-passwd-redacted.png
└── reports/
    ├── incident-report.md
    └── mitre-attack-mapping.md
```

## Batasan

- Kesimpulan berdasarkan satu file PCAP dan konteks lab RetailBreach.
- Timestamp kunjungan admin ditulis dengan presisi menit sesuai format jawaban lab.
- Tidak ada klaim bahwa host/IP tersebut masih aktif atau berbahaya di luar skenario lab.
- Token sesi dan data autentikasi tidak disertakan dalam artefak publik.

## Referensi

- [CyberDefenders — RetailBreach](https://cyberdefenders.org/blueteam-ctf-challenges/retailbreach/)
- Wireshark display filters: [Wireshark User’s Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- MITRE ATT&CK: [attack.mitre.org](https://attack.mitre.org/)
