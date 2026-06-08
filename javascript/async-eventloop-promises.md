# Асинхронность, Promise, Event Loop

## Вопросы с собеседования

### 1. JavaScript это синхронный или асинхронный язык?

```js
// JavaScript — синхронный, однопоточный язык.
// Но он может обрабатывать асинхронные операции с помощью Event Loop.

// Синхронный код выполняется последовательно:
console.log("Первый");
console.log("Второй");
// Вывод: Первый, Второй

// Асинхронный код не блокирует выполнение:
setTimeout(() => console.log("Асинхронный"), 0);
console.log("Синхронный");
// Вывод: Синхронный, Асинхронный
```

### 2. Что помогает JS использовать неблокирующую модель ввода/вывода?

```js
// Помогают:
// 1. Event Loop — управляет очередью задач
// 2. Web API — setTimeout, fetch, DOM события
// 3. Очереди задач (микротаски и макротаски)

// Блокирующая операция (плохо):
while (true) {} // бесконечный цикл заблокирует страницу

// Неблокирующая операция (хорошо):
setTimeout(() => console.log("Не блокирует"), 0);
```

### 3. Что такое Event Loop?

```js
// Event Loop — бесконечный цикл, который отслеживает Call Stack и очереди задач.
// Алгоритм:
// 1. Выполнить все синхронные задачи в Call Stack
// 2. Выполнить ВСЕ микротаски (Promise.then, queueMicrotask)
// 3. Выполнить ОДНУ макротаску (setTimeout, события)
// 4. Повторить

console.log("1"); // синхронный

setTimeout(() => console.log("2"), 0); // макротаска

Promise.resolve().then(() => console.log("3")); // микротаска

console.log("4"); // синхронный

// Вывод: 1, 4, 3, 2
// (сначала синхронные, потом микротаски, потом макротаски)
```

### 4. Что такое Web API в Event Loop?

```js
// Web API — функции, предоставляемые браузером (не входят в сам JS):
// - setTimeout / setInterval
// - fetch / AJAX
// - DOM события (click, keydown)
// - localStorage
// - console (частично)

// Как это работает:
setTimeout(() => console.log("таймер"), 1000);
// 1. setTimeout попадает в Web API
// 2. Таймер ждет 1000мс
// 3. Колбэк отправляется в очередь макротасок
// 4. Event Loop забирает его, когда Call Stack пуст
```

### 5. Что такое Promise?

```js
// Promise — объект, представляющий результат асинхронной операции.

// Состояния:
// - pending (ожидание)
// - fulfilled (успешно выполнен)
// - rejected (ошибка)

const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve("Успех!");
    } else {
      reject("Ошибка!");
    }
  }, 1000);
});

promise.then((result) => console.log(result)).catch((error) => console.error(error));

// Цепочки промисов:
Promise.resolve(1)
  .then((x) => x + 1)
  .then((x) => console.log(x)); // 2
```

### 6. Как работает async/await?

```js
// async — превращает функцию в Promise
async function getData() {
  return "Hello";
}
getData().then(console.log); // "Hello"

// await — приостанавливает выполнение функции до разрешения Promise
async function fetchData() {
  console.log("Начало");
  const result = await Promise.resolve("Данные");
  console.log(result); // выполнится после разрешения промиса
  console.log("Конец");
}
fetchData();
console.log("Синхронный");
// Вывод: Начало, Синхронный, Данные, Конец

// Обработка ошибок:
async function fetchWithError() {
  try {
    const result = await Promise.reject("Ошибка");
  } catch (error) {
    console.log(error); // "Ошибка"
  }
}
```

### 7. Определения синхронности и асинхронности?

```js
// Синхронность — код выполняется последовательно,
// каждая следующая операция ждет завершения предыдущей.

// Асинхронность — код не ждет завершения операций,
// продолжает выполнение, а результат обрабатывается позже.

// Синхронный пример:
function syncExample() {
  const start = Date.now();
  while (Date.now() - start < 3000) {} // ждем 3 секунды
  console.log("Прошло 3 секунды");
  console.log("Конец");
}
// syncExample(); // 3 секунды блокировки

// Асинхронный пример:
function asyncExample() {
  setTimeout(() => console.log("Прошло 3 секунды"), 3000);
  console.log("Конец");
}
// asyncExample(); // "Конец" сразу, потом через 3 секунды "Прошло 3 секунды"
```

### 8. Что будет при рекурсии микротасок и макротасок?

```js
// Обычная рекурсия — заполняет Call Stack, будет ошибка переполнения
function normalRecursion() {
  normalRecursion(); // RangeError: Maximum call stack size exceeded
}

// Рекурсия микротасок — БЕСКОНЕЧНО выполняется, блокирует рендеринг
let count = 0;
function microRecursion() {
  queueMicrotask(() => {
    count++;
    console.log("Микро:", count);
    microRecursion(); // Бесконечно, страница зависнет
  });
}
// microRecursion(); // ОСТОРОЖНО! ЗАВЕСИТ БРАУЗЕР!

// Рекурсия макротасок — работает бесконечно, НЕ блокирует рендеринг
function macroRecursion() {
  setTimeout(() => {
    count++;
    console.log("Макро:", count);
    macroRecursion(); // Бесконечно, но между тасками есть микротаски и рендер
  }, 0);
}
// macroRecursion(); // Безопасно, но бесконечно
```

