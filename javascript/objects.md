# Objects in JavaScript

## Теория (вопросы и ответы)

---

## 1. Какие способы создания объекта вы знаете?

В JavaScript есть несколько способов создания объектов.

---

### Object literal (литерал объекта)

Самый частый и базовый способ.

```js
const user = {
  name: "Maria",

  age: 25,
};
```

---

### new Object()

Создание через конструктор Object.

```js
const user = new Object();

user.name = "Maria";

user.age = 25;
```

---

### Object.create()

Создаёт объект с заданным прототипом.

```js
const proto = {
  greet() {
    console.log("Hello");
  },
};

const user = Object.create(proto);

user.name = "Maria";
```

---

### Функция-конструктор

```js
function User(name) {
  this.name = name;
}

const user = new User("Maria");
```

---

### class (ES6)

```js
class User {
  constructor(name) {
    this.name = name;
  }
}

const user = new User("Maria");
```

---

### Object.fromEntries()

```js
const entries = [
  ["name", "Maria"],

  ["age", 25],
];

const user = Object.fromEntries(entries);
```

---

## 2. Как можно защитить данные объекта от изменений?

---

### Object.freeze()

Полностью запрещает изменения объекта.

```js
const user = {
  name: "Maria",
};

Object.freeze(user);

user.name = "Anna"; // не изменится
```

---

⚠️ Только shallow freeze (вложенные объекты не защищены)

---

### Object.seal()

Можно менять свойства, но нельзя добавлять/удалять.

```js
const user = {
  name: "Maria",
};

Object.seal(user);

user.name = "Anna"; // можно

delete user.name; // нельзя
```

---

### defineProperty()

```js
const user = {};

Object.defineProperty(user, "name", {
  value: "Maria",

  writable: false,
});

user.name = "Anna";
```

---

### Deep freeze (рекурсивно)

```js
function deepFreeze(obj) {
  Object.keys(obj).forEach((key) => {
    if (typeof obj[key] === "object" && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });

  return Object.freeze(obj);
}
```

---

## 3. Как можно копировать объекты?

---

### Shallow copy

#### Object.assign()

```js
const copy = Object.assign({}, user);
```

---

#### spread

```js
const copy = { ...user };
```

---

⚠️ Вложенные объекты копируются по ссылке

---

### Deep copy

#### JSON способ

```js
const copy = JSON.parse(JSON.stringify(user));
```

---

❗ Минусы:

- функции теряются

- Date ломается

- Map/Set не поддерживаются

---

#### structuredClone()

```js
const copy = structuredClone(user);
```

---

#### рекурсивный clone

```js
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;

  const copy = Array.isArray(obj) ? [] : {};

  for (let key in obj) {
    copy[key] = deepClone(obj[key]);
  }

  return copy;
}
```

---

## 4. Какие есть дескрипторы у полей?

У каждого свойства есть дескрипторы.

---

### value

значение свойства

---

### writable

можно ли менять

```js
writable: true;
```

---

### enumerable

участвует ли в циклах

---

### configurable

можно ли удалить/изменить

---

### пример

```js
const user = {};

Object.defineProperty(user, "name", {
  value: "Maria",

  writable: false,

  enumerable: true,

  configurable: false,
});
```

---

### просмотр

```js
Object.getOwnPropertyDescriptor(user, "name");
```

---

## 5. Как определить наличие свойства?

---

### in

```js
"name" in user;
```

проверяет и прототип

---

### hasOwnProperty()

```js
user.hasOwnProperty("name");
```

только свои свойства

---

### Object.hasOwn()

```js
Object.hasOwn(user, "name");
```

---

### таблица

| способ | прототип | own |

|--------|----------|-----|

| in | да | да |

| hasOwnProperty | нет | да |

| Object.hasOwn | нет | да |

---

## 6. Зачем нужны get и set?

---

### get

```js
const user = {
  firstName: "Maria",

  lastName: "Ivanova",

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
};
```

---

### set

```js
const user = {
  set name(value) {
    this._name = value.trim();
  },

  get name() {
    return this._name;
  },
};
```

---

### зачем нужны

- контроль данных

- валидация

- вычисляемые поля

- инкапсуляция

- реактивность

---

## Что сказать на собеседовании

Объекты в JavaScript создаются через литералы, классы, функции-конструкторы и Object.create.
Для защиты данных используются freeze, seal и дескрипторы свойств.
Копирование бывает shallow и deep, включая structuredClone.
Проверка свойств делается через in, hasOwnProperty и Object.hasOwn.
Getters и setters позволяют контролировать доступ к данным и добавлять логику при чтении и записи.
