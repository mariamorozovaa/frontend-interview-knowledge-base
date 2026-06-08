# Interface, Type, Union, Intersection, Utility Types

## 1. Что такое type и interface?

```typescript
// ========== type (псевдоним типа) ==========
// Более универсальный инструмент для создания типов
type User = {
  name: string;
  age: number;
};

// ========== interface ==========
// Способ описания формы объекта
interface IUser {
  name: string;
  age: number;
}

// ========== Основные отличия ==========
// 1. interface только для объектов
// 2. type для всего: объекты, примитивы, union, tuple и т.д.
// 3. interface можно расширять (declaration merging)
// 4. type нельзя переопределить
```

### 1.1 Когда лучше использовать type?

```typescript
// ========== Union Types (объединение) ==========
type Status = "success" | "error" | "loading";
type ID = string | number;

// ========== Intersection Types (пересечение) ==========
type A = { a: number };
type B = { b: number };
type C = A & B; // { a: number; b: number }

// ========== Tuple (кортеж) ==========
type Point = [number, number];
type RGB = [number, number, number];

// ========== Примитивы ==========
type UserId = string;
type Age = number;

// ========== Mapped Types ==========
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// ========== Conditional Types ==========
type IsString<T> = T extends string ? true : false;
```

### 1.2 Когда лучше использовать interface?

```typescript
// ========== Для описания сущностей (объектов) ==========
interface User {
  id: number;
  name: string;
  email: string;
}

// ========== Для API-ответов ==========
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// ========== Для классов ==========
interface Animal {
  name: string;
  makeSound(): void;
}

class Dog implements Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  makeSound() {
    console.log("Гав!");
  }
}
```

### 1.3 Как сделать поле интерфейса не обязательным?

```typescript
interface User {
  name: string;
  age: number;
  phone?: string; // ? — опциональное поле
  address?: string;
}

const user1: User = {
  name: "Anna",
  age: 25,
  // phone и address можно не указывать
};

const user2: User = {
  name: "John",
  age: 30,
  phone: "+123456789",
};
```

### 1.4 Для чего нужно ключевое слово extends в интерфейсе?

```typescript
// extends — наследование интерфейсов

interface Animal {
  name: string;
  age: number;
}

interface Dog extends Animal {
  breed: string; // добавляем новое поле
  bark(): void; // добавляем новый метод
}

// Dog теперь имеет: name, age, breed, bark()

const myDog: Dog = {
  name: "Рекс",
  age: 3,
  breed: "Овчарка",
  bark() {
    console.log("Гав!");
  },
};

// ========== Множественное наследование ==========
interface Flyable {
  fly(): void;
}

interface Swimmable {
  swim(): void;
}

interface Duck extends Flyable, Swimmable {
  quack(): void;
}
```

### 1.5 Для чего нужно ключевое слово implements в интерфейсе?

```typescript
// implements — класс обязуется реализовать интерфейс

interface User {
  name: string;
  age: number;
  greet(): string;
}

// ❌ Без implements — нет проверки
class Person1 {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  greet() {
    return `Привет, я ${this.name}`;
  }
}

// ✅ С implements — TypeScript проверит, что все поля и методы реализованы
class Person implements User {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Привет, я ${this.name}`;
  }
}

// ❌ Ошибка! Не хватает метода greet
class WrongPerson implements User {
  name: string;
  age: number;
  // Property 'greet' is missing in type 'WrongPerson'
}
```

---

## 2. Операторы & и |

### 2.1 & — Intersection Types (Пересечение типов)

```typescript
// Объединяет типы — объект должен иметь ВСЕ свойства из объединяемых типов

type HasName = { name: string };
type HasAge = { age: number };
type HasEmail = { email: string };

type User = HasName & HasAge & HasEmail;
// { name: string; age: number; email: string }

const user: User = {
  name: "Anna",
  age: 25,
  email: "anna@example.com",
};

// ========== Пример с функциями ==========
type Logger = {
  log(message: string): void;
};

type Sender = {
  send(message: string): void;
};

type LoggerSender = Logger & Sender;

