+++
date = "2019-05-08T18:09:14-05:00"
draft = false
title = "Перенос проектов между ЦОДами. Часть 4"
slug = "move-projects-beetween-cod-part-4"
tags = [ "ЦОД","Миграции","Проект"]
type = ["posts","post"]
categories = [
    "Development"
]
+++
![](/images/nnru.svg)

### Переключение трафика

##### 4:20 - переключение трафика в новый цод

##### 4:20 - 7:30 - фикс основных багов, которые пропустили

- не работали некоторые ссылки на галереи
- некоторые дополнительные страницы не открывались по https из-за отсутствия сертификата
- часть легаси картинок пропала
- тюнининг всех сервисов по их потреблению ресурсов
- проверка отправки почты, работы медиа, платного функционала, авторизация,  проверка основного функционала.

##### 7:30 - обнаружили, что часть данных не синхронизовалась с текущей базой.
Лаг данных составил 11:00-4:00.

Данные с проблемами:

- темы на форумах - пропала часть сообщений
- галереи - пропала часть комментариев и фотографий
- дополнительные сервисы, которые не аффектили работу НН

##### 7:37 - принял решение о том, что будем восстановливать старые данные

- откат спустя 3 часа не  поможет, придется все равно восстанавливать текущие добавленные данные
- добавит еще больше энтропии в данными
- текущий нн работает очень не стабильно
- увеличили все служебные идентификаторы на 1000000, для того, чтобы не было пересечений со старыми

##### начали разбираться с восстановлением данных по приоритетам
- темы СП + обычные
- сообщения и комментарии СП + обычные
- галереи СП + обычные

##### 7:40 - начали восстанавливать данные

##### 13:53 - восстановили большую часть тем + СП, которые не пересекались

##### 14:34 - восстановили все сообщения в приватах

##### 15:40 - начали искать количество пересекающихся тем

##### 16:30 - нашли 4 пересекающиеся темы по ID с СП

### Следующий день

##### 4:30 - 11:54 
перенесли руками темы СП, у которых произошло задвоение idшников. Таких тем оказалось 4 штуки. Передали данные сп. Попутно шли небольшие правки

##### 11:54 - 14:19 - восстановили данные в галереях + комментарии к эти фотографиям.

##### 14:19 - 16:00 - еще некоторые правки, которые фиксят мелкие проблемы - вам отвечают, редиректы и тп

Все эти 2 дня шла оперативная работа с ТП форумов и на самих форумах. На данном этапе можно сказать, что работа по перевозке завершена и весь функционал работает.

### Что мы делали:

- составили подробный план запуска, но забыли проверить консистентность данных в БД. Отсюда основные проблемы.
- шли согласно плану и чинили мелкие баги
- в целом подготовились к переключению - подготовили всех участников заинтересованных сторон
- восстановили данные силами НСК и ЧЛБ (3 человека)
- крупных багов, кроме как факапа с бд, не было.

### Зачем все это затевалось:

- Перевели полностью сервисы НН на нашу инфраструктуру:
- Убираем ЦОД НН. Он не поддерживаемый. Перевозим железо из НН в ЕКБ
- Поддержка НН станет проще из-за наличия дев инфраструктуры
- Понятное количество ресурсов, который потребляет НН
- наличие замотивированной команды в ЧЛБ, которая затащит НН
- Нет инженеров ТО в НН
- Разобрались с тем, как устроен нн, поэтому понятный мониторинг ресурсов





