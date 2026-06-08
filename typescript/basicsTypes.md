# Типизация переменных, функций, any, unknown, void, never

## 1. Как типизировать переменные?

### 1.1 Примитивные переменные

```typescript
// ========== Простая типизация ==========
let name: string = "Anna";
let age: number = 25;
let isAdmin: boolean = true;
let id: symbol = Symbol("id");
let bigNumber: bigint = 9007199254740991n;

// ========== Общая типизация (Union Types) ==========
// Переменная может быть одного из нескольких типов
let value: number | string = 5;
value = "asdasd"; // ✅ OK
value = true; // ❌ Ошибка! Type 'boolean' is not assignable

// ========== Узкая типизация (Literal Types) ==========
// Переменная может принимать только конкретные значения
let status: "success" | "error" | "loading" = "success";
status = "error"; // ✅ OK
status = "failed"; // ❌ Ошибка! Тип '"failed"' не назначен

let digit: 5 | 10 | "str" | true = 5;
digit = true; // ✅ OK
digit = 7; // ❌ Ошибка!
```

### 1.2 Массивы

```typescript
// ========== Синтаксис 1: тип[] ==========
let numbers: number[] = [1, 2, 3];
let names: string[] = ["Anna", "John"];
let booleans: boolean[] = [true, false];

// ========== Синтаксис 2: Array<тип> (Generic) ==========
let numbers2: Array<number> = [1, 2, 3];
let names2: Array<string> = ["Anna", "John"];

// ========== Массивы с несколькими типами (Union) ==========
let mixed: (number | string)[] = [1, 2, "Anna", "John", 3];
let mixed2: Array<number | string> = [1, "two", 3];

// ========== Пустой массив ==========
let empty: number[] = []; // ✅ OK
empty.push(5); // ✅ OK
empty.push("str"); // ❌ Ошибка!

// ========== Readonly массив (неизменяемый) ==========
let readonly: readonly number[] = [1, 2, 3];
readonly.push(4); // ❌ Ошибка! Property 'push' does not exist
```

### 1.3 Объекты

```typescript
// ========== Анонимный тип (прямое указание) ==========
let user: {
  name: string;
  age: number;
  isStudent?: boolean; // ? — опциональное поле
} = {
  name: "Anna",
  age: 25,
  // isStudent — можно не указывать
};

// ========== Через type (псевдоним типа) ==========
type User = {
  name: string;
  age: number;
  email: string;
};

let user1: User = {
  name: "Anna",
  age: 25,
  email: "anna@example.com",
};

// ========== Через interface ==========
interface IUser {
  name: string;
  age: number;
  email: string;
}

let user2: IUser = {
  name: "John",
  age: 30,
  email: "john@example.com",
};

// ========== Record<K, V> — тип для объектов с ключами типа K и значениями V ==========
const obj1: Record<string, number> = {
  age: 25,
  height: 180,
};

const obj2: Record<number, boolean> = {
  5: true,
  6: false,
};

// ========== Индексная сигнатура ==========
const obj3: { [key: string]: number } = {
  age: 25,
  height: 180,
};
```

---

## 2. Как типизировать функции?

### 2.1 Типизация параметров и возвращаемого значения

```typescript
// ========== Базовая типизация функции ==========
// Параметры: (a: number, b: number)
// Возвращаемое значение: : number
function sum(a: number, b: number): number {
  return a + b;
}

// ========== Если функция ничего не возвращает ==========
function logMessage(message: string): void {
  console.log(message);
  // return не нужен (или return;)
}

// ========== Опциональные параметры (?) ==========
function greet(name: string, age?: number): string {
  if (age) {
    return `Привет, ${name}! Тебе ${age} лет.`;
  }
  return `Привет, ${name}!`;
}

console.log(greet("Anna")); // "Привет, Anna!"
console.log(greet("Anna", 25)); // "Привет, Anna! Тебе 25 лет."

// ========== Параметры по умолчанию ==========
function multiply(a: number, b: number = 2): number {
  return a * b;
}
console.log(multiply(5)); // 10 (b = 2 по умолчанию)
console.log(multiply(5, 3)); // 15

// ========== Rest параметры ==========
function sumAll(...numbers: number[]): number {
  return numbers.reduce((acc, num) => acc + num, 0);
}
console.log(sumAll(1, 2, 3, 4)); // 10
```

### 2.2 Типизация стрелочных функций

