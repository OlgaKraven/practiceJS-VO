# 2. Этап 1 — Разметка и данные <span class="points-badge">5 баллов</span>

## Цель этапа

Создать HTML-страницу с базовой структурой и подготовить данные в JavaScript.

---

## 1.1 HTML-разметка

Создайте файл `index.html` со следующей структурой:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Матрица приоритизации БП</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>

  <header>
    <h1>Матрица приоритизации бизнес-процессов</h1>
  </header>

  <main>
    <!-- Секция: Форма добавления -->
    <section id="add-form">
      <h2>Добавить процесс</h2>
      <!-- Форма будет здесь -->
    </section>

    <!-- Секция: Управление -->
    <section id="controls">
      <!-- Фильтры и сортировка -->
    </section>

    <!-- Секция: Таблица процессов -->
    <section id="process-table">
      <h2>Список процессов</h2>
      <table>
        <thead>
          <tr>
            <th>Название</th>
            <th>Тип</th>
            <th>Опер. эфф.</th>
            <th>Стратег. важн.</th>
            <th>Зона</th>
            <th>Действия</th>
          </tr>
        </thead>
        <tbody id="table-body">
          <!-- Строки генерируются через JS -->
        </tbody>
      </table>
    </section>

    <!-- Секция: Матрица -->
    <section id="matrix">
      <h2>Матрица приоритизации</h2>
      <div class="matrix-grid">
        <div class="zone zone-1" id="zone-1">
          <h3>Срочные изменения</h3>
        </div>
        <div class="zone zone-2" id="zone-2">
          <h3>Развитие</h3>
        </div>
        <div class="zone zone-3" id="zone-3">
          <h3>Стандартизация</h3>
        </div>
        <div class="zone zone-4" id="zone-4">
          <h3>Мониторинг</h3>
        </div>
      </div>
    </section>
  </main>

  <script src="app.js"></script>
</body>
</html>
```

!!! hint "Подсказка: CSS-фреймворк"

    Если хотите использовать Bootstrap, добавьте в `<head>`:

    ```html
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
          rel="stylesheet">
    ```

    Для Tailwind CSS:

    ```html
    <script src="https://cdn.tailwindcss.com"></script>
    ```

    Фреймворк **не обязателен** — можно написать CSS с нуля.

---

## 1.2 Массив данных в JavaScript

Создайте файл `app.js` и объявите массив начальных процессов:

```javascript
// ========== ДАННЫЕ ==========

const initialProcesses = [
  {
    id: 1,
    name: "Приём и обработка заявки",
    type: "Основной",
    cycleTime: 2, cost: 3, automation: 2, rework: 4,
    strategicValue: 5, criticality: 4, innovation: 5, techDebt: 2
  },
  {
    id: 2,
    name: "Планирование маршрутов",
    type: "Основной",
    cycleTime: 1, cost: 2, automation: 1, rework: 5,
    strategicValue: 5, criticality: 5, innovation: 5, techDebt: 1
  },
  {
    id: 3,
    name: "Исполнение рейса",
    type: "Основной",
    cycleTime: 3, cost: 3, automation: 2, rework: 3,
    strategicValue: 4, criticality: 5, innovation: 4, techDebt: 2
  },
  {
    id: 4,
    name: "Расчёт стоимости и счета",
    type: "Поддерживающий",
    cycleTime: 2, cost: 2, automation: 2, rework: 4,
    strategicValue: 3, criticality: 4, innovation: 3, techDebt: 2
  },
  {
    id: 5,
    name: "Управление парком (ТО)",
    type: "Поддерживающий",
    cycleTime: 4, cost: 4, automation: 3, rework: 2,
    strategicValue: 3, criticality: 4, innovation: 3, techDebt: 3
  },
  {
    id: 6,
    name: "Работа с претензиями",
    type: "Управленческий",
    cycleTime: 1, cost: 2, automation: 1, rework: 5,
    strategicValue: 4, criticality: 3, innovation: 4, techDebt: 1
  }
];

// Рабочая копия массива (с ней будем работать)
let processes = [...initialProcesses];
```

!!! task "Что нужно понять"

    - `initialProcesses` — **константа**, исходные данные для демонстрации
    - `processes` — **рабочая копия**, которую мы будем изменять (добавлять/удалять элементы)
    - Оператор `...` (spread) создаёт **копию массива**, а не ссылку на оригинал

---

## 1.3 Функции расчёта

Добавьте в `app.js` две функции:

```javascript
// ========== РАСЧЁТЫ ==========

