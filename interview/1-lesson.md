# Дополнительные вопросы: сеть, рендеринг, копирование, типы, HTTP, WebSocket

## 1. Рукопожатия во время поиска IP (TCP Handshake + TLS)

```js
// Что происходит, когда ты вводишь URL и нажимаешь Enter

// ========== 1. Разбор URL ==========
// https://example.com:443/page
// - https — протокол
// - example.com — домен
// - 443 — порт (по умолчанию для HTTPS)
// - /page — путь

// ========== 2. Проверка кэша браузера ==========
// - DNS кэш (домен → IP)
// - HTTP кэш (страницы, стили, скрипты)

// ========== 3. DNS-запрос (домен → IP) ==========
// Браузер → DNS-резолвер → DNS-сервер → возвращает IP
// Пример: example.com → 93.184.216.34

// ========== 4. TCP-рукопожатие (3-way handshake) ==========
// Клиент ──SYN──> Сервер (я хочу соединиться)
// Клиент <──SYN-ACK── Сервер (разрешаю, давай)
// Клиент ──ACK──> Сервер (соединение установлено)

// Это нужно, чтобы установить надежное соединение

// ========== 5. TLS-рукопожатие (для HTTPS) ==========
// Клиент ──Client Hello (версии TLS, шифры)──> Сервер
// Клиент <──Server Hello (выбранный шифр, сертификат)── Сервер
// Клиент проверяет сертификат (валидность, срок, доверенный CA)
// Клиент ──Pre-master secret (зашифрованный)──> Сервер
// Клиент <──Finished── Сервер
// Все дальнейшие данные шифруются

// ========== 6. HTTP-запрос ==========
// GET /page HTTP/1.1
// Host: example.com
// User-Agent: Chrome/...

// ========== 7. HTTP-ответ ==========
// HTTP/1.1 200 OK
// Content-Type: text/html
//
// <html>...</html>
```

---

## 2. Этапы отрисовки (Render Pipeline)

```js
// Как браузер превращает HTML/CSS в пиксели на экране

// ========== Этап 1: Парсинг HTML → DOM ==========
// HTML → токены → узлы → DOM дерево
// <div class="container">
//   <h1>Заголовок</h1>
//   <p>Текст</p>
// </div>
// ↓
// Document
//   └─ html
//       └─ body
//           └─ div.container
//               ├─ h1 → "Заголовок"
//               └─ p → "Текст"

// ========== Этап 2: Парсинг CSS → CSSOM ==========
// CSS → CSSOM (CSS Object Model)
// .container { width: 100px; }
// h1 { color: red; }
// ↓
// CSSOM дерево стилей

// ========== Этап 3: Render Tree ==========
// DOM + CSSOM = Render Tree
// (только видимые элементы, без display: none)

// ========== Этап 4: Layout (Reflow) ==========
// Расчёт геометрии: размеры и позиции каждого элемента
// Сколько места занимает элемент? Где он находится?

// ========== Этап 5: Paint ==========
// Заполнение пикселей: цвет, фон, границы, тени
// Рисуем каждый слой отдельно

// ========== Этап 6: Composite ==========
// Объединение слоёв в финальное изображение
// Отправка на GPU для отображения

// Визуально:
// HTML → DOM ─┐
//            ├→ Render Tree → Layout → Paint → Composite
// CSS  → CSSOM┘

// =========! Что вызывает Reflow (дорого) ==========
// - изменение размеров (width, height)
// - изменение позиции (top, left)
// - добавление/удаление элементов
// - изменение содержимого
// - изменение шрифтов
// - getComputedStyle() (принудительный reflow)

// =========! Что вызывает Repaint (дешевле) ==========
// - изменение цвета (color, background)
// - изменение видимости (visibility)
// - тени, обводки

// =========! Что вызывает только Composite (самое дешёвое) ==========
// - transform
// - opacity
// - filter
```

---

## 3. Нюансы глубокого копирования через JSON