### 9. Бесконечный ли Call Stack?

```js
// Нет, у Call Stack есть ограничение размера (обычно ~10-15 тысяч вызовов)

function recurse() {
  recurse();
}
// recurse(); // RangeError: Maximum call stack size exceeded

// Проверить лимит:
let count = 0;
function test() {
  count++;
  test();
}
try {
  test();
} catch (e) {
  console.log("Максимум вызовов:", count); // ~10-15 тысяч
}
```

### 10. Что возвращает setTimeout?

```js
// setTimeout возвращает числовой идентификатор (ID таймера)

const timerId = setTimeout(() => console.log("Привет"), 1000);
console.log(timerId); // 1, 2, 3... (число)

// Отмена таймера:
clearTimeout(timerId); // таймер больше не выполнится

// На это можно повесить лисенер
```

### 11. Можно ли в конструктор Promise засунуть Promise?

```js
// Нет, в конструктор Promise нужно передавать функцию (executor),
// а не готовый Promise

// НЕПРАВИЛЬНО:
const wrong = new Promise(Promise.resolve(1)); // TypeError

// ПРАВИЛЬНО:
const correct = new Promise((resolve) => {
  resolve(1);
});

// Если у вас уже есть Promise, используйте его напрямую:
const existingPromise = Promise.resolve(1);
existingPromise.then(console.log);
```

### 12. Что такое requestAnimationFrame?

```js
// requestAnimationFrame — API для анимаций.
// Выполняется ПЕРЕД рендерингом, после макротасок и микротасок.

// Порядок выполнения:
// 1. Макротаски (setTimeout, события)
// 2. Микротаски (Promise.then, queueMicrotask)
// 3. requestAnimationFrame (все колбэки)
// 4. Рендеринг (стили, layout, paint)
// 5. Новая макротаска

function animate() {
  console.log("Анимация");
  requestAnimationFrame(animate);
}
// animate(); // выполняется ~60 раз в секунду
```

### 13. Как работает fetch?

```js
// fetch — API для HTTP-запросов, возвращает Promise

// GET запрос:
fetch("https://api.example.com/users")
  .then((response) => response.json())
  .then((data) => console.log(data))
  .catch((error) => console.error("Ошибка:", error));

// POST запрос:
fetch("https://api.example.com/users", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ name: "Alice", age: 25 }),
})
  .then((res) => res.json())
  .then((data) => console.log(data));

// с async/await:
async function getUsers() {
  try {
    const response = await fetch("https://api.example.com/users");
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

---

## Практические задачи с решениями

### Задача 1. Easy — порядок вывода

```js
console.log(1);
setTimeout(() => console.log(2));
Promise.resolve().then(() => console.log(3));
Promise.resolve().then(() => setTimeout(() => console.log(4)));
Promise.resolve().then(() => console.log(5));
setTimeout(() => console.log(6));
console.log(7);

// Результат: 1, 7, 3, 5, 2, 6, 4

// Пояснение:
// 1 и 7 — синхронные, выполняются первыми
// 3 и 5 — микротаски (Promise.then), после синхронного кода
// 2, 6, 4 — макротаски (setTimeout)
// 4 внутри setTimeout, который внутри микротаски → последний
```

### Задача 2. Normal — более сложный порядок

```js
console.log("1");

const f = () => {
  setTimeout(function () {
    console.log("2");
    Promise.resolve().then(() => console.log("3"));
  }, 0);

  queueMicrotask(() => {
    console.log("4");
    setTimeout(() => console.log("5"), 0);
  });

  new Promise((resolve) => {
    console.log("11");
    resolve("10");
  }).then(console.log);
};

f();
console.log("8");

// Результат: 1, 8, 11, 4, 10, 2, 3, 5

// Пояснение:
// 1 и 8 — синхронные
// 11 — синхронный код внутри конструктора Promise
// 4 — микротаска (queueMicrotask)
// 10 — микротаска (Promise.then)
// 2, 3, 5 — макротаски (setTimeout)
```

### Задача 3. Hard — async/await

```js
console.log("Start");

setTimeout(() => console.log("Timeout 1"), 0);
Promise.resolve().then(() => console.log("Promise 1"));

async function asyncFunc() {
  console.log("Async Function Start");
  await Promise.resolve();
  console.log("Async Function End");
}

asyncFunc();

Promise.resolve().then(() => {
  setTimeout(() => console.log("Timeout 2"), 0);
});

setTimeout(() => {
  Promise.resolve().then(() => console.log("Promise 2"));
}, 0);

