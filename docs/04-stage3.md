# 4. Этап 3 — Интерактивность <span class="points-badge">5 баллов</span>

## Цель этапа

Сделать приложение **интерактивным**: пользователь может добавлять и удалять процессы, сортировать таблицу и фильтровать по зонам.

---

## 3.1 Форма добавления процесса

Замените содержимое секции `#add-form` в `index.html`:

```html
<section id="add-form">
  <h2>Добавить процесс</h2>
  <form id="process-form">
    <div class="form-row">
      <label>Название:
        <input type="text" id="inp-name" required placeholder="Название процесса">
      </label>
      <label>Тип:
        <select id="inp-type">
          <option value="Основной">Основной</option>
          <option value="Поддерживающий">Поддерживающий</option>
          <option value="Управленческий">Управленческий</option>
        </select>
      </label>
    </div>

    <fieldset>
      <legend>Операционные критерии (1–5)</legend>
      <div class="form-row">
        <label>Время цикла: <input type="number" id="inp-cycleTime" min="1" max="5" value="3"></label>
        <label>Стоимость: <input type="number" id="inp-cost" min="1" max="5" value="3"></label>
        <label>Автоматизация: <input type="number" id="inp-automation" min="1" max="5" value="3"></label>
        <label>Доля переделок: <input type="number" id="inp-rework" min="1" max="5" value="3"></label>
      </div>
    </fieldset>

    <fieldset>
      <legend>Стратегические критерии (1–5)</legend>
      <div class="form-row">
        <label>Стратег. значимость: <input type="number" id="inp-strategicValue" min="1" max="5" value="3"></label>
        <label>Критичность: <input type="number" id="inp-criticality" min="1" max="5" value="3"></label>
        <label>Инновац. потенциал: <input type="number" id="inp-innovation" min="1" max="5" value="3"></label>
        <label>Техдолг: <input type="number" id="inp-techDebt" min="1" max="5" value="3"></label>
      </div>
    </fieldset>

    <button type="submit" class="btn-add">Добавить</button>
  </form>
</section>
```

Добавьте стили для формы в `style.css`:

```css
/* --- Форма --- */
.form-row {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 12px;
}

.form-row label {
  display: flex;
  flex-direction: column;
  font-size: 0.85rem;
  color: #555;
}

.form-row input,
.form-row select {
  padding: 6px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 0.9rem;
  margin-top: 4px;
}

fieldset {
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  padding: 12px 16px;
  margin-bottom: 12px;
}

legend {
  font-weight: 600;
  color: #1a237e;
  font-size: 0.9rem;
}
```

---

## 3.2 Обработчик формы в JS

Добавьте в `app.js`:

```javascript
// ========== ДОБАВЛЕНИЕ ПРОЦЕССА ==========

function handleAddProcess(event) {
  event.preventDefault(); // Предотвращаем перезагрузку страницы

  // Генерируем уникальный id
  var maxId = 0;
  processes.forEach(function(p) {
    if (p.id > maxId) maxId = p.id;
  });
  var newId = maxId + 1;

  // Собираем данные из формы
  var newProcess = {
    id: newId,
    name: document.getElementById('inp-name').value,
    type: document.getElementById('inp-type').value,
    cycleTime:       Number(document.getElementById('inp-cycleTime').value),
    cost:            Number(document.getElementById('inp-cost').value),
    automation:      Number(document.getElementById('inp-automation').value),
    rework:          Number(document.getElementById('inp-rework').value),
    strategicValue:  Number(document.getElementById('inp-strategicValue').value),
    criticality:     Number(document.getElementById('inp-criticality').value),
    innovation:      Number(document.getElementById('inp-innovation').value),
    techDebt:        Number(document.getElementById('inp-techDebt').value)
  };

  // Добавляем в массив
  processes.push(newProcess);

  // Обновляем отображение
  renderAll();

  // Очищаем форму
  event.target.reset();
}
```

!!! concept "event.preventDefault() — зачем?"

    По умолчанию при отправке формы (`submit`) браузер **перезагружает страницу**. Метод `preventDefault()` отменяет это поведение, позволяя обработать данные в JavaScript без перезагрузки.

---

## 3.3 Удаление процесса

Добавьте обработчик кликов по кнопкам «Удалить»:

```javascript
// ========== УДАЛЕНИЕ ПРОЦЕССА ==========

function handleDelete(event) {
  // Проверяем, что клик был именно по кнопке удаления
  if (event.target.classList.contains('btn-delete')) {
    var id = Number(event.target.getAttribute('data-id'));

    // Удаляем процесс из массива
    processes = processes.filter(function(p) {
      return p.id !== id;
    });

    renderAll();
  }
}
```

