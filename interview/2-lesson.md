# Дополнительные вопросы: решение задач, массивы, классы, hoisting, оптимизация, Lazy

## 1. По решению задач: сначала решить хоть как-то, потом улучшить

Главный принцип решения задач:

1. Сделай работающее решение (любым способом)
2. Проанализируй проблему
3. Улучши (оптимизируй, упрости, перепиши)

```js
// Задача: найти сумму элементов массива

// Шаг 1: Решение "в лоб" (работает!)
function sumArray(arr) {
  let sum = 0;
  for (let i = 0; i < arr.length; i++) {
    sum = sum + arr[i];
  }
  return sum;
}

// Шаг 2: Улучшаем (reduce)
function sumArray(arr) {
  return arr.reduce((acc, num) => acc + num, 0);
}
```

=========! Почему это важно =========!

1. Страх идеального решения блокирует прогресс
2. Легче рефакторить работающий код, чем писать идеальный с нуля
3. Работающее неоптимальное решение лучше идеального неработающего

=========! Стратегия решения задач =========!

1. brute force (перебор) — решает, но медленно
2. оптимизация по времени (убрать вложенные циклы)
3. оптимизация по памяти (убрать лишние структуры)

---

## 2. Сложность операций с массивом (Big O)

```js
// =========! Добавление/удаление в конец = O(1) =========!
const arr = [1, 2, 3];
arr.push(4); // O(1) — добавляем в конец
arr.pop(); // O(1) — удаляем с конца

// =========! Добавление/удаление в начало = O(n) =========!
arr.unshift(0); // O(n) — все элементы сдвигаются вправо
arr.shift(); // O(n) — все элементы сдвигаются влево

// =========! Добавление/удаление в середину = O(n) =========!
arr.splice(1, 0, 99); // O(n) — вставка в середину, остальные сдвигаются

// =========! Поиск элемента = O(n) =========!
arr.indexOf(3); // O(n) — линейный поиск
arr.includes(2); // O(n)
arr.find((x) => x > 2); // O(n)

// =========! Доступ по индексу = O(1) =========!
arr[2]; // O(1) — моментальный доступ

// =========! Поиск в Set/Map = O(1) =========!
const set = new Set([1, 2, 3]);
set.has(2); // O(1) — быстрее, чем indexOf!
```

=========! Итог =========!
Операция | Сложность
-------------------------|----------
Доступ по индексу | O(1)
push/pop (конец) | O(1)
unshift/shift (начало) | O(n)
splice (середина) | O(n)
indexOf/includes/find | O(n)

---

## 3. Для чего добавили классы в JS?

```js
// Классы в JS появились в ES6 (2015) как синтаксический сахар над прототипами

// =========! До классов (прототипы) =========!
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function () {
  console.log(this.name + " издаёт звук");
};

const dog = new Animal("Рекс");
dog.speak(); // "Рекс издаёт звук"

// =========! После классов =========!
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} издаёт звук`);
  }
}

const dog = new Animal("Рекс");
dog.speak(); // "Рекс издаёт звук"

// =========! Зачем добавили классы =========!
// 1. Знакомый синтаксис для разработчиков из других языков (Java, C++, PHP)
// 2. Чище и понятнее прототипного наследования
// 3. Упрощает ООП в JavaScript
// 4. Лучшая поддержка в IDE (автодополнение, рефакторинг)
// 5. Наследование через extends (было сложно через прототипы)

// =========! Пример наследования =========!
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    console.log(`${this.name} издаёт звук`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name); // вызов конструктора родителя
    this.breed = breed;
  }
  speak() {
    console.log(`${this.name} гавкает!`);
  }
  bark() {
    console.log("Гав-гав!");
  }
}

const rex = new Dog("Рекс", "Овчарка");
rex.speak(); // "Рекс гавкает!"
rex.bark(); // "Гав-гав!"

// =========! Классы — это всё ещё прототипы =========!
console.log(typeof Animal); // "function"
console.log(Animal.prototype === dog.__proto__); // true
```

---

## 4. Задачка на hoisting (поднятие переменных)

```js
function hoist() {
  console.log(c); // undefined (var c поднялось, но значение ещё не присвоено)
  a = 20;
  var b = 100;
  var c = 50;
}

