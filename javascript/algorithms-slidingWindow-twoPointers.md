# Hash Table, Sliding Window, Two Pointers

## 1. Скользящее окно (Sliding Window)

### 1.1 Что такое техника скользящего окна и когда она применяется?

```js
// Скользящее окно — техника для работы с подмассивом/подстрокой,
// где мы двигаем границы окна, вместо того чтобы пересчитывать всё заново.

// Применяется, когда есть массив или строка и нужно найти:
// - максимальную/минимальную сумму подмассива
// - подстроку с определёнными условиями
// - количество подходящих подмассивов

// Пример: найти максимальную сумму подмассива длины 3
// Вместо того чтобы перебирать все подмассивы (O(n*k)),
// мы скользим окном (O(n))
```

### 1.2 Фиксированное vs переменное окно

```js
// ========== ФИКСИРОВАННОЕ ОКНО ==========
// Размер окна не меняется (например, длина = k)

function fixedWindow(arr, k) {
  let sum = 0;
  // Считаем сумму первого окна
  for (let i = 0; i < k; i++) {
    sum += arr[i];
  }
  let maxSum = sum;

  // Скользим окном
  for (let i = k; i < arr.length; i++) {
    // Убираем левый элемент, добавляем правый
    sum = sum - arr[i - k] + arr[i];
    maxSum = Math.max(maxSum, sum);
  }
  return maxSum;
}

// ========== ПЕРЕМЕННОЕ ОКНО ==========
// Размер окна меняется в зависимости от условия

function variableWindow(arr, target) {
  let left = 0;
  let sum = 0;
  let minLength = Infinity;

  for (let right = 0; right < arr.length; right++) {
    sum += arr[right];

    // Сужаем окно, пока условие выполняется
    while (sum >= target) {
      minLength = Math.min(minLength, right - left + 1);
      sum -= arr[left];
      left++;
    }
  }
  return minLength;
}
```

---

## 2. Hash таблицы (Hash Tables)

### 2.1 Что такое hash таблица и как она работает?

```js
// Hash таблица — структура данных вида: ключ → значение

// Как работает:
// 1. Хеш-функция превращает ключ в число (индекс)
// 2. Значение сохраняется по этому индексу
// 3. При поиске — ключ снова хешируется и значение находится мгновенно

// В JS:
const obj = { name: "Alex", age: 25 }; // Объект
const map = new Map(); // Map
map.set("name", "Alex");
map.set(123, "числовой ключ");
map.set({ id: 1 }, "объект как ключ");
```

### 2.2 Временная сложность операций

```js
// В среднем случае:
// Поиск:    O(1)
// Вставка:  O(1)
// Удаление: O(1)

// В худшем случае (коллизии): O(n)
// Коллизия — когда разные ключи дают одинаковый хеш
```

### 2.3 Object vs Map в JavaScript

```js
// ========== Object ({}) ==========
const obj = {};

// Особенности:
// - Ключи — только строки или Symbol
// - Не гарантирует порядок ключей
// - Имеет прототипное наследование
// - Размер нужно вычислять (Object.keys().length)

obj.name = "Alex";
obj[123] = "число"; // 123 станет строкой "123"
obj[{ a: 1 }] = "объект"; // объект станет "[object Object]"

console.log(obj); // { "123": "число", name: "Alex", "[object Object]": "объект" }

// ========== Map ==========
const map = new Map();

// Особенности:
// - Ключи любого типа (объекты, функции, числа)
// - Сохраняет порядок вставки
// - Нет прототипа
// - Есть свойство .size

map.set("name", "Alex");
map.set(123, "число");
map.set({ a: 1 }, "объект как ключ");

console.log(map.size); // 3
console.log(map.get(123)); // "число"

// ========== Что лучше использовать? ==========
// Object: для простых словарей, JSON, когда ключи — строки
// Map: для частых операций set/get, когда ключи не строки, нужен порядок
```

---

## 3. Техника двух указателей (Two Pointers)

### 3.1 Что такое техника двух указателей?

```js
// Техника, где используются два указателя (обычно left и right)
// Позволяет снизить сложность с O(n²) до O(n)

// Применяется для:
// - Отсортированных массивов
// - Поиска пар с заданной суммой
// - Проверки палиндромов
// - Удаления дубликатов
```

### 3.2 Противоположно направленные vs Однонаправленные указатели

