# GUSERA Maps — V5 (PDF Map Engine + Track Recorder + Ukur)

Aplikasi peta lokal seperti Avenza Maps, berjalan sepenuhnya di browser tanpa server dan tanpa koneksi internet (setelah pustaka pihak ketiga tersedia — lihat catatan offline di bawah).

## Perubahan besar di V5

### Versi 5.0 — Measure (fitur baru)
Alat ukur langsung di peta, dihitung sepenuhnya di perangkat (tanpa layanan luar):

* **Jarak** — mode Polyline: ketuk beberapa titik di peta, jarak total (Haversine) dihitung berjalan, ditampilkan dalam meter/kilometer.
* **Luas** — mode Polygon: ketuk minimal 3 titik, luas (proyeksi ekuirektangular + rumus shoelace) & keliling dihitung otomatis, ditampilkan dalam m² dan hektare (Ha).
* **Bearing** — arah antar 2 titik dalam format kuadran survei (mis. `N 45°30'12" E`).
* **Azimuth** — arah antar 2 titik dalam derajat 0–360° searah jarum jam dari Utara, disertai nama mata angin (mis. "Timur Laut").
* **Polyline** — mode gambar garis multi-titik (dipakai pada pengukuran Jarak), ditampilkan sebagai garis oranye dengan penanda nomor urut tiap titik.
* **Polygon** — mode gambar bidang tertutup multi-titik (dipakai pada pengukuran Luas), ditampilkan sebagai area biru transparan.

Cara pakai: tombol **📏 (Ukur)** di peta membuka panel Ukur dengan 3 tab mode (Jarak / Luas / Bearing). Ketuk titik-titik langsung di peta untuk menambah vertex; readout pada panel diperbarui langsung setiap titik ditambahkan. Tombol **↩ Undo** menghapus titik terakhir, **🧹 Hapus** mengosongkan pengukuran. Mode Bearing hanya memakai 2 titik — ketukan ketiga otomatis memulai pasangan titik baru. Alat ukur ini murni visual/sementara (tidak disimpan ke IndexedDB); modul baru `js/measure.js` berisi util perhitungannya, digambar lewat fungsi baru di `js/map.js`.

## Perubahan besar di V4

### 1. Mesin peta diganti dari MBTiles ke PDF
Sesuai arsitektur "GUSERA Maps (PDF Edition)", mulai V4 aplikasi **tidak lagi memakai MBTiles**. Peta sekarang diimpor langsung dari berkas **`.pdf`**:
- Modul baru `js/pdfmap.js` menggantikan `js/mbtiles.js` — memakai **PDF.js** untuk membuka PDF dan me-render halaman terpilih menjadi gambar raster, seluruhnya di `<canvas>` lokal (tidak ada data yang keluar perangkat).
- Modal **"Tambah Peta (PDF)"** kini menampilkan pratinjau halaman + pemilih halaman jika PDF terdiri dari beberapa halaman, sebelum diimpor.
- `map.js` tidak lagi memakai `L.GridLayer` (ubin) — peta PDF ditampilkan sebagai satu **image overlay** Leaflet yang diposisikan lewat *bounds* geografis.
- `storage.js`: object store `maps` berubah skema — menyimpan `pdfBlob` (berkas asli), `imageBlob` (hasil render), `pixelWidth/pixelHeight`, `pageNum/numPages`, dan `calibration/geoBounds`. Versi database IndexedDB naik ke 3.

### 2. Kalibrasi 2-titik (pengganti "GeoPDF otomatis")
Parsing GeoPDF asli berbeda-beda antar penerbit dan sulit diandalkan offline untuk semua kasus. Sebagai gantinya, GUSERA Maps V4 menyediakan **kalibrasi manual 2 titik**:
- Tombol **📐 (Kalibrasi)** di peta mengaktifkan mode "ketuk 2 titik".
- Untuk tiap titik: ketuk lokasinya pada citra PDF, lalu masukkan koordinat GPS sesungguhnya (atau tombol **"🛰 GPS"** jika sedang berdiri di titik tersebut).
- Setelah 2 titik terisi, aplikasi menghitung *bounds* geografis peta (asumsi peta utara-atas, tanpa rotasi) dan menyimpannya — sejak saat itu peta berperilaku seperti GeoPDF sesungguhnya: GPS, waypoint, dan track akan selaras dengan citra.
- Peta yang belum dikalibrasi tetap bisa dibuka & dipakai sebagai "denah" (posisi sementara, hanya untuk pan/zoom visual) — ditandai di daftar peta dengan label **"Denah (belum dikalibrasi)"** vs **"📐 Terkalibrasi"**.

### 3. Versi 4.0 — Track Recorder (fitur baru)
- Tombol **🛤** di peta membuka panel Track Recorder dengan kontrol **🟢 Start / ⏸ Pause / ▶ Resume / 🔴 Stop**.
- Readout realtime: **Jarak** (dihitung dari rangkaian titik GPS memakai rumus Haversine, `js/track.js`), **Waktu** (mm:ss / hh:mm:ss, memperhitungkan jeda saat di-pause), **Kecepatan** (dari GPS, fallback ke rata-rata jarak/waktu).
- Titik GPS otomatis ditambahkan ke jalur (polyline oranye) selama status "Merekam"; titik dengan pergeseran < 1.5 m diabaikan untuk mengurangi *noise* GPS saat diam.
- Saat **Stop**, muncul modal untuk memberi nama & menyimpan track (atau membuangnya) ke IndexedDB (object store baru: `tracks`).
- Tab **🛤 Track** baru di sidebar menampilkan daftar track tersimpan (nama, jarak, durasi); ketuk untuk menampilkan jalurnya di peta (garis putus-putus biru) & hapus track dari daftar.
- GPS otomatis diaktifkan jika belum aktif saat Start ditekan; GPS tidak bisa dimatikan selama track masih berjalan.

