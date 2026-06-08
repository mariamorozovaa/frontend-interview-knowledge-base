# Классы, ООП, SOLID, DRY, KISS

## Вопросы с собеседования

### 1. Что такое ООП?

```js
// ООП (Объектно-ориентированное программирование) — парадигма,
// где программа строится вокруг объектов, которые содержат:
// - данные (свойства)
// - поведение (методы)

// ========== 4 основных принципа ООП ==========

// 1. Инкапсуляция — скрытие внутренней логики
class User {
  #password; // приватное поле

  constructor(password) {
    this.#password = password;
  }

  checkPassword(input) {
    return this.#password === input;
  }
}
const user = new User("secret123");
console.log(user.#password); // Ошибка! Недоступно снаружи

// 2. Наследование — объекты могут наследовать поведение
class Animal {
  speak() {
    return "Издает звук";
  }
}
class Cat extends Animal {
  speak() {
    return "Мяу!"; // переопределение метода
  }
}
const cat = new Cat();
console.log(cat.speak()); // "Мяу!"

// 3. Полиморфизм — один метод, разное поведение
class Bird extends Animal {
  speak() {
    return "Чирик!";
  }
}
const animals = [new Animal(), new Cat(), new Bird()];
animals.forEach((animal) => console.log(animal.speak()));
// "Издает звук", "Мяу!", "Чирик!"

// 4. Абстракция — скрываем сложность, показываем только нужное
class CoffeeMachine {
  #waterAmount = 0;

  #boilWater() {
    console.log("Кипятим воду...");
  }

  makeCoffee(amount) {
    this.#waterAmount = amount;
    this.#boilWater();
    console.log("Кофе готов!");
  }
}
const machine = new CoffeeMachine();
machine.makeCoffee(200); // Просто вызываем метод, не думая о деталях
```

### 2. О чем гласит принцип SOLID?

```js
// SOLID — 5 принципов написания гибкого и поддерживаемого кода

// ========== S — Single Responsibility Principle ==========
// У класса должна быть одна причина для изменения

// ❌ НЕПРАВИЛЬНО:
class UserService {
  saveUser(user) {
    /* сохраняет в БД */
  }
  sendEmail(email, message) {
    /* отправляет email */
  }
  logError(error) {
    /* логирует ошибку */
  }
}

// ✅ ПРАВИЛЬНО:
class UserRepository {
  saveUser(user) {
    /* сохраняет в БД */
  }
}
class EmailService {
  sendEmail(email, message) {
    /* отправляет email */
  }
}
class Logger {
  logError(error) {
    /* логирует ошибку */
  }
}

// ========== O — Open/Closed Principle ==========
// Код открыт для расширения, но закрыт для изменения

// ❌ НЕПРАВИЛЬНО:
class Discount {
  calculate(type, price) {
    if (type === "summer") return price * 0.9;
    if (type === "blackfriday") return price * 0.5;
    // при добавлении нового типа нужно менять код
  }
}

// ✅ ПРАВИЛЬНО:
class Discount {
  calculate(price) {
    return price;
  }
}
class SummerDiscount extends Discount {
  calculate(price) {
    return price * 0.9;
  }
}
class BlackFridayDiscount extends Discount {
  calculate(price) {
    return price * 0.5;
  }
}

// ========== L — Liskov Substitution Principle ==========
// Подкласс должен корректно заменять родителя

// ❌ НЕПРАВИЛЬНО:
class Bird {
  fly() {
    console.log("Летит");
  }
}
class Penguin extends Bird {
  fly() {
    throw new Error("Пингвины не летают!");
  }
}

// ✅ ПРАВИЛЬНО:
class Bird {}
class FlyingBird extends Bird {
  fly() {
    console.log("Летит");
  }
}
class Penguin extends Bird {}

// ========== I — Interface Segregation Principle ==========
// Лучше много маленьких интерфейсов, чем один большой

// ❌ НЕПРАВИЛЬНО:
class Worker {
  work() {}
  eat() {}
  sleep() {}
}

// ✅ ПРАВИЛЬНО:
class Workable {
  work() {}
}
class Eatable {
  eat() {}
}
class Sleepable {
  sleep() {}
}

// ========== D — Dependency Inversion Principle ==========
// Зависим от абстракций, а не от конкретных реализаций

// ❌ НЕПРАВИЛЬНО:
class MySQLDatabase {
  query(sql) {
    /* MySQL */
  }
}
class UserService {
  constructor() {
    this.db = new MySQLDatabase(); // жесткая зависимость
  }
}

// ✅ ПРАВИЛЬНО:
class Database {
  query(sql) {}
}
class UserService {
  constructor(db) {
    // зависимость через конструктор
    this.db = db;
  }
}
```

