# Partners API
API принимает GET/POST запросы на функции указанные в этой документации. Методы не различаются: как GET, так и POST вернут идентичный ответ.

Все запросы отправляются на сервер: https://qtickets.ru
### Авторизация
Каждый запрос к API должен содержать заголовок 
`Authorization: Bearer YOUR_API_KEY`. API ключ выдается техподдержкой.

В случае, если ключ неверный или отсутствует заголовок, сервер вернет JSON с кодом ошибки:
```
{
   "status": "error",
   "code": "WRONG_AUTHORIZATION",
   "message": "Wrong authorization"
}
``` 
### Листинг мероприятий
Метод `api/partners/v1/events/lists` возвращает список предстоящих мероприятий, в которых включена возможность продажи билетов через партнеров. Пример ответа:
```json
{
    "status": "success",
    "result":[
        {
            "event_id": 12,
            "show_id": 20,
            "event_name": "Название мероприятия 1",
            "start_date": "2018-05-04T06:22:00+10:00",
            "scheme":{
                "id": 48,
                "name": "Кремлевский дворец",
                "place_name": "Кремлевский дворец",
                "place_address": "ул. Воздвиженка, 1",
                "place_description": "м. Библиотека им. Ленина, Александровский сад, Боровицкая",
                "zones":[
                    {"zone_id": 8, "name": "VIP left", "seats":[{"seat_id": "8-1;42"/*....*/}]},
                    {"zone_id": 7, "name": "Supervip right", "seats":[{"seat_id": "7-1;8"/*....*/}]},
                    {"zone_id": 6, "name": "VIP right 2", "seats":[{"seat_id": "6-1;52"/*....*/}]},
                    {"zone_id": 5, "name": "Supervip left", "seats":[{"seat_id": "5-1;8"/*....*/}]},
                    {"zone_id": 4, "name": "VIP left 1", "seats":[{"seat_id": "4-1;49"/*....*/}]},
                    {"zone_id": 3, "name": "Танцевальный партер", "seats":[{"seat_id": "3-1;1"/*....*/}]},
                    {"zone_id": 2, "name": "VIP right 1", "seats":[{"seat_id": "2-2;49"/*....*/}]},
                    {"zone_id": 1, "name": "VIP 3", "seats":[{"seat_id": "1-1;49"/*....*/}]}
                ]
            },
            "city":{
                "id": 2,
                "name": "Москва",
                "timezone": "Europe/Moscow",
                "coords":[
                    55.7558,
                    37.6176
                ]
            },
            "last_update_hash": "63e9effbdeca7a633d73d118a65dc78478110a39"
        },
        {
            "event_id": 12,
            "show_id": 21,
            "event_name": "Название мероприятия 1",
            "start_date": "2019-01-03T17:00:00+10:00",
            "scheme":{
                "id": 48,
                "name": "Кремлевский дворец",
                "place_name": "Кремлевский дворец",
                "place_address": "ул. Воздвиженка, 1",
                "place_description": "м. Библиотека им. Ленина, Александровский сад, Боровицкая"
            },
            "city":{
                "id": 2,
                "name": "Москва",
                "timezone": "Europe/Moscow",
                "coords":[
                    55.7558,
                    37.6176
                ]
            },
            "last_update_hash": "becffa60e0c682208f537fe71818aab62ba2b5a4"
        },
        {
            "event_id": 33,
            "show_id": 41,
            "event_name": "Название мероприятия 2",
            "start_date": "2018-12-31T18:00:00+07:00",
            "scheme":{
                "id": 48,
                "name": "Кремлевский дворец",
                "place_name": "Кремлевский дворец",
                "place_address": "ул. Воздвиженка, 1",
                "place_description": "м. Библиотека им. Ленина, Александровский сад, Боровицкая"
            },
            "city":{
                "id": 2,
                "name": "Москва",
                "timezone": "Europe/Moscow",
                "coords":[
                    55.7558,
                    37.6176
                ]
            },            
            "last_update_hash": "9e02e444620a3621f08c85732f1634f48fe35571"
        }
    ]
}
``` 
Где:
 
* `event_id` - Идентификатор мероприятия
* `show_id`- Идентификатор даты мероприятия
* `event_name` - Название мероприятия
* `scheme` - Площадка. В `scheme.zones` содержится полный список доступных категорий, а также мест в них.
* `city` - Город
* `start_date` - Дата начала мероприятия
* `last_update_hash` - Содержит хэш, который меняется при некоторых измененииях в мероприятии. Например, поменялись квоты или места для продажи. Этот параметр будет возвращаться во всех последующих методах, и если он изменится, то это сигнал для пересинхронизации. 

Все последующие запросы к API должны содержать `event_id`. В случае, если мероприятие содержит более 1-й даты, то также нужно указывать и `show_id`. 

