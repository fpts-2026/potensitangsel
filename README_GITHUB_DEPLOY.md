# Panduan Deploy Website FPTS ke GitHub Pages

Panduan ini menyiapkan website FPTS agar bisa dipublikasikan melalui GitHub Pages dan domain Squarespace bisa diarahkan dari Google Sites ke GitHub.

## File Deploy yang Ditambahkan

- `Index.html`  
  File utama website. Saat deploy, workflow GitHub Actions otomatis menyalinnya menjadi `index.html` di artifact GitHub Pages, karena server GitHub Pages mencari entry lowercase `index.html`.

- `404.html`  
  Fallback jika pengunjung membuka path seperti `/sigap` atau `/admin-console`. Pengunjung akan diarahkan kembali ke website utama.

- `.nojekyll`  
  Menonaktifkan proses Jekyll agar GitHub Pages menyajikan file statis apa adanya.

- `.github/workflows/deploy-github-pages.yml`  
  Workflow GitHub Actions untuk deploy otomatis ke GitHub Pages. Workflow hanya memasukkan file website ke artifact Pages: `Index.html`, hasil salinan `index.html`, `404.html`, `.nojekyll`, dan file gambar.

- `CNAME.example`  
  Contoh isi file `CNAME` jika ingin memakai custom domain. Jangan dipakai langsung sebelum diganti dengan domain asli.

## Catatan Backend Penting

GitHub Pages hanya bisa menyajikan website statis. Backend Google Apps Script di `Code.gs` memakai `google.script.run`, sehingga fitur penyimpanan permanen ke Google Sheets/Drive berjalan penuh ketika website dibuka sebagai Apps Script Web App.

Jika website dibuka dari GitHub Pages, `google.script.run` tidak tersedia dan beberapa fitur akan memakai fallback lokal browser (`localStorage`). Artinya data tidak menjadi database pusat. Untuk produksi penuh di GitHub Pages, backend perlu dibuat sebagai API terpisah, misalnya Apps Script API bridge, Cloud Run, Firebase, atau backend lain yang mendukung CORS dan autentikasi.

## Deploy ke GitHub Pages

### 1. Buat Repository

1. Buka GitHub.
2. Klik `New repository`.
3. Gunakan nama repository, contoh: `fpts-website`.
4. Jika memakai GitHub Free, repository untuk Pages biasanya perlu `Public`.
5. Buat repository.

Referensi GitHub: GitHub Pages membutuhkan file entry seperti `index.html`, `index.md`, atau `README.md` di source publikasi.

### 2. Upload File

Upload file/folder berikut ke repository:

- `Index.html`
- `404.html`
- `.nojekyll`
- `.github/workflows/deploy-github-pages.yml`
- `missing-original.png`
- `sigap-original.png`
- `CNAME.example`
- `README_GITHUB_DEPLOY.md`

Opsional untuk backup/source:

- `Code.gs`
- `appsscript.json`
- `README_INSTALL.txt`

Catatan keamanan: jika repository publik, semua file yang diupload dapat dilihat publik. Segera ganti password Admin Console di deployment Apps Script dan jangan upload file berisi token/API key pribadi.

### 3. Aktifkan GitHub Pages

1. Buka repository di GitHub.
2. Masuk ke `Settings`.
3. Pilih `Pages`.
4. Pada bagian `Build and deployment`, pilih `Source: GitHub Actions`.
5. Push/upload file ke branch `main`.
6. Buka tab `Actions` dan tunggu workflow `Deploy GitHub Pages` selesai.
7. Setelah selesai, buka `Settings -> Pages -> Visit site`.

URL sementara biasanya:

```text
https://USERNAME.github.io/NAMA-REPOSITORY/
```

Jika repository bernama `USERNAME.github.io`, URL-nya:

```text
https://USERNAME.github.io/
```

## Memindahkan Domain dari Google Sites ke GitHub Pages

Ini bukan transfer registrar. Domain tetap berada di Squarespace, yang dipindahkan adalah arah DNS dari Google Sites ke GitHub Pages.

Rekomendasi struktur:

- Domain utama GitHub Pages: `www.domain-anda.com`
- Domain apex/root `domain-anda.com` ikut diarahkan ke GitHub Pages agar otomatis redirect ke `www`.

