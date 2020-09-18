# Mengatasi No Space Left on Device

Pada sistem operasi Linux, ada istilah Inodes. Inodes adalah indexing file dalam sistem operasi Linux. Index ini akan menentukan berapa jumlah file yang dapat disimpan pada senuah direktori atau partisi. Perlu dicatat bahwa jumlah file berbeda dengan total ukuran file.

Sebagai contoh:

Sebuah direktori berisi 100 file yang ukurannya sama besar yaitu 10 kb. Dengan demikian, Inodes untuk direktori tersebut adalah 101 (100 file dan 1 direktori itu sendiri) sedangkan total ukuran file adalah 1000 kb.

Lalu, apa pengaruh Inodes pada sebuah komputer Linux?

Saya mempunyai pengalaman buruk tentang Inodes. Server saya menggunakan sistem operasi Linux dengan server web Apache + PHP. Entah bagaimana PHP tidak menghapus secara otomatis file session yang sudah tidak terpakai lagi. Ditambah dengan setiap request baru yang tidak membawa informasi cookie akan membuat sebuah session baru. Seharusnya ini dapat dihindari dengan memeriksa apakah request tersebut mengandung cookie atau tidak. Tapi sudah terjadi. File session sudah dibuat dengan jumlah yang tidak tanggung yaitu 20.000.000 file lebih. Lalu apa yang terjadi?

Inodes untuk partisi /dev/vda1 mempunyai Inodes sebesar 20.970.944. Akhibatnya, ketika Inodes ini sudah terpakai semua, Linux tidak bisa lagi membuat file baru. Pesan error yang muncul adalah "No space left on device" padahal ruang terpakai baru 49%.