const service: LoggerSender = {
  log(msg) {
    console.log("LOG:", msg);
  },
  send(msg) {
    console.log("SEND:", msg);
  },
};
```

### 2.2 | — Union Types (Объединение типов)

```typescript
// Значение может быть одним из перечисленных типов

type Status = "loading" | "success" | "error";
type ID = string | number;
type Result = number | boolean | string;

let status: Status = "loading";
status = "success"; // ✅ OK
status = "error"; // ✅ OK
// status = "pending"; // ❌ Ошибка!

// ========== Discriminated Union (размеченное объединение) ==========
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}

console.log(getArea({ kind: "circle", radius: 5 })); // 78.54
console.log(getArea({ kind: "square", side: 4 })); // 16
```

---

## 3. Utility Types (вспомогательные типы)

### 3.1 Record<K, V>

```typescript
// Record — создаёт объект с ключами типа K и значениями типа V

type Users = Record<string, { name: string; age: number }>;

const users: Users = {
  user1: { name: "Anna", age: 25 },
  user2: { name: "John", age: 30 },
};

// Аналог через индексную сигнатуру:
type Users2 = {
  [key: string]: { name: string; age: number };
};

// Record с конкретными ключами:
type StatusMap = Record<"loading" | "success" | "error", string>;

const messages: StatusMap = {
  loading: "Загрузка...",
  success: "Успешно!",
  error: "Ошибка!",
};
```

### 3.2 Omit<T, K>

```typescript
// Omit — исключает указанные поля из типа

interface User {
  id: number;
  name: string;
  age: number;
  password: string;
  email: string;
}

// Создаём тип без конфиденциальных данных
type PublicUser = Omit<User, "password" | "email">;

const publicUser: PublicUser = {
  id: 1,
  name: "Anna",
  age: 25,
  // password и email нельзя указать
};

// Пример с функцией:
function createUser(data: Omit<User, "id">): User {
  return {
    id: Date.now(),
    ...data,
  };
}
```

### 3.3 Pick<T, K>

```typescript
// Pick — выбирает только указанные поля из типа

interface User {
  id: number;
  name: string;
  age: number;
  email: string;
  password: string;
}

type UserPreview = Pick<User, "id" | "name">;

const preview: UserPreview = {
  id: 1,
  name: "Anna",
  // другие поля нельзя указать
};

// Пример в API:
function getUserPreview(user: User): Pick<User, "id" | "name"> {
  return {
    id: user.id,
    name: user.name,
  };
}
```

### 3.4 Partial<T>

```typescript
// Partial — делает все поля необязательными

interface User {
  id: number;
  name: string;
  age: number;
}

// Все поля становятся опциональными
type PartialUser = Partial<User>;

const updateData: PartialUser = {
  name: "Новое имя",
  // id и age можно не указывать
};

// Пример: функция обновления пользователя
function updateUser(id: number, updates: Partial<User>): User {
  // updates может содержать любые поля User
  return { id, name: "default", age: 0, ...updates };
}
```

### 3.5 Required<T>

```typescript
// Required — делает все поля обязательными (противоположность Partial)

interface User {
  id?: number;
  name?: string;
  age?: number;
}

type RequiredUser = Required<User>;
// { id: number; name: string; age: number; }

const user: RequiredUser = {
  id: 1,
  name: "Anna",
  age: 25,
  // все поля обязательны
};
```

### 3.6 ReturnType<T>

```typescript
// ReturnType — получает тип возвращаемого значения функции

function getUser() {
  return { id: 1, name: "Anna", age: 25 };
}

type UserType = ReturnType<typeof getUser>;
// { id: number; name: string; age: number; }

const user: UserType = {
  id: 2,
  name: "John",
  age: 30,
};

// С асинхронной функцией:
async function fetchData(): Promise<{ data: string }> {
  return { data: "hello" };
}

type FetchResult = ReturnType<typeof fetchData>; // Promise<{ data: string }>
```

### 3.7 Parameters<T>

```typescript
// Parameters — получает тип параметров функции в виде кортежа

function sum(a: number, b: number): number {
  return a + b;
}