### 2.1. Как принципы SOLID интерпретировать для React?

```js
// ========== SRP в React ==========
// Компонент делает одну вещь

// ❌ НЕПРАВИЛЬНО:
function UserPage() {
    const [users, setUsers] = useState([]);
    useEffect(() => {
        fetch("/api/users").then(res => res.json()).then(setUsers);
    }, []);
    return (
        <div>
            {users.map(user => <div key={user.id}>{user.name}</div>)}
        </div>
    );
}

// ✅ ПРАВИЛЬНО:
function useUsers() {  // кастомный хук для загрузки
    const [users, setUsers] = useState([]);
    useEffect(() => {
        fetch("/api/users").then(res => res.json()).then(setUsers);
    }, []);
    return users;
}
function UserList({ users }) {  // компонент только для отображения
    return users.map(user => <div key={user.id}>{user.name}</div>);
}
function UserPage() {
    const users = useUsers();
    return <UserList users={users} />;
}

// ========== OCP в React ==========
// Расширяем через props / children

function Button({ variant, children }) {
    const variants = {
        primary: "bg-blue-500",
        secondary: "bg-gray-500",
        danger: "bg-red-500"
    };
    return <button className={variants[variant]}>{children}</button>;
}
<Button variant="primary">Сохранить</Button>
<Button variant="danger">Удалить</Button>

// ========== DIP в React ==========
// Используем зависимости через props

function UserComponent({ api, children }) {
    const [user, setUser] = useState(null);
    useEffect(() => {
        api.getUser().then(setUser);
    }, [api]);
    return children(user);
}

// Передаем зависимость через пропсы
<UserComponent api={userAPI}>
    {user => <div>{user?.name}</div>}
</UserComponent>
```

### 3. Принцип DRY (Don't Repeat Yourself)

```js
// DRY — не дублируй код. Каждая часть знаний должна быть в одном месте.

// ❌ НЕПРАВИЛЬНО (повторение кода):
function calculateOrderTotal(items) {
  let total = 0;
  for (let item of items) {
    total += item.price * item.quantity;
  }
  return total;
}

function calculateCartTotal(cart) {
  let total = 0;
  for (let item of cart) {
    total += item.price * item.quantity;
  }
  return total;
}

// ✅ ПРАВИЛЬНО:
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
// Переиспользуем одну функцию везде

// В React: кастомные хуки и переиспользуемые компоненты
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  return [value, setValue];
}

// Используем в разных компонентах:
const [theme, setTheme] = useLocalStorage("theme", "light");
const [user, setUser] = useLocalStorage("user", null);
```

### 4. Принцип KISS (Keep It Simple, Stupid)

```js
// KISS — делай максимально просто. Сложность — враг.

// ❌ НЕПРАВИЛЬНО (усложнение):
if (!!value === true) {
}
const result = arr.filter((item) => (item !== undefined ? true : false));
function isEven(num) {
  return num % 2 === 0 ? true : false;
}

// ✅ ПРАВИЛЬНО (просто):
if (value) {
}
const result = arr.filter((item) => item !== undefined);
function isEven(num) {
  return num % 2 === 0;
}

// ❌ НЕПРАВИЛЬНО (избыточная абстракция):
class StringReverser {
  static reverse(str) {
    return str.split("").reverse().join("");
  }
}
console.log(StringReverser.reverse("hello"));

// ✅ ПРАВИЛЬНО (просто):
const reverse = (str) => str.split("").reverse().join("");
console.log(reverse("hello"));

// ❌ НЕПРАВИЛЬНО (сложный код):
let x = 0;
let y = 0;
let z = 0;
if (a && (b || c) && !d) {
  x = 1;
  y = 2;
  z = 3;
}

// ✅ ПРАВИЛЬНО:
const isValid = a && (b || c) && !d;
if (isValid) {
  const values = { x: 1, y: 2, z: 3 };
}
```

### 5. Для чего были придуманы классы?

