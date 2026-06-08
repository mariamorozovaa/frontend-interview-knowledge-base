# Рекурсия: возведение в степень, факториал, зеркальная строка, flatten массива, вывод дерева

## Задача 1. Возведение в степень (pow)

```js
/**
 * Возводит число x в натуральную степень n
 * @param {number} x - основание
 * @param {number} n - показатель степени (натуральное число)
 * @returns {number} - результат x в степени n
 *
 * Рекурсивное определение:
 * pow(x, 0) = 1
 * pow(x, n) = x * pow(x, n - 1)
 */

function pow(x, n) {
  // Базовый случай: любое число в степени 0 равно 1
  if (n === 0) return 1;

  // Рекурсивный случай: x * x^(n-1)
  return x * pow(x, n - 1);
}

// Проверка:
console.log(pow(2, 2)); // 4
console.log(pow(2, 3)); // 8
console.log(pow(2, 4)); // 16
console.log(pow(5, 0)); // 1
console.log(pow(3, 1)); // 3

// ========== Альтернативная версия (хвостовая рекурсия) ==========
function powTail(x, n, acc = 1) {
  if (n === 0) return acc;
  return powTail(x, n - 1, acc * x);
}

console.log(powTail(2, 3)); // 8

// ========== Итеративная версия (без рекурсии) ==========
function powIterative(x, n) {
  let result = 1;
  for (let i = 0; i < n; i++) {
    result *= x;
  }
  return result;
}

console.log(powIterative(2, 3)); // 8
```

---

## Задача 2. Факториал числа

```js
/**
 * Вычисляет факториал числа n (n!)
 * @param {number} n - неотрицательное целое число
 * @returns {number} - факториал числа n
 *
 * Определение:
 * 0! = 1
 * n! = n * (n-1) * (n-2) * ... * 1
 *
 * Рекурсивное определение:
 * factorial(0) = 1
 * factorial(n) = n * factorial(n - 1)
 */

function deepFactorial(n) {
  // Базовый случай: факториал 0 и 1 равен 1
  if (n <= 1) return 1;

  // Рекурсивный случай: n * (n-1)!
  return n * deepFactorial(n - 1);
}

// Проверка:
console.log(deepFactorial(5)); // 120 (5 * 4 * 3 * 2 * 1)
console.log(deepFactorial(0)); // 1
console.log(deepFactorial(1)); // 1
console.log(deepFactorial(3)); // 6

// ========== Альтернативная версия (хвостовая рекурсия) ==========
function deepFactorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return deepFactorialTail(n - 1, acc * n);
}

console.log(deepFactorialTail(5)); // 120

// ========== Итеративная версия ==========
function deepFactorialIterative(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

console.log(deepFactorialIterative(5)); // 120
```

---

## Задача 3. Зеркальная строка со скобками

```js
/**
 * Создаёт зеркальную строку, где открывающие скобки заменяются на закрывающие,
 * а остальные символы отражаются в обратном порядке.
 *
 * @param {string} str - исходная строка (буквы и открывающие скобки)
 * @returns {string} - строка с добавленной зеркальной половиной
 *
 * Пример: "(abc(def(g" → "(abc(def(gg)fed)cba)"
 *
 * Алгоритм:
 * 1. Отражаем каждый символ: '(' → ')', остальные символы остаются как есть
 * 2. Разворачиваем полученную строку в обратном порядке
 * 3. Добавляем в конец исходной строки
 */

// ========== Версия 1: с использованием рекурсии ==========
function makeMirrorStringRecursive(str) {
  // Базовый случай: пустая строка
  if (str.length === 0) return "";

  const first = str[0];
  const rest = makeMirrorStringRecursive(str.slice(1));

  // Преобразуем открывающую скобку в закрывающую
  const mirrored = first === "(" ? ")" : first;

  // Собираем: исходный символ + рекурсивный результат + зеркальный символ
  return first + rest + mirrored;
}

// ========== Версия 2: без рекурсии (более эффективная) ==========
function makeMirrorString(str) {
  let result = str;

  // Проходим по строке от конца к началу
  for (let i = str.length - 1; i >= 0; i--) {
    const char = str[i];
    // Заменяем '(' на ')', остальные символы оставляем
    const mirroredChar = char === "(" ? ")" : char;
    result += mirroredChar;
  }

  return result;
}

// ========== Версия 3: через массивы (более читаемая) ==========
function makeMirrorStringArray(str) {
  const mirrorPart = str
    .split("")
    .reverse()
    .map((char) => (char === "(" ? ")" : char))
    .join("");

  return str + mirrorPart;
}

// Проверка:
console.log(makeMirrorString("(abc(def(g"));
// "(abc(def(gg)fed)cba)"

console.log(makeMirrorString("("));
// "(("

console.log(makeMirrorString("abc"));
// "abccba"

console.log(makeMirrorString("(a(b(c"));
// "(a(b(cc)b)a)"

// Пошаговое объяснение работы:
// Исходная:    (  a  b  c  (  d  e  f  (  g
// Зеркальная:  g  )  f  e  d  )  c  b  a  )
// Результат:   (abc(def(gg)fed)cba)
```

