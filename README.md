# Docker

Docker
https://habr.com/ru/post/272145/
https://training.play-with-docker.com/ops-s1-hello/
#01
* docker-machine create --driver virtualbox Char
		--driver flag to indicate which provider

#02
* docker-machine ip Char

#03
При использовании Docker Machine ваш механизм Docker работает в виртуальной машине, которая фактически является удаленной машиной, поэтому для подключения к ней необходимо настроить локальный интерфейс командной строки. Docker Machine знает сведения о подключении для механизмов, которыми управляет, поэтому при запуске docker-machine env по умолчанию выводятся сведения о компьютере по умолчанию

$ docker-machine env default
 export DOCKER_TLS_VERIFY="1"
 export DOCKER_HOST="tcp://172.16.62.130:2376"
 export DOCKER_CERT_PATH="/Users/elton/.docker/machine/machines/default"
 export DOCKER_MACHINE_NAME="default"

Использование eval выполняет каждую из этих команд экспорта, а не просто записывает их в консоль, так что это быстрый способ настройки переменных среды.


* eval $(docker-machine env Char)

для проверки 
> docker-machine ls
В active будет *

Using eval executes each of those export commands, instead of just writing them to the console, so it's a quick way of setting up your environment variables.
You can undo it and reset the local environment with 
docker-machine env --unset
, which gives you the output for unsetting the environment (so the CLI will try to connect to the local Docker Engine).

#04
Находим контейнер , юзаем ссылку
https://hub.docker.com/_/hello-world

* docker pull hello-world

Проверяем скачаные изображения 
Docker images

#05
Запустим контейнер “Hello-World”

* docker run hello-world

#06

*  docker run --name overlord --restart=always --detach --publish 5000:80 nginx

Docker ru nginx запускает контейнер nginx
-d, --detach. Запуск контейнера в фоновом режиме. Напишет его id
-p, --publish list опубликует порты контейнера для хоста
-p is a ports mapping <HOST PORT>:<CONTAINER PORT>.
--name string — присваивает имя контейнеру
--restart Указывает политику перезапуска контейнера
=always Всегда перезапускает контейнер независимо от состояния выхода.

для проверки http://192.168.99.100:5000/

#07

*  docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' overlord

Return low-level information on Docker objects
--format , -f		Отформатировать вывод, используя данный шаблон

#08

* docker run -it --rm alpine /bin/sh

- --rm Automatically remove the container when it exits (docker run --help)
- -i Interactive mode (Keep STDIN open even if not attached)
- -t Allocate a pseudo-TTY

#09
* apt-get update
* apt-get upgrade
* apt-get install build-essential -y #компилятора пакетик

 Но сначала зайдем ка в контейнер
docker run -it --rm debian

#10

* docker volume create --name hatchery

Volume – это дисковое пространство между хостом и контейнером. Проще – это папка на вашей локальной машине примонтированная внутрь контейнера. Меняете тут меняется там, и наоборот, миракл.

#11

* docker volume ls

ls        -  List volumes

#12

* docker run -d --restart=on-failure --name spawning-pool --volume hatchery:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Kerrigan -e MYSQL_DATABASES=zerglings mysql:5.7

-d -фоновый режим
--restart=on-failure - авторестарт при неудачном запуске
--name spawning-pool -имя контейнера который создаем
--volume hatchery:/var/lib/mysql -место куда сохраняется бд
-e MYSQL_ROOT_PASSWORD=Kerrigan - через среду задаем пароль для рута бд
-e MYSQL_DATABASES=zerglings - через среду задаем имя бд
mysql:5.7 используем конкретную версию что б не юзать плагин для установки пароля

#13

* docker exec spawning-pool env

Команда 'docker exec' применяется к запущенному контейнеру, запускает новый процесс внутри пространства процессов контейнера.

# 14

* docker run -d --name lair --publish 8080:80 --link spawning-pool:mysql wordpress

-d фоновый режим
—publish - редирект портов
—link ссылка на другой контейнер. В нашем случае нам нужна база данных в spawning-pool

#15
* docker run -d --name roach-warden --publish 8081:80 --link spawning-pool:db phpmyadmin/phpmyadmin

phpMyAdmin — веб-приложение с открытым кодом, написанное на языке PHP и представляющее собой веб-интерфейс для администрирования СУБД MySQL.

-d фоновый режим
—publish - редирект портов
--link spawning-pool:db -присоединяем контейнер
phpmyadmin/phpmyadmin - такое название потому что гладиолус https://hub.docker.com/r/phpmyadmin/phpmyadmin/

#16

* docker logs spawning-pool --follow

--follow - непрерывный вывод логов контейнера

