# Netology Docker Homework

Автор: Pertsev Maxim  

---

## ✅ Задача 1

### Установка Docker и Compose
```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable docker --now
```

### Настройка зеркал (если недоступен DockerHub)
```bash
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://mirror.gcr.io",
    "https://daocloud.io",
    "https://c.163.com/",
    "https://registry.docker-cn.com"
  ]
}
EOF
sudo systemctl restart docker
```

### DockerHub
- Зарегистрирован аккаунт `maxpert`
- Создан репозиторий `custom-nginx`

### Dockerfile
```Dockerfile
FROM nginx:1.21.1
COPY index.html /usr/share/nginx/html/index.html
```

### index.html
```html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
```

### Сборка и пуш образа
```bash
docker build -t custom-nginx:1.0.0 .
docker tag custom-nginx:1.0.0 maxpert/custom-nginx:1.0.0
docker push maxpert/custom-nginx:1.0.0
```

---

## ✅ Задача 2

```bash
docker run -d --name SHDH-custom-nginx-t2 -p 127.0.0.1:8080:80 maxpert/custom-nginx:1.0.0
docker rename SHDH-custom-nginx-t2 custom-nginx-t2

date +"%d-%m-%Y %T.%N %Z"
sleep 0.150
docker ps
ss -tlpn | grep 127.0.0.1:8080
docker logs custom-nginx-t2 -n1
docker exec -it custom-nginx-t2 base64 /usr/share/nginx/html/index.html
curl http://127.0.0.1:8080
```

---

## ✅ Задача 3

```bash
docker attach custom-nginx-t2
# нажимаем Ctrl+C

docker ps -a
# контейнер остановился, т.к. получил SIGINT

docker start custom-nginx-t2
docker exec -it custom-nginx-t2 bash
apt update && apt install -y nano

nano /etc/nginx/conf.d/default.conf
# меняем listen 80 на listen 81

nginx -s reload
curl http://127.0.0.1:80
curl http://127.0.0.1:81
exit
```

### Проверка
```bash
ss -tlpn | grep 127.0.0.1:8080
docker port custom-nginx-t2
curl http://127.0.0.1:8080
```

---

## ✅ Задача 4

```bash
docker run -d --name centos-container -v $(pwd):/data centos tail -f /dev/null
docker run -d --name debian-container -v $(pwd):/data debian tail -f /dev/null

docker exec -it centos-container bash
echo "Hello from CentOS" > /data/from-centos.txt
exit

echo "Hello from HOST" > ./from-host.txt

docker exec -it debian-container bash
ls -la /data
cat /data/from-centos.txt
cat /data/from-host.txt
```

---

## ✅ Задача 5

### Структура
- `compose.yaml`
```yaml
version: "3"
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

- `docker-compose.yaml`
```yaml
version: "3"
services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
```

### Объединение в один compose.yaml
```yaml
version: "3"
services:
  portainer:
    network_mode: host
    image: portainer/portainer-ce:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  registry:
    image: registry:2
    ports:
      - "5000:5000"
```

### Публикация образа в локальный registry
```bash
docker tag custom-nginx:1.0.0 127.0.0.1:5000/custom-nginx
docker push 127.0.0.1:5000/custom-nginx
```

### Portainer
- Открыть http://127.0.0.1:9000
- Создать пароль, выбрать "local" окружение
- Зайти в **Stacks → Web Editor** и вставить:
```yaml
version: "3"
services:
  nginx:
    image: 127.0.0.1:5000/custom-nginx
    ports:
      - "9090:80"
```

### Проверка и ошибка orphan контейнера
```bash
rm compose.yaml
docker compose up -d
# ⚠️ Warning: Found orphan containers...
docker compose down
```
