# LAPORAN PENGERJAAN ASSESMENT TENGAH SEMESTER

---

* **Nama**: Faidil Zahpar
* **NPM**: 714240007
* **Mata Kuliah**: Pemrograman Web Service
* **Modul**: 7 — Pengumuman & Kategori

---

## 1. Deskripsi Modul
Modul **Pengumuman & Kategori** adalah papan pengumuman digital di kampus yang bisa digunakan oleh:
1. **Pengunjung (Mahasiswa/Dosen)**: Bisa langsung membaca pengumuman terbaru, mencari berdasarkan kategori atau tanggal tertentu, dan membuka file lampiran (seperti foto atau dokumen).
2. **Pengelola (Admin Akademik)**: Bisa membuat, mengubah, atau menghapus pengumuman dan kategori, serta mengupload file pendukung.

---

## 2. Tech Stack
* **Backend**: Go + Go Fiber v2 
* **Database**: MongoDB Atlas
* **Frontend**: HTML5 + Vanilla CSS + JavaScript 
* **Hosting**: Alwaysdata 

---

## 3. Skema Data (Models)

### **3.1 Model Pengumuman**
Model untuk menyimpan data pengumuman akademik beserta tautan lampiran berkas.
```go
type Pengumuman struct {
	ID        primitive.ObjectID `bson:"_id,omitempty" json:"id,omitempty"`
	Judul     string             `bson:"judul" json:"judul"`
	Isi       string             `bson:"isi" json:"isi"`
	Kategori  string             `bson:"kategori" json:"kategori"`
	Tanggal   time.Time          `bson:"tanggal" json:"tanggal"`
	Penulis   string             `bson:"penulis,omitempty" json:"penulis,omitempty"`
	FileUrl   string             `bson:"file_url,omitempty" json:"file_url,omitempty"`
	FileType  string             `bson:"file_type,omitempty" json:"file_type,omitempty"`
}
```

### **3.2 Model Kategori**
Model untuk mengelompokkan jenis informasi (misal: "Akademik", "Ormawa", "Keuangan").
```go
type Kategori struct {
	ID   primitive.ObjectID `bson:"_id,omitempty" json:"id,omitempty"`
	Nama string             `bson:"nama" json:"nama"`
}
```

---

## 4. Dokumentasi API (Endpoints)

Format standard response sukses dari server:
```json
{
  "status": "success",
  "data": { ... }
}
```

Format standard response error dari server:
```json
{
  "status": "error",
  "message": "Deskripsi detail kesalahan"
}
```

### **4.1 Endpoint Pengumuman**

#### **A. Ambil Semua Pengumuman (Tampilan Website)**
* **Endpoint**: `GET /pengumuman-data`
* **Kegunaan**: Digunakan oleh website kita untuk mengambil seluruh daftar pengumuman terbaru saat filter **"Semua"** diklik.
* **Response Sukses (200 OK)**:
  ```json
  {
    "status": "success",
    "data": [
      {
        "id": "664c8d50fcf86cd799439001",
        "judul": "Pemberitahuan Ujian Akhir Semester Ganjil",
        "isi": "Ujian Akhir Semester (UAS) akan diselenggarakan pada tanggal...",
        "kategori": "Akademik",
        "tanggal": "2026-05-21T20:10:00Z",
        "penulis": "Admin Akademik",
        "file_url": "/pengumuman/uploads/1716301800_uas_jadwal.pdf",
        "file_type": "pdf"
      }
    ]
  }
  ```

#### **B. Ambil Semua Pengumuman (Standar Sistem Kampus)**
* **Endpoint**: `GET /pengumuman`
* **Kegunaan**: Pintu akses resmi untuk mendapatkan seluruh data pengumuman.
* **Response Sukses (200 OK)**: Sama persis dengan data pada `GET /pengumuman-data`.

#### **C. Ambil Detail Pengumuman Spesifik**
* **Endpoint**: `GET /pengumuman/:id`
* **Kegunaan**: Mengambil isi berita lengkap beserta lampiran file dari **satu pengumuman tertentu** ketika pembaca mengeklik pengumuman tersebut.
* **Kunci Pengenal (`id`)**: Kode unik 24-karakter pengumuman (contoh: `664c8d50fcf86cd799439001`).
* **Response Sukses (200 OK)**:
  ```json
  {
    "status": "success",
    "data": {
      "id": "664c8d50fcf86cd799439001",
      "judul": "Pemberitahuan Ujian Akhir Semester Ganjil",
      "isi": "Ujian Akhir Semester (UAS)...",
      "kategori": "Akademik",
      "tanggal": "2026-05-21T20:10:00Z",
      "penulis": "Admin Akademik",
      "file_url": "/pengumuman/uploads/1716301800_uas_jadwal.pdf",
      "file_type": "pdf"
    }
  }
  ```
* **Response Error (Jika ID tidak terdaftar / salah kode)**:
  ```json
  {
    "status": "error",
    "message": "Pengumuman tidak ditemukan"
  }
  ```

#### **D. Tambah Pengumuman Baru**
* **Endpoint**: `POST /pengumuman`
* **Request Header**: `Content-Type: application/json`
* **Request Body**:
  ```json
  {
    "judul": "Rapat Koordinasi Himpunan Mahasiswa",
    "isi": "Diharapkan seluruh pengurus himpunan hadir pada...",
    "kategori": "Kegiatan",
    "penulis": "HIMA D4 TI",
    "file_url": "/pengumuman/uploads/1716301800_hima.png",
    "file_type": "image"
  }
  ```
