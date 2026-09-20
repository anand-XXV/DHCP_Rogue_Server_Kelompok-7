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

## Hands-On PCAP

### link file PCAP

https://github.com/baldassarreFe/FEP3370-advanced-ethical-hacking/blob/main/media/attack.pcap

### Identifikasi Serangan dari Wireshark

File 'attack.pcap' dibuka menggunakan Wireshark dan difilter dengan:
```
dhcp
```

<img src="./assets/01.png" width="300">

dari hasil capture, terdapat beberapa perangkat yang terlibat:

| Perangkat | IP Address | MAC Address | Peran |
|---|---|---|---|
| Gateway | `192.168.0.1` | `08:00:dd:dd:dd:dd` | DHCP Server resmi |
| Attacker | `192.168.0.101` | `08:00:aa:aa:aa:aa` | Rogue DHCP Server |
| Victim | - | `08:00:ff:ff:ff:ff` | DHCP Client |

Berdasarkan skenario pada repository, perangkat dengan MAC address `08:00:aa:aa:aa:aa` merupakan mesin attacker, sedangkan `08:00:dd:dd:dd:dd` merupakan DHCP server resmi. Pada hasil capture juga terlihat bahwa IP `192.168.0.101` terhubung dengan MAC address `08:00:aa:aa:aa:aa`.

Perangkat tersebut kemudian terlihat mengirimkan DHCP Offer dan DHCP ACK kepada victim. Karena perangkat yang diketahui sebagai attacker bertindak sebagai DHCP server dan memberikan konfigurasi jaringan kepada client, perangkat tersebut dapat diidentifikasi sebagai **Rogue DHCP Server**.

### Identifikasi Rogue DHCP
Secara proses, Rogue DHCP Server bekerja dengan alur yang sama seperti DHCP Server resmi, yaitu melalui tahapan DORA: Discover, Offer, Request, dan ACK. Perbedaannya bukan pada urutan proses DHCP, tetapi pada identitas dan otorisasi perangkat yang memberikan konfigurasi jaringan.

Pada capture ini, victim tetap melakukan DHCP Discover, kemudian menerima DHCP Offer, mengirim DHCP Request, dan memperoleh DHCP ACK. Namun Offer dan ACK tersebut diberikan oleh perangkat attacker, bukan oleh DHCP Server resmi. Karena itu, meskipun alurnya terlihat normal, proses tersebut tetap dikategorikan sebagai Rogue DHCP.

Rogue DHCP dapat diamati pada frame / baris 18 hingga 21:

<img src="./assets/02.png" width="300">

- **Frame 18 (DHCP Discover)**
    Victim mengirim DHCP Discover secara broadcast untuk mencari DHCP server yang dapat memberikan konfigurasi jaringan.

- **Frame 19 (DHCP Offer)**
    DHCP Offer dikirim oleh 192.168.0.101 dengan MAC address 08:00:aa:aa:aa:aa. Berdasarkan skenario PCAP, perangkat tersebut merupakan attacker, bukan DHCP server resmi. Attacker menawarkan alamat IP 192.168.0.2 kepada victim.
- **Frame 20 (DHCP Request)**
    Victim mengirim DHCP Request sebagai tanda bahwa victim menerima dan meminta alamat IP yang telah ditawarkan sebelumnya.
- **Frame 21 (DHCP ACK)**
    Attacker mengirim DHCP ACK untuk mengonfirmasi pemberian alamat IP kepada victim.

## Kesimpulan
Dokumen ini membahas secara komprehensif mekanisme dan identifikasi Rogue DHCP Server Attack, yaitu serangan yang memanfaatkan absennya mekanisme autentikasi pada protokol DHCP sehingga server tidak sah dapat ikut merespons permintaan DHCP Discover dari client dan mendahului jawaban server resmi. Dengan mengendalikan parameter konfigurasi jaringan seperti Default Gateway dan DNS Server, penyerang dapat membelokkan seluruh trafik korban untuk disadap (Man-in-the-Middle) maupun mengarahkannya ke situs phishing, bahkan melumpuhkan akses jaringan korban (Denial of Service). Dampak nyata dari teknik ini terlihat pada studi kasus TunnelVision (CVE-2024-3661) yang diungkap Leviathan Security pada Mei 2024, di mana penyerang menyalahgunakan DHCP Option 121 (classless static routes) untuk menjadikan dirinya sebagai gateway dan membocorkan trafik pengguna VPN tanpa memutus koneksi VPN itu sendiri. Berdasarkan analisis forensik jaringan pada file PCAP (attack.pcap) menggunakan Wireshark, serangan ini berhasil diidentifikasi melalui proses DORA (Discover–Offer–Request–ACK) yang terlihat normal secara alur, namun paket DHCP Offer dan DHCP ACK pada frame 19 dan 21 ternyata berasal dari perangkat attacker (192.168.0.101, MAC 08:00:aa:aa:aa:aa) dan bukan dari DHCP server resmi (192.168.0.1, MAC 08:00:dd:dd:dd:dd). Temuan ini menegaskan bahwa perbedaan mendasar antara DHCP resmi dan Rogue DHCP bukan terletak pada urutan protokolnya, melainkan pada identitas dan otorisasi perangkat yang memberikan konfigurasi jaringan kepada client, sehingga verifikasi terhadap IP dan MAC address pengirim paket Offer/ACK menjadi langkah krusial dalam mendeteksi keberadaan server DHCP liar di suatu jaringan.

## Sources
https://www.bleepingcomputer.com/news/security/new-tunnelvision-attack-leaks-vpn-traffic-using-rogue-dhcp-servers/

https://github.com/baldassarreFe/FEP3370-advanced-ethical-hacking/blob/main/media/attack.pcap