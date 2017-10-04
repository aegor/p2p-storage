# API сервера трансформации и хранения документов

## Предназначено для хранения и преобразования различных документов с помощью конвееров обработки и поиска по ним.

### Типы документов

- .epub
- видео
- аудио
- изображения
- документы офисного формата (.doc, .xls, .odt, .ods, .pdf)

### Типы трансформаций

- epub_verify - верификация .epub с отчётом
- epub_merge - склейка нескольких epub
- epub_split - разрезание epub
- unzip - распаковка архивов .zip
- doc2html - преобразование документов формата .doc, .odt в html
- xls2html - преобразование документов формата .xls, .ods в html
- video2web - преобразование любого распознанного формата видео в DASH/HLS контент для репрезентации в веб-браузерах
- audio2web - тоже самое для аудио
- picture2web - подготовка картинки в разных разрешениях для показа на различных устройствах
- ifok - Если результат предыдущих трансформаций успешен
- iferror - Если результат предыдущих трансформаций неуспешен
- ifwarning - если возникли предупреждения
- else - Иначе
- deleteorig - удалить исходный файл
- deletestorage - удалить весь storage

## Общее описание API

> При описании API сервер будет называться http://filestorage.com

Файлы загружаются на endpoint http://filestorage.com/files с помощью запроса post

Эквивалентная форма в html выглядит так:

```
<form action="http://filestorage.com/files"
   enctype="multipart/form-data" method="post">
   <p><input type="file" name="имя файла.расширение"></p>
</form>
```

Эквивалентный запрос с помощью curl выглядит так:

```
curl -F "name=имя файла.расширение" -F "file=@/Users/I/Pictures/test.jpg" http://filestorage.com/files
```

Результат загрузки от сервера в виде json будет выглядеть так:

```
{
  "id": "550e8400-e29b-41d4-a716-446655440000", //uuid in storage
  "name": "имя файла.расширение",               // file name
  "uploaded": "1507098783", 	                // unix timestamp
  "user": "bob"                                 // Имя пользователя из oauth2 аутентификации
}
```

Хранилище файла - это каталог с именем id. В успешно созданное хранилище впоследствии можно добавлять файлы по этому же URL с добавлением Id, например:

```
curl -F "name=имя файла.расширение -F "file=@/Users/I/Pictures/test.jpg" http://filestorage.com/files/550e8400-e29b-41d4-a716-44665544000
```

При этом результат загрузки от сервера в виде json будет выглядеть так:

```
{
  "id": "550e8400-e29b-41d4-a716-446655440000", //uuid in storage
  "name": "имя файла.расширение",               // file name
  "uploaded": "1507098787", 	                // unix timestamp
  "user": "bob"                                 // Имя пользователя из oauth2 аутентификации
}
```

Несколько файлов также могут быть загружены в виде zip-архива с указанием задачи распаковки.


Над самим хранилищем, как и над всеми файлами хранилища допустимы все CRD-операции (Create, Read, Delete).

Например:

- Удаление хранилища:
```
curl -X DELETE http://filestorage.com/files/550e8400-e29b-41d4-a716-44665544000
```

- Удаление файла в хранилище:

```
curl -X DELETE http://filestorage.com/files/550e8400-e29b-41d4-a716-44665544000/имя%20файла.расширение
```

- Чтение файла из хранилища:

```
curl http://filestorage.com/files/550e8400-e29b-41d4-a716-44665544000/имя%20файла.расширение
```

- Чтение содержимого хранилища с пагинацией:
```
curl http://filestorage.com/files/550e8400-e29b-41d4-a716-44665544000
```

- Чтение всех хранилищ с пагинацией:
```
curl http://filestorage.com/files
```

HATEOAS-ответ будет в таком виде:

```
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/files{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/files/search"
    }
  },
  "_embedded" : {
"files": [
{
  "id": "550e8400-e29b-41d4-a716-446655440000", 
  "name": "имя файла.расширение",               
  "uploaded": "1507098787",                   
  "user": "bob",
  "_links" : {
        "self" : {
          "href" : "http://localhost:8080/files/550e8400-e29b-41d4-a716-446655440000"
        }
  }                               
}
]
  },
  "page" : {
    "size" : 20,
    "totalElements" : 1,
    "totalPages" : 1,
    "number" : 0
  }
}
```