---

## Задача 4. Flatten — разворачивание вложенного массива

```js
/**
 * Преобразует вложенный массив произвольной глубины в линейный массив
 * @param {Array} arr - вложенный массив чисел
 * @returns {Array} - плоский массив чисел
 *
 * Условия:
 * - Работает с любым уровнем вложенности
 * - Все элементы — числа
 * - Сохраняется порядок
 */

// ========== Версия 1: рекурсивная ==========
function flatten(arr) {
  let result = [];

  for (const element of arr) {
    if (Array.isArray(element)) {
      // Рекурсивно разворачиваем вложенный массив
      result.push(...flatten(element));
    } else {
      result.push(element);
    }
  }

  return result;
}

// Проверка:
console.log(flatten([1, [2, 3, [4]], 5]));
// [1, 2, 3, 4, 5]

console.log(flatten([1, [2, 3], [[2], 5], 6]));
// [1, 2, 3, 2, 5, 6]

console.log(flatten([[[9]], [1, 2], [[8]]]));
// [9, 1, 2, 8]

console.log(flatten([]));
// []

// ========== Версия 2: через reduce ==========
function flattenReduce(arr) {
  return arr.reduce((acc, element) => {
    if (Array.isArray(element)) {
      acc.push(...flattenReduce(element));
    } else {
      acc.push(element);
    }
    return acc;
  }, []);
}

console.log(flattenReduce([1, [2, 3, [4]], 5]));
// [1, 2, 3, 4, 5]

// ========== Версия 3: итеративная (без рекурсии, через стек) ==========
function flattenIterative(arr) {
  const result = [];
  const stack = [...arr];

  while (stack.length) {
    const current = stack.pop();
    if (Array.isArray(current)) {
      stack.push(...current);
    } else {
      result.unshift(current);
    }
  }

  return result;
}

console.log(flattenIterative([1, [2, 3, [4]], 5]));
// [1, 2, 3, 4, 5]

// ========== Версия 4: с использованием flat (встроенный метод) ==========
function flattenBuiltIn(arr) {
  return arr.flat(Infinity);
}

console.log(flattenBuiltIn([1, [2, 3, [4]], 5]));
// [1, 2, 3, 4, 5]
```

---

## Задача 5. Вывод дерева файлов и папок

