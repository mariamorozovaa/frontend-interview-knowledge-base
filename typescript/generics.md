# Дженерики (Generics)

---

## 1. Что такое дженерики (Generics) в TS?

Дженерики — это способ создавать **обобщённые типы и функции**, которые могут работать с разными типами данных, но при этом **сохраняют строгую типизацию**.

Проще:

> дженерик = “тип как параметр”

### Проблема без generics:

```typescript
function identity(arg: any): any {
  return arg;
}
```

Минус: теряется тип.

### Решение с generics:

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

### Что происходит:

- `T` — это переменная типа
- TypeScript подставляет реальный тип при вызове
- сохраняется точная типизация

---

## 2. Как указать значение generics по умолчанию?

Значение по умолчанию задаётся через `=`:

```typescript
function wrap<T = string>(value: T): T[] {
  return [value];
}
```

### Примеры:

```typescript
wrap("hello"); // T = string (по умолчанию)
wrap<number>(123); // T = number (явно)
```

### Где ещё используется:

```typescript
type ApiResponse<T = unknown> = {
  data: T;
};
```

---

## 3. Можно ли один дженерик описать сразу несколько типов?

Да — через **tuple / union / multiple generic parameters**.

### 1. Несколько дженериков:

```typescript
function pair<T, U>(a: T, b: U) {
  return [a, b];
}
```

---

### 2. Один дженерик как “набор типов” (tuple):

```typescript
type Pair<T extends [any, any]> = {
  first: T[0];
  second: T[1];
};
```

---

### 3. Один дженерик как union:

```typescript
type Value<T> = T extends string | number ? T : never;
```

---

### Итог:

- ❗ Один generic = один параметр типа
- ✔ Но он может быть:
  - union (`string | number`)
  - tuple (`[T, U]`)
  - объектом (`{a: T, b: U}`)

---

## 4. Почему нужно `extends keyof T`, а не просто `keyof T`?

### ❌ Неправильно:

```typescript
function get<T, K: keyof T>(obj: T, key: K) {}
```

### ❗ Почему нельзя просто `keyof T`?

`keyof T` — это **готовый union всех ключей**, а не ограничение.

---

### ✔ Правильный вариант:

```typescript
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

---

### Почему `extends` нужен:

- `K` — это **переменная типа**
- `keyof T` — это **набор допустимых значений**
- `extends` = “ограничить K этим набором”

---

### Пример:

```typescript
type User = { name: string; age: number };

type K = keyof User;
// "name" | "age"
```

Если не использовать `extends`, TS не понимает:

- что K должен быть частью User
- и теряется связь между key → value

---

### Простая формула:

> `K extends keyof T` = “K должен быть одним из ключей T”

---

## 5. `in` в дженериках `<T, K in keyof T>`

Это используется в **mapped types**.

---

### Что делает `in`?

`in` — это перебор ключей объекта.

---

### Пример:

```typescript
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

---

### Разбор:

```typescript
K in keyof T
```

означает:

> для каждого ключа K из T создать новое поле

---

### Пример работы:

```typescript
type User = {
  name: string;
  age: number;
};

type ReadonlyUser = Readonly<User>;
```

Результат:

```typescript
{
  readonly name: string;
  readonly age: number;
}
```

---

## 🔥 Ещё пример (очень важно)

### Сделать все поля optional:

```typescript
type Partial<T> = {
  [K in keyof T]?: T[K];
};
```

---

### Что происходит:

| Часть     | Значение       |
| --------- | -------------- |
| `keyof T` | список ключей  |
| `K in`    | перебор ключей |
| `T[K]`    | тип значения   |

---

## 💡 Итог по `in`:

> `in` = "пройтись по ключам объекта и создать новый тип"

---

## 🔥 Мини-шпаргалка

| Конструкция         | Значение                     |
| ------------------- | ---------------------------- |
| `<T>`               | один универсальный тип       |
| `<T = X>`           | значение по умолчанию        |
| `<T, U>`            | несколько типов              |
| `T extends U`       | ограничение типа             |
| `K extends keyof T` | ключ объекта                 |
| `K in keyof T`      | перебор ключей (mapped type) |

---

## 1. Как создать дженерик?

### 1.1 Создание типа с дженериком через interface

