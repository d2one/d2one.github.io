+++
title = "Как выходила черепаха"
slug = "turtoise-go-away"
date =  "2020-09-12T20:01:03+07:00"
draft = false
tags = [ "cto","ngs","e1"]
type = ["posts","post"]
categories = [
    "cto"
]
+++
Любой стартап начинается на коленке. 
Делается все быстро, главное - “чтоб работало”. Лишь малое количество людей задумываются, а как система будет работать в будущем, и как ее масштабировать. 
Если же вашему продукту уже больше 20 лет, то, скорее всего, он содержит артефакты, которые написаны очень давно и не соответствуют современным требованиям разработки. Однако этот код работает и приносит деньги. И тут возникает проблема - порой, довольно сложно объяснить бизнесу необходимость проведения тех или иных изменений в коде.

В своем время, когда мы начинали переписывать нгс.новости на новую и масштабируемую платформу, перед нами стояло несколько вопросов, на которые у нас не было готового ответа:

- Будут ли это микросервисы?/Будут ли это сервисы? /Или оставить монолит? 
- Как мы будем масштабироваться, при увеличении нагрузки? 
- Как все это накладывается на изменение языка? И нужно ли писать все на go?

Есть вероятность, что вы можете ответить - “да, нужны микросервисы”, сразу все делать в k8s, а писать сразу на go. На момент принятия решения у нас был php5.6 и ubuntu10 в продакшне. Самое интересное, что 3 года назад еще и фронтенд преобразился. Ребята уже не хотели верстать в шаблонах на smarty, а для взаимодействия с пользователем использовать jquery. Ведь уже есть webpack, vue.js, react, angular, на худой конец. 

Проект должен приносить прибыль, ведь мы коммерческая компания, которая занимается медиа-бизнесом. Медиа-бизнес, а, особенно, новости - это низкомаржинальный бизнес. В компании работает больше пятиста человек. Пока мы делаем новую платформу - новости должны выходить, а продукт развиваться. Останавливаться в развитии нельзя.

Если мы будем делать сервисы - то сколько их должно быть? Насколько “микро” они должны быть? А ведь еще есть dev, где мы должны обеспечить разработчикам изолированное окружение для работы. И для тестирования. И доставлять в продакшн. Мы релизимся больше 10 раз в день и не ограничены в количестве.

В идеальном мире - пишем сервисы на go. Сколько толковых разработчиков на go вы знаете в Новосибирске? Которые могут в продакшн пилить сервисы. Деплоить их. Кажется, что таких людей не очень много. И самое главное - стоимость поиска таких разработчиков на go сильно выше, чем на php. Вы можете возразить, что можно переучить команду. Да, можно. И мы попробовали это сделать на небольшой фокус-группе. Да, сделали сервис на go. Правда, до production он не дошел, потому что под нагрузкой с ним начинались проблемы. Если вспомнить, то 3 года назад, толком стандартов-то не было для go. Сплошная неопределенность.

А давайте сразу в k8s? Если у вас нет экспертизы в том, как работает k8s, нагрузку продакшн вы туда не можете пускать. При простоях - вы потеряете деньги. Напомню, что новости - достаточно большой проект. На нашей платформе сейчас работает > 43 сайтов.

Еще из особенностей было то, что у нас много внешних клиентов - мобильные приложения (2 старых и 2 новых), паблики на сторонних платформах (74.ru, e1 и тд), выгрузки в яндекс, гугл, и т.д.

Для себя мы выбрали следующий стек. Основным языком для команд разработки остается php. Очевидно, что надо переходить на последнюю версию - на тот момент это 7.2.9. 
И фреймворк… О, сколько копий было сломано… Дорогой читатель, если ты дошел до сюда, то скорее всего у тебя уже припасен готовый ответ. Symfony. Symfony - silver bullet для разработки любых проектов. Но так ли это? У нас было решение, построенное на своем легковесном фреймфорке. Но там не было стандартов, и архитектурно оно устарело, однако, были некоторые наработки, которые хотелось бы сохранить.

За что люди любят symfony? Кажется, тут есть несколько аргументов:

-  понятная документация, 
- много пакетов для использования,
- Doctrine в качестве ORM,
- PSR, где он возможен. 

Больше всего вопросов вызывала doctrine при работе с базой данных. Можно возразить, что, мол, ну не используйте ее. Делайте запросы напрямую SQL. Но тогда, в чем еще профит от Symfony?

Мы решили, что будем использовать Slim3. Он очень простой. В нем есть роутинг, DI, и стандарты PSR. Все. Это полволило переиспользовать некоторые штуки из старого фреймфорка, которые мы хотели оставить. 

Кажется, что для каждой задачи должен быть свой инструмент. Если для вас наличие Symfony в проекте - это верх, и другое для вас не приемлимо, то у меня для вас плохие новости. Во всех больших проектах есть легаси. И не symfony едины разработчики на php.

Нам нужно много искать, поэтому мы продолжили использовать Sphinx. Мы используем его очень давно, показал он себя хорошо. Правда, с масштабированием все сложно. В дальнейшем мы перешли на Manticore Search. Ubuntu 10 сменилась на Arch Linux, в нем имплиментирована нативная поддержка ZFS. Да, ZFS у нас основная файловая система.

Мы решили делать API, причем мы хотели максимально унести логику с фронта (у нас на это были причины, поверьте). Чтобы скрыть логику, мы использовали паттерн Gateway API. Правда, нативной production ready реализации не нашлось (Kraken и Co появились несколько позже),поэтому пришлось несколько извратиться и накрутить реализацию с помощью Nginx (великий и могучий). Остается frontend. На фронт пошел Vue.js + SSR. И SPA. Да, у нас есть серверный рендеринг.

Я думаю, что сейчас мы идем в правильном направлении. Пока что у нас довольно много легаси на проектах, но мы стараемся его выпиливать, и фокус разработки сейчас смещен в эту сторону. Да, есть некоторые проблемы с быстродействием, но это тоже выправляется. Самое главное - что платформа, которая получилась, позволяет менять все достаточно быстро, не ломаясь при этом.

А с какими трудностями в выборе технологий и инструментов сталкивались вы? Насколько глубоко вы проникали в потребности, чтобы вносить изменения? Чтобы вы изменили 3 года назад?