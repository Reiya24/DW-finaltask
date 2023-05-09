![](.Readme_images/abdaa96b.png)

pindah ke branch production
```
git branch production
```
![](.Readme_images/cf72bd82.png)

# Setup for production

## Setup frontend

buat file .dockerignore

![](.Readme_images/752f04c6.png)

buat file .env untuk menghubungkan ke API backend
![](.Readme_images/827230b8.png)

buat Dockerfile

```yaml
FROM node:16-alpine as build
WORKDIR /home/app
COPY . .
RUN npm install --production
ARG REACT_APP_BASEURL="https://api.reiya.my.id/api/v1"
RUN npm run build

FROM node:16-alpine
WORKDIR /home/app
COPY --from=build /home/app /home/app
RUN npm install -g serve
EXPOSE 3000
CMD ["serve","-s","build"]
```
![](.Readme_images/4528fe34.png)

build Dockerfile
```yaml
docker build -t reiya24/dumbmerch-frontend-production . --progress=plain --no-cache
```
![](.Readme_images/58bebec7.png)

buat docker compose
```yaml
version: '3.7'
services:
  frontend-production:
    container_name: frontend-production
    image: reiya24/dumbmerch-frontend-production
    stdin_open: true
    restart: unless-stopped
    ports:
      - 3000:3000
```
![](.Readme_images/ebaa49f8.png)

jalankan docker compose
```yaml
docker compose up -d
```
![](.Readme_images/cfabbc1c.png)

## setup database

buat docker compose
```yaml
version: '3.7'
services:
  database-production:
    image: postgres:alpine
    container_name: database-production
    restart: unless-stopped
    environment:
      - POSTGRES_USER=reiya
      - POSTGRES_PASSWORD=reiya
      - POSTGRES_DB=reiya
    ports:
      - '5432:5432'
    volumes:
      - ~/konfigurasi_posgres_production:/var/lib/postgresql/data
```
![](.Readme_images/6ef82f9e.png)

jalankan docker compose
```yaml
docker compose up -d
```
![](.Readme_images/a99d8f27.png)

## setup backend

pastikan di branch production

buat .dockerignore
```yaml
.git
.gitignore
Dockerfile
docker-compose.yaml
```

![](.Readme_images/8d07cc74.png)

buat Dockerfile

```yaml
FROM golang:1.18-alpine as builder
WORKDIR /home/app
COPY . .
RUN go mod download
ARG DB_HOST=10.116.106.150
ARG DB_USER=reiya
ARG DB_PASSWORD=reiya
ARG DB_NAME=reiya
ARG DB_PORT=5432
ARG PORT=5000
RUN CGO_ENABLED=0 go build

FROM gcr.io/distroless/cc-debian11
WORKDIR /home/app
COPY --from=builder /home/app/dumbmerch /home/app
COPY --from=builder /home/app/.env /home/app
EXPOSE 5000
CMD ["/home/app/dumbmerch"]
```
![](.Readme_images/c2959924.png)

build Dockerfile
```yaml
docker build -t reiya24/dumbmerch-backend-production . --progress=plain --no-cache
```
![](.Readme_images/2c1719f4.png)

buat file docker compose

```yaml
version: '3.7'
services:
 backend-production:
  image: reiya24/dumbmerch-backend-production
  container_name: backend-production
  stdin_open: true
  restart: unless-stopped
  ports:
   - 5000:5000
```
![](.Readme_images/7c2728ba.png)

jalankan docker compose
```yaml
docker compose up -d
```
![](.Readme_images/ba3b1456.png)

integrasi backend dengan database berhasil berjalan 
![](.Readme_images/c7f93684.png)

![](.Readme_images/1eba9d83.png)

# setup for staging
lakukan hal yang kurang lebih sama dengan production,
namun dengan konfigurasi Dockerfile yang sedikit berbeda

