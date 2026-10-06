Salin seluruh blok kode di bawah ini dan paste (tempel) di terminal Termux kamu. Perintah ini akan otomatis membuat file opencodetermux.md yang siap kamu push ke GitHub:
cat << 'EOF' > opencodetermux.md
# Panduan Instalasi OpenCode di Termux (Android aarch64)

Dokumentasi ini berisi langkah-langkah untuk menginstal **OpenCode** secara manual di Termux. Metode ini sangat berguna untuk mengatasi dua masalah umum yang sering terjadi di perangkat Android:
1. Gagal saat menjalankan `npm install -g opencode-termux` karena sistem Android memblokir eksekusi *binary* dari direktori `tmp`.
2. Muncul error `Bad system call` saat menjalankan `opencode` akibat filter keamanan *seccomp* Android yang memblokir *syscall* dari library `musl`.

Dengan menggunakan metode manual dan bantuan `proot`, kita bisa menjalankan OpenCode dengan lancar.

## Persyaratan
Pastikan perangkat Android kamu menggunakan arsitektur `aarch64`. Kamu bisa mengeceknya dengan menjalankan perintah:
```bash
uname -m

Langkah-langkah Instalasi
1. Install Dependencies
Perbarui repositori Termux dan install semua paket yang dibutuhkan, termasuk nodejs dan proot.
pkg update -y
pkg install curl tar patchelf coreutils ca-certificates resolv-conf proot nodejs -y

2. Unduh dan Ekstrak Binary OpenCode
Buat direktori khusus untuk OpenCode, kemudian unduh file release terbaru (dalam contoh ini v1.18.34-0) dan atur izin eksekusinya.
mkdir -p ~/.opencode
cd ~/.opencode
curl -L -o release.tar.gz "[https://github.com/C04-wq/opencode-termux/releases/download/v1.18.34-0/opencode-termux-aarch64.tar.gz](https://github.com/C04-wq/opencode-termux/releases/download/v1.18.34-0/opencode-termux-aarch64.tar.gz)"
tar -xzf release.tar.gz
chmod +x opencode ld-musl-aarch64.so.1

3. Patch Interpreter dan Kelabui NPM
Karena kita tidak menggunakan script instalasi otomatis, kita perlu mengaitkan interpreter secara manual menggunakan patchelf. Selain itu, buat file penanda versi agar NPM mengira instalasi awal sudah berhasil.
patchelf --set-interpreter "$HOME/.opencode/ld-musl-aarch64.so.1" "$HOME/.opencode/opencode"
echo "1.18.34-0" > ~/.opencode/.opencode-termux-version

4. Install via NPM
Sekarang, jalankan instalasi via NPM. Script bawaan akan mendeteksi file penanda yang kita buat pada Langkah 3 dan langsung melewati proses download yang sering menyebabkan error.
npm install -g opencode-termux

5. Buat Alias Proot (Fix "Bad system call")
Untuk menghindari error Bad system call yang disebabkan oleh limitasi syscall Android, OpenCode harus dijalankan di dalam lingkungan proot. Agar tidak perlu mengetik proot setiap saat, buat alias di .bashrc:
echo "alias opencode='proot -0 opencode'" >> ~/.bashrc
source ~/.bashrc

6. Verifikasi Instalasi
Cek apakah OpenCode sudah terpasang dan berjalan dengan baik dengan mengecek versinya:
opencode --version

Jika terminal menampilkan angka versi (misalnya 1.18.34), maka instalasi telah berhasil!
Catatan: Pastikan untuk menyesuaikan link download (URL) dengan versi rilis terbaru dari repositori asli OpenCode Termux jika ada pembaruan di masa mendatang.
EOF

Setelah menekan `Enter`, file `opencodetermux.md` akan langsung jadi di folder tempat kamu berada sekarang. Kamu bisa mengeceknya dengan mengetik `ls`, lalu langsung *commit* dan *push* ke repositori GitHub kamu.

