[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/lx-core/doc/plugin-config.md)

## Конфигурация плагина

### Оглавление
* [Общая информация](#common)
* [Пример конфигурации](#example)
* [aliases и useCommonAliases](#aliases)
* [jsBootstrap и jsMain](#frontend)
* [rootSnippet](#rootSnippet)
* [snippets](#snippets)
* [images и css](#assets)
* [respondents](#respondents)


<a name="common"><h3>Общая информация</h3></a>
Следующие утверждения справедливы для конфигурационных файлов плагинов:
* Конфигурационные файлы могут быть в форматax: `yaml`, `php`, `json`.
* Если данные конфигурации нет смысла разносить по нескольких файлам, конфигурационный файл может быть один. Тогда он должен иметь имя `config.(yaml|php|json)`.
* Если данные конфигурации удобнее разнести по нескольким файлам, главный файл конфигурации должен иметь имя `main.(yaml|php|json)` и находиться в каталоге `config`.
* Конфигурационный файл (или каталог с конфигурационными файлами) должен располагаться непосредственно в каталоге плагина.
* Есть файл с дефолтной конфигурацией для плагинов. Все несконфигурированные параметры в самих плагинах будут взяты из дефолтной конфигурации. Путь к данному файлу: `path/to/project/lx/config/plugin.php`.


<a name="example"><h3>Пример конфигурации</h3></a>
Вариант в синтаксисе `yaml`
```yaml
aliases: []
useCommonAliases: true

frontend: frontend
jsBootstrap: _bootstrap.js
jsMain: _main.js

respondents:
	Respondent: backend\Respondent

rootSnippet: snippets/_root.php
snippets: snippets

images: assets/images
css: assets/css

i18nMap: some\Translater
```


<a name="aliases"><h3>Параметры конфигурации: <b>aliases</b> и <b>useCommonAliases</b></h3></a>
Ключу `aliases` в конфигурации плагина сопоставляется ассоциативный массив, ключами которого являются псевдонимы для путей относительно корневого каталога плагина, а значениями - сами пути. Пример:
```yaml
aliases:
  # Псевдоним для пути относительно каталога плагина
  someAlias: some-directory

  # Псевдоним, объявленный через другой псевдоним
  anotherAlias: @someAlias/directory

  # Можно даже так - для каталога другого плагина
  relativeAlias: {service:another/service-name:anotherModeul}@aliasFromAnotherPlugin
```
Пример использования псевдонимов:
```php
// Код PHP

// Получение пути
$path = $plugin->getFileName('@someAlias/someFile');

// Получение файла
$file = $plugin->getFile('@anotherAlias/someFile');
```
Ключу `useCommonAliases` в конфигурации плагина сопоставляется булево значение, определяющее нужно ли импортировать к данный клас псевдонимы, определенные к общем конфигурационном файле для всех плагинов.


<a name="frontend"><h3>Параметры конфигурации: <b>jsBootstrap</b> и <b>jsMain</b></h3></a>
Ключу `jsBootstrap` в конфигурации плагина сопоставляется путь к файлу, содержащему JS-код, который будет выполнен до разворачивания плагина на стороне клиента.<br>
Ключу `jsBootstrap` в конфигурации плагина сопоставляется путь к файлу, содержащему JS-код, который будет выполнен после разворачивания плагина на стороне клиента.


<a name="rootSnippet"><h3>Параметр конфигурации: <b>rootSnippet</b></h3></a>
Ключу `rootSnippet` в конфигурации плагина сопоставляется путь к файлу, содержащему код представления, описывающий корневой сниппет (графический интерфейс) данного плагина.


<a name="snippets"><h3>Параметр конфигурации: <b>snippets</b></h3></a>
Ключу `snippets` в конфигурации плагина сопоставляется путь (или массив путей) к каталогу, содержащему сниппеты данного плагина.


<a name="assets"><h3>Параметры конфигурации: <b>images</b> и <b>css</b></h3></a>
Ключу `images` в конфигурации плагина сопоставляется путь (или массив путей) к каталогу, содержащему изображения для данного плагина.<br>
Ключу `css` в конфигурации плагина сопоставляется путь (или массив путей) к каталогу, содержащему CSS-файлы, необходимые для данного плагина. Никаких других действий для подключения CSS-файлом предпринимать не нужно - они будут подключены автоматически.


<a name="respondents"><h3>Параметр конфигурации: <b>respondents</b></h3></a>
Респонденты это функциональные элементы плагина. Фактически являются php классами. Представляют собой ajax-контроллеры, которые отдают данные клиентской части плагина.
Пример респондента:
* Определение в конфигурации плагина
```yaml
respondents:
  Respondent: backend\Respondent
```
* Код респондента (согласно приведенной конфигурации должен находиться в файле `backend/Respondent.php` относительно корня плагина)
```php
// Код PHP

namespace path\to\plugin\backend;

class Respondent extends \lx\Respondent
{
  public function test()
  {
    return 'Hello from server';
  }
}
```
* Использование респондента в js-коде плагина
```js
^Respondent.test().then((result) => {
  // result содержит строку 'Hello from server'
  console.log(result);
});
```
