## 1. `.map()`
Transforms each element in an array and returns a **new array**.

### Syntax:
```js
const newArray = array.map((element, index, array) => {
  return modifiedElement;
});
```

### Example:
```js
const nums = [1, 2, 3];
const doubled = nums.map(num => num * 2);
console.log(doubled); // [2, 4, 6]
```

---

## 2. `.reduce()`
Reduces the array to a **single value** using an accumulator.

### Syntax:
```js
const result = array.reduce((accumulator, currentValue, index, array) => {
  return updatedAccumulator;
}, initialValue);
```

### Example:
```js
const nums = [1, 2, 3, 4];
const sum = nums.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 10
```

---

## 3. `.filter()`
Returns a new array with elements that **pass the condition** (truthy values).

### Syntax:
```js
const filtered = array.filter((element, index, array) => {
  return condition;
});
```

### Example:
```js
const nums = [1, 2, 3, 4, 5];
const even = nums.filter(num => num % 2 === 0);
console.log(even); // [2, 4]
```

---

## 4. `.forEach()`
Runs a function **for each element**. It does **not** return anything (undefined).

### Syntax:
```js
array.forEach((element, index, array) => {
  // do something
});
```

### Example:
```js
const names = ['Alice', 'Bob', 'Carol'];
names.forEach(name => console.log(name));
// Logs each name
```

---

## 5. `.flat()`
Flattens **nested arrays** up to the specified depth.

### Syntax:
```js
const flattened = array.flat(depth);
```

### Example:
```js
const arr = [1, [2, [3, 4]]];
console.log(arr.flat());        // [1, 2, [3, 4]]
console.log(arr.flat(2));       // [1, 2, 3, 4]
```

---

## 6. `.find()`
Returns the **first element** that matches the condition.

### Syntax:
```js
const found = array.find((element, index, array) => {
  return condition;
});
```

### Example:
```js
const users = [{id: 1}, {id: 2}, {id: 3}];
const user = users.find(u => u.id === 2);
console.log(user); // {id: 2}
```

---

## 7. `.some()` & `.every()`

### `.some()`: Returns `true` if **at least one** element passes the test.
```js
[1, 2, 3].some(x => x > 2); // true
```

### `.every()`: Returns `true` if **all elements** pass the test.
```js
[1, 2, 3].every(x => x > 0); // true
```

---

## 8. `.findIndex()`
Like `.find()`, but returns the **index** of the matching element (or -1 if not found).
```js
const index = [5, 12, 8].findIndex(num => num > 10);
console.log(index); // 1
```

---

## 9. `.includes()`
Checks if the array **includes** a specific value (uses strict equality).
```js
[1, 2, 3].includes(2); // true
```

---

## 10. `.sort()` and `.reverse()`

### `.sort()`:
Sorts array **in place** (modifies original array). Use a comparator for custom logic.
```js
[3, 1, 4].sort((a, b) => a - b); // [1, 3, 4]
```

### `.reverse()`:
Reverses array **in place**.
```js
[1, 2, 3].reverse(); // [3, 2, 1]
```

---

## 11. `.flatMap()`
Like `.map()` followed by `.flat(1)` – useful for returning and flattening arrays from each map call.

### Example:
```js
const arr = [1, 2, 3];
const result = arr.flatMap(n => [n, n * 2]);
console.log(result); // [1, 2, 2, 4, 3, 6]
```

---

## Quick Reference Summary

| Method       | Returns        | Use Case                                  |
|--------------|----------------|--------------------------------------------|
| `.map()`     | New array       | Transform each element                    |
| `.reduce()`  | Single value    | Aggregate (sum, product, etc.)            |
| `.filter()`  | New array       | Filter elements by condition              |
| `.forEach()` | undefined       | Execute side effects                      |
| `.flat()`    | New array       | Flatten nested arrays                     |
| `.flatMap()` | New array       | Map + flatten                             |
| `.find()`    | First match     | Get one element by condition              |
| `.findIndex()`| Index of match | Get index of first match                  |
| `.some()`    | Boolean         | Check if *any* match a condition          |
| `.every()`   | Boolean         | Check if *all* match a condition          |
| `.includes()`| Boolean         | Check for value existence                 |
| `.sort()`    | Same array      | Sort elements                             |
| `.reverse()` | Same array      | Reverse array order                       |

---
