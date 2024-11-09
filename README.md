# Grafana-Prometheus-VictoriaMetrics
## Описание
<br>ПО: VirtualBox, Docker + Docker Compose, Grafana, Prometheus, NodeExporter; ОС: OracleLinux.
## Начало
<br>Установка утилиты "wget", используя менеджер пакетов "yum"
```
sudo yum install wget
```
<br>![image](https://github.com/user-attachments/assets/9951d9cc-baa5-40ef-a10d-90d282437442)

<br>Установка файла с указанного репозитория и сохранение его в директорию `/etc/yum.repos.d/`. Этот файл имеет информацию о репозиториях, откуда выгружаются пакеты Docker.
```
sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo
```
<br>![image](https://github.com/user-attachments/assets/2467a30f-3f85-48be-8e6e-c2510ed48499)

<br>Устанавлием непосредтсвенно пакеты Docker (сам Docker Engine, Docker CLI и Containerd).
+ `docker-ce` — это основной пакет Docker Community Edition.
+ `docker-ce-cli` — это командная строка Docker, которая позволяет взаимодействовать с Docker через терминал.
+ `containerd.io` — это контейнерный демон, который управляет жизненным циклом контейнеров.

```
sudo yum install docker-ce docker-ce-cli containerd.io
```
<br>![image](https://github.com/user-attachments/assets/72621cc0-bb2a-40ee-bf7e-7b2e391e0cf5)

<br>Подтверждаем загрузку и установку пакетов
<br>![image](https://github.com/user-attachments/assets/3cb80880-8ab9-4ad4-a428-96d3f4102f85)

<br> Автозагрузка Docker
```
sudo systemctl enable docker --now
```
<br>![image](https://github.com/user-attachments/assets/62f7d7a3-8d86-4f62-9792-cbd204ea9528)

<br>Установка curl
```
sudo yum install curl
```
<br>![image](https://github.com/user-attachments/assets/2e87a596-901d-4f15-9e7f-5871bdd64425)

<br>Обращаемся к репозиторию, чтобы получить последнюю версию Docker Compose.
<br>Обратите внимание(!), что команду нужно вводить без sudo, потому что она только получает информацию о версии и не требует прав суперпользователя для выполнения.
```
COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
```

<br>Загружаем исполняемый файл Docker Compose соответствующей версии для вашей системы.
```
sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
```
<br>![image](https://github.com/user-attachments/assets/2a7d1fb6-a372-4f7f-ab4b-bc734d20a425)

<br>Изменяем прва доступа для файла (права на выполнение как исполняемого файла).
```
sudo chmod +x /usr/bin/docker-compose
```

<br>Показывает версию Docker Compose.
```
docker-compose --version
```
<br>![image](https://github.com/user-attachments/assets/5ca8b3a1-b910-4205-a900-6b26e5398e95)
## Git+Grafana
Эти конфиги есть в репозитории на всякий пожарный.
<br>Установка гита + клонирование репозитория
```
git clone https://github.com/skl256/grafana_stack_for_docker.git
```
<br>![image](https://github.com/user-attachments/assets/27ccb083-fb4d-4fa5-8ff3-95cba5ce52be)

<br>Переходим в папку `grafana_stack_for_docker`
```
cd grafana_stack_for_docker
```
<br>![image](https://github.com/user-attachments/assets/cc13abcb-cc4c-4bae-b605-841f5ed3218b)

<br>Создаем директоррию `config`
```
sudo mkdir -p /mnt/common_volume/swarm/grafana/config
```

<br>Создаем несколько диреткорий для хранения конфигурационных файлов и данных от Grafana.
```
sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}
```

<br>Меняет владельца на того, кто выполнил команду (для управления).
```
sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}
```

<br>Если файла нет - создает, если есть изменяет временные метки доступа до текущего времени, не изменяя содержимое.
```
touch /mnt/common_volume/grafana/grafana-config/grafana.ini
```

<br>Копируем **все файлы** из диреткории `config` в директорию `/mnt/common_volume/swarm/grafana/config/`.
```
cp config/* /mnt/common_volume/swarm/grafana/config/
```

<br>Переименовываем файл `grafana.yaml` в `docker-compose.yaml`. (можно проверить через ls)
```
mv grafana.yaml docker-compose.yaml
```
<br>![image](https://github.com/user-attachments/assets/5ccee708-440a-46ab-8d24-315dde6eb59e)

<br>Поднимаем Docker Compose.
```
sudo docker compose up -d
```
<br>![image](https://github.com/user-attachments/assets/33dd3b04-9fa8-4660-b792-89f6e8eceda2)
<br>![image](https://github.com/user-attachments/assets/04a77c2c-6eb4-48f3-af39-80a5235c01db)
## Работа в Grafana
<br>Находится на `localhost:3000`
<br>Данные для авторизации:
<br>Username: admin
<br>Password: admin
<br>![image](https://github.com/user-attachments/assets/d321c724-4eb1-4718-bb14-ec1120be0cb6)

<br>Тут для того чтобы брать откуда-то данные вставляем экспортер.
```
cd grafana_stack_for_docker
```
```
sudo vi docker-compose.yaml
```


```node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
```
<br>А затем в `/mnt/common_volume/swarm/grafana/config/prometheus.yaml` меняем targets:(тут айпишник) на **exporter:9100**.
<br>ПОСЛЕ ВСЕГО ЭТОГО НАДО ПЕРЕСОБРАТЬ ДОКЕР КОМПОЗ!!!
```
sudo docker-compose stop
```
<br>![image](https://github.com/user-attachments/assets/5e314a50-abc0-4292-916b-5a0358634413)
```
sudo docker compose up -d
```
<br>![image](https://github.com/user-attachments/assets/00442035-ce11-49f9-a879-10c10286c46c)

<br>Для того чтобы проверить что все работает можно пройтись по:
<br>Grafana
```
localhost:3000
```
<br>Prometheus
```
localhost:9090
```
<br>Node Exporter
```
localhost:9100
```
## Prometheus
<br>Жмем кнопку Create Dashboards
<br>![image](https://github.com/user-attachments/assets/22dc275c-6ec1-4eba-b1d5-796624624876)
<br>Жмем кнопку +Add visualization, а после "Configure a new data source"
<br>![image](https://github.com/user-attachments/assets/d7b2d2e3-7bd5-431f-a88e-16b9c672fd77)
<br>Выбираем Prometheus
<br>![image](https://github.com/user-attachments/assets/892a9e5a-f82d-422b-8ba1-0a9b69f4c94d)
<br>Connection
```
http://prometheus:9090
```
<br>В Authentication выбираем **Basic authentication** и задаем логин и пароль:
<br>User: admin
<br>Password:admin
<br>![image](https://github.com/user-attachments/assets/ddc8eac4-85f5-4a08-961e-9d7309c4601d)

<br>Потом жмем на плюсик справа сверху и выбираем **Import Dashboard**. Вводим 1860, жмем Load.
<br>![image](https://github.com/user-attachments/assets/3fc6a778-4966-4301-88cb-45ad79b0dd88)
<br>Выбираем Prometheus
<br>![image](https://github.com/user-attachments/assets/b290c8ef-a4e4-4e7f-8991-7a2a5d1a9034)
<br> Итог:
<br>![image](https://github.com/user-attachments/assets/c1bf86f4-beeb-446e-b661-2c3de31b44b1)
## VictoriaMetrics
<br>Останавливаем докер.
```
sudo docker-compose stop
```
<br>Переделываем `docker-compose.yaml`. Убираем все лишнее а ниже код к VictoriaMetrics, который надо **добавить**.
```
 victoriametrics:
    container_name: victoriametrics
    image: victoriametrics/victoria-metrics:v1.105.0
    ports:
      - 8428:8428
      - 8089:8089
      - 8089:8089/udp
      - 2003:2003
      - 2003:2003/udp
      - 4242:4242
    volumes:
      - vmdata:/storage
    command:
      - "--storageDataPath=/storage"
      - "--graphiteListenAddr=:2003"
      - "--opentsdbListenAddr=:4242"
      - "--httpListenAddr=:8428"
      - "--influxListenAddr=:8089"
      - "--vmalert.proxyURL=http://vmalert:8880"
    networks:
      - vm_net
    restart: always
```
<br>Снова поднимаем
```
sudo docker compose up -d
```
<br>![image](https://github.com/user-attachments/assets/81b0532f-8e9a-4bc7-ab85-2f9163f8a043)
<br>Команда отправляет метрику OILCOINT_metric2 с типом gauge и значением 0 на локальный сервер Prometheus по адресу http://localhost:8428/api/v1/import/prometheus. Она использует echo для формирования данных и curl для их передачи.
```
echo -e "# TYPE OILCOINT_metric2 gauge\nOILCOINT_metric2 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus
```
<br>Команда выполняет GET-запрос к локальному серверу Prometheus для получения значения метрики OILCOINT_metric2.
```
curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric2'
```
<br>Повторяем команду, но со значением 50, чтобы увидеть изменения.
```
echo -e "# TYPE OILCOINT_metric2 gauge\nOILCOINT_metric2 50" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus
```
<br>Открываем VictoriaMetrics на `localhost:8428`
<br>![image](https://github.com/user-attachments/assets/e2f7ab46-4584-43ab-a90b-51632fc8bf3f)
<br>Переходим в `vmui - Web UI` и видим отображение наших запросов, значит данные доходят.
<br>![image](https://github.com/user-attachments/assets/6605f833-1468-4b91-a678-8746011f4583)
<br>В графане надо будет вместо нодэкспортера поставить викториаметрикс, потом посмотреть переменную которую мы кидали в викториюэхом и курлом.
<br>![image](https://github.com/user-attachments/assets/15d86f28-e504-476f-a62f-676b47a98f3b)
