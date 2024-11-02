# DOCKER COMPOSE

Digunakan untuk melakukan multiple container. Docker compose memudahkan dalam melakukan konfigurasi, yang sebelumnya harus membuat image manual harus menghapus dan membuat lagi jika ingin mengubah konfigurasi nya.

Dengan docker compose kita dapat melakukan konfigurasi melakukan file .yml
File konfigurasinya namannya:
docker-compose.yaml

perlu melakukan set version docker-compose.yaml
version: "4.33.0" -> versi terbaru, bisa menyesuaikan

Contoh penggunaan .yaml

```yaml
firstName: Ricky
middleName: Adam
lastName: Saputra
hobbies: 
  - berenang
  - freestyle
  - kayang
```

kalau json nya:

```json
{
  "firstName" : "Ricky",
  "middleName" : "Adam"
  "lastName" : "Saputra"
  "hoobies" : [
    "berenang",
    "freestyle",
    "kayang"
  ]
}
```

Lebih simple .yaml kan?

- Contoh nested object di yaml (jangan lupa tab nya)

```yaml
address:
  street: pahlawan
  city: surabaya
  country: indonesia

- Array nested Object (jangan lupa tab nya)
wallet:
  - type: cash
    amount: 20
  - type: kredit
    amount: 30
```

Sebelumnya kalau membuat docker container harus melakukan docker container create, disini kita bisa membuat container nya melalui yaml nya

NOTE:
Susunan dari folder kita

```txt
|-- name_folder
  |-- docker-compose.yaml
```

untuk menjalankan perintah" seperti yang akan kita bahas selanjutnya, harus masuk ke name_folder

## Membuat container dengan docker-compose.yaml

```yaml
version: "4.33.0"

services:
  key_container: ->usahakan sama dengan nama container
    container_name: nama_container
    image: name_image:tags
```
----------- implementasi:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  nginx-example:
    container_name: nginx-example
    image: nginx:latest
```

setelah melakukan konfigurasi di file tersebut, oh ya pastika docker-compose.yaml nya berada di folder. untuk menjalankan tinggal masuk ke folder yang ada docker-compose.yaml nya dan lakukan perintah di terminal:
docker compose create

Bagaimana jika membuat 2 container yang mana container yang 1 sudah ada?
contoh:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  nginx-example:
    container_name: nginx-example
    image: nginx:latest
  nginx-example2:
    container_name: nginx-example2
    image: nginx:latest
```

Untuk container nginx-example2 akan dibuat, namun untuk yang sudah ada nginx-example tidak akan dibuat lagi

note: masuk di folder yang ada docker-compose.yaml untuk melakukan action

## Melihat list Docker Compose yang Berjalan
Untuk melihat list docker compose yang ada di configuration file nya, tidak tercampur dengan docker container lainya:
- sudo docker compose ps

## Menghentikan Container
Untuk menghentikan semua container yang berjalan di configurationnya:
- sudo docker compose stop

## Menghapus Container
Bisa dengan docker container rm name_container

Namun ada cara yang baik untuk menghapus, penghapusan ini akan menghapus semua container, network  yang ada di konfigurasi file tersebut.
- sudo docker compose down

## Melihat Project Compose yang Berjalan
sudo docker compose ls

## Membuat 2 container

docker-compose.yaml

```yaml
docker-compose.yaml

version: "4.33.0"

services:
  nginx-example:
    container_name: nginx-example
    image: nginx:latest
  mongo-example:
    container_name: mongo-example
    image: mongo:latest
```

Jalankan perintah berikut untuk membuat container nya
- sudo docker compose create

Jalankan containernya
- sudo docker compose start

Cek apakah sudah jalan 2 container tersebut
- sudo docker compose ps

Jika ingin menghapus containernya
- sudo docker compose down

