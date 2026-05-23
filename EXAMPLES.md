# Examples

Before/after for each TypeScript convention.

---

## 1. `strict: true`

**Before** (loose tsconfig):
```json
{
  "compilerOptions": {
    "strict": false,
    "noImplicitAny": false
  }
}
```

**After**:
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

If migrating an existing codebase, do it incrementally: enable each strict flag one at a time, fix the resulting errors, ship before enabling the next.

---

## 2. `any` → `unknown` + narrow

**Before**:
```ts
function processInput(data: any) {
  return data.value.toUpperCase();
}
```

**After**:
```ts
function processInput(data: unknown): string {
  if (typeof data !== 'object' || data === null) {
    throw new TypeError('Expected object');
  }
  if (!('value' in data) || typeof data.value !== 'string') {
    throw new TypeError('Expected data.value to be a string');
  }
  return data.value.toUpperCase();
}
```

Or with a schema library like Zod:

```ts
import { z } from 'zod';

const InputSchema = z.object({ value: z.string() });
type Input = z.infer<typeof InputSchema>;

function processInput(data: unknown): string {
  const parsed = InputSchema.parse(data);
  return parsed.value.toUpperCase();
}
```

---

## 3. Discriminated union over optional fields

**Before**:
```ts
type State = {
  status: 'idle' | 'loading' | 'success' | 'error';
  data?: User[];
  error?: string;
};

// Caller has to remember which fields are valid for which status
if (state.status === 'success') {
  state.data?.map(u => u.name); // optional even though it's always present in success
}
```

**After**:
```ts
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User[] }
  | { status: 'error'; error: string };

if (state.status === 'success') {
  state.data.map(u => u.name); // type-checker knows data is User[]
}
```

The type-checker now enforces that `data` is only accessed when status is `success`.

---

## 4. Result types over throw

**Before**:
```ts
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  return response.json();
}

// Caller has to remember to try/catch
const user = await fetchUser('123'); // may throw
```

**After**:
```ts
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

async function fetchUser(id: string): Promise<Result<User, string>> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    return { ok: false, error: `HTTP ${response.status}` };
  }
  return { ok: true, value: await response.json() };
}

// Caller must handle both cases at the type level
const result = await fetchUser('123');
if (result.ok) {
  console.log(result.value.name);
} else {
  console.error(result.error);
}
```

For unrecoverable errors (programmer errors, invariant violations), still throw.

---

## 5. `as const` over `enum`

**Before**:
```ts
enum Status {
  Pending = 'pending',
  Active = 'active',
  Closed = 'closed',
}

function update(s: Status) { ... }
update(Status.Active);
```

**After**:
```ts
const STATUSES = ['pending', 'active', 'closed'] as const;
type Status = (typeof STATUSES)[number];

function update(s: Status) { ... }
update('active'); // type-checked literal
```

Benefits:
- No runtime overhead (enums emit JS objects; `as const` is erased)
- Tree-shakeable
- The values ARE the literals — no `Status.Active` vs `'active'` mismatch

---

## 6. `satisfies` over `as`

**Before**:
```ts
const config = {
  port: 8080,
  host: 'localhost',
  protocol: 'http',
} as Config;

// `as` lies — if Config has more required fields, no error
// Also widens types: config.protocol is now string, not 'http'
```

**After**:
```ts
const config = {
  port: 8080,
  host: 'localhost',
  protocol: 'http',
} satisfies Config;

// `satisfies` checks the value against Config without widening
// config.protocol is still 'http' literal, type-checker still validates shape
```

---

## 7. `readonly` for immutability

**Before**:
```ts
function processItems(items: Item[]) {
  items.sort(); // mutates the caller's array
  return items;
}
```

**After**:
```ts
function processItems(items: readonly Item[]): Item[] {
  return [...items].sort();
}

// Or for prop types in React:
type Props = {
  readonly items: readonly Item[];
  readonly onClick: (item: Item) => void;
};
```

---

## 8. Brand types for IDs