### 1. Pastikan Website GitHub Sudah Hidup

Sebelum mengubah DNS, buka dulu URL sementara GitHub Pages dan pastikan website tampil.

### 2. Set Custom Domain di GitHub

1. Buka repository GitHub.
2. Masuk ke `Settings -> Pages`.
3. Pada `Custom domain`, isi:

```text
www.domain-anda.com
```

4. Klik `Save`.
5. Jangan aktifkan `Enforce HTTPS` dulu jika GitHub masih menunggu DNS valid. Aktifkan setelah DNS terdeteksi.

### 3. Ubah DNS di Squarespace

1. Buka Squarespace.
2. Masuk ke `Domains dashboard`.
3. Klik domain Anda.
4. Buka `DNS -> DNS Settings`.
5. Jika ada record lama Google Sites yang konflik, hapus/ganti:
   - `CNAME` host `www` yang mengarah ke `ghs.googlehosted.com`
   - record forwarding lama ke `sites.google.com`
   - record `A`/`AAAA` lama untuk hosting sebelumnya
6. Jangan hapus record email seperti `MX`, `SPF`, `DKIM`, atau `DMARC` jika email domain masih dipakai.

Tambahkan record berikut.

Untuk `www`:

```text
Type: CNAME
Host: www
Value/Alias Data: USERNAME.github.io
```

Ganti `USERNAME` dengan username/organization GitHub. Jangan tambahkan nama repository di CNAME.

Untuk domain root/apex:

```text
Type: A
Host: @
Value: 185.199.108.153

Type: A
Host: @
Value: 185.199.109.153

Type: A
Host: @
Value: 185.199.110.153

Type: A
Host: @
Value: 185.199.111.153
```

Opsional IPv6:

```text
Type: AAAA
Host: @
Value: 2606:50c0:8000::153

Type: AAAA
Host: @
Value: 2606:50c0:8001::153

Type: AAAA
Host: @
Value: 2606:50c0:8002::153

Type: AAAA
Host: @
Value: 2606:50c0:8003::153
```

Squarespace menyebut custom DNS record punya TTL default 4 jam, jadi perubahan bisa terasa bertahap.

### 4. Verifikasi Domain di GitHub

Sangat disarankan untuk mencegah domain takeover.

1. Di GitHub, buka `Profile photo -> Settings`.
2. Pilih `Pages`.
3. Klik `Add a domain`.
4. Isi domain Anda.
5. GitHub akan memberi TXT record, biasanya dengan host seperti:

```text
_github-pages-challenge-USERNAME
```

6. Tambahkan TXT record itu di DNS Squarespace.
7. Kembali ke GitHub dan klik `Verify`.

### 5. Aktifkan HTTPS

Setelah DNS terbaca di GitHub:

1. Buka repository `Settings -> Pages`.
2. Centang `Enforce HTTPS`.
3. GitHub menyebut opsi HTTPS bisa memerlukan waktu sampai tersedia.

### 6. Lepas dari Google Sites

Setelah GitHub Pages + custom domain berjalan:

1. Buka Google Sites lama.
2. Lepaskan custom domain/mapping jika masih ada.
3. Simpan Google Sites sebagai backup sementara sampai DNS stabil.

## Checklist Cutover

- Website GitHub Pages sudah tampil di URL sementara.
- Custom domain sudah diisi di GitHub Pages.
- DNS `www` sudah CNAME ke `USERNAME.github.io`.
- DNS root `@` sudah memakai 4 A record GitHub Pages.
- Tidak ada record Google Sites yang konflik.
- Email record tidak terhapus.
- Domain sudah diverifikasi di GitHub.
- `Enforce HTTPS` sudah aktif.
- Admin Console Apps Script sudah diganti password dari default.

## Sumber Resmi

- GitHub Docs - Creating a GitHub Pages site: https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site
- GitHub Docs - Managing a custom domain for GitHub Pages: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site
- GitHub Docs - Verifying a custom domain for GitHub Pages: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages
- Squarespace Help - Adding DNS records: https://support.squarespace.com/hc/en-us/articles/360002101888-Adding-DNS-records-to-your-domain
- Squarespace Help - Pointing a Squarespace domain: https://support.squarespace.com/hc/en-us/articles/215744668-Pointing-a-Squarespace-domain
