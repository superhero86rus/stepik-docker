#### Docker для начинающих + практический опыт
### https://stepik.org/course/123300/syllabus

### Как узнать версию ОС в контейнере
```bash
# Смотрим основной файл информации об ОС: /etc/os-release
cat /etc/*rel*

# Смотрим в контейнере запущенном в режиме -d
docker exec f8005df898d6 cat /etc/*rel* | grep PRETTY_NAME

# Выполняем команду после создания
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