**Before**:
```ts
function getOrder(userId: string, orderId: string) {
  // What stops you from accidentally swapping the args?
}

getOrder(orderId, userId); // type-checker happy, runtime broken
```

**After**:
```ts
type Brand<K, T> = K & { readonly __brand: T };
type UserId = Brand<string, 'UserId'>;
type OrderId = Brand<string, 'OrderId'>;

function getOrder(userId: UserId, orderId: OrderId) { ... }

getOrder(orderId, userId); // type error: argument of type 'OrderId' is not assignable to UserId
```

Worth the ceremony when mixing IDs is a real risk (orders, users, products, customers).

---

## 9. `// @ts-expect-error` over `// @ts-ignore`

**Before**:
```ts
// @ts-ignore — this is fine for now
const result = brokenFunction();
```

**After**:
```ts
// @ts-expect-error — TODO(2026-07): fix when brokenFunction has proper types
const result = brokenFunction();
```

`@ts-expect-error` errors when the underlying issue is fixed (so the comment can be removed). `@ts-ignore` stays forever.

---

## 10. Promise.all vs. Promise.allSettled

**Before**:
```ts
const [users, orders, products] = await Promise.all([
  fetchUsers(),
  fetchOrders(),
  fetchProducts(),
]);
// If any one fails, the others' work is wasted; only one error surfaces
```

**After** (when partial success is meaningful):
```ts
const [usersResult, ordersResult, productsResult] = await Promise.allSettled([
  fetchUsers(),
  fetchOrders(),
  fetchProducts(),
]);

if (usersResult.status === 'fulfilled') { /* use usersResult.value */ }
if (ordersResult.status === 'fulfilled') { /* use ordersResult.value */ }
if (productsResult.status === 'fulfilled') { /* use productsResult.value */ }
```

Use `Promise.all` when failure of one means the whole operation should fail. Use `Promise.allSettled` when each result is independent.

---

## 11. Never `} catch {}`

**Before**:
```ts
try {
  await riskyOp();
} catch {
  // silently swallow
}
```

**After**:
```ts
try {
  await riskyOp();
} catch (error) {
  logger.error('riskyOp failed', { error });
  // Either re-throw, or handle deliberately. Never silent.
}
```

---

## 12. `import type` for type-only imports

**Before**:
```ts
import { User, UserService } from './users';
// UserService is a runtime value; User is a type
// Bundler can't tree-shake User out
```

**After**:
```ts
import { UserService } from './users';
import type { User } from './users';
// Bundler knows User is type-only; tree-shakes it away
```

Or in a single statement:

```ts
import { UserService, type User } from './users';
```

---

## 13. `noUncheckedIndexedAccess`

**Before** (`noUncheckedIndexedAccess: false`):
```ts
const arr: string[] = ['a', 'b', 'c'];
const first = arr[0]; // type: string
first.toUpperCase(); // OK at type-check time, throws at runtime if arr is empty
```

**After** (`noUncheckedIndexedAccess: true`):
```ts
const arr: string[] = ['a', 'b', 'c'];
const first = arr[0]; // type: string | undefined
first.toUpperCase(); // type error: object is possibly 'undefined'

// Force-narrow:
if (first !== undefined) {
  first.toUpperCase();
}
```

Catches off-by-one + missing-key bugs at compile time.

---

## 14. `exactOptionalPropertyTypes`

**Before**:
```ts
type Config = { port?: number };

function configure(c: Config) { ... }

configure({ port: undefined }); // accepted; some code treats { port: undefined } same as no port
```

**After** (`exactOptionalPropertyTypes: true`):
```ts
type Config = { port?: number };

configure({ port: undefined }); // type error
configure({}); // OK
configure({ port: 8080 }); // OK
```

Distinguishes "property absent" from "property explicitly set to undefined." Catches the bug where API code copies optional fields and accidentally inserts `undefined` values.

---

## 15. Standard project setup

```bash
# Init
npm init -y
npm i -D typescript @types/node tsx vitest eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier

# Create tsconfig
cat > tsconfig.json <<'EOF'
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}
EOF
```

Then write code that adheres to this CLAUDE.md.