```typescript
// ========== Полная типизация стрелочной функции ==========
const sumArrow: (a: number, b: number) => number = (a, b) => {
  return a + b;
};

// ========== Сокращённая запись (вывод типов) ==========
const sumArrow2 = (a: number, b: number): number => a + b;

// ========== Типизация колбэков ==========
function processArray(arr: number[], callback: (item: number, index: number) => void): void {
  arr.forEach((item, index) => callback(item, index));
}

processArray([1, 2, 3], (item, index) => {
  console.log(`Индекс ${index}: ${item}`);
});
```

### 2.3 Типизация функции как типа (Function Type)

```typescript
// ========== Объявление типа функции ==========
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;
const multiply2: MathOperation = (a, b) => a * b;

// ========== Тип функции с несколькими вариантами (overload) ==========
// Перегрузка функции — несколько сигнатур вызова
function process(value: string): string;
function process(value: number): number;
function process(value: string | number): string | number {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value * 2;
}

console.log(process("hello")); // "HELLO"
console.log(process(10)); // 20
```

---

## 3. Дополнительные типы TypeScript

### 3.1 any

```typescript
// any — отключает проверку типов (по возможности НЕ использовать)

let something: any = "hello";
something = 42; // ✅ OK
something = true; // ✅ OK
something = { name: "John" }; // ✅ OK
something.anything.goes(); // ✅ OK (нет проверки!)

// Когда может пригодиться any:
// 1. Миграция с JS на TS
// 2. Работа с динамическими данными из внешних библиотек
// 3. Прототипирование

// ❌ ПЛОХО: теряем все преимущества TypeScript
let data: any = fetchData(); // тип не проверяется
data.toUpperCase(); // если data — число, будет ошибка в runtime

// ✅ ЛУЧШЕ: использовать unknown или union types
```

### 3.2 unknown

```typescript
// unknown — любой тип, но БЕЗОПАСНЕЕ чем any
// Нельзя использовать unknown без проверки типа

let data: unknown = "hello";
data = 42; // ✅ OK
data = true; // ✅ OK

// ❌ Ошибка! Cannot use unknown without type check
// console.log(data.toUpperCase());

// ✅ Нужно проверить тип перед использованием
if (typeof data === "string") {
  console.log(data.toUpperCase()); // ✅ OK
}

if (typeof data === "number") {
  console.log(data * 2); // ✅ OK
}

// ========== Реальный пример: парсинг JSON ==========
function parseJSON(jsonString: string): unknown {
  return JSON.parse(jsonString);
}

const result = parseJSON('{"name": "Anna"}');

// ❌ Ошибка! unknown нельзя использовать напрямую
// console.log(result.name);

// ✅ Проверяем тип
if (typeof result === "object" && result !== null && "name" in result) {
  console.log(result.name); // "Anna"
}
```

### 3.3 void

```typescript
// void — функция ничего не возвращает (или возвращает undefined)

function log(message: string): void {
  console.log(message);
  // return не нужен
}

function doNothing(): void {
  return; // ✅ OK (возвращает undefined)
}

// ❌ Ошибка! Нельзя вернуть значение
// function wrong(): void {
//     return 42;  // Type 'number' is not assignable to type 'void'
// }

// void в колбэках (позволяет игнорировать возвращаемое значение)
const numbers = [1, 2, 3];
numbers.forEach((num): void => {
  console.log(num);
  // можно вернуть что угодно, это будет проигнорировано
});
```

### 3.4 never

```typescript
// never — функция НИКОГДА не завершается нормально
// (всегда выбрасывает ошибку или уходит в бесконечный цикл)

function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // бесконечный цикл
  }
}

// ========== never в exhaustive проверках ==========
type Shape = "circle" | "square";

function getArea(shape: Shape, size: number): number {
  switch (shape) {
    case "circle":
      return Math.PI * size * size;
    case "square":
      return size * size;
    default:
      // Если добавится новый тип "triangle", TypeScript покажет ошибку
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}

// ========== never vs void ==========
// void: функция завершается, но ничего не возвращает
// never: функция НИКОГДА не завершается (ошибка или бесконечность)
```

---

## 4. Type vs Interface