### Статус мест в мероприятии
Метод `api/partners/v1/events/seats/{event_id}/{show_id?}`

Пример ответа:
```json
{  
   "event_id":12,
   "show_id":21,
   "event_name":"Название мероприятия 1",
   "start_date":"2019-01-03T17:00:00+10:00",
   "scheme":{
        "id": 48,
        "name": "Кремлевский дворец",
        "place_name": "Кремлевский дворец",
        "place_address": "ул. Воздвиженка, 1",
        "place_description": "м. Библиотека им. Ленина, Александровский сад, Боровицкая",
        "zones":[
            {"zone_id": 8, "name": "VIP left", "seats":[{"seat_id": "8-1;42"/*....*/}]},
            {"zone_id": 7, "name": "Supervip right", "seats":[{"seat_id": "7-1;8"/*....*/}]},
            {"zone_id": 6, "name": "VIP right 2", "seats":[{"seat_id": "6-1;52"/*....*/}]},
            {"zone_id": 5, "name": "Supervip left", "seats":[{"seat_id": "5-1;8"/*....*/}]},
            {"zone_id": 4, "name": "VIP left 1", "seats":[{"seat_id": "4-1;49"/*....*/}]},
            {"zone_id": 3, "name": "Танцевальный партер", "seats":[{"seat_id": "3-1;1"/*....*/}]},
            {"zone_id": 2, "name": "VIP right 1", "seats":[{"seat_id": "2-2;49"/*....*/}]},
            {"zone_id": 1, "name": "VIP 3", "seats":[{"seat_id": "1-1;49"/*....*/}]}
        ]
    },
   "city":{
        "id": 2,
        "name": "Москва",
        "timezone": "Europe/Moscow",
        "coords":[
            55.7558,
            37.6176
        ]
   },   
   "last_update_hash":"2f030832e2eb309b811dc9675df611273aefe618",
   "zones": [
     {
        "zone_id": 7,
        "name": "Supervip right",
        "seats": [
           {
              "seat_id": "7-1;8",
              "available": true,
              "coords": [
                 283,
                 977
              ],
              "row": 1,
              "place": 8,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "7-1;7",
              "available": true,
              "coords": [
                 299,
                 963
              ],
              "row": 1,
              "place": 7,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "7-1;6",
              "available": true,
              "coords": [
                 316,
                 949
              ],
              "row": 1,
              "place": 6,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "7-1;5",
              "available": true,
              "coords": [
                 333,
                 935
              ],
              "row": 1,
              "place": 5,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "7-1;4",
              "available": true,
              "coords": [
                 349,
                 921
              ],
              "row": 1,
              "place": 4,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "7-1;3",
              "available": true,
              "coords": [
                 366,
                 907
              ],
              "row": 1,
              "place": 3,
              "price": 0,
              "currency_id": "RUB"
           }
        ]
     },
     {
        "zone_id": 6,
        "name": "VIP right 2",
        "seats": [
           {
              "seat_id": "6-1;52",
              "available": true,
              "coords": [
                 58,
                 600
              ],
              "row": 1,
              "place": 52,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "6-1;53",
              "available": true,
              "coords": [
                 74,
                 614
              ],
              "row": 1,
              "place": 53,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "6-1;51",
              "available": true,
              "coords": [
                 58,
                 579
              ],
              "row": 1,
              "place": 51,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "6-1;50",
              "available": true,
              "coords": [
                 74,
                 565
              ],
              "row": 1,
              "place": 50,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "6-1;49",
              "available": true,
              "coords": [
                 91,
                 551
              ],
              "row": 1,
              "place": 49,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "6-1;48",
              "available": true,
              "coords": [
                 108,
                 537
              ],
              "row": 1,
              "place": 48,
              "price": 0,
              "currency_id": "RUB"
           },
           {
              "seat_id": "6-1;47",
              "available": true,
              "coords": [
                 125,
                 523
              ],
              "row": 1,
              "place": 47,
              "price": 0,
              "currency_id": "RUB"
           }
        ]
     },
     {
        "zone_id": 3,
        "name": "Танцевальный партер",
        "seats": [
           {
              "seat_id": "3-1;1",
              "available": true,
              "coords": [
                 730,
                 806
              ],
              "free_quantity": 14,
              "admission": true,
              "price": 300,
              "currency_id": "RUB"
           }
        ]
     },
     {
        "zone_id": 1,
        "name": "VIP 3",
        "seats": [
           {
              "seat_id": "1-1;3",
              "available": true,
              "coords": [
                 1032,
                 137
              ],
              "row": 1,
              "place": 3,
              "price": 1234,
              "currency_id": "RUB"
           },
           {
              "seat_id": "1-1;2",
              "available": true,
              "coords": [
                 1052,
                 137
              ],
              "row": 1,
              "place": 2,
              "price": 100,
              "currency_id": "RUB"
           }
        ]
     }
  ]
}

```
В `zones` содержится список категорий, каждая категория имеет свой уникальный идентификатор `zone_id`, название `name` и места `seats`. 