```typescript
// Дженерик в интерфейсе — обозначается через <T>

interface Box<T> {
  value: T;
  getValue(): T;
}

// Использование с разными типами:
const stringBox: Box<string> = {
  value: "Hello",
  getValue() {
    return this.value;
  },
};

const numberBox: Box<number> = {
  value: 42,
  getValue() {
    return this.value;
  },
};

// Несколько дженериков:
interface Pair<L, R> {
  left: L;
  right: R;
}

const pair: Pair<number, string> = {
  left: 1,
  right: "один",
};
```

### 1.2 Создание типа с дженериком через type

```typescript
// Дженерик в type-псевдониме

type Box<T> = {
  value: T;
  getValue: () => T;
};

type Result<T> = T | null;
type AsyncResult<T> = Promise<Result<T>>;

// Несколько дженериков:
type Dictionary<K, V> = {
  [key in K]: V;
};

type UserMap = Dictionary<string, { name: string; age: number }>;
```

### 1.3 Использование дженерика в функциях

```typescript
// Базовая функция с дженериком
function identity<T>(arg: T): T {
  return arg;
}

// Явное указание типа:
const num = identity<number>(10); // T = number
const str = identity<string>("Hello"); // T = string

// Вывод типа (type inference) — TypeScript сам определяет тип:
const num2 = identity(10); // T = number
const str2 = identity("Hello"); // T = string

// Функция с несколькими дженериками:
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge({ name: "Anna" }, { age: 25 });
// merged: { name: string } & { age: number }
console.log(merged.name, merged.age); // "Anna" 25

// Дженерик в стрелочной функции:
const wrapInArray = <T>(value: T): T[] => [value];

const numbers = wrapInArray(42); // number[]
const strings = wrapInArray("hello"); // string[]
```

---

## 2. Дефолтное значение для дженерика

```typescript
// Значение по умолчанию указывается через = после имени дженерика

function makeArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

// T не указан явно → используется тип по умолчанию (string)
const arr1 = makeArray(3, "hi"); // string[]

// T указан явно → используется указанный тип
const arr2 = makeArray<number>(2, 10); // number[]
const arr3 = makeArray<boolean>(3, true); // boolean[]

// С несколькими дженериками:
type ApiResponse<T = any, E = Error> = {
  data?: T;
  error?: E;
};

const response1: ApiResponse = { data: "OK" }; // T = any, E = Error
const response2: ApiResponse<string> = { data: "OK" }; // T = string
const response3: ApiResponse<number, string> = { data: 42 };
```

---

## 3. Ограничение дженерика (extends)

```typescript
// extends — ограничивает, какие типы можно передавать в дженерик

// Пример 1: объект должен иметь поле name
function logName<T extends { name: string }>(obj: T): void {
  console.log(obj.name);
}

logName({ name: "Anna", age: 25 }); // ✅ OK
logName({ age: 25 }); // ❌ Ошибка: нет поля name

// Пример 2: только числа
function sum<T extends number>(a: T, b: T): T {
  return (a + b) as T;
}

console.log(sum(5, 3)); // ✅ OK
// console.log(sum("5", 3)); // ❌ Ошибка

// Пример 3: ограничение через интерфейс
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): T {
  console.log(item.length);
  return item;
}

logLength("hello"); // ✅ string имеет length
logLength([1, 2, 3]); // ✅ array имеет length
logLength(123); // ❌ number не имеет length

// Пример 4: ограничение через union
type Primitive = string | number | boolean;

function formatValue<T extends Primitive>(value: T): string {
  return String(value);
}

console.log(formatValue(42)); // "42"
console.log(formatValue(true)); // "true"
console.log(formatValue("hello")); // "hello"
// formatValue({}); // ❌ Ошибка
```

---

## 4. Для чего придумали дженерики?

```typescript
// Дженерики нужны для:
// 1. Переиспользуемости кода
// 2. Сохранения типизации при работе с разными типами

// ========== БЕЗ дженериков (используем any) ==========
function identityAny(arg: any): any {
  return arg;
}
const resultAny = identityAny(10);
// ❌ resultAny имеет тип any — информация о типе потеряна
resultAny.toUpperCase(); // Нет ошибки TS, но в runtime ошибка!

// ========== С дженериками ==========
function identityGeneric<T>(arg: T): T {
  return arg;
}
const resultGeneric = identityGeneric(10);
// ✅ resultGeneric имеет тип number
resultGeneric.toFixed(2); // ✅ OK
// resultGeneric.toUpperCase(); // ❌ Ошибка компиляции

// ========== Реальные примеры ==========
// Массивы:
const numbers: Array<number> = [1, 2, 3]; // Array<T>
const promises: Promise<string> = Promise.resolve("hello");

// Встроенные дженерики:
// - Array<T>
// - Promise<T>
// - Map<K, V>
// - Set<T>
// - Record<K, V>
// - Partial<T>
// - Readonly<T>
```