hoist();
console.log(a); // 20 (a стала глобальной, потому что объявлена без var/let/const)
console.log(b); // ❌ ReferenceError: b is not defined (b существует только внутри функции)

// =========! Как это работает =========!
// Функция hoist интерпретируется так:
function hoist() {
  var b; // поднятие (hoisting) var b
  var c; // поднятие var c

  console.log(c); // undefined (значение ещё не присвоено)
  a = 20; // переменная без var → становится глобальной
  b = 100;
  c = 50;
}

// =========! Дополнительные примеры для тренировки =========!

// Пример 1
console.log(x); // undefined
var x = 5;
console.log(x); // 5

// Пример 2
console.log(y); // ❌ ReferenceError (let не доступен до объявления)
let y = 5;

// Пример 3
foo(); // "привет" (function declaration поднимается полностью)
function foo() {
  console.log("привет");
}

// Пример 4
bar(); // ❌ TypeError: bar is not a function (var bar поднято, но значение undefined)
var bar = function () {
  console.log("пока");
};

// Пример 5
console.log(typeof undeclaredVar); // "undefined" (без ошибки)
console.log(undeclaredVar); // ❌ ReferenceError

// Пример 6
var a = 1;
function test() {
  console.log(a); // undefined (не 1! переопределение var a внутри функции)
  var a = 2;
  console.log(a); // 2
}
test();

// Как это работает:
// function test() {
//     var a;        // поднятие, перекрывает глобальную a
//     console.log(a); // undefined
//     a = 2;
//     console.log(a); // 2
// }
```

---

## 5. Lazy (Ленивая загрузка)

```js
// Lazy Loading — техника, при которой код/данные загружаются только когда они реально нужны

// =========! 1. Lazy Loading компонентов в React =========!
import { lazy, Suspense } from "react";

// Компонент загрузится только когда он понадобится
const HeavyComponent = lazy(() => import("./HeavyComponent"));

function App() {
  const [show, setShow] = useState(false);

  return (
    <div>
      <button onClick={() => setShow(true)}>Показать</button>
      <Suspense fallback={<div>Загрузка...</div>}>{show && <HeavyComponent />}</Suspense>
    </div>
  );
}

// =========! 2. Lazy Loading изображений =========!
// Браузер загружает изображение только когда оно появляется в области видимости
<img src="image.jpg" loading="lazy" alt="описание" />;

// JavaScript реализация (Intersection Observer)
const images = document.querySelectorAll("img[data-src]");
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});
images.forEach((img) => observer.observe(img));

// =========! 3. Lazy Loading маршрутов (React Router) =========!
const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));
const Contact = lazy(() => import("./pages/Contact"));

// =========! 4. Lazy Loading модулей (динамический импорт) =========!
button.addEventListener("click", async () => {
  const module = await import("./heavy-module.js");
  module.doSomething();
});

// =========! Зачем нужен Lazy Loading =========!
// 1. Уменьшает начальный размер бандла
// 2. Ускоряет первоначальную загрузку страницы
// 3. Экономит трафик пользователя
// 4. Улучшает метрики (FCP, LCP)

// =========! Когда использовать =========!
// ✅ Тяжёлые компоненты, которых нет в первом экране
// ✅ Модальные окна, попапы, тултипы
// ✅ Страницы, до которых пользователь может не дойти
// ✅ Изображения (loading="lazy")

// ❌ Критичные для первого экрана компоненты
// ❌ Очень маленькие компоненты (оверхед на загрузку больше пользы)
```

---

## 6. Оптимизация: сайт долго грузится — алгоритм действий

```js
// =========! Алгоритм диагностики и оптимизации =========!

// Шаг 1: Измерить текущие метрики
// - Chrome DevTools → Lighthouse
// - PageSpeed Insights (web.dev)
// - WebPageTest

// Ключевые метрики (Core Web Vitals):
// - LCP (Largest Contentful Paint) — < 2.5с (самый большой элемент)
// - FID (First Input Delay) — < 100мс (отклик на первое взаимодействие)
// - CLS (Cumulative Layout Shift) — < 0.1 (скачки вёрстки)

