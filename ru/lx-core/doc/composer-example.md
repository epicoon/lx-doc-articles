## Зависимости для композера
Данные зависимости нужно сохранить как файл `composer.json` в корне проекта, после чего можно будет использовать команду `composer install`.

Загружаемые пакеты:
* `lx/lx-core` - ядро lx-платформы
* `lx/lx-tools` - набор часто используемых в приложениях виджетов и сниппетов
* `lx/lx-model` - ORM для lx-платформы
* `lx/lx-auth` - инструменты аутентификации OAuth2 и авторизации RBAC
* `lx/lx-doc` - документация классов для используемых в платформе пакетов
* `lx/lx-demo` - набор демонстраций различных инструментов

```json
{
    "require":{
        "lx/lx-core":"dev-master",
        "lx/lx-tools":"dev-master",
        "lx/lx-model":"dev-master",
        "lx/lx-auth":"dev-master",
        "lx/lx-doc":"dev-master",
        "lx/lx-demo":"dev-master"
    },
    "repositories":[
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-core"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-tools"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-model"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-auth"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-doc"
        },
        {
            "type":"git",
            "url":"https://github.com/epicoon/lx-demo"
        }
    ],
    "scripts": {
        "post-update-cmd": [
            "php vendor/lx/lx-core/lx-install"
        ]
    }
}
```