```js
// JSON.parse(JSON.stringify(obj)) — НЕ идеальное глубокое копирование

// =========! Нюанс 1: Функции ==========
const obj1 = { name: "Alice", greet: () => console.log("Hi") };
const copy1 = JSON.parse(JSON.stringify(obj1));
console.log(copy1); // { name: "Alice" }  ← функция пропала!

// =========! Нюанс 2: undefined ==========
const obj2 = { name: "Alice", age: undefined };
const copy2 = JSON.parse(JSON.stringify(obj2));
console.log(copy2); // { name: "Alice" }  ← undefined пропал!

// =========! Нюанс 3: Symbol ==========
const obj3 = { name: "Alice", [Symbol("id")]: 123 };
const copy3 = JSON.parse(JSON.stringify(obj3));
console.log(copy3); // { name: "Alice" }  ← Symbol пропал!

// =========! Нюанс 4: Дата ==========
const obj4 = { date: new Date("2024-01-01") };
const copy4 = JSON.parse(JSON.stringify(obj4));
console.log(copy4.date); // "2024-01-01T00:00:00.000Z" (строка, а не Date!)
console.log(typeof copy4.date); // "string"

// =========! Нюанс 5: NaN и Infinity ==========
const obj5 = { nan: NaN, inf: Infinity, negInf: -Infinity };
const copy5 = JSON.parse(JSON.stringify(obj5));
console.log(copy5); // { nan: null, inf: null, negInf: null }  ← стали null!

// =========! Нюанс 6: Циклические ссылки ==========
const obj6 = { name: "Alice" };
obj6.self = obj6; // циклическая ссылка
// JSON.parse(JSON.stringify(obj6)); // ❌ TypeError: Converting circular structure to JSON

// =========! Нюанс 7: Map и Set ==========
const obj7 = { map: new Map([["key", "value"]]), set: new Set([1, 2, 3]) };
const copy7 = JSON.parse(JSON.stringify(obj7));
console.log(copy7); // { map: {}, set: {} }  ← пустые объекты!
```

---

## 4. Глубокое vs поверхностное копирование

```js
// ========== Поверхностное копирование (shallow copy) ==========
// Копируется только первый уровень, вложенные объекты остаются по ссылке

// 1. Spread оператор
const shallow1 = { ...original };

// 2. Object.assign()
const shallow2 = Object.assign({}, original);

// 3. Array.slice() (для массивов)
const shallow3 = originalArray.slice();

// 4. Array.from()
const shallow4 = Array.from(originalArray);

// 5. Spread для массивов
const shallow5 = [...originalArray];

// Пример проблемы:
const original = { a: 1, b: { c: 2 } };
const shallow = { ...original };
shallow.b.c = 99;
console.log(original.b.c); // 99 ← изменилось! (общая ссылка)

// ========== Глубокое копирование (deep copy) ==========
// Копируются все уровни вложенности, полная независимость

// 1. structuredClone() (современный способ, Node 17+ / все браузеры)
const deep1 = structuredClone(original);

// 2. JSON.parse(JSON.stringify()) (но у него есть нюансы!)
const deep2 = JSON.parse(JSON.stringify(original));

// 3. Lodash.cloneDeep()
// import cloneDeep from 'lodash/cloneDeep';
// const deep3 = cloneDeep(original);

// 4. Рекурсивная функция
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Map) return new Map(Array.from(obj, ([k, v]) => [k, deepClone(v)]));
  if (obj instanceof Set) return new Set(Array.from(obj, (v) => deepClone(v)));

  const clone = Array.isArray(obj) ? [] : {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key]);
    }
  }
  return clone;
}

// Пример правильного глубокого копирования:
const original2 = { a: 1, b: { c: 2 } };
const deep = structuredClone(original2);
deep.b.c = 99;
console.log(original2.b.c); // 2 ← не изменилось!
```

---

## 5. typeof NaN

```js
console.log(typeof NaN); // "number"

// NaN — это специальное числовое значение (Not a Number)
// Но по типу это number!

// Проверка на NaN:
console.log(NaN === NaN); // false (NaN не равен сам себе!)
console.log(isNaN(NaN)); // true (старый способ, не надёжен)
console.log(Number.isNaN(NaN)); // true (правильный способ)

// Number.isNaN() — безопаснее, не преобразует аргумент в число
console.log(isNaN("abc")); // true (сначала "abc" → NaN)
console.log(Number.isNaN("abc")); // false (не число, не NaN)
```