* **Response Sukses (200 OK)**: Mengembalikan objek pengumuman baru yang berhasil dibuat lengkap dengan `id` dan `tanggal` server.

#### **E. Update Pengumuman**
* **Endpoint**: `PUT /pengumuman/:id`
* **Path Parameter**: `id` (Hexadecimal ObjectID)
* **Request Body**: Struktur JSON sama seperti `POST /pengumuman` dengan modifikasi atribut.
* **Response Sukses (200 OK)**: Mengembalikan objek pengumuman yang telah diperbarui.

#### **F. Hapus Pengumuman**
* **Endpoint**: `DELETE /pengumuman/:id`
* **Path Parameter**: `id` (Hexadecimal ObjectID)
* **Response Sukses (200 OK)**:
  ```json
  {
    "status": "success",
    "data": {
      "message": "Pengumuman berhasil dihapus"
    }
  }
  ```

---

### **4.2 Endpoint Kategori**

#### **A. Ambil Semua Kategori**
* **Endpoint**: `GET /kategori`
* **Deskripsi**: Mengambil seluruh daftar kategori yang terdaftar di database.
* **Response Sukses (200 OK)**:
  ```json
  {
    "status": "success",
    "data": [
      {
        "id": "664c8d50fcf86cd79943900a",
        "nama": "Akademik"
      },
      {
        "id": "664c8d50fcf86cd79943900b",
        "nama": "Penting"
      }
    ]
  }
  ```

#### **B. Tambah Kategori Baru**
* **Endpoint**: `POST /kategori`
* **Request Body**:
  ```json
  {
    "nama": "Kemahasiswaan"
  }
  ```
* **Response Sukses (200 OK)**: Mengembalikan objek kategori baru yang berhasil dibuat.

#### **C. Hapus Kategori**
* **Endpoint**: `DELETE /kategori/:id`
* **Path Parameter**: `id` (Hexadecimal ObjectID)
* **Response Sukses (200 OK)**:
  ```json
  {
    "status": "success",
    "data": {
      "message": "Kategori berhasil dihapus"
    }
  }
  ```

#### **D. Filter Pengumuman Berdasarkan Kategori**
* **Endpoint**: `GET /pengumuman/kategori/:nama`
* **Deskripsi**: Mengambil semua pengumuman yang sesuai dengan nama kategori secara *case-insensitive*.
* **Path Parameter**: `nama` (Nama kategori, misal: `Akademik`)
* **Response Sukses (200 OK)**: Array berisi pengumuman pada kategori tersebut terurut dari tanggal terbaru.

---

### **4.3 Endpoint Media & Upload**

#### **Unggah File Lampiran**
* **Endpoint**: `POST /pengumuman/upload`
* **Request Header**: `Content-Type: multipart/form-data`
* **Form Data**: `file` (Berkas binary)
* **Response Sukses (200 OK)**:
  ```json
  {
    "status": "success",
    "data": {
      "file_type": "pdf",
      "file_url": "/pengumuman/uploads/1716302821_664c8d50fcf86cd799439001.pdf",
      "filename": "Panduan_UAS.pdf"
    }
  }
  ```
* **Response Error (400 Bad Request — Contoh File Terlalu Besar)**:
  ```json
  {
    "status": "error",
    "message": "Ukuran file terlalu besar. Maksimum: 5MB"
  }
  ```

---

## 5. Dokumentasi Screenshot Aplikasi
Berikut adalah beberapa dokumentasi tangkapan layar (screenshot) jalannya aplikasi Papan Pengumuman Digital & Kelola Pengumuman:

### **5.1 Halaman Kelola Pengumuman (Admin)**
1. **Daftar Pengumuman**:
   ![Daftar Pengumuman](UI/Screenshot%202026-05-21%20at%2020-51-02%20Kelola%20Pengumuman%20—%20SIAKAD.png)

2. **Form Tambah & Edit Pengumuman**:
   ![Form Tambah & Edit Pengumuman](UI/Screenshot%202026-05-21%20at%2020-41-51%20Kelola%20Pengumuman%20—%20SIAKAD.png)

3. **Kelola Kategori**:
   ![Kelola Kategori](UI/Screenshot%202026-05-21%20at%2020-41-40%20Kelola%20Pengumuman%20—%20SIAKAD.png)

---

### **5.2 Halaman Board Pengumuman (Pengunjung)**
4. **Tampilan Board Utama**:
   ![Tampilan Board Utama](UI/Screenshot%202026-05-21%20at%2020-49-26%20Papan%20Pengumuman%20Digital%20—%20SIAKAD.png)

5. **Detail Informasi & Lampiran**:
   ![Detail Informasi & Lampiran](UI/Screenshot%202026-05-21%20at%2021-38-32%20Papan%20Pengumuman%20Digital%20—%20SIAKAD.png)

---

## 6. Kesimpulan & Penutup
Modul **7 — Pengumuman & Kategori** telah selesai dikembangkan dan berfungsi dengan baik dari sisi backend maupun frontend. Fitur pengelolaan pengumuman (CRUD), pencarian berdasarkan waktu/kategori, unggah dan pratinjau file telah berfungsi dengan baik.

Seluruh pengerjaan modul ini dapat diakses melalui repositori berikut: https://github.com/faidilzahpar/siakad-2A

