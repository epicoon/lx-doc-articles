[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/en/app-dev/expl1/8_Game.md)

### Шаг 8. Создание класса игры

Настало время реализовать сердце логики - класс `tetris.Game. Не следует рассматривать код данного класса как образец архитектурного решения, т.к. как минимум две идеи, решаемые данным классом, можно вынести в отдельные классы. Но для упрощения примера, и с учетом того, что масштабирование в данном случае нам вряд ли понадобится, вполне преемлемо реализовать данный класс так, как предложено ниже.

Поскольку класс `tetris.Game` играет центральную роль в логике модуля, создадим для него файл `Game.js` рядом с файлом `_main.js` в каталоге `services/tetris/module/game/frontend`.

Начинаем писать код. Директива `#lx:private;` инкапсулирует код, находящийся в данном файле, в независимом контексте выполнения. Фактически, при сборке JS-кода, код из файлов с такой директивой оборачивается анонимной функцией, которая тут же выполняется. Соответственно, все переменные, объявленные в файле не будут доступны извне. Исключением являются классы, помещаемые в пространства имен - к ним доступ будет иметься через эти пространства имен.
```js
#lx:private;
```

Следующие две строчки - использование директивы `#lx:require`. Данная директива выполняет подключение кода по указанному адресу. В качестве адреса можно указать путь к файлу. Так как подключаются только файлы с расширением `.js`, расширение указывать не обязательно. Также путь может быть указан к директории (тогда путь должен заканчиваться символом `/`), тогда будут подключены все файлы с расширением `.js`, непосредственно находящиеся в директории (если же перед путем указать опцию `-R`, то файлы будут подключены рекурсивно и из вложенных директорий). При этом при указании пути можно использовать псевдонимы, можно указать путь относительно корня сайта (если путь начинается с символа `/`), можно указать путь относительно сервиса или модуля (примерно так: `#lx:require {service:some/serviceName}path/in/service`).
```js
#lx:require figures/;
#lx:require classes/;
```

Теперь объявим переменную, в которой будем хранить экземпляр игры. Помним, что сама эта переменная не будет доступна извне, поэтому изменить ее не удастся:
```js
let gameInstance = null;
```

Начинаем писать класс. Отнаследуем его от класса `lx.BindableModel`. Данный класс представляет из себя модель, которая может связываться с виджетами. Модели имеют схемы, в схемах описываются поля, которые могут отслеживаться виджетами. В качестве схемы зададим три поля, которые планируем отображать: `level` - текущий уровень игры, `filled` - количество заполненных рядов и `score` - текущее количество очков.
```js
class Game extends lx.BindableModel #lx:namespace tetris {
	#lx:schema level, filled, score;
}

```

В конструкторе создаем обычные поля (которые нельзя отслеживать виджетами). Нам понадобятся:
* `nextFigure` - экземпляр детали, которая будет следующей
* `activeFigure` - экземпляр текущей активной детали
* `timer` - экземпляр таймера
* `map` - экземпляр карты игровой поверхности
* `miniMap` - экземпляр карты для отображения следующей детали
* `active` - флаг активности игры, чтобы можно было ставить на паузу
Также запишем созданный экземпляр игры в переменную `gameInstance`, которую мы создали ранее, и которая недоступна извне:
```js
	constructor() {
		super();

		this.nextFigure = null;
		this.activeFigure = null;

		this.timer = new tetris.Timer(this);
		this.map = new tetris.Map(Module->>map);
		this.miniMap = new tetris.Map(Module->>next);

		this.active = false;

		gameInstance = this;
	}
```

Добавляем геттер для доступа к экземпляру игры
```js
	static get instance() {
		return gameInstance;
	}
```

Нужен метод запуска новой игры. Для этого сбрасываются все текущие данные, генерируется первая в новой игровой сессии деталь, сама игра активируется. Также обеспечим кликабельность кнопки паузы кодом `Module->>pause.disabled(false)`: переменная `Module` доступна во всем JS-коде модуля и представляет экземпляр модуля, оператор `->>` осуществляет поиск виджета, в данном случае с ключом `pause`.
```js
	newGame() {
		this.reset();
		this.genFigure();
		this.resume();
		Module->>pause.disabled(false);
	}
```