```js
/**
 * Рекурсивная функция для вывода структуры файлов и папок
 * @param {Object} node - узел дерева (папка или файл)
 * @param {number} depth - уровень вложенности (количество отступов)
 * @returns {string} - отформатированная строка
 */

const data = {
  name: "folder",
  children: [
    { name: "file1.txt" },
    { name: "file2.txt" },
    {
      name: "images",
      children: [
        { name: "image.png" },
        {
          name: "vacation",
          children: [{ name: "crocodile.png" }, { name: "penguin.png" }],
        },
      ],
    },
    { name: "shopping-list.pdf" },
  ],
};

// ========== Версия 1: рекурсивная (с возвратом строки) ==========
function printTreeRecursive(node, indent = 0) {
  let result = "  ".repeat(indent) + node.name + "\n";

  if (node.children) {
    for (const child of node.children) {
      result += printTreeRecursive(child, indent + 1);
    }
  }

  return result;
}

console.log(printTreeRecursive(data));
// folder
//   file1.txt
//   file2.txt
//   images
//     image.png
//     vacation
//       crocodile.png
//       penguin.png
//   shopping-list.pdf

// ========== Версия 2: рекурсивная (с прямым выводом в консоль) ==========
function printTreeDirect(node, indent = 0) {
  console.log("  ".repeat(indent) + node.name);

  if (node.children) {
    for (const child of node.children) {
      printTreeDirect(child, indent + 1);
    }
  }
}

printTreeDirect(data);

// ========== Версия 3: итеративная (без рекурсии, через стек) ==========
function printTreeIterative(root) {
  const stack = [{ node: root, indent: 0 }];

  while (stack.length) {
    const { node, indent } = stack.pop();

    // Выводим текущий узел
    console.log("  ".repeat(indent) + node.name);

    // Добавляем детей в стек (в обратном порядке, чтобы сохранить порядок)
    if (node.children) {
      for (let i = node.children.length - 1; i >= 0; i--) {
        stack.push({ node: node.children[i], indent: indent + 1 });
      }
    }
  }
}

console.log("\n=== Итеративный вывод ===");
printTreeIterative(data);

// ========== Версия 4: форматированный вывод с указанием типа ==========
function printTreeWithTypes(node, indent = 0) {
  const isFolder = !!node.children;
  const icon = isFolder ? "📁 " : "📄 ";
  console.log("  ".repeat(indent) + icon + node.name);

  if (node.children) {
    for (const child of node.children) {
      printTreeWithTypes(child, indent + 1);
    }
  }
}

console.log("\n=== Вывод с иконками ===");
printTreeWithTypes(data);
// 📁 folder
//   📄 file1.txt
//   📄 file2.txt
//   📁 images
//     📄 image.png
//     📁 vacation
//       📄 crocodile.png
//       📄 penguin.png
//   📄 shopping-list.pdf

// ========== Версия 5: без рекурсии (по условию задачи) ==========
// Условие: "Решение должно учитывать любую вложенность элементов
//           (т.е. не должно содержать рекурсивные вызовы)."

function printTreeNoRecursion(root) {
  // Используем массив как стек
  const stack = [{ node: root, indent: 0, visited: false }];
  let output = "";

  while (stack.length) {
    const { node, indent, visited } = stack.pop();

    if (!visited) {
      // Добавляем имя узла с отступом
      output += "  ".repeat(indent) + node.name + "\n";

      // Если есть дети, добавляем их в стек для последующей обработки
      if (node.children) {
        // Добавляем метку, что узел обработан
        stack.push({ node, indent, visited: true });

        // Добавляем детей в обратном порядке для правильного порядка вывода
        for (let i = node.children.length - 1; i >= 0; i--) {
          stack.push({
            node: node.children[i],
            indent: indent + 1,
            visited: false,
          });
        }
      }
    }
  }

  return output;
}

console.log("\n=== Вывод без рекурсии ===");
console.log(printTreeNoRecursion(data));
```

---

## Шпаргалка по рекурсии

| Задача        | Базовый случай        | Рекурсивный случай         | Сложность |
| ------------- | --------------------- | -------------------------- | --------- |
| pow(x, n)     | n === 0 → 1           | x \* pow(x, n - 1)         | O(n)      |
| factorial(n)  | n ≤ 1 → 1             | n \* factorial(n - 1)      | O(n)      |
| mirror string | str.length === 0 → "" | first + recurse + mirrored | O(n²)     |
| flatten(arr)  | не массив → push      | spread + recurse           | O(n)      |
| print tree    | нет детей → вывод     | рекурсивный вызов детей    | O(n)      |

---

## Важные моменты

### 1. Рекурсия и стек вызовов

```js
// При рекурсивном вызове каждый новый вызов добавляется в Call Stack
// pow(2, 3):
// - pow(2, 3) ждёт pow(2, 2)
//   - pow(2, 2) ждёт pow(2, 1)
//     - pow(2, 1) ждёт pow(2, 0)
//       - pow(2, 0) возвращает 1
//     - возвращает 2 * 1 = 2
//   - возвращает 2 * 2 = 4
// - возвращает 2 * 4 = 8
```

### 2. Хвостовая рекурсия

```js
// Хвостовая рекурсия — рекурсивный вызов в конце функции
// Некоторые движки JS могут оптимизировать её (PTC)

function tailFactorial(n, acc = 1) {
  if (n <= 1) return acc;
  return tailFactorial(n - 1, acc * n);
}
```

### 3. Альтернативы рекурсии

```js
// Итерация (циклы) — обычно более производительна
// Не вызывает переполнение стека

function factorialIterative(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}
```

---
