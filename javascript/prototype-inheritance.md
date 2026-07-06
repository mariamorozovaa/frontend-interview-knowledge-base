# Прототипное наследование, prototype, **proto**, instanceof

## Вопросы с собеседования

### 1. Что такое прототипное наследование в JS?

```js
// Прототипное наследование — механизм, при котором объекты могут получать доступ
// к свойствам и методам других объектов через цепочку прототипов (Prototype Chain).

// Каждый объект имеет скрытую ссылку [[Prototype]] на другой объект.
// Если свойство не найдено в самом объекте, JS ищет его в прототипе,
// потом в прототипе прототипа и так до null.

// Простой пример:
const parent = {
  name: "Родитель",
  greet() {
    console.log(`Привет, я ${this.name}`);
  },
};

const child = {
  name: "Ребенок",
};

// Устанавливаем прототип
Object.setPrototypeOf(child, parent);

child.greet(); // "Привет, я Ребенок" (метод взят из parent)
console.log(child.name); // "Ребенок" (свойство из самого объекта)

// Цепочка прототипов:
// child → parent → Object.prototype → null
```

### 2. Что такое **proto** и на что он ссылается?

```js
// __proto__ — это геттер/сеттер, который дает доступ к внутреннему свойству [[Prototype]]
// (устаревший, но до сих пор используется)

const obj = { x: 10 };
console.log(obj.__proto__ === Object.prototype); // true
console.log(obj.__proto__.__proto__); // null

// Современный способ:
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true
console.log(Object.getPrototypeOf(obj).__proto__); // null

// Пример цепочки:
const arr = [];
console.log(arr.__proto__ === Array.prototype); // true
console.log(arr.__proto__.__proto__ === Object.prototype); // true
console.log(arr.__proto__.__proto__.__proto__); // null

// arr → Array.prototype → Object.prototype → null
```

### 3. Как понять, какие методы наследуются, а какие нет?

```js
// Наследуются ВСЕ методы из прототипа, если они не переопределены в самом объекте.
// Проверить, есть ли свойство у самого объекта:

const obj = { a: 1 };

// hasOwnProperty — проверяет ТОЛЬКО свои свойства
console.log(obj.hasOwnProperty("a")); // true (свое)
console.log(obj.hasOwnProperty("toString")); // false (из прототипа)

// Оператор in — проверяет свои + унаследованные
console.log("a" in obj); // true
console.log("toString" in obj); // true (из прототипа)

// Практический пример:
const animal = {
  eat() {
    console.log("Ем");
  },
};
const dog = Object.create(animal);
dog.bark = function () {
  console.log("Гав!");
};

console.log(dog.hasOwnProperty("bark")); // true (свой)
console.log(dog.hasOwnProperty("eat")); // false (унаследован)
console.log("eat" in dog); // true (есть в цепочке)
```

### 4. Как создать объект без прототипа?

```js
// Используем Object.create(null) — объект без прототипа

const emptyObj = Object.create(null);

// У него нет toString, hasOwnProperty, __proto__ и т.д.
console.log(emptyObj.toString); // undefined
console.log(emptyObj.__proto__); // undefined

// Можно использовать как чистую хеш-таблицу без конфликтов
const dict = Object.create(null);
dict["key"] = "value";
dict["toString"] = "не сломается";

console.log(dict["toString"]); // "не сломается"

// Сравнение с обычным объектом:
const normalObj = {};
console.log(normalObj.toString); // ƒ toString() { [native code] }
```

### 5. Для чего используется instanceof?

```js
// instanceof проверяет, есть ли прототип конструктора в цепочке прототипов объекта

function Animal(name) {
  this.name = name;
}
function Dog(name) {
  this.name = name;
}
Dog.prototype = Object.create(Animal.prototype);

const rex = new Dog("Рекс");

console.log(rex instanceof Dog); // true
console.log(rex instanceof Animal); // true (Dog.prototype → Animal.prototype)
console.log(rex instanceof Object); // true

// instanceof работает только с объектами
console.log([] instanceof Array); // true
console.log([] instanceof Object); // true
console.log({} instanceof Array); // false

// Важный момент: instanceof не работает при смене прототипа
function Person() {}
const alice = new Person();
Person.prototype = {};
console.log(alice instanceof Person); // false (прототип изменился)

// Ручная реализация instanceof:
function myInstanceof(obj, Constructor) {
  let proto = Object.getPrototypeOf(obj);
  while (proto) {
    if (proto === Constructor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}

console.log(myInstanceof(rex, Dog)); // true
```

### 6. Как посмотреть цепочку прототипов в консоли?

