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

# Event Loop, Promise, Async/Await in JavaScript

## Теория (вопросы и ответы)

---

## 1. Что такое Event Loop?

Event Loop — это **механизм в JavaScript, который управляет выполнением кода, очередями задач и Call Stack**.

JavaScript однопоточный, поэтому Event Loop обеспечивает «псевдоасинхронность».

---

### Как работает:

```text

1. Выполняется весь синхронный код (Call Stack)

2. Выполняются ВСЕ микротаски

3. Выполняется ОДНА макротаска

4. Повтор

```

---

### Пример:

```js id="e1"
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");

// 1, 4, 3, 2
```

---

## 2. Что такое Web API в Event Loop?

Web API — это **возможности браузера, которые не являются частью JS**.

---

### Примеры:

- setTimeout / setInterval

- fetch

- DOM события

- localStorage

- MutationObserver

---

### Как работает:

```js id="e2"
setTimeout(() => console.log("timer"), 1000);
```

1. JS передаёт задачу в Web API
2. Web API ждёт
3. После — отправляет в очередь макротасок
4. Event Loop забирает и выполняет

---

## 3. Микро и макротаски

---

### Микротаски (Microtasks)

Выполняются **ВСЕГДА сразу после синхронного кода**.

Примеры:

- Promise.then

- queueMicrotask

- MutationObserver

---

### Макротаски (Macrotasks)

Выполняются **по одной за цикл Event Loop**.

Примеры:

- setTimeout

- setInterval

- DOM events

- fetch callbacks

---

### Порядок:

```text

Sync → Microtasks → Macrotask → Render

```

---

## 4. Асинхронность и синхронность

---

### Синхронность

Код выполняется **последовательно**, блокируя поток.

```js id="e3"
console.log(1);

console.log(2);
```

---

### Асинхронность

Код выполняется **не сразу**, а позже через Event Loop.

```js id="e4"
setTimeout(() => console.log(2), 0);

console.log(1);
```

---

## 5. Что такое Promise?

Promise — это объект, который представляет **результат асинхронной операции**.

---

### Состояния:

- pending

- fulfilled

- rejected

---

### Пример:

```js id="e5"
const p = new Promise((resolve, reject) => {
  resolve("OK");
});
```

---

## 6. Что возвращает setTimeout?

```js id="e6"
const id = setTimeout(() => {}, 1000);
```

Возвращает:

- числовой ID таймера (в браузере)

- используется для clearTimeout

---

## 7. Как работает .then().then()?

---

Каждый `.then`:

- получает результат предыдущего

- возвращает новый Promise

---

### Пример:

```js id="e7"
Promise.resolve(1)

  .then((x) => x + 1)

  .then((x) => x + 1)

  .then(console.log); // 3
```

---

### Если return нет:

возвращается `undefined`

---

## 8. Двойной .then() — разбор

```js id="e8"
Promise.resolve()

  .then(() => {
    console.log(1);

    return 2;
  })

  .then((x) => {
    console.log(x);
  });

// 1 → 2
```

---

## 9. Как работает async/await?

---

### async

Функция всегда возвращает Promise.

---

### await

- останавливает выполнение внутри async

- ждёт Promise

---

### Пример:

```js id="e9"
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

test();

console.log("C");

// A, C, B
```

---

## 10. Что если не обработать Promise.reject?

---

Если нет `.catch`:

- появляется Unhandled Promise Rejection

- в Node.js может завершить процесс

- в браузере — warning в консоли

---

## 11. Рекурсия микротасок и макротасок

---

### Микротаски (опасно)

```js id="e10"
function loop() {
  queueMicrotask(loop);
}
```

❌ блокирует Event Loop полностью

---

### Макротаски (безопаснее)

```js id="e11"
function loop() {
  setTimeout(loop, 0);
}
```

✔ Event Loop остаётся живым

---

### Обычная рекурсия

```js id="e12"
function f() {
  f();
}
```

