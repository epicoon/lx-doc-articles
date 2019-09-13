[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/7_Timer.md)

### Шаг 7. Создание таймера

Таймер будет срабатывать при смене фреймов и реагировать на изменение времени. Создадим в каталоге `service/tetris/plugin/game/frontend/classes` файл `Timer.js`. Класс таймера отнаследуем от класса `lx.Timer` и поместим в пространство имен `tetris`:
```js
class Timer extends lx.Timer #lx:namespace tetris {
}
```

В конструктор передадим экземпляр игры. Флаг `counterOn` выставим в `false`, т.к. считать периоды в стиле секундомера нам не нужно. Длительность периода определим как 500 миллисекунд:
```js
	constructor(owner) {
		super();

		this.owner = owner;
		this.counterOn = false;
		this.periodDuration = 500;
	}
```

Переоределим родительский метод `iteration()`, который вызывается в конце каждого периода (в нашем случае каждые 500 миллисекунд). При этом будем вызывать метод `fallFigure()` экземпляра игры, который инициирует падение детали вниз:
```js
	iteration() {
		this.owner.fallFigure();
	}
```

В итоге должен получиться такой код:
```js
class Timer extends lx.Timer #lx:namespace tetris {
	constructor(owner) {
		super();

		this.owner = owner;
		this.counterOn = false;
		this.periodDuration = 500;
	}

	iteration() {
		this.owner.fallFigure();
	}
}

```

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/8_Game.md)