!!! concept "Делегирование событий"

    Мы вешаем обработчик **не на каждую кнопку**, а на родительский элемент (`<tbody>`). Когда кнопка внутри таблицы нажата, событие «всплывает» до `<tbody>`, и мы проверяем `event.target` — была ли это кнопка удаления. Это называется **делегирование событий** (event delegation).

    Преимущество: обработчик работает даже для **динамически добавленных** строк.

---

## 3.4 Сортировка по столбцам

```javascript
// ========== СОРТИРОВКА ==========

var currentSort = { field: null, ascending: true };

function sortProcesses(field) {
  // Если кликнули по тому же столбцу — меняем направление
  if (currentSort.field === field) {
    currentSort.ascending = !currentSort.ascending;
  } else {
    currentSort.field = field;
    currentSort.ascending = true;
  }

  processes.sort(function(a, b) {
    var valA, valB;

    if (field === 'name' || field === 'type') {
      valA = a[field];
      valB = b[field];
      if (valA < valB) return currentSort.ascending ? -1 : 1;
      if (valA > valB) return currentSort.ascending ? 1 : -1;
      return 0;
    }

    if (field === 'operational') {
      valA = calcOperational(a);
      valB = calcOperational(b);
    } else if (field === 'strategic') {
      valA = calcStrategic(a);
      valB = calcStrategic(b);
    } else if (field === 'zone') {
      valA = getZone(a);
      valB = getZone(b);
    }

    return currentSort.ascending ? valA - valB : valB - valA;
  });

  renderAll();
}
```

Обновите заголовки таблицы в `index.html`, чтобы они были кликабельными:

```html
<thead>
  <tr>
    <th onclick="sortProcesses('name')">Название</th>
    <th onclick="sortProcesses('type')">Тип</th>
    <th onclick="sortProcesses('operational')">Опер. эфф.</th>
    <th onclick="sortProcesses('strategic')">Стратег. важн.</th>
    <th onclick="sortProcesses('zone')">Зона</th>
    <th>Действия</th>
  </tr>
</thead>
```

---

## 3.5 Фильтрация по зонам

Добавьте в секцию `#controls` в HTML:

```html
<section id="controls">
  <label>Фильтр по зоне:
    <select id="zone-filter" onchange="filterByZone(this.value)">
      <option value="all">Все зоны</option>
      <option value="1">Зона 1 — Срочные изменения</option>
      <option value="2">Зона 2 — Развитие</option>
      <option value="3">Зона 3 — Стандартизация</option>
      <option value="4">Зона 4 — Мониторинг</option>
    </select>
  </label>
</section>
```

В `app.js`:

```javascript
// ========== ФИЛЬТРАЦИЯ ==========

var activeFilter = 'all';

function filterByZone(zoneValue) {
  activeFilter = zoneValue;
  renderAll();
}
```

Теперь модифицируйте `renderTable` и `renderMatrix`, чтобы они учитывали фильтр:

```javascript
function getFilteredProcesses() {
  if (activeFilter === 'all') return processes;
  return processes.filter(function(p) {
    return getZone(p) === Number(activeFilter);
  });
}
```

!!! task "Самостоятельная работа"

    Замените в `renderTable` и `renderMatrix` прямое обращение к `processes` на вызов `getFilteredProcesses()`. Например:

    ```javascript
    // Было:
    processes.forEach(function(process) { ... });

    // Стало:
    getFilteredProcesses().forEach(function(process) { ... });
    ```

    Это нужно сделать **в обеих функциях**.

---

## 3.6 Подключение обработчиков

Обновите секцию запуска в конце `app.js`:

```javascript
// ========== ЗАПУСК ==========
document.addEventListener('DOMContentLoaded', function() {
  // Подключаем форму добавления
  document.getElementById('process-form')
    .addEventListener('submit', handleAddProcess);

  // Подключаем удаление через делегирование
  document.getElementById('table-body')
    .addEventListener('click', handleDelete);

  // Первичная отрисовка
  renderAll();
});
```

---

## Чек-лист этапа 3

| # | Что проверить | Баллы |
|---|---|---|
| 3.1 | Форма добавления работает: процесс появляется в таблице и матрице | 2 |
| 3.2 | Кнопка «Удалить» удаляет процесс из таблицы и матрицы | 1 |
| 3.3 | Сортировка по столбцам работает (клик по заголовку) | 1 |
| 3.4 | Фильтр по зонам скрывает процессы из других зон | 1 |
| | **Итого за этап 3** | **5** |

!!! hint "Частые ошибки"

    - **Форма перезагружает страницу** → забыли `event.preventDefault()`
    - **Удаление не работает** → обработчик повешен до создания элементов (используйте делегирование)
    - **Сортировка сбрасывается при добавлении** → это нормально, если `renderAll` вызывается без повторной сортировки
    - **Фильтр не влияет на матрицу** → забыли заменить `processes` на `getFilteredProcesses()` в `renderMatrix`