type SumParams = Parameters<typeof sum>; // [number, number]

function logUser(name: string, age: number, isAdmin: boolean): void {
  console.log(name, age, isAdmin);
}

type LogParams = Parameters<typeof logUser>; // [string, number, boolean]

// Пример: обёртка для функции
function withLogging<T extends (...args: any[]) => any>(fn: T): (...args: Parameters<T>) => ReturnType<T> {
  return (...args: Parameters<T>) => {
    console.log("Вызов функции с аргументами:", args);
    return fn(...args);
  };
}

const loggedSum = withLogging(sum);
loggedSum(5, 3); // "Вызов функции с аргументами: [5, 3]" → 8
```

### 3.8 Readonly<T>

```typescript
// Readonly — делает все поля только для чтения

interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = {
  id: 1,
  name: "Anna",
};

user.id = 2; // ❌ Ошибка! Cannot assign to 'id' because it is a read-only property
user.name = "John"; // ❌ Ошибка!

// Пример: конфигурация приложения
const config: Readonly<{ apiUrl: string; timeout: number }> = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
};
```

---

## 4. Практические задачи (Discriminated Union)

### Задача 1: User + верификация

```typescript
// если verified = true → есть verifiedAt
// если verified = false → verifiedAt не должно быть

type User = { id: number; name: string; verified: true; verifiedAt: string } | { id: number; name: string; verified: false };

// Проверка:
const verifiedUser: User = {
  id: 1,
  name: "Anna",
  verified: true,
  verifiedAt: "2024-01-15",
};

const unverifiedUser: User = {
  id: 2,
  name: "John",
  verified: false,
  // verifiedAt не нужен
};
```

### Задача 2: Product + скидка

```typescript
// если hasDiscount = true → есть discount
// если false → discount запрещён

type Product = { price: number; hasDiscount: true; discount: number } | { price: number; hasDiscount: false };

// Проверка:
const discountedProduct: Product = {
  price: 100,
  hasDiscount: true,
  discount: 20,
};

const regularProduct: Product = {
  price: 100,
  hasDiscount: false,
};
```

### Задача 3: Order статус

```typescript
// pending, shipped, delivered, cancelled
// при delivered → deliveredAt
// при cancelled → reason

type Order =
  | { status: "pending" }
  | { status: "shipped" }
  | { status: "delivered"; deliveredAt: string }
  | { status: "cancelled"; reason: string };

// Проверка:
const pendingOrder: Order = { status: "pending" };
const deliveredOrder: Order = { status: "delivered", deliveredAt: "2024-01-15" };
const cancelledOrder: Order = { status: "cancelled", reason: "Не оплачен" };
```

### Задача 4: Payment

```typescript
// card → cardNumber, cvv
// paypal → email
// crypto → walletAddress

type Payment =
  | { method: "card"; cardNumber: string; cvv: string }
  | { method: "paypal"; email: string }
  | { method: "crypto"; walletAddress: string };

// Проверка:
const cardPayment: Payment = {
  method: "card",
  cardNumber: "1234-5678-9012-3456",
  cvv: "123",
};

const paypalPayment: Payment = {
  method: "paypal",
  email: "user@example.com",
};
```

### Задача 5: API Response (Generic)

```typescript
// success → data: T
// error → error: string
// loading → ничего

type ApiResponse<T> = { status: "loading" } | { status: "success"; data: T } | { status: "error"; error: string };

// Использование:
type User = { id: number; name: string };

function fetchUser(): ApiResponse<User> {
  // может вернуть любой из трёх вариантов
  return { status: "loading" };
  // или { status: "success", data: { id: 1, name: "Anna" } }
  // или { status: "error", error: "Ошибка сети" }
}
```

### Задача 6: CartItem

```typescript
// digital → quantity всегда 1
// physical → quantity любое число

type CartItem = { product: { type: "digital" }; quantity: 1 } | { product: { type: "physical" }; quantity: number };

// Проверка:
const digitalItem: CartItem = {
  product: { type: "digital" },
  quantity: 1,
};

