# База знаний JavaScript / TypeScript / Архитектура

## Вопросы и ответы для Senior Frontend

---

# 1. Что такое рекурсия?

### Ответ

Рекурсия — это техника программирования, при которой функция вызывает саму себя до тех пор, пока не выполнится условие остановки.

Рекурсия состоит из двух частей:

- Базовый случай (условие выхода)
- Рекурсивный вызов

### Пример

```js
function factorial(n) {
  if (n <= 1) return 1;

  return n * factorial(n - 1);
}

console.log(factorial(5)); // 120
```

### Как работает?

```text
factorial(5)
→ 5 * factorial(4)
→ 5 * 4 * factorial(3)
→ 5 * 4 * 3 * factorial(2)
→ 5 * 4 * 3 * 2 * factorial(1)
→ 120
```

### Минусы

- Может привести к переполнению стека вызовов (Stack Overflow)
- Иногда менее эффективна, чем цикл

---

# 2. Что такое каррирование?

### Ответ

Каррирование — это преобразование функции с несколькими аргументами в цепочку функций с одним аргументом.

### Без каррирования

```js
function sum(a, b, c) {
  return a + b + c;
}
```

### С каррированием

```js
function sum(a) {
  return function (b) {
    return function (c) {
      return a + b + c;
    };
  };
}

sum(1)(2)(3); // 6
```

### Зачем нужно?

- Частичное применение функций
- Создание переиспользуемых функций
- Функциональное программирование

```js
const multiply = (a) => (b) => a * b;

const double = multiply(2);

double(5); // 10
```

---

# 3. Что такое функция-генератор?

### Ответ

Генератор — специальная функция, которая может приостанавливать своё выполнение.

Объявляется через `function*`.

### Пример

```js
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
```

Результат:

```js
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
```

### Где используются?

- Создание итераторов
- Ленивые вычисления
- Обработка больших объемов данных

---

# 4. Как работает поиск переменной во внешнем scope?

### Ответ

При создании функции движок сохраняет ссылку на окружение, в котором она была создана.

Это называется:

```text
[[Environment]]
```

или

```text
Lexical Environment
```

### Пример

```js
function outer() {
  const x = 10;

  return function inner() {
    console.log(x);
  };
}

const fn = outer();

fn();
```

### Что происходит?

```text
inner не находит x внутри себя
↓
идет во внешний scope
↓
находит x = 10
```

Это называется:

```text
Scope Chain
```

---

# 5. Что значит ?? и || ?

## OR

Возвращает первое truthy значение.

```js
const value = "" || "default";

console.log(value);
```

Результат:

```js
"default";
```

### Считаются ложными

```js
false;
0;
("");
null;
undefined;
NaN;
```

---

## Nullish Coalescing

Работает только для:

```js
null;
undefined;
```

```js
const value = 0 ?? 100;

console.log(value);
```

Результат:

```js
0;
```

### Частый вопрос

```js
0 || 100;
```

Вернет:

```js
100;
```

А

```js
0 ?? 100;
```

Вернет:

```js
0;
```

---

# 6. Что означает obj!.property ?

### Ответ

Это оператор Non-null assertion в TypeScript.

Говорит компилятору:

```text
Я уверен, что здесь не null и не undefined.
```

### Пример

```ts
const element = document.getElementById("root")!;

element.innerHTML = "Hello";
```

Без `!`

```ts
Object is possibly null
```

### Важно

Во время выполнения ничего не проверяется.

```ts
obj!.test;
```

превращается в

```js
obj.test;
```

---

# 7. Как понять, что объект итерируемый?

### Ответ

Объект является итерируемым, если содержит:

```js
Symbol.iterator;
```

### Проверка

```js
const arr = [1, 2, 3];

console.log(Symbol.iterator in arr);
```

### Встроенные итерируемые объекты

```js
Array;
String;
Map;
Set;
NodeList;
```

### Создание своего итератора

```js
const obj = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  },
};

for (const value of obj) {
  console.log(value);
}
```

---

# 8. Что такое Set и Map?

## Set

Коллекция уникальных значений.

```js
const set = new Set();

set.add(1);
set.add(1);

console.log(set);
```

Результат:

```js
Set {1}
```

### Использование

Удаление дублей:

```js
const unique = [...new Set([1, 1, 2, 2])];
```

---

## Map

Коллекция пар:

```js
ключ → значение
```

### Пример

```js
const map = new Map();

map.set("name", "Maria");

console.log(map.get("name"));
```

### Преимущества

- Любой тип ключа
- Быстрее объекта при больших данных

---

# 9. Разница между Map и WeakMap

## Map

```js
const map = new Map();
```

Ключ может быть любым типом.

### Перебирается

```js
for (const item of map)
```

работает.

---

## WeakMap

```js
const weakMap = new WeakMap();
```

Ключ только объект.

```js
weakMap.set({}, "data");
```

### Особенность

Если объект удален из памяти:

```js
let obj = {};

weakMap.set(obj, "data");

obj = null;
```

Запись автоматически удалится.

### Используется

- Кэширование
- Приватные данные

---

# 10. Что такое Web API?

### Ответ

Web API — возможности браузера, которых нет в JavaScript.

Примеры:

```text
setTimeout
fetch
DOM
localStorage
WebSocket
Geolocation
```

