# Classes in JavaScript

## Теория (вопросы и ответы)

---

## 3. Что такое классы в JS, зачем они были добавлены?

Класс в JavaScript — это **синтаксическая обёртка над прототипным наследованием**.

---

### Пример класса

```js id="k9v2q7"
class User {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    console.log(this.name);
  }
}
```

---

### Зачем добавили классы?

До ES6 работа с объектами и наследованием выглядела сложнее (через prototype).

Классы добавили для:

- упрощения синтаксиса

- улучшения читаемости

- удобства ООП-подхода

- более привычной модели (как в Java, C#, etc.)

---

### Важно

Классы в JS — это НЕ новая модель ООП.

Это всё тот же prototype под капотом.

---

## 4. Как можно инициализировать переменные внутри класса?

---

### 1. Через constructor

Самый базовый способ.

```js id="c1m8tq"
class User {
  constructor(name, age) {
    this.name = name;

    this.age = age;
  }
}
```

---

### 2. Поля класса (class fields)

Современный синтаксис.

```js id="v4p8wq"
class User {
  name = "Default";

  age = 0;
}
```

---

### 3. Инициализация через методы

```js id="n8q2ld"
class User {
  constructor(name) {
    this.init(name);
  }

  init(name) {
    this.name = name;
  }
}
```

---

### 4. Через приватные поля

```js id="h3k9sd"
class User {
  #name;

  constructor(name) {
    this.#name = name;
  }
}
```

---

## 5. Что такое статичные методы у класса?

Static методы — это методы, которые принадлежат **самому классу**, а не его экземпляру.

---

### Пример

```js id="s7d2pq"
class MathUtils {
  static sum(a, b) {
    return a + b;
  }
}

MathUtils.sum(2, 3); // 5
```

---

### Особенность

```js
const obj = new MathUtils();

obj.sum(); // ❌ ошибка
```

---

### Где используются static методы?

- утилиты (helpers)

- фабрики объектов

- парсинг данных

- валидация

- логика, не зависящая от состояния экземпляра

---

## 6. Что если в классе, который расширяет родительский, внутри конструктора не указать super()?

---

### Короткий ответ

Будет **ошибка выполнения**.

---

### Пример

```js id="m8q1dp"
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name) {
    this.name = name; // ❌ ошибка
  }
}
```

---

### Ошибка

```text

ReferenceError: Must call super constructor in derived class before accessing 'this'

```

---

### Почему так происходит?

В наследуемом классе (`extends`) `this` не существует до вызова `super()`.

---

### Правильный вариант

```js id="r1x8qp"
class Dog extends Animal {
  constructor(name) {
    super(name);

    this.breed = "husky";
  }
}
```

---

## 7. Как проверить, из какого класса был создан объект (instance of)?

Используется оператор **instanceof**.

---

### Пример

```js id="t5k9qz"
class Animal {}

class Dog extends Animal {}

const dog = new Dog();

console.log(dog instanceof Dog); // true

console.log(dog instanceof Animal); // true
```

---

### Как работает?

Проверяет цепочку прототипов.

```text

dog → Dog.prototype → Animal.prototype → Object.prototype

```

---

### Альтернатива

```js
Object.getPrototypeOf(obj);
```

(но `instanceof` — основной способ)

---

## 8. Можно ли из конструктора в классе сделать асинхронную функцию?

---

### Короткий ответ

❌ Нельзя сделать `constructor` async.

---

### Почему?

Конструктор обязан **сразу вернуть объект**, а `async` всегда возвращает Promise.

---

### Неправильно

```js id="p3x7ld"

class User {

    async constructor() {} // ❌ SyntaxError

}

```

---

### Как делают правильно?

---

### 1. async init() метод

```js id="q9m2sd"
class User {
  constructor() {
    this.ready = this.init();
  }

  async init() {
    this.data = await fetch("/api/user");
  }
}
```

---

### 2. Фабричный async метод

```js id="z7k3lp"
class User {
  constructor(data) {
    this.data = data;
  }

  static async create() {
    const data = await fetch("/api/user");

    return new User(data);
  }
}

const user = await User.create();
```

---

### 3. Promise внутри конструктора (редко, не рекомендуется)

```js id="w2k8mn"
class User {
  constructor() {
    this.ready = new Promise(async (resolve) => {
      this.data = await fetch("/api/user");

      resolve(this);
    });
  }
}
```

---

## Что сказать на собеседовании

Классы в JavaScript — это синтаксический сахар над прототипами, введённый для упрощения работы с объектно-ориентированным стилем программирования.
Переменные внутри класса можно инициализировать через constructor, class fields и методы.
Static методы принадлежат самому классу и используются как утилиты.
При наследовании обязательно нужно вызывать super(), иначе доступ к this вызовет ошибку.
Проверка принадлежности объекта классу выполняется через instanceof, который проверяет цепочку прототипов.
Constructor не может быть async, поэтому асинхронную инициализацию выносят в отдельные методы или фабрики.