#17

* docker ps

по дефолту только запущенные машины

#18
* docker restart overlord

#19
* docker run --name Abathur -v ~/:/root -p 3000:3000 -dit python:2-slim
* docker exec Abathur pip install Flask
* echo 'from flask import Flask\napp = Flask(__name__)\n\n@app.route("/")\n    def hello_world():\n\treturn "<h1>Hello, World!</h1>"' > ~/hello.py
* docker exec -e FLASK_APP=/root/hello.py Abathur flask run --host=0.0.0.0  --port 3000

-d фоновый режим
-i —interactive Благодаря этому флагу поток STDIN поддерживается в открытом состоянии даже если контейнер к STDIN не подключён.
-t —tty. Благодаря этому флагу выделяется псевдотерминал, который соединяет используемый терминал с потоками STDIN и STDOUT контейнера
-v ~/:/root расшариваем папку домашнюю на хосте и root на виртуалке

Команда 'docker exec' применяется к запущенному контейнеру, запускает новый процесс внутри контейнера 

Можно изменить файл и перезапуститься
docker restart Abathur
docker exec -e FLASK_APP=/root/hello.py Abathur flask run --host=0.0.0.0  --port 3000


Flask — фреймворк для создания веб-приложений на языке программирования Python
python:<version>-slim
This image does not contain the common packages contained in the default tag and only contains the minimal packages needed to run python. Unless you are working in an environment where only the python image will be deployed and you have space constraints, we highly recommend using the default image of this repository.
https://hub.docker.com/_/python?tab=tags&page=1&name=slim

--host=0.0.0.0  --port 3000 потому что по умолчанию другое и не будет работать в нашем случае
https://flask.palletsprojects.com/en/1.1.x/quickstart/

#20

* docker swarm init --advertise-addr 192.168.99.100

https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/

#21

* docker-machine create --driver virtualbox Aiur

#22

* docker-machine ssh Aiur "docker swarm join --token $(docker swarm join-token worker -q) 192.168.99.100:2377"

docker-machine ssh Aiur выполняем команду (с уже результатом запроса токена) на aiur машине по ssh
docker swarm join --token -выполняется на машине которую делаем воркером
docker swarm join-token worker -q - запрашиваем токен для воршена на машине менеджере
-q -выдает исключительно токен.без воды
92.168.99.100:2377 - айпишник менеджера и порт который ему назначался при назначении менеджером

https://docs.docker.com/v17.09/machine/reference/ssh/
https://docs.docker.com/engine/swarm/join-nodes/
#23

* docker network create -d overlay overmind

Overlay-сети используются в контексте кластеров (Docker Swarm), где виртуальная сеть, которую используют контейнеры, связывает несколько физических хостов, на которых запущен Docker. Когда вы запускаете контейнер на swarm-кластере (как часть сервиса), множество сетей присоединяется по умолчанию, и каждая из них соответствует разным требованиям связи.
(Оверлей (Наложенная сеть), — это сетевой драйвер для соединения несколько демонов Docker между собой и которые позволяют  docker-swarm службам взаимодействовать друг с другом. Вы также можете использовать оверлейные сети для облегчения связи между docker-swarm и автономным контейнером или между двумя отдельными контейнерами на разных Docker демонах. Эта стратегия устраняет необходимость выполнения маршрутизации на уровне ОС между этими контейнерами.)
https://docs.docker.com/network/overlay/

#24 

* docker service create --name orbital-command --network overmind -d -e RABBITMQ_DEFAULT_USER=dtaisha -e RABBITMQ_DEFAULT_PASS=123 rabbitmq

https://docs.docker.com/engine/reference/commandline/service/

docker service create - создаем сервис
--name orbital-command - задаем имя
--network overmind указываем уже существующую сеть
-d -фоновый режим
-e RABBITMQ_DEFAULT_USER=dtaisha - создаем пользователя
-e RABBITMQ_DEFAULT_PASS=123 - задаем пароль

Сервисы — это ещё один уровень абстракции над контейнерами. Как и у контейнера, у сервиса будет имя, базовый образ, опубликованные порты и тома (volumes). В отличие от контейнера, сервису можно задать требования к хосту (constraints), на которых его можно запускать. Да и вообще, сервис можно масштабировать прямо в момент создания, указав, сколько именно контейнеров для него нужно запустить.
Но важно понимать одну большую разницу. Сама по себе команда docker service create не создаёт никаких контейнеров. Она описывает желаемое состояние сервиса, а дальше уже Swarm менеджер будет искать способы это состояние достигнуть. Найдёт подходящие хосты, запустит контейнеры, будет следить, чтобы с ними (и хостами под ними) всё было хорошо, и перезапустит контейнер, если «хорошо» закончится. Иногда желаемое состояние сервиса так и не будет достигнуто. Например, если в кластере закончились доступные хосты. В таком случае сервис будет висеть в режиме ожидания до тех пор, пока что-нибудь не изменится.

