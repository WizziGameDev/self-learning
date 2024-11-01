# DOCKER DASAR

## Menjalankan docker
- systemctl --user start  docker-desktop

## Cek version
- sudo docker version

3# Cek List Image
- sudo docker image ls

## Install Image : hub.docker
- sudo docker image pull name:tags
- sudo docker image pull redis:latest

## Delete Image
- sudo docker image rm name:tags
- sudo docker image rm redis:latest

## Cek ALL Container (run and Stop)
- sudo docker container ls -a

## Cek Container Running
- sudo docker container ls

## Note
- Nama container yang menggunakan image yang sama  harus berbeda namannya misal:
image redis:
-> container 1 -> redis
-> container 2 -> redis
- Jika image tidak ada akan auto download
- 1 image bisa banyak container
- container terisolasi, jika container ada yang bermasalah tidak akan mempengaruhi container lainnya
- jika ingin menghapus container, harus menghentikan container tersebut apabila sedang berjalan

## Membuat Container
- sudo docker container create --name name_container image:tags
- sudo docker container create --name learnredis redis:latest

## Menjalankan Container
- sudo docker container start name_container
- sudo docker container start learnredis

## Menghantikan Container
- sudo docker container stop name_container
- sudo docker container stop learnredis

## Menghapus Container
Menghapus container, harus menghentikan terlebih dahulu container yang berjalan
- sudo docker container rm name_container
- sudo docker container rm learnredis

## Melihat logs Container
Melihat logs code yang berjalan di container
- sudo docker container logs name_container
- sudo docker container logs learnredis
- sudo docker container logs -f learnredis -> menunggu sampai ada logs baru

## Masuk ke dalam Container
Melakukan aksi pada container
- sudo docker container exec -i -t name_container /bin/bash
- sudo docker container exec -i -t learnredis /bin/bash
- -i -> berupa argumen, menjaga input tetap aktif
- -t -> terminal akses
- /bin/bash -> code program yang ada di container

## Port Forwarding
Melakukan forwarding dengan port tertentu secara publish ke port tertentu misal nginx, dimana nginx memakai port 80, namun untuk akses nya bisa melalui port yang di tentukan misal 8080. Ketika akses port 8080 akan redirect ke port 80 milik nginx.
Penggunaan: 
- sudo docker container create --name name_container --publish publishPort:defaultPort image:tags
- sudo docker container create --name learnNginx --publish 8080:80 nginx:latest

note:
Jika container yang berbeda berisi image yang sama dan port yang sama tidak bisa di akses kecuali memakai load balancing atau mengubah port nya agar bisa diakses

## Container Environtment Variable
Mengubah code env secara langsung memakai docker tanpa hardcode ke dalam code nya
- sudo docker container create --name name_container --publish portPublish:portAsal --env nameEnv=value --env nameEnv=value image:tags

- sudo docker container create --name contohmongo --publish 27017:27017 --env MONGO_INITDB_ROOT_USERNAME=adam --env MONGO_INITDB_ROOT_PASSWORD=adam mongo:latest

Tidak bisa update --env kalau sudah dibuat, harus sejak awal di settings

## Melihat Detail Resource Container Run
Melihat resource yang digunakan pada setiap container
- sudo docker container stats

## Mengatur limit
Mengatur limit memory, cpu pada container
- sudo docker container create --name name_container --publish publishPort:defaultPort --memory 100m --cpus 2.3 image:tags

- sudo docker container create --name cekenginx --publish 8081:80 --memory 100m --cpus 2.3 nginx:latest

Update container memory saat berjalan
sudo docker update testnginx --memory 100m --memory-swap 200m

## NOTE FOR DATA
- type -> berupa type data -> bind/volume
- source -> letak folder local sebagai resource/sharing (pwd ->cek letak folder)
- destination -> letak penyimpanan data container, cek di hub.docker untuk mengetahui default penyimpanan dari image tertentu

## BIND MOUNT
Melakukan sharing data dari local ke container, jika container dihapus, data di local nya tidak akan ikut kehapus, dan jika membuat lagi kita bisa mengambil data di local sebelumnya

- sudo docker container create --name name_container --publish publishPort:defaultPort -- mount "type=bind,source=/local_computer,destination=/default_data" --env MONGO_INITDB_ROOT_USERNAME=adam --env MONGO_INITDB_ROOT_PASSWORD=adam image:tags

- sudo docker container create --name contohmongos --publish 27018:27017 -- mount "type=bind,source=/home/wizzi/mongo-data,destination=/data/db" --env MONGO_INITDB_ROOT_USERNAME=adam --env MONGO_INITDB_ROOT_PASSWORD=adam mongo:latest

## DOCKER VOLUME
Penyimpanan file/data di docker nya tidak di host/local nya. Penyimpanan dan management data sudah di ataur oleh dockernya
- sudo docker volume ls

Membuat volume
- sudo docker volume create name_volume
- sudo docker volume create mongovolume

Menghapus volume
- sudo docker volume rm name_volume
- sudo docker volume rm mongodata

