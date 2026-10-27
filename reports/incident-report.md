# Incident Report — RetailBreach

## Ringkasan eksekutif

Analisis PCAP pada lab RetailBreach menunjukkan rangkaian aktivitas yang konsisten dengan reconnaissance pada aplikasi web, injeksi XSS pada halaman review, penyalahgunaan sesi administrator, dan percobaan Local File Inclusion/path traversal pada log viewer. IP penyerang dalam konteks lab adalah `111.224.180.128`, sedangkan server web adalah `73.124.17.52` dan host administrator `135.143.142.5`.

## Dampak potensial

- Cookie sesi admin dapat terekspos jika dapat dibaca JavaScript.
- Sesi administrator dapat disalahgunakan untuk mengakses endpoint admin.
- Path traversal pada log viewer berpotensi memungkinkan pembacaan file sistem yang tidak semestinya.

Dampak di atas adalah penilaian risiko dari pola lalu lintas; laporan ini tidak mengklaim dampak pada sistem nyata di luar lab.

## Timeline

| Waktu (UTC) | Aktivitas | Bukti/interpretasi |
|---|---|---|
| 2024-03-29 12:01 (sekitar) | Enumerasi direktori | Permintaan berulang dengan `User-Agent: gobuster/3.6` dan banyak respons 404. |
| 2024-03-29 12:08 (sekitar) | POST pada `/reviews.php` | Parameter review mengandung skrip yang mengirim cookie ke IP penyerang. |
| 2024-03-29 12:09 | Kunjungan admin ke `/reviews.php` | Timestamp jawaban lab, presisi menit. |
| 2024-03-29 12:11–12:12 (sekitar) | Akses endpoint admin | Permintaan dari IP penyerang menyertakan cookie sesi dan menuju beberapa endpoint admin. |
| 2024-03-29 12:12–12:13 (sekitar) | Path traversal di log viewer | `/admin/log_viewer.php` diminta dengan parameter `file` yang mengarah ke `/etc/passwd`. |

Waktu di atas merupakan pembacaan dari bukti layar/PCAP dan dibulatkan untuk kronologi ringkas. Gunakan timestamp frame asli pada PCAP sebagai sumber utama bila dibutuhkan presisi detik.

## Bukti utama

1. IPv4 Conversations menghubungkan IP penyerang dan server.
2. HTTP Stream menampilkan `gobuster/3.6` pada User-Agent.
3. POST `/reviews.php` berisi parameter review yang URL-encoded dan menunjukkan penggunaan `document.cookie`.
4. Permintaan administrator ke `/reviews.php` setelah injeksi memberi waktu kunjungan relevan.
5. HTTP Stream dari endpoint `/admin/log_viewer.php` menunjukkan traversal direktori untuk membaca `/etc/passwd`.

Screenshot bukti ada di folder `../evidence/`. Screenshot yang memperlihatkan token sesi disunting untuk publikasi.

## Rekomendasi prioritas

1. Invalidate seluruh sesi admin yang berpotensi terdampak dan minta login ulang.
2. Perbaiki output encoding/input handling di `/reviews.php`; evaluasi CSP.
3. Perbaiki `log_viewer.php` dengan allowlist file dan validasi canonical path.
4. Tambahkan deteksi untuk enumerasi 404, URI traversal, dan pola akses file sensitif.
5. Pertahankan PCAP dan log web; korelasikan dengan log autentikasi dan aktivitas administrator.

## Kesimpulan

PCAP menyediakan bukti yang cukup untuk menyusun rangkaian investigasi lab: enumerasi, injeksi XSS, indikasi pencurian sesi, lalu akses endpoint admin dan percobaan membaca file lokal. Validasi perbaikan harus dilakukan di lingkungan uji yang terotorisasi.