---

## 6. Математические операции с undefined и null

```js
// ========== null ==========
console.log(5 + null); // 5  (null преобразуется в 0)
console.log(5 - null); // 5
console.log(5 * null); // 0
console.log(5 / null); // Infinity  (5 / 0)

// ========== undefined ==========
console.log(5 + undefined); // NaN
console.log(5 - undefined); // NaN
console.log(5 * undefined); // NaN
console.log(5 / undefined); // NaN

// Почему?
// null — явное "пустое значение", преобразуется в 0
// undefined — "значение не установлено", преобразуется в NaN

// Строковые операции:
console.log(5 + null); // 5
console.log("5" + null); // "5null" (строковая конкатенация)
console.log(5 + undefined); // NaN
console.log("5" + undefined); // "5undefined"
```

---

## 7. Создание объекта с родителем null

```js
// Объект без прототипа
const obj = Object.create(null);

// У такого объекта нет:
// - toString()
// - hasOwnProperty()
// - __proto__
// - и всех других методов Object.prototype

console.log(obj.toString); // undefined
console.log(obj.__proto__); // undefined

// Преимущества:
// 1. Полностью чистая хеш-таблица (без конфликтов с прототипными методами)
// 2. Безопаснее для хранения пользовательских данных

// Пример:
const dict = Object.create(null);
dict["toString"] = "не сломается";
dict["hasOwnProperty"] = "тоже безопасно";
console.log(dict.toString); // "не сломается" (не метод!)

// Обычный объект:
const normalObj = {};
normalObj["toString"] = "перезапишет метод?";
console.log(normalObj.toString); // "перезапишет метод?" — да, перезаписали!
```

---

## 8. Hoisting (поднятие)

```js
// Hoisting — механизм, при котором объявления переменных и функций
// "поднимаются" вверх своей области видимости

// ========== var ==========
console.log(a); // undefined (не ошибка!)
var a = 5;
// Как это работает:
// var a;
// console.log(a);
// a = 5;

// ========== let / const ==========
console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 5;
// Они тоже поднимаются, но находятся во "временной мёртвой зоне" (TDZ)
// до строки объявления

// ========== function declaration ==========
sayHi(); // "Привет!" (работает!)
function sayHi() {
  console.log("Привет!");
}
// Поднимается полностью: и объявление, и тело

// ========== function expression ==========
sayBye(); // ❌ TypeError: sayBye is not a function
var sayBye = function () {
  console.log("Пока!");
};
// Поднимается только объявление переменной, не присвоение

// =========! Итог =========!
// var    → поднимается (значение undefined)
// let/const → поднимаются, но в TDZ (ошибка при доступе до объявления)
// function declaration → поднимается полностью
// function expression → поднимается как переменная (var или let)
```

---

## 9. HTTP методы: GET, OPTIONS, PUT и другие

```js
// ========== GET ==========
// - Получение данных
// - Параметры в URL
// - Без тела запроса
// - Идемпотентный (одинаковые запросы → одинаковый результат)
// - Кэшируется

// Пример: GET /users?id=123

// ========== POST ==========
// - Создание нового ресурса
// - Данные в теле запроса
// - НЕ идемпотентный (два одинаковых запроса → два ресурса)
// - Не кэшируется

// Пример: POST /users { "name": "Alice" }

// ========== PUT ==========
// - Полное обновление ресурса (заменяет целиком)
// - Данные в теле запроса
// - Идемпотентный
// - Требует указания ID

// Пример: PUT /users/123 { "name": "Alice", "age": 25 }

// ========== PATCH ==========
// - Частичное обновление ресурса
// - Данные в теле запроса
// - НЕ идемпотентный (обычно)
// - Требует указания ID

// Пример: PATCH /users/123 { "age": 26 }

// ========== DELETE ==========
// - Удаление ресурса
// - Идемпотентный
// - Требует указания ID

// Пример: DELETE /users/123

// ========== OPTIONS ==========
// - Запрос поддерживаемых методов (CORS)
// - Отправляется браузером автоматически перед "опасными" запросами
// - Проверяет, разрешён ли запрос с другого домена

// Пример: OPTIONS /users
// Ответ: Allow: GET, POST, PUT, DELETE

// ========== HEAD ==========
// - Как GET, но без тела ответа (только заголовки)
// - Проверка существования ресурса

// =========! Что такое идемпотентность =========!
// Одинаковые запросы приводят к одинаковому результату
// GET, PUT, DELETE — идемпотентные
// POST, PATCH — не идемпотентные
```

