````md
# Контекст, this, bind/call/apply

---

# ❓ Вопросы и ответы

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
````

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

```

```