### Файндеры. Примеры поиска (HATEOAS-ответы будут в виде, приведённом выше):

```
curl http://filestorage.com/files/search/findByName?name=имя%20файла.расширение

curl http://filestorage.com/files/search/findByNameLikeIgnoreCase?name=иМя%20файЛа

curl http://filestorage.com/files/search/findByTagNameLikeIgnoreCase?name=viDeo

curl http://filestorage.com/files/search/findByContentLikeIgnoreCase?content=Об%20этом%20уроке
```

и т.д. 

Список всех доступных файндеров можно получить в виде HATEOAS-ответа по адресу:

```
curl http://filestorage.com/files/search
```

Ответ будет в таком виде:

```
{
  "_links" : {
    "findByName" : {
      "href" : "http://filestorage.com/files/search/findByLastName{?name}",
      "templated" : true
    }
  }
}
```

### Работа с трансформациями

Рассмотрим пример:
 
- Загрузить несколько файлов .epub в виде zip-архива
- Проверить их на корректность
- Если нет ошибок, удалить исходный .zip - файл  
- Если есть ошибки - удалить весь storage
- Проверить все .epub-файлы на корректность
- Если есть ошибки - удалить весь storage

```
curl \
-F "name=имя файла.расширение \
-F "file=@/Users/I/Pictures/test.zip" \
http://filestorage.com/files?transform=unzip,iferror,deletestorage,else,epubverify,iferror,deletestorage,else,{deleteorig,epubmerge}
```
> Как видно из примера, операции конвеера можно группировать с помощью фигурных скобок. Это особенно удобно для группировки блоков if* и else

> В седующей версии продукта будут предложены более изощрённые способы управления конвеером трансформаций.

## Обработка ошибок

При возникновении ошибок на любом этапе (загрузка, поиск, трансформации, манипуляции с файлами и т.д.) все перехватываемые исключения отображаются в виде:

```
{
"timestamp": 1507104495025,
"status": 404,
"error": "Not Found",
"message": "Path Not Found",
"path": "/d"
}
```

## Самодокументированное API

Всё API представлено в двух самодукументированных форматах: HATEOAS и Swagger

Само по себе API выполнено в стиле [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)

Кроме того, имеется автосгенерированная документация и API в стиле swagger по адресам:

http://filestorage.com/v2/api-docs (Swagger JSON API)

и 

http://filestorage.com/swagger-ui.html (Swagger UI)

## Мониторинг состояния приложения

Мониторинг состояния приложения, health-индикаторы, все endpoints, состояние памяти и многое другое доступно на /actuator endpoint.

Документация на этот API находится здесь:
https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-monitoring.html 

Все actuator endpoints:

```
curl http://filestorage.com/actuator

{
"links": [
{
"rel": "self",
"href": "http://localhost:8080/actuator"
},
{
"rel": "trace",
"href": "http://localhost:8080/actuator/trace"
},
{
"rel": "mappings",
"href": "http://localhost:8080/actuator/mappings"
},
{
"rel": "info",
"href": "http://localhost:8080/actuator/info"
},
{
"rel": "health",
"href": "http://localhost:8080/actuator/health"
},
{
"rel": "autoconfig",
"href": "http://localhost:8080/actuator/autoconfig"
},
{
"rel": "auditevents",
"href": "http://localhost:8080/actuator/auditevents"
},
{
"rel": "dump",
"href": "http://localhost:8080/actuator/dump"
},
{
"rel": "env",
"href": "http://localhost:8080/actuator/env"
},
{
"rel": "metrics",
"href": "http://localhost:8080/actuator/metrics"
},
{
"rel": "heapdump",
"href": "http://localhost:8080/actuator/heapdump"
},
{
"rel": "configprops",
"href": "http://localhost:8080/actuator/configprops"
},
{
"rel": "beans",
"href": "http://localhost:8080/actuator/beans"
},
{
"rel": "logfile",
"href": "http://localhost:8080/actuator/logfile"
},
{
"rel": "shutdown",
"href": "http://localhost:8080/actuator/shutdown"
},
{
"rel": "loggers",
"href": "http://localhost:8080/actuator/loggers"
}
]
}
```

## TODOs

- Описание трансформаторов и форматов их выходных файлов
- Описание механизма аутентификации
- Более гибкий механизм описания заданий
- Обсуждение и формализация файндеров

## Аутентификация