RabbitMQ — программный брокер сообщений на основе стандарта AMQP — тиражируемое связующее программное обеспечение, ориентированное на обработку сообщений.

#25

* docker service ls

Лист всех сервисов

#26

* docker service create --name engineering-bay --network overmind --replicas=2 -d -e OC_USERNAME=dtaisha -e OC_PASSWD=123 42school/engineering-bay

docker service create - создаем сервис
--name engineering-bay - название сервиса
--network overmind указываем уже существующую сет
--replicas=2 - кол-во заданий
-d -фоновый режим
-e OC_USERNAME=dtaisha -e OC_PASSWD=123 -задаем имя пользователя и пароль согласно описанию на докерхабе
https://hub.docker.com/r/42school/engineering-bay/

#27

* docker service logs -f $(docker service ps engineering-bay --filter NAME=engineering-bay.1 -q)

docker service logs -f (--follow , -f		Follow log output)
docker service ps engineering-bay -List the tasks of one or more services
--filter NAME=engineering-bay.1 -фильтруем по имени
-q - выводим только id

#28

* docker service create --name marines --network overmind --replicas=2 -d -e OC_USERNAME=dtaisha -e OC_PASSWD=123 42school/marine-squad

--name marines -задаем имя сервису
--network overmind указываем уже существующую сеть
--replicas=2 - кол-во заданий
-d -фоновый режим
-e OC_USERNAME=dtaisha -e OC_PASSWD=123 -задаем имя пользователя и пароль согласно описанию на докерхабе
https://hub.docker.com/r/42school/marine-squad

#29
* docker service ps marines

docker service logs -f marines

#30

* docker service scale marines=20 -d

scale  - масштабировать один или несколько реплицируемых сервисов.
-d -фоновый режим
docker service logs -f marines

#31

* docker service rm $(docker service ls -q)

$(docker service ls -q) берем ID имеющихся сервисов
docker service rm  и удаляем их нахрен

#32

* docker container rm --force $(docker container ls --all -q)

docker container rm --force - удаление даже запущенных контейнеров

docker container ls --all - список всех контейнеров

-q -только ID

#33

* docker image rm $(docker images --all -q)

docker image rm - удаляем имеджи на текущей машине
docker images --all -q -выводим лист ID

#34

* docker-machine rm -y Aiur
https://docs.docker.com/v17.12/machine/reference/rm/#examples

###############01_dockerfiles#############



Если вы хотите, чтобы ваши приложения запускались быстрее, а docker-образ был меньше, тогда вам стоит попробовать Alpine в качестве базового образа
Главное преимущество — сжатие размеров

Если вы используете Docker, то вы должны стремиться к этому сжатию. Благодаря этому ваш docker-образ будет меньше.
Apline 3.6 весит всего 3,98 мб.
https://docs.docker.com/engine/reference/commandline/build/
https://docs.docker.com/engine/reference/commandline/run/

ex00
Dockerfile
————————————————————————————
  1 FROM alpine
  2 LABEL maintainer="dtaisha@student.21-school.ru"
  3 LABEL description="Container with Vim editor"
  4 RUN apk update && apk upgrade && apk add vim
  5
  6 ENTRYPOINT vim
——————————————————————————————
command1 && command2, то command2 выполняется в том, и только в том случае, если статус выхода из команды command1 равен нулю, что говорит об успешном ее завершении.

FROM — задаёт базовый (родительский) образ
LABEL — описывает метаданные. Например — сведения о том, кто создал и поддерживает образ
RUN — выполняет команду и создаёт слой образа. Используется для установки в контейнер пакетов.
ENTRYPOINT — предоставляет команду с аргументами для вызова во время выполнения контейнера.

* docker build -t ex00 .
* docker run --rm -it ex00

--rm		Automatically remove the container when it exits
--tty , -t		Allocate a pseudo-TTY
--interactive , -i		Keep STDIN open even if not attached

By default the docker build command will look for a Dockerfile at the root of the build context

