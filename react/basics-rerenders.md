# React: Основы, ререндеры, useState

## 1. Для чего придумали React?

```jsx
// React придумали, когда стало сложно поддерживать большие динамические интерфейсы на чистом JS и ручном DOM

// ========== Какие проблемы решает React ==========
// 1. Ручное управление DOM — больше не нужно писать document.createElement, appendChild, removeChild
// 2. Сложная синхронизация данных и интерфейса — state сам обновляет UI
// 3. Масштабирование — компонентная архитектура упрощает разработку

// ========== Плюсы React по сравнению с чистым JS ==========
// 1. Декларативный подход — описываем КАК должно выглядеть UI, а не КАК его создать
// 2. Компонентность — код можно переиспользовать
// 3. Предсказуемое обновление UI — изменился state → UI обновился

// ========== Минусы React (когда НЕ нужно использовать) ==========
// 1. Небольшие проекты — проще написать на чистом JS
// 2. Нужно понимать много концепций (state, props, hooks, render, memo, refs...)
// 3. Производительность можно снизить плохим кодом (если не понимать ререндеры)
```

---

## 2. Что такое ререндеры в React?

```jsx
// Ререндер — повторный вызов функции компонента, чтобы React пересчитал JSX
// и понял, что нужно обновить реальный DOM

// ========== Триггеры ререндера (3 основных) ==========
// 1. Изменился state (через useState, useReducer)
// 2. Изменились props (родитель передал новые значения)
// 3. Обновился родитель (родитель перерендерился → дочерние тоже)
```

---

## 3. Хук useState

```jsx
// ========== Как использовать ==========
const [count, setCount] = useState(0);
// count — текущее значение
// setCount — функция для обновления

// Обновление:
setCount(count + 1); // новый явный value
setCount((prev) => prev + 1); // функциональное обновление (безопаснее)

// ========== Зачем придумали useState? ==========
// Не можем просто хранить переменную вне компонента, потому что:
// 1. При ререндере переменная создаётся заново
// 2. React не знает, что UI надо обновить

// ❌ НЕПРАВИЛЬНО:
let count = 0;
function App() {
  const handleClick = () => {
    count++;
    // UI не обновится, React не знает об изменении
  };
  return <button onClick={handleClick}>{count}</button>;
}

// ✅ ПРАВИЛЬНО:
function App() {
  const [count, setCount] = useState(0);
  const handleClick = () => setCount(count + 1);
  return <button onClick={handleClick}>{count}</button>;
}

// ========== Особенность: useState с объектами/массивами ==========
// React сравнивает ссылки. Если ссылка та же — ререндера не будет

// ❌ НЕПРАВИЛЬНО (мутация):
const [user, setUser] = useState({ name: "Аня", age: 20 });
const updateAge = () => {
  user.age++; // мутируем объект
  setUser(user); // ссылка та же → ререндера НЕ будет
};

// ✅ ПРАВИЛЬНО (новая ссылка):
const updateAge = () => {
  setUser({ ...user, age: user.age + 1 }); // новый объект
};

// Для массивов:
// ❌ setItems.push(newItem); setItems(items);
// ✅ setItems([...items, newItem]);

// ========== useState — синхронный или асинхронный? ==========
// setState — асинхронный! React планирует обновление, но не выполняет сразу

const handleClick = () => {
  setCount(count + 1);
  setCount(count + 1);
  console.log(count); // старый count (ещё не обновлён)
};
// UI обновится только на +1, а не +2, потому что count замкнут на старом значении

// Решение — функциональное обновление:
const handleClick = () => {
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
  // теперь будет +2, потому что prev — актуальное предыдущее значение
};

// ========== Batched Updates (пакетирование обновлений) ==========
// React объединяет несколько setState в один ререндер для производительности

// Пример: несколько setState внутри одного обработчика
setCount((c) => c + 1);
setFlag((f) => !f);
setUser({ ...user, name: "John" });
// React сделает ОДИН ререндер, а не три
```

---

## Практические задачи

### Задача 1. Обычная переменная вместо state

```jsx
// Что будет отображаться при клике на кнопку?
// Будет ли число увеличиваться в интерфейсе?

function App() {
  let count = 0;

  const handleClick = () => {
    count++;
    console.log("Clicked:", count);
  };

  return (
    <div>
      <p>Счётчик: {count}</p>
      <button onClick={handleClick}>+</button>
    </div>
  );
}

// Ответ:
// ❌ Число НЕ будет увеличиваться в интерфейсе
// ✅ В консоли будет увеличиваться (1, 2, 3...)
// Причина: React не знает об изменении переменной, ререндера нет
```

### Задача 2. Мутация объекта в state

```jsx
function App() {
  const [user, setUser] = React.useState({ name: "Аня", age: 20 });

  const updateAge = () => {
    user.age++;
    setUser(user);
    console.log("age++", user.age);
  };

  console.log("App render");

  return (
    <div>
      <p>Возраст: {user.age}</p>
      <button onClick={updateAge}>+1 год</button>
    </div>
  );
}

// Ответ:
// ✅ В консоли age++ будет увеличиваться (21, 22, 23...)
// ❌ Компонент НЕ будет ререндериться (возраст в UI не меняется)
// Причина: setUser(user) — ссылка на объект не изменилась, React не видит разницы
```

### Задача 3. Два setCount подряд