```js
// ========== ПРОТИВОПОЛОЖНО НАПРАВЛЕННЫЕ ==========
// Двигаются навстречу друг другу
// left = 0, right = arr.length - 1

function twoSumSorted(numbers, target) {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const sum = numbers[left] + numbers[right];

    if (sum === target) {
      return [left, right];
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  return [];
}

// ========== ОДНОНАПРАВЛЕННЫЕ ==========
// Двигаются в одну сторону (right бежит вперёд)
// left меняется по условию

function removeDuplicates(nums) {
  if (nums.length === 0) return 0;

  let left = 0;
  for (let right = 1; right < nums.length; right++) {
    if (nums[left] !== nums[right]) {
      left++;
      nums[left] = nums[right];
    }
  }
  return left + 1;
}
```

---

## Практические задачи с решениями

### Задача 1. Максимальная сумма подмассива размера k

```js
// Фиксированное скользящее окно
// Сложность: O(n)

function maxSumSubarray(arr, k) {
  // Считаем сумму первого окна
  let sum = 0;
  for (let i = 0; i < k; i++) {
    sum += arr[i];
  }

  let maxSum = sum;

  // Скользим окном
  for (let i = k; i < arr.length; i++) {
    sum = sum - arr[i - k] + arr[i];
    maxSum = Math.max(maxSum, sum);
  }

  return maxSum;
}

// Проверка:
console.log(maxSumSubarray([2, 1, 5, 1, 3, 2], 3)); // 9
console.log(maxSumSubarray([1, 4, 2, 10, 2, 3, 1, 0, 20], 4)); // 24
console.log(maxSumSubarray([100, 200, 300, 400], 2)); // 700
```

### Задача 2. Два числа с заданной суммой (Two Pointers)

```js
// Противоположно направленные указатели
// Сложность: O(n)
// Требование: массив отсортирован

function twoSum(numbers, target) {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const sum = numbers[left] + numbers[right];

    if (sum === target) {
      return [left, right];
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  return []; // Если не найдено
}

// Проверка:
console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
console.log(twoSum([2, 3, 4], 6)); // [0, 2]
console.log(twoSum([-1, 0], -1)); // [0, 1]
```

### Задача 3. Самая длинная подстрока без повторяющихся символов

```js
// Переменное скользящее окно + HashSet
// Сложность: O(n)

function lengthOfLongestSubstring(s) {
  const set = new Set();
  let left = 0;
  let maxLength = 0;

  for (let right = 0; right < s.length; right++) {
    // Если символ уже есть в окне — сужаем окно
    while (set.has(s[right])) {
      set.delete(s[left]);
      left++;
    }
    set.add(s[right]);
    maxLength = Math.max(maxLength, right - left + 1);
  }

  return maxLength;
}

// Проверка:
console.log(lengthOfLongestSubstring("abcabcbb")); // 3 ("abc")
console.log(lengthOfLongestSubstring("bbbbb")); // 1 ("b")
console.log(lengthOfLongestSubstring("pwwkew")); // 3 ("wke")
console.log(lengthOfLongestSubstring("")); // 0
```

### Задача 4. Группировка анаграмм (Hash Table)

```js
// Используем Map для группировки
// Сложность: O(n * m log m), где m — длина слова

function groupAnagrams(strings) {
  const map = new Map();

  for (const word of strings) {
    // Ключ — отсортированная версия слова
    const key = word.split("").sort().join("");

    if (!map.has(key)) {
      map.set(key, []);
    }
    map.get(key).push(word);
  }

  return Array.from(map.values());
}

// Проверка:
console.log(groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]));
// [["bat"], ["nat","tan"], ["ate","eat","tea"]]
console.log(groupAnagrams([""])); // [[""]]
console.log(groupAnagrams(["a"])); // [["a"]]
console.log(groupAnagrams(["abc", "bca", "cab", "xyz"]));
// [["abc","bca","cab"], ["xyz"]]
```

### Задача 5. Поиск пика в массиве (Бинарный поиск / Two Pointers)