ex01
Dockerfile
—————————————————————————————————————
FROM debian
LABEL maintainer="dtaisha@student.21-school.ru"
LABEL description="Container with TeamSpeak"
RUN apt-get update && apt-get upgrade -y && apt-get install wget -y && apt-get install bzip2 -y
RUN wget https://files.teamspeak-services.com/releases/server/3.10.2/teamspeak3-server_linux_amd64-3.10.2.tar.bz2
RUN tar xvf teamspeak3-server_linux_amd64-3.10.2.tar.bz2
RUN touch teamspeak3-server_linux_amd64/.ts3server_license_accepted
CMD teamspeak3-server_linux_amd64/ts3server_minimal_runscript.sh
—————————————————————————————————————
https://firstvds.ru/technology/faq/teamspeak

wget что бы скачать тимспик по ссылке
bzip2 что бы распаковать сказанный архив
tar xvf - извлечение в текущий каталог
CMD ts3server_minimal_runscript.sh  -запуск сервера
CMD — описывает команду с аргументами, которую нужно выполнить когда контейнер будет запущен. Аргументы могут быть переопределены при запуске контейнера. В файле может присутствовать лишь одна инструкция CMD.
Во время запуска данной команды будут созданы все необходимые файлы на сервере, а также вам выдаст пароль от serveradmin и ключ привилегий от группы server admin в самом клиенте teamspeak
loginname= "serveradmin", password= "3nfyZdLn"
token=8cywT+qi+fyPOS4bVw3EPiU6ta4eTyQLxaFOPfOy

touch teamspeak3-server_linux_amd64/.ts3server_license_accepted - тобы принять лицензионное соглашение TeamSpeak, создадаем файл

* docker build -t ex01 .

--tag , -t		Name and optionally a tag in the ‘name:tag’ format


* docker run -d --name teamspeak --rm -p 9987:9987/udp -p 10011:10011 -p 30033:30033 ex01
--detach , -d		Run container in background and print container ID
--rm		Automatically remove the container when it exits
—publish, -p  - редирект портов

https://teamspeak-club.ru/instructions/kakie-porty-ispolzuet-teamspeak-3-server.html
https://hub.docker.com/_/teamspeak/

Голосовой порт (UDP): 9987
Порт для передачи файлов (TCP): 30033
Порт для передачи запросов на сервер (TCP): 10011

* docker logs teamspeak - посмотреть логи

ex02

Dockerfile1
—————————————————————————————————————————————————————
FROM ruby:2.4.1
LABEL maintainer="dtaisha@student.21-school.ru"
LABEL description="Container Rails"
RUN apt-get update -y && apt-get upgrade -y && apt-get install nodejs -y
ONBUILD COPY My_app /opt/app
ONBUILD WORKDIR /opt/app
ONBUILD EXPOSE 3000
ONBUILD RUN bundle install
ONBUILD RUN rake db:migrate
ONBUILD RUN rake db:seed
—————————————————————————————————————————————————————

Dockerfile2
————————————————————————————————————————————————————--
FROM ft-rails:on-build 
EXPOSE 3000 
CMD ["rails", "s", "-b", "0.0.0.0", "-p", "3000"]
—————————————————————————————————————————————————————
Приложение на Ruby для запуска:
git clone https://github.com/RailsApps/learn-rails 
положить в папку My_app вместе с Dockerfile1

* docker build -t ft-rails:on-build .

Запускается из директории с Dockerfile1
--tag , -t		Name and optionally a tag in the ‘name:tag’ format (ft-rails:on-build потому что Dockerfile2 будет использовать такой образ)

* docker build -t dock2 .

Запускается из директории с Dockerfile2

* docker run -d -p 3000:3000  dock2
--detach , -d		Run container in background and print container ID
—publish, -p  - редирект портов

Теперь доступно на localhost:3000
nodejs - программная платформа, основанная на движке V8, превращающая JavaScript из узкоспециализированного языка в язык общего назначения. В данном случае нужна для работы приложения.

ONBUILD - все это будет запускаться когда будет блилдится Dockerfile2
RUN позволяет создать слой во время сборки образа. После её выполнения в образ добавляется новый слой, его состояние фиксируется. Инструкция RUN часто используется для установки в образы дополнительных пакетов. 
COPY — копирует в контейнер файлы и папки.
WORKDIR — задаёт рабочую директорию для следующей инструкции.Инструкция WORKDIR позволяет изменить рабочую директорию контейнера. С этой директорией работают инструкции COPY, ADD, RUN, CMD и ENTRYPOINT, идущие за WORKDIR. 
EXPOSE — указывает на необходимость открыть порт.
CMD — описывает команду с аргументами, которую нужно выполнить когда контейнер будет запущен. Аргументы могут быть переопределены при запуске контейнера. В файле может присутствовать лишь одна инструкция CMD.

Exec-форма использует синтаксис, напоминающий описание JSON-массива. Например, это может выглядеть так: RUN ["my_executable", "my_first_param1", "my_second_param2"].
shell-форма инструкции RUN в таком виде: RUN apk update && apk upgrade && apk add bash.