Место имеет свой уникальный идентификатор `seat_id`, **boolean** флаг доступности для продажи `available`, а также ряд `row` и номер места `place`. Координаты места указаны в свойстве `coords`


Для мест, где указано `admission: true` продаются входные билеты, в параметре `free_quantity` оставшееся кол-во для продажи.

### Бронирование билета для места
Метод `api/partners/v1/tickets/add/{event_id}/{show_id?}`

В GET или POST параметрах нужно передать:
* `seat_id` иденитификатор места
* `paid` признак оплаты билета 0 или 1
* `reserved_to` если билет еще не оплачен, нужно передавать дату, до которой билет бронируется, например `2018-05-24T17:00:00+10:00`
* `external_id` необязательный параметр, может содержать произвольное уникальное значение, по которому в дальнейшем будет вестись работа по этому конкретному билету, например `ticket12345`

Пример ответа:
```json
{
     "event_id": 12,
     "show_id": 21,
     "event_name": "Название мероприятия 1",
     "start_date": "2019-01-03T17:00:00+10:00",
     "scheme":{
         "id": 48,
         "name": "Кремлевский дворец",
         "place_name": "Кремлевский дворец",
         "place_address": "ул. Воздвиженка, 1",
         "place_description": "м. Библиотека им. Ленина, Александровский сад, Боровицкая"
     },
     "city":{
         "id": 2,
         "name": "Москва",
         "timezone": "Europe/Moscow",
         "coords":[
             55.7558,
             37.6176
         ]
     },      
     "last_update_hash": "2f030832e2eb309b811dc9675df611273aefe618",
     "status": "success",
     "result": {
         "id": "27061", 
         "paid": 0,
         "reserved_to": "2018-05-24T17:00:00+10:00",
         "external_id": "ticket12345",
         "barcode": null
     }
}
```
где в `result` содержится:
* `id` - идентификатор билета, в дальнейшем потребуется для работы в других методах
* `external_id` - ваш внешний идентификатор, если был передан в запросе
* `barcode` - будет содержать значение для ШК кода, в случае, если билет оплачен `paid: 1`

В случае, если закончились билеты, в ответе вернется ошибка:
```json
{
"status": "error",
"code": "SEAT_NOT_AVAILABLE",
"message": "Seat \"6-1;52\" is not available"
}
```
### Обновление билета
Метод `api/partners/v1/tickets/add/{event_id}/{show_id?}` служит для обновления билета, например продлить бронь `reserved_to` или сделать билет оплаченным `paid: 1`
В запросе нужно передать:
* `id` идентификатор билета или `external_id`
* `paid` 0 или 1
* `reserved_to` в случае, если `paid: 0` 

### Удаление билета
Метод `api/partners/v1/tickets/remove/{event_id}/{show_id?}` служит для отмены брони. Например, пользователь сначала добавил билет в корзину, а потому удалил его сразу или закрыл вкладку в браузере.
В запросе нужно передать:
* `id` идентификатор билета или `external_id`
Ответ:
```json
{
    "event_id": 12,
    "show_id": 21,
    "event_name": "Название мероприятия 1",
    "scheme":{
        "id": 48,
        "name": "Кремлевский дворец",
        "place_name": "Кремлевский дворец",
        "place_address": "ул. Воздвиженка, 1",
        "place_description": "м. Библиотека им. Ленина, Александровский сад, Боровицкая"
    },
    "city":{
        "id": 2,
        "name": "Москва",
        "timezone": "Europe/Moscow",
        "coords":[
            55.7558,
            37.6176
        ]
    },     
    "start_date": "2019-01-03T17:00:00+10:00",
    "last_update_hash": "2f030832e2eb309b811dc9675df611273aefe618",
    "status": "success",
    "result": true
}
```

### Список возможных ошибок в ответе
В случае, если `status: error`
* `UNKNOWN_ERROR` - неизвестная ошибка, вероятно стоит обратиться в техподдержку
* `WRONG_AUTHORIZATION` - авторизация не удалась, вероятно неверный API KEY
* `EVENT_NOT_FOUND` - мероприятие не найдено
* `SEAT_NOT_FOUND` - место не найдено
* `SEAT_NOT_AVAILABLE` - место недоступно для продажи
* `TICKET_NOT_FOUND` - билет не найден
* `VALIDATION_ERROR` - ошибка валидации параметров в запросе
