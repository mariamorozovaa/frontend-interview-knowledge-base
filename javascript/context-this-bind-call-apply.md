# Контекст, this, bind/call/apply

---

## 1. Что такое контекст (this) в JavaScript?

**Ответ:**

Контекст (`this`) — это объект, в контексте которого выполняется функция.

⚠️ Важно: `this` определяется **в момент вызова функции**, а не её создания.

### Как определяется this:

- `obj.fn()` → `this = obj`
- `fn()` → `this = window` (или `undefined` в strict mode)
- стрелочная функция → берёт `this` из внешнего контекста
- `call/apply/bind` → задают `this явно`
- `new Fn()` → создаётся новый объект

---

## 2. На что ссылается this?

**Ответ:**

`this` указывает на объект, который вызвал функцию.

```js
console.log(this); // window (в браузере)

const obj = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
  greetArrow: () => {
    console.log(this.name);
  },
};

obj.greet(); // Alice
obj.greetArrow(); // undefined
```

---

## 3. Чем отличаются call, apply и bind?

**Ответ:**

### call

Вызывает функцию сразу, аргументы через запятую.

### apply

Вызывает функцию сразу, аргументы массивом.

### bind

Возвращает новую функцию с привязанным this.

```js
function greet(greeting) {
  console.log(greeting + ", " + this.name);
}

const user = { name: "Alice" };

greet.call(user, "Hi");
greet.apply(user, ["Hi"]);

const bound = greet.bind(user);
bound("Hi");
```

---

## 4. Что такое глобальный объект window?

**Ответ:**

`window` — глобальный объект в браузере.

Он содержит:

- глобальные функции (`setTimeout`, `alert`)
- глобальные переменные (`var`)
- объект `document`

```js
var a = 10;
console.log(window.a); // 10

let b = 20;
console.log(window.b); // undefined
```

---

## 5. Что такое document?

**Ответ:**

`document` — объект, который представляет HTML-страницу.

Используется для:

- поиска элементов
- создания элементов
- изменения DOM

```js
const btn = document.querySelector("button");
btn.textContent = "Click me";
```

---

## Что такое Execution Context (Контекст выполнения)?

Execution Context (контекст выполнения) — это внутренний объект, который создается движком JavaScript при выполнении кода.

Он содержит:

- информацию о переменных;
- информацию о функциях;
- значение `this`;
- ссылку на внешнее лексическое окружение (Lexical Environment).

Каждый раз при запуске программы или вызове функции создается новый Execution Context.

Можно представить его так:

```text
Execution Context

├── Lexical Environment
├── Variable Environment
├── this
└── Outer Environment
```

---

## Какие бывают Execution Context?

В JavaScript существует три основных вида контекста выполнения.

### 1. Global Execution Context

Создается один раз при запуске программы.

Содержит:

- глобальные переменные;
- глобальные функции;
- глобальный объект;
- глобальный `this`.

После завершения программы уничтожается.

---

### 2. Function Execution Context

Создается каждый раз при вызове функции.

```js
function foo() {}

foo();
foo();
```

Будет создано два разных Execution Context.

---

### 3. Eval Execution Context

Создается при использовании `eval()`.

Практически не используется.

---

## Что такое глобальный контекст?

Это первый Execution Context, который создается после запуска JavaScript.

В браузере:

```js
console.log(this);
```

Ответ:

```js
window;
```

В ES-модулях:

```js
this;
```

равен

```js
undefined;
```

В Node.js поведение отличается.

---

## Что такое this?

`this` — это специальное ключевое слово, которое содержит ссылку на объект, в контексте которого выполняется функция.

Важно помнить:

> `this` определяется **не в момент объявления функции**, а **в момент её вызова**.

Например:

```js
const user = {
  name: "Maria",

  say() {
    console.log(this.name);
  },
};

user.say();
```

Ответ:

```js
Maria;
```

Потому что функцию вызвал объект `user`.

---

## Какой контекст у стрелочных функций?

У стрелочных функций **нет собственного `this`**.

Они берут его из внешнего лексического окружения.

Например:

```js
const user = {
  name: "Maria",

  say() {
    const inner = () => {
      console.log(this.name);
    };

    inner();
  },
};

user.say();
```

Ответ:

```js
Maria;
```

Потому что стрелочная функция использует `this` метода `say`.

---

## В чем отличие event.target, event.currentTarget и this?

Очень популярный вопрос на собеседованиях.

Предположим:

```html
<div id="parent">
  <button>Click</button>
</div>
```

