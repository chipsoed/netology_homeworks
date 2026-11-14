Задача 3.
Чтобы подключиться к стандартным потокам ввода/вывода/ошибок (stdin, stdout, stderr) работающего контейнера, используется команда docker attach. Команда подключает текущий терминал к основному процессу (PID 1) внутри контейнера. Выводится поток вывода контейнера в реальном времени.
Нажатие Ctrl+C отправляет сигнал SIGKILL основному процессу и остановливает контейнер. Поэтому в выоде docker ps -a этот контейнер со статусом "Exited"
root@e04e8232c9f6:/# nginx -s reload
2026/11/09 17:17:01 [notice] 192#192: signal process started
root@e04e8232c9f6:/# curl http://127.0.0.1:80
curl: (7) Failed to connect to 127.0.0.1 port 80 after 0 ms: Couldn't connect to server
root@e04e8232c9f6:/# curl http://127.0.0.1:81
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I will be DevOps Engineer!</h1>
</body>
</html>
root@e04e8232c9f6:/# 
exit
root@netology:/home/user/custom_docker_images/custom_nginx# ss -tlpn | grep 127.0.0.1:8080
LISTEN    0         4096             127.0.0.1:8080             0.0.0.0:*        users:(("docker-proxy",pid=18070,fd=7))                                        
root@netology:/home/user/custom_docker_images/custom_nginx# docker port custom-nginx-t2
80/tcp -> 127.0.0.1:8080
root@netology:/home/user/custom_docker_images/custom_nginx# curl http://127.0.0.1:8080
curl: (56) Recv failure: Connection reset by peer
root@netology:/home/user/custom_docker_images/custom_nginx# 
По условиям задачу необходимо было изменить файл конфигурации nginx - поменять слушающий порт с 80 на 81. После выполнения изменений вывод команды ss -tlpn | grep 127.0.0.1:8080 отображает слушающий порт 8080 на хост системе. В данном выводе ниче не изменилосб, так как команда выволпняется только для хост системы и не видит что порт 8080 пробрасывает на порт контейнера
Вывод команды docker port custom-nginx-t2 отображает текущую настройку проброса портов для нашего контейнера. При первом запуске контейнера мы указывали, что порт 8080 хостовой системы будет пробрасывать подключение на 80 порт контейнера. Эту настройку мы не меняли, поэтому здесь никаких проблем нет. 
Команда curl http://127.0.0.1:8080 отображает сброс соединения целевым хостом. Так происходит, потому что порт 80 на контейнере перестал слушать после изменения конфигурационного файла. Теперь контейнер слушает на 81 первом порту. 

Для восстановления доступности веб-страницы на контейнере, без изменения конфигурационного файла nginx, необходимо лишь изменить проброс 8080 порта хоста с 80 на 81 порт контейнера. Но для этого необходим перезапуск контейнера. Изменить проброс портов у уже запущенного контейнера без его пересоздания невозможно.

