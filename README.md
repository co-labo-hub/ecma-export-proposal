# Proposal for Limiting the Scope of `export`

In classes of languages like Java, appropriate encapsulation is achieved through the `protected` modifier in addition to `private` and `public`.
However, `export` in JavaScript currently offers only a binary choice: either export or not.

Therefore, if one wants to split code into multiple files, they are forced to `export`. This makes the exported members accessible from all other files, compelling all other files to understand and handle these exported members appropriately.
Conversely, not exporting leads to monolithic files that become excessively large.

Given that JavaScript commonly exists without classes, we propose an extension to the `export` specification rather than seeking `protected` support.

# Proposed Specification

Allow `export` to optionally accept an array of path patterns to define the scope of visibility.

```javascript
export['path1', 'path2', ...] const foo;
```

# Examples of the Proposed Specification

Allowing import from all files within the same directory:

```javascript
export['./*'] const foo;
```

Allowing import from all files within the parent directory:

```javascript
export['../*'] const foo;
```

Allowing import from all files within the current directory and its subdirectories:

```javascript
export['./**'] const foo;
```

Allowing import only from `bar.js` within the same directory:

```javascript
export['./bar.js'] const foo;
```

Combination:

```javascript
export['../baz.js', './*'] const foo;
```

# Expected Effects

This will enable proper encapsulation in JavaScript, even in cases where classes are not used.

Example:

````javascript
// Entry point to the library, needs encapsulation (should not be accessed from business logic)
app/features/counter/lib.js

// Counter logic implementation, uses lib.js
app/features/counter/counter.js

// Business logic implementation, uses counter.js
app/business.js

---

app/features/counter/lib.js

Import libraries only here.

```javascript
import storage from 'something-lib';

const count = storage.create('key');

// Export only within the same directory for encapsulation
export['./*'] const getCount = () => count.get();
export['./*'] count setCount = (newValue) => count.set(newValue);
````

---

app/features/counter/counter.js

Implements the counter functionality.

```javascript
import { getCount as get, setCount } from "./lib.js";

export const getCount = () => get();
export const addCount = () => setCount(get() + 1);
export const resetCount = () => setCount(0);
```

---

app/business.js

```javascript
import { getCount, addCount, resetCount } from "./features/counter";

// 以下はインポートできない、ビジネスロジックでは以下の仕様を把握する必要がない
// import { getCount, setCount } from './features/counter/lib';
```