---

## 5. Оператор typeof в TypeScript

```typescript
// typeof в TS для типов — получает тип переменной

const user = {
  name: "Anna",
  age: 25,
  address: {
    city: "Moscow",
    street: "Tverskaya",
  },
};

// Получаем тип объекта
type UserType = typeof user;
// {
//     name: string;
//     age: number;
//     address: { city: string; street: string; }
// }

// Использование с функциями:
function getUser() {
  return { id: 1, name: "Anna" };
}
type GetUserResult = ReturnType<typeof getUser>;
// { id: number; name: string }

// typeof с классами:
class Person {
  constructor(
    public name: string,
    public age: number,
  ) {}
}
type PersonType = typeof Person; // Конструктор типа new (name: string, age: number) => Person

// typeof с ключами объекта:
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

type ConfigKey = keyof typeof config; // "apiUrl" | "timeout" | "retries"
```

---

## 6. Оператор keyof в дженериках

```typescript
// keyof — берёт ключи объекта и создаёт из них объединение строк

type User = {
  name: string;
  age: number;
  email: string;
};

type UserKeys = keyof User; // "name" | "age" | "email"

// ========== keyof с дженериками ==========
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Anna", age: 25, email: "anna@example.com" };

const name = getProperty(user, "name"); // ✅ string
const age = getProperty(user, "age"); // ✅ number
// const invalid = getProperty(user, "invalid"); // ❌ Ошибка

// ========== Пример: функция pluck ==========
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}

const users = [
  { name: "Anna", age: 25 },
  { name: "John", age: 30 },
];

const names = pluck(users, "name"); // string[]
const ages = pluck(users, "age"); // number[]

// ========== keyof с Record ==========
function getValue<K extends string, V>(obj: Record<K, V>, key: K): V {
  return obj[key];
}

const dict = { a: 1, b: 2, c: 3 };
console.log(getValue(dict, "a")); // 1
```

---

## 7. Оператор as (Type Assertion)

```typescript
// as — ручное указание типа (type assertion)
// Используется, когда TypeScript не может вывести тип, но разработчик уверен

// ========== Когда использовать as ==========

// 1. Работа с DOM элементами:
const input = document.querySelector("input") as HTMLInputElement;
// Без as TypeScript не знает, что это именно HTMLInputElement
console.log(input.value); // ✅ OK

// 2. Работа с unknown:
const data: unknown = "Hello World";
const length = (data as string).length; // ✅ OK

// 3. Объекты с динамическими ключами:
const response = { data: { user: { name: "Anna" } } };
const userName = (response as any).data.user.name;

// 4. Двойное приведение (when you know better):
const value = "123";
const num = value as unknown as number; // осторожно!

// ========== Альтернатива as: <> ==========
const input2 = <HTMLInputElement>document.querySelector("input");

// =========! Важные правила ==========
// ❌ НЕЛЬЗЯ приводить несвязанные типы:
// const str = "hello" as number;  // ❌ Ошибка!

// ✅ Можно приводить через unknown:
const str2 = "hello" as unknown as number; // ✅ Но опасно!

// =========! Лучшие практики ==========
// 1. Используйте type guards вместо as:
function isString(value: unknown): value is string {
  return typeof value === "string";
}

if (isString(data)) {
  console.log(data.length); // type guard безопаснее
}

// 2. Используйте satisfies (TS 4.9+):
const config = {
  theme: "dark",
  fontSize: 16,
} satisfies Record<string, string | number>;
```

---

## Практические задачи с решениями

### Задание 1. Entity<T> с optional полем

```typescript
// Создай универсальный тип Entity<T>, где:
// id: number
// data: T
// createdAt?: Date

type Entity<T> = {
  id: number;
  data: T;
  createdAt?: Date;
};

// Проверка:
const user: Entity<{ name: string }> = {
  id: 1,
  data: { name: "Anna" },
};

const product: Entity<{ title: string; price: number }> = {
  id: 2,
  data: { title: "Laptop", price: 1000 },
  createdAt: new Date(),
};
```

### Задание 2. Generic функция lastItem

