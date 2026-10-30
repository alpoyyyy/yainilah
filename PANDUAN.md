# Hitung Stok Kamera: Panduan

Aplikasi web untuk menghitung barang di rak secara langsung. Arahkan kamera HP ke rak, barang terdeteksi dan dihitung otomatis per jenis. Teknisi memeriksa angkanya, lalu menyimpan hasilnya.

## Isi folder

```
hitung-kamera/
├── index.html                    ← aplikasinya (semua dalam satu file)
├── manifest.json, icon.svg       ← supaya bisa "dipasang" di layar utama HP
├── model/                        ← taruh best.onnx hasil training di sini
└── training/latih_model_colab.py ← script melatih model di Google Colab
```

---

## Tahap 1: Coba sekarang dengan mode Demo (5 menit)

Mode Demo memakai model umum yang sudah bisa mengenali **botol, gelas, gunting, buku, HP**, dan sejenisnya. Kaleng spray seperti Rexco biasanya terbaca sebagai "bottle". Mode ini cukup untuk membuktikan konsep hitung otomatis.

Kamera di browser **hanya bisa menyala lewat alamat `https://` atau `localhost`**. Membuka file langsung dari folder HP tidak akan berhasil.

**Cara termudah, pakai GitHub Pages (gratis):**
1. Buat akun di github.com, lalu klik **New repository** (misalnya `hitung-stok`, Public).
2. Klik **Add file → Upload files**, lalu upload semua isi folder ini.
3. Buka **Settings → Pages**, pilih Source: *Deploy from a branch*, Branch: `main`, folder `/root`, lalu klik **Save**.
4. Tunggu 1–2 menit. Aplikasi bisa dibuka di `https://<username>.github.io/hitung-stok/`.
5. Buka alamat itu di Chrome HP, izinkan kamera, lalu tekan **Nyalakan kamera**.
6. Supaya muncul ikon di layar utama: buka menu Chrome (⋮), lalu pilih **Tambahkan ke layar utama**.

> Repository GitHub Public bisa dilihat siapa saja. **Jangan upload foto depo atau data perusahaan ke sana.** Cukup aplikasinya saja. Untuk pemakaian resmi, minta IT memasangnya di server internal (lihat Tahap 4).

**Alternatif di laptop (tanpa internet publik):** buka terminal di folder ini, jalankan `python -m http.server 8000`, lalu buka `http://localhost:8000` di Chrome laptop dan pakai webcam.

---

## Tahap 2: Kumpulkan dan beri label foto barang depo

Agar aplikasi mengenali barang spesifik (Rexco, WD-40, kardus Novax, lem, dll), model harus dilatih dengan foto barang aslinya.

**Minta izin atasan dulu sebelum memotret area depo.**

### Mengambil foto
- **50–150 foto per jenis barang.** Lebih banyak lebih bagus.
- Foto langsung di **rak aslinya**, dalam kondisi sehari-hari.
- Variasikan: sudut kiri, kanan, dan depan; jarak dekat dan jauh; lampu terang dan redup; barang sedikit dan penuh.
- Satu foto boleh berisi banyak barang. Justru itu bagus untuk melatih kemampuan menghitung.
- Sertakan juga foto rak tanpa barang target, supaya model belajar membedakan.

### Memberi label (pakai Roboflow, gratis)
1. Daftar di **roboflow.com**, lalu buat project baru dengan tipe **Object Detection**.
2. Upload semua foto.
3. Di setiap foto, **gambar kotak** di setiap barang dan pilih nama kelasnya. Pakai nama tanpa spasi, misalnya `rexco_25`, `wd40`, `kardus_novax`.
4. Pastikan **semua** barang di foto diberi kotak. Barang yang terlewat akan membuat model ikut melewatkannya.
5. Klik **Generate dataset**, aktifkan augmentasi (flip dan brightness), lalu **Export** dengan format **YOLOv8** dan simpan sebagai zip.

> Kalau data foto tidak boleh keluar perusahaan, pakai **CVAT** atau **Label Studio** yang bisa dipasang di komputer lokal.

---

## Tahap 3: Latih model (Google Colab, gratis)

1. Buka **colab.research.google.com** dan buat notebook baru.
2. Pilih **Runtime → Change runtime type → T4 GPU**.
3. Salin isi `training/latih_model_colab.py` per bagian (SEL 1 sampai SEL 6) ke sel notebook, lalu jalankan berurutan.
4. Proses training memakan waktu sekitar 20–60 menit, tergantung jumlah foto.
5. Download file **`best.onnx`**, lalu taruh di folder `model/` dan upload ulang ke GitHub.
6. Di aplikasi, buka tab **Jenis barang**:
   - Ubah Sumber model menjadi **Model sendiri**.
   - Isi daftar jenis barang **sesuai urutan** hasil SEL 6. Contoh:
     ```
     rexco_25 = Rexco 25 (kaleng)
     wd40 = WD-40 (kaleng)
     kardus_novax = Kardus Novax
     ```
   - Tekan **Terapkan**.

Menambah jenis barang baru di kemudian hari: tambahkan foto dan label untuk kelas baru di Roboflow, latih ulang, lalu ganti `best.onnx` dan daftar jenis barang di aplikasi.

### Kalau hasilnya kurang bagus
| Gejala | Solusi |
|---|---|
| Barang sering terlewat | Tambah foto, turunkan ambang keyakinan, periksa apakah ada barang yang lupa diberi label |
| Salah jenis (Rexco terbaca WD-40) | Tambah foto close-up kedua barang, naikkan ambang keyakinan |
| Aplikasi lambat di HP | Pakai model `yolo11n`, export dengan `imgsz=480`, lalu isi 480 di aplikasi |
| Angka naik-turun | Pegang HP stabil 2–3 detik, lalu tekan **Kunci hitungan** |

---

## Tahap 4: Dipakai resmi

- Minta IT memasang folder ini di **server internal** dengan HTTPS, sehingga data dan model tidak keluar ke layanan publik.
- Riwayat hitungan saat ini masih tersimpan di HP masing-masing. Untuk dipakai satu tim, perlu ditambah database di server dan disambungkan ke aplikasi stok.
- Bisa juga dibungkus menjadi APK Android (misalnya dengan Capacitor) dan dibagikan ke HP kerja lewat IT.

## Batasan yang perlu dijelaskan saat presentasi
- Kamera hanya menghitung barang yang **terlihat**. Barang di belakang barang lain atau di dalam kardus tidak terhitung, jadi barang perlu disusun menghadap depan.
- Barang kecil yang bertumpuk (baut, mur, washer) lebih akurat dihitung dengan **timbangan** (berat total ÷ berat per pcs). Majun dihitung per kg.
- AI **membantu** menghitung. Angka akhir tetap **diperiksa dan dikonfirmasi teknisi** sebelum disimpan. Kolom "Hitung AI" di riwayat menunjukkan selisihnya, dan bisa dipakai untuk mengukur akurasi model.