---

## 10. REST API

```js
// REST (Representational State Transfer) — архитектурный стиль для API

// ========== Основные принципы REST ==========
// 1. Клиент-серверная архитектура
// 2. Отсутствие состояния (stateless) — каждый запрос содержит всю информацию
// 3. Кэширование
// 4. Единообразие интерфейса
// 5. Слои (между клиентом и сервером могут быть прокси, кэши)

// =========! REST использует ресурсы =========!
// Ресурс — это объект (user, product, order)

// =========! URL определяет ресурс =========!
// GET    /users         → список пользователей
// GET    /users/1       → пользователь с id=1
// POST   /users         → создать пользователя
// PUT    /users/1       → обновить пользователя целиком
// PATCH  /users/1       → обновить пользователя частично
// DELETE /users/1       → удалить пользователя

// =========! Вложенные ресурсы =========!
// GET /users/1/posts    → посты пользователя 1
// POST /users/1/posts   → создать пост для пользователя 1

// =========! HTTP статусы в REST =========!
// 200 OK — успех
// 201 Created — ресурс создан
// 400 Bad Request — ошибка в запросе
// 401 Unauthorized — не авторизован
// 403 Forbidden — нет прав
// 404 Not Found — ресурс не найден
// 500 Internal Server Error — ошибка сервера
```

---

## 11. Коды ответов HTTP

```js
// ========== 1xx: Informational (информационные) ==========
// 100 Continue          — продолжай, сервер получил заголовки
// 101 Switching Protocols — смена протокола (WebSocket)
// 103 Early Hints       — предварительные подсказки

// ========== 2xx: Success (успех) ==========
// 200 OK                — успешный запрос
// 201 Created           — ресурс создан (POST)
// 202 Accepted          — запрос принят в обработку
// 204 No Content        — успех, но тело пустое (DELETE)

// ========== 3xx: Redirection (перенаправление) ==========
// 301 Moved Permanently — ресурс перемещён навсегда
// 302 Found             — временное перенаправление
// 304 Not Modified      — ресурс не изменился (кэш)

// ========== 4xx: Client Error (ошибка клиента) ==========
// 400 Bad Request       — неверный запрос
// 401 Unauthorized      — не авторизован (надо войти)
// 403 Forbidden         — нет прав доступа
// 404 Not Found         — ресурс не найден
// 405 Method Not Allowed — метод не поддерживается
// 409 Conflict          — конфликт (например, дубликат)
// 429 Too Many Requests — слишком много запросов (rate limit)

// ========== 5xx: Server Error (ошибка сервера) ==========
// 500 Internal Server Error — внутренняя ошибка сервера
// 501 Not Implemented      — метод не реализован
// 502 Bad Gateway          — плохой шлюз
// 503 Service Unavailable  — сервис недоступен
// 504 Gateway Timeout      — таймаут шлюза
```

---

## 12. WebSocket