→ переполнение Call Stack

---

## 12. Бесконечен ли Call Stack?

---

❌ Нет.

У него есть ограничение памяти.

---

### Итог:

```text

Call Stack → ограничен

Event Loop → бесконечен

```

---

## Что сказать на собеседовании

Event Loop — это механизм, который управляет выполнением синхронного и асинхронного кода в JavaScript.
Асинхронность реализуется через Web API и очереди микротасок и макротасок.
Микротаски выполняются полностью перед макротасками.
Promise — это объект, представляющий результат асинхронной операции.
async/await — синтаксический сахар над Promise, который делает асинхронный код синхронным по виду.
setTimeout возвращает ID таймера.
Promise.reject без обработки приводит к необработанной ошибке.
Call Stack ограничен, а Event Loop работает бесконечно.

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

## 7. Event Loop: рекурсия микротасок

```js
// Что будет, если произойдет рекурсия микротасок?

// ========== Пример 1: Обычная рекурсия ==========
function normalRecursion() {
  normalRecursion();
}
// normalRecursion(); // RangeError: Maximum call stack size exceeded
// Call Stack переполняется → ошибка

// ========== Пример 2: Рекурсия микротасок ==========
let count = 0;
function microRecursion() {
  queueMicrotask(() => {
    count++;
    console.log(count);
    microRecursion(); // Запускаем новую микротаску
  });
}
// microRecursion(); // ОСТОРОЖНО! Браузер зависнет!

// Что происходит:
// 1. Выполняется синхронный код
// 2. В очередь микротасок попадает первая задача
// 3. Event Loop берет микротаску, она создает НОВУЮ микротаску
// 4. Новая микротаска попадает в ту же очередь микротасок
// 5. Event Loop продолжает брать микротаски БЕСКОНЕЧНО
// 6. Макротаски (setTimeout, рендеринг) НИКОГДА не выполняются
// 7. Страница зависает

// ========== Пример 3: Рекурсия макротасок (безопасно) ==========
function macroRecursion() {
  setTimeout(() => {
    count++;
    console.log(count);
    macroRecursion(); // Запускаем новую макротаску
  }, 0);
}
// macroRecursion(); // БЕЗОПАСНО! Страница не зависает.

// Что происходит:
// 1. Выполняется синхронный код
// 2. Макротаска попадает в очередь макротасок
// 3. Event Loop берет ОДНУ макротаску
// 4. После нее выполняются ВСЕ микротаски и РЕНДЕРИНГ
// 5. Потом берется следующая макротаска
// 6. Страница отзывчивая, анимации работают

// Итог: рекурсия микротасок — смертельная для страницы
//       рекурсия макротасок — безопасна (но бесконечна)
```

---

## 8. Для чего нужен async/await?

```js
// async/await — синтаксический сахар над Promise.
// Делает асинхронный код похожим на синхронный.

// ========== Без async/await (цепочки then) ==========
function getUserData() {
  return fetch("/api/user")
    .then((response) => response.json())
    .then((user) => fetch(`/api/posts/${user.id}`))
    .then((response) => response.json())
    .then((posts) => {
      console.log(posts);
      return posts;
    })
    .catch((error) => console.error(error));
}

// ========== С async/await (чище и понятнее) ==========
async function getUserData() {
  try {
    const userResponse = await fetch("/api/user");
    const user = await userResponse.json();

    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();

    console.log(posts);
    return posts;
  } catch (error) {
    console.error(error);
  }
}

// ========== async превращает функцию в Promise ==========
async function getNumber() {
  return 42; // неявно оборачивается в Promise.resolve(42)
}
getNumber().then(console.log); // 42

// ========== await работает только внутри async функций ==========
// await можно использовать только внутри async (или в модулях верхнего уровня)

// ========== Обработка нескольких независимых промисов ==========
async function fetchAll() {
  // ПЛОХО: последовательно (медленно)
  const user = await fetchUser();
  const posts = await fetchPosts();

  // ХОРОШО: параллельно (быстро)
  const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
}

// ========== Главные преимущества ==========
// 1. Читаемость — линейный код вместо вложенных then
// 2. try/catch — привычная обработка ошибок
// 3. Дебаг — можно ставить breakpoint на каждую строчку
```