```jsx
function App() {
  const [count, setCount] = React.useState(0);

  const handleClick = () => {
    setCount(count + 1);
    setCount(count + 1);
    console.log("Clicked count:", count);
  };

  console.log("App render");

  return (
    <div>
      <p>Счётчик: {count}</p>
      <button onClick={handleClick}>+</button>
    </div>
  );
}

// Ответ:
// ✅ В консоли: 0 (при первом клике), потом 1, потом 2...
// ✅ В UI: счётчик увеличится на +1 (а не на +2)
// ❌ Второй setCount(count + 1) не даёт эффекта

// Почему: оба вызова используют одно и то же значение count (которое замкнуто)
// Решение:
// setCount(prev => prev + 1);
// setCount(prev => prev + 1); // теперь +2
```

### Задача 4. Ререндеры: Header и Counter

```jsx
const Header = () => {
  console.log("● Header render");
  return <h1>Приложение</h1>;
};

const Counter = () => {
  const [count, setCount] = React.useState(0);
  console.log("● Counter render");

  return (
    <div>
      <p>Счётчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
};

function App() {
  console.log("● App render");
  return (
    <div>
      <Header />
      <Counter />
    </div>
  );
}

// Ответ:
// 1. При загрузке: App render → Header render → Counter render
// 2. При клике на +1: Counter render (Header НЕ перерендерится)
// Причина: изменился только state в Counter, родитель (App) не менялся
```

### Задача 5. Ререндеры: App + UserInput

```jsx
const UserInput = () => {
  const [name, setName] = React.useState("");
  console.log("▲ UserInput render");

  return (
    <div>
      <input onChange={(e) => setName(e.target.value)} />
      <p>Имя: {name}</p>
    </div>
  );
};

function App() {
  const [count, setCount] = React.useState(0);
  console.log("● App render");

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <p>Счётчик: {count}</p>
      <UserInput />
    </div>
  );
}

// Ответ:
// 1. При загрузке: App render → UserInput render
// 2. При клике на +1: App render → UserInput render (дочерний перерендерился, потому что родитель обновился)
// 3. При вводе текста в input: UserInput render (App не перерендеривается)
```

### Задача 6. Ререндеры: Likes, Views, Stats, Page

```jsx
const Likes = ({ onLike, count }) => {
  console.log("Likes render");
  return <button onClick={onLike}>Лайков: {count}</button>;
};

const Views = () => {
  const [views, setViews] = React.useState(0);
  console.log("Views render");
  return <button onClick={() => setViews(views + 1)}>Просмотров: {views}</button>;
};

const Stats = () => {
  const [likes, setLikes] = React.useState(0);
  console.log("Stats render");

  const handleLike = () => {
    setLikes((prev) => prev + 1);
  };

  return (
    <div>
      <Likes count={likes} onLike={handleLike} />
      <Views />
    </div>
  );
};

function Page() {
  console.log("Page render");
  return (
    <div>
      <h1>Статья</h1>
      <Stats />
    </div>
  );
}

// Ответ:
// 1. При загрузке: Page render → Stats render → Likes render → Views render
// 2. При клике на лайк: Stats render → Likes render → Views render
//    (потому что state likes в Stats → Stats перерендерился → все дочерние тоже)
// 3. При клике на просмотры: Views render (только Views)
//    (state views в самом Views, родитель не меняется)
```

---

## DOM-задача: Таблица с редактированием ячеек

**HTML:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Editable Table</title>
    <style>
      table {
        border-collapse: collapse;
      }
      td {
        border: 1px solid #000;
        padding: 10px 15px;
        cursor: pointer;
        min-width: 100px;
      }
      input {
        width: 100%;
        border: none;
        outline: none;
        font-size: inherit;
        padding: 0;
        margin: -5px 0;
      }
    </style>
  </head>
  <body>
    <table id="editableTable">
      <tr>
        <td>Ячейка 1</td>
        <td>Ячейка 2</td>
      </tr>
      <tr>
        <td>Ячейка 3</td>
        <td>Ячейка 4</td>
      </tr>
    </table>

    <script>
      const table = document.querySelector("#editableTable");
      let editingCell = null;

      table.addEventListener("dblclick", (e) => {
        const cell = e.target.closest("td");
        if (!cell) return;

        // Нельзя редактировать несколько ячеек одновременно
        if (editingCell) return;

        editingCell = cell;
        const oldValue = cell.textContent;

        // Создаём input
        const input = document.createElement("input");
        input.type = "text";
        input.value = oldValue;

        // Очищаем ячейку и добавляем input
        cell.textContent = "";
        cell.appendChild(input);
        input.focus();
        input.select();

        function save() {
          cell.textContent = input.value;
          editingCell = null;
        }

        input.addEventListener("blur", save);
        input.addEventListener("keydown", (event) => {
          if (event.key === "Enter") {
            save();
          }
        });
      });
    </script>
  </body>
</html>
```

---

## Шпаргалка по ререндерам

| Триггер             | Пример                          | Ререндер                       |
| ------------------- | ------------------------------- | ------------------------------ |
| Изменение state     | `setCount(5)`                   | ✅ текущий компонент           |
| Изменение props     | родитель передал новое значение | ✅ дочерний компонент          |
| Обновление родителя | родитель перерендерился         | ✅ все дочерние (по умолчанию) |

---

## Шпаргалка по useState

| Вопрос                                         | Ответ                                   |
| ---------------------------------------------- | --------------------------------------- |
| Хранит ли значение между рендерами?            | Да                                      |
| Вызывает ли ререндер?                          | Да                                      |
| Синхронный или асинхронный?                    | Асинхронный (setState)                  |
| Как обновить объект?                           | `setUser({ ...user, newField: value })` |
| Как обновить массив?                           | `setItems([...items, newItem])`         |
| Как обновить, опираясь на предыдущее значение? | `setCount(prev => prev + 1)`            |

---
