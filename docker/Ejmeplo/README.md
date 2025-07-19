docker build -t hola-docker-js .
docker run -p 4000:3000 hola-docker-js

docker-compose up --build


pwd # para obtener la ruta completa en mi vps
scp -r . root@129.22:/root/app # para copiar toda mi carpeta en mi vps