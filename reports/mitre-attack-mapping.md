# MITRE ATT&CK Mapping — RetailBreach

Pemetaan ini bersifat analisis awal berdasarkan artefak PCAP dan harus diperlakukan sebagai hipotesis yang didukung bukti, bukan klasifikasi resmi.

| Perilaku yang diamati | Taktik/teknik ATT&CK kandidat | Alasan |
|---|---|---|
| Pemindaian direktori dan jalur web | Reconnaissance — T1595.003, Wordlist Scanning | Tool `gobuster/3.6` dan banyak permintaan jalur yang menghasilkan 404 konsisten dengan enumerasi berbasis wordlist. |
| Eksploitasi kelemahan aplikasi web | Initial Access — T1190, Exploit Public-Facing Application | XSS dan LFI terlihat sebagai eksploitasi kelemahan pada aplikasi yang terekspos. Gunakan pemetaan ini secara hati-hati karena konteks lab tidak membuktikan seluruh tahapan akses awal. |
| Pengumpulan cookie sesi | Credential Access — T1539, Steal Web Session Cookie (kandidat) | Skrip XSS berupaya mengirim `document.cookie` ke host penyerang. Keberhasilan pencurian ditunjang oleh aktivitas sesi berikutnya pada lab. |
| Penggunaan sesi untuk mengakses endpoint admin | Defense Evasion / Valid Accounts (kandidat, perlu validasi) | Permintaan admin berikutnya menggunakan cookie sesi. Pemetaan teknik yang tepat bergantung pada konteks autentikasi dan tidak boleh disimpulkan hanya dari satu header. |
| Membaca file sistem melalui path traversal/LFI | Collection — T1005, Data from Local System (kandidat) | Parameter path traversal menargetkan `/etc/passwd`; respons stream tampak memuat konten file. Ini adalah klasifikasi berdasarkan tujuan pengumpulan data, bukan label eksploitasi spesifik. |

## Catatan

- XSS, LFI, dan path traversal tidak selalu memiliki satu teknik ATT&CK yang tepat; pemetaan perlu mengikuti perilaku yang benar-benar dibuktikan.
- Jangan menyatakan persistence, privilege escalation, atau exfiltration berhasil jika bukti PCAP tidak mendukungnya.
- Referensi: [MITRE ATT&CK](https://attack.mitre.org/).
