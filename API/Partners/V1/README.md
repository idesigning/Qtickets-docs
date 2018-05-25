# Partners API
API принимает GET/POST запросы на функции указанные в этой документации. Методы не различаются: как GET, так и POST вернут идентичный ответ.

Все запросы отправляются на сервер: https://qtickets.ru Тестовый сервер: https://new.qtickets.ru
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
            "last_update_hash": "63e9effbdeca7a633d73d118a65dc78478110a39"
        },
        {
            "event_id": 12,
            "show_id": 21,
            "event_name": "Название мероприятия 1",
            "start_date": "2019-01-03T17:00:00+10:00",
            "last_update_hash": "becffa60e0c682208f537fe71818aab62ba2b5a4"
        },
        {
            "event_id": 33,
            "show_id": 41,
            "event_name": "Название мероприятия 2",
            "start_date": "2018-12-31T18:00:00+07:00",
            "last_update_hash": "9e02e444620a3621f08c85732f1634f48fe35571"
        }
    ]
}
``` 
Где:
 
* `event_id` - Идентификатор мероприятия
* `show_id`- Идентификатор даты мероприятия
* `event_name` - Название мероприятия
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
   "last_update_hash":"2f030832e2eb309b811dc9675df611273aefe618",
   "zones":[  
      {  
         "zone_id":7,
         "name":"Supervip right",
         "seats":[  
            {  
               "seat_id":"7-1;8",
               "available":false,
               "row":1,
               "place":8
            },
            {  
               "seat_id":"7-1;7",
               "available":true,
               "row":1,
               "place":7
            },
            {  
               "seat_id":"7-1;6",
               "available":true,
               "row":1,
               "place":6
            },
            {  
               "seat_id":"7-1;5",
               "available":true,
               "row":1,
               "place":5
            },
            {  
               "seat_id":"7-1;4",
               "available":true,
               "row":1,
               "place":4
            },
            {  
               "seat_id":"7-1;3",
               "available":true,
               "row":1,
               "place":3
            },
            {  
               "seat_id":"7-1;2",
               "available":true,
               "row":1,
               "place":2
            },
            {  
               "seat_id":"7-1;1",
               "available":true,
               "row":1,
               "place":1
            }
         ]
      },
      {  
         "zone_id":6,
         "name":"VIP right 2",
         "seats":[  
            {  
               "seat_id":"6-1;52",
               "available":false,
               "row":1,
               "place":52
            }
         ]
      },
      {  
         "zone_id":3,
         "name":"Танцевальный партер",
         "seats":[  
            {  
               "seat_id":"3-1;1",
               "available":true,
               "free_quantity":11,
               "admission":true
            }
         ]
      },
      {  
         "zone_id":1,
         "name":"VIP 3",
         "seats":[  
            {  
               "seat_id":"1-1;3",
               "available":false,
               "row":1,
               "place":3
            },
            {  
               "seat_id":"1-1;2",
               "available":false,
               "row":1,
               "place":2
            }
         ]
      }
   ]
}
```
В `zones` содержится список категорий, каждая категория имеет свой уникальный идентификатор `zone_id`, название `name` и места `seats`. 

Место имеет свой уникальный идентификатор `seat_id`, **boolean** флаг доступности для продажи `available`, а также ряд `row` и номер места `place`

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
