#### Docker для начинающих + практический опыт
### https://stepik.org/course/123300/syllabus

### Как узнать версию ОС в контейнере
```bash
# Смотрим основной файл информации об ОС: /etc/os-release
cat /etc/*rel*

# Смотрим в контейнере запущенном в режиме -d
docker exec f8005df898d6 cat /etc/*rel* | grep PRETTY_NAME

# Выполняем команду после создания и смотрим какая версия ОС в контейнере
docker run alpine:3.8 cat /etc/*rel*
```

### Команда docker run
```bash
# Запускаем с режиме псевдо-терминала с ключем -d
docker run -t rotorocloud/prompt-docker

# Добавляем интерактивный режим (перенаправление ввода с консоли в контейнер) с помощью ключа -i
docker run -it rotorocloud/prompt-docker

# Автоматический пайп
echo "rotoro" | docker run -i rotorocloud/prompt-docker

# Обучающий модуль rotorocloud
docker run -d -p 30123:8080 --name=webapp rotorocloud/simple-webapp-rockets:v2

# Удаление контейнеров
docker rm -f $(docker container ls -aq)
docker rm -f $(docker container ps -aq)
docker rm -f $(docker ps -aq)

# Создание контейнера без запуска
docker create --name redis redis

# Посмотреть статистику потребляемых ресурсов в виде таблицы
docker stats

# Копирование из контейнера webapp на хост
docker container cp webapp:/etc/nginx /tmp/

# Запуск контейнера mysql
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=db_pass123 mysql
# Запуск контейнера mysql с монтированием volume
docker run -v /opt/data:/var/lib/mysql -d --name mysql-db -e MYSQL_ROOT_PASSWORD=db_pass123 mysql
# Посмотреть содержимое таблицы
docker exec mysql-db mysql -pdb_pass123 -e 'use foo; select * from authors'
```

### Запуск Jenkins в контейнере
```bash
# Запуск Jenkins в контейнере (версия latest в данном случае не является последней)
# Делаем docker inspect jenkins и получаем ip и порт внутри контейнера, после чего заходим на него через браузер
# 172.17.0.2:8080
docker run --name jenkins jenkins/jenkins:lts-alpine
docker inspect jenkins | inspect jenkins
# Либо останавливаем контейнер и делаем пробро портов
docker run --name jenkins -p 8080:8080 jenkins/jenkins:lts-alpine
# По умолчанию Jenkins хранит информацию в каталоге пользователя, а значит при запуске внутри контейнера - в контейнере. Нужно создать внешний том для сохранения настроек и запустить от админа, чтобы была возможность делать запись на диск
docker cp jenkins:/var/jenkins_home /home/$USER/jenkins_home
docker run --name jenkins -p 8080:8080 -p 50000:50000 -v /home/$USER/jenkins_home:/var/jenkins_home -u root jenkins/jenkins:lts-alpine
```

### Особенности команды docker run
```bash
# Контейнер закроется через 20 секунд (CTRL+C не позволит выйти из контейнера)
# Выйти можно только через docker stop
docker run alpine sleep 20
```

### Проброс портов
```bash
# Порт хоста 80, порт в контейнере 5000
docker run -p 80:5000 rotorocloud/webapp
```

### Volumes
```bash
# Связываем директорию /opt/datadir на хосте с директорией /var/lib/mysql в контейнере
docker run -v /opt/datadir/:/var/lib/mysql mysql:8.0.34-debian
```

### Получить информацию о контейнере
```bash
docker inspect 17f3d5eb5cc7

# Посмотреть mac-адрес/ip-адрес контейнера
docker inspect webportal | grep MacAddress
docker inspect webportal | grep IPAddress
```

### Просмотр логов
```bash
docker logs 17f3d5eb5cc7
```

## Образы Docker

### Создание своего образа
```bash
docker build . -t rotorocloud/webapp

# Отправить образ на dockerhub
docker login
docker push rotorocloud/webapp

# Посмотреть историю и место которое отъел каждый слой образа можно с помощью истории
docker history rotorocloud/webapp
```

### Сборка образа из текущей директории с Dockerfile
```bash
docker build - < .Dockerfile
# Замена entrypoint
# docker run --entrypoint super-sleep ubuntu-sleeper 10
```

### Docker Compose
```bash
# Как мы собираем стек приложений без compose
# Voting App
# Устаревшая конструкция link
docker run -d --name redis red
docker run -d --name db postgres
docker run -d --name vote -p 5000:80 --link redis:redis voting-app
docker run -d --name result -p 5001:80 --link db:db result-app
docker run -d --name worker worker

# Docker compose
# Увеличиваем кол-во реплик контейнера, что в итоге поднимет 2 контейнера
docker-compose up -d --scale rocketcounter=2
```

### Удаленное управление docker на другой машине
```bash
docker -H=192.168.56.1:2375 run nginx
```

### Изоляция процессов PID Namespace
```bash
# Процессы в контейнере на самом деле выполняются на хосте под своим PID
# Далее идут различные команды просмотра и задания ресурсов
docker run --cpus=.5 --memory=100m nginx
docker run -d --name alpine-100mb --memory 100m alpine top
docker run -d --name stress25 --cpuset-cpus 0 --cpu-shares 256 benhall/stress
docker run -it --net=host alpine ip addr show

# Посмотреть потребляемые ресурсы контейнерами
docker stats --no-stream

# Запуск контейнера в обычном режиме изоляции и в PID-пространстве хоста
docker run -it alpine ps aux
docker run -it --pid=host alpine ps aux

# Запуск с использование пространства имен контейнера http
docker run --net=container:http benhall/curl curl -s localhost

# Запуск в конкретном процессе
docker run --pid=container:http alpine ps aux

# Отслеживание системных вызовов
strace
```

### Хранилище и файловая система
```bash
# Существуют слои контейнера и слои образа

# Создаем volume. По факту создается папка /var/lib/docker/volumes/date_volume
docker volume create data_volume
# Если мы не выполнили команду выше, docker все равно все поймет и создаст папку где нужно
# Монтируем (volume mounting)
docker run -v data_volume:/var/lib/mysql mysql

# Монтируем с привязкой из любого места на хосте (bind mounting)
docker run -v /data/mysql:/var/lib/mysql mysql

# --mount - новый способ монтирования вместо -v
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

### Сеть в Docker
```bash
# По умолчанию создаются 3 сети: bridge (мост), none, host
# Bridge между контейнерами 172.17.0.x
# None: docker run webapp --network=none
# Host вариант убирает необходимость пробрасывать порты. Порты становятся общими из хоста. В данном случае не получится поднять несколько контейнеров одного приложения
# docker run webapp --network=host

# Изолируем контейнеры
docker network create --driver bridge --subnet 182.18.0.0/16 my-custom-network
docker network create --driver bridge --subnet 172.22.0.1/24 --gateway 172.22.0.1 wp-mysql-network
docker run -d -e MYSQL_ROOT_PASSWORD=db_pass123 --name mysql-db --network wp-mysql-network mysql
docker run --network=wp-mysql-network -e DB_Host=mysql-db -e DB_Password=db_pass123 -p 30080:8080 --name webapp -d rotorocloud/webapp-mysql
# Смотрим сети
docker network ls
# Cмотрим какая сеть в контейнере
docker inspect webserver
# Можно подключить контейнер к сети и также отключить его (feature - имя контейнера)
docker network connect stage-ns feature
docker network disconnect stage-ns feature
```