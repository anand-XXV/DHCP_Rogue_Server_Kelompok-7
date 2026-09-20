# DHCP_Rogue_Server_Kelompok-7

| Nama | NRP |
|:--|:-:|
| Sultan Ahmad Maulana Bahyshidqi | 5027251070|
| I Made Gyanendra Anand Wisnawa | 5027251072 |
| Muhammad Rifki Pribadi | 5027251087 |

## Pendahuluan

DHCP Rogue Server (Server DHCP Liar/Palsu) merupakan sebuah server DHCP tidak sah yang berjalan di dalam suatu infrastruktur jaringan komputer tanpa izin maupun sepengetahuan administrator jaringan. Kehadiran server liar ini bisa terjadi akibat kesengajaan dari pihak penyerang (attacker) yang ingin menyusup, atau akibat kelalaian pengguna yang tanpa sadar menghubungkan perangkat router tambahan ke port jaringan area lokal (LAN).

Pada kondisi operasional normal, setiap kali ada perangkat baru seperti laptop atau hp yang terhubung ke jaringan, perangkat tersebut akan menyiarkan (broadcast) permintaan khusus untuk mendapatkan alamat IP. Dalam situasi ini, server DHCP resmi bertugas merespons permintaan tersebut dan membagikan parameter konfigurasi jaringan yang valid, termasuk IP Address, Subnet Mask, Default Gateway, hingga DNS Server.

Ancaman mulai terjadi ketika DHCP Rogue Server ikut mendengarkan permintaan broadcast tersebut. Server liar ini akan langsung merespons dengan mengirimkan paket tawaran (DHCP Offer) secepat mungkin untuk mendahului jawaban dari server DHCP resmi. Karena sistem operasi klien secara otomatis akan menerima tawaran pertama yang tiba, perangkat korban akhirnya mengambil konfigurasi IP dari server palsu tersebut.

Dampaknya sangat berbahaya bagi keamanan data. Penyerang dapat mengatur alamat komputer mereka sendiri sebagai Default Gateway, sehingga seluruh arus data internet milik korban akan dipaksa melewati komputer penyerang terlebih dahulu (Man-in-the-Middle Attack). Selain itu, penyerang juga bisa mengubah alamat DNS Server untuk mengarahkan pengguna ke situs web phishing, atau sekadar memberikan parameter IP yang salah hingga menyebabkan korban kehilangan akses internet (Denial of Service).

## Studi Kasus
Diungkap oleh peneliti Leviathan Security pada 6 Mei 2024. Serangan ini menyalahgunakan DHCP option 121 (classless static routes) — sebuah opsi konfigurasi resmi dalam protokol DHCP yang seharusnya dipakai untuk mengatur rute jaringan, namun disalahgunakan penyerang untuk membelokkan trafik korban.

Cara kerjanya penyerang menjalankan server DHCP di jaringan yang sama dengan target pengguna VPN, dan mengatur konfigurasi DHCP tersebut agar menjadikan dirinya sendiri sebagai gateway. Ketika trafik korban sampai ke gateway palsu itu, penyerang memakai aturan forwarding untuk meneruskan trafik ke gateway asli sambil menyadapnya.
(https://www.bleepingcomputer.com/news/security/new-tunnelvision-attack-leaks-vpn-traffic-using-rogue-dhcp-servers/)
