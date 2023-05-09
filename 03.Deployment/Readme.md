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