function calcOperational(process) {
  return (process.cycleTime + process.cost
        + process.automation + process.rework) / 4;
}

function calcStrategic(process) {
  return (process.strategicValue + process.criticality
        + process.innovation + process.techDebt) / 4;
}

function getZone(process) {
  const op = calcOperational(process);
  const st = calcStrategic(process);

  if (op < 3 && st >= 3) return 1; // Срочные изменения
  if (op >= 3 && st >= 3) return 2; // Развитие
  if (op < 3 && st < 3)  return 3; // Стандартизация
  return 4;                         // Мониторинг
}
```

!!! concept "Почему функции, а не свойства объекта?"

    Средние оценки **не хранятся** в данных — они **вычисляются** каждый раз при отображении. Это правильный подход: если пользователь изменит одну оценку, средние автоматически пересчитаются без дополнительной синхронизации.

---

## 1.4 Базовый CSS

Создайте файл `style.css` с минимальным оформлением:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Arial, sans-serif;
  background: #f5f5f5;
  color: #333;
  padding: 20px;
  max-width: 1100px;
  margin: 0 auto;
}

header h1 {
  text-align: center;
  color: #1a237e;
  margin-bottom: 24px;
}

section {
  background: white;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

/* --- Таблица --- */
table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px 12px;
  text-align: left;
  border-bottom: 1px solid #e0e0e0;
}

th {
  background: #e8eaf6;
  color: #1a237e;
  font-weight: 600;
  cursor: pointer;
}

th:hover {
  background: #c5cae9;
}

/* --- Матрица --- */
.matrix-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 12px;
  max-width: 700px;
  margin: 0 auto;
}

.zone {
  border-radius: 8px;
  padding: 16px;
  min-height: 120px;
}

.zone h3 {
  font-size: 0.95rem;
  margin-bottom: 8px;
}

.zone-1 { background: #ffebee; border: 2px solid #e53935; color: #b71c1c; }
.zone-2 { background: #e8f5e9; border: 2px solid #43a047; color: #1b5e20; }
.zone-3 { background: #fff9c4; border: 2px solid #fdd835; color: #f57f17; }
.zone-4 { background: #e3f2fd; border: 2px solid #1e88e5; color: #0d47a1; }

/* --- Карточка процесса в матрице --- */
.process-chip {
  display: inline-block;
  background: rgba(255,255,255,0.8);
  border-radius: 4px;
  padding: 4px 8px;
  margin: 3px;
  font-size: 0.82rem;
  font-weight: 500;
}

/* --- Кнопки --- */
button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: opacity 0.2s;
}

button:hover {
  opacity: 0.85;
}

.btn-add {
  background: #1a237e;
  color: white;
}

.btn-delete {
  background: #e53935;
  color: white;
  padding: 4px 10px;
  font-size: 0.78rem;
}

.btn-export {
  background: #2e7d32;
  color: white;
}

.btn-reset {
  background: #757575;
  color: white;
}
```

!!! hint "Это — стартовый CSS"

    Вы можете полностью переписать стили или заменить их фреймворком. Главное, чтобы интерфейс был **читаемым** и зоны матрицы были **визуально различимы** по цвету.

---

## Чек-лист этапа 1

| # | Что проверить | Баллы |
|---|---|---|
| 1.1 | `index.html` открывается в браузере без ошибок в консоли | 1 |
| 1.2 | Массив `initialProcesses` объявлен в `app.js` с 6 объектами | 1 |
| 1.3 | Функции `calcOperational`, `calcStrategic`, `getZone` работают | 2 |
| 1.4 | CSS: таблица и матрица визуально различимы, страница аккуратная | 1 |
| | **Итого за этап 1** | **5** |

!!! sandbox "Проверка в консоли"

    Откройте `index.html` в браузере, нажмите `F12` → вкладка **Console** и введите:

    ```javascript
    console.log(calcOperational(processes[0])); // Ожидаем: 2.75
    console.log(calcStrategic(processes[0]));    // Ожидаем: 4
    console.log(getZone(processes[0]));          // Ожидаем: 1
    ```

    Если всё верно — функции работают. Переходите к этапу 2.
