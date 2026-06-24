# Laporan Praktikum Jaringan Komputer | Modul 13

**Nama:** Farrellino Ulung Satya Amando  
**NIM:** 103072400005  
**Kelas:** IF 04-01     
---

## 1. Analisis Beacon Frame
Langkah-langkahnya adalah:
  1. Buka Wireshark dan muat file rekaman lalu lintas jaringan nirkabel (`Wireshark_802_11.pcap`).
  2. Terapkan filter logis `wlan.fc.type_subtype == 8` untuk mengisolasi tipe *Management Frame*.
  3. Pilih salah satu *Beacon Frame* dari *Access Point* (AP) bernama "30 Munroe St" dan ekspansi detail lapisannya.


**Analisis:**
*Beacon frame* berfungsi sebagai medium bagi *Access Point* (AP) untuk memancarkan eksistensi jaringan ke lingkungan sekitarnya. Berdasarkan penangkapan, paket dikirimkan secara sebaran luwes (*Broadcast*) dengan alamat tujuan `ff:ff:ff:ff:ff:ff` pada interval tetap sebesar 100 *Time Units* (sekitar 102,4 ms). Di dalam muatan utamanya, AP membungkus atribut krusial seperti identitas SSID ("30 Munroe St"), spektrum frekuensi operasional di Channel 6, hingga rentang kecepatan transmisi yang didukung (mulai dari *basic rates* 1 Mbps hingga *extended rates* 54 Mbps). Ditemukan pula elemen *Vendor Specific* dari Microsoft (Tag 221) yang merinci kapabilitas *Quality of Service* (QoS) melalui parameter WMM/WME untuk mengatur prioritas jenis lalu lintas data.

## 2. Analisis Association Request
Langkah-langkahnya adalah:
  1. Ganti parameter saringan di Wireshark menggunakan filter `wlan.fc.type_subtype == 0`.
  2. Temukan dan sorot paket pengajuan asosiasi (*Association Request*) dari antarmuka nirkabel klien.
  3. Periksa kecocokan parameter yang dinegosiasikan dengan AP.


**Analisis:**
Sebelum pertukaran data dapat terjadi, klien nirkabel (dengan alamat MAC `Intel_d1:b6:4f`) harus mendaftarkan dirinya ke AP melalui frame *Association Request*. Transmisi ini bersifat *unicast* langsung menuju alamat BSSID dari peladen jaringan. Dalam paket negosiasi ini, klien menyatakan kesesuaian kapabilitasnya terhadap jaringan, termasuk pengiriman rentang tarif kecepatan (*Supported Rates* dan *Extended Supported Rates*) yang setara dengan penawaran AP, serta kepatuhan terhadap indikator QoS. Hal ini memastikan kedua sisi antarmuka dapat berkomunikasi dalam satu standar frekuensi dan protokol yang disepakati bersama.

## 3. Analisis Deauthentication Frame
Langkah-langkahnya adalah:
  1. Ubah kembali filter Wireshark menggunakan sintaks `wlan.fc.type_subtype == 12`.
  2. Amati jejak paket penutupan sesi yang dikirimkan oleh mesin klien.
  3. Ekspansi rincian protokol 802.11 untuk menemukan parameter alasan pemutusan.


**Analisis:**
Pemutusan koneksi pada ekosistem nirkabel diinisiasi melalui *Management Frame* bersubtipe *Deauthentication*. Pada frame yang direkam, klien Intel dengan sengaja memutuskan asosiasinya dari AP Ciscolinksys dengan melampirkan *Reason Code* bernilai 0x0001, yang mendeskripsikan "Unspecified reason" atau tanpa alasan pengakhiran yang spesifik. Fenomena pengiriman frame pemutusan yang diulang secara beruntun (*multiple frames*) mengindikasikan upaya preventif dari klien untuk memastikan bahwa AP benar-benar menerima notifikasi terminasi koneksi tersebut tanpa memerlukan mekanisme pengiriman paket balasan (ACK).

