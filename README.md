# zombrox_microservices
zombrox microservices repository

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