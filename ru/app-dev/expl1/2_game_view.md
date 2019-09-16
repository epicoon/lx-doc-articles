[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/2_game_view.md)

### Шаг 2. Формирование представления игры

Будем писать код в файле `services/tetris/plugin/game/snippet/_root.js`. Данный файл уже содержит код:
```js
/**
 * @const lx.Application App
 * @const lx.Plugin Plugin
 * @const lx.Snippet Snippet
 * */

```
В данном файле доступно три контекстные переменные:
* App - экземпляр приложения
* Plugin - экземпляр плагина, к которому относится код данного сниппета. В нашем случае это плагин `tetris:game`.
* Snippet - экземпляр самого сниппета. Это обертка над виртуальным аватаром виджета (сниппет имеет поле `widget`, являющееся экземпляром класса `lx.Box`), в который будет произведен рендеринг кода, написанного в данном файле.

Подключим модули с необходимыми нам виджетами.
```js
#lx:use lx.Button;
#lx:use lx.Table;
```

Сразу определяем размеры поля. Тетрис играется на поле 10x20.
```js
let cols = 10;
let rows = 20;
```

Добавим виджет, в который поместим всё связанное с игрой. Зальем его белым фоном. Параметр `key` - важен для нахождения виджета на стороне клиента. Экзепляр виджета будет воссоздан на стороне клиента. Создавая данный виджет на серверной стороне с ключом `tetris`, на клиентской стороне мы сможем получить соответствующий ему объект класса `lx.Box`, найдя его при помощи кода `Plugin->>tetris`. Зададим значение `geom: true`, чтобы виджет занял всё доступное пространство и по вертикали, и по горизонтали. Также определим этому виджету стратегию позиционирования `slot` с единственным слотом и коэффициентом 0.7, чтобы соотношение высоты и ширины было постоянное. Также для аккуратности укажем отступ '10px'. Применение данной стратегии позиционирования сразу создаст один слот. Этот слот используем в качестве формы для всего необходимого. 

```js
let tetris = new lx.Box({key: 'tetris', geom: true});
tetris.fill('white');
tetris.slot({
	indent: '10px',
	cols: 1,
	rows: 1,
	k: 0.7
});
```

Определим стратегию позиционирования в слоте `gridProportional` - чтобы все дочерние элементы располагались согласно сетке, зададим размеры сетки и отступ. Методом `begin()` "открываем" форму - она станет родительским элементом для всех вновь создаваемых элементов.
```js
let slot = tetris.child(0);
slot.gridProportional({
	rows: rows,
	cols: cols + 4,
	indent: '10px'
});

slot.begin();
```

Создадим игровое поле. Для этого просто добавим таблицу. Укажем размер, который она займет в сетке, и количество строк и колонок.
```js
	new lx.Table({
		key: 'map',
		size: [cols, rows],
		cols: cols,
		rows: rows
	});
```

Добавляем кнопку запуска новой игры.
```js
	new lx.Button({
		key: 'newGame',
		size: [4, 2],
		text: 'New game'
	});
```

Добавляем небольшое поле, которое будет показывать следующую фигуру.
```js
	(new lx.Box({width:4, text:'Next'})).align(lx.CENTER, lx.MIDDLE);
	new lx.Table({
		key: 'next',
		size: [4, 4],
		rows: 4,
		cols: 4
	});
```

Добавим кнопку постановки игры на паузу.
```js
	new lx.Button({
		key: 'pause',
		size: [4, 2],
		text: 'Pause'
	});
```

Добавим поле, где будет выводиться текущий уровень игры. Параметр `field` работает как `key` (если `key` не указан явно, но как правило в этом нет необходимости), плюс сообщает данному виджету, что он будет ослеживать поле с соответствующим названием (в нашем случае `level`) у модели, с которой виджет будет связан.
```js
	(new lx.Box({width:4, text:'Level:'})).align(lx.CENTER, lx.MIDDLE);
	(new lx.Box({
		field: 'level',
		size: [4, 2],
		style: {fill:'lightgray'}
	})).align(lx.CENTER, lx.MIDDLE);
```

Аналогично предыдущему, добавим поле, где будет выводиться количество убранных полос.
```js
	(new lx.Box({width:4, text:'Lines:'})).align(lx.CENTER, lx.MIDDLE);
	(new lx.Box({
		field: 'lines',
		size: [4, 2],
		style: {fill:'lightgray'}
	})).align(lx.CENTER, lx.MIDDLE);
```

Аналогично предыдущему, добавим поле, где будет выводиться количество заработанных очков.
```js
	(new lx.Box({width:4, text:'Score:'})).align(lx.CENTER, lx.MIDDLE);
	(new lx.Box({
		field: 'score',
		size: [4, 2],
		style: {fill:'lightgray'}
	})).align(lx.CENTER, lx.MIDDLE);
```

Все необходимые поля созданы, закрываем форму.
```js
slot.end();
```

Используем полезную форму для ввода значений, оформленную как сниппет. Сниппет реализован в сервисе `lx/lx-tools`, Вы ведь установили его?
```js
Snippet.addSnippet({
	plugin: 'lx/lx-tools:snippets',
	snippet: 'inputPopup'
});
```

И так, получился такой код представления:
```js
/**
 * @const lx.Application App
 * @const lx.Plugin Plugin
 * @const lx.Snippet Snippet
 * */

#lx:use lx.Button;
#lx:use lx.Table;

let cols = 10;
let rows = 20;

let tetris = new lx.Box({key:'tetris', geom:true});
tetris.fill('white');
tetris.slot({
	indent: '10px',
	cols: 1,
	rows: 1,
	k: 0.7
});

let slot = tetris.child(0);
slot.gridProportional({
	rows: rows,
	cols: cols + 4,
	indent: '10px'
});
slot.begin();
	new lx.Table({
		key: 'map',
		size: [cols, rows],
		cols: cols,
		rows: rows
	});

	new lx.Button({
		key: 'newGame',
		size: [4, 2],
		text: 'New game'
	});

	(new lx.Box({width:4, text:'Next'})).align(lx.CENTER, lx.MIDDLE);
	new lx.Table({
		key: 'next',
		size: [4, 4],
		rows: 4,
		cols: 4
	});

	new lx.Button({
		key: 'pause',
		size: [4, 2],
		text: 'Pause'
	});

	(new lx.Box({width:4, text:'Level:'})).align(lx.CENTER, lx.MIDDLE);
	(new lx.Box({
		field: 'level',
		size: [4, 2],
		style: {fill:'lightgray'}
	})).align(lx.CENTER, lx.MIDDLE);

	(new lx.Box({width:4, text:'Lines:'})).align(lx.CENTER, lx.MIDDLE);
	(new lx.Box({
		field: 'lines',
		size: [4, 2],
		style: {fill:'lightgray'}
	})).align(lx.CENTER, lx.MIDDLE);

	(new lx.Box({width:4, text:'Score:'})).align(lx.CENTER, lx.MIDDLE);
	(new lx.Box({
		field: 'score',
		size: [4, 2],
		style: {fill:'lightgray'}
	})).align(lx.CENTER, lx.MIDDLE);
slot.end();

Snippet.addSnippet({
	plugin: 'lx/lx-tools:snippets',
	snippet: 'inputPopup'
});

```
Проверим что у нас получилось, введем в браузере URL `your.domain/tetris/game`.

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/3_figures.md)
