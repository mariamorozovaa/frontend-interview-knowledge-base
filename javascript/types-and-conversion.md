````markdown
# База знаний JavaScript: вопросы, ответы и практика

## 1. Какие типы данных существуют в JavaScript?

```js
// Примитивные (7):
// number, string, boolean, undefined, null, symbol, bigint
// Ссылочные:
// object (включая массивы, функции, даты, регулярные выражения)

// Пример:
console.log(typeof 42); // "number"
console.log(typeof "Hello"); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object" (историческая ошибка JS)
console.log(typeof {}); // "object"
console.log(typeof []); // "object"
console.log(typeof function () {}); // "function"
console.log(typeof Symbol("id")); // "symbol"
console.log(typeof 9007199254740991n); // "bigint"
```
````

## 2. Явное и неявное преобразование в число, строку и булеан

```js
// В число
// Явное:
Number("123");    // 123
+"123";           // 123
parseInt("123");  // 123
// Неявное:
"5" * 2;          // 10
"10" - "5";       // 5

// В строку
// Явное:
String(123);      // "123"
(123).toString(); // "123"
// Неявное:
5 + "5";          // "55"

// В булеан
// Явное:
Boolean(1);       // true
!!1;              // true
// Неявное:
if (1) {}         // true
value || default
```

## 2.5. Какие значения всегда преобразуются в false (falsy)?

```js
// Все falsy значения в JS:
false;
0 - 0;
0n; // BigInt ноль
(""); // пустая строка
null;
undefined;
NaN;

// Всё остальное — true (включая [], {}, "0", "false")
```

## 3. Что такое NaN?

```js
// NaN = Not a Number
// Возникает при некорректной математической операции

console.log(typeof NaN); // "number"
console.log(NaN === NaN); // false (NaN не равен сам себе)
console.log(isNaN("abc")); // true
console.log(Number.isNaN(NaN)); // true (строгая проверка)
console.log(5 * undefined); // NaN
console.log(+"123abc"); // NaN
```

## 4. Как работают операторы == и ===?

```js
// ==  — нестрогое равенство (с приведением типов)
// === — строгое равенство (без приведения)

console.log(5 == "5"); // true
console.log(5 === "5"); // false
console.log(null == undefined); // true
console.log(null === undefined); // false
console.log(0 == false); // true
console.log(0 === false); // false
```

## 5. Что такое ссылки в JavaScript?

```js
// Примитивы копируются по значению
// Объекты — по ссылке

let a = { x: 1 };
let b = a; // b ссылается на тот же объект
b.x = 2;
console.log(a.x); // 2

// Пример с перезаписью
let obj = { x: 10 };
let ref = obj;
ref.x = 20;
obj = { x: 30 };
console.log(ref.x); // 20 (ref всё ещё указывает на старый объект)
```

## 6. Что такое поверхностное и глубокое копирование?

```js
// Поверхностное копирование (shallow clone)
const original = { a: 1, b: { c: 2 } };
const shallow = { ...original }; // spread
const shallow2 = Object.assign({}, original);

// Глубокое копирование (deep clone)
const deep = structuredClone(original); // современный способ
// или
const deepJSON = JSON.parse(JSON.stringify(original));

// Проверка
original.b.c = 99;
console.log(shallow.b.c); // 99 (общая ссылка)
console.log(deep.b.c); // 2 (независим)

// Минусы JSON метода:
// - не копирует функции, undefined, Symbol
// - не работает с датами, Map, Set, циклическими ссылками
```

---

# Практические задачи с решениями

## Задача 1. Определение типа данных (typeof)

```js
console.log(typeof 42); // "number"
console.log(typeof "Hello"); // "string"
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object"
console.log(typeof {}); // "object"
console.log(typeof []); // "object"
console.log(typeof function () {}); // "function"
console.log(typeof Symbol("id")); // "symbol"
console.log(typeof 9007199254740991n); // "bigint"
```

## Задача 2. Ссылки в объектах (easy)

```js
const user = { name: "Some", age: 22, pet: { nickname: "Gosha", age: 16 } };
const userTwo = user;
userTwo.name = "Denis";

console.log(user.name); // "Denis"
console.log(user === userTwo); // true (один и тот же объект)
```

## Задача 3. Функция изменяет объект