### Event Loop

```text
Call Stack
↓
Web API
↓
Callback Queue
↓
Event Loop
↓
Call Stack
```

---

# 11. Какую проблему решает async/await?

### Ответ

Избавляет от Callback Hell и упрощает чтение асинхронного кода.

### Callback Hell

```js
getUser(function(user){
  getPosts(user,function(posts){
    getComments(posts,function(comments){
      ...
    })
  })
})
```

### Promise

```js
getUser().then(getPosts).then(getComments);
```

### Async Await

```js
const user = await getUser();
const posts = await getPosts(user);
const comments = await getComments(posts);
```

---

# 12. Всплытие и погружение событий

### Фазы события

```text
Capturing
↓
Target
↓
Bubbling
```

### Пример

```html
<div>
  <button></button>
</div>
```

### Bubbling

```js
button.click
↓
div
↓
body
↓
document
```

### Capturing

```js
document
↓
body
↓
div
↓
button
```

### Capture Listener

```js
element.addEventListener("click", handler, true);
```

---

# 13. LocalStorage, SessionStorage, Cookie

## LocalStorage

```text
Хранится бессрочно
≈5-10 МБ
```

Закрытие браузера не очищает.

---

## SessionStorage

```text
До закрытия вкладки
≈5 МБ
```

---

## Cookie

```text
≈4 КБ
```

Автоматически отправляются серверу.

Используются для:

```text
JWT
Сессии
Авторизация
```

---

# 14. Флаги HttpOnly и Secure

## HttpOnly

```text
Запрещает доступ через JS
```

```js
document.cookie;
```

не увидит cookie.

Защита от XSS.

---

## Secure

```text
Передавать только по HTTPS
```

---

# 15. Что такое Web Workers?

### Ответ

Отдельный поток браузера для тяжелых вычислений.

### Без Worker

```text
UI зависает
```

### С Worker

```text
Вычисления отдельно
UI работает
```

### Пример

```js
const worker = new Worker("worker.js");

worker.postMessage(data);

worker.onmessage = (event) => {
  console.log(event.data);
};
```

---

# 16. Что такое Shared Worker?

### Ответ

Worker, который может использоваться несколькими вкладками одновременно.

### Обычный Worker

```text
1 вкладка
=
1 worker
```

### SharedWorker

```text
Много вкладок
=
1 worker
```

### Использование

- Чаты
- WebSocket соединения
- Общий кэш

---

# 17. Дескрипторы свойств объекта

### Получение

```js
Object.getOwnPropertyDescriptor(obj, "name");
```

### Основные флаги

```js
writable;
enumerable;
configurable;
value;
```

### Пример

```js
Object.defineProperty(obj, "name", {
  value: "Maria",
  writable: false,
});
```

---

# 18. Паттерны проектирования в React

## Container / Presentational

Разделение логики и UI.

---

## HOC

```js
withAuth(Component);
```

---

## Render Props

```jsx
<DataProvider>{(data) => <List data={data} />}</DataProvider>
```

---

## Custom Hooks

```js
useUsers();
```

Самый популярный паттерн сегодня.

---

## Observer

Redux, Zustand.

---

## Factory

```js
createButton(type);
```

---

## Singleton

```js
ApiClient.getInstance();
```

---

# 19. Что такое ООП?

### Ответ

Объектно-ориентированное программирование — подход, при котором программа строится из объектов.

Объект объединяет:

```text
данные
+
поведение
```

---

# 20. Принципы ООП

## Инкапсуляция

Скрытие внутренней реализации.

```js
class User {
  #password;
}
```

---

## Наследование

```js
class Admin extends User {}
```

---

## Полиморфизм

```js
animal.makeSound();
```

Разные реализации метода.

---

## Абстракция

Показываем важное, скрываем лишнее.

---

# 21. Зачем нужны принципы программирования?

### Ответ

Чтобы код был:

- читаемым
- масштабируемым
- поддерживаемым
- тестируемым

Без принципов проекты быстро превращаются в Legacy.

---

# 22. Что такое DRY?

### Don't Repeat Yourself

Не дублируй код.

Плохо:

```js
fetchUsers();
fetchProducts();
fetchOrders();
```

Одинаковая логика внутри.

Хорошо:

```js
request(url);
```

---

# 23. Что такое KISS?

### Keep It Simple, Stupid

Не усложняй решение.

Плохо:

```js
FactoryBuilderManagerProxy;
```

для одной кнопки.

Хорошо:

```js
function Button() {}
```

---

# 24. Что такое SOLID?

## S — Single Responsibility Principle

Один класс = одна ответственность.

---

## O — Open Closed Principle

Открыт для расширения.

Закрыт для изменения.

---

## L — Liskov Substitution Principle

Наследник должен заменять родителя без ошибок.

---

## I — Interface Segregation Principle

Лучше много маленьких интерфейсов.

---

## D — Dependency Inversion Principle

Зависимость от абстракций, а не реализаций.

### Плохой пример

```js
class UserService {
  constructor() {
    this.db = new PostgreSQL();
  }
}
```

### Хороший пример

```js
class UserService {
  constructor(database) {
    this.db = database;
  }
}
```

Можно подставить:

```js
PostgreSQL;
MongoDB;
MockDatabase;
```

---
