[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/en/app-dev/expl1/1_routing.md)

### Шаг 1. Настройка роутинга

Настроим роутинг. Для этого откроем файл `services/tetris/lx-config.yaml` и отредактируем блок `router`:
```yaml
  router:
    type: map
    routes:
      /: {module: main}
      game: {module: game}
      leaders: {module: leaders}
```
Мы создали три роута, которые сервис направит непосредственно на модули. По этим роутам будет отдаваться результат рендеринга соответствующего модуля.<br>
Теперь нужно указать приложению как перенаправлять запросы на сервис. В файле `lx/config/routes.php` добавим:
```php
	'routes' => [
		// ... код

		'!tetris' => 'tetris',
	],
```
Теперь все запросы, путь которых начинается на `tetris` будут перенаправлены приложением на соответствующий сервис. Можем проверить - введем в браузере URL `your.domain/tetris` - если видим абсолютно пустую белую страницу, все хорошо. Если видим надпись "Page not found..." - что-то пошло не так, нужно еще раз проверить все ли сделано по инструкции.

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/ru/app-dev/expl1/2_game_view.md)
