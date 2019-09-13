[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/9_keyboard.md)

### Шаг 9. Оживление клавиатуры

Удобнее всего управлять игровым процессом при помощи клавиатуры. В каталоге `services/tetris/plugin/game/frontend` создадим файл `keyboard.js`.

Для разнообразия напишем данный код в процедурном стиле. Мы напишем функции, которые будут вызываться при наступлении событий нажатия на кнопки, поэтому сами функции снаружи будут не нужны, сделаем код приватным:
```js
#lx:private;
```

Теперь добавим обработчик события нашатия на клавишу, в зависимости от кода клавиши вызовем одну из функций, которые опишем ниже.
```js
lx.on('keydown', (event)=>{
	var game = tetris.Game.instance;
	if (!game.active || !game.activeFigure) return;

	var code = event.charCode ? event.charCode: event.keyCode;
	switch (+code) {
		case 40: toDown(); break;
		case 37: toLeft(); break;
		case 39: toRight(); break;
		case 38: nextState(); break;
		case 13: nextState(); break;

		case 81: game.toggleActive(); break;
	}
});
```

Смещение детали влево
```js
function toLeft() {
	tetris.Game.instance.moveFigure([-1, 0]);
}
```

Смещение детали вправо
```js
function toRight() {
	tetris.Game.instance.moveFigure([1, 0]);
}
```

Смещение детали вниз
```js
function toDown() {
	tetris.Game.instance.fallFigure();
}
```

Поворот детали. Предусмотрим проведение проверки на пересечение с препятствиями после поворота, если разрешить пересечение не удается, не позволим повернуть деталь (повернем обратно):
```js
function nextState() {
	var game = tetris.Game.instance,
		figure = game.activeFigure;

	game.map.clearFigure(figure);

	figure.nextState();
	if (!confirmFigureState())
		figure.prevState();

	game.map.highlightFigure(figure);
}
```

Проверка на пересечение с препятствиями после поворота с попыткой разрешить его путем отталкивания фигуры вправо или влево:
```js
function confirmFigureState() {
	return confirmDirection(-1) || confirmDirection(1);
}
```

Попытка разрешить пересечение фигуры с препятствием путем ее отталкивания в определенную сторону (либо вправо, либо влево):
```js
function confirmDirection(shift) {
	var game = tetris.Game.instance,
		figure = game.activeFigure,
		coords = [figure.coords[0], figure.coords[1]],
		attempts = 0;

	while (!game.map.getFreeCells(figure.mask(coords))) {
		coords[0] += shift;
		if (++attempts == game.map.colsCount) return false;
	}

	if (attempts) figure.coords = coords;
	return true;
}
```

В итоге должен получиться такой код:
```js
#lx:private;

lx.on('keydown', (event)=>{
	var game = tetris.Game.instance;
	if (!game.active || !game.activeFigure) return;

	var code = event.charCode ? event.charCode: event.keyCode;
	switch (+code) {
		case 40: toDown(); break;
		case 37: toLeft(); break;
		case 39: toRight(); break;
		case 38: nextState(); break;
		case 13: nextState(); break;

		case 81: game.toggleActive(); break;
	}
});

function toLeft() {
	tetris.Game.instance.moveFigure([-1, 0]);
}

function toRight() {
	tetris.Game.instance.moveFigure([1, 0]);
}

function toDown() {
	tetris.Game.instance.fallFigure();
}

function nextState() {
	var game = tetris.Game.instance,
		figure = game.activeFigure;

	game.map.clearFigure(figure);

	figure.nextState();
	if (!confirmFigureState())
		figure.prevState();

	game.map.highlightFigure(figure);
}

function confirmFigureState() {
	return confirmDirection(-1) || confirmDirection(1);
}

function confirmDirection(shift) {
	var game = tetris.Game.instance,
		figure = game.activeFigure,
		coords = [figure.coords[0], figure.coords[1]],
		attempts = 0;

	while (!game.map.getFreeCells(figure.mask(coords))) {
		coords[0] += shift;
		if (++attempts == game.map.colsCount) return false;
	}

	if (attempts) figure.coords = coords;
	return true;
}

```

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/10_game_complete.md)