```js
parent.addEventListener("click", function (event) {
  console.log(event.target);
  console.log(event.currentTarget);
  console.log(this);
});
```

Если нажать на кнопку:

### event.target

Это элемент, по которому действительно кликнули.

```text
button
```

---

### event.currentTarget

Элемент, на котором висит обработчик.

```text
div
```

---

### this

В обычной функции

```js
this === event.currentTarget;
```

То есть тоже

```text
div
```

---

В стрелочной функции

```js
this;
```

не связан с обработчиком события.

Он берется из внешнего Scope.

---

## Когда использовать call, apply и bind?

### call()

Если нужно вызвать функцию сразу.

```js
fn.call(user, a, b);
```

---

### apply()

Если аргументы уже лежат в массиве.

```js
fn.apply(user, [a, b]);
```

---

### bind()

Если нужно получить новую функцию с привязанным `this`.

```js
const bound = fn.bind(user);

button.onclick = bound;
```

Именно поэтому `bind()` часто используется:

- в React Class Components;
- в setTimeout;
- в обработчиках событий.

---

## Как работают bind, call и apply?

Все три метода позволяют явно установить значение `this`.

Различия:

| Метод | Вызывает сразу | Возвращает новую функцию | Передача аргументов |
| ----- | -------------- | ------------------------ | ------------------- |
| call  | ✅             | ❌                       | Через запятую       |
| apply | ✅             | ❌                       | Массивом            |
| bind  | ❌             | ✅                       | Через запятую       |

---

## Что сказать на собеседовании

> Контекст выполнения (Execution Context) создается движком при запуске программы и при каждом вызове функции. Он содержит информацию о переменных, лексическом окружении и значении `this`. Само значение `this` определяется способом вызова функции: у обычных функций оно динамическое, а у стрелочных функций собственного `this` нет — они используют значение из внешнего лексического окружения. Методы `call`, `apply` и `bind` позволяют явно задать `this`: `call` и `apply` вызывают функцию сразу, а `bind` возвращает новую функцию с привязанным контекстом.

---

# 🧠 Практические задачи

---

## Задача 1. Обычная функция

```js
function sayHello() {
  console.log(this);
}

sayHello();
```

**Ответ:**
`window` (или `undefined` в strict mode)

---

## Задача 2. Стрелочная функция в объекте

```js
const user = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
  greet2: () => {
    console.log(this.name);
  },
};

user.greet();
user.greet2();
```

**Ответ:**

- greet → Alice
- greet2 → undefined

---

## Задача 3. Потеря контекста

```js
const user = {
  name: "Artem",
  getName() {
    console.log(this.name);
  },
};

setTimeout(user.getName, 1000);
```

**Ответ:**
undefined (теряется контекст)

### Исправления:

```js
setTimeout(user.getName.bind(user), 1000);
setTimeout(() => user.getName(), 1000);
```

---

## Задача 4. Контекст в объекте и вложенной функции

```js
const obj = {
  name: "Alice",
  show() {
    function inner() {
      console.log(this.name);
    }
    inner();
  },
};

obj.show();
```

**Ответ:**
undefined

---

## Задача 5. Исправление контекста

```js
const obj = {
  name: "Alice",
  show() {
    const inner = () => {
      console.log(this.name);
    };
    inner();
  },
};
```

**Ответ:**
Alice

---

## Задача 6. bind / call / apply

```js
function greet() {
  console.log(this.name);
}

greet.call({ name: "Alex" });
greet.apply({ name: "Alex" });

const bound = greet.bind({ name: "Alex" });
bound();
```

**Ответ:**
Alex во всех случаях

---

## Задача 7. new и this

```js
function Person(name) {
  this.name = name;
}

const p = new Person("Artem");
```

**Ответ:**
this → новый созданный объект

---

## Задача 8. Потеря контекста

```js
const obj = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
};

const fn = obj.greet;
fn();
```

**Ответ:**
undefined

---

# 📊 Шпаргалка this

| Вызов              | this               |
| ------------------ | ------------------ |
| fn()               | window / undefined |
| obj.fn()           | obj                |
| call/apply/bind    | заданный объект    |
| new Fn()           | новый объект       |
| стрелочная функция | внешний this       |
| setTimeout(fn)     | window             |

---

# 🚀 Итог

Контекст (`this`) — одна из самых частых тем на собеседованиях.

Главное помнить:

👉 this зависит НЕ от места объявления
👉 this зависит от способа вызова функции
