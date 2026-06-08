# Data Types, Type Conversion, References

## Теория (вопросы и ответы)

### Какие типы данных существуют в JavaScript?

В JavaScript существует 8 типов данных.

### Примитивные типы

```js
string;
number;
boolean;
undefined;
null;
symbol;
bigint;
```

### Ссылочный тип

```js
object;
```

К объектам относятся:

- Object
- Array
- Function
- Date
- Map
- Set
- RegExp

Примеры:

```js
typeof 42; // "number"
typeof "Hello"; // "string"
typeof true; // "boolean"
typeof undefined; // "undefined"
typeof Symbol(); // "symbol"
typeof 10n; // "bigint"
typeof {}; // "object"
typeof []; // "object"
typeof function () {}; // "function"
```

---

### Почему typeof null возвращает "object"?

```js
typeof null; // "object"
```

Это историческая ошибка JavaScript.

Она появилась в первой реализации языка и была сохранена для обратной совместимости.

---

### Что такое явное преобразование типов?

Когда разработчик самостоятельно приводит значение к нужному типу.

Примеры:

```js
Number("123");
String(123);
Boolean(1);
```

---

### Что такое неявное преобразование типов?

Когда JavaScript автоматически приводит значения к нужному типу.

Примеры:

```js
"5" * 2; // 10
5 + "5"; // "55"
```

---

### Как преобразовать значение в число?

Способы:

```js
Number("123");
+"123";
parseInt("123");
parseFloat("12.5");
```

---

### Как преобразовать значение в строку?

Способы:

```js
String(123);
(123).toString();
```

---

### Как преобразовать значение в boolean?

Способы:

```js
Boolean(value);
!!value;
```

---

### Какие значения являются falsy?

Falsy значения всегда преобразуются в false.

Полный список:

```js
false;
0;
-0;
0n;
("");
null;
undefined;
NaN;
```

---

### Какие значения являются truthy?

Все остальные значения.

Например:

```js
[];
{
}
("0");
("false");
Infinity;
```

будут преобразованы в true.

---

### Что такое NaN?

NaN означает:

```text
Not a Number
```

Возникает при невозможности выполнить математическую операцию.

Примеры:

```js
5 * undefined;
+"abc";
Math.sqrt(-1);
```

---

### Почему NaN === NaN возвращает false?

```js
NaN === NaN; // false
```

По спецификации NaN не равен ничему, включая самого себя.

---

### Как правильно проверить NaN?

Лучший способ:

```js
Number.isNaN(value);
```

Пример:

```js
Number.isNaN(NaN); // true
```

---

### Чем отличается isNaN от Number.isNaN?

Плохо:

```js
isNaN("abc");
```

Результат:

```js
true;
```

Потому что сначала происходит преобразование в число.

Лучше:

```js
Number.isNaN("abc");
```

Результат:

```js
false;
```

---

### Чем отличается == от ===?

```js
==
```

Нестрогое сравнение.

Происходит приведение типов.

```js
===
```

Строгое сравнение.

Приведение типов не выполняется.

Примеры:

```js
5 == "5"; // true
5 === "5"; // false
```

---

### Почему null == undefined возвращает true?

Это специальное правило JavaScript.

```js
null == undefined; // true
```

Но:

```js
null === undefined; // false
```

---

### Что такое ссылка в JavaScript?

Примитивы копируются по значению.

Объекты копируются по ссылке.

Пример:

```js
const user = { name: "Maria" };

const copy = user;

copy.name = "Denis";

console.log(user.name);
```

Результат:

```js
"Denis";
```

Потому что обе переменные указывают на один объект.

---

### Что такое поверхностное копирование (Shallow Copy)?

Создается новый объект только верхнего уровня.

Вложенные объекты остаются общими.

Пример:

```js
const clone = { ...obj };
```

или

```js
Object.assign({}, obj);
```

---

### Что такое глубокое копирование (Deep Copy)?

Создаются новые копии всех вложенных объектов.

Современный способ:

```js
structuredClone(obj);
```

---

### Почему JSON.parse(JSON.stringify()) считается плохим способом глубокого копирования?

Потому что теряются:

```js
function
undefined
symbol
Date
Map
Set
```

Также не поддерживаются циклические ссылки.

---

## Практические вопросы собеседований

### Что выведет код?

```js
console.log(typeof null);
```

Ответ:

```js
"object";
```

Историческая ошибка JavaScript.

---

### Что выведет код?

```js
console.log(typeof []);
```

Ответ:

```js
"object";
```

Массивы являются объектами.

---

### Что выведет код?

```js
console.log(typeof function () {});
```

Ответ:

```js
"function";
```

---

### Что выведет код?

```js
console.log(5 + "5");
```

Ответ:

```js
"55";
```

Оператор + выполняет конкатенацию строк.

---

### Что выведет код?

```js
console.log("10" - "5");
```

Ответ:

```js
5;
```

Оба значения будут преобразованы в числа.

---

### Что выведет код?

```js
console.log(+"123");
```

Ответ:

```js
123;
```

Унарный плюс приводит строку к числу.

---

### Что выведет код?

```js
console.log(+"123abc");
```

Ответ:

```js
NaN;
```

---

### Что выведет код?

```js
console.log(5 == "5");
console.log(5 === "5");
```

Ответ:

```js
true;
false;
```

---

### Что выведет код?

```js
console.log(null == undefined);
console.log(null === undefined);
```

Ответ:

```js
true;
false;
```

---

### Что выведет код?

```js
console.log(0 == false);
console.log(0 === false);
```

Ответ:

```js
true;
false;
```

---

### Что выведет код?

```js
const user = { name: "Maria" };

const copy = user;

copy.name = "Denis";

console.log(user.name);
```

Ответ:

```js
"Denis";
```

Обе переменные содержат ссылку на один объект.

---

### Что выведет код?

```js
const obj1 = { a: { b: 1 } };

const obj2 = { ...obj1 };

obj2.a.b = 999;

console.log(obj1.a.b);
```

Ответ:

```js
999;
```

Spread выполняет только поверхностное копирование.

---

### Что выведет код?

```js
console.log(NaN === NaN);
```

Ответ:

```js
false;
```

---

### Что выведет код?

```js
console.log({} === {});
console.log([] === []);
```

Ответ:

```js
false;
false;
```

Это разные объекты в памяти.

---

### Что выведет код?

```js
let obj = { x: 10 };

let ref = obj;

ref.x = 20;

obj = { x: 30 };

console.log(ref.x);
```

Ответ:

```js
20;
```

Переменная ref продолжает ссылаться на старый объект.
