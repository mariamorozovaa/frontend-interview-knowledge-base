# Решения задач по типизации TypeScript

## Задача 1: Система комментариев

```typescript
// ------------------------------------------------------------
// 1) Пользователь
// ------------------------------------------------------------

type User = {
  id: number;
  username: string;
  avatar?: string; // необязательное поле
  role: "user" | "admin";
  readonly createdAt: string; // только для чтения
};

// Проверка:
const user: User = {
  id: 1,
  username: "Den",
  role: "user",
  createdAt: "2024-01-01",
  // avatar — необязательный
};

// ------------------------------------------------------------
// 2) Комментарий
// ------------------------------------------------------------

type Comment = {
  id: number;
  authorId: number;
  message: string;
  createdAt: string;
  updatedAt?: string; // необязательное
  parentId: number | null; // null = корневой комментарий
};

// Проверка:
const comment: Comment = {
  id: 10,
  authorId: 1,
  message: "Hello",
  createdAt: "2024-01-01",
  parentId: null,
};

// ------------------------------------------------------------
// 3) Статус комментария
// ------------------------------------------------------------

type CommentStatus = "visible" | "hidden" | "deleted";

const status: CommentStatus = "visible";

// ------------------------------------------------------------
// 4) Расширенный комментарий (для UI)
// ------------------------------------------------------------

type FullComment = Comment & {
  author: User;
  replies: Comment[];
  status: CommentStatus;
};

// Проверка:
const fullComment: FullComment = {
  id: 10,
  authorId: 1,
  message: "Hello",
  createdAt: "2024-01-01",
  parentId: null,
  author: {
    id: 1,
    username: "Den",
    role: "user",
    createdAt: "2024-01-01",
  },
  replies: [],
  status: "visible",
};

// ------------------------------------------------------------
// 5) Частичное обновление комментария
// ------------------------------------------------------------
// Вариант 1: ручное указание
// type UpdateCommentPayload = {
//     message?: string;
//     status?: CommentStatus;
// };

// Вариант 2: автоматический (через Pick)
type UpdateCommentPayload = Pick<FullComment, "message" | "status">;

// Вариант 3: частичный (если бы нужно было обновлять больше полей)
// type UpdateCommentPayload = Partial<Pick<FullComment, 'message' | 'status'>>;

// Проверка:
const updateData: UpdateCommentPayload = {
  message: "Updated text",
  status: "hidden",
};

// ------------------------------------------------------------
// 6) Универсальная функция обновления
// ------------------------------------------------------------

function updateComment(comment: FullComment, payload: UpdateCommentPayload): FullComment {
  return { ...comment, ...payload };
}

const updated = updateComment(fullComment, {
  message: "New message",
});

// ------------------------------------------------------------
// 7) Сложность — автоматическое обновление типа
// ------------------------------------------------------------
// Уже реализовано через Pick!
// Если в FullComment изменится тип message или status,
// UpdateCommentPayload автоматически подхватит изменения

export {};
```

---

## Задача 2: Система уведомлений

```typescript
// ------------------------------------------------------------
// 1) Пользователь
// ------------------------------------------------------------

type User = {
  id: number;
  username: string;
  email: string;
  role: "user" | "admin";
  readonly createdAt: string;
};

// Проверка:
const user: User = {
  id: 1,
  username: "Den",
  email: "den@example.com",
  role: "user",
  createdAt: "2024-04-01",
};

// ------------------------------------------------------------
// 2) Тип уведомления (Discriminated Union)
// ------------------------------------------------------------

// Базовая часть уведомления
type BaseNotification = {
  id: string;
  userId: number;
  createdAt: string;
};

// Payload для каждого типа
type SystemPayload = {
  message: string;
};

type OrderPayload = {
  orderId: string;
  status: "pending" | "shipped" | "delivered" | "cancelled";
};

type MarketingPayload = {
  title: string;
  discount: number;
  expiresAt: string;
};

// Размеченное объединение (discriminated union)
type Notification =
  | (BaseNotification & { type: "system"; payload: SystemPayload })
  | (BaseNotification & { type: "order"; payload: OrderPayload })
  | (BaseNotification & { type: "marketing"; payload: MarketingPayload });

// Проверка:
const systemNotification: Notification = {
  id: "n1",
  type: "system",
  userId: 1,
  createdAt: "2024-04-01",
  payload: {
    message: "Система будет обновляться сегодня в 22:00",
  },
};

const orderNotification: Notification = {
  id: "n2",
  type: "order",
  userId: 1,
  createdAt: "2024-04-01",
  payload: {
    orderId: "abc123",
    status: "shipped",
  },
};

const marketingNotification: Notification = {
  id: "n3",
  type: "marketing",
  userId: 1,
  createdAt: "2024-04-01",
  payload: {
    title: "Скидка 20% на все товары",
    discount: 20,
    expiresAt: "2024-04-10",
  },
};

// ------------------------------------------------------------
// 3) Полный объект уведомления для UI
// ------------------------------------------------------------

type FullNotification = Notification & {
  user: User;
};

const fullNotification: FullNotification = {
  id: "n2",
  type: "order",
  userId: 1,
  createdAt: "2024-04-01",
  payload: {
    orderId: "abc123",
    status: "shipped",
  },
  user: user,
};

export {};
```

---

## Задача 3: Utility Types