## Fitur V3 — Waypoint
- **Tambah waypoint** dengan dua cara: ketuk tombol 📍 di peta lalu ketuk lokasi yang diinginkan (mode "bidik"), atau tombol **"Tambah Waypoint"** di tab Waypoint (memakai titik tengah peta saat ini)
- **Nama, kategori, deskripsi, foto, warna ikon** — kategori punya warna default (Umum, Titik Batas, Pohon/Vegetasi, Titik Sampel, Bahaya, Camp/Basecamp, Lainnya) dan warna ikon tetap bisa disesuaikan bebas
- **Foto** otomatis dikompres & diubah ke JPEG (maks. 900px, kualitas 72%) lewat `<canvas>` sebelum disimpan ke IndexedDB — semua diproses lokal
- **Tombol "🛰 GPS"** di form waypoint & kalibrasi untuk langsung mengisi Lat/Lon dari posisi GPS aktif saat ini
- **Edit & Hapus**, **Cari waypoint** (nama/kategori/deskripsi), **Tab Peta / Waypoint / Track** di sidebar

## Fitur V2 — GPS & Kompas
- **Posisi GPS realtime** (Geolocation API), ditandai titik denyut di peta + lingkaran akurasi
- **Accuracy, Altitude, Speed** ditampilkan pada panel GPS
- **Heading/Bearing**: kompas magnetometer perangkat (DeviceOrientation API), fallback ke *course* GPS
- **Auto-center / Ikuti GPS**: tombol 🎯

**Catatan:** GPS hanya berfungsi jika perangkat/browser memberi izin akses lokasi dan memiliki penerima GPS. Untuk pengujian akurat, coba dari HP di luar ruangan.

## Fitur V1 — PDF Map Viewer
- **Splash screen** saat aplikasi dibuka, sambil daftar peta tersimpan dimuat dari IndexedDB
- **Impor peta PDF** lewat drag-and-drop atau pilih berkas (`.pdf`), dengan pemilih halaman untuk PDF multi-halaman
- **Penyimpanan lokal permanen** di IndexedDB — berkas tidak hilang saat halaman ditutup/dimuat ulang, dan tidak pernah dikirim ke server
- **Daftar peta** tersimpan, bisa pilih & hapus
- **Viewer peta**: pan & zoom penuh (Leaflet), menampilkan citra hasil render PDF sebagai overlay
- **Skala peta** (metric) di pojok kiri bawah, **Fullscreen**, **Tampilan koordinat** (lat/lon pusat peta & zoom) di status bar bawah

## Cara menjalankan
1. Buka `index.html` langsung di browser (Chrome/Edge/Firefox terbaru), **atau**
2. Jalankan lewat server lokal sederhana (disarankan, agar semua fitur PDF.js/IndexedDB bekerja normal):
   ```
   cd GUSERA-Maps
   python3 -m http.server 8080
   ```
   lalu buka `http://localhost:8080`.
3. Klik **"Tambah Peta (PDF)"**, pilih berkas `.pdf` peta lapangan Anda, pilih halaman jika perlu, lalu **"Impor Peta"**.
4. (Opsional tapi disarankan untuk pekerjaan lapangan) Ketuk **📐 Kalibrasi**, tandai 2 titik dengan koordinat GPS yang diketahui, supaya posisi GPS/waypoint/track selaras dengan peta.
5. Aktifkan **🛰 GPS**, lalu **🛤 Track** untuk mulai merekam jalur survei.

## Struktur proyek
```
GUSERA-Maps/
  index.html          Halaman utama & tata letak UI
  css/style.css        Identitas visual (tema gelap ala instrumen survei lapangan)
  js/
    storage.js         Lapisan penyimpanan lokal (IndexedDB) — object store "maps" (PDF), "waypoints", "tracks" (baru V4)
    pdfmap.js           (baru V4, menggantikan mbtiles.js) Pembaca & perender PDF (PDF.js) — murni di browser
    map.js              Setup peta Leaflet + image overlay PDF + kalibrasi 2-titik + marker/lingkaran GPS + marker waypoint + polyline track
    gps.js              Posisi realtime (Geolocation API) + arah kompas (DeviceOrientation API)
    waypoint.js         Kategori waypoint, kompresi foto, util pencarian
    track.js            (baru V4) Util jarak (Haversine) & format durasi untuk Track Recorder
    measure.js          (baru V5) Util Ukur: jarak, luas, keliling, bearing, azimuth — murni perhitungan
    app.js              Menghubungkan semua modul ke UI: impor PDF, kalibrasi, GPS, waypoint, track recorder, ukur
  maps/                 (opsional) taruh berkas .pdf contoh di sini untuk referensi, tidak dimuat otomatis
```

## Roadmap berikutnya (belum dikerjakan)
- **Versi 6.0 — Import**: GeoJSON, GPX, CSV
- **Versi 7.0 — Export**: GPX, CSV, GeoJSON, Backup ZIP (termasuk ekspor track hasil V4)
- **Versi 8.0 — Project Manager**: multi-proyek, backup/restore
- **Versi 9.0 — UI Profesional**: dark mode lanjutan, animasi, bottom sheet, drawer