## frontend
Dockerfile
```yaml
FROM node:12.16-alpine3.11 as build
WORKDIR /home/app
COPY . .
RUN npm install

FROM node:12.16-alpine3.11
WORKDIR /home/app
COPY --from=build /home/app /home/app
EXPOSE 3000
CMD ["npm","start"]
``` 
![](.Readme_images/d8e06fed.png)

docker compose:
![](.Readme_images/f1638d29.png)

## database
docker compose:
```yaml
version: '3.7'
services:
  database-staging:
    image: postgres:alpine
    container_name: database-staging
    restart: unless-stopped
    environment:
      - POSTGRES_USER=reiya
      - POSTGRES_PASSWORD=reiya
      - POSTGRES_DB=reiya
    ports:
      - '4400:5432'
    volumes:
      - ~/konfigurasi_posgres_staging:/var/lib/postgresql/data
```
![](.Readme_images/c9597110.png)

## backend
Dockerfile : 

```yaml
FROM golang:1.18-alpine as builder
WORKDIR /home/app
COPY . .
RUN CGO_ENABLED=0 go build

FROM gcr.io/distroless/cc-debian11
WORKDIR /home/app
COPY --from=builder /home/app/dumbmerch /home/app
COPY --from=builder /home/app/.env /home/app
EXPOSE 5000
CMD ["/home/app/dumbmerch"]
```
![](.Readme_images/4e4d4be9.png)


docker compose : 
![](.Readme_images/0472c050.png)

![](.Readme_images/b3b1d43a.png)

![](.Readme_images/da75f26d.png)

# CI CD menggunakan jenkins

## instalasi jenkins

copy password di
```yaml
docker logs jenkins
```
![](.Readme_images/28803a54.png)

paste ke website 

![](.Readme_images/8b1a139a.png)

pilih select plugin to install 

![](.Readme_images/a1134566.png)

pilih plugin yang diperlukan 
![](.Readme_images/7f884014.png)

tunggu sampai proses instalasi selesai 

![](.Readme_images/5a75e793.png)

masukan form 
![](.Readme_images/213a69ea.png)

setup domain 
![](.Readme_images/a1f465bc.png)

## setup credentials di jenkins
pada dashboard jenkins, pergi ke manage 
jenkins > manage credentials > global > 
add credentials 
![](.Readme_images/691326ae.png)

masukan private key 
![](.Readme_images/1bba2f6e.png)

## setup credentials di github
pada website github, pergi ke setting >
SSH and GPG keys > new SSH key, masukan public key 

![](.Readme_images/bafd1739.png)

SSH key berhasil ditambahkan 
![](.Readme_images/5b003e5e.png)

## setup agar bisa terkoneksi dengan github di koneksi pertama
pilih manage jenkins > configure global security 
![](.Readme_images/7b074937.png)

scroll ke bagian paling bawah, accept first connection 
![](.Readme_images/4b6cf3d5.png)

## Setup notifikasi dengan slack
install plugin slack notification, dashboard > manage jenkins, manage plugins 
![](.Readme_images/c72ffdb6.png)

pada available plugins, cari plugin slack notification, pilih 
download now and isntall after restart 
![](.Readme_images/c0e2476b.png)

tunggu sampai proses instalasi berhasil 
![](.Readme_images/affb719a.png)

buka halaman https://my.slack.com/services/new/jenkins-ci,
lalu pilih workspace

pilih channel 
![](.Readme_images/a8055be7.png)

setelah itu scroll kebawah, kita akan menemukan team subdomain, dan 
integration token credential ID 

![](.Readme_images/6e6fd5ed.png)

copy integration token credential ID, lalu tambahkan di credential
jenkins, pada menu kind, pilih secret text 
![](.Readme_images/319a634f.png)

setelah itu, pada dashboard jenkins > manage jenkins > configure 
system, cari menu slack, masukan form yang diperlukan, untuk 
workspace masukan team subdomain tadi, disarankan untuk test 
connection terlebih dahulu sebelum save 
![](.Readme_images/f4f2a7d4.png)

## membuat pipeline

klik create job, pilih pipeline job 

