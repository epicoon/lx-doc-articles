[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/12_common_plugin.md)

### Шаг 12. Плагин, консолидирующий предыдущие

Объединим оба плагина `game` и `leaders` в одном. После чего можно будет даже убрать роутинг на эти отдельные плагины.

Будем писать код в файле `services/tetris/plugin/main/snippets/_root.php`.

Все максимально просто. Для каждого плагина создаем виджет `lx\ActiveBox` и загружаем в них соответствующие плагины. Ключи `adhesive` в конфигурациях создания данных виджетов делают их края прилипающими друг к другу.
```php
/**
 * @var lx\Application $App
 * @var lx\Plugin $Plugin
 * @var lx\Snippet $Snippet
 * */

$tetris = new lx\ActiveBox([
	'geom' => [5, 5, 40, 90],
	'header' => 'Tetris',
	'adhesive' => true,
]);
$tetris->border();
$tetris->get('body')->setPlugin($App->getPlugin('tetris:game'));

$leaders = new lx\ActiveBox([
	'geom' => [55, 10, 40, 40],
	'header' => 'Leaders',
	'adhesive' => true,
]);
$leaders->border();
$leaders->get('body')->setPlugin($App->getPlugin('tetris:leaders'));
```

Поздравляю, теперь Вы знаете слишком много...