---

## 9. Кастомные методы Promise

### Promise.all

```js
// Promise.all — ждет выполнения ВСЕХ промисов, возвращает массив результатов.
// Если любой промис упадет с ошибкой → сразу reject с этой ошибкой.

function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    // Пустой массив → сразу resolve([])
    if (promises.length === 0) {
      resolve(results);
      return;
    }

    promises.forEach((promise, index) => {
      // Promise.resolve() оборачивает не-промисы в промисы
      Promise.resolve(promise)
        .then((result) => {
          results[index] = result;
          completed++;

          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch((error) => {
          reject(error); // первая же ошибка
        });
    });
  });
}

// Пример:
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.resolve(3);

promiseAll([p1, p2, p3]).then(console.log); // [1, 2, 3]
```

### Promise.race

```js
// Promise.race — возвращает результат ПЕРВОГО завершенного промиса
// (неважно, resolve или reject)

function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach((promise) => {
      Promise.resolve(promise)
        .then(resolve) // первый успешный
        .catch(reject); // первая ошибка
    });
  });
}

// Пример:
const slow = new Promise((resolve) => setTimeout(() => resolve("медленный"), 300));
const fast = new Promise((resolve) => setTimeout(() => resolve("быстрый"), 100));

promiseRace([slow, fast]).then(console.log); // "быстрый"
```

### Promise.allSettled

```js
// Promise.allSettled — ждет ВСЕ промисы, возвращает массив объектов
// с результатами каждого (статус + value/reason)

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

// Пример:
const pSuccess = Promise.resolve("OK");
const pFail = Promise.reject("Error");
const pSlow = new Promise((resolve) => setTimeout(() => resolve("Медленный"), 200));

promiseAllSettled([pSuccess, pFail, pSlow]).then((results) => {
  console.log(results);
  // [
  //   { status: "fulfilled", value: "OK" },
  //   { status: "rejected", reason: "Error" },
  //   { status: "fulfilled", value: "Медленный" }
  // ]
});
```

### Promise.any (продвинутый)

```js
// Promise.any — возвращает результат ПЕРВОГО УСПЕШНОГО промиса.
// Если все промисы упали → reject с AggregateError

function promiseAny(promises) {
  return new Promise((resolve, reject) => {
    const errors = [];
    let rejectedCount = 0;

    if (promises.length === 0) {
      reject(new AggregateError([], "All promises were rejected"));
      return;
    }

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(resolve) // первый успешный → resolve
        .catch((error) => {
          errors[index] = error;
          rejectedCount++;

          if (rejectedCount === promises.length) {
            reject(new AggregateError(errors, "All promises were rejected"));
          }
        });
    });
  });
}

// Пример:
const p1 = Promise.reject("Ошибка 1");
const p2 = Promise.resolve("Успех!"); // будет этот
const p3 = Promise.reject("Ошибка 2");

promiseAny([p1, p2, p3]).then(console.log); // "Успех!"
```

---

## Шпаргалка по Promise методам

| Метод                  | Что делает                              | Какой результат                              |
| ---------------------- | --------------------------------------- | -------------------------------------------- |
| `Promise.all()`        | Ждет ВСЕ промисы                        | массив результатов ИЛИ первая ошибка         |
| `Promise.allSettled()` | Ждет ВСЕ промисы                        | массив объектов (статус + результат каждого) |
| `Promise.race()`       | Первый завершенный (resolve или reject) | первый результат ИЛИ первая ошибка           |
| `Promise.any()`        | Первый УСПЕШНЫЙ                         | первый успешный результат ИЛИ AggregateError |

---

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