console.log("End");

// Результат:
// Start, Async Function Start, End, Promise 1, Async Function End,
// Timeout 1, Promise 2, Timeout 2

// Пояснение:
// asyncFunc выполняет синхронную часть до await сразу
// await Promise.resolve() — микротаска
// Async Function End — выполнится после других микротасок
```

### Задача 4. Hard — цепочка Promise с ошибками

```js
Promise.resolve()
  .then(() => {
    return "1";
  })
  .then((data) => {
    console.log(data);
    return Promise.reject(new Error("2"));
  })
  .then(() => {
    console.log("5");
    return "3";
  })
  .catch((error) => {
    console.log("6", error.message);
    return "7";
  })
  .then((data) => {
    console.log("8", data);
    throw new Error("9");
  })
  .then(() => {
    console.log("10");
    return "11";
  })
  .catch((error) => {
    console.log("12", error.message);
  })
  .then((data) => {
    console.log("13", data);
    return "14";
  })
  .then((data) => {
    console.log("15", data);
    return Promise.resolve("16");
  })
  .then((data) => {
    console.log("17", data);
  })
  .catch((error) => {
    console.log("20", error);
  });

// Результат:
// 1
// 6 2
// 8 7
// 12 9
// 13 undefined
// 15 14
// 17 16
```

---

## Кастомные реализации Promise методов

### Promise.all

```js
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    if (promises.length === 0) {
      resolve(results);
      return;
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then((result) => {
          results[index] = result;
          completed++;

          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch((error) => {
          reject(error);
        });
    });
  });
}

// Пример использования:
const p1 = new Promise((resolve) => setTimeout(() => resolve("Первый"), 300));
const p2 = new Promise((resolve) => setTimeout(() => resolve("Второй"), 100));
const p3 = new Promise((resolve) => setTimeout(() => resolve("Третий"), 200));

promiseAll([p1, p2, p3]).then((results) => console.log(results)); // ["Первый", "Второй", "Третий"]
```

### Promise.race

```js
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach((promise) => {
      Promise.resolve(promise).then(resolve).catch(reject);
    });
  });
}

// Пример использования:
const fast = new Promise((resolve) => setTimeout(() => resolve("Быстрый"), 100));
const slow = new Promise((resolve) => setTimeout(() => resolve("Медленный"), 300));

promiseRace([slow, fast]).then((result) => console.log(result)); // "Быстрый"
```

### Promise.allSettled

```js
function promiseAllSettled(promises) {
  return new Promise((resolve) => {
    const results = [];
    let completed = 0;

    if (promises.length === 0) {
      resolve(results);
      return;
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then((value) => {
          results[index] = { status: "fulfilled", value };
          completed++;
          if (completed === promises.length) resolve(results);
        })
        .catch((reason) => {
          results[index] = { status: "rejected", reason };
          completed++;
          if (completed === promises.length) resolve(results);
        });
    });
  });
}

// Пример использования:
const pSuccess = Promise.resolve("OK");
const pFail = Promise.reject("Error");

promiseAllSettled([pSuccess, pFail]).then((results) => console.log(results));
// [
//   { status: "fulfilled", value: "OK" },
//   { status: "rejected", reason: "Error" }
// ]
```

---

## DOM-задача: Счётчик с кнопками +/-

**HTML:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Счётчик</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        gap: 20px;
      }
      button {
        font-size: 24px;
        padding: 10px 20px;
        cursor: pointer;
      }
      span {
        font-size: 32px;
        min-width: 50px;
        text-align: center;
      }
    </style>
  </head>
  <body>
    <button id="decrement">-</button>
    <span id="counter">0</span>
    <button id="increment">+</button>

    <script>
      const decrementBtn = document.querySelector("#decrement");
      const incrementBtn = document.querySelector("#increment");
      const counterSpan = document.querySelector("#counter");

      incrementBtn.addEventListener("click", () => {
        counterSpan.textContent = Number(counterSpan.textContent) + 1;
      });

      decrementBtn.addEventListener("click", () => {
        let value = Number(counterSpan.textContent);
        if (value > 0) {
          counterSpan.textContent = value - 1;
        }
      });
    </script>
  </body>
</html>
```

---

## Шпаргалка по Event Loop

| Тип задачи     | Примеры                                              | Когда выполняется                                |
| -------------- | ---------------------------------------------------- | ------------------------------------------------ |
| Синхронный код | `console.log()`, циклы, функции                      | Сразу, в Call Stack                              |
| Микротаски     | `Promise.then`, `queueMicrotask`, `MutationObserver` | После синхронного кода, перед макротасками (ВСЕ) |
| Макротаски     | `setTimeout`, `setInterval`, события, `fetch`        | После микротасок (по ОДНОЙ)                      |
| Анимации       | `requestAnimationFrame`                              | После макротасок и микротасок, перед рендером    |
| Рендеринг      | отрисовка страницы                                   | После requestAnimationFrame                      |

---
