# Проверка знаний JavaScript: решения задач

## Задача 1. Метод groupBy для массива

```js
/**
 * Реализует метод groupBy для массивов.
 * Группирует элементы массива по ключу, возвращаемому переданной функцией.
 *
 * @param {Function} fn - функция, возвращающая ключ для группировки
 * @returns {Object} - объект, где ключи — результаты fn, значения — массивы элементов
 */

Array.prototype.groupBy = function (fn) {
  const result = {};

  for (const item of this) {
    const key = fn(item);

    if (!result[key]) {
      result[key] = [];
    }

    result[key].push(item);
  }

  return result;
};

// Проверка:
const array1 = [{ id: 1 }, { id: 1 }, { id: 2 }];

const fn = (item) => item.id;
console.log(array1.groupBy(fn));
// {
//   '1': [ { id: 1 }, { id: 1 } ],
//   '2': [ { id: 2 } ]
// }

const array2 = [1, 2, 3];
console.log(array2.groupBy(String));
// {
//   '1': [1],
//   '2': [2],
//   '3': [3]
// }

// ========== Альтернативная версия через reduce ==========
Array.prototype.groupByReduce = function (fn) {
  return this.reduce((acc, item) => {
    const key = fn(item);
    if (!acc[key]) acc[key] = [];
    acc[key].push(item);
    return acc;
  }, {});
};

console.log(array1.groupByReduce(fn));
// тот же результат
```

---

## Задача 2. Поиск пропущенного числа

```js
/**
 * Находит пропущенное число в массиве от 0 до N.
 * Массив не отсортирован, ровно одно число пропущено.
 *
 * @param {number[]} nums - массив чисел от 0 до N с одним пропуском
 * @returns {number} - пропущенное число
 *
 * Алгоритмы:
 * 1. Через сумму (самый эффективный) — O(n) по времени, O(1) по памяти
 * 2. Через XOR — O(n), O(1)
 * 3. Через сортировку — O(n log n)
 */

// ========== Способ 1: через сумму ==========
function solveWithSum(nums) {
  const n = nums.length; // так как одно число пропущено, длина = N (или N+1?)
  // Если массив [0..N] с одним пропуском, то длина = N
  // Сумма всех чисел от 0 до N = N * (N + 1) / 2
  const expectedSum = (n * (n + 1)) / 2;
  const actualSum = nums.reduce((acc, num) => acc + num, 0);
  return expectedSum - actualSum;
}

// ========== Способ 2: через XOR (исключающее ИЛИ) ==========
function solveWithXOR(nums) {
  let xor = 0;
  const n = nums.length;

  // XOR всех чисел от 0 до n
  for (let i = 0; i <= n; i++) {
    xor ^= i;
  }

  // XOR всех чисел в массиве
  for (const num of nums) {
    xor ^= num;
  }

  return xor;
}

// ========== Способ 3: через сортировку ==========
function solveWithSort(nums) {
  nums.sort((a, b) => a - b);

  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== i) {
      return i;
    }
  }

  return nums.length;
}

// Проверка:
const nums = [1, 4, 3, 0, 6, 5]; // пропущено 2
console.log(solveWithSum(nums)); // 2
console.log(solveWithXOR(nums)); // 2
console.log(solveWithSort(nums)); // 2

// Дополнительные тесты:
console.log(solveWithSum([0, 1, 2, 3, 5])); // 4
console.log(solveWithSum([0, 1, 2, 3, 4, 5])); // 6 (пропущено последнее)
console.log(solveWithSum([1, 2, 3, 4])); // 0 (пропущено первое)
```

---

## Задача 3. Обход дерева (получение всех значений)

