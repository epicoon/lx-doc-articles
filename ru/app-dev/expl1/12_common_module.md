[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/en/app-dev/expl1/12_common_module.md)

### Шаг 12. Модуль, консолидирующий предыдущие

Объединим оба модуля `game` и `leaders` в одном. После чего можно будет даже убрать роутинг на эти отдельные модули.

Будем писать код в файле `services/tetris/module/main/view/_root.php`.

Все максимально просто. Для каждого модуля создаем виджет `lx\ActiveBox` и загружаем в них соответствующие модули. Ключи `adhesive` в конфигурациях создания виджетов делают их края прилипающими друг к другу.
```php
&lt;?php
/**
 * @var lx\Module Module
 * @var lx\Block Block
 * */

$tetris = new lx\ActiveBox([
	'geom' => [5, 5, 40, 90],
	'header' => 'Tetris',
	'adhesive' => true,
]);
$tetris->border();
$tetris->get('body')->setModule(lx::getModule('tetris:game'));

$leaders = new lx\ActiveBox([
	'geom' => [55, 10, 40, 40],
	'header' => 'Leaders',
	'adhesive' => true,
]);
$leaders->border();
$leaders->get('body')->setModule(lx::getModule('tetris:leaders'));
```

Поздравляю, теперь Вы слишком много знаете...
