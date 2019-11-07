[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/6_Map.md)

### Шаг 6. Создание карты игровой поверхности

На текущем шаге и следующем будем создавать классы, которые помогут лучше структурировать код. Создадим для этих классов каталог `service/tetris/plugin/game/frontend/classes`.

Добавим класс `Map` в пространство имен `tetris`. Он будет ответственен за игровую поверхность. Займется подсвечиванием ячеек, поиском свободных ячеек для фигур, определением заполнения рядов. Для этого создадим в каталоге `service/tetris/plugin/game/frontend/classes` файл `Map.js`.

Объявим класс и конструктор. Конструктор в качестве аргумента будет принимать виджет, представляющий визуализацию игровой поверхности. Также пригодятся поля `colsCount`и `rowsCount`, чтобы лишний раз не лазить в виджет. Поле `rows` будет содержать массив, с помощью которого будем отслеживать заполнение рядов.
```js
class Map #lx:namespace tetris {
	constructor(map) {
		this.widget = map;

		this.colsCount = this.widget.colsCount();
		this.rowsCount = this.widget.rowsCount();
		this.rows = [];
	}
```

Добавляем метод перезапуска карты. Поскольку виджет это таблица (класс `lx.Table`), у него есть метод `cells()`, который возвращает коллекцию (класс `lx.Collection`) всех ячеек таблицы. А у коллекции есть метод `each(element, index)` для перебора всех элементов коллекции. Индекс элемента идет вторым аргументом, так как нужен далеко не всегда, нам в данном случае как раз не нужен.
```js
	reset() {
		this.rows = [];
		this.widget.cells().each((cell)=> this.clearCell(cell));
	}
```

Добавим метод, который по координатам проверит занята ли ячейка карты, и вернет булево значение.
```js
	cellIsFilled(row, col) {
		return (this.rows[row]
			&& this.rows[row].isArray
			&& this.rows[row].contains(col)
		);
	}
```

Следующий метод по координатам возвратит ячейку, если она свободна, либо `false`, если она занята или не существует (переданы координаты вне карты).
```js
	getFreeCell(row, col) {
		if (this.cellIsFilled(row, col)) return false;

		var cell = this.widget.cell(row, col);
		if (!cell) return false;

		return cell;
	}
```

Следующий метод постарается вернуть массив свободных ячеек согласно карте координат. Если хотя бы одна ячейка из требуемых занята, будет возвращен `null`.
```js
	getFreeCells(mask) {
		var cells = [];
		for (var i in mask) {
			var cell = this.getFreeCell(mask[i].y, mask[i].x);
			if (!cell) return null;
			cells.push(cell);
		}
		return cells;
	}
```

Следующий метод фиксирует карту координат - занимает ячейки, после чего необходимо вызвать проверку на заполнение рядов.
```js
	fix(mask) {
		for (var i in mask) {
			var item = mask[i];
			if (!this.rows[item.y])
				this.rows[item.y] = [];
			this.rows[item.y].push(item.x);
		}

		this.checkFilled();
	}
```

Теперь напишем метод проверки заполнения рядов. Для этого пройдемся по рядам снизу вверх, все заполненные очистим, все незаполненные опустим вниз. После этого отрапортуем сущности игры о количестве заполненных рядов. Сущность игры - экземпляр класса `tetris.Game` создадим позже, но уже сейчас напишем строчку `tetris.Game.instance.addLines(filled);`, подразумевая, что для простоты и удобства сделаем экземпляр игры один, и организуем к нему доступ через статическое поле `instance` (не называю это синглтоном, т.к. это им в чистом виде не является; платформа имеет встроенный класс `lx.Singletone`, истинный синглтон с невозможностью создать более одного экземпляра класса).
```js
	checkFilled() {
		var cols = this.widget.colsCount(),
			rows = this.widget.rowsCount(),
			filled = 0;

		for (var i=rows-1; i>=0; i--) {
			if (!this.rows[i]) break;

			if (this.rows[i].len == cols) {
				this.widget.row(i).cells().each((cell)=> this.clearCell(cell));
				this.rows[i] = null;
				filled++;
			} else if (filled) {
				var rowFrom = this.widget.row(i),
					rowTo = this.widget.row(i + filled);

				rowFrom.each((cellFrom)=> {
					if (!this.cellIsFilled(i, cellFrom.index)) return;
					var cellTo = rowTo.cell(cellFrom.index);
					this.highlightCell(cellTo, cellFrom.style('backgroundColor'));
					this.clearCell(cellFrom);
				});

				this.rows[i + filled] = this.rows[i];
				this.rows[i] = null;
			}
		}

		tetris.Game.instance.addLines(filled);
	}
```