![](.Readme_images/6d5fef5c.png)

ceklis github hook trigger for scm polling 

![](.Readme_images/71bef397.png)

definition, pilih git, masukan repository, ssh ke y, dsb 

![](.Readme_images/44d73ba3.png)

matikan lightweight checkout

pada direktori frontend, buat Jenkinsfile

```groovy
pipeline {
    agent any

    environment{
        def branch = "production"
        def nama_repository = "origin"
        def directory = "~/fe-dumbmerch"
        def credential = 'id_rsa'
        def server = 'reiya24@10.116.106.150'
        def docker_image = 'reiya24/dumbmerch-frontend-production'
        def nama_container = 'frontend-production'
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }

    stages {

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "mulai job baru : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }

        stage('pull repository dari github ') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git pull ${nama_repository} ${branch}
                    exit
                    EOF"""
                }
            }
        }

        stage('hapus container') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    docker container stop ${nama_container}
                    docker container rm ${nama_container}
                    exit
                    EOF"""
                }
            }
        }

        stage('build image frontend') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker build -t ${docker_image}:latest .
                    exit
                    EOF"""
                }
            }
        }

        stage('jalankan docker compose') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose up -d
                    exit
                    EOF
                    """
                }
            }
        }
        
        stage('push image ke dockerhub') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker image push ${docker_image}:latest
                    exit
                    EOF"""
                }
            }
        }
    }

    post {

        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
    }
}
```

push ke github repository
![](.Readme_images/529ea348.png)

jenkins akan membuild secara otomtatis 
![](.Readme_images/557e29a2.png)

notifikasi berhasil
![](.Readme_images/a1a393a6.png)

lakukan hal yang sama untuk backend
![](.Readme_images/cd59c54a.png)

![](.Readme_images/1af851a3.png)

## multibranch pipeline

install plugin multibranch scan webhook trigger 
![](.Readme_images/c4d3557d.png)

pilih new item

masukan nama branch, pilih multibranch pipeline
![](.Readme_images/24796c37.png)

setup branch source
![](.Readme_images/48ec6f2b.png)

![](.Readme_images/5a199632.png)

ceklis scan by webhook 
![](.Readme_images/96b63eaa.png)

jenkins akan mengscan branch yang memiliki Jenkinsfile 
![](.Readme_images/f44c9e12.png)

```groovy
pipeline {
    agent any

    environment{
        def branch = "production"
        def nama_repository = "origin"
        def directory = "~/fe-dumbmerch"
        def credential = 'id_rsa'
        def server = 'reiya24@10.116.106.150'
        def docker_image = 'reiya24/dumbmerch-frontend-production'
        def nama_container = 'frontend-production'
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }

    stages {

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "mulai job baru : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }

        stage('pull repository dari github ') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git checkout ${branch}
                    git pull ${nama_repository} ${branch}
                    exit
                    EOF"""
                }
            }
        }

        stage('hapus container') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    docker container stop ${nama_container}
                    docker container rm ${nama_container}
                    exit
                    EOF"""
                }
            }
        }

        stage('build image frontend') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker build -t ${docker_image}:latest .
                    exit
                    EOF"""
                }
            }
        }

        stage('jalankan docker compose') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose up -d
                    exit
                    EOF
                    """
                }
            }
        }
        
        stage('push image ke dockerhub') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker image push ${docker_image}:latest
                    exit
                    EOF"""
                }
            }
        }
    }

    post {

        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
    }
}
```
![](.Readme_images/cfbf0705.png)

## setup github webhook

pada repository github > pilih setting > webhook, masukan
```groovy
https://jenkins.reiya.my.id/multibranch-webhook-trigger/invoke?token=backend
```
![](.Readme_images/f3bcfac1.png)

push ke github 

![](.Readme_images/9c01a688.png)

trigger berhasil 

![](.Readme_images/fed02b9a.png)
![](.Readme_images/3a366f61.png)
![](.Readme_images/1794a559.png)