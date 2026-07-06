# Functions in JavaScript

## Теория (вопросы и ответы)

---

## Function Declaration и Function Expression — что это такое и в чем разница?

### Function Declaration

Это объявление функции.

```js
function sum(a, b) {
  return a + b;
}
```

Главная особенность — функция полностью поднимается (Hoisting).

Поэтому ее можно вызвать до объявления.

```js
sayHi();

function sayHi() {
  console.log("Hello");
}
```

Работает без ошибок.

---

### Function Expression

Функция присваивается переменной.

```js
const sum = function (a, b) {
  return a + b;
};
```

Вызвать раньше нельзя.

```js
sayHi();

const sayHi = function () {};
```

Получим

```text
ReferenceError
```

---

### Основные отличия

| Function Declaration        | Function Expression        |
| --------------------------- | -------------------------- |
| Поднимается полностью       | Не поднимается как функция |
| Можно вызвать до объявления | Нельзя вызвать раньше      |
| Есть собственное имя        | Может быть анонимной       |

---

## Что такое функция высшего порядка (Higher-Order Function)?

Это функция, которая:

- принимает другую функцию как аргумент;
- или возвращает функцию.

Например:

```js
const numbers = [1, 2, 3];

numbers.map((n) => n * 2);
```

`map` — функция высшего порядка.

---

Еще пример

```js
function greet(name) {
  return function () {
    console.log(`Hello ${name}`);
  };
}
```

---

Самые популярные HOF:

- map
- filter
- reduce
- find
- some
- every
- sort
- forEach

---

## Что такое чистая функция (Pure Function)?

Чистая функция:

- всегда возвращает одинаковый результат при одинаковых аргументах;
- не имеет побочных эффектов.

Пример:

```js
function sum(a, b) {
  return a + b;
}
```

---

Нечистая функция

```js
let count = 0;

function increment() {
  count++;
}
```

Она изменяет внешнее состояние.

---

Побочными эффектами считаются:

- изменение внешних переменных;
- запросы к серверу;
- запись в localStorage;
- изменение DOM;
- console.log();
- Math.random();
- Date.now().

---

## Что такое замыкание (Closure)?

Замыкание — это функция, которая "помнит" свое внешнее лексическое окружение даже после завершения внешней функции.

Пример:

```js
function counter() {
  let count = 0;

  return function () {
    return ++count;
  };
}

const inc = counter();

console.log(inc()); // 1
console.log(inc()); // 2
console.log(inc()); // 3
```

Почему работает?

Потому что внутренняя функция хранит ссылку на `[[Environment]]`.

---

Замыкания используются для:

- инкапсуляции;
- приватных переменных;
- мемоизации;
- debounce;
- throttle;
- React Hooks.

---

## Что такое IIFE?

Immediately Invoked Function Expression.

Функция, которая вызывается сразу после создания.

```js
(function () {
  console.log("Hello");
})();
```

или

```js
(() => {
  console.log("Hello");
})();
```

---

Раньше использовалась для создания собственной области видимости.

После появления `let`, `const` и ES Modules используется значительно реже.

---

## Чем отличаются стрелочные функции от обычных?

### 1. Нет собственного `this`

```js
const obj = {
  name: "Maria",

  say() {
    console.log(this.name);
  },
};
```

Обычная функция получает собственный `this`.

Стрелочная — берет его из внешней области видимости.

---

### 2. Нет `arguments`

```js
const fn = () => {
  console.log(arguments);
};
```

Получим ошибку.

Используют Rest Parameters.

```js
(...args)
```

---

### 3. Нельзя использовать как конструктор

```js
new (() => {})();
```

Ошибка.

---

### 4. Нет собственного `prototype`

Поэтому невозможно создать экземпляр.

---

### 5. Нельзя использовать как генератор

```js
function* gen() {}
```

Стрелочная функция такого синтаксиса не поддерживает.

---

## Что такое каррирование?