```typescript
// Напиши функцию lastItem, которая:
// принимает массив
// возвращает последний элемент

function lastItem<T>(array: T[]): T {
  if (array.length === 0) {
    throw new Error("Array is empty");
  }
  return array[array.length - 1];
}

// Проверка:
console.log(lastItem([1, 2, 3])); // 3
console.log(lastItem(["a", "b"])); // "b"
console.log(lastItem([true, false])); // false
```

### Задание 3. extends + несколько условий

```typescript
// Создай функцию getId, которая:
// принимает объект
// объект обязательно должен иметь id: number | string

function getId<T extends { id: number | string }>(obj: T): T["id"] {
  return obj.id;
}

// Проверка:
console.log(getId({ id: 1 })); // 1
console.log(getId({ id: "abc" })); // "abc"
console.log(getId({ id: 1, name: "Anna" })); // 1
// console.log(getId({ name: "Anna" }));  // ❌ Ошибка: нет id
```

### Задание 4. keyof + generic (pluck)

```typescript
// Напиши функцию pluck:
// принимает массив объектов
// принимает ключ
// возвращает массив значений этого ключа

function pluck<T, K extends keyof T>(arr: T[], key: K): T[K][] {
  return arr.map((item) => item[key]);
}

// Проверка:
const users = [
  { name: "Anna", age: 25 },
  { name: "John", age: 30 },
  { name: "Alice", age: 28 },
];

console.log(pluck(users, "name")); // ["Anna", "John", "Alice"]
console.log(pluck(users, "age")); // [25, 30, 28]
```

### Задание 5. Generic + union (Response)

```typescript
// Создай тип Response<T>:
// { status: "success"; data: T }
// | { status: "error"; error: string }

type Response<T> = { status: "success"; data: T } | { status: "error"; error: string };

// Проверка:
const successResponse: Response<number> = {
  status: "success",
  data: 42,
};

const errorResponse: Response<string> = {
  status: "error",
  error: "Something went wrong",
};

// Функция, использующая тип:
function handleResponse<T>(response: Response<T>): T | null {
  if (response.status === "success") {
    return response.data;
  }
  console.error(response.error);
  return null;
}
```

### Задание 6. Partial + generic (update)

```typescript
// Напиши функцию update<T>:
// принимает объект
// принимает обновления (частичные)
// возвращает новый объект

function update<T extends object>(obj: T, updates: Partial<T>): T {
  return { ...obj, ...updates };
}

// Проверка:
const userObj = { name: "Anna", age: 20, city: "Moscow" };
const updated = update(userObj, { age: 21, city: "SPb" });
console.log(updated); // { name: "Anna", age: 21, city: "SPb" }
```

### Задание 7. Readonly + generic (DeepReadonly)

```typescript
// Создай тип DeepReadonly<T>:
// делает ВСЕ поля readonly (включая вложенные)

type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// Проверка:
type A = {
  user: {
    name: string;
    address: {
      city: string;
      street: string;
    };
  };
  version: number;
};

type B = DeepReadonly<A>;

const data: B = {
  user: {
    name: "Anna",
    address: {
      city: "Moscow",
      street: "Tverskaya",
    },
  },
  version: 1,
};

// ❌ Все поля стали readonly:
// data.user.name = "John";        // Ошибка
// data.user.address.city = "SPb"; // Ошибка
// data.version = 2;               // Ошибка
```

### Задание 8. Record + keyof (mapValues)

```typescript
// Создай функцию mapValues:
// принимает объект
// принимает функцию
// возвращает новый объект с теми же ключами

function mapValues<T extends object, R>(obj: T, fn: (value: T[keyof T]) => R): Record<keyof T, R> {
  const result = {} as Record<keyof T, R>;
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      result[key] = fn(obj[key]);
    }
  }
  return result;
}

// Проверка:
const numbers = { a: 1, b: 2, c: 3 };
const doubled = mapValues(numbers, (x) => x * 2);
console.log(doubled); // { a: 2, b: 4, c: 6 }

const strings = { a: "hello", b: "world" };
const upper = mapValues(strings, (s) => s.toUpperCase());
console.log(upper); // { a: "HELLO", b: "WORLD" }
```

---

## Шпаргалка по дженерикам

