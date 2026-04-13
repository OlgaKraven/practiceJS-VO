# 3. Этап 2 — Рендеринг и расчёты <span class="points-badge">6 баллов</span>

## Цель этапа

Научиться **динамически генерировать HTML** из массива данных через JavaScript. После этого этапа таблица и матрица будут отображаться автоматически.

---

## 2.1 Рендеринг таблицы

Добавьте в `app.js` функцию, которая генерирует строки таблицы:

```javascript
// ========== РЕНДЕРИНГ ==========

function renderTable() {
  const tbody = document.getElementById('table-body');
  tbody.innerHTML = ''; // Очищаем перед перерисовкой

  processes.forEach(function(process) {
    const op = calcOperational(process).toFixed(2);
    const st = calcStrategic(process).toFixed(2);
    const zone = getZone(process);

    const row = document.createElement('tr');
    row.innerHTML =
      '<td>' + process.name + '</td>' +
      '<td>' + process.type + '</td>' +
      '<td>' + op + '</td>' +
      '<td>' + st + '</td>' +
      '<td>Зона ' + zone + '</td>' +
      '<td><button class="btn-delete" data-id="' + process.id + '">Удалить</button></td>';

    tbody.appendChild(row);
  });
}
```

!!! concept "Что здесь происходит?"

    1. `getElementById('table-body')` — находим элемент `<tbody>` в HTML
    2. `innerHTML = ''` — очищаем содержимое (чтобы при повторном вызове не дублировались строки)
    3. `forEach` — проходим по каждому процессу в массиве
    4. `createElement('tr')` — создаём новую строку таблицы
    5. `innerHTML` строки — заполняем ячейки данными
    6. `appendChild` — добавляем строку в таблицу

---

## 2.2 Рендеринг матрицы

Теперь функция для отрисовки карточек в зонах матрицы:

```javascript
function renderMatrix() {
  // Очищаем все зоны (оставляем только заголовки)
  for (var i = 1; i <= 4; i++) {
    var zoneEl = document.getElementById('zone-' + i);
    // Удаляем все карточки, но оставляем <h3>
    var chips = zoneEl.querySelectorAll('.process-chip');
    chips.forEach(function(chip) { chip.remove(); });
  }

  // Распределяем процессы по зонам
  processes.forEach(function(process) {
    var zone = getZone(process);
    var zoneEl = document.getElementById('zone-' + zone);

    var chip = document.createElement('div');
    chip.className = 'process-chip';
    chip.textContent = process.name;
    zoneEl.appendChild(chip);
  });
}
```

---

## 2.3 Общая функция перерисовки

Создайте одну функцию, которая обновляет **всё**:

```javascript
function renderAll() {
  renderTable();
  renderMatrix();
}
```

---

## 2.4 Запуск при загрузке страницы

В конце файла `app.js` добавьте:

```javascript
// ========== ЗАПУСК ==========
document.addEventListener('DOMContentLoaded', function() {
  renderAll();
});
```

!!! concept "Зачем DOMContentLoaded?"

    Скрипт подключён в конце `<body>`, поэтому DOM уже загружен. Но `DOMContentLoaded` — **хорошая практика**: он гарантирует, что все элементы доступны, даже если скрипт случайно переместят в `<head>`.

---

## 2.5 Что должно отображаться

После этого этапа при открытии `index.html` вы должны увидеть:

**Таблица:**

| Название | Тип | Опер. эфф. | Стратег. важн. | Зона |
|---|---|---|---|---|
| Приём и обработка заявки | Основной | 2.75 | 4.00 | Зона 1 |
| Планирование маршрутов | Основной | 2.25 | 4.00 | Зона 1 |
| Исполнение рейса | Основной | 2.75 | 3.75 | Зона 1 |
| Расчёт стоимости и счета | Поддерживающий | 2.50 | 3.00 | Зона 1 |
| Управление парком (ТО) | Поддерживающий | 3.25 | 3.25 | Зона 2 |
| Работа с претензиями | Управленческий | 2.25 | 3.00 | Зона 1 |

**Матрица:** В красной зоне (Зона 1) — 5 процессов. В зелёной (Зона 2) — 1 процесс (Управление парком).

!!! insight "Проверьте себя"

    Если в таблице отображаются все 6 строк, а в матрице карточки распределены по зонам — этап 2 выполнен. Если строки не появляются — откройте **Console** в DevTools (`F12`) и посмотрите ошибки.

---

## 2.6 Счётчик процессов в зонах (бонус к оформлению)

Добавьте в функцию `renderMatrix` отображение количества процессов в каждой зоне:

```javascript
// Внутри renderMatrix(), после распределения процессов:
for (var i = 1; i <= 4; i++) {
  var zoneEl = document.getElementById('zone-' + i);
  var count = zoneEl.querySelectorAll('.process-chip').length;
  var h3 = zoneEl.querySelector('h3');

  // Названия зон
  var zoneNames = {
    1: 'Срочные изменения',
    2: 'Развитие',
    3: 'Стандартизация',
    4: 'Мониторинг'
  };

  h3.textContent = zoneNames[i] + ' (' + count + ')';
}
```

---

## Чек-лист этапа 2

| # | Что проверить | Баллы |
|---|---|---|
| 2.1 | Таблица отображает все 6 процессов с корректными средними оценками | 2 |
| 2.2 | Матрица показывает карточки процессов в правильных зонах | 2 |
| 2.3 | При изменении данных в массиве и вызове `renderAll()` отображение обновляется | 1 |
| 2.4 | Нет ошибок в консоли браузера | 1 |
| | **Итого за этап 2** | **6** |

!!! sandbox "Проверка через консоль"

    Откройте консоль браузера и выполните:

    ```javascript
    // Добавим тестовый процесс
    processes.push({
      id: 99, name: "Тест", type: "Тестовый",
      cycleTime: 5, cost: 5, automation: 5, rework: 5,
      strategicValue: 1, criticality: 1, innovation: 1, techDebt: 1
    });
    renderAll();
    // Должна появиться новая строка в таблице
    // и карточка "Тест" в Зоне 4 (Мониторинг)
    ```

    После проверки удалите тестовый процесс:

    ```javascript
    processes = processes.filter(function(p) { return p.id !== 99; });
    renderAll();
    ```