```js
// Классы — это синтаксический сахар над прототипами.
// Они нужны, чтобы писать код понятнее, структурировать логику,
// использовать ООП-подход.

// До ES6 (через прототипы):
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  console.log(this.name + " издает звук");
};
const dog = new Animal("Рекс");

// После ES6 (через классы):
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log(`${this.name} издает звук`);
  }
}
const dog = new Animal("Рекс");

// Классы дают:
// 1. Более чистый синтаксис
// 2. Наследование через extends
// 3. Приватные поля (#)
// 4. Статические методы (static)
// 5. Геттеры/сеттеры
```

### 5.1. Как создать приватный метод класса?

```js
// Приватные поля и методы создаются через символ #

class User {
  #password; // приватное поле
  #secretKey = "abc123"; // приватное поле с начальным значением

  constructor(password) {
    this.#password = password;
    this.name = "Гость";
  }

  // приватный метод
  #hashPassword() {
    return this.#password.split("").reverse().join("");
  }

  // публичный метод, использующий приватный
  login(inputPassword) {
    if (this.#hashPassword() === inputPassword) {
      console.log("Вход выполнен");
    }
  }
}

const user = new User("secret");
console.log(user.#password); // Ошибка! SyntaxError
console.log(user.#secretKey); // Ошибка!
user.#hashPassword(); // Ошибка!
user.login("terces"); // "Вход выполнен" (но приватный метод вызван внутри)
```

### 5.2. Для чего нужно ключевое слово static?

```js
// static — метод или свойство принадлежит самому классу,
// а не его экземплярам.

class MathUtils {
  static PI = 3.14159;
  static E = 2.71828;

  static sum(a, b) {
    return a + b;
  }

  static multiply(a, b) {
    return a * b;
  }

  // фабричный метод
  static createEmptyUser() {
    return new User("guest", "");
  }
}

// Вызываем на самом классе, а не на экземпляре
console.log(MathUtils.sum(5, 3)); // 8
console.log(MathUtils.PI); // 3.14159

// ❌ Так нельзя:
// const math = new MathUtils();
// math.sum(1, 2); // TypeError

// Использование в React:
class Button extends React.Component {
  static defaultProps = {
    variant: "primary",
  };
  static propTypes = {
    onClick: PropTypes.func,
  };
}

// Применение static:
// - Утилиты (MathUtils.sum)
// - Фабрики (User.createEmpty())
// - Константы (Colors.PRIMARY)
// - Кэширование (Cache.getInstance())
```

### 5.3. Геттеры и сеттеры (get / set)

```js
// Геттеры и сеттеры позволяют контролировать доступ к свойствам

class User {
  constructor(firstName, lastName) {
    this._firstName = firstName;
    this._lastName = lastName;
    this._age = 0;
  }

  // Геттер — как свойство, но вычисляется при чтении
  get fullName() {
    return `${this._firstName} ${this._lastName}`;
  }

  // Сеттер — валидация при установке значения
  set age(value) {
    if (value < 0) {
      throw new Error("Возраст не может быть отрицательным");
    }
    if (value > 150) {
      throw new Error("Некорректный возраст");
    }
    this._age = value;
  }

  get age() {
    return this._age;
  }

  // Геттер без сеттера (только для чтения)
  get isAdult() {
    return this._age >= 18;
  }
}

const user = new User("Иван", "Петров");
console.log(user.fullName); // "Иван Петров" (как свойство, без вызова)

user.age = 25;
console.log(user.age); // 25
console.log(user.isAdult); // true

user.age = -5; // Error! Возраст не может быть отрицательным

// ========== Реальный пример: валидация формы ==========
class FormData {
  constructor() {
    this._email = "";
  }

  set email(value) {
    if (!value.includes("@")) {
      throw new Error("Некорректный email");
    }
    this._email = value;
  }

  get email() {
    return this._email;
  }
}
```

---

## Практические задачи с решениями

### Задача 1. LRU Cache (Least Recently Used)

