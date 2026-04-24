## 📦 Semantic Versioning

Project ini sudah menggunakan **Semantic Versioning (SemVer)** yang diotomatisasi menggunakan *semantic-release*.

### 🔢 Format Versi

Versi mengikuti format:

```
MAJOR.MINOR.PATCH
```

Contoh:

* `1.0.0` → rilis awal
* `1.0.1` → perbaikan bug (patch)
* `1.1.0` → penambahan fitur (minor)
* `2.0.0` → perubahan besar / breaking change (major)

---

### ⚙️ Cara Kerja

Versioning dilakukan secara otomatis berdasarkan **commit message**:

| Tipe Commit                | Dampak         |
| -------------------------- | -------------- |
| `fix:`                     | Patch (x.x.+1) |
| `feat:`                    | Minor (x.+1.0) |
| `feat!:` / BREAKING CHANGE | Major (+1.0.0) |

Contoh:

```
feat: tambah dashboard
fix: perbaiki login error
```

---

### 🚀 Proses Release

1. Developer push commit ke branch `main`
2. CI/CD menjalankan semantic-release
3. Sistem otomatis:

   * Menentukan versi baru
   * Membuat tag Git (`vX.X.X`)
   * Update `CHANGELOG.md`
   * (Opsional) update `package.json`
4. Docker image dibuat menggunakan versi tersebut

---

### 🐳 Docker Tagging

Docker image menggunakan version yang dihasilkan:

```
my-react-app:latest
my-react-app:v1.1.0
my-react-app:<commit-sha>
```

---

### ⚠️ Catatan Penting

* Gunakan format commit yang sesuai (`feat`, `fix`, dll)
* Commit tanpa format tersebut **tidak akan memicu release**
* Version tidak diubah manual, semuanya otomatis

---

### 📌 Contoh Alur

```
feat: tambah fitur login
↓
Release otomatis → v1.1.0
↓
Docker image → my-react-app:v1.1.0
```
