# Shared database
База данных типа ключ-значения для текстовых данных с возможностью синхорнизацией между нодами. 

## Запуск в Docker контейнере
```sh
$ cd shared_database/
$ docker build --tag=shared_database .
$ docker-compose up
```
После старта доступны две ноды:
```sh
node1 http://localhost:8080
node2 http://localhost:8081
```

## WebAPI

| Метод | URL | Описание |
| ------ | ------ | ------ |
| GET | /nodes/update?name=key1&value=val1 | Update or set key value |
| GET | /nodes/get?name=key1 | Get key value |
| GET | /nodes/remove?name=key1 | Remove key |
| GET | /nodes/dump | Dump all data |


### Update or set key value
Для установки значения ключа необходимо выполнить запрос **GET** с переменными `name` и `value`:
```sh
GET /nodes/update?name=key1&value=val1
```
Если ключ существуе, значение обновляется, если ключ отсутствует в базе -- он создается и ему присваевается значение. 


### Get key value
Для получения значения ключа необходимо выполнить запрос **GET** с переменными `name`:
```sh
GET /nodes/get?name=key1 
```

### Remove key 
Для удаления ключа и его значения необходимо выполнить запрос **GET** с переменными `name`:
```sh
GET | /nodes/remove?name=key1
```

### Dump all data
Для всех ключе и их значений необходимо выполнить запрос **GET**:
```sh
GET /nodes/dump
```


## Описание и архитектура
Приложение основано на модульной архитектуре на базе асинхронного фреймворка `aiohttp`, `python 3.6` и `celery`.
- Main App
    - Nodes
    - Subapp (in future)
**Main App** - приложение-контейнер, к которому подключаются субприложения. Для расширения функцилнала в будущем, достаточно подключить новое субприложение.
**Main App** выполняет фунуцию роутера, распределя запросы между субприложениями.

**Nodes** -- субприложение которое отвечает за обработку запросов к базе и создает задания для их репликации на другие ноды.

В качестве хранилища данных используется `Redis`.
Для хранения списка нод используется общая `Redis` база для всех нод.

## Алгоритм работы
При старте ноды, имя хоста ноды добавляется в общую для всех базу.
При поступления запроса `update` и `remove` сначала выполняется действие с базой хранени данных на конкретной ноде.
Далее создается асинхронное задание для внесения изменений в другие ноды.
Нода передает аналогичный запрос к api по всех нодам с дополнительной переменной `replicate` для зацикливания.