Задача 4.
root@netology:/home/user/custom_docker_images/custom_nginx# docker pull debian:stable-slim
stable-slim: Pulling from library/debian
2ec981fa50fa: Pull complete 
Digest: sha256:fbfb4059d2127120aa05b6e6064523d2102552f0cf0026ddf0701c12f1f8f3e0
Status: Downloaded newer image for debian:stable-slim
docker.io/library/debian:stable-slim
root@netology:/home/user/custom_docker_images/custom_nginx# docker images
REPOSITORY            TAG              IMAGE ID       CREATED         SIZE
chip08/custom-nginx   1.0.0            5fd2c38e3ed4   22 hours ago    192MB
custom-nginx          1.0.0            5fd2c38e3ed4   22 hours ago    192MB
debian                stable-slim      ea782539d7ae   5 days ago      78.9MB
nginx                 1.29.0           7a073be66c4c   16 months ago   192MB
centos                centos7.9.2009   eeb6ee3f44bd   5 years ago     204MB
root@netology:/home/user/custom_docker_images/custom_nginx# cd ../..
root@netology:/home/user# docker run -d --name centos-container -v "$(pwd)":/data centos:centos7.9.2009
7a25b080f704cd00eb37ad52c3b117f51eda69f312d5cb6ef505e46efeaba91a
root@netology:/home/user# docker run -d --name debian-container -v "$(pwd)":/data debian:stable-slim
862ecb67aeb61049abace52048025fb26ce4f2f5809daf8a8a276fdd3596a755
root@netology:/home/user# docker exec -it centos-container bash
Error response from daemon: container 7a25b080f704cd00eb37ad52c3b117f51eda69f312d5cb6ef505e46efeaba91a is not running
root@netology:/home/user# docker ps -a
CONTAINER ID   IMAGE                   COMMAND       CREATED         STATUS                     PORTS     NAMES
862ecb67aeb6   debian:stable-slim      "bash"        6 minutes ago   Exited (0) 6 minutes ago             debian-container
7a25b080f704   centos:centos7.9.2009   "/bin/bash"   7 minutes ago   Exited (0) 7 minutes ago             centos-container
root@netology:/home/user# docker exec -it debian-container bash
Error response from daemon: container 862ecb67aeb61049abace52048025fb26ce4f2f5809daf8a8a276fdd3596a755 is not running
root@netology:/home/user# docker rm centos-container debian-container
centos-container
debian-container
root@netology:/home/user# docker run -d --name centos-container -v "$(pwd)":/data centos:centos7.9.2009 tail -f /dev/null
c9da9d67c1ecdb6b7f8f5637ffc14c24720a3a5fac9a10f7dc89b9b504c3b565
root@netology:/home/user# docker run -d --name debian-container -v "$(pwd)":/data debian:stable-slim tail -f /dev/null
287e2d710c78e4236832bb1563386254347ebc726ebb928001db5c2f7196eb68
root@netology:/home/user# docker exec -it centos-container bash
[root@c9da9d67c1ec /]# cd /data/
[root@c9da9d67c1ec data]# nano test 
bash: nano: command not found
[root@c9da9d67c1ec data]# vi test
[root@c9da9d67c1ec data]# cat test
test message
[root@c9da9d67c1ec data]# exit
root@netology:/home/user# ls
custom_docker_images  Desktop  Documents  Downloads  get-docker.sh  Music  Pictures  Public  snap  Templates  test  Videos  yandex-cloud
root@netology:/home/user# nano test2
root@netology:/home/user# docker exec -it debian-container bash
root@287e2d710c78:/# ls /data/
Desktop  Documents  Downloads  Music  Pictures	Public	Templates  Videos  custom_docker_images  get-docker.sh	snap  test  test2  yandex-cloud
root@287e2d710c78:/# 


Задача 5.
1. Если в одной директории существуют два файла .yaml, Compose отдаёт предпочтение compose.yaml . Поэтому при выполнении docker compose up -d без явного указания файла система автоматически выбрала compose.yaml. Развернется один контейнре - portainer.
2. Директива include позволяет встроить один файл Compose в другой . Когда запускаетcz docker compose up -d, Compose читает основной файл compose.yaml, видит там инструкцию include и подгружает указанный файл docker-compose.yam, добавляя все его серdbcs в общую модель приложения.
3. 
7. rm compose.yaml 
root@netology:/tmp/netology/docker/task5# docker compose up -d
WARN[0000] /tmp/netology/docker/task5/docker-compose.yaml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
WARN[0000] Found orphan containers ([task5-portainer-1]) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up. 
[+] Running 1/1
 ✔ Container task5-registry-1  Running
Программа успешно выполнилась, но Docker Compose выдал два предупреждения, потому что удален один из файлов и теперь конфигурация проекта изменилась.
1-ый WARN: В файле docker-compose.yaml, который теперь единственный, в первой строке написано version: "3". Параметр version не обязателен. Docker предупреждает: «Я вижу эту строчку, но я ее пропускаю. Уберите ее, чтобы не путаться». На работу это никак не влияет.
2-ой WARN: Docker Compose управляет контейнерами, группируя их в «проекты».
Раньше, когда было два файла, Docker знал, что проект task5 состоит из двух сервисов.
После удаления compose.yaml Docker читает только docker-compose.yaml, где описан только registry.
Docker видит, что в проекте task5 все еще работает контейнер task5-portainer-1, но в текущем конфигурационном файле его больше нет. Docker называет такие контейнеры «сиротами» (orphans) — они относятся к проекту, но их «родителя» (описания в файле) больше не существует.