```js
/**
 * Возвращает массив всех значений вершин дерева (обход в глубину).
 *
 * @param {Object} tree - узел дерева с полями value и children
 * @returns {number[]} - массив всех значений
 */

const tree = {
  value: 1,
  children: [
    {
      value: 2,
      children: [{ value: 4 }, { value: 5 }],
    },
    {
      value: 3,
      children: [{ value: 6 }, { value: 7 }],
    },
  ],
};

// ========== Способ 1: рекурсивный обход (DFS) ==========
function getTreeValuesRecursive(someTree) {
  let result = [someTree.value];

  if (someTree.children) {
    for (const child of someTree.children) {
      result.push(...getTreeValuesRecursive(child));
    }
  }

  return result;
}

console.log(getTreeValuesRecursive(tree));
// [1, 2, 4, 5, 3, 6, 7]

// ========== Способ 2: итеративный (стек) — без рекурсии ==========
function getTreeValuesIterative(someTree) {
  const result = [];
  const stack = [someTree];

  while (stack.length) {
    const node = stack.pop();
    result.push(node.value);

    if (node.children) {
      // Добавляем детей в обратном порядке для сохранения порядка
      for (let i = node.children.length - 1; i >= 0; i--) {
        stack.push(node.children[i]);
      }
    }
  }

  return result;
}

console.log(getTreeValuesIterative(tree));
// [1, 2, 4, 5, 3, 6, 7]

// ========== Способ 3: обход в ширину (BFS — очередь) ==========
function getTreeValuesBFS(someTree) {
  const result = [];
  const queue = [someTree];

  while (queue.length) {
    const node = queue.shift();
    result.push(node.value);

    if (node.children) {
      queue.push(...node.children);
    }
  }

  return result;
}

console.log(getTreeValuesBFS(tree));
// [1, 2, 3, 4, 5, 6, 7]  (в ширину порядок другой)

// ========== Способ 4: с обработкой пустых children ==========
function getTreeValuesSafe(someTree) {
  if (!someTree) return [];

  const result = [someTree.value];
  const children = someTree.children || [];

  for (const child of children) {
    result.push(...getTreeValuesSafe(child));
  }

  return result;
}
```

---