Метод окончания игры. Останавливаем таймер, деактивируем кнопку паузы. После чего отправляем AJAX-запрос, чтобы узнать - занял ли игрок какое-то место. Синтаксис обращения к респонденту с клиентской стороны: `^RespondentName.methodName(...arguments):(result)=>{/* some code with result */};`. Если достойное место игрок смог занять - открываем форму для ввода его имени, после чего будет отправлен еще один запрос - на обновление таблицы лидеров. При получении ответа сообщаем супервизору событий о том, что обновилать таблица лидеров, по ключу события его можно будет отслеживать.
```js
	over() {
		this.active = false;
		this.timer.stop();
		Module->>pause.disabled(true);

		^Respondent.checkLeaderPlace(this.score):(place)=>{
			if (place === false) return;

			Module->inputPopup.open('You took place ' + res + '. Enter your name', (name)=>{
				^Respondent.updateLeaders({
					name,
					score: this.score,
					level: this.level
				}):()=>{lx.EventSupervisor.trigger('tetris_change_leaders')};
				lx.Tost('You was saved to the hall of honor!');
			});
		};

		lx.Tost('Game over');
	}
```

Метод постановки игры на паузу
```js
	stop() {
		if (!this.active) return;
		this.active = false;
		this.timer.stop();
		Module->>pause.text('Resume');
	}
```

Метод снятия игры с паузы
```js
	resume() {
		if (this.active) return;
		this.active = true;
		this.timer.start();
		Module->>pause.text('Pause');
	}
```

Метод переключения состояния активности игры
```js
	toggleActivity() {
		if (this.active) this.stop();
		else this.resume();
	}
```

Метод для сбрасывания всех данных (подготовка к очередной игровой сессии)
```js
	reset() {
		if (this.nextFigure) this.miniMap.clearFigure(this.nextFigure);
		this.activeFigure = null;
		this.nextFigure = null;

		this.level = 1;
		this.setSpeed();
		this.filled = 0;
		this.score = 0;
		this.map.reset();
	}
```

Метод создания новой фигуры. Сразу заготавливаем фигуру, которая будет следующей и отображаем ее в миникарте. Новую фигуру, которую вводим в игру нужно сместить в середину по ширине.
```js
	genFigure() {
		if (this.nextFigure) this.miniMap.clearFigure(this.nextFigure);
		else this.nextFigure = tetris.Figure.getRand();

		this.activeFigure = this.nextFigure;
		this.nextFigure = tetris.Figure.getRand();
		this.miniMap.highlightFigure(this.nextFigure);

		if (!this.moveFigure([(this.map.colsCount - 4) * 0.5, 0]))
			this.fixFigure();
	}
```

Метод смещения фигуры. Возвращает булево значение - удалось ли фигуру переместить.
```js
	moveFigure(shift) {
		var figure = this.activeFigure;
		var coords = [
			figure.coords[0] + shift[0],
			figure.coords[1] + shift[1]
		];

		var cells = this.map.getFreeCells(figure.mask(coords));
		if (!cells) return false;

		this.map.clearFigure(figure);
		figure.coords = coords;
		this.map.highlightFigure(figure);

		return true;
	}
```

Метод падения фигуры вниз. Если смещение невозможно - значит фигура во что-то уперлась и дальше падать не может.
```js
	fallFigure() {
		if (!this.moveFigure([0, 1]))
			this.fixFigure();		
	}
```

Метод фиксации фигуры, после того, как она уперлась после падения. Если свободных ячеек не хватает - это проигрыш. Иначе зафиксируем фигуру на карте и запустим логику создания следующей фигуры.
```js
	fixFigure() {
		var mask = this.activeFigure.mask();

		if (!this.map.getFreeCells(mask)) {
			this.map.highlightFigure(this.activeFigure);
			this.over();
			return;
		}

		this.map.fix(mask);
		this.genFigure();
	}
```

Метод увеличения счетчика заполненных рядов. Тут начисляются очки - за одновременное заполнение нескольких рядов очков будет больше. За каждые 10 заполненных рядов повышаем уровень игры.
```js
	addFilled(filled) {
		this.filled += filled;

		var score = 0;
		switch (filled) {
			case 1: score = 1; break;
			case 2: score = 3; break;
			case 3: score = 6; break;
			case 4: score = 10; break;
		}

		this.score += score;

		var level = Math.floor(this.filled * 0.1) + 1;
		if (level != this.level) this.upLevel();
	}
```

