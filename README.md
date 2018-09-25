# zombrox_microservices
zombrox microservices repository

Homework #15

Что сделано :

Как запустить проект:

Как проверить работоспособность:

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

export GOOGLE_PROJECT=docker-zombrox

docker-machine create \
--driver google \
--google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-d docker-host - для создания истанса для docker-host

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