```js
// Способ 1: console.dir() в браузере
const obj = { a: 1 };
console.dir(obj); // Раскройте треугольник → [[Prototype]]

// Способ 2: через код
function logPrototypeChain(obj) {
  let current = obj;
  let level = 0;
  while (current) {
    console.log(`${"  ".repeat(level)}→`, current);
    current = Object.getPrototypeOf(current);
    level++;
  }
  console.log(`${"  ".repeat(level)}→ null`);
}

// Пример для массива:
logPrototypeChain([1, 2, 3]);
// → [1, 2, 3]
//   → Array.prototype
//     → Object.prototype
//       → null

// Пример для строки:
logPrototypeChain("hello");
// → String {"hello"}
//   → String.prototype
//     → Object.prototype
//       → null
```

---

## Практические задачи с решениями

### Задача 1. Добавить метод last() в Array

```js
// Добавляем метод last() в прототип Array
// Теперь он будет доступен у всех массивов

Array.prototype.last = function () {
  // this указывает на массив, у которого вызван метод
  if (this.length === 0) return undefined;
  return this[this.length - 1];
};

// Проверка:
const numbers = [1, 2, 3, 4, 5];
console.log(numbers.last()); // 5

const empty = [];
console.log(empty.last()); // undefined

// Важно: не используйте стрелочные функции для методов прототипа,
// так как у стрелочных функций нет своего this
```

### Задача 2. Добавить метод sum() в Array

```js
// Добавляем метод sum(), возвращающий сумму всех элементов

Array.prototype.sum = function () {
  // reduce суммирует все элементы
  return this.reduce((acc, value) => acc + value, 0);
};

// Проверка:
const numbers2 = [1, 2, 3, 4];
console.log(numbers2.sum()); // 10

const empty2 = [];
console.log(empty2.sum()); // 0 (пустой массив → 0)

// Также работает с разными типами чисел:
console.log([1, 2, 3.5, 4].sum()); // 10.5
```

### Задача 3. Кастомная функция isArray()

```js
// Реализуем аналог Array.isArray()

function isArray(value) {
  // 1 способ: instanceof
  // return value instanceof Array;

  // 2 способ: проверка через Object.prototype.toString
  // return Object.prototype.toString.call(value) === "[object Array]";

  // 3 способ: проверка конструктора
  // return value && value.constructor === Array;

  // Полная версия с обработкой всех случаев:
  if (value === null || value === undefined) return false;
  return value instanceof Array;
}

// Проверка:
console.log(isArray([1, 2, 3])); // true
console.log(isArray("строка")); // false
console.log(isArray({ a: 1 })); // false
console.log(isArray([])); // true
console.log(isArray(null)); // false
console.log(isArray(undefined)); // false
console.log(isArray(new Array())); // true
```

### Задача 4. Кастомная функция myCustomMap

```js
// Реализуем свой .map() без использования встроенного

Array.prototype.myCustomMap = function (callback) {
  const result = [];

  // this — массив, для которого вызван метод
  for (let i = 0; i < this.length; i++) {
    // Вызываем callback с элементом, индексом и самим массивом
    result.push(callback(this[i], i, this));
  }

  return result;
};

// Проверка:
const multiplyBy2 = (item) => item * 2;
console.log([1, 2, 3, 4].myCustomMap(multiplyBy2)); // [2, 4, 6, 8]

const withIndex = (item, index) => `${index}:${item}`;
console.log([1, 2, 3, 4].myCustomMap(withIndex));
// ["0:1", "1:2", "2:3", "3:4"]

// Проверка, что не изменяет исходный массив:
const original = [10, 20, 30];
const mapped = original.myCustomMap((x) => x / 10);
console.log(original); // [10, 20, 30] (не изменился)
console.log(mapped); // [1, 2, 3]
```

---

## 1. Что такое прототип?

Прототип — это **обычный объект**, от которого другие объекты могут наследовать свойства и методы.

Если свойство не найдено в объекте, JavaScript ищет его в прототипе по цепочке.

```js
const animal = {
  eats: true,
};

const dog = Object.create(animal);

console.log(dog.eats); // true (взято из прототипа)
```

---

## 2. Что такое [[Prototype]]?

`[[Prototype]]` — это **внутреннее скрытое свойство объекта**, которое указывает на его прототип.

Оно недоступно напрямую, но используется движком JS для поиска свойств.

```js
const obj = {};

console.log(Object.getPrototypeOf(obj));
```

Фактически:

```text

obj.[[Prototype]] → Object.prototype

```

---

## 3. Что такое **proto**?

`__proto__` — это **геттер/сеттер для доступа к [[Prototype]]**.

Он устаревший, но всё ещё поддерживается.

```js
const obj = {};

console.log(obj.__proto__ === Object.prototype); // true
```

Современный вариант:

```js
Object.getPrototypeOf(obj);

Object.setPrototypeOf(obj, someProto);
```

---

## 4. Что попадает в **proto**?

В `__proto__` попадает **ссылка на прототип объекта**.

То есть туда записывается:

- Object.prototype (для обычных объектов)

- Array.prototype (для массивов)

- Function.prototype (для функций)

- либо null (если нет прототипа)

```js
const arr = [];

console.log(arr.__proto__ === Array.prototype); // true
```

