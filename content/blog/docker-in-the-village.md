+++
date = "2017-09-12T04:03:46-06:00"
draft = false
title = "Свой блог на Docker, Hugo, traefic, nginx"
slug = "docker-in-the-village"
tags = [ "docker", "hugo", "traefic"]

+++
# Свой блог на Docker, Hugo, traefic, nginx

Сегодня в мире возникает большое количество различных технологических решений. Docker показал, что он в нашей жизни всерьез и надолго. В сегодняшних реалиях становится понятно, что его должен уметь использовать каждый разработчик.

В этой статье я расскажу о своем опыте использовании Docker’a на небольшом проекте. Я решил сделать свой блог, и максимально использовать решения, предлагаемые Docker’ом. 

Требования, которые я для себя выделил, при разработке решения, для ведения блога:

- легкий перенос данных между хостингами
- бесплатный ssl для блога
- Docker 17.06+
- Docker swarm
- Docker stack
- docker-compose.yml v3

Необходимо было выбрать самый простой способ для получения html-файлов со статьями. 

**Hugo**
Написанный на go статический генератор сайта, имеет встроенный сервер для просмотра вида статьи локально. Hugo генерирует html из файлов в markdown разметки. Имеет  100+ тем на любой вкус. Большинство тем имеют простую интеграцию с разными сервисами, например аналитика от google. 

Дальнейший workflow в написании статей определился. Я пишу статьи в .md файлах, hugo генерирует html. Такая схема позволяет запускать блог на любом хостинге.

Хранить данные своих материалов я собирался на **github.com** в markdown разметке. Чтобы hugo узнал о том, что был произошли изменения в репозитории со статьями, необходимо выполнить webhook. Существует простой webhook handler, который позволяет выполнить любую, заранее предопределенную  команду в конфиге, на сервере. В моем случае, необходимо обновить получить последнюю версию репозитория со статьями, и перегенерировать сайт.

**Traefic**

Прокси-сервер, написанный на go. Позволяет организовать проксирование запросов к запущенным сервисам. Имеется возможность получения и продление ssl-сертификатов.
Для работы блога у меня получилась следующая схема из сервисов:

![Схема работы блога](https://lh3.googleusercontent.com/w4n0OSxpa0DWcQvCcvjIVkXyPhVQNbbn01E8R_ANPf8_wHeOVnHd46l4oBgCh04cKiSeZZ6j_sYcUO8ERLhzW55j656pTuseKhzg-w4xUfsF6LgL01nQQQTUlg1dbldAmLUEJtCYrog)

**Итого**:

- swarm host + overlay network
- trarfic,  у которого открыты 80 и 443 порты для обработки внешних соеднинений
- сервис с nginx, для раздачи html файлов
- сервис с hugo + webhook handler, который ловит хуки и генерирует html разметку

Я решил развернуть блог на DigitalOcean. Минимальный droplet стоит 5$ в месяц и имеет следующие характеристики: 

- 1 cpu
- 512mb оперативки
- 20GB ssd дисков

Есть возможность выбрать предустановленный в droplet софт. В том числе и последнюю версию Docker (17.06). Как раз с этой версии Docker появился multi-stage сборка образов, что пригодится для сборки hugo + webhook. 
Описываем сборку кастомного образа с hugo и webhook-handler:

    FROM golang:1.8-alpine AS builder
    RUN apk add --no-cache git
    RUN go get -v github.com/adnanh/webhook
    RUN go get -v github.com/spf13/hugo
    FROM alpine:3.5
    RUN apk add --no-cache bash git && rm -rf /var/cache/apk/*
    COPY --from=builder /go/bin/ /usr/local/bin/
    COPY hooks.json /etc/webhook/hooks.json
    COPY run.sh /run.sh
    EXPOSE 9000
    ENTRYPOINT ["/bin/bash", "/run.sh"]
    CMD ["run"]

Образы traefic и nginx я взял оригинальные. Вся задача свелась к  монтированию volumes со сгенерированными html файлами из контейнеров с hugo к контейнерам с nginx.
**Hugo:**

    image: hugo
    volumes:
      - $PWD/volumes/nginx/html/:/usr/share/nginx/html/

**Nginx:**

    image: nginx
    volumes:
      - $PWD/volumes/nginx/html/:/usr/share/nginx/html/

Так же необходимо было сделать описать обработку webhook’s  в образе с hugo.  

    [
      {
        "id": "redeploy",
        "execute-command": "/run.sh",
        "pass-arguments-to-command": [
            { "source": "string", "name": "webhook" }
        ]
      }
    ]

Чтобы веб-хук сработал, необходимо спроксировать запрос через traefic. У контейнера с hugo открыт 9000 порт, который слушает webhook handler. Для проксирование запросов с webhook в контейнер с hugo, в docker-compose.yml необходимо описать правила проксирования.

Описываем сервисы в docker-compose.yml

```nginx
image: hugo
deploy:
  mode: replicated
  replicas: 1
  labels:
    - "traefik.port=9000"
    - "traefik.docker.network=traefik-net"
    - "traefik.backend=hugo"
    - "traefik.frontend.rule=PathPrefix:/hooks/"
    - "traefik.backend.loadbalancer.method=drr"
    - "traefik.backend.loadbalancer.swarm=true"
```

Это значит, что при обращении по url somedomain.com/hooks/ этот запрос будет спроксирован в контейнер с hugo на 9000 порт. В качестве load-balancer выступает Docker swarm.
 
 В веб-панели traefic’а можно посмотреть, что получилось:

![](https://d2mxuefqeaa7sj.cloudfront.net/s_3030C2C7484341CCE62F6E9735610BB0C233511BADFB1760C4E859BFBD71ED13_1503826014825_image.png)


Получился блог, который:

- самостоятельно обновляет и получает ssl сертификаты
- материалы легко перенести на другую платформу
- атоматическое обновление материалов на сайте при пуше в репозиторий со статьями

Весь исходный код можно посмотреть по [ссылке](https://github.com/d2one/docker-blog/).