Bundle — менеджер для управления gem'ами. Подтянет необходимые зависимости

rake db:migrate - запускает (одиночные) миграции, которые еще не выполнялись.
Вместо того, чтобы записывать модификации схемы на чистом SQL, миграции позволяют использовать простой Ruby DSL для описания изменений в ваших таблицах.

rake db:seed- Заполнение базы данных(Populating the Database)

Rake — инструмент для автоматизации сборки программного кода, написаный на Ruby, подобен таким инструментам как make, Ant или Phing.

ex03
https://www.nixp.ru/recipes/46.html
https://www.shellhacks.com/ru/create-csr-openssl-without-prompt-non-interactive/

Dockerfile
—————————————————————————————————————————————————————

  1 FROM ubuntu:18.04
  2 RUN apt-get update -y && apt-get upgrade -y && \
  3     echo "postfix postfix/mailname string example.com" | debconf-set-selections && \
  4     echo "postfix postfix/main_mailer_type string 'Internet Site'" | debconf-set-selections && \
			-для тихой установки постфикс
  5     apt-get install -y ca-certificates curl openssh-server postfix && \
			-Установка дополнительных компонентов
  6     curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash && \
			-скрипт для подготовки рабочей среды под GitLab
  7     apt-get install gitlab-ce
			-установка самого GitLab
  8 ENV DEBIAN_FRONTEND=noninteractive
			-установка пакетов без запроса подтверждения
  9 RUN apt-get install -y tzdata
			-установка даты
 10 RUN mkdir -p /etc/gitlab/ssl/ && chmod 700 /etc/gitlab/ssl/ && \
			-создаем каталог для ключей.(-р даже если существует) и даем права
 11     cd /etc/gitlab/ssl/ && \
			-переходим в папку
 12     openssl genrsa -out dtaishagitlab.com.key 2048 && \
 13     openssl req -new -x509 -nodes -days 60 -key dtaishagitlab.com.key -out dtaishagitlab.com.crt \
 14     -subj "/C=RU/ST=MOSCOW/L=MOSCOW/O=21-school/OU=21-school/CN=dtaishagitlab.com" && \
			-генерируем ключи и заполняем ssl сертификат
 15     echo "letsencrypt['enable'] = false" >> /etc/gitlab/gitlab.rb && \
			-редактируем конфигурационный файл, выключаем встроенный nginx
 16     sed -i "s/^external_url.*/external_url \'https:\/\/dtaishagitlab.com\'/" /etc/gitlab/gitlab.rb && \
			-s/что_заменять/на_что_заменять/опции
			-^инициирует начало строки(что бы выполнить замену только в строке которая начинается с external_url.*) 
 17     echo "gitlab_rails['gitlab_shell_ssh_port'] = 22" >> /etc/gitlab/gitlab.rb
			-задаем порт в который пушить по ssh
 18 RUN (/opt/gitlab/embedded/bin/runsvdir-start &) && \
 19     gitlab-ctl reconfigure
 20
 21 CMD (/opt/gitlab/embedded/bin/runsvdir-start &) && \
 22     gitlab-ctl start && service ssh start && gitlab-ctl tail

———————————————————————————————————————————————————————


https://coderwall.com/p/lryimq/postfix-silent-install-on-ubuntu - тихая установка постфикса
https://netpoint-dc.com/blog/gitlab-setup-centos-debian-ubuntu/ -инструксион по установке gitlab
https://www.shellhacks.com/ru/create-csr-openssl-without-prompt-non-interactive/ - ssl сертификат
https://docs.gitlab.com/omnibus/common_installation_problems/#init-daemon-detection-in-non-docker-container -для gitlab-ctl

Запуск
* 	docker-machine create --driver virtualbox --virtualbox-memory 3072 --virtualbox-cpu-count 3 gitlab
* 	docker build . -t mygitlab
* 	docker run -d -p 50022:22 -p 443:443 -p 80:80 --privileged mygitlab
	
-d фоновый режим 
--privileged flag gives all the capabilities to the container and also access to the host’s devices 
Гитлаб доступен по https://localhost
Через сафари

Create a new repository and try to git clone it with https "https://localhost/root/EXAMPLE.git"
	#--SSH PART--#

cat /Users/dtaisha/.ssh/id_rsa.pub скопировать ключик и забросить в гитлаб

ssh -p 50022 -T git@[MAC ip] что бы протестить

	#git clone ssh://git@localhost:50022/root/testssh.git

——————————————————————
Бонусы
docker restart robotfindskittten
docker exec -ti robotfindskittten /bin/sh