Метод для повышения уровня игры. Актуализирует скорость игры согласно новому уровню.
```js
	upLevel() {
		this.level++;
		this.setSpeed();
	}
```

Метод для управления скоростью игры. С ростом уровня уменьшаем поле таймера `periodDuration`.
```js
	setSpeed() {
		this.timer.periodDuration = Math.floor(1000 / (this.level + 1));
	}
```

В итоге должен получиться такой код:
```js
#lx:private;
#lx:require figures/;
#lx:require classes/;

let gameInstance = null;

class Game extends lx.BindableModel #lx:namespace tetris {
	#lx:schema level, filled, score;

	constructor() {
		super();

		this.nextFigure = null;
		this.activeFigure = null;

		this.timer = new tetris.Timer(this);
		this.map = new tetris.Map(Module->>map);
		this.miniMap = new tetris.Map(Module->>next);

		this.active = false;

		gameInstance = this;
	}

	static get instance() {
		return gameInstance;
	}

	newGame() {
		this.reset();
		this.genFigure();
		this.resume();
		Module->>pause.disabled(false);
	}

	over() {
		this.active = false;
		this.timer.stop();
		Module->>pause.disabled(true);

		^Respondent.checkLeaderPlace(this.score):(place)=>{
			if (place === false) return;

			Module->inputPopup.open('You took place ' + res + '. Enter your name', (name)=>{
				^Respondent.updateLeaders({
					name,
					score: this.score,
					level: this.level
				}):()=>{lx.EventSupervisor.trigger('tetris_change_leaders')};
				lx.Tost('You was saved to the hall of honor!');
			});
		};

		lx.Tost('Game over');
	}

	stop() {
		if (!this.active) return;
		this.active = false;
		this.timer.stop();
		Module->>pause.text('Resume');
	}

	resume() {
		if (this.active) return;
		this.active = true;
		this.timer.start();
		Module->>pause.text('Pause');
	}

	toggleActivity() {
		if (this.active) this.stop();
		else this.resume();
	}

	reset() {
		if (this.nextFigure) this.miniMap.clearFigure(this.nextFigure);
		this.activeFigure = null;
		this.nextFigure = null;

		this.level = 1;
		this.setSpeed();
		this.filled = 0;
		this.score = 0;
		this.map.reset();
	}

	genFigure() {
		if (this.nextFigure) this.miniMap.clearFigure(this.nextFigure);
		else this.nextFigure = tetris.Figure.getRand();

		this.activeFigure = this.nextFigure;
		this.nextFigure = tetris.Figure.getRand();
		this.miniMap.highlightFigure(this.nextFigure);

		if (!this.moveFigure([(this.map.colsCount - 4) * 0.5, 0]))
			this.fixFigure();
	}

	moveFigure(shift) {
		var figure = this.activeFigure;
		var coords = [
			figure.coords[0] + shift[0],
			figure.coords[1] + shift[1]
		];

		var cells = this.map.getFreeCells(figure.mask(coords));
		if (!cells) return false;

		this.map.clearFigure(figure);
		figure.coords = coords;
		this.map.highlightFigure(figure);

		return true;
	}

	fallFigure() {
		if (!this.moveFigure([0, 1]))
			this.fixFigure();
	}

	fixFigure() {
		var mask = this.activeFigure.mask();

		if (!this.map.getFreeCells(mask)) {
			this.map.highlightFigure(this.activeFigure);
			this.over();
			return;
		}

		this.map.fix(mask);
		this.genFigure();
	}

	addFilled(filled) {
		this.filled += filled;

		var score = 0;
		switch (filled) {
			case 1: score = 1; break;
			case 2: score = 3; break;
			case 3: score = 6; break;
			case 4: score = 10; break;
		}

		this.score += score;

		var level = Math.floor(this.filled * 0.1) + 1;
		if (level != this.level) this.upLevel();
	}

	upLevel() {
		this.level++;
		this.setSpeed();
	}

	setSpeed() {
		this.timer.periodDuration = Math.floor(1000 / (this.level + 1));
	}
}

```

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/ru/app-dev/expl1/9_keyboard.md)
