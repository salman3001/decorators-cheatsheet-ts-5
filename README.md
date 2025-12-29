# decorators-cheatsheet-ts-5

# 🧠 TypeScript Modern Decorators — Cheat Sheet (2025)

> ✅ Works with **TypeScript 5+**
>
> ❌ Does **NOT** use `experimentalDecorators`
> ❌ Does **NOT** use `reflect-metadata`
>
> Uses **TC39 Stage-3 decorators**

---

## ⚙️ Required `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "useDefineForClassFields": true
  }
}
```

---

# 📌 1. Class Decorator

### Purpose

Used to attach metadata or modify a class.

### Signature

```ts
(value: Function, context: ClassDecoratorContext) => void
```

### Example – Controller Metadata

```ts
const controllerMeta = new WeakMap<Function, string>();

function Controller(path: string) {
  return (value: Function, context: ClassDecoratorContext) => {
    controllerMeta.set(value, path);
  };
}
```

### Usage

```ts
@Controller("/users")
class UserController {}
```

### Accessing metadata

```ts
controllerMeta.get(UserController); // "/users"
```

---

# 🧩 2. Method Decorator

### Purpose

Intercept or modify method behavior.

### Signature

```ts
(
  value: Function,
  context: ClassMethodDecoratorContext
) => Function | void
```

### Example – Logging decorator

```ts
function Log() {
  return function (
    originalMethod: Function,
    context: ClassMethodDecoratorContext
  ) {
    return function (...args: any[]) {
      console.log(`Calling ${String(context.name)}`, args);
      return originalMethod.apply(this, args);
    };
  };
}
```

### Usage

```ts
class UserService {
  @Log()
  getUser(id: number) {
    return { id };
  }
}
```

---

# 🧩 3. Property Decorator

### Purpose

Attach metadata to class fields.

### Signature

```ts
(value: undefined, context: ClassFieldDecoratorContext)
```

### Example – Field metadata

```ts
const fieldMeta = new WeakMap<object, Map<string | symbol, string>>();

function Required() {
  return function (_: undefined, context: ClassFieldDecoratorContext) {
    return function (initialValue: any) {
      let map = fieldMeta.get(this);
      if (!map) {
        map = new Map();
        fieldMeta.set(this, map);
      }
      map.set(context.name, "required");
      return initialValue;
    };
  };
}
```

### Usage

```ts
class User {
  @Required()
  name!: string;
}
```

---

# 🧩 4. Accessor Decorator (getter/setter)

### Purpose

Wrap or validate access to properties.

```ts
function Readonly() {
  return function (
    value: { get?: Function; set?: Function },
    context: ClassAccessorDecoratorContext
  ) {
    return {
      get: value.get,
      set() {
        throw new Error(`Cannot modify ${String(context.name)}`);
      }
    };
  };
}
```

### Usage

```ts
class Config {
  @Readonly()
  accessor version = "1.0";
}
```

---

# 🧩 5. Parameter Decorator (Limited Use)

⚠️ Parameter decorators **cannot modify runtime behavior**, only metadata.

```ts
const paramMeta = new WeakMap<Function, number[]>();

function Inject() {
  return (value: undefined, context: ClassParameterDecoratorContext) => {
    const list = paramMeta.get(context.target) ?? [];
    list.push(context.index);
    paramMeta.set(context.target, list);
  };
}
```

```ts
class Service {
  constructor(@Inject() dep: any) {}
}
```

---

# 🧠 Recommended Metadata Pattern (Reusable)

```ts
function createMetadata<T>() {
  const store = new WeakMap<object, T>();

  return {
    set(target: object, value: T) {
      store.set(target, value);
    },
    get(target: object): T | undefined {
      return store.get(target);
    }
  };
}
```

Used like:

```ts
const RouteMeta = createMetadata<string>();

function Route(path: string) {
  return (target: Function) => RouteMeta.set(target, path);
}
```

---