// =========! Шаг 2: Анализ бандла =========!
// 1. Посмотреть размер бандла:
//    - npm run build -- --profile
//    - Использовать Bundle Analyzer: source-map-explorer, webpack-bundle-analyzer

// 2. Вопросы к себе:
//    - Есть ли библиотеки, которые можно заменить на лёгкие альтернативы?
//    - Не дублируются ли библиотеки?
//    - Не импортируется ли вся библиотека, когда нужна только функция?

// =========! Шаг 3: Оптимизация ресурсов =========!
// 1. Изображения:
//    - Форматы: WebP, AVIF (вместо PNG/JPG)
//    - Сжатие: imagemin, Squoosh
//    - Адаптивные изображения: srcset, picture
//    - Ленивая загрузка: loading="lazy"

// 2. Шрифты:
//    - display: swap (чтобы не блокировал отрисовку)
//    - subset (оставить только нужные символы)

// 3. CSS:
//    - Минификация (cssnano, csso)
//    - Удалить неиспользуемый CSS (PurgeCSS)
//    - Critical CSS (для первого экрана)

// =========! Шаг 4: Оптимизация JS =========!
// 1. Code Splitting (разделение кода)
//    - Dynamic import()
//    - React.lazy + Suspense

// 2. Tree Shaking (удаление мёртвого кода)
//    - Используйте ES modules (import/export, не require)
//    - Проверьте sideEffects: false в package.json

// 3. Минификация (Terser, esbuild)

// 4. Убрать дублирование кода

// =========! Шаг 5: Кэширование =========!
// 1. HTTP кэширование (Cache-Control, ETag)
// 2. Service Worker (PWA, offline first)
// 3. Content Delivery Network (CDN)

// =========! Шаг 6: Оптимизация сети =========!
// 1. HTTP/2 или HTTP/3
// 2. Сжатие (gzip, brotli)
// 3. Preload / Preconnect / Prefetch

// Пример:
<link rel="preconnect" href="https://api.example.com">
<link rel="preload" href="critical.css" as="style">
<link rel="prefetch" href="next-page.js">

// =========! Шаг 7: Оптимизация рендеринга =========!
// 1. Избегать рефлоу и репейнта
// 2. Использовать transform/opacity для анимаций
// 3. Virtualization для длинных списков (react-window)

// =========! Чек-лист для быстрой проверки =========!
// [ ] 1. Сжал изображения?
// [ ] 2. Включил gzip/brotli на сервере?
// [ ] 3. Настроил кэширование?
// [ ] 4. Разделил код на чанки?
// [ ] 5. Удалил неиспользуемые зависимости?
// [ ] 6. Сделал ленивую загрузку?
// [ ] 7. Оптимизировал шрифты?
// [ ] 8. Подключил CDN?
// [ ] 9. Исправил метрики CLS (скачки вёрстки)?
// [ ] 10. Проверил через Lighthouse?

// =========! Инструменты =========!
// - Lighthouse (встроенный в DevTools)
// - PageSpeed Insights
// - WebPageTest
// - Bundle Analyzer
// - Coverage (вкладка в DevTools → непокрытый код)
// - Performance (вкладка в DevTools → запись и анализ)
```

---

## Шпаргалка

### Сложность операций с массивами

| Операция               | Сложность |
| ---------------------- | --------- |
| `arr[i]`               | O(1)      |
| `push` / `pop`         | O(1)      |
| `unshift` / `shift`    | O(n)      |
| `splice` (середина)    | O(n)      |
| `indexOf` / `includes` | O(n)      |

### Hoisting

| Объявление             | Поведение                                       |
| ---------------------- | ----------------------------------------------- |
| `var`                  | Поднимается (значение `undefined`)              |
| `let` / `const`        | Поднимается, но в TDZ (ошибка до инициализации) |
| `function declaration` | Поднимается полностью                           |
| `function expression`  | Как переменная (`var` или `let`)                |

### Оптимизация (короткая версия)

1. **Измерить** (Lighthouse, DevTools)
2. **Анализировать** (Bundle Analyzer)
3. **Оптимизировать** (изображения, шрифты, CSS)
4. **Разделить** (code splitting, lazy loading)
5. **Кэшировать** (HTTP cache, Service Worker)
6. **Сжать** (gzip, brotli)
7. **Проверить** (повторный Lighthouse)

---