```js
// WebSocket — протокол для полноценного двустороннего соединения
// Отличается от HTTP: соединение остаётся открытым

// ========== Как работает WebSocket ==========
// 1. Клиент отправляет HTTP-запрос на Upgrade
// 2. Сервер отвечает 101 Switching Protocols
// 3. Соединение переключается на WebSocket
// 4. Остаётся открытым для двусторонней передачи

// =========! Пример на клиенте =========!
const ws = new WebSocket("wss://example.com/socket");

// Открытие соединения
ws.onopen = () => {
  console.log("Соединение открыто");
  ws.send(JSON.stringify({ type: "message", data: "Привет!" }));
};

// Получение сообщения
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log("Получено:", data);
};

// Ошибка
ws.onerror = (error) => {
  console.error("Ошибка:", error);
};

// Закрытие соединения
ws.onclose = () => {
  console.log("Соединение закрыто");
};

// Отправить сообщение
ws.send(JSON.stringify({ text: "Привет" }));

// Закрыть соединение
ws.close();

// =========! WebSocket vs HTTP =========!
// HTTP: запрос → ответ → закрыть
// WebSocket: открыть → общаться → закрыть

// Когда нужен WebSocket:
// - Чаты
// - Онлайн-игры
// - Финансовые тикеры (биржевые котировки)
// - Совместное редактирование (Google Docs)
// - Уведомления в реальном времени
```

---

## 13. Способы реализации real-time приложений на фронте

```js
// ========== 1. WebSocket ==========
// - Полноценная двусторонняя связь
// - Одно соединение на всё время
// - Самый эффективный для real-time
// - Сложнее в реализации

// Пример: чаты, игры, биржевые котировки

// ========== 2. Server-Sent Events (SSE) ==========
// - Односторонняя связь (сервер → клиент)
// - Легче WebSocket
// - Автоматическое переподключение
// - Только текст

// Пример: уведомления, лента новостей, тикеры

const eventSource = new EventSource("/events");
eventSource.onmessage = (event) => {
  console.log("Новое событие:", event.data);
};

// ========== 3. Long Polling ==========
// - Клиент отправляет запрос, сервер держит его открытым
// - Когда есть данные — отвечает, клиент сразу отправляет новый запрос
// - Проще в реализации, но больше накладных расходов

async function longPolling() {
  const response = await fetch("/poll");
  const data = await response.json();
  console.log("Данные:", data);
  longPolling(); // сразу следующий запрос
}

// ========== 4. Short Polling ==========
// - Обычный периодический запрос (через setInterval)
// - Самый простой, но самый неэффективный

setInterval(async () => {
  const response = await fetch("/updates");
  const data = await response.json();
  console.log("Обновления:", data);
}, 1000); // каждую секунду

// =========! Сравнение =========!
// Метод           | Сложность | Эффективность | Двусторонняя | Авто-переподключение
// ----------------|-----------|---------------|--------------|---------------------
// WebSocket       | высокая   | высокая       | ✅           | ручное
// SSE             | средняя   | высокая       | ❌ (только→) | ✅
// Long Polling    | средняя   | средняя       | ❌           | ручное
// Short Polling   | низкая    | низкая        | ❌           | ❌

// =========! Что выбрать? =========!
// Чат / игра → WebSocket
// Уведомления с сервера → SSE
// Простое обновление (не частое) → Short/Long Polling
// Современный вариант → WebSocket + fallback на Long Polling
```

---

## Шпаргалка

### Копирование объектов

| Метод                  | Тип               | Обрабатывает                                             |
| ---------------------- | ----------------- | -------------------------------------------------------- |
| `{ ...obj }`           | shallow           | функции, даты, Map, Set, циклы                           |
| `Object.assign()`      | shallow           | то же самое                                              |
| `structuredClone()`    | deep              | ✅ всё (кроме функций, Symbol)                           |
| `JSON.parse/stringify` | deep (с нюансами) | ❌ функции, undefined, Symbol, даты → строки, NaN → null |

### HTTP статусы

| Диапазон | Назначение      |
| -------- | --------------- |
| 1xx      | Информационные  |
| 2xx      | Успех           |
| 3xx      | Перенаправление |
| 4xx      | Ошибка клиента  |
| 5xx      | Ошибка сервера  |

### Real-time методы

- **WebSocket** — двусторонняя, сложно, эффективно
- **SSE** — только от сервера, просто, эффективно
- **Long Polling** — эмуляция real-time, средняя эффективность
- **Short Polling** — просто, неэффективно

---
