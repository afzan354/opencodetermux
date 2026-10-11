# Panduan Instalasi OpenCode di Termux (Android aarch64)

Dokumentasi ini berisi langkah-langkah untuk menginstal **OpenCode** secara manual di Termux. Metode ini berguna untuk mengatasi dua masalah umum di perangkat Android:

1. Gagal saat menjalankan `npm install -g opencode-termux` karena Android memblokir eksekusi *binary* dari direktori `tmp`.
2. Muncul error `Bad system call` saat menjalankan `opencode` akibat filter keamanan *seccomp* Android yang memblokir *syscall* dari library `musl`.

Dengan metode manual dan bantuan `proot`, OpenCode bisa berjalan dengan lancar.

## Persyaratan

Pastikan perangkat Android menggunakan arsitektur `aarch64`. Cek dengan perintah:

```bash
uname -m
```

Hasilnya harus `aarch64`.

## Langkah-langkah Instalasi

### 1. Install Dependencies

Perbarui repositori Termux:

```bash
pkg update -y
```

Install semua paket yang dibutuhkan (termasuk `nodejs` dan `proot`):

```bash
pkg install curl tar patchelf coreutils ca-certificates resolv-conf proot nodejs -y
```

### 2. Unduh dan Ekstrak Binary OpenCode

Buat direktori khusus lalu masuk ke dalamnya:

```bash
mkdir -p ~/.opencode
cd ~/.opencode
```

Unduh file release (contoh ini v1.18.34-0):

```bash
curl -L -o release.tar.gz "https://github.com/C04-wq/opencode-termux/releases/download/v1.18.34-0/opencode-termux-aarch64.tar.gz"
```

Ekstrak file:

```bash
tar -xzf release.tar.gz
```

Atur izin eksekusi:

```bash
chmod +x opencode ld-musl-aarch64.so.1
```

### 3. Patch Interpreter dan Kelabui NPM

Kaitkan interpreter secara manual menggunakan `patchelf`:

```bash
patchelf --set-interpreter "$HOME/.opencode/ld-musl-aarch64.so.1" "$HOME/.opencode/opencode"
```

Buat file penanda versi agar NPM mengira instalasi awal sudah berhasil:

```bash
echo "1.18.34-0" > ~/.opencode/.opencode-termux-version
```

### 4. Install via NPM

Script bawaan akan mendeteksi file penanda dari Langkah 3 dan melewati proses download yang sering menyebabkan error:

```bash
npm install -g opencode-termux
```

### 5. Buat Alias Proot (Fix "Bad system call")

Agar tidak terkena error `Bad system call`, OpenCode harus dijalankan di dalam `proot`. Buat alias di `.bashrc` supaya tidak perlu mengetik `proot` setiap saat:

```bash
echo "alias opencode='proot -0 opencode'" >> ~/.bashrc
```

Muat ulang konfigurasi:

```bash
source ~/.bashrc
```

### 6. Verifikasi Instalasi

Cek versi OpenCode:

```bash
opencode --version
```

Jika terminal menampilkan nomor versi (misalnya `1.18.34`), instalasi berhasil!

---

> **Catatan:** Sesuaikan URL download pada Langkah 2 (dan angka versi pada Langkah 3) dengan versi rilis terbaru dari repositori OpenCode Termux jika ada pembaruan.