```js
// LRU Cache — кэш, который хранит ограниченное количество элементов.
// При превышении размера удаляется самый старый (давно не используемый).

class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map(); // Map сохраняет порядок вставки
  }

  get(key) {
    if (!this.cache.has(key)) return -1;

    // Обновляем порядок: удаляем и вставляем заново
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);

    return value;
  }

  put(key, value) {
    // Если ключ уже есть, удаляем старую запись
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    // Добавляем новую
    this.cache.set(key, value);

    // Если превысили лимит, удаляем первый (самый старый)
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
}

// Проверка:
const cache = new LRUCache(2);
cache.put(1, "A");
cache.put(2, "B");
console.log(cache.get(1)); // "A"
cache.put(3, "C"); // удалится ключ 2
console.log(cache.get(2)); // -1
console.log(cache.get(3)); // "C"
```

### Задача 2. Browser History

```js
// История браузера с возможностью назад/вперед

class BrowserHistory {
  constructor(homepage) {
    this.history = [homepage];
    this.currentIndex = 0;
  }

  visit(url) {
    // При переходе на новую страницу удаляем всю историю "вперед"
    this.history = this.history.slice(0, this.currentIndex + 1);
    this.history.push(url);
    this.currentIndex++;
  }

  back() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
    }
    return this.current();
  }

  forward() {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
    }
    return this.current();
  }

  current() {
    return this.history[this.currentIndex];
  }
}

// Проверка:
const browser = new BrowserHistory("google.com");
browser.visit("youtube.com");
browser.visit("github.com");
console.log(browser.current()); // "github.com"
browser.back();
console.log(browser.current()); // "youtube.com"
browser.forward();
console.log(browser.current()); // "github.com"
browser.visit("stackoverflow.com");
console.log(browser.current()); // "stackoverflow.com"
console.log(browser.history); // ["google.com", "youtube.com", "github.com", "stackoverflow.com"]
```

### Задача 3. Stack (реализация без встроенных методов)

```js
// Реализация стека вручную (LIFO — Last In, First Out)

class Stack {
  constructor() {
    this.items = {}; // объект для хранения данных
    this.count = 0; // количество элементов
  }

  push(value) {
    this.items[this.count] = value;
    this.count++;
  }

  pop() {
    if (this.isEmpty()) return null;
    this.count--;
    const value = this.items[this.count];
    delete this.items[this.count];
    return value;
  }

  peek() {
    if (this.isEmpty()) return null;
    return this.items[this.count - 1];
  }

  size() {
    return this.count;
  }

  isEmpty() {
    return this.count === 0;
  }

  // Дополнительный метод: очистить стек
  clear() {
    this.items = {};
    this.count = 0;
  }

  // Дополнительный метод: получить все элементы
  toArray() {
    const result = [];
    for (let i = 0; i < this.count; i++) {
      result.push(this.items[i]);
    }
    return result;
  }
}

// Проверка:
const stack = new Stack();

stack.push(10);
stack.push(20);
stack.push(30);

console.log(stack.peek()); // 30
console.log(stack.pop()); // 30
console.log(stack.pop()); // 20
console.log(stack.size()); // 1
console.log(stack.isEmpty()); // false
console.log(stack.pop()); // 10
console.log(stack.isEmpty()); // true
console.log(stack.pop()); // null
console.log(stack.peek()); // null
```

---

## Шпаргалка по ООП в JS

| Концепция         | Синтаксис                    | Назначение                           |
| ----------------- | ---------------------------- | ------------------------------------ |
| Класс             | `class Name {}`              | Шаблон для создания объектов         |
| Конструктор       | `constructor() {}`           | Инициализация свойств при создании   |
| Наследование      | `class Child extends Parent` | Получение свойств и методов родителя |
| Приватное поле    | `#field`                     | Доступно только внутри класса        |
| Приватный метод   | `#method() {}`               | Доступен только внутри класса        |
| Статический метод | `static method() {}`         | Вызывается на самом классе           |
| Геттер            | `get prop() {}`              | Вычисляемое свойство при чтении      |
| Сеттер            | `set prop(val) {}`           | Валидация при записи                 |
| super             | `super.method()`             | Вызов метода родителя                |

---

## Принципы SOLID кратко

| Буква | Название              | Суть                                        |
| ----- | --------------------- | ------------------------------------------- |
| S     | Single Responsibility | Одна ответственность у класса               |
| O     | Open/Closed           | Открыт для расширения, закрыт для изменения |
| L     | Liskov Substitution   | Подкласс заменяет родителя                  |
| I     | Interface Segregation | Много маленьких интерфейсов                 |
| D     | Dependency Inversion  | Зависимость от абстракций, не от реализаций |

---
