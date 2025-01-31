[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/lx-core/doc/service-config.md)

## Конфигурация сервиса

### Оглавление
* [Общая информация](#common)
* [Пример конфигурации](#example)
* [name](#name)
* [autoload](#autoload)
* [service](#service)
* [class](#class)
* [aliases и useCommonAliases](#aliases)
* [router](#router)
* [plugins и dynamicPlugins](#plugins)
* [models](#models)
* [modelCrudAdapter](#modelCrudAdapter)
* [db и dbList](#db)
* [i18nMap и i18nFile](#i18nMap)


<a name="common"><h3>Общая информация</h3></a>
Сервис это пакет, умеющий отвечать на запросы. Следующие утверждения справедливы для конфигурационных файлов сервисов:
* Конфигурационные файлы могут быть в форматax: `yaml`, `php`, `json`.
* Если данные конфигурации нет смысла разносить по нескольких файлам, конфигурационный файл может быть один. Тогда он должен иметь имя `config.(yaml|php|json)`.
* Если данные конфигурации удобнее разнести по нескольким файлам, главный файл конфигурации должен иметь имя `main.(yaml|php|json)` и находиться в каталоге `config`.
* Конфигурационный файл (или каталог с конфигурационными файлами) должен располагаться непосредственно в каталоге сервиса.
* Есть файл с дефолтной конфигурацией для сервисов. Все несконфигурированные параметры в самих сервисах будут взяты из дефолтной конфигурации. Путь к данному файлу: `path/to/project/lx/config/service.php`.


<a name="example"><h3>Пример конфигурации</h3></a>
Вариант в синтаксисе `yaml`
```yaml
name: your/new-service

autoload:
  psr-4:
    your\newService\: ''

service:
  class: your\newService\Service

  router:
    type: map
    routes:
      some-route: { plugin: main }
      some-route/special: { plugin: somePlugin }

  plugins: plugin
  dynamicPlugins:
    somePlugin:
      prototype: some/service-name:somePluginPrototype
      renderParams: { paramName: value }

  models: model
  modelCrudAdapter: \lx\DbCrudAdapter
  db:
    hostname: localhost
    username: username
    password: 123456
    dbName: dbName

  i18nMap:
    class: namespace\to\SomeTranslater
    tags:
      la-LA: la
```


<a name="name"><h3>Параметр конфигурации: <b>name</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляется полное имя сервиса (пакета).<br>
Согласно PSR-4, полное имя сервиса (пакета) определяется как %имя производителя%/%имя самого пакета%, например `super-star/mega-package`: имя производителя - `super-star`, имя самого пакета - `mega-package`.<br>
Каталог сервиса должен располагаться в проекте согласно этому имени, в одном из каталогов, определенных в конфигурации приложения как каталог с пакетами. Подробнее о [каталогах с пакетами](https://github.com/epicoon/lx-doc-articles/blob/master/ru/lx-core/doc/app-config.md#packagesMap).<br>
Например, в конфигурации приложения определен каталог с пакетами:
```yaml
packagesMap:
  - services
```
И в проекте есть сервис `super-star/mega-package`, тогда путь к каталогу этого сервиса: `path/to/project/services/super-star/mega-package`.<br>
В коде по имени сервиса можно:
* получать экземпляр самого сервиса:
  ```php
  // Код PHP
  $service = \lx::$app->getService('super-star/mega-package');
  ```
* получать путь к каталогу сервиса:
  ```php
  // Код PHP
  $path = '{service:super-star/mega-package}/some/directory/in/service';
  ```


<a name="autoload"><h3>Параметр конфигурации: <b>autoload</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляется ассоциативный массив, описывающий правила актозагрузки PHP-классов.<br>
Механизмы автозагрузки:
* `psr-4`: ассоциативный массив, ключи - пространства имен, задаваеме для PHP-классов сервиса, значения - пути к классам относительно корневого каталога сервиса.
* `classmap`: массив путей к файлам или каталогам. В указанных файлах и каталогах будут найдены и проиндексированы все PHP-классы (для переиндексирования - команда `\amr`, [подробнее](https://github.com/epicoon/lx-doc-articles/blob/master/ru/lx-core/doc/app-config.md#cli)), после чего их можно будет использовать в коде.
* `files`: массив путей к файлам с PHP-кодом, который будет выполнен немедненно и единожды при запуске приложения.
Пример:
```yaml
autoload:
  psr-4:
    megaPackage\: src

  classmap:
    - path/to/SomePhpClass.php
    - path/to/directory/with/phpClasses

  files: 
    - path/to/some/file
    - path/to/another/file
```
> Настоятельно рекомендуем использовать именно механизм автозагрузки `psr-4`, только он позволяет избежать коллизий в именах классов. Другие механизмы нужны только в крайнем случае.

Приведем пример именования класса по PSR-4, согласно вышеприведенной настройке, допустим имеем файл `SomeClass.php` с кодом PHP-класса:
```php
namespace megaPackage\helpers\group;

class SomeClass
{
  // ... код
}

```
Тогда этот файл должен располагаться по пути `path/to/service/src/helpers/group`.


<a name="service"><h3>Параметр конфигурации: <b>service</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляется ассоциативный массив с настройками уже непосредственно функционала сервиса. Именно этот параметр делает пакет сервисом. Сами настройки для этого массива описаны ниже.


<a name="class"><h3>Параметр конфигурации: <b>class</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляется имя класса сервиса. Сервис не обязан иметь свой собственный класс. Если этот параметр не определен, то для создания экземпляра сервиса используется класс `lx\Service`.
Указать информацию о классе можно несколькими способами:
```yaml
# Указание имени класса
class: ClassName

# Указание ассоциативного массива с именем класса и параметрами для его создания
class:
  class: ClassName
  param1: value1
  param2: value2

# Указание ассоциативного массива с именем класса
# и параметрами для его создания явно выделенными в отденый массив для наглядности
class:
  class: ClassName
  params:
    param1: value1
    param2: value2
```


<a name="aliases"><h3>Параметры конфигурации: <b>aliases</b> и <b>useCommonAliases</b></h3></a>
Ключу `aliases` в конфигурации сервиса сопоставляется ассоциативный массив, ключами которого являются псевдонимы для путей относительно корневого каталога сервиса, а значениями - сами пути. Пример:
```yaml
aliases:
  # Псевдоним для пути относительно каталога сервиса
  someAlias: some-directory

  # Псевдоним, объявленный через другой псевдоним
  anotherAlias: @someAlias/directory

  # Можно даже так - для каталога другого сервиса
  relativeAlias: {service:another/service-name}@aliasFromAnotherService
```
Пример использования псевдонимов:
```php
// Код PHP

// Получение пути
$path = $service->getFilePath('@someAlias/someFile');

// Получение файла
$file = $service->getFile('@anotherAlias/someFile');
```
Ключу `useCommonAliases` в конфигурации сервиса сопоставляется булево значение, определяющее нужно ли импортировать к данный клас псевдонимы, определенные к общем конфигурационном файле для всех сервисов.


<a name="router"><h3>Параметр конфигурации: <b>router</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляется ассоциативный масив, который обязательно должен содержать элемент `type` с одним из двух значений:
* `map` - в этом случае требуется поясняющий параметр `routes` - карта роутинга. Карта представляет из себя ассоциативный массив, где ключ - URL запроса, значение - данные об объекте, куда перенаправляется запрос.
* `class` - в этом случае требуется поясняющий параметр `name` - имя класса, отнаследованного от `lx\ServiceRouter`
Пример:
```yaml
router:
  type: map
  routes:
    # Направление запроса на контроллер
    # Будет возвращен результат вызова метода контроллера [[run()]]
    some-route-1: ControllerClassName

    # Направление запроса на контроллер
    # Будет возвращен результат вызова метода(экшена) контроллера [[actionName()]]
    some-route-2: ControllerClassName::actionName

    # Направление запроса на экшен. Будет возвращен результат вызова метода экшена [[run()]]
    some-route-3: {action: ActionClassName}

    # Направление запроса на плагин. Будет возвращен результат рендеринга плагина
    some-route-4: {plugin: pluginName}
```
Подробнее о механизме роутинга можно узнать по [ссылке](https://github.com/epicoon/lx-doc-articles/blob/master/ru/lx-core/doc/routing.md).


<a name="plugins"><h3>Параметры конфигурации: <b>plugins</b> и <b>dynamicPlugins</b></h3></a>
Ключу `plugins` в конфигурации сервиса сопоставляется массив путей, или один путь (относительно корневого каталога сервиса) к каталогам, в которых располагаются плагины данного сервиса. Имена каталогов плагинов являются именами самих плагинов.<br>
Пример:
```yaml
name: current/service

service:
  # Модули сервиса будут располагаться в каталоге 'path/to/current/service/plugin'
  plugins: plugin
```
В каталоге `path/to/current/service/plugin` создаем плагин с именем `myPlugin`, тогда каталог этого плагина располагается по пути `path/to/current/service/plugin/myPlugin`. Сам плагин в коде доступен по имени `current/service:myPlugin`, это означает, что можно, например, написать такой код в файле любого сниппета любого другого плагина:
```js
// Создадим виджет
let box = new lx.Box(boxConfig);

// Модуль будет срендерен в данном виджете
box.setPlugin('current/service:myPlugin');
```

Ключу `dynamicPlugins` в конфигурации сервиса сопоставляется ассоциативный массив, ключами которого являются имена динамических плагинов, формируемых для данного сервиса, а значениями - данные о плагине-прототипе.<br>
Пример:
```yaml
name: current/service

service:
  dynamicPlugins:
    # Данный плагин будет доступен по имени 'current/service:models'
    models:
      # Выбираем прототип для динамического плагина - какой-то другой плагин
      prototype: lx/lx-model:modelManager
      # Можно передать параметры, если прототипу они нужны
      renderParams: { service: current/service }
```
Динамический плагин формируется исключительно в конфигурации, в остальном он функционирует также, как обычный плагин. К нему можно из любого места кода обращаться по имени, его можно рендерить в любых сниппетах.


<a name="models"><h3>Параметр конфигурации: <b>models</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляется массив путей, или один путь (относительно корневого каталога сервиса) к каталогам, в которых располагаются модели данного сервиса.<br>
Модели - это классы данных, поддерживающие CRUD-операции.<br>
Данный параметр не обязательно конфигурировать, если использвание моделей не требуется.


<a name="modelCrudAdapter"><h3>Параметр конфигурации: <b>modelCrudAdapter</b></h3></a>
Данному ключу в конфигурации сервиса сопоставляются данные об инструменте, который осуществляет CRUD-операции над моделями.
Указать информацию о классе CRUD-инструмента можно несколькими способами:
```yaml
# Указание имени класса
modelCrudAdapter: ClassName

# Указание ассоциативного массива с именем класса и параметрами для его создания
modelCrudAdapter:
  class: ClassName
  param1: value1
  param2: value2

# Указание ассоциативного массива с именем класса
# и параметрами для его создания явно выделенными в отденый массив для наглядности
modelCrudAdapter:
  class: ClassName
  params:
    param1: value1
    param2: value2
```
Если параметр не сконфигурирован, никакой другой инструмент по умолчанию использован не будет. Данный параметр не нужно конфигурировать только в случае, когда CRUD-функционал работы с моделями не требуется.101 


<a name="db"><h3>Параметры конфигурации: <b>db</b> и <b>dbList</b></h3></a>
Ключу `db` в конфигурации сервиса сопоставляется ассоциативный массив настроек подключения к базе данных.<br>
Если подключений несколько, поможет параметр конфигурации `dbList`, которому сопоставляется ассоциативный массив, в котором ключи - названия настроек подключения, значения - сами настройки подключения.<br>
Пример:
```yaml
# Подключение по умолчанию (не требует явного использования ключа 'db'), пример кода:
# $dbConnection = $service->db();
db:
  hostname: localhost
  username: username
  password: 123456
  dbName: dbName

# Несколько настроек для подключения
dbList:
  # Использование конкретного набора настроек требует явного использования ключа, пример кода:
  # $dbConnection = $service->db('db1');
  db1: [...]
  db2: [...]
```
Если подключения к базе не требуется, эти параметры конфигурировать не нужно.


<a name="i18nMap"><h3>Параметры конфигурации: <b>i18nMap</b> и <b>i18nFile</b></h3></a>
Ключу `i18nMap` в конфигурации сервиса сопоставляется информация об инструменте интернационализации.<br>
Если параметр не сконфигурирован, используется класс `lx\I18nServiceMap`.

Ключу `i18nFile` в конфигурации сервиса сопоставляется путь к карте интернационализации.

[Подробнее о механизме интернационализации](https://github.com/epicoon/lx-doc-articles/blob/master/ru/lx-core/doc/i18n.md).