## 4. Analisis Transfer Data (HTTP over 802.11)
Langkah-langkahnya adalah:
  1. Hapus filter sebelumnya dan masukkan saringan baru `tcp.port == 80`.
  2. Lacak paket *Data Frame* yang membungkus permintaan HTTP GET (Frame 480).
  3. Telusuri anatomi spesifik enkapsulasi 802.11 pada paket tersebut.


**Analisis:**
Pemeriksaan pada lalu lintas data mengungkap arsitektur pengalamatan khusus dari mode infrastruktur 802.11 yang lebih kompleks ketimbang Ethernet kabel. *Data frame* ini memanfaatkan empat blok *Address field* sekaligus: alamat tujuan akhir (DA), sumber perangkat (SA), alamat penerima antara (RA/MAC AP), dan pengirim langsung (TA/MAC Klien). Flag *Distribution System* (DS) diatur dengan nilai `To DS: 1` dan `From DS: 0`, mengonfirmasi bahwa arah paket mengalir dari stasiun klien menuju jaringan distribusi via AP. Karena menggunakan standar nirkabel, muatan paket IP terlebih dahulu dibungkus oleh header penjembatan LLC/SNAP (Logical Link Control/Subnetwork Access Protocol) sebelum data HTTP sepenuhnya diteruskan. 

## 5. Perbandingan Karakteristik Frame 802.11 
Langkah-langkahnya adalah:
  1. Tinjau ulang daftar visibilitas paket jaringan secara keseluruhan.
  2. Bandingkan secara komprehensif susunan protokol komunikasi 802.11 dengan protokol 802.3 (Ethernet) yang pernah dipraktikkan.


**Analisis:**
Tangkapan jejak koneksi nirkabel ini membuktikan perbedaan arsitektural yang mendasar antara antarmuka WiFi dengan antarmuka kabel tradisional. Tidak seperti Ethernet yang murni hanya membawa lalu lintas data, topologi nirkabel 802.11 sarat akan lalu lintas administratif berupa *Management Frame* (seperti *Beacon*, *Association*) dan *Control Frame* (seperti paket *Acknowledgement* untuk konfirmasi pengiriman) yang beroperasi menggunakan skema akses media CSMA/CA. WiFi juga menghadirkan sistem perunutan menggunakan *Sequence Number* dan *Fragment Number* untuk mendeteksi paket ganda dan pemecahan fragmen, serta mendukung skema manajemen daya lanjutan (WME QoS) yang tak lazim ditemui pada jaringan berbasis kabel konvensional.

### 6. Kesimpulan
Berdasarkan praktikum Modul 13 mengenai protokol WiFi 802.11, dapat dipelajari hal-hal sebagai berikut:

1. Protokol jaringan nirkabel (802.11) memfasilitasi komunikasi berlapis menggunakan tiga varian tipe frame utama: *Management* untuk manajemen akses, *Control* untuk konfirmasi integritas sinyal, dan *Data* untuk pengiriman *payload* aktual lapis atas.
2. *Access Point* menggunakan *Beacon Frame* untuk terus-menerus memancarkan informasi ketersediaan infrastrukturnya, termasuk SSID, nomor *channel*, hingga kebijakan rentang laju (*supported rates*) kepada stasiun-stasiun pemindai (*client stations*).
3. Transisi status keterhubungan sebuah antarmuka dikendalikan secara formal (*stateful*) melalui proses *Association Request/Response* di awal, dan diakhiri secara asinkron menggunakan notifikasi *Deauthentication*.
4. Topologi transmisi melalui media udara memaksa implementasi header MAC yang lebih ekspansif; memanfaatkan hingga empat lajur *Address Field* secara serentak untuk memetakan arah *hop* radio lokal maupun identitas *router* akhir.
5. Mekanisme kendali kualitas (*Quality of Service*) sangat ditekankan melalui eksistensi WMM (*Wi-Fi Multimedia*), yang memberikan alokasi prioritas transmisi berbeda bagi jenis lalu lintas *Background*, *Best Effort*, Video, maupun *Voice*.
