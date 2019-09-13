## Зависимости для композера
Данные зависимости нужно сохранить как файл `composer.json` в корне проекта, после чего можно будет использовать команду `composer install`.

Загружаемые пакеты:
* `lx/lx-core` - ядро lx-платформы
* `lx/lx-doc` - документация классов для используемых в платформе пакетов
* `lx/lx-demo` - набор демонстраций различных инструментов
* `lx/lx-dev-wizard` - набор инструментов, помогающих в разработке
* `lx/lx-tools` - набор часто используемых в приложениях виджетов и сниппетов

```json
{
    "require":{
        "lx/lx-core":"dev-master",
        "lx/lx-doc":"dev-master",
        "lx/lx-demo":"dev-master",
        "lx/lx-dev-wizard":"dev-master",
        "lx/lx-tools":"dev-master"
    },
    "repositories":[
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-core"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-doc"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-demo"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-dev-wizard"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-tools"
        }
     ]
}
```
