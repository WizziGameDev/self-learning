# REDIS DASAR

Redis merupakan database key-value, key merupakan primary key nya dan value adalah datannya (database nosql)

contoh:

key1	data-json
key2	text
key3  text, angka, text

jadi penyimpanannya bebas

Redis menyimpan datanya di memory, terus gimana kalau leptop kita dimatikan? bukannya datanya akan hilang? 
Tenang aja data nya juga di backup secara berkala di disk, ketika menjalankan ulang akan membaca data di disk komputer kita. 

Kapan butuh redis?
Ketika database yang kita tuning menjadi lambat. Cara kerjanya:

```txt
user  <---> aplikasi <---> database
              |	
            redis
```
Ketika melakukan tuning ke database datanya akan ke aplikasi dan menyimpan di redis. Ketika user ingin mengambil lagi datanya, user bisa langsung ambil di redis nya tanpa perlu tuning ke database. Kalau tuning lagi kan repot perlu waktu banyak karena lambat.

Contoh lain pada third party kayak bank, dimana ketika dalam 2 detik hanya bisa melakukan 1 request, karena banyak request yang masuk mau tidak mau akan disimpan ke redis untuk pengambilan datanya. Kalau data di redis ingin diperbarui bisa sesuai ketentuan 1 req/ 2 sec

## Membuat Delayed Job
Dimana ketika melakukan pembayaran dari pada kita melakukan request terus menerus untuk mengetahui apakah sudah dibayar atau belum, kita bisa memanfaatkan scheduler di redis, yang mana jika belum di bayar akan melakukan pembatalan otomatis.

Redis digunakan untuk aplikasi yang lambat

## Install Redis
Saat installing Redis akan melakukan installing client dan server
Redis server merupakan server untuk redis nya
Kalau Redis client untuk berkomunikasi ke server nya 

Cara kerja redis
user ----> Redis client ----> Redis server

## Menjalankan Redis
Menjalankan server nya
- redis-server

Menjalankan redis cli untuk berkomunikasi dengan servernya
- redis-cli -h localhost -p 6379

## Custom Config
Buat folder bebas yang berisikan redis.conf . Untuk penamaan bebas yang penting menggunakan .conf
Selanjutnya copy config dari url berikut: 

https://github.com/redis/redis/blob/7.0/redis.conf

Selanjutnya untuk menjalankan .conf nya 
redis-server name_folder/file.conf

Pada .conf ada namanya databases. Jika kita membuat 20 aplikasi kita bisa set databases dengan jumlah aplikasinya. Apabila setiap aplikasi memiliki redis yang berbeda-beda

Untuk menggunakan database redis kita tidak menggunakan nama secara umum, namun menggunakan nilai angka yang dimulai dari 0, jika kita mmebuat 20 database maka kita bisa menggunakan 0 - 19
Untuk menggunakan database dengan perintah
select 0, select 10, select 19, dll

## OPERASI DI REDIS
```txt
Operasi                     Keterangan
- set key value             mengubah string value dari key
- get key                   mendapatkan value menggunakan key
- exists key                mengecek apakah key memiliki value (1 = ada)
- del key [key ...]         menghapus berdasarkan key
- append key value          menambahkan value di belakangnya berdasarkan key
- keys pattern              mencari key menggunakan patterns (tidak disarankan)
```
```txt
> set test "bro apa kabar"
> get test -> "bro apa kabar"
> exists test -> 1
> append test " koi"
> get test -> "bro apa kabar koi"
> keys * -> test
> del test -> 1
```
## OPERASI RANGE
Perubahan data nya di mulai dari 0 dan jika set ke-5, data ke-5 itulah yang akan keubah

```txt
Operasi                        Keterangan
- setrange key offset value    mengubah value dari offset yang di tentukan
- getrange key start end       mengambil value dari range yang di tentukan
```
```txt
Contoh:
> set data "Ricky Adam Saputra"      |  Menambahkan data dlu
> setrange data 6 "Keren"            |  Melakukan perubahan data dimulai dari ke-6
> get data -> "Ricky Keren"          |  Melakukan print data nya
> getrange data 6 8 -> "Ker"         |  Mengambil data berdasarkan range tertent
```

Note: Melakukan replace untuk mengubah data, jika kondisi kalau data misal 
Ricky Adam, trus di ubah mulai Adam, diubah dengan "Ka" value akhir nya akan menjadi Ricky Kaam -> replace  "Ad" saja ke "Ka"