## KOMENTAR
penggunaan komentar dengan (#)
contoh:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  # nginx container
  nginx-example:
    container_name: nginx-example
    image: nginx:latest
  # mongo container
  mongo-example:
    container_name: mongo-example
    image: mongo:latest
```

docker-compose.yaml

## PORT
Penggunaan port terdapat 2 jenis
-> Dengan menggunakan short port

docker-compose.yaml

```yaml
ports: 
  - "8080:80"
```

-> Dengan menggunakan long port

```yaml
ports:
  - protocol: tcp -> default tcp
    published: 8080 -> akses publish
    target: 80 -> port pada application
```

contoh penggunaanya:

```yaml
version: "4.33.0"

services:
  nginx-port1:
    container_name: nginx-port1
    image: nginx:latest
    ports:
      - protocol: tcp
        published: 8080
        target: 80
  nginx-port2:
    container_name: nginx-port2
    image: nginx:latest
    ports:
      - "8081:80"
```

note: port yang di publish harus beda jika jadi 1 dalam configuration, karena akan melakukan running semuanya

Buat containse dengan:
- sudo docker compose create

Menjalankan container:
- sudo docker compose start

Melakukan check apakah sudah berjalan
- sudo docker compose ps

Melakukan stop dan menghapus container
- sudo docker compose down

## ENVIRONMENT
Penggunaan environtment pada mogodb

contoh:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongodb-example:
    container_name: mongodb-example
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
```

## BIND dari Local ke Container
- Short syntax
SOURCE:TARGET:MODE
-> SOURCE : lokasi host di leptop (relative (./) dan absolut path)
-> TARGET : lokasi di container
-> MODE : mode bind mount, ro untuk read only dan rw read and write (dafault)

Buat folder di local/komputer untuk menyimpan data di komputer, contoh nya yang akan kita gunakan "data-mongo"

penggunaan volumes:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongodb-example:
    container_name: mongodb-example
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
  volumes:
    - "./data-mongo:/data/db"
```

Terus MODE nya gimana? mode defaultya rw (read and write)
Untuk ./data-mongo berupa folder yang kita buat sebelumnya

- Long syntax

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongo-example:
    container_name: mongo-example
    image: mongo:latest
    ports:
      - "27017:17017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
    volumes:
      - type: bind
        source: "./data-mongo"
        target: "/data/db"
        read_only: false
```

## VOLUME
membuat volume terpisah, namun untuk melakukan create harus membuat containernya dlu dan memasangkan volume nya

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongodb-example:
    container_name: mongodb-example
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
  volumes:
    - "mongo-data1:/data/db" -> menggunakan key attribute

volumes:
  mongo-data1: -> merupakan key attribute
    name: mongo-data1 -> name volumenya
  mongo-data2:
    name:mongo-data2
```

Ketika menghapus container dengan "docker compose down", container dan network akan terhapus namun tidak dengan volume nya. Dan ketika menjalankan konfigurasi lagi yang sama dengan nama volume, akan menggunakan volume yang lama.

## NETWORK
Untuk network menggunakan docker compose akan terhubung antar containernya (default). Untuk nama project network nya name-folder_default. Jadi sebenarnya tidak perlu membuat network secara manual.

note: network dan volume bergantung pada container, jika network dan volume tidak digunakan di container, tidak akan dibuat. Untuk memasukkan network dan volume menggunakan nama id/key bukan namenya

Contoh penggunaanya:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongodb-example:
    container_name: mongodb-example
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
  networks:
    - network_example -> name id/key network nya

networks:
  network_example:
    name: network_example
    driver: bridge
```

## DEPENDS ON
Digunakan untuk membuat container setelah pembuatan container tertentu

contoh:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongodb-example:
    container_name: mongodb-example
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
  networks:
    - network_example

mongodb-express-example:
  image: mongo-express:latest
  container_name: mongodb-express-example
  environment:
    ME_CONFIG_MONGODB_ADMINUSERNAME: adam
    ME_CONFIG_MONGODB_ADMINPASSWORD: adam
    ME_CONFIG_MONGODB_SERVER: mongodb-example -> container mongodb
  networks:
    - network_example -> id/key network nya
  depends_on: -> express akan di jalankan setelah mongodb dibuat
    - mongodb_example -> id/key nya bukan nama container

networks:
  network_example:
    name: network_example
    driver: bridge
```

Saat di jalankan/start container yang dibuat nanti akan error, karena ada masalah dan berhenti kenapa? Karena mongodb perlu waktu untuk melakukan start, jadi harus melakukan docker compose start untuk menjalankan lagi agar mongo express jalan. Solusi nya gimana? kita akan menggunakan restart, pada materi berikut ini:

## RESTART
Value restart:
- no : default nya tidak pernah di restart
- always : selalu restart jika container berhenti, jika leptop mati dan nyala 			akan mulai menjalankan
- on-failure: restart jika container mengalami error/exit
- unless-stopped: selalu restart container, kecuali ketika di hentikan manual,
ketika container sudah di stop dan restart leptop/sistem, container tidak akan dijalankan

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  mongodb-example:
    container_name: mongodb-example
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adam
      MONGO_INITDB_ROOT_PASSWORD: adam
      MONGO_INITDB_DATABASE: admin
    networks:
      - network_example

mongodb-express-example:
  image: mongo-express:latest
  container_name: mongodb-express-example
  restart: always
  environment:
    ME_CONFIG_MONGODB_ADMINUSERNAME: adam
    ME_CONFIG_MONGODB_ADMINPASSWORD: adam
    ME_CONFIG_MONGODB_SERVER: mongodb-example -> container mongodb
  networks:
    - network_example -> id/key network nya
  depends_on: -> express akan di jalankan setelah mongodb dibuat
    - mongodb_example -> id/key nya bukan nama container

networks:
  network_example:
    name: network_example
    driver: bridge
```

Sekarang ketika mongodb butuh waktu untuk mulai dan mongo express harus dijalankan dan terjadi error karena mongodb masih butuh waktu mulai, mongo express akan di jalankan lagi otomatis dan tidak manual lagi

## Melihat logs/prosess Real Time
- sudo docker events --filter 'container=name_container'
- sudo docker events --filter 'container=mongodb'

## Mengatur Limit
Limit pada setiap container merupakan batas memory/cpus yang digunakan
Reservation merupakan minimal memory/cpus yang digunakan setiap containernya
 
Contoh penggunaannya:

docker-compose.yaml

```yaml
version: "4.33.0"

services:
  nginx-port1:
    container_name: nginx-port1
    image: nginx:latest
    ports:
      protocol: tcp
      published: 8080
      target: 80
    deploy:
      reservations: -> batas minimal
      cpus: "0.25"
      memory: 50M
    limits: -> batas maksimal
      cpus: "0.50"
      memory: 100M
```

# Dockerfile
Melakukan build dockerfile dengan docker compose

format folder and file:
```txt
|-- example-folder
  |-- app -> folder
    |-- Dockerfile -> file
    |-- main.go -> file
  |-- docker-compose.yaml -> file
```
Untuk Dockerfile isi sebagai berikut:

Dockerfile

```txt
FROM golang:1.18-alpine

ENV APP_PORT=8080

RUN mkdir app
COPY main.go app

EXPOSE ${APP_PORT}

CMD go run app/main.go
```

untuk main.go

```go
package main

import (
  "fmt"
  "net/http"
  "os"
)

func main() {
  port := os.Getenv("APP_PORT")
  fmt.Println("Run app in port : " + port)
  http.HandleFunc("/", HelloServer)
  http.ListenAndServe(":"+port, nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, World Lagi!")
}
```

Untuk docker-compose.yaml

```yaml
version: "4.33.0"

services:
  app:
    container_name: app
    build:
      context: "./app" -> letak konfigurasi Dockerfile nya
      dockerfile: Dockerfile -> untuk penamaan file konfigurasi bebas
      image: "app-golang:1.0.0" -> nama image output nya, tags bebas
    environment:
      - "APP_PORT=8080" -> port akses golang nya
    ports:
      - "8080:8080" -> expose ports container nya
```

Kemudian lakukan build
- sudo docker compose build

selanjutknya buat containernya dengan 
- sudo docker compose create

jalankan container nya dengan
- sudo docker compose start
Untuk menghapus container nya dengan
- sudo docker compose down

namun ada masalah nih dimana image tetap masih ada, jadi gimana? untuk menghapus image nya harus dihapus manual dengan 
- sudo docker image rm name_image
- sudo docker image rm app-golang:1.0.0

## Kondisi Penting
Jika kita merubah code program kita, kita tidak bisa melakukan build langsung. Jika melakukan hal tersebut akan menampilkan code program sebelumnya, Jadi gimana?
Untuk mengatasi hal tersebut kita harus melakukan down terlebih dahulu container nya dengan "sudo docker compose down", hapus image sebelumnya, lakukan build ulang -> create -> start

## Health Check
Attibute untuk health check
- test: berisi cara melakukan test
- interval: interval melakukan health check
- timeout: timeout melakukan health check
- retries: total retry ketika gagal
- start_period

```txt
|-- health-check
  |-- app
    |-- Dockerfile
    |-- main.go
  |-- docker-compose.yaml
```

Dockerfile:
```txt
FROM golang:1.18-alpine

RUN apk --no-cache add curl
RUN mkdir app

COPY main.go app

EXPOSE 8080

CMD go run app/main.go
```

main.go:

```go
package main

import (
  "fmt"
  "net/http"
)

var counter = 0

func main() {
  http.HandleFunc("/", HelloServer)
  http.HandleFunc("/health", HealthCheck)

  http.ListenAndServe(":8080", nil)
}

func HealthCheck(w http.ResponseWriter, r *http.Request) {
  counter = counter + 1
  if counter > 5 {
    w.WriteHeader(500)
    fmt.Fprintf(w, "KO")
  } else {
    fmt.Fprintf(w, "OK")
  }
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, World!")
}
```

pada code diatas melakukan counter dan jika dounter bernilai lebih dari 5 akan error

docker-compose.yaml
```yaml
version: "3.8"

services:
  app:
    container_name: app
    build:
      context: "./app"
      dockerfile: Dockerfile
    image: "app-golang:1.0.0"
    environment:
      - "APP_PORT=8080"
    ports:
      - "8080:8080"
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:8080/health" ]
      interval: 5s
      timeout: 5s
      retries: 3
      start_period: 5s
```

# Extend Service (Multi Env)
Dimana jika kita memiliki 3 compose:
- development
- stagging
- production

itu akan berat untuk melakukan maintenence jika ada perubahan, dengan extend service kita bisa melakukan merge di bagian tertentu
cara penggunaannya:

Folder format:
```txt
|-- extend-service
  |-- app
    |-- Dockerfile
    |-- main.go
  |-- dev.yaml
  |-- local.yaml
  |-- prod.yaml
  |-- docker-compose.yaml
```
->Dockerfile:
```txt
FROM golang:1.18-alpine

ENV APP_PORT=8080
ENV MODE=local

RUN mkdir app
COPY main.go app

EXPOSE ${APP_PORT}

CMD go run app/main.go
```

-> main.go:

```go
package main

import (
  "fmt"
  "net/http"
  "os"
)

func main() {
  port := os.Getenv("APP_PORT")
  fmt.Println("Run app in port : " + port)
  http.HandleFunc("/", HelloServer)
  http.ListenAndServe(":"+port, nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
  mode := os.Getenv("MODE")
  response := "Hello " + mode
  fmt.Fprintf(w, response)
}
```

-> docker-compose.yaml

```yaml
version: "3.8"

services:
  app:
    container_name: app
    build:
      context: "./app"
      dockerfile: Dockerfile
    image: "app-golang:1.0.0"
    environment:
      - "APP_PORT=8080"
    ports:
      - "8080:8080"
```

-> dev.yaml

```yaml
version: "3.8"

services:
  app:
    environment:
      - "MODE=dev"
```

-> local.yaml

```yaml
version: "3.8"

services:
  app:
    environment:
      - "MODE=local"
```

-> prod.yaml

```yaml
version: "3.8"

services:
  app:
    environment:
      - "MODE=local"
```

Untuk melakukan merge dari template docker-compose.yaml dengan:

sudo docker compose -f docker-compose.yaml -f prod.yaml create

note:
ketika membuat bind volume entah itu dari dockerfile ataupun docker compose sama saja cara kerja volume nya. Cuman deklarasian nya dari dockerfile atau compose (1 object sama)
