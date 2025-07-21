# UAS Keamanan Komputer

## Identitas
- Nama: Fredieq Nur Muhammad
- NIM: 221080200019
- Kelas: B1 Informatika
- Repo GitHub: [link]

---

## Bagian A – Bug Fixing JWT REST API

### Bug 1: Token tetap aktif setelah logout
**Penjelasan:**  
1. Tidak ada pembatasan waktu token dibuat, sehingga membuat token tetap valid. 

2. tidak ada pembatasan akses endpoint membuat semua user bisa mengakses siapa saja.

3. data dari role yang lain, misal user biasa, dapat mengakses data yang dimiliki oleh admin.

**Solusi:**  
Penjelasan bug :  
API tidak menyimpan token yang aktif (sessionless), sehingga saat logout tidak ada cara memverifikasi apakah token itu sudah "dicabut" atau tidak. Akibatnya, meskipun user sudah logout, token sebelumnya tetap bisa digunakan untuk mengakses endpoint.

Solusi bug :
Tambahkan token blacklist system atau simpan token yang aktif di database/Redis dan validasi token saat request masuk. Ketika user logout, hapus token dari daftar aktif.


## Bagian B – Simulasi Serangan dan Solusi

### Jenis Serangan: Broken Access Control  
**Simulasi Postman:**  
Serangan terjadi karena tidak adanya pembatasan akses berdasarkan identitas pengguna. Ini memungkinkan user biasa mengubah data user lain (termasuk admin) dengan memanipulasi user_id di body request. Pengguna biasa mengubah user_id menjadi ID admin.

POST /api/users/update HTTP/1.1
Content-Type: application/json
Authorization: Bearer <USER_TOKEN>

{
  "user_id": "admin_user_id",  // <- Diubah oleh user biasa
  "email": "hacked@example.com",
  "role": "superadmin"  
}


**Solusi Implementasi:**  
Kita perlu menerapkan Ownership Validation : Hanya admin atau pemilik akun yang bisa mengubah data.
Jika data tidak cocok dan bukan admin, maka akan tolak permintaan.

// Middleware untuk validasi kepemilikan akun
const checkUserOwnership = (req, res, next) => {
  const requestUserId = req.body.user_id || req.params.user_id;
  const loggedInUserId = req.user.id;  // Diambil dari JWT/auth middleware
  const loggedInRole = req.user.role;

  // Jika bukan admin dan mencoba mengubah data orang lain → BLOCK
  if (loggedInRole !== "admin" && requestUserId !== loggedInUserId) {
    return res.status(403).json({ error: "Unauthorized: You can only modify your own data." });
  }

  next();  // Lanjut jika valid
};

module.exports = checkUserOwnership;


---

## Bagian C – Refleksi Teori & Etika

### 1. CIA Triad dalam Keamanan Informasi  
... 
1. Confidentiality (Kerahasiaan) : Menjamin bahwa informasi hanya dapat diakses oleh pihak yang berwenang. 
2. Integrity (Integritas) : Memastikan bahwa informasi tetap akurat dan tidak diubah tanpa izin. 
3. Availability (Ketersediaan) : Menjamin bahwa informasi dan sistem selalu tersedia untuk pengguna yang berwenang ketika dibutuhkan. 

### 2. UU ITE yang relevan 
---
1. Pasal 30 : tentang larangan akses ilegal terhadap sistem elektronik. Setiap orang yang dengan sengaja dan tanpa hak mengakses sistem elektronik milik orang lain dapat dikenakan sanksi pidana.
2. Pasal 32 : tentang larangan penyebaran informasi elektronik yang dapat merugikan pihak lain. 

### 3. Pandangan Al-Qur'an  
- Surah Al-Baqarah: 205  
pada ayat ini menekankan pentingnya menjaga dan tidak merusak apa yang ada di bumi, termasuk sistem dan infrastruktur yang ada. Dalam konteks cyber ethics, merusak sistem informasi atau data dapat dianggap sebagai tindakan yang tidak sesuai dengan ajaran Islam, karena dapat merugikan orang lain dan menciptakan kerusakan. 

### 4. Etika Cyber dan Kejujuran  
--- 
menurut saya dalam dunia cybersecurity modern, penerapan nilai kejujuran dan amanah sangat penting. kejujuran dalam pengelolaan data dan informasi berarti bahwa individu dan organisasi harus transparan dalam  mengumpulkan, menyimpan, dan menggunakan data. lalu untuk amanah, berkaitan dengan tanggung jawab untuk melindungi data dan informasi yang dipercayakan kepada kita. Dengan menerapkan nilai-nilai tersebut, kita dapat membangun kepercayaan antara pengguna dan penyedia layanan.