Каррирование — преобразование функции с несколькими аргументами в цепочку функций с одним аргументом.

Обычная функция:

```js
sum(2, 3);
```

Каррированная:

```js
sum(2)(3);
```

Пример:

```js
function sum(a) {
  return function (b) {
    return a + b;
  };
}

sum(2)(3);
```

Ответ:

```js
5;
```

Используется для:

- частичного применения функций;
- создания переиспользуемых функций.

---

## Как реализовать бесконечную цепочку функций?

Самый популярный вариант — использовать замыкание.

```js
function sum(a) {
  return function (b) {
    return sum(a + b);
  };
}
```

Для получения результата обычно переопределяют `valueOf` или `toString`.

Например:

```js
function sum(a) {
  const fn = (b) => sum(a + b);

  fn.valueOf = () => a;

  return fn;
}

Number(sum(1)(2)(3));
```

Ответ:

```js
6;
```

---

## Что такое функции-конструкторы?

До появления ES6 классов объекты создавались через функции.

```js
function User(name) {
  this.name = name;
}

const user = new User("Maria");
```

Что делает `new`?

1. Создает новый объект.
2. Привязывает `this`.
3. Связывает объект с `prototype`.
4. Возвращает объект.

---

Добавление методов

```js
User.prototype.sayHi = function () {
  console.log(this.name);
};
```

---

## Что такое функция-генератор?

Генератор позволяет приостанавливать выполнение функции.

Объявляется через

```js
function* generator() {}
```

Пример:

```js
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numbers();

gen.next();
```

Ответ:

```js
{
    value: 1,
    done: false,
}
```

Следующий вызов

```js
gen.next();
```

Ответ:

```js
{
    value: 2,
    done: false,
}
```

---

Генераторы используются:

- для ленивых вычислений;
- обработки больших данных;
- создания собственных итераторов.

---

## Наследование в функциях и классах. Почему классы проще?

### До ES6

Использовались функции-конструкторы.

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.say = function () {
  console.log(this.name);
};

function Dog(name) {
  Animal.call(this, name);
}

Dog.prototype = Object.create(Animal.prototype);

Dog.prototype.constructor = Dog;
```

Такой код довольно сложный.

---

### ES6

Появились классы.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  say() {
    console.log(this.name);
  }
}

class Dog extends Animal {}
```

Все выглядит значительно проще.

---

Важно понимать:

Классы **не являются новым механизмом наследования**.

Это синтаксический сахар над прототипным наследованием.

Под капотом все равно используется `prototype`.

---

## Практические вопросы собеседований

### Что выведет код?

```js
sayHi();

function sayHi() {
  console.log("Hello");
}
```

Ответ:

```text
Hello
```

---

### Что выведет код?

```js
const sum = (a) => (b) => a + b;

console.log(sum(2)(3));
```

Ответ:

```js
5;
```

---

### Что выведет код?

```js
function counter() {
  let count = 0;

  return () => ++count;
}

const inc = counter();

console.log(inc());
console.log(inc());
```

Ответ:

```js
1;
2;
```

---

### Что выведет код?

```js
function User(name) {
  this.name = name;
}

const user = new User("Maria");

console.log(user.name);
```

Ответ:

```js
"Maria";
```

---

### Что выведет код?

```js
function* gen() {
  yield 1;
  yield 2;
}

const g = gen();

console.log(g.next());
```

Ответ:

```js
{
    value: 1,
    done: false,
}
```

---

## Что сказать на собеседовании

> В JavaScript функции являются объектами первого класса. Они могут передаваться как аргументы, возвращаться из других функций и храниться в переменных. Function Declaration отличается от Function Expression механизмом hoisting. Замыкания позволяют функциям сохранять доступ к внешнему лексическому окружению, что лежит в основе каррирования, мемоизации и многих современных паттернов. ES6-классы являются синтаксическим сахаром над прототипным наследованием и упрощают создание иерархий объектов по сравнению с функциями-конструкторами.
