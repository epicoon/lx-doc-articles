[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/11_leaders_module.md)

### Шаг 11. Модуль для визуализации лидеров

Опишем представление в файле `services/tetris/module/leaders/view/_root.php`. Принцип аналогичен описанному во [втором уроке](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/2_game_view.md). Из нового - только использование стратегий позиционирования `streamProportional` и `stream`. Элемент `$header` будет содержать имена колонок. Элемент с ключом `leadersTable` заготовим, чтобы в него выводить список лидеров. Сам вывод лидеров будет происходить на стороне клиента.
```php
/**
 * @var lx\Module Module
 * @var lx\Block Block
 * */

$leaders = new lx\Box();
$leaders->fill('white');
$leaders->slot([
	'indent' => '10px',
	'cols' => 1,
	'rows' => 1,
	'k' => 1.3,
]);

$slot = $leaders->child(0);
$slot->streamProportional(['indent' => '10px']);
$slot->begin();
	$header = new lx\Box();
	$header->fill('lightgray');
	$header->gridProportional();
	$header->begin();
		(new lx\Box([ 'text' => '#',     'width' => 1 ]))->align(lx::CENTER, lx::MIDDLE);
		(new lx\Box([ 'text' => 'name',  'width' => 5 ]))->align(lx::CENTER, lx::MIDDLE);
		(new lx\Box([ 'text' => 'score', 'width' => 3 ]))->align(lx::CENTER, lx::MIDDLE);
		(new lx\Box([ 'text' => 'level', 'width' => 3 ]))->align(lx::CENTER, lx::MIDDLE);
	$header->end();

	$info = new lx\Box([ 'key' => 'leadersTable', 'height' => 5 ]);
	$info->stream([ 'direction' => lx::VERTICAL ]);
$slot->end();

```

Теперь модифицируем код респондента, находящийся в файле `service/tetris/module/leaders/backend/Respondent.php`. Добавим метод, возвращающий список лидеров.
```php
namespace tetris\module\leaders\backend;

class Respondent extends \lx\Respondent {
	public function getLeaders() {
		$leaders = $this->getModelManager('TetrisLeader')->loadModels([
			'ORDER BY' => 'place ASC',
		]);

		$result = [];
		foreach ($leaders as $leader) {
			$result[] = $leader->getFields();
		}

		return $result;
	}
}

```

Задача JS-кода данного модуля - выводить списов лидеров. Код будет находиться в файле `services/tetris/module/leaders/frontend/_main.js`.<br>
С самим списком удобно работать в виде коллекции моделей. Есть очень лаконичный синтаксис, позволяющий создать коллекцию определенных моделей: `#lx:model-collection collectionName = ModelName;`. Таким образом мы создаем пустую коллекцию, в которую можно передавать данные для создания моделей.<br>
```js
#lx:model-collection leadersList = TetrisLeader;
```

Коллекцию свяжем с виджетом посредством матричного связывания. Для этого у виджетов есть метод `matrix`, который получает конфигурацию с ключами:
* `item` - коллекция, с которой связываемся
* `itemBox` - конфигурация для виджета, который будет представлять отдельный элемент коллекции
* `itemRender` - коллбэк, который будет вызываться при создании очередного виджета, представляющего отдельный элемент коллекции. Коллбэк в качестве аргументов получает сам виджет и соответствующую модель (в нашем случае модель нам здесь не нужна, мы ее не принимаем). В коде коллбэка предполагается описание как именно будет визуализирована модель. В нашем случае мы создаем поля формы, которые будут автоматически связываться с соответствующими полями моделей.
```js
Module->>leadersTable.matrix({
	items: leadersList,
	itemBox: [ lx.Form, {grid: true} ],
	itemRender: function(form) {
		form.fields({
			place: [lx.Box, {width: 1}],
			name:  [lx.Box, {width: 5}],
			score: [lx.Box, {width: 3}],
			level: [lx.Box, {width: 3}]
		});
		form.getChildren().each((child)=> child.align(lx.CENTER, lx.MIDDLE));
	}
});
```

Добавим функцию обновления коллекции моделей. Данные запрашиваем у респондента. Сразу вызовем эту функцию, чтобы при загрузке страницы таблица лидеров уже отобразилась.
```js
function loadLeaders() {
	^Respondent.getLeaders():(list)=>{
		leadersList.clear();
		list.each((leaderData)=>leadersList.add(leaderData));
	};
}
loadLeaders();
```

Модуль `game` при изменении таблицы лидеров генерирует событие `tetris_change_leaders`, мы можем подписаться на него, чтобы актуализировать представление таблицы лидеров.
```js
lx.EventSupervisor.subscribe('tetris_change_leaders', ()=>loadLeaders());
```

В итоге получаем код:
```js
#lx:model-collection leadersList = TetrisLeader;

Module->>leadersTable.matrix({
	items: leadersList,
	itemBox: [ lx.Form, {grid: true} ],
	itemRender: function(form) {
		form.fields({
			place: [lx.Box, {width: 1}],
			name:  [lx.Box, {width: 5}],
			score: [lx.Box, {width: 3}],
			level: [lx.Box, {width: 3}]
		});
		form.getChildren().each((child)=> child.align(lx.CENTER, lx.MIDDLE));
	}
});

function loadLeaders() {
	^Respondent.getLeaders():(list)=>{
		leadersList.clear();
		list.each((leaderData)=>leadersList.add(leaderData));
	};
}
loadLeaders();

lx.EventSupervisor.subscribe('tetris_change_leaders', ()=>loadLeaders());

```

Теперь мы можем проверить в браузере что получилось по URL `your.domain/tetris/leaders`.

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/12_common_module.md)