## Задача 4. Корзина товаров (DOM + JS)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Корзина товаров</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
      }
      .products-list,
      .cart-list {
        list-style: none;
        padding: 0;
      }
      .products-list li,
      .cart-list li {
        padding: 10px;
        margin: 5px 0;
        background: #f0f0f0;
        border-radius: 5px;
        display: flex;
        justify-content: space-between;
        align-items: center;
      }
      button {
        padding: 5px 10px;
        cursor: pointer;
        border: none;
        border-radius: 3px;
      }
      .add-btn {
        background: #4caf50;
        color: white;
      }
      .remove-btn {
        background: #ff4444;
        color: white;
      }
      .quantity-btn {
        background: #ddd;
        margin: 0 3px;
      }
      .cart-item {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
      }
      .cart-item-info {
        flex: 1;
      }
      .cart-item-actions {
        display: flex;
        gap: 5px;
      }
      .summary {
        margin-top: 20px;
        padding: 15px;
        background: #e0e0e0;
        border-radius: 5px;
        font-weight: bold;
      }
      h3 {
        margin-top: 30px;
      }
    </style>
  </head>
  <body>
    <h2>Товары</h2>
    <ul id="products" class="products-list">
      <li data-id="1" data-price="100">
        Товар 1 – 100₽
        <button class="add-btn">Добавить</button>
      </li>
      <li data-id="2" data-price="250">
        Товар 2 – 250₽
        <button class="add-btn">Добавить</button>
      </li>
      <li data-id="3" data-price="80">
        Товар 3 – 80₽
        <button class="add-btn">Добавить</button>
      </li>
    </ul>

    <h3>Корзина:</h3>
    <ul id="cart" class="cart-list"></ul>

    <div id="summary" class="summary">Товаров: 0, Сумма: 0₽</div>

    <script>
      // Данные корзины: { productId, name, price, quantity }
      let cart = [];

      // Элементы DOM
      const productsContainer = document.querySelector("#products");
      const cartContainer = document.querySelector("#cart");
      const summaryDiv = document.querySelector("#summary");

      // Функция обновления отображения корзины
      function renderCart() {
        if (cart.length === 0) {
          cartContainer.innerHTML = "<li>Корзина пуста</li>";
          updateSummary();
          return;
        }

        cartContainer.innerHTML = "";

        cart.forEach((item, index) => {
          const li = document.createElement("li");
          const totalItemPrice = item.price * item.quantity;

          li.innerHTML = `
                    <div class="cart-item">
                        <div class="cart-item-info">
                            ${item.name} – ${item.quantity} шт. – ${totalItemPrice}₽
                        </div>
                        <div class="cart-item-actions">
                            <button class="quantity-btn" data-index="${index}" data-delta="-1">-</button>
                            <span>${item.quantity}</span>
                            <button class="quantity-btn" data-index="${index}" data-delta="1">+</button>
                            <button class="remove-btn" data-index="${index}">Удалить</button>
                        </div>
                    </div>
                `;

          cartContainer.appendChild(li);
        });

        // Добавляем обработчики для кнопок изменения количества и удаления
        document.querySelectorAll(".quantity-btn").forEach((btn) => {
          btn.addEventListener("click", (e) => {
            const index = parseInt(btn.dataset.index);
            const delta = parseInt(btn.dataset.delta);
            updateQuantity(index, delta);
          });
        });

        document.querySelectorAll(".remove-btn").forEach((btn) => {
          btn.addEventListener("click", (e) => {
            const index = parseInt(btn.dataset.index);
            removeItem(index);
          });
        });

        updateSummary();
      }

      // Обновление количества товара
      function updateQuantity(index, delta) {
        const newQuantity = cart[index].quantity + delta;

        if (newQuantity <= 0) {
          // Если количество становится 0 или меньше, удаляем товар
          cart.splice(index, 1);
        } else {
          cart[index].quantity = newQuantity;
        }

        renderCart();
      }

      // Полное удаление товара из корзины
      function removeItem(index) {
        cart.splice(index, 1);
        renderCart();
      }

      // Обновление итоговой суммы и количества
      function updateSummary() {
        const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
        const totalSum = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);

        summaryDiv.innerHTML = `Товаров: ${totalItems}, Сумма: ${totalSum}₽`;
      }

      // Добавление товара в корзину
      function addToCart(productId, productName, productPrice) {
        const existingItem = cart.find((item) => item.id === productId);

        if (existingItem) {
          existingItem.quantity++;
        } else {
          cart.push({
            id: productId,
            name: productName,
            price: productPrice,
            quantity: 1,
          });
        }

        renderCart();
      }

      // Обработчики для кнопок "Добавить" на товарах
      productsContainer.addEventListener("click", (e) => {
        const addBtn = e.target.closest(".add-btn");
        if (!addBtn) return;

        const li = addBtn.closest("li");
        const productId = parseInt(li.dataset.id);
        const productPrice = parseInt(li.dataset.price);
        const productName = li.firstChild.textContent.trim().split("–")[0].trim();

        addToCart(productId, productName, productPrice);
      });

      // Инициализация
      renderCart();
    </script>
  </body>
</html>
```

---

## Шпаргалка по решениям

| Задача                | Техника             | Сложность | Ключевые моменты                      |
| --------------------- | ------------------- | --------- | ------------------------------------- |
| **groupBy**           | Объект-аккумулятор  | O(n)      | `Array.prototype`, динамический ключ  |
| **Пропущенное число** | Сумма / XOR         | O(n)      | Математический подход, без сортировки |
| **Обход дерева**      | DFS (рекурсия/стек) | O(n)      | Рекурсия или итеративный стек         |
| **Корзина товаров**   | Массив объектов     | O(n)      | DOM-события, рендер, состояние        |

---

## Важные моменты для проверки

### 1. groupBy

- ✅ Метод добавляется в `Array.prototype`
- ✅ Используется `this` для доступа к массиву
- ✅ Возвращает объект, а не массив

### 2. Поиск пропущенного числа

- ✅ Работает с любым пропущенным числом (в том числе 0 или N)
- ✅ Не требует сортировки
- ✅ O(n) по времени, O(1) по памяти

### 3. Обход дерева

- ✅ Учитывает произвольную вложенность
- ✅ Обрабатывает отсутствие `children`
- ✅ Можно реализовать как DFS, так и BFS

### 4. Корзина

- ✅ Использует `data-*` атрибуты
- ✅ Состояние корзины в JS-массиве
- ✅ Динамический рендер при каждом изменении
- ✅ Подсчёт количества и суммы

---