const physicalItem: CartItem = {
  product: { type: "physical" },
  quantity: 5,
};
```

### Задача 7: Filter

```typescript
// либо нет фильтра цены
// либо ОБЯЗАТЕЛЬНО есть и min, и max

type Filter = {} | { min: number; max: number };

// Проверка:
const noFilter: Filter = {};
const priceFilter: Filter = { min: 10, max: 100 };
```

### Задача 8: Access (Роли + доступ)

```typescript
// admin → permissions: "*"
// manager → permissions: string[]
// user → нет permissions

type Access = { role: "admin"; permissions: "*" } | { role: "manager"; permissions: string[] } | { role: "user" };

// Проверка:
const admin: Access = { role: "admin", permissions: "*" };
const manager: Access = { role: "manager", permissions: ["read", "write"] };
const user: Access = { role: "user" };
```

### Задача 9: Event (Лог событий)

```typescript
// login → ip
// logout → ничего
// purchase → orderId, amount

type Event = { type: "login"; ip: string } | { type: "logout" } | { type: "purchase"; orderId: number; amount: number };

// Проверка:
const loginEvent: Event = { type: "login", ip: "192.168.1.1" };
const logoutEvent: Event = { type: "logout" };
const purchaseEvent: Event = { type: "purchase", orderId: 123, amount: 99.99 };
```

### Задача 10: Notification

```typescript
// email → email
// sms → phone

type Notification = { type: "email"; email: string } | { type: "sms"; phone: string };

// Проверка:
const emailNotification: Notification = { type: "email", email: "user@example.com" };
const smsNotification: Notification = { type: "sms", phone: "+123456789" };
```

### Задача 11: Widget (Комбинированный тип)

```typescript
// button → onClick
// input → placeholder

type Widget = {
  id: string;
} & ({ type: "button"; onClick: () => void } | { type: "input"; placeholder: string });

// Проверка:
const buttonWidget: Widget = {
  id: "btn1",
  type: "button",
  onClick: () => console.log("Clicked!"),
};

const inputWidget: Widget = {
  id: "input1",
  type: "input",
  placeholder: "Введите текст...",
};
```

---

## 5. Шпаргалка по Utility Types

| Utility Type     | Описание                           | Пример                                   |
| ---------------- | ---------------------------------- | ---------------------------------------- |
| `Partial<T>`     | Все поля необязательные            | `Partial<User>`                          |
| `Required<T>`    | Все поля обязательные              | `Required<User>`                         |
| `Readonly<T>`    | Все поля только для чтения         | `Readonly<User>`                         |
| `Pick<T, K>`     | Выбрать указанные поля             | `Pick<User, "id" \| "name">`             |
| `Omit<T, K>`     | Исключить указанные поля           | `Omit<User, "password">`                 |
| `Record<K, V>`   | Объект с ключами K и значениями V  | `Record<string, number>`                 |
| `ReturnType<T>`  | Тип возвращаемого значения функции | `ReturnType<typeof fn>`                  |
| `Parameters<T>`  | Типы параметров функции (кортеж)   | `Parameters<typeof fn>`                  |
| `Exclude<T, U>`  | Исключить типы из объединения      | `Exclude<"a" \| "b", "a">` → `"b"`       |
| `Extract<T, U>`  | Выбрать типы из объединения        | `Extract<"a" \| "b", "a">` → `"a"`       |
| `NonNullable<T>` | Убрать null и undefined            | `NonNullable<string \| null>` → `string` |

---

## 6. Шпаргалка: type vs interface

| Характеристика       | type         | interface |
| -------------------- | ------------ | --------- |
| Объекты              | ✅           | ✅        |
| Примитивы            | ✅           | ❌        |
| Union (`\|`)         | ✅           | ❌        |
| Intersection (`&`)   | ✅           | ❌        |
| Tuple                | ✅           | ❌        |
| Расширение (extends) | ❌ (через &) | ✅        |
| Declaration merging  | ❌           | ✅        |
| Mapped types         | ✅           | ❌        |
| Conditional types    | ✅           | ❌        |
| implements в классах | ✅           | ✅        |

---
