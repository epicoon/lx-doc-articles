[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/main.md)

# Пример создания приложения

Данный пример посвящен созданию простенькой игры - тетриса, но он демонстрирует базовые приемы и инструменты, предоставляемые платформой.

Если Вы еще не создали проект, нужно это сделать. Нужно поставить платформу с рекомендуемыми пакетами. [Инструкция](https://github.com/epicoon/lx-core/blob/master/README-ru.md#deploy).

Создадим сервис `tetris`, в нем создадим три плагина:
* `game` - здесь будем писать саму игру
* `leaders` - здесь напишем плагин для визуализации лучших результатов
* `main` - этот плагин объединит два предыдущих (чтобы показать насколько это легко)<br>
Как это сделать можно узнать по [ссылке](https://github.com/epicoon/lx-core/blob/master/README-ru.md#cli).

> Не забывайте, что сервис является пакетом, а согласно PSR-4 имя пакета долно состоять из имени производителя и собственно имени пакета: `vendorName/packageName`. Но для простоты в данном примере имя производителя мы не используем и называем сервис просто `tetris`.

Теперь у нас в проекте есть каталог `services/tetris`. Будем с ним работать.<br>
Шаги:
1. [Настройка роутинга](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/1_routing.md)
2. [Формирование представления игры](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/2_game_view.md)
3. [Описание фигур](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/3_figures.md)
4. [Создание модели TetrisLeader](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/4_model.md)
5. [Описание респондета для плагина "game"](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/5_game_respondent.md)
6. [Создание карты игровой поверхности](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/6_Map.md)
7. [Создание таймера](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/7_Timer.md)
8. [Создание класса игры](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/8_Game.md)
9. [Оживление клавиатуры](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/9_keyboard.md)
10. [Собираем все воедино. Играем](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/10_game_complete.md)
11. [Модуль для визуализации лидеров](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/11_leaders_plugin.md)
12. [Модуль, консолидирующий предыдущие](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/12_common_plugin.md)