Добавим метод очищения карты от маски детали:
```js
	clearFigure(figure) {
		var cells = this.getFreeCells(figure.mask());
		if (!cells) return;
		cells.each((cell)=> this.clearCell(cell));
	}
```

Добавим метод отрисовки на карте маски детали:
```js
	highlightFigure(figure) {
		var cells = this.getFreeCells(figure.mask());
		if (!cells) return;
		cells.each((cell)=> this.highlightCell(cell, figure.color));
	}
```

Добавим метод очищения отдельной ячейки карты:
```js
	clearCell(cell) {
		cell.fill('');
		cell.border({color: '', style: ''});
	}
```

Добавим метод отрисовки одного элемента детали в одной ячейке карты:
```js
	highlightCell(cell, color) {
		cell.fill(color);
		cell.border();
	}
```

В итоге должен получиться такой код:
```js
class Map #lx:namespace tetris {
	constructor(map) {
		this.widget = map;

		this.colsCount = this.widget.colsCount();
		this.rowsCount = this.widget.rowsCount();
		this.rows = [];
	}

	reset() {
		this.rows = [];
		this.widget.cells().each((cell)=> this.clearCell(cell));
	}

	cellIsFilled(row, col) {
		return (this.rows[row]
			&& this.rows[row].isArray
			&& this.rows[row].contains(col)
		);
	}

	getFreeCell(row, col) {
		if (this.cellIsFilled(row, col)) return false;

		var cell = this.widget.cell(row, col);
		if (!cell) return false;

		return cell;
	}

	getFreeCells(mask) {
		var cells = [];
		for (var i in mask) {
			var cell = this.getFreeCell(mask[i].y, mask[i].x);
			if (!cell) return null;
			cells.push(cell);
		}
		return cells;
	}

	fix(mask) {
		for (var i in mask) {
			var item = mask[i];
			if (!this.rows[item.y])
				this.rows[item.y] = [];
			this.rows[item.y].push(item.x);
		}

		this.checkFilled();
	}

	checkFilled() {
		var cols = this.widget.colsCount(),
			rows = this.widget.rowsCount(),
			filled = 0;

		for (var i=rows-1; i>=0; i--) {
			if (!this.rows[i]) break;

			if (this.rows[i].len == cols) {
				this.widget.row(i).cells().each((cell)=> this.clearCell(cell));
				this.rows[i] = null;
				filled++;
			} else if (filled) {
				var rowFrom = this.widget.row(i),
					rowTo = this.widget.row(i + filled);

				rowFrom.each((cellFrom)=> {
					if (!this.cellIsFilled(i, cellFrom.index)) return;
					var cellTo = rowTo.cell(cellFrom.index);
					this.highlightCell(cellTo, cellFrom.style('backgroundColor'));
					this.clearCell(cellFrom);
				});

				this.rows[i + filled] = this.rows[i];
				this.rows[i] = null;
			}
		}

		tetris.Game.instance.addLines(filled);
	}

	clearFigure(figure) {
		var cells = this.getFreeCells(figure.mask());
		if (!cells) return;
		cells.each((cell)=> this.clearCell(cell));
	}

	highlightFigure(figure) {
		var cells = this.getFreeCells(figure.mask());
		if (!cells) return;
		cells.each((cell)=> this.highlightCell(cell, figure.color));
	}

	clearCell(cell) {
		cell.fill('');
		cell.border({color: '', style: ''});
	}

	highlightCell(cell, color) {
		cell.fill(color);
		cell.border();
	}
}

```

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/7_Timer.md)
