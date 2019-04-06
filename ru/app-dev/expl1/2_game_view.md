[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/2_game_view.md)

### Шаг 2. Формирование представления игры

Будем писать код в файле `services/tetris/module/game/view/_root.php`. Данный файл уже содержит код:
```php
/**
 * @var lx\Module $Module
 * @var lx\Block $Block
 * */

```
В данном файле доступно две контекстные переменные:
* $Module - экземпляр модуля, к которому относится код данного блока представления. В нашем случае это модуль `tetris:game`.
* $Block - экземпляр самого блока представления. Фактически это виртуальный аватар виджета (класса `lx\Box`), в который будет произведен рендеринг кода, написанного в данном файле.

Сразу определяем размеры поля. Тетрис играется на поле 10x20.
```php
$cols = 10;
$rows = 20;
```

Добавим виджет, в который поместим все всязанное с игрой. Зальем его белым фоном. Параметр `key` - важен для нахождения виджета на стороне клиента. Классу виджета `lx\Box` на серверной стороне соответствует класс `lx.Box` на клиентской стороне (и так для любого виджета). Создавая данный виджет на серверной стороне с ключом `tetris`, на клиентской стороне мы сможем получить соответствующий ему объект класса `lx.Box`, найдя его при помощи кода `Module->>tetris`. Также определим этому виджету стратегию позиционирования `slot` с единственным слотом и коэффициентом 0.7, чтобы соотношение высоты и ширины было постоянное. Также для аккуратности укажем отступ '10px'. Применение данной стратегии позиционирования сразу создаст один слот. Этот слот используем в качестве формы для всего необходимого. 

```php
$tetris = new lx\Box(['key' => 'tetris']);
$tetris->fill('white');
$tetris->slot([
	'indent' => '10px',
	'cols' => 1,
	'rows' => 1,
	'k' => 0.7,
]);
$slot = $tetris->child(0);
```

Определим стратегию позиционирования в слоте `gridProportional` - чтобы все дочерние элементы располагались согласно сетке, зададим размеры сетки и отступ. Методом `begin()` "открываем" форму - она станет родительским элементом для всех вновь создаваемых элементов.
```php
$slot->gridProportional([
	'rows' => $rows,
	'cols' => $cols + 4,
	'indent' => '10px'
]);

$slot->begin();
```

Создадим игровое поле. Для этого просто добавим таблицу. Укажем размер, который она займет в сетке, и количество строк и колонок.
```php
	new lx\Table([
		'key' => 'map',
		'size' => [$cols, $rows],
		'cols' => $cols,
		'rows' => $rows
	]);
```

Добавляем кнопку запуска новой игры.
```php
	new lx\Button([
		'key' => 'newGame',
		'size' => [4, 2],
		'text' => 'New game'
	]);
```

Добавляем небольшое поле, которое будет показывать следующиу фигуру.
```php
	(new lx\Box(['width'=>4, 'text'=>'Next']))->align(lx::CENTER, lx::MIDDLE);
	new lx\Table([
		'key' => 'next',
		'size' => [4, 4],
		'rows' => 4,
		'cols' => 4
	]);
```

Добавим кнопку постановки игры на паузу.
```php
	new lx\Button([
		'key' => 'pause',
		'size' => [4, 2],
		'text' => 'Pause'
	]);
```

Добавим поле, где будет выводиться текущий уровень игры. Парамерт `field` работает как `key` (если `key` не указан явно, но как правило в этом нет необходимости), плюс сообщает данному виджету, что он будет ослеживать поле с соответствующим названием (в нашем случае `level`) у модели, с которой виджет будет связан.
```php
	(new lx\Box(['width'=>4, 'text'=>'Level:']))->align(lx::CENTER, lx::MIDDLE);
	(new lx\Box([
		'field' => 'level',
		'size' => [4, 2],
		'style' => ['fill'=>'lightgray']
	]))->align(lx::CENTER, lx::MIDDLE);
```

Аналогично предыдущему, добавим поле, где будет выводиться количество убранных полос.
```php
	(new lx\Box(['width'=>4, 'text'=>'Filled:']))->align(lx::CENTER, lx::MIDDLE);
	(new lx\Box([
		'field' => 'filled',
		'size' => [4, 2],
		'style' => ['fill'=>'lightgray']
	]))->align(lx::CENTER, lx::MIDDLE);
```

Аналогично предыдущему, добавим поле, где будет выводиться количество заработанных очков.
```php
	(new lx\Box(['width'=>4, 'text'=>'Score:']))->align(lx::CENTER, lx::MIDDLE);
	(new lx\Box([
		'field' => 'score',
		'size' => [4, 2],
		'style' => ['fill'=>'lightgray']
	]))->align(lx::CENTER, lx::MIDDLE);
```

Все необходимые поля созданы, закрываем форму.
```php
$slot->end();
```

Используем полезную форму для ввода значений, оформленную как блок. Блок реализован в сервисе `lx/lx-tools`, Вы ведь установили его?
```php
$tools = lx::getService('lx/lx-tools');
$tools->renderBlock('inputPopup');
```

И так, получился такой код представления:
```php
/**
 * @var lx\Module $Module
 * @var lx\Block $Block
 * */

$cols = 10;
$rows = 20;

$tetris = new lx\Box(['key' => 'tetris']);
$tetris->fill('white');
$tetris->slot([
	'indent' => '10px',
	'cols' => 1,
	'rows' => 1,
	'k' => 0.7,
]);
$slot = $tetris->child(0);

$slot->gridProportional([
	'rows' => $rows,
	'cols' => $cols + 4,
	'indent' => '10px'
]);

$slot->begin();
	new lx\Table([
		'key' => 'map',
		'size' => [$cols, $rows],
		'cols' => $cols,
		'rows' => $rows
	]);

	new lx\Button([
		'key' => 'newGame',
		'size' => [4, 2],
		'text' => 'New game'
	]);

	(new lx\Box(['width'=>4, 'text'=>'Next']))->align(lx::CENTER, lx::MIDDLE);
	new lx\Table([
		'key' => 'next',
		'size' => [4, 4],
		'rows' => 4,
		'cols' => 4
	]);

	new lx\Button([
		'key' => 'pause',
		'size' => [4, 2],
		'text' => 'Pause'
	]);

	(new lx\Box(['width'=>4, 'text'=>'Level:']))->align(lx::CENTER, lx::MIDDLE);
	(new lx\Box([
		'field' => 'level',
		'size' => [4, 2],
		'style' => ['fill'=>'lightgray']
	]))->align(lx::CENTER, lx::MIDDLE);

	(new lx\Box(['width'=>4, 'text'=>'Filled:']))->align(lx::CENTER, lx::MIDDLE);
	(new lx\Box([
		'field' => 'filled',
		'size' => [4, 2],
		'style' => ['fill'=>'lightgray']
	]))->align(lx::CENTER, lx::MIDDLE);

	(new lx\Box(['width'=>4, 'text'=>'Score:']))->align(lx::CENTER, lx::MIDDLE);
	(new lx\Box([
		'field' => 'score',
		'size' => [4, 2],
		'style' => ['fill'=>'lightgray']
	]))->align(lx::CENTER, lx::MIDDLE);
$slot->end();

$tools = lx::getService('lx/lx-tools');
$tools->renderBlock('inputPopup');

```
Проверим что у нас получилось, введем в браузере URL `your.domain/tetris/game`.

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/3_figures.md)