## MULTIPLE SET DAN GET
Melakukan set dengan multiple key dan value
- mset key value key value
- mget key key key

Implementasi:
```txt
> mset data1 "solo1" data2 "solo2" data3 "solo3"
> mget data1 data2 data3
result:
1) "data1"
2) "data2"
3) "data3"
```
## OPERASI EXPIRE
```txt
Operasi                         Keterangan
- expire key seconds            Mengatur berapa lama key hidup/ada
- setex key seconds value       Membuat key dan value dengan masa hidup
- ttl key                       Mengecek berapa lama suatu key hidup
```
Kegunaannya untuk apa? jika data banya dan karena ini memakai memory akan ada masalah memory out karena data kebanyakan, jadi ini akan auto menghapus data nya dengan menggunakan waktu

Implementasi:
```txt
> set data "Ricky Adam Saputra"
> expire data 5
> ttl data                            -> cek masa hidup, jika habis akan menghasilkan output -2
> setex data 5 "cek masa hidup"       -> membuat data dan set masa hidup 5 detik
> ttl data                            -> setelah 5 detik akan hilang (-2)
```
## Increment dan Decrement
Ada suatu masalah dimana ketika kita melakukan increment manual dan jika ada lebih dari 1 user mengubah key yang sama karena pengecekan secara manual (race condition), datanya akan tertimpa. Maka dari itu kita akan menggunakan OPERASI berikut ini:

```txt
Operasi                         Keterangan
- incr key                      increment value dengan +1
- decr key                      decr ement value dengan -1
- incrby key increment          increment dengan value bisa ditentuin
- decrby key decrement          decrement dengan value bisa ditentuin
```

Dari 0 jika belum ada key nya di database
Implementasi:
```txt
> incr counter -> 1
> incr counter -> 2
> decr counter -> 1
> decr counter -> 0
```
```txt
ini mulai dari 0
> incrby counter 5 -> 5
> decrby counter 3 -> 2
```

## Menghapus Isi Database
Terdapat 2 cara untuk menghapus data dari database

Operasi								Keterangan
- flushdb							Menghapus data di database terkait
- flushall							Menghapus isi dari semua database

langsung eksekusi saja, jika ingin flushdb, masuk ke db yang ingin data nya dihapus semua

## Redis Pipeline
Digunakan untuk memasukkan data sekaligus, bisa menggunakan file.txt

file.txt
```txt
set key value
set 1 "ricky"
set 2 "adam"
set 3 "saputra"
```

Keluar dari redis cli nya terlebih dahulu, kemudian ketikkan perintah seperti berikut
- redis-cli -h localhost -p port -n no-database < name-file.txt
- redis-cli -h localhost -p 6379 -n 0 --pipe < file.txt

## TRANSACTION
Seperti pada relational database lainnya, seperti penggunaan Begin, Commit, Rollback

```txt
Operasi                 Keterangan
- multi                 Membuka transaction
- exec                  Melakukan eksekusi dari berbagai proses
- discard               Melakukan rollback ketika ada 1 proses gagal
```
Implementasi
```txt
> multi
> set ricky "berenang"      --|
> set adam "mancing"        --|-> semua data ini dimasukkan ke antrian/queued
> set putra "ke danau"      --|
> exec                      -> melakukan eksekusi untuk input ke database
```
```txt
rollback:
> multi
> set ricky "berenang"      --|
> set adam "mancing"        --|-> semua data ini dimasukkan ke antrian/queued
> set putra "ke danau"      --|
> discard                   -> semua data di queue di batalkan
```

## Monitoring
Untuk melakukan monitoring untuk melihat aktivitas apa yang terjadi dalam server dapat dilakukan seperti berikut:

```txt
masuk ke dalam cli redis nya
> redis-cli -h localhost -p 6379

masukkan berikut ini
> monitor
```
anda dapat melihat semua aktifitas yang terjadi pada redis server

## Melihat Informasi Redis
Jika kita  ingin melihat informasi yang ada  di redis seperti memory yang di gunakan, port, jumlah database, bisa menggunakan perintah berikut:

```txt
> info
> config get databases
> config get port
```

## Client Connection
Kita dapat melihat siapa saja yang terhubung di redis servernya dengan perintah berikut ini:

```txt
Operasi                Keterangan
client list            Melihat id sapa saja yang terhubung
client id              Melihat id kita saat ini di terminal
client kill id         Memutuskan koneksi client berdasarkan id
```

## Protected Mode
Untuk menggunakan mode ini, kita harus mengganti bind di redis.conf nya menjadi ip komputer kita, carannya dengan:

Cek ip kita (ipconfig/ifconfig)
- 192.168.**.**

Konfigurasi redis.conf dan ganti bind nya sesuai dengan ip nya
- restart redis

Masuk ke redis cli dengan
- redis-cli -h 192.168.**.** -p 6379

Namun setelah itu kita tidak bisa melakukan perintah terlebih dahulu, perlu melakukan security pada langkah berikut ini

## Security
Authentifikasi digunakan untuk melakukan verifikasi bahwa identitas tersebut benar

Di redis ada fitur authentifikasi, untuk menggunakan fitur ini kita harus melakukan konfigurasi di redis.conf nya

Perlu diingat untuk password nya disarankan panjang dan unik, untuk mencegah bruteforce dikarenakan sangat cepat untuk akses redisnya

Selanjutnya Authorization yang mana setelah melakukan Auth.. user boleh dan hanya boleh melakukan hal tertentu. Detail lengkap untuk Auth.. nya ada di link ini: https://redis.io/topics/acl

- Konfigurasi di redis.conf

Tambahkan:

- user default on +@connection

User username semua_perintah semua_key >password
- user adam on +@all ~* >rahasia

Setelah itu save dan jalankan sembali redis-server redis.conf
- masuk dengan redis-cli -h ip -p 6379
- masuk dengan auth username password

- auth adam rahasia

## Presistence
Untuk mengatur seberapa lama/kondisi untuk menyimpan data ke disk. Untuk penyimpanan di disk terdapat kondisi yang perlu dikonfigurasi. Berikut penjelasan dan cara pemakaiannya:
Untuk konfigurasinya terletak pada file .conf nya
```txt
- Unless specified otherwise, by default Redis will save the DB:
> After 3600 seconds (an hour) if at least 1 change was performed
> After 300 seconds (5 minutes) if at least 100 changes were performed
> After 60 seconds if at least 10000 changes were performed

penjelasan
> setelah 1 jam jika ada 1 yang berubah di memory akan disimpan di disk
> setalah 5 menit kalau ada 100 perubahan di memory akan disimpan ke disk
> setelah 60 detik jika ada 10000 perubahan di memory akan disimpan ke disk
```
Penggunaannya:
save 3600 1 300 100 60 10000

note:
perlu di perhatikan jika memakai expire dan kondisi penyimpanan di disk tidak memenuhi maka data yang expire akan hilang dan tidak disimpan. Dan 1 lagi kalau dimatikan redis nya akan disimpan di disk juga

## Eviction
Digunakan ketika memory penuh, jikalau memory pehuh maka data selanjutnya akan di reject, hal tersebut akan menyulitkan jika tiba" ada lonjakan data yang besar. Ada beberapa mode yang bisa kita gunakan untuk menangani hal tersebut.

Untuk melakukan konfigurasi dapat dilakukan di redis.conf nya.

- Melakukan setting maxmemory <..mb>
- Dengan menggunakan beberapa max-policy ..., beberapa mode nya:

- noeviction: New values aren’t saved when memory limit is reached. When a database uses replication, this applies to the primary database
- allkeys-lru: Keeps most recently used keys; removes least recently used (LRU) keys
- allkeys-lfu: Keeps frequently used keys; removes least frequently used (LFU) keys
- volatile-lru: Removes least recently used keys with the expire field set to true.
- volatile-lfu: Removes least frequently used keys with the expire field set to true.
- allkeys-random: Randomly removes keys to make space for the new data added.
- volatile-random: Randomly removes keys with expire field set to true.
- volatile-ttl: Removes keys with expire field set to true and the shortest remaining time-to-live (TTL) value.

Terdapat 2 cara penyimpanan:
- save
- bgsave

Jika Redis secara tiba-tiba mati, BGSAVE memberikan jaminan bahwa data masih tersimpan sampai snapshot terakhir, sementara dengan SAVE, Redis dapat mengalami downtime selama penyimpanan. Konfigurasi yang ada di atas:

save 3600 1 300 100 60 10000