Perlu diperhatikan jika volume dipakai oleh container, volume tidak bisa dihapus. Jadi pastikan tidak digunakan oleh container

Pembuatan container nya:
- sudo docker container create --name name_container --publish publishPort:defaultPort --mount "type=volume,source=name_volume,	destination=folder/db_container" mongo:latest

- sudo docker container create --name mongocontainer --publish 27018:27017 --mount "type=volume,source=mongodata,	destination=/data/db" mongo:latest

## Backup Volume TO Bind
- Matikan container yang berjalan/container yang memiliki akses di volume tersebut harus mati semua
- Buat container baru untuk membantu peyimpanan ke local/komputer

- Membuat container sementara untuk backup
->
sudo docker container create --name name_container --publish publishPort:defaultPort --mount "type=bind,source=/folder_host_computer,destination=/name_folder_bebas" --mount "type=volume,source=/name_volume_for_backup,destination=/folder_isi_volume" image:tags
->
sudo docker container create --name mongoprivate --publish 20002:27018 --mount "type=bind,source=/home/wizzi/Backup,destination=/backup" --mount "type=volume,source=mongodata,destination=/data-mongo" mongo:latest
	
- Setelah itu start container nya
- Masuk kedalam container nya:
sudo docker container exec -i -t name_container /bin/bash
sudo docker container exec -i -t mongoprivate /bin/bash

- Cek apakah folder backup dan data-mongo ada dengan (ls)
- Melakukan backup dengan:
tar cvf /folder_backup/nama_file /folder_yang_ingin_dibackup
tar cvf /backup/backup.tar.gz /data-mongo
 
- Cek di folder local anda, akan ada folder .tar.gz
- Jika sudah hapus container sementarannya

## SIMPLE BACKUP
- run -> membuat dan menjalankan containernya
- --rm -> akan berhenti dan otomatis menghapus containernya

sudo docker container run --rm  --name mongobackup  --mount "type=bind,source=/home/wizzi/Backup,destination=/backup" --mount "type=volume,source=mongodata,destination=/data-mongo" mongo:latest tar cvf /backup/backup.tar.gz /data-mongo

## SIMPLE RESTORE
Beberapa perubahan untuk extract ke volume, alur nya menghubungkan local ke folder container setelah terhubung melakukan copy/extract ke folder container yang terhubung dengan volume

sudo docker container run --rm  --name mongorestore  --mount "type=bind,source=/home/wizzi/Backup,destination=/backup" --mount "type=volume,source=mongorestore,destination=/data-mongo" mongo:latest bash -c "cd /data-mongo && tar xvf /backup/backup.tar.gz --strip 1"

"cd /data-mongo && tar xvf /backup/backup.tar.gz --strip 1" -> masuk kedalam folder /data-mongo dan melakukan extract dari .tar.gz

## DOCKER NETWORK
Membuat agar container dapat berkomunikasi dengan virtual network, dengan memasukkan container ke dalam network akan bisa saling berkomunikasi

- Melihat semua setwork
sudo docker network ls

- Membuat network
sudo docker network create --driver name_driver name_network
sudo docker network create --driver bridge contohnetwork

- Menghapus network
Pastikan menghapus dulu container nya baru menghapus network nya
sudo docker network rm name_network
sudo docker network rm contohnetwork

- Memasukkan container ke network
sudo docker container create --name name_container --network name_network image:tags

sudo docker container create --name testCon --network contohnetwork nginx:latest

- Contoh melakukan koneksi network antar container dimana container a akan mengakses container b

-> sudo docker network create --driver bridge mongonetwork
-> sudo docker container create --name mongodb --network mongonetwork  --env MONGO_INITDB_ROOT_USERNAME=adam --env MONGO_INITDB_ROOT_PASSWORD=adam mongo:latest

-: tidak memakai --publish karena nanti memakai mongo-express untuk akses/manajement Mongo DB di web
->  sudo docker image pull mongo-express:latest -> cari di hub.docker
-> sudo docker container create --name mongoexpress --network --publish 8081:8081 mongonetwork --env ME_CONFIG_MONGODB_URL=mongodb://adam:adam@mongodb:27017/ mongo-express:latest
note: mongodb://username:password@name_host/container:27017/

-> Jalankan container mongodb dan mongoexpress

- Menghapus container di dalam network nya
sudo docker network disconnect name_network name_container
sudo docker network disconnect mongonetwork mongodb

- Menambah container ke network
Penambahan ini apabila belum gabung network
sudo docker network connect name_network name_container

sudo docker network connect mongonetwork mongodb

## INSPECT
Melihat detail dari image, container, volume, network
- sudo docker image inspect name_image
- sudo docker container inspect nama_container
- sudo docker container inspect nama_container
- sudo docker container inspect nama_container

## PRUNE
Membersihkan fitur" yang tidak diperlukan, seperti image yang tidak dibutuhkan, container dll
- sudo docker image prune
- sudo docker container prune
- sudo docker volume prune
- sudo docker network prune


docker start -name namecontainer tes/latest
