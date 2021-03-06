# Filter Json

Functions related to object comparing.

### Optional Initialization:

```js
const { configure } = require("@lefcott/filter-json");

// Define custom prefix for operators, default to '$'.
configure({ prefix: "$" });
```

### Checking an object with a condition:

```js
const { checkJsonCondition } = require("@lefcott/filter-json");

const ObjToCheck = {
  a: {
    b: {
      c: "someString",
      d: [1, 2, 3, 4],
      e: true,
      f: null,
      g: undefined,
    },
  },
  h: "someString",
  i: { j: 1234 },
};

// Some examples
let Condition;

Condition = { "a.b.c.g": { $exists: true } }; // Checks that a.b.c.g exists
checkJsonCondition(ObjToCheck, Condition); // returns false

Condition = { "a.b.c": { $like: "/^someS/i" } }; // Checks that a.b.c meets regex condition
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { "a.b.c": { $like: "/^somes/" } }; // Checks that a.b.c meets regex condition
checkJsonCondition(ObjToCheck, Condition); // returns false

Condition = { "a.b.c": { $like: /^someS/i } }; // Checks that a.b.c meets regex condition
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { "a.b.c": { $like: /^somes/ } }; // Checks that a.b.c meets regex condition
checkJsonCondition(ObjToCheck, Condition); // returns false

Condition = { h: { $field: "a.b.c" } }; // Checks that h equals a.b.c
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { "a.b.e": true }; // Checks a.b.e equals true
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { i: { j: 1234 } }; // Checks i equals json: { j: 1234 }
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { "a.b.d.length": 4 }; // Checks that length of array a.b.d equals 4
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { "a.b.d.3": 4 }; // Checks a.b.d[3] equals 4
checkJsonCondition(ObjToCheck, Condition); // returns true

Condition = { "a.b.d.0": { $transform: (d) => d + 2, $field: "a.b.d.2" } }; // Sums 2 to a.b.d[0] and checks if result is equal to a.b.d[2]
checkJsonCondition(ObjToCheck, Condition); // returns true
```

### Checking whether a value meets a condition:

```js
const { checkJsonCondition } = require("@lefcott/filter-json");

checkJsonCondition(5, { $: { $gt: 2 } }); // returns true
checkJsonCondition(7, { $: { $gte: 7, $lt: 3 } }); // returns false
```

### Checking a condition over all elements in an array:

```js
const { checkJsonCondition } = require("@lefcott/filter-json");

checkJsonCondition(
  { arr: [{ a: 1 }, { a: 2 }, { a: 3 }, { a: 3.5 }] },
  { arr: { $arrayAnd: { a: { $gte: 1, $lte: 3 } } } }
); // returns false (not all elements in the array meet the condition)

checkJsonCondition(
  { arr: [{ a: 1 }, { a: 2 }, { a: 3 }, { a: 3.5 }] },
  { arr: { $arrayOr: { a: { $gte: 1, $lte: 3 } } } }
); // returns true (at least one element in the array meets the condition)
```

### Filtering an array of objects with a condition:

```js
const { filter } = require("@lefcott/filter-json");

const OriginalObjects = [
  { a: 123 },
  { a: { b: 1 } },
  { a: { b: 2 } },
  { a: { b: 3 } },
  { a: { b: 4 } },
  { a: { b: 5 } },
];

filter(OriginalObjects, { "a.b": { $gte: 3 } });
// Returns [{ a: { b: 3 } }, { a: { b: 4 } }, { a: { b: 5 } }]
```

### Agreggate with min and max:

```js
const { filter } = require("@lefcott/filter-json");

filter([{ a: 1 }, { a: 3 }, { a: 2 }], {
  _aggregate: { field: "a", type: "min" },
}); // Returns [{ a: 1 }]

filter([{ a: 1 }, { a: 3 }, { a: 2 }], {
  _aggregate: { field: "a", type: "max" },
}); // Returns [{ a: 3 }]
```

### Checking if all objects of an array are equal

```js
const { compareN } = require("@lefcott/filter-json");

const ObjectList = [
  { a: { b: 1, c: 2 } },
  { a: { b: 1, c: 3 } },
  { a: { b: 1, c: 4, d: null } },
];
const FieldsToIgnore = ["a.c", "a.d"];

// Calling compareN without fields to ignore
compareN(ObjectList); // returns false

// Calling compareN with a list of fields to ignore
compareN(ObjectList, FieldsToIgnore); // returns true
```

### Parsing an invalid JSON (It may contain single quotes or no quotes in keys):

```js
const { parse } = require("@lefcott/filter-json");

parse("{ test: 1 }"); // Returns { test: 1 }
parse("{ 'hello': 'word' }"); // Returns { hello: 'word' }
```