```typescript
// ========== type (псевдоним типа) ==========
type TUser = {
  name: string;
  age: number;
  email: string;
};

// Объединение (union) — только у type
type Status = "loading" | "success" | "error";
type ID = string | number;

// Кортеж (tuple) — только у type
type Point = [number, number];

// ========== interface ==========
interface IUser {
  name: string;
  age: number;
  email: string;
}

// Расширение (extends)
interface IAdmin extends IUser {
  permissions: string[];
}

// ========== Ключевые отличия ==========
// 1. interface можно расширять (declaration merging)
interface IUser {
  phone?: string; // Добавляем поле в существующий интерфейс
}

// 2. type нельзя расширить (ошибка при повторном объявлении)
// type TUser = { phone?: string }; // ❌ Ошибка! Duplicate identifier

// 3. type может использовать union, intersection, mapped types
type Nullable<T> = T | null;
type Readonly<T> = { readonly [P in keyof T]: T[P] };

// ========== Что использовать? ==========
// - Интерфейсы (interface) для объектов, классов, API
// - Типы (type) для union, кортежей, утилит
```

---

## 5. Как правильно читать ошибки TypeScript

### Ошибка 1: Отсутствует свойство

```typescript
interface IUser {
  name: string;
  age: number;
}

const user: IUser = {
  name: "Anna",
  // ❌ Property 'age' is missing in type '{ name: string; }'
};
// Решение: добавить поле age
```

### Ошибка 2: Неправильный тип значения

```typescript
interface IUser {
  name: string;
  age: number;
}

const user: IUser = {
  name: "Anna",
  age: "25",
  // ❌ Type 'string' is not assignable to type 'number'
};
// Решение: age: 25 (число)
```

### Ошибка 3: Лишнее свойство

```typescript
interface IUser {
  name: string;
  age: number;
}

const user: IUser = {
  name: "Anna",
  age: 25,
  phone: "123-456",
  // ❌ Object literal may only specify known properties
};
// Решение: удалить phone или добавить в интерфейс
```

### Ошибка 4: unknown не подходит

```typescript
let data: unknown = "hello";
console.log(data.toUpperCase());
// ❌ Object is of type 'unknown'

// Решение: проверить тип перед использованием
if (typeof data === "string") {
  console.log(data.toUpperCase()); // ✅ OK
}
```

### Ошибка 5: never в switch (exhaustive check)

```typescript
type Shape = "circle" | "square" | "triangle";

function getArea(shape: Shape, size: number): number {
  switch (shape) {
    case "circle":
      return Math.PI * size * size;
    case "square":
      return size * size;
    default:
      // ❌ Type 'string' is not assignable to type 'never'
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
// Решение: добавить case для "triangle"
```

---

## 6. Шпаргалка по типам TypeScript

| Тип            | Описание                          | Пример                                  |
| -------------- | --------------------------------- | --------------------------------------- | --------------- | ----------- |
| `string`       | Строки                            | `let name: string = "Anna"`             |
| `number`       | Числа                             | `let age: number = 25`                  |
| `boolean`      | true/false                        | `let isAdmin: boolean = true`           |
| `null`         | null                              | `let empty: null = null`                |
| `undefined`    | undefined                         | `let notSet: undefined = undefined`     |
| `any`          | ❌ Любой тип (отключает проверку) | `let data: any = 5`                     |
| `unknown`      | ✅ Любой тип (нужна проверка)     | `let data: unknown = 5`                 |
| `void`         | Ничего не возвращает              | `function log(): void {}`               |
| `never`        | Никогда не завершается            | `function error(): never { throw ... }` |
| `type[]`       | Массив                            | `let nums: number[] = [1, 2]`           |
| `[type, type]` | Кортеж                            | `let pair: [string, number] = ["x", 5]` |
| `type          | type`                             | Объединение (union)                     | `let id: string | number = 5` |
| `type & type`  | Пересечение (intersection)        | `type Admin = User & { role: string }`  |

---

## 7. Шпаргалка по ошибкам TypeScript

| Ошибка                                            | Смысл                     | Решение                                  |
| ------------------------------------------------- | ------------------------- | ---------------------------------------- |
| `is missing in type`                              | Нет обязательного поля    | Добавить поле                            |
| `is not assignable`                               | Неправильный тип значения | Исправить тип                            |
| `only specifies known properties`                 | Лишнее поле               | Удалить или добавить в интерфейс         |
| `Object is of type 'unknown'`                     | unknown без проверки      | Проверить тип перед использованием       |
| `Type 'string' is not assignable to type 'never'` | Неполный switch           | Добавить недостающий case                |
| `Property does not exist on type`                 | Нет свойства в типе       | Проверить имя свойства или расширить тип |

---
