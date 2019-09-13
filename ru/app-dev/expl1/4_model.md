[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/4_model.md)

### Шаг 4. Создание модели TetrisLeader

Чтобы продемонстрировать работу с моделями данных, предусмотрим функционал сохранения пяти лучших результатов по итогам игр. Для этого создадим модель `TetrisLeader`. В каталоге `services/tetris/model` создадим файл `TetrisLeader.yaml`, добавим в него описание нашей модели:
```yaml
TetrisLeader:
  table: tetris_leader
  fields:
    name: {type: varchar}
    place: {type: integer}
    score: {type: integer}
    level: {type: integer}

```

Также добавим в этот файл такой код:
```yaml
!!addTable: [
  [ name,   place, score, level ]
  [ noname, 1,     0,     0     ]
  [ noname, 2,     0,     0     ]
  [ noname, 3,     0,     0     ]
  [ noname, 4,     0,     0     ]
  [ noname, 5,     0,     0     ]
]
```
Это директива для добавления нескольких экземпляров модели. Нужно эту директиву выполнить.

Для начала нужно создать базу данных и настроить подключение к ней. Процесс создания базы данных к платформе не относится, но автор надеется, что это не станет трудной задачей. Когда база создана, нужно открыть файл `services/tetris/lx-config.yaml` и в блоке `service` добавить настройки подключения к Вашей базе, примерно так:
```yaml
name: tetris

autoload:
  psr-4:
    tetris\: ''

service:
  class: tetris\Service

  router:
    type: map
    routes:
      /: {plugin: main}
      game: {plugin: game}
      leaders: {plugin: leaders}

  plugins: plugin
  models: model
  modelCrudAdapter: lx\DbCrudAdapter

  # Обратите внимание на имя базы, пользователя и пароль. Подставьте свои
  db:
    hostname: localhost
    username: furcon
    password: 123456
    dbName: furcon

```
Также обратите внимание на такой параметр, как `modelCrudAdapter`. Это инструмент, который осуществляет CRUD-операции над моделями. Можно использовать другие механизмы, можно написать свой инструмент, но поскольку мы используем базу данных, нам как раз подходит класс `lx\DbCrudAdapter`.

Если все сделано, пора выполнить директиву добавления экземпляров модели. Для этого будем использовать [CLI](https://github.com/epicoon/lx-core/README-ru.md#cli). Выполним команду `migrate-check`, чтобы предварительно проверить модели, которые требуют применения изменений. Мы дожны увидеть:
```
Following migrations and models need be applied:
Service: tetris
* Models:
- TetrisLeader
```
Выполним команду `migrate-run`, изменения будут применены, а именно: в базе данных создастся таблица `tetris_leader` и в нее добавится пять записей. Теперь если мы посмотрим на содержимое файла `service/tetis/model/TetrisLeader.yaml`, директиву мы уже не увидим. В то же время, если проверить базу данных - обнаружим там таблицу и нужные нам записи.

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/5_game_respondent.md)