```js
// Бинарный поиск (модифицированный)
// Сложность: O(log n)

function findPeakElement(nums) {
  let left = 0;
  let right = nums.length - 1;

  while (left < right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] < nums[mid + 1]) {
      // Пик справа
      left = mid + 1;
    } else {
      // Пик слева или на mid
      right = mid;
    }
  }

  return left;
}

// Проверка:
console.log(findPeakElement([1, 2, 3, 1])); // 2 (элемент 3)
console.log(findPeakElement([1, 2, 1, 3, 5, 6, 4])); // 1 или 5
console.log(findPeakElement([1])); // 0
console.log(findPeakElement([1, 2])); // 1
```

---

## Дополнительные задачи для практики

### Задача 6. Максимальное количество гласных в подстроке длины k

```js
// Фиксированное скользящее окно

function maxVowels(s, k) {
  const vowels = new Set(["a", "e", "i", "o", "u"]);
  let count = 0;

  // Первое окно
  for (let i = 0; i < k; i++) {
    if (vowels.has(s[i])) count++;
  }

  let maxCount = count;

  // Скользим окном
  for (let i = k; i < s.length; i++) {
    if (vowels.has(s[i - k])) count--;
    if (vowels.has(s[i])) count++;
    maxCount = Math.max(maxCount, count);
  }

  return maxCount;
}

console.log(maxVowels("abciiidef", 3)); // 3
console.log(maxVowels("aeiou", 2)); // 2
```

### Задача 7. Контейнер с наибольшим количеством воды

```js
// Два указателя (противоположно направленные)

function maxArea(height) {
  let left = 0;
  let right = height.length - 1;
  let maxArea = 0;

  while (left < right) {
    const h = Math.min(height[left], height[right]);
    const w = right - left;
    const area = h * w;
    maxArea = Math.max(maxArea, area);

    // Двигаем указатель с меньшей высотой
    if (height[left] < height[right]) {
      left++;
    } else {
      right--;
    }
  }

  return maxArea;
}

console.log(maxArea([1, 8, 6, 2, 5, 4, 8, 3, 7])); // 49
```

### Задача 8. Минимальная длина подмассива с суммой ≥ target

```js
// Переменное скользящее окно

function minSubArrayLen(target, nums) {
  let left = 0;
  let sum = 0;
  let minLength = Infinity;

  for (let right = 0; right < nums.length; right++) {
    sum += nums[right];

    while (sum >= target) {
      minLength = Math.min(minLength, right - left + 1);
      sum -= nums[left];
      left++;
    }
  }

  return minLength === Infinity ? 0 : minLength;
}

console.log(minSubArrayLen(7, [2, 3, 1, 2, 4, 3])); // 2
console.log(minSubArrayLen(7, [2, 3, 1, 2, 3, 1, 1])); // 4
```

### Задача 9. Первый уникальный символ в строке

```js
// Hash Table

function firstUniqChar(s) {
  const map = new Map();

  // Подсчитываем частоту каждого символа
  for (const char of s) {
    map.set(char, (map.get(char) || 0) + 1);
  }

  // Ищем первый с частотой 1
  for (let i = 0; i < s.length; i++) {
    if (map.get(s[i]) === 1) {
      return i;
    }
  }

  return -1;
}

console.log(firstUniqChar("leetcode")); // 0 ('l')
console.log(firstUniqChar("leetcodel")); // 3 ('t')
```

---

## Шпаргалка по техникам

| Техника                             | Сложность      | Когда использовать                         | Типичные задачи                               |
| ----------------------------------- | -------------- | ------------------------------------------ | --------------------------------------------- |
| **Sliding Window (фиксированное)**  | O(n)           | Размер окна известен (k)                   | max сумма подмассива, среднее значение        |
| **Sliding Window (переменное)**     | O(n)           | Нужно найти подмассив/подстроку с условием | min длина с суммой ≥ target, max без повторов |
| **Two Pointers (навстречу)**        | O(n)           | Отсортированный массив, поиск пары         | two sum, палиндромы                           |
| **Two Pointers (однонаправленные)** | O(n)           | Удаление дубликатов, разделение массива    | remove duplicates, partition                  |
| **Hash Table**                      | O(1) в среднем | Быстрый поиск/вставка                      | group anagrams, unique chars                  |

---

## Шпаргалка по сложности

| Операция | Hash Table (Object/Map) | Array (поиск)  |
| -------- | ----------------------- | -------------- |
| Поиск    | O(1)                    | O(n)           |
| Вставка  | O(1)                    | O(1) (в конец) |
| Удаление | O(1)                    | O(n)           |

---
