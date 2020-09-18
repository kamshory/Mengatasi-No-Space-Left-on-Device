# Mengatasi No Space Left on Device

Pada sistem operasi Linux, ada istilah Inodes. Inodes adalah indexing file dalam sistem operasi Linux. Index ini akan menentukan berapa jumlah file yang dapat disimpan pada senuah direktori atau partisi. Perlu dicatat bahwa jumlah file berbeda dengan total ukuran file.

Sebagai contoh:

Sebuah direktori berisi 100 file yang ukurannya sama besar yaitu 10 kB. Dengan demikian, Inodes untuk direktori tersebut adalah 101 (100 file dan 1 direktori itu sendiri) sedangkan total ukuran file adalah 1000 kB.

Lalu, apa pengaruh Inodes pada sebuah komputer Linux?

Saya mempunyai pengalaman buruk tentang Inodes. Server saya menggunakan sistem operasi Linux dengan server web Apache + PHP. Entah bagaimana PHP tidak menghapus secara otomatis file session yang sudah tidak terpakai lagi. Ditambah dengan setiap request baru yang tidak membawa informasi cookie akan membuat sebuah session baru. Seharusnya ini dapat dihindari dengan memeriksa apakah request tersebut mengandung cookie atau tidak. Tapi sudah terjadi. File session sudah dibuat dengan jumlah yang tidak tanggung yaitu 20.000.000 file lebih. Lalu apa yang terjadi?

Inodes untuk partisi /dev/vda1 mempunyai Inodes sebesar 20.970.944. Akhibatnya, ketika Inodes ini sudah terpakai semua, Linux tidak bisa lagi membuat file baru. Pesan error yang muncul adalah "No space left on device" padahal ruang terpakai baru 49%. 

Kapasitas penyimpanan total adalah 40 GB sehingga masih ada sisa ruang sebesar 20 GB. Namun untuk membuat sebuah file kosong baru atau file baru yang berukuran 1 byte pun tidak bisa. Pesan error yang muncul adalah "No space left on device". Tentu saja bagi orang yang tidak paham seperti saya bingung bahkan sempat berganti akun dan memindahkan semua file dan database ke akun baru.

Tiga bulan kemudian, kejadian ini terulang kembali. Tentu saja saya tidak mau membuat akun baru lagi.

Setelah mencari informasi ke sana kemari, saya baru mengetahui bahwa masalah tersebut terjadi karena Inodes pada partisi di mana saya menyimpan aplikasi dan data sudah penuh. Saya kemudian menghapus puluhan ribu file agar sistem kembali berjalan dengan normal.

Tidak sampai 1 jam, masalah kembali terjadi. Sampai akhirnya diketahui bahwa file yang jumlahnya tidak terkendali adalah file session PHP. Saya menggunakan PHP 5 pada sistem operasi Centos 7. Lokasi file session ada di `/var/lib/php/session`. Tidak mungkin saya menghapus manual satu persatu file tersebut. Menghapus sekaligus file dalam direktori tersebut juga berbahaya apalahi hingga menghapus direktorinya karena kepemilikan dan permission direktori harus disesuaikan dengan pengguna direktori itu sendiri. Jika direktori tersebut terhapus, tentu direktori tersebut harus dibuat lagi secara manual lalu diatur kepemilikan dan permissionnya seperti sedia kala.

Langkah yang saya lakukan adalah menghapus file lama dari session PHP dengan perintah sebagai berikut:

```bash
/usr/bin/find /var/lib/php/session -mindepth 1 -maxdepth 1 -type f -cmin +1440 -print0 -exec rm {} \; >/dev/null 2>&1
```

Di mana `+1440` mengandung arti file yang dimodifikasi lebih dari 24 menit yang lalu. Angka 24 menit ini sama dengan umur default session dan cookie di PHP dan kebetulan sButuh waktu lebih kurang 10 jam untuk membersihkan file session PHP ini karena jumlahnya terlalu banyak.

Untuk mencegah kejadian kehabisan Inodes di masa mendatang, saya membuat sebuah cron job dengan menggunakan crontab.

```bash
crontab -e
```

Adapun isi dari file crontab adalah sebagai berikut:

```bash
0 1 * * * /usr/bin/find /var/lib/php/session -mindepth 1 -maxdepth 1 -type f -cmin +1440 -print0 -exec rm {} \; >/dev/null 2>&1
```

Selain membuat file crontab, saya juga mengubah file session.php menjadi sebagai berikut:

```php
<?php
$start_session = false;
if(isset($_COOKIE['NAMASESSION']))
{
$start_session = true;
}
if(isset($_POST['username']) && isset($_POST['password']))
{
$start_session = true;
}
if($start_session)
{
$lifetime = 1440;
$path = "/";
$domain = ".domain.tld";
$domain = "";
$session_name = "NAMASESSION";
session_name($session_name); 
session_set_cookie_params ($lifetime, $path, $domain);
session_start();
}
?>
```

Dengan kondisi ini, PHP hanya akan membuat session jika request dari client mengandung cookie yang akan kita gunakan sesbagai nama session atau jika client mengirimkan credential untuk login. Ini akan mencegah PHP membuat file session secara membabibuta hingga jumlahnya puluhan juta karena aplikasi saya diakses oleh jutaan orang di seluruh dunia dalam waktu 24 menit.

Semoga pengalaman ini bermanfaat bagi pembaca yang mengalami masalah yang sama.
