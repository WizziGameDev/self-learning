# DOCKER FILE

Pembentuk dari docker image berasal dari docker file
Docker file berisi file text perintah  yang digunakan untuk melakukan instalasi aplikasi a, b, c dll

## Docker Build
Menggunakan docker build untuk mengubah docker file ke bentuk docker image
- sudo docker build -t name_application:version folder_dockerfile

## Format File
- Dockerfile -> semua instruksi akan di taruh di file tersebut

## Format Instruction
- (#) untuk komentar
- INSTRUCTION argument -> saran UPPERCASE

## Tahapan membuat Docker Image
- Buat folder (bebas) yang berisi file dengan nama Dockerfile

- Dikarenakan untuk membuat image membutuhkan image yang sudah ada di docker hub jadi kita menambahkan (wajib): 
-> From name_image:tags
-> From alpine:3

- Kemudian jalankan Docker Build nya
-> sudo docker build -t name_application:version folder_dockerfile
-> sudo docker build -t wizzidevs/apps1:latest apps

- Tunggu sampai images dibuat, dan selesai

## RUN Instruction Format (Waktu Build)
Kegunaan RUN untuk melakukan instruksi ataupun menginstall dependency, menginstall mysql, elasticsearch, mongo db dll.

- Perintah didalam file:
RUN mkdir hello
RUN echo "Hello World" > "hello/world.txt"
RUN cat "hello/world.txt"

- Kalau sudah berhasil build dan melakukan build status nya akan CACHED
- Cara melihat proses build nya, tambahkan:
--progress=plan
-> sudo docker build --progress=plan -t wizzidevs/apps1:latest apps

- Mengulangi build tanpa menggunakan cache, karena cache itu menjalankan build yang sudah ada
--no-cache
-> sudo docker build --no-cache -t wizzidevs/apps1:latest apps

## Command Instructuion (CMD Container)
Command dijalankan ketika Docker Container berjalan. Ketika membuat Command di docker file lebih dari 1, yang akan di jalankan yang terakhir

- Perintah CMD
-> CMD command param param
-> CMD ["executable", "param", "param"]
-> CMD ["param", "param"], Menggunakan executable ENTRY POINT

- Buat file Dockerfile pada folder command, isi dengan:
FROM alpine:latest

RUN mkdir hello
RUN echo "Hello World" > "hello/world.txt"
RUN cat "hello/world.txt"

CMD cat hello/world.txt -> melakukan pembacaan file .txt

- Build ke bentuk image
sudo docker build -t wizzidevs/command command

- Buat container dengan image tersebut
sudo docker container create --name command wizzidevs/command:latest

- Jalankan, dan lihat logs nya, jika benar akan muncul logs Hello World
sudo docker container start command
sudo docker container logs command

## Label Instruction
Untuk penambahan informasi saja seperti versi, Author,dll
Format:
LABEL <key> = <value>
LABEL <key1> = <value1> <key2> = <value2> ...

LABEL author="Ricky Adam Saputra"
LABEL company="wizzi" website= "https://www.wizzidevs.com/"

Buat ke dalam image
sudo docker build -t wizzidevs/label label

Cara melihat label nya dengan:
sudo docker image inspect wizzidevs/label

## Add Instruction
Add menambahkan file/foder serta melakukan extract. Juga dapat mendownload source dari url. Auto extract

Untuk menambahkan file kedalam image, untuk pattern nya:
/*.txt semua file dengan format akhir .txt
/* semua file/folder tanpa terkecuali

- Buat folder di image untuk menampung file dari luar
RUN mkdir name_folder_image

- Pindahkan semua file yang ada di folder local ke dalam folder image
ADD name_folder/*.txt name_folder_image

- File Dockerfile:
FROM alpine:latest

RUN mkdir hello
ADD text/*.txt hello

CMD cat "hello/data.txt"

Kemudian jalankan build untuk merubah ke docker image

Buat container serta jalankan, terus cek log nya

## COPY Instruction
Melakukan copy paste file dari local ke folder docker images. Perlu diperhatikan hanya melakukan copy, jika file berbentuk tar.gz dll akan melakukan extract dengan menggunakan perintah RUN

COPY source destination
COPY world.txt hello -> copy world.txt ke folder hello
COPY *.txt hello -> copy semua .txt ke folder hello

Direkomendasikan menggunakan COPY dari pada ADD (best practice)

- Buat folder di image untuk menampung file dari luar
RUN mkdir name_folder_image

- Pindahkan semua file yang ada di folder local ke dalam folder Docker images
ADD name_folder/*.txt name_folder_image

- File Dockerfile:
RUN mkdir hello
COPY text/*.txt hello

CMD cat "hello/data.txt"

Kemudian jalankan build untuk merubah ke docker image

Buat container serta jalankan, terus cek log nya

## Docker Ignore
Cara melakukan ignore file dengan nama tertentu maupun format tertentu agar tidak ter upload ke image saat melakukan build

Tambahkan .dockerignore untuk melakukan ignore file
Bentuk folder and file
```json
|-- folder_docker
	|-- folder_data -> yang ingin di unggah
  	 	|-- data1.txt
  	 	|-- data2.log
  	 	|-- data3.log 
	|-- .dockerignore
	|-- dockerfile 
 ```

pada .dockerignore lakukan ignore dengan format bebas/dengan nama spesifik
misal *.log / data2.log ...

untuk dockerfile:

```yaml
FROM alpine:latest

RUN mkdir hello
COPY  folder_docker/folder_data/* hello
	
CMD ls -l hello
```

Ubah ke docker image -> buat container -> start container nya -> cek log nya

## Expose Port
Menjalankan container dari image tertentu dengan port tertentu, membuat default port untuk menjalankan container kita

- Buat code untuk dijalankan seperti go / java
- Contah file main.go:

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}
```
- Pada file Dockerfile:

FROM golang:1.18-alpine

RUN mkdir app
COPY main.go app

EXPOSE 8080 -> port image default nya

CMD go run app/main.go

- Ubah ke docker image -> buat container dengan image tersebut dan jalankan dengan port --p dengan 8080:8080

## ENV
Mengubah suatu nilai di env tanpa harus melakukan hardcode ke code nya:

- main.go file
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
	fmt.Fprintf(w, "Hello, World!")
}

- Dockerfile:
FROM golang:1.18-alpine

ENV APP_PORT=8080 -> key=value

RUN mkdir app
COPY main.go app

EXPOSE ${APP_PORT} -> nilai nya berupa 8080

CMD go run app/main.go

## ENV Setting

## Volume Instruction
Images yang dapat membuat volume sendiri yang digunakan untuk menyimpan semua data yang dibutuhkan oleh code kita

Membuat code go  nya:
package main

import (
	"fmt"
	"io/ioutil"
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
	fmt.Fprintf(w, "Hello, %s", r.URL.Path[1:])

	dataString := "Hello " + r.URL.Path[1:]
	dataBytes := []byte(dataString)

	destination := os.Getenv("APP_DATA")
	file := destination + "/" + r.URL.Path[1:] + ".txt"
	err := ioutil.WriteFile(file, dataBytes, 0666)
	if err != nil {
		panic(err)
	}
	fmt.Println("DONE Write File : " + file)
}

file Dockerfile:
FROM golang:1.18-alpine

ENV APP_PORT=8080 -> mendifinisikan port aplikasi ketika dijalankan
ENV APP_DATA=/logs -> path direktory untuk mengimpan logs

RUN mkdir ${APP_DATA} -> membuat direktory di dalam container dengan value dari APP_DATA
RUN mkdir app

COPY main.go app

EXPOSE ${APP_PORT} -> expose port dengan value dari ENV APP_PORT
VOLUME ${APP_DATA} -> volume ini hanya mengaitkan direktori /logs ke penyimpanan jika container dihapus data tetap ada.

CMD go run app/main.go

Terus nama volume nya apa dong? nama volume nya akan di beri nama anonim/acak, tak perlu kawatir jika ingin menyimpan dengan volume secara explisit bisa dengan menggunakan cara berikut ini:

note: saat build ke image tidak akan langsung membuat Volume nya hanya path nya, Volume akan di buat ketika membuat container dari volume tersebut

Membuat container dengan penambahan:
-v name_explisit:/logs -> waktu membuat container nya (buat volume nya dlu)

Buat images -> Buat Container -> Jalankan Container -> cek logs

## WORKDIR (Working Direktory)
Apa bedannya RUN mkdir ... dan WORKDIR ...
berikut bedannya:
1 mkdir memerlukan penentuan secara explisit untuk melakukan eksekusi file yang ada dalam folder tertentu, serta melakukan hal yang sama berulang kali. Contoh perintah nya:

FROM alpine
RUN mkdir /app/data
RUN cd /app/data && touch file.txt -> perlu masuk ke direktory tersebut
RUN cd /app/data && echo "Hello World" > hello.txt -> menuliskan lagi

2. dengan WORKDIR semua perintah akan berada pada direktori tersebut, seperti halnya home. Contohnya:

FROM alpine
WORKDIR /app/data -> sekarang semua eksekusi ada di "data"(working directory)
RUN touch file.txt -> langsung eksekusi tanpa perlu penulisan path /app/data
RUN echo "Hello World" > hello.txt

## USER Instruction

file go:
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

File Dockerfile:
FROM golang:1.18-alpine

RUN mkdir /app

// membuat user
RUN addgroup -S adamgroup -> membuat grup
RUN adduser -S -D -h /app adamuser adamgroup -> membuat user linux dan menaruh di grup tersebut
RUN chown -R adamuser:adamgroup /app -> akses penuh user tersebut terhadap direktory /app

USER pznuser -> mengaktifkan user di container nya

COPY main.go /app

EXPOSE 8080
CMD go run /app/main.go

Build ke images, Buat container, Start, masuk ke containernya, cek user nya

## Argument (ARG) Instruction
Hanya bisa digunakan pada saat build time. Ketika berjalan dalam Docker Container ARG tidak bisa digunakan.
ARG cocok untuk perubahan pada Docker Image (tidak bisa diubah ketika docker container berjalan)
ENV cocok disaat Docker Container berjalan

Cara akses variable nya dengan ${name_variable}

File main.go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

Dockerfile:

FROM golang:1.18-alpine

ARG app=main -> key=value

RUN mkdir app
COPY main.go app
RUN mv app/main.go app/${app}.go -> cara akses argument nya

EXPOSE 8080

ENV app=${app} -> Mengakali agar file nya kebaca
CMD go run app/${app}.go

note: ketika sudah menjadi image, ARG sudah tidak bisa melakukan setting seperti ENV, kalau ENV saat docker container berjalan/dibuat dapat melakukan settings

Penggunaan build:
sudo docker build -t name_image folder_dockerfile --build-arg key=value
sudo docker build -t wizzidevs/arg arg --build-arg app=adam

buat container dengan image tersebut, start dan masuk untuk mengecek apakah adam.go ada

## HEALTH CHECK INSTRUCTION
Untuk mengetahui apakah container tersebut sehat atau tidak
Nilai awal starting
Jika sukses bernilai healthy
Jika gagal bernilai unhealthy

Format
HEALTHCHECK NONE -> disable healthcheck
HEALTHCHECK [OPTION] CMD command
OPTION:
- --interval=DURATION (default 30s)
- --timeout=DURATION (default 30s)
- --start-period=DURATION (default 0s)
- --retries=N (default 3)

- Buat folder health
- Isi dengan file main.go
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

File di atas melakukan counter, jika counter mencapai lebih dari 5 status akan menjadi KO

- Dockerfile
FROM golang:1.18-alpine

RUN apk --no-cache add curl
RUN mkdir app

COPY main.go app

EXPOSE 8080

HEALTHCHECK --interval=5s --start-period=5s CMD curl -f http://localhost:8080/health

CMD go run app/main.go

Build ke image, buat container dari image tersebut, jalankan dan lihat dengan 
sudo docker container ls  -> check status health nya

Melakukan inspect 
sudo docker container inspect name_container

## ENTRYPOINT Instruction
Menjalankan perintah tertentu setiap kali container dijalankan

- Format
ENTRYPOINT ["executable", "param1", dst]

- file main.go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

- file Dockerfile
FROM golang:1.18-alpine

RUN mkdir /app/
COPY main.go /app/

EXPOSE 8080

ENTRYPOINT ["go", "run"]

CMD ["/app/main.go"]

Jalankan seperti biasa, build ke images, buat container dengan images tersebut dan jalankan

## Multi Stage Build
Digunakan untuk melakukan build ke bentuk images dengan mengakali agar ukuran images tidak besar. Dengan melakukan build sementara dan menaruh file hasil build tersebut yang berupa binary dan melakukan copy ke dalam images yang lebih kecil 

Dockerfile:
FROM golang:1.18-alpine as builder -> sebagai builder stage
WORKDIR /app/
COPY main.go /app/
RUN go build -o /app/main /app/main.go -> melakukan kompilasi /app/main.go kedalam /app/main

FROM alpine:3
WORKDIR /app/
-> dari build stage yang lain dengan nama builder dan melakukan copy dari /app/main (binary) kedalam /app/
COPY --from=builder /app/main /app/
cmd /app/main

Note: melakukan kompilasi/build stage dengan FROM .... akan berjalan di buildernya jadi tidak sampai jadi images nya.

## Docker Registry
Melakukan upload images ke Docker Registry. Free docker registry ada di Docker Hub. Pastikan login terlebih dahulu dengan access token

Melakukan push:
Pastikan nama images berupa:
name_user/name_images

docker push images
docker push wizzidevs/command
