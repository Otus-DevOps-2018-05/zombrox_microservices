# zombrox_microservices
zombrox microservices repository

Homework #16

Что сделано :

Как запустить проект:

1) Создать истанс в GCP для сервера gitlab

2) разрешить подключение к серверу по HTTP/HTTPS

Как проверить работоспособность:

##############################################################################

Homework #15

Что сделано :
Опробован запуск контейнеров с использованием различных сетевых драйверов (none, host)
Опробовны различные варианты запуска удаленных комманд на docker-host (docker exec, docker-machine ssh)
Запущены контейнеры с микросервисами в одной сети
Запущены контейнеры с микросервисами во front_net и back_net сетях
Для запуска контейнеров использован docker-compose
При помощи .env параметризованы: имя проекта, порт публикации, версии сервисов.

Ответы на вопросы:
1) Выводы команд 
docker exec -ti net_test ifconfig
docker-machine ssh docker-host ifconfig
идентичны
2) Запуск нескольких контейнеров с nginx на 80 порту приводит к тому что все кроме 1-го останавливаются т.к. 80 поорт уже занят.
3) Контейнер с драйверм none имеет 2 namespace (default и в моем случае 3d41fb8d2616), а с драйверм host только 1 (default) 
4) Для смены базового имени надо в .env добавить параметр COMPOSE_PROJECT_NAME=<project_name>

Как запустить проект:
# Позапускать контейнеры с различными драйверами и посмотреть, что получилось разными способами
docker run --network none --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
docker exec -ti net_test ifconfig

docker run --network host --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
docker exec -ti net_test ifconfig
docker-machine ssh docker-host ifconfig

# Запустить контейнер с сервисом на 80 порту
docker run --network host -d nginx
# посмотреть что запустилось
docker ps

# Остановить все запущенные контейнеры
docker kill $(docker ps -q)

# создать символическую ссылку на docker-host что бы ip netns видел namespace создаваемые docker
docker-machine ssh docker-host sudo ln -s /var/run/docker/netns /var/run/netns
# посмотреть создаваемые namespace
docker-machine ssh docker-host sudo ip netns

# Создать бридж 
docker network create reddit --driver bridge

# Запустить контейнеры с микросевисами в одной сети
docker run -d --network=reddit mongo:latest
docker run -d --network=reddit zombrox/post:1.0
docker run -d --network=reddit zombrox/comment:1.0
docker run -d --network=reddit -p 9292:9292 zombrox/ui:1.0

# Запустить контейнеры с микросевисами в одной сети с алиасами
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post zombrox/post:1.0
docker run -d --network=reddit --network-alias=comment zombrox/comment:1.0
docker run -d --network=reddit -p 9292:9292 zombrox/ui:1.0

# Создать подсети и запустить контейнеры 
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
docker run -d --network=front_net -p 9292:9292 --name ui zombrox/ui:1.0
docker run -d --network=back_net --name comment zombrox/comment:1.0
docker run -d --network=back_net --name post zombrox/post:1.0
docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest

# Подключить к заупущенным контейнерам по 2й сети.
docker network connect front_net post
docker network connect front_net comment

### Compose
Запустить контейнеры с микросервисами посредством compose
docker-compose up -d

Посмотреть что получилось
docker-compose ps

Как проверить работоспособность:

- В адресной строке браузера перейти по http://docker-host-ip-address-in-GCP:9292

##############################################################################

Homework #14

Что сделано :
- Написаны Dockerfile для разделенного на части приложения
- Собраны образы контейнеров разделенного приложения
  (сборка ui началась с 8го шага, остальные видимо собирать не имело смысла)
- Собраные контейнеры запущены на docker-host в CGP
- С целью уменьшения размера изменен Dockerfile  для ui контейнера, замен образ лежащий в основе.
  (Сборка началась с 1 первого шага)
- Создан и подключен docker volume для контенера с БД

Как запустить проект:
# Создать docker host 
## 1) задать переменную окружения
export GOOGLE_PROJECT=docker-zombrox

## 2) Создать истанс GCP в качестве docker-host 

docker-machine create \
--driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-d docker-host - для создания истанса для docker-host

## 3) 
eval $(docker-machine env docker-host) - для подключения к docker-host для запуска контейнеров

docker pull mongo:latest - для закгрузки на docker-host образа для БД
docker build -t zombrox/post:1.0 ./post-py
docker build -t zombrox/comment:1.0 ./comment
docker build -t zombrox/ui:2.0 ./ui - для сборки образов контейнеров с приложением

docker network create reddit - для создания сети приложения
docker volume create reddit_db - для создания docker volume для БД
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest - для запуска контейнера с БД
далее команды для запуска контейнеров с приложением:
docker run -d --network=reddit --network-alias=post zombrox/post:1.0
docker run -d --network=reddit --network-alias=comment zombrox/comment:1.0
docker run -d --network=reddit -p 9292:9292 zombrox/ui:2.0

docker kill $(docker ps -q) - для остановки контейнеров

docker-machine rm docker-host - для удаления истанса docker-host

Как проверить работоспособность:
- В адресной строке браузера перейти по http://docker-host-ip-address-in-GCP:9292

##############################################################################

Homework #13

Что сделано :
- Установлена docker-machine 
- создан инстанс в GCP для работы с docker
- подготовлены конфиги БД, приложения и скрипт запуска приложения
- написан Dockerfile
- подготовлен образ контейнера с приложением
- контенер запущен на doker хосте (инстансе GCP)
- создано правило файрвола для разрешения доступа на порт 9292 doker хоста
- созданный образ размещен на dockerhub
- контейнер перезапущен из dockerhub

Команды:
 docker run --rm -ti tehbilly/htop
 и
 docker run --rm --pid host -ti tehbilly/htop 
 различаются тем, что htop запускаемый в контейнере в первом случае имеет PID namespace контейнера и видит только те процессы что запущены в контейнере, а во втором PID namespace хоста на котором запускается контейнер и видит все процессы хоста.

Как запустить проект:
- выполнить команду на localhost:~$ export GOOGLE_PROJECT=docker-zombrox
-  выполнить команду на localhost:~$ 
docker-machine create \
--driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-d docker-host

для создания инстанса GCP для работы с docker
- для запуска контейнера docker run --name reddit -d --network=host reddit:latest
- для создания правила файрвола выполнить на localhost:~$ gcloud compute firewall-rules create reddit-app --allow tcp:9292 --target-tags=docker-machine --description="Allow PUMA connections" --direction=INGRESS
- для размещения образа в dockerhub :
docker tag reddit:latest zombrox/otus-reddit:1.0
docker push zombrox/otus-reddit:1.0
- запуска контейнера на основе образа из dockerhub
sudo docker run --name reddit -d -p 9292:9292 zombrox/otus-reddit:1.0

Как проверить работоспособность:
- В адресной строке браузера перейти по http://docker-host-ip-address-in-GCP:9292

##############################################################################
Homework #12

Что сделано :
- Установлен Docker
- Опробованы различные команды по работе с контейнерами
 
Как запустить проект:
-

Как проверить работоспособность:
-
