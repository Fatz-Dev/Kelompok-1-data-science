# Sistem Rekomendasi Terdistribusi (PySpark) - Kelompok 1

Repository ini berisi implementasi sistem rekomendasi berbasis konten (*Content-Based Filtering*) menggunakan **TF-IDF Terdistribusi** dengan framework **Apache Spark (PySpark)** di lingkungan Google Colab.

---

## Penjelasan Detail Workflow (Cell 2 - Cell 6)

Berikut adalah penjelasan mendalam mengenai alur kerja sistem dari sisi teknis, fungsi, dan contoh penerapannya:

### **Cell 2: Inisialisasi Spark Session (The Distributed Engine)**
*   **Fungsi**: Membuat *entry point* utama untuk semua fungsi Apache Spark. Tanpa ini, kita tidak bisa menggunakan fitur komputasi terdistribusi.
*   **Cara Kerja**: Spark mengalokasikan RAM dan CPU pada Google Colab untuk membuat "Cluster Lokal". Data akan dibagi menjadi beberapa fragmen (*partitions*) agar bisa diproses secara paralel oleh banyak inti CPU secara bersamaan.
*   **Contoh Sederhana**: Bayangkan Anda memiliki 1.000 buku untuk diperiksa. Daripada diperiksa oleh 1 orang, Spark mempekerjakan 4 orang (inti CPU) untuk memeriksa masing-masing 250 buku secara serentak.

### **Cell 3: Pengunduhan & Ingesti Data (Robust CSV Loading)**
*   **Fungsi**: Mengambil data mentah dari internet dan memasukkannya ke dalam struktur data Spark (*DataFrame*) dengan aman.
*   **Cara Kerja**: Menggunakan parameter `quote='"'` dan `escape='"'`. Ini sangat penting karena dataset film mengandung kolom JSON (seperti genre) yang memiliki banyak tanda koma. Tanpa parameter ini, data akan bergeser (*shifted*) dan rusak.
*   **Contoh Sederhana**: 
    *   *Data Salah*: `Spectre, [{"id": 28, "name": "Action"}]` dibaca sebagai 3 kolom karena ada koma di tengah JSON.
    *   *Data Benar*: Dengan `quote`, Spark tahu bahwa teks di dalam tanda kutip adalah satu kesatuan, sehingga tetap menjadi 2 kolom: [Judul] dan [Genre].

### **Cell 4: Preprocessing Terdistribusi (Spark UDF)**
*   **Fungsi**: Membersihkan teks sinopsis agar "dimengerti" oleh mesin.
*   **Cara Kerja**: Menggunakan **Spark UDF (User Defined Function)**. Fungsi pembersih dikirimkan ke setiap worker di cluster Spark untuk dijalankan pada setiap baris data secara mandiri.
*   **Contoh Sederhana**:
    *   *Input*: "I loved the action scenes in Avatar!"
    *   *Proses*: Kecilkan huruf, hapus tanda baca, hapus kata umum (*the, in, is*), dan ubah ke kata dasar (*loved* -> *love*).
    *   *Output*: "love action scene avatar"

### **Cell 5: Pipeline TF-IDF (Vektorisasi Numerik)**
*   **Fungsi**: Mengubah teks bersih menjadi koordinat angka (Vektor).
*   **Cara Kerja**:
    1.  **HashingTF**: Menghitung berapa kali sebuah kata muncul dalam satu sinopsis.
    2.  **IDF**: Mengidentifikasi seberapa unik kata tersebut. Kata yang muncul di hampir semua film (seperti "movie") diberi skor rendah. Kata yang unik (seperti "Galaksi") diberi skor tinggi.
*   **Contoh Sederhana**: Pada film *Star Wars*, kata "Jedis" akan mendapatkan skor sangat tinggi karena sangat jarang muncul di film lain, sehingga menjadi ciri khas utama film tersebut.

### **Cell 6: Engine Rekomendasi (Similarity Parallel)**
*   **Fungsi**: Menentukan tingkat kemiripan antar film dan memberikan hasil terbaik.
*   **Cara Kerja**: Menggunakan **Cosine Similarity** melalui perhitungan *Dot Product*. Spark membandingkan vektor film yang Anda cari dengan seluruh vektor film di dataset (misalnya 5.000 film) secara paralel.
*   **Penjelasan Output & Faktor Similarity**:
    Setelah menjalankan fungsi `get_recommendations`, sistem menampilkan tabel dengan kolom `title` dan `similarity`. Skor kemiripan tersebut didasarkan pada faktor-faktor berikut:
    
    1.  **Bobot Kata Unik (TF-IDF Value)**:
        *   Similarity **BUKAN** hanya berdasarkan jumlah kata yang sama.
        *   Sistem memberikan bobot lebih tinggi pada kata-kata langka/unik. Jika dua film sama-sama memiliki kata "Dinosaur", skor kemiripannya akan jauh lebih tinggi daripada jika mereka sama-sama memiliki kata "Man".
    2.  **Kecocokan Konteks Sinopsis**:
        *   Skor ini mencerminkan seberapa dekat topik atau genre film tersebut berdasarkan deskripsi teksnya. Film dengan skor similarity tinggi biasanya memiliki tema, latar tempat, atau konflik yang serupa.
    3.  **Arah Vektor (Cosine Angle)**:
        *   Secara matematis, setiap film dianggap sebagai sebuah "titik" dalam ruang dimensi tinggi (5.000 dimensi). 
        *   Skor similarity mendekati **1.0** berarti "arah" konten kedua film tersebut hampir identik.
        *   Skor mendekati **0.0** berarti konten kedua film tersebut tidak memiliki keterkaitan kata kunci sama sekali.
    4.  **Normalisasi Panjang Teks**:
        *   Sistem sudah menormalkan panjang sinopsis. Jadi, film dengan sinopsis sangat panjang tidak akan secara otomatis dianggap "lebih mirip" daripada film dengan sinopsis pendek. Yang dihitung adalah **kualitas dan relevansi** kata kuncinya.

*   **Contoh Nyata**:
    *   Jika Anda mencari **"Spectre"**, sistem akan memberikan skor tinggi pada film yang mengandung kata-kata seperti: *James Bond, spy, secret agent, MI6, terrorist organization*. 
    *   Meskipun film lain memiliki ribuan kata, jika tidak mengandung kata kunci spesifik tersebut, skor similarity-nya akan tetap rendah.


---

## Cara Menjalankan
1. Pastikan Anda memiliki akun Google untuk mengakses [Google Colab](https://colab.research.google.com/).
2. Buka file `.ipynb` di Colab.
3. Jalankan seluruh cell secara berurutan (**Runtime -> Run all**).
4. Masukkan judul film/buku pada cell terakhir untuk melihat hasil rekomendasi.