```typescript
interface IEmployee {
  id: number;
  name: string;
  email: string;
  salary: number;
  department: string;
}

// 1. Тип для формы обновления — все поля необязательные
type UpdateEmployee = Partial<IEmployee>;

// 2. Тип только с name и email — для отображения в списке
type EmployeePreview = Pick<IEmployee, "name" | "email">;

// 3. Тип без salary — для публичного профиля
type PublicEmployee = Omit<IEmployee, "salary">;

// 4. Словарь: ключ — название отдела, значение — количество сотрудников
type DepartmentCount = Record<string, number>;

// Проверка:
const update: UpdateEmployee = { name: "Bob" };

const preview: EmployeePreview = { name: "Alice", email: "alice@mail.com" };

const publicProfile: PublicEmployee = {
  id: 1,
  name: "Alice",
  email: "alice@mail.com",
  department: "Dev",
};

const stats: DepartmentCount = {
  Engineering: 12,
  Design: 5,
  Marketing: 8,
};

export {};
```

---

## Задача 4: Функция wrapInArray (Generic)

```typescript
// Generic функция, которая оборачивает значение в массив
function wrapInArray<T>(value: T): T[] {
  return [value];
}

// Проверка:
const nums = wrapInArray(42); // number[]
const strs = wrapInArray("hello"); // string[]
const bools = wrapInArray(true); // boolean[]

console.log(nums); // [42]
console.log(strs); // ['hello']
console.log(bools); // [true]

export {};
```

---

## Задача 5: Функция filterByField (Generic + keyof)

```typescript
const users = [
  { id: 1, name: "Alice", age: 25, isActive: true },
  { id: 2, name: "Bob", age: 30, isActive: false },
  { id: 3, name: "Charlie", age: 25, isActive: true },
];

// Универсальная функция фильтрации
function filterByField<T, K extends keyof T>(items: T[], key: K, value: T[K]): T[] {
  return items.filter((item) => item[key] === value);
}

// Проверка:
filterByField(users, "age", 25); // ✅
filterByField(users, "name", "Alice"); // ✅
// filterByField(users, "age", "25");   // ❌ ошибка: тип string не подходит под number
// filterByField(users, "test", 123);   // ❌ ошибка: 'test' нет в ключах

export {};
```

---

## Задача 6: Система интернет-магазина

```typescript
// ------------------------------------------------------------
// 1) Пользователь
// ------------------------------------------------------------

type User = {
  id: number;
  username: string;
  email: string;
  role: "customer" | "admin";
  readonly createdAt: string;
};

const user: User = {
  id: 1,
  username: "Den",
  email: "den@example.com",
  role: "customer",
  createdAt: "2024-04-01",
};

// ------------------------------------------------------------
// 2) Товар
// ------------------------------------------------------------

type Product = {
  id: number;
  title: string;
  price: number;
  stock: number;
  category: string;
  discount?: number; // опционально
  description?: string; // опционально
};

const product: Product = {
  id: 101,
  title: "JavaScript Book",
  price: 1000,
  stock: 5,
  category: "books",
  discount: 10,
};

// ------------------------------------------------------------
// 3) Заказ
// ------------------------------------------------------------

type OrderItem = {
  productId: number;
  quantity: number;
};

type OrderStatus = "pending" | "processing" | "shipped" | "delivered" | "cancelled";

type Order = {
  id: string; // строковый id
  userId: number;
  items: OrderItem[];
  status: OrderStatus;
  comment?: string; // опционально
  updatedAt?: string; // опционально (дата обновления)
};

const order: Order = {
  id: "abc123",
  userId: 1,
  items: [
    { productId: 101, quantity: 2 },
    { productId: 102, quantity: 1 },
  ],
  status: "pending",
  comment: "Пожалуйста, доставьте до 18:00",
};

// ------------------------------------------------------------
// 4) Платёж
// ------------------------------------------------------------

type PaymentMethod = "card" | "cash" | "paypal" | "crypto";
type PaymentStatus = "pending" | "completed" | "failed" | "refunded";

type Payment = {
  id: string;
  orderId: string;
  amount: number;
  method: PaymentMethod;
  status: PaymentStatus;
  paidAt?: string; // опционально
};

const payment: Payment = {
  id: "pay001",
  orderId: "abc123",
  amount: 2100,
  method: "card",
  status: "completed",
  paidAt: "2024-04-02",
};

// ------------------------------------------------------------
// 5) Полный заказ для UI
// ------------------------------------------------------------

type FullOrder = Order & {
  user: User;
  products: Product[];
  payments: Payment[];
};

const fullOrder: FullOrder = {
  id: "abc123",
  userId: 1,
  items: [
    { productId: 101, quantity: 2 },
    { productId: 102, quantity: 1 },
  ],
  status: "pending",
  comment: "Пожалуйста, доставьте до 18:00",
  user: user,
  products: [
    {
      id: 101,
      title: "JavaScript Book",
      price: 1000,
      stock: 5,
      category: "books",
      discount: 10,
    },
    {
      id: 102,
      title: "TypeScript Guide",
      price: 1200,
      stock: 2,
      category: "books",
    },
  ],
  payments: [
    {
      id: "pay001",
      orderId: "abc123",
      amount: 2100,
      method: "card",
      status: "completed",
      paidAt: "2024-04-02",
    },
  ],
};

export {};
```

---

## Шпаргалка по использованным типам

| Задача | Использованные типы                                     |
| ------ | ------------------------------------------------------- |
| 1      | `type`, `interface`, `Pick`, `readonly`, `?` (optional) |
| 2      | `Discriminated Union`, `&` (intersection), `type`       |
| 3      | `Partial`, `Pick`, `Omit`, `Record`                     |
| 4      | `Generic <T>`                                           |
| 5      | `Generic <T, K extends keyof T>`, `T[K]`                |
| 6      | `type`, `readonly`, `?`, `&`, `Discriminated Union`     |