| Концепция             | Синтаксис                       | Пример                     |
| --------------------- | ------------------------------- | -------------------------- |
| Функция               | `function fn<T>(arg: T): T`     | `identity<number>(10)`     |
| Интерфейс             | `interface Box<T> { value: T }` | `Box<string>`              |
| Тип (type)            | `type Box<T> = { value: T }`    | `Box<number>`              |
| Класс                 | `class Stack<T> { items: T[] }` | `Stack<string>`            |
| Ограничение           | `T extends SomeType`            | `T extends { id: number }` |
| Значение по умолчанию | `T = DefaultType`               | `T = string`               |
| keyof                 | `K extends keyof T`             | `getProperty(obj, key)`    |
| typeof                | `typeof variable`               | `type User = typeof user`  |

---

# Conditional Types и infer в TypeScript (кратко)

## Conditional Types (Условные типы)

```typescript
// Синтаксис: T extends U ? X : Y

type IsString<T> = T extends string ? true : false;

type R1 = IsString<"hello">; // true
type R2 = IsString<42>; // false

// С несколькими условиями
type TypeName<T> = T extends string ? "string" : T extends number ? "number" : T extends boolean ? "boolean" : "other";

// Распределение по union (дистрибутивность)
type ToArray<T> = T extends any ? T[] : never;
type R3 = ToArray<string | number>; // string[] | number[]

// Отключение распределения (через кортеж)
type ToArrayNonDistributive<T> = [T] extends [any] ? T[] : never;
type R4 = ToArrayNonDistributive<string | number>; // (string | number)[]
```

### Встроенные условные типы

```typescript
type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // "c"
type T2 = Extract<"a" | "b" | "c", "a" | "b">; // "a" | "b"
type T3 = NonNullable<string | null | undefined>; // string
type T4 = ReturnType<() => string>; // string
type T5 = Parameters<(a: number, b: string) => void>; // [number, string]
```

---

## infer

```typescript
// infer — достаёт тип изнутри другого типа (работает только в conditional type)

// Достать тип элемента массива
type GetArrayItem<T> = T extends Array<infer U> ? U : never;

type R1 = GetArrayItem<number[]>; // number
type R2 = GetArrayItem<string[]>; // string

// Достать тип из Promise
type UnwrapPromise<T> = T extends Promise<infer U> ? U : never;

type R3 = UnwrapPromise<Promise<string>>; // string

// Рекурсивное разворачивание
type DeepUnwrapPromise<T> = T extends Promise<infer U> ? DeepUnwrapPromise<U> : T;

type R4 = DeepUnwrapPromise<Promise<Promise<number>>>; // number

// Достать возвращаемый тип функции
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type R5 = GetReturnType<() => string>; // string

// Достать параметры функции
type GetParams<T> = T extends (...args: infer P) => any ? P : never;

type R6 = GetParams<(a: number, b: string) => void>; // [number, string]

// Достать тип свойства объекта
type GetPropertyType<T, K> = T extends { [key in K]: infer U } ? U : never;

type User = { id: number; name: string };
type R7 = GetPropertyType<User, "name">; // string
```

---

## Комбинация conditional + infer

```typescript
// MyReturnType — своя реализация
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// MyParameters — своя реализация
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

// Flatten массива
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;

type R1 = Flatten<number[][][]>; // number

// IsUnion — проверка, является ли тип объединением
type IsUnion<T, U = T> = T extends any ? (U extends T ? false : true) : never;

type R2 = IsUnion<string>; // false
type R3 = IsUnion<string | number>; // true

// Union в Intersection
type UnionToIntersection<U> = (U extends any ? (k: U) => void : never) extends (k: infer I) => void ? I : never;

type R4 = UnionToIntersection<{ a: number } | { b: string }>; // { a: number } & { b: string }
```

---

## Шпаргалка

| Задача               | Решение                                             |
| -------------------- | --------------------------------------------------- |
| Тип элемента массива | `T extends Array<infer U> ? U : never`              |
| Тип из Promise       | `T extends Promise<infer U> ? U : never`            |
| ReturnType функции   | `T extends (...args: any[]) => infer R ? R : never` |
| Параметры функции    | `T extends (...args: infer P) => any ? P : never`   |
| Исключить тип        | `T extends U ? never : T` (как Exclude)             |
| Выбрать тип          | `T extends U ? T : never` (как Extract)             |

```typescript
// Правило запоминания:
// 1. Conditional type = T extends U ? X : Y
// 2. infer = "достань тип оттуда"
// 3. infer работает ТОЛЬКО внутри conditional type
```
