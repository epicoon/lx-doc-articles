[Английская версия (English version)](https://github.com/epicoon/lx-doc-articles/blob/master/en/app-dev/expl1/5_game_respondent.md)

### Шаг 5. Описание респондета для плагина "game"

Модель для сохранения лучших результатов у нас уже есть. Нужно предусмотреть механизм непосредственно сохранения. Для этого будет использоваться способность плагина общаться со своей серверной стороной посредством AJAX-запросов. Ответы формирует респондент плагина. У нас один уже есть, он был автоматически сгенерирован при создании плагина, его и будем использовать. Это класс, расположенный в файле `service/tetris/plugin/game/backend/Respondent.php`, там Вы увидите такой код:
```php
namespace tetris\plugin\game\backend;

class Respondent extends \lx\Respondent {

}

```
Нам нужно будет добавить несколько методов.

Данный метод будет отвечать на вопрос - заслуживает ли лидерской позиции переданное количество заработанных очков. Саму логику определения занятого места вынесем в отдельный метод, который реализуем позже:
```php
	public function checkLeaderPlace($score) {
		return $this->checkLeaderPlaceProcess($score);
	}
```

Следующий метод будет обновлять таблицу лидеров. Логика простая - всех, кто имеет меньше очков, чем добавляемый, спускаем на один пункт. Самого последнего перезатираем. На освободившееся место записываем нового лидера. После этого сохраняем обновившиеся данные.
```php
	public function updateLeaders($data) {
		$place = $this->checkLeaderPlaceProcess($data['score']);
		if (!$place) {
			return;
		}

		$leaders = $this->loadLeaders();
		for ($i = 4; $i >= $place; $i--) {
			$prevLeaderFields = $leaders[$i - 1]->getFields(['name', 'level', 'score']);
			$leaders[$i]->setFields($prevLeaderFields);			
		}

		$leaders[$place - 1]->setFields($data);

		$this->getModelManager('TetrisLeader')->saveModels($leaders);
	}
```

Мы использовали метод `checkLeaderPlaceProcess`, в который предполагается вынесение логики определения занятого места, реализуем его:
```php
	private function checkLeaderPlaceProcess($score) {
		$leaders = $this->loadLeaders();
		foreach ($leaders as $leader) {
			if ($score > $leader->score) {
				return $leader->place;
			}
		}

		return false;
	}
```

Также мы использовали метод получения текущих лидеров `loadLeaders`, реализуем и его:
```php
	private function loadLeaders() {
		return $this->getModelManager('TetrisLeader')->loadModels([
			'ORDER BY' => 'place ASC',
		]);
	}
```

На данном этапе больше ничего не нужно. Идем дальше.

[Следующий шаг](https://github.com/epicoon/lx-doc-articles/blob/master/ru/app-dev/expl1/6_Map.md)
