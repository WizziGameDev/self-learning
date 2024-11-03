# REDIS DATA STRUCTURE

```txt
Structur Data                Keterangan
- Lists                      Struktur data Linked List berisi string 
- Sets                       Koleksi data string yang tidak berurut
- Hashes                     Struktur data key-value
- Sorted Sets                Struktur data seperti set namun berurut
- Stream                     Struktur data yang bertambah di belakang
- Geospatial                 Struktur data koordinat
- HyperLogLog                Struktur data untuk melakukan estimasi
```

## Lists (Antrian dan Tumpukan)
Memasukkan data dengan format list ke dalam key
```txt
Operasi                        Keterangan
- RPUSH key value              Menambahkan data ke kanan
- LPUSH key value              Menambahkan data ke kiri
- RPOP key jumlah_data         Mengambil data dari kanan
- LPOP key jumlah_data         Mengambil data dari kiri
- LRANGE key start end         Melihat data dari range
```

Antrian (FIFO (First In First Out))
Kita akan memasukkan data dengan urutan ke kanan, terus kita ambil dari kiri.

Implementasi:
```txt
> RPUSH data "ada"
> RPUSH data "budi"
> LRANGE data 0 1 -> untuk melihat dari index ke-0 sampai ke-1
> LPOP data 2 -> mengambil 2 data
```

Stack (FILO (First In Last Out))
Implementasi
```txt
> RPUSH data "budi"
> RPUSH data "didi"
> RPUSH data "meli"
> LRANGE data 0 2 -> "budi" "dedi" "meli"
> RPOP data 1
> LRANGE data 0 2 -> "budi" "dedi"
```

## SETS
Untuk data set datanya tidak bisa double, misal ada data yang sama maka hanya 1 yang disimpan. Untuk urutan datanya akan acak.

```txt
Operasi                         Keterangan
- SADD key value ...            Menambahkan data ke set
- SCARD key                     Melihat jumlah data di set
- SMEMBERS key                  Melihat kumpulan data si set
- SREM key value ...            Untuk menghapus data nya
```
Implementasi
```txt
> SADD data "budi" "tono" "beni"
> SCARD data                         -> 3
> SMEMBERS data                      -> "budi" "tono" "beni"
> SREM data "budi"
> SMEMBERS data                      -> "tono" "beni"
```

## Membandingkan Sets
Kita dapat membandingan key data sets, ada beberapa operasi yang bisa kita lakukan:

SDIFF | Membandingkan data pertama dan kedua yang mana key/data ke-1 akan muncul jika tidak ada di key/data ke-2. Jika data 1 ada semua di data 2 akan bernilai nill
```txt
> SADD data1 "ricky" "adam"
> SADD data2 "adam" "saputra"
> SDIFF data 1 data 2 -> "ricky"
```
SINTER | Irisan dari kedua data, data yang sama akan diambil
```txt
> SADD data1 "ricky" "adam"
> SADD data2 "adam" "saputra"
> SINTER data1 data 2 -> "adam"
```
SUNION | Menggabungkan 2 data
```txt
> SADD data1 "ricky" "adam"
> SADD data2 "adam" "saputra"
> SUNION data1 data 2 -> "ricky" "adam" "saputra"
```

## HASHES
Data berupa key-value, Jika data belum ada akan dibuatkan, jika sudah ada dan value ada yang berubah akan diperbarui

Untuk menambahkan data
- hset key field value       | field value itu seperti key-value

```txt
> hset "data" name "adam" age "20"
> hgetall "data" -> name "adam" age "20"
> hget "data" name -> "adam"
```

## HINCRBY
Untuk melakukan increment maupun decrement dalam value data nya
Contoh:
```txt
> hset "data" name "budi" age "20"
> hincrby data age 10
> hget data age --> 30
```

## SORTED SET
Melakukan pengurutan dengan score, pengurutan melalui score dan Ascending. cara penggunaannya:

```txt
Operasi                               Keterangan
- zadd key score value                | Menambahkan value di key
- zcard key                           | Cek jumlah value di key
- zrange key start end                | Melihat value dengan rentang index
- zrem key value                      | Menghapus berdasarkan value
- zremrangebyscore key start end      | Menghapus dengan range score
```
Implementasi
```txt
> zadd data 1 "bro apa kabar"
> zadd data 2 "bro dimana"
> zrange data 0 2 -> hasilnya value dari data
```

## Streams
Penambahan data terus dibelakang, cocok untuk menyimpan data kejadian yang terurut.

*/id -> untuk * akan di generate id otomatis tinggal masukkan

- xadd key * field value          | Menambahkan stream
- xread count 10 streams key 0    | membaca data dengan 10 data, start bebas (0)


## Consumer Group 
Dimana jika kita memiliki beberapa aplikasi, yang mana akan membaca stream nya. Ada masalah jika data stream yang sama dimiliki oleh banyak aplikasi. Jika aplikasi tersebut melakukan penyimpanan data akan mengakibatkan duplicate dikarenakan data yang sama. 

Solusinnya adalah Kita akan membagi data tersebut agar setiap aplikasi yang menerima akan mendapatkan data yang berbeda beda. 1 group stream hanya bisa terukat oleh 1 data stream

Kita akan menangani dengan consumer group yang memiliki multiple consumer (aplikasi kita)

Membuat consumer group:
> xgroup create name_stream/key name_group $ mkstream
- $ -> digunakan untuk melakukan consume berdasarkan id terakhir yang dibaca
- mkstream -> membuat stream jika tidak ada

> xgroup create data data_group $ mkstream

Membuat consumer agar datanya dapat di consume oleh aplikasi
> 	xgroup createconsumer name_stream/key name_group name_user_consumer

> xgroup createconsumer data data_group member_consumer_1
> xgroup createconsumer data data_group member_consumer_2

Menambahkan Data:
> xadd data * status "done"

Melakukan consume:
> xreadgroup group name_group name_user_consumer count 1 block 0 key >
- Kegunaan (>) untuk membaca id yang belum di read

apk 1
> xreadgroup group data_group member_consumer_1 count 1 block 0 data >

apk 2
> xreadgroup group data_group member_consumer_2 count 1 block 0 data >

# Geospatial
Digunakan untuk menyimpan data koordinat, cocok untuk digunakan untuk mencari koordinat terdekat, jarak,radius, dll. dibelakang layar tipe data yang dipakai memakai sorted set

note:
untuk longitude latitude di maps harus dibalik dlu karena berbeda urutannya

Operasi:
> geoadd key longitude latitude member			| Menambahkan data lokasi
- key -> nama data untuk penyimpanannya
- member itu semacam identitas dari lokasi tersebut

> geopos key member 				| Mengambil data lokasi berdasarkan member

> geodist key member1 member2 satuan_jarak
> geosearch key formlonlat longitude latitude BYRADIUS <jarak> satuan_jarak

Implementasi:
> geoadd seller.location 112.7288790502218 -7.29822073987516 seller1
> geoadd seller.location 112.72635511827937 -7.305684971615155 seller2

> geodist seller.location seller1 seller2 km

# HyperLogLog
Digunakan untuk melakukan count pengunjung tanpa harus menyimpan data pengunjung secara lengkap dan melakukan count. Dengan hyperloglog kita dapat melakukan count pengunjung dengan kelebihan ukuran besar file yang sedikit. Jika ada nama pengunjung yang sama akan diambil 1

Operasi:
> pfadd key value value value ... 			| Menambahkan visitor
> pfcount key														| Melakukan count visitor
> del key																| Menghapus visitor

Implementasi
> pfadd visitor adam juma budi
> pfadd visitor adam didi bambang
> pfcount visitor -> 5 hasil count, jika sama akan dihitung 1
> del visitor -> menghapus key 