Важно:

`__proto__` НЕ содержит собственные свойства объекта — только прототип.

---

## 5. Как создать объект без наследования от какого-либо prototype?

Используется:

```js
Object.create(null);
```

Такой объект:

- не имеет prototype

- не имеет методов Object.prototype

- не имеет **proto**

```js
const dict = Object.create(null);

console.log(dict.toString); // undefined

console.log(dict.__proto__); // undefined
```

Используется как безопасный словарь (map).

---

## 6. Насколько глубоким может быть наследование?

Теоретически — **ограничено только движком JavaScript**, но на практике:

- цепочка прототипов может быть любой длины

- но слишком длинная цепочка = плохая производительность

---

### Пример глубокой цепочки

```js
const a = {};

const b = Object.create(a);

const c = Object.create(b);

const d = Object.create(c);

console.log(d.a); // поиск пойдёт по всей цепочке
```

---

### Ограничение

В конце всегда:

```text

... → Object.prototype → null

```

---

### Важно для собеседования

- длинная цепочка = медленный lookup свойств

- лучше избегать глубокой иерархии

- предпочтительнее композиция вместо наследования

---

## Что сказать на собеседовании

Прототип — это объект, от которого наследуются свойства.
[[Prototype]] — внутренняя ссылка на этот объект, а **proto** — способ доступа к ней.
Если объект не находит свойство, он ищет его по цепочке прототипов до null.
Object.create(null) позволяет создать объект без прототипа.
Глубина прототипного наследования теоретически не ограничена, но на практике длинные цепочки ухудшают производительность, поэтому предпочтительна композиция.

---

## DOM-задача: Подсветка активного пункта списка

**HTML:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Активный пункт меню</title>
    <style>
      .nav-item {
        padding: 10px 20px;
        cursor: pointer;
        list-style: none;
        transition: background 0.3s;
      }
      .nav-item:hover {
        background-color: #e0e0e0;
      }
      .active {
        background-color: lightblue;
        font-weight: bold;
      }
    </style>
  </head>
  <body>
    <ul id="nav">
      <li class="nav-item">Главная</li>
      <li class="nav-item">О нас</li>
      <li class="nav-item">Контакты</li>
    </ul>

    <script>
      // Вариант 1: через querySelectorAll и forEach
      const nav = document.querySelector("#nav");
      const items = document.querySelectorAll(".nav-item");

      items.forEach((item) => {
        item.addEventListener("click", () => {
          // Удаляем active у всех
          items.forEach((li) => li.classList.remove("active"));
          // Добавляем active текущему
          item.classList.add("active");
        });
      });

      // Вариант 2: через делегирование событий (более эффективно)
      // nav.addEventListener("click", (e) => {
      //     const target = e.target.closest(".nav-item");
      //     if (!target) return;

      //     items.forEach(item => item.classList.remove("active"));
      //     target.classList.add("active");
      // });
    </script>
  </body>
</html>
```

**CSS:**

```css
.active {
  background-color: lightblue;
  font-weight: bold;
}
.nav-item {
  padding: 10px 20px;
  cursor: pointer;
  list-style: none;
  transition: background 0.3s;
}
.nav-item:hover {
  background-color: #e0e0e0;
}
```

---

## Шпаргалка по прототипам

| Концепция                 | Описание                                         | Пример                              |
| ------------------------- | ------------------------------------------------ | ----------------------------------- |
| `[[Prototype]]`           | Внутренняя ссылка на прототип                    | `Object.getPrototypeOf(obj)`        |
| `__proto__`               | Геттер/сеттер для прототипа (устаревший)         | `obj.__proto__`                     |
| `prototype`               | Свойство функции-конструктора                    | `Array.prototype`                   |
| `Object.create(proto)`    | Создает объект с указанным прототипом            | `Object.create(null)`               |
| `hasOwnProperty()`        | Проверяет СВОЕ свойство                          | `obj.hasOwnProperty("key")`         |
| `in`                      | Проверяет свое + унаследованное                  | `"key" in obj`                      |
| `instanceof`              | Проверяет прототип в цепочке                     | `[] instanceof Array`               |
| `Object.getPrototypeOf()` | Получить прототип (современный)                  | `Object.getPrototypeOf(obj)`        |
| `Object.setPrototypeOf()` | Установить прототип (медленно, не рекомендуется) | `Object.setPrototypeOf(obj, proto)` |

---

## Цепочка прототипов для разных типов

```js
// Массив:
// [] → Array.prototype → Object.prototype → null

// Строка (примитив оборачивается в объект):
// "str" → String.prototype → Object.prototype → null

// Функция:
// function(){} → Function.prototype → Object.prototype → null

// Число:
// 123 → Number.prototype → Object.prototype → null

// Объект без прототипа:
// Object.create(null) → null

// Проверка:
const arr = [];
console.log(arr.__proto__ === Array.prototype); // true
console.log(Array.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null
```

---