```js
function getName(person) {
  person.name = `My name is ${person.name}`;
  return person.name;
}

console.log(getName(user)); // "My name is Denis"
console.log(user); // { name: "My name is Denis", age: 22, pet: {...} }
```

## Задача 4. Поверхностное копирование (spread)

```js
const user = { name: "Some", age: 22, pet: { nickname: "Gosha", age: 16 } };
const userThree = { ...user };
user.pet.typeAnimal = "cat";

console.log(user === userThree); // false (разные объекты верхнего уровня)
console.log(user.pet === userThree.pet); // true (pet — общая ссылка)
console.log(user.name === userThree.name); // true (строки по значению)
```

## Задача 5. Глубокое копирование (structuredClone)

```js
const car = { mark: "Denis", details: { model: "BMW", year: 2020 } };
const carTwo = structuredClone(car);
carTwo.details.year = 2024;

console.log(car.details.year === carTwo.details.year); // false
console.log(car === carTwo); // false
console.log(car.details === carTwo.details); // false (полностью независимые)
```

## Задача 6. Неявное преобразование в строку

```js
console.log(5 + "5"); // "55"   (число + строка = строка)
console.log("10" + 2 * "5"); // "1010" (2*5=10, "10"+10 = "1010")
console.log(10 + 2 + "5"); // "125"  (10+2=12, 12+"5"="125")
console.log(10 + "2" + 5); // "1025" (10+"2"="102", "102"+5="1025")
console.log("10" - "2"); // 8      (строка - строка → число)
console.log("10" * "2"); // 20
console.log("10" / "2"); // 5
```

## Задача 7. Неявное преобразование в число

```js
console.log(+"123"); // 123
console.log(+" 123 "); // 123 (пробелы игнорируются)
console.log(+"123abc"); // NaN
console.log(5 * "5"); // 25
console.log(5 - true); // 4   (true → 1)
console.log(5 + false); // 5   (false → 0)
console.log(5 / null); // Infinity (null → 0)
console.log(5 * undefined); // NaN
```

## Задача 8. Сравнение == и ===

```js
console.log(5 == "5"); // true
console.log(5 === "5"); // false
console.log(null == undefined); // true
console.log(null === undefined); // false
console.log(0 == false); // true
console.log(0 === false); // false
```

## Задача 9. Перезапись объекта и ссылки

```js
let obj = { x: 10 };
let ref = obj;
ref.x = 20;
obj = { x: 30 };

console.log(ref.x); // 20 (ref не изменился, он ссылается на старый объект)
```

## Задача 10. Перезапись объекта в функции

```js
function updateProfile(user) {
  user = { name: "Mike" }; // локальная перезапись, не влияет на оригинал
}
function process() {
  let person = { name: "Peter" };
  updateProfile(person);
  console.log(person); // { name: "Peter" } (не изменился)
}
process();
```

## Задача 11. Замыкание и ссылки

```js
function createCounter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
}
const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
// Замыкание сохраняет ссылку на переменную count
```

## Задача 12. Неочевидное сравнение с NaN

```js
console.log(NaN == NaN); // false
console.log(NaN === NaN); // false
console.log(isNaN(NaN)); // true
console.log(Number.isNaN(NaN)); // true
console.log(isNaN("abc")); // true (сначала приводит к числу)
console.log(Number.isNaN("abc")); // false (строгая проверка)
```

## Задача 13. Сравнение объектов и массивов

```js
console.log({} == {}); // false (разные ссылки)
console.log({} === {}); // false
console.log([] == []); // false
console.log([] === []); // false
console.log([] == ![]); // true  ("" == false → 0 == 0)
```

## Задача 14. Опасности поверхностного копирования

```js
const obj1 = { a: { b: 1 } };
const obj2 = { ...obj1 };
obj2.a.b = 999;
console.log(obj1.a.b); // 999 (так как a — общая ссылка)

// Правильное глубокое копирование:
const obj3 = structuredClone(obj1);
obj3.a.b = 777;
console.log(obj1.a.b); // 999 (не изменилось)
```

## Задача 15. Преобразование через плюс и минус

```js
console.log(+"5" + +"3"); // 8 (оба стали числами)
console.log("5" - "3"); // 2
console.log("5" + "3"); // "53"
console.log("5" - 3); // 2
console.log(5 + "3"); // "53"
console.log("5" * "3"); // 15
console.log("10" / "2"); // 5
```
