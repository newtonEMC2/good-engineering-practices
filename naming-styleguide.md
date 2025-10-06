# 🧭 TypeScript Naming & Function Style Guide

A concise guide on writing **meaningful functions and variables** in TypeScript,  
focused on clarity, consistency, and intent-driven naming.

---

## 1. 🎯 Core Principles

| Principle | Description | Example |
|------------|--------------|----------|
| **Clarity over brevity** | Prefer descriptive names over short ones. | ✅ `calculateDiscount()`<br>❌ `calc()` |
| **Consistency matters** | Use the same naming patterns across your codebase. | ✅ `getX`, `setX`, `updateX` |
| **Intent first** | Names should express *why* something exists, not *what type* it is. | ✅ `userHasAccess` |
| **Avoid encoding types** | Don’t include types in names — TS already knows them. | ✅ `users: User[]`<br>❌ `userArray: User[]` |
| **Be pronounceable** | Names should read naturally in conversation. | ✅ `totalRevenue` |

---

## 2. 🏷️ Variable Naming

### ✅ Rules
- Use **camelCase** for variables and functions.
- Use **PascalCase** for types, interfaces, classes, and enums.
- Prefix **boolean** variables with `is`, `has`, `can`, or `should`.
- Use **plural nouns** for arrays or collections.

### 💡 Examples
```typescript
const userName = "Alice";
const users: User[] = [];
const isUserLoggedIn = true;
const maxRetryCount = 3;
```

### ❌ Avoid
```typescript
const strUserName = "Alice"; // redundant type info
const user_list = []; // wrong case
const flag = true; // meaningless
```

---

## 3. ⚙️ Function Naming

### ✅ Use Verbs
Functions **do something**, so start with a verb that expresses an action.

| Verb | Typical Use |
|------|--------------|
| `get` | Retrieve a value |
| `set` | Assign a value |
| `fetch` | Retrieve remote data |
| `create` | Construct or instantiate |
| `update` | Modify existing data |
| `delete` / `remove` | Eliminate something |
| `validate` | Check correctness |
| `calculate` | Compute a value |
| `handle` | Respond to an event |
| `convert` / `transform` | Change format or structure |

### 💡 Examples
```typescript
function getUserById(id: string): User | null { ... }
function calculateTotal(items: Item[]): number { ... }
function handleFormSubmit(event: SubmitEvent): void { ... }
function transformUserResponse(raw: any): User { ... }
```

---

## 4. 🧠 Boolean Functions and Variables

### ✅ Naming Convention

| Type | Represents | Example | Reads as |
|------|-------------|----------|----------|
| **Boolean variable** | A *state* or *fact* | `isLoggedIn`, `hasPermission` | “The user **is logged in**.” |
| **Boolean function** | A *question* or *check* | `isUserLoggedIn()`, `hasAccess()` | “Is the user logged in?” |

**Difference:** only `()` — but semantically:
- Variable → known fact  
- Function → computed fact

### 💡 Examples
```typescript
const isLoggedIn = true; // known state
function isUserLoggedIn(user: User): boolean {
  return !!user.sessionToken;
}
```

✅ Usage:
```typescript
if (isLoggedIn) showDashboard();
if (isUserLoggedIn(currentUser)) showDashboard();
```

---

## 5. 🧩 When to Use `check`

### ✅ Use `check` when:
- The function **enforces** a condition.
- It **throws**, **logs**, or **asserts** something.
- It’s an **imperative action**, not a question.

### ❌ Don’t use `check` when:
- The function simply **returns a boolean**.
- The function **doesn’t have side effects**.

### 💡 Examples

```typescript
// ✅ Enforcement — throws if condition not met
function checkUserAuthenticated(user: User): void {
  if (!user.sessionToken) throw new Error("User not authenticated");
}

// ✅ Passive boolean check
function isUserAuthenticated(user: User): boolean {
  return !!user.sessionToken;
}
```

### 🧠 Rule of Thumb

| Use Case | Verb | Return Type | Example |
|-----------|------|--------------|----------|
| Ask a question | `is`, `has`, `can`, `should` | `boolean` | `isEmailValid()` |
| Enforce correctness | `check`, `assert`, `ensure` | `void` / throws | `checkEmailValid()` |
| Validate and return boolean | `validate` | `boolean` | `validateEmail()` |

### 💡 Combined Example

```typescript
function isPasswordStrong(password: string): boolean {
  return password.length >= 8 && /\d/.test(password);
}

function checkPasswordStrength(password: string): void {
  if (!isPasswordStrong(password)) {
    throw new Error("Password is too weak");
  }
}
```

---

## 6. 🧾 Type & Interface Naming

- Use **PascalCase**.
- Prefer **nouns** (describe data, not behavior).
- Prefix with `I` only if your team standardizes it.

```typescript
interface UserProfile {
  id: string;
  name: string;
  email: string;
}

type ApiResponse<T> = {
  data: T;
  status: number;
};
```

---

## 7. ⚠️ Common Pitfalls

| Pitfall | Example | Why It's Bad |
|----------|----------|--------------|
| Generic names | `data`, `info`, `stuff` | Don’t convey meaning |
| Abbreviations | `usr`, `cfg`, `tmp` | Harder to read/search |
| Context redundancy | `user.userName` | Repeats context |
| Negated booleans | `isNotValid` | Confusing logic (use `isValid`) |

---

## 8. 🧰 Example in Context

```typescript
async function fetchUserProfileAsync(userId: string): Promise<UserProfile> {
  const response = await fetch(`/api/users/${userId}`);
  const data = await response.json();
  return transformUserResponse(data);
}

function transformUserResponse(raw: any): UserProfile {
  return {
    id: raw.id,
    name: raw.full_name,
    email: raw.contact.email,
  };
}

function calculateDiscountedPrice(price: number, rate: number): number {
  return price - price * rate;
}

function checkUserAuthorized(user: User): void {
  if (!user.roles.includes("admin")) {
    throw new Error("User not authorized");
  }
}
```

---

## 9. 🧱 ESLint Naming Rules

To enforce naming patterns automatically:

```json
{
  "rules": {
    "camelcase": "error",
    "@typescript-eslint/naming-convention": [
      "error",
      { "selector": "variable", "format": ["camelCase", "UPPER_CASE"] },
      { "selector": "typeLike", "format": ["PascalCase"] },
      { "selector": "function", "format": ["camelCase"] }
    ]
  }
}
```

---

## 🔍 10. Function Return Type Discipline

### ✅ Always explicitly type public or exported functions
Relying on inference for local helpers is fine, but **public functions** should **declare** their return type.  
This prevents accidental API changes and improves editor hints.

```typescript
// ✅ Explicit for exported/public
export function getUserProfile(id: string): Promise<UserProfile> { ... }

// ✅ Implicit is fine for small internal helpers
const toSlug = (s: string) => s.toLowerCase().replace(/\s+/g, "-");
```

---

## 🧱 11. Module & File Naming

### ✅ Rules
- File names are **kebab-case** (not camelCase or PascalCase).
- The file name should reflect the **main export**.
- Use **index.ts** only for module entrypoints.

| Type | Example |
|------|----------|
| Function | `calculate-discount.ts` |
| Class | `user-service.ts` |
| Component | `UserCard.tsx` |
| Barrel (entrypoint) | `index.ts` |

---

## 🧠 12. Parameter Naming & Ordering

### ✅ Principles
1. Keep the **most important argument first**.  
2. Use **contextual names** (avoid `a`, `b`, `x`, `y` unless truly generic).  
3. For booleans in parameters — **avoid**! Use options objects instead.

❌ Bad:
```typescript
function sendEmail(to: string, urgent: boolean) { ... }
sendEmail("alice@example.com", true);
```

✅ Better:
```typescript
function sendEmail(to: string, options: { urgent?: boolean } = {}) { ... }
sendEmail("alice@example.com", { urgent: true });
```

---

## 🧩 13. Constants & Enums

### ✅ Rules
- Constants: `UPPER_CASE_SNAKE_CASE`
- Enums: **PascalCase** with **noun-like** names.

```typescript
const MAX_RETRY_COUNT = 5;

enum UserRole {
  Admin = "admin",
  Editor = "editor",
  Viewer = "viewer",
}
```

---

## 🧾 14. Doc Comments & Auto-Hints

Use **TSDoc** style comments for functions that are exported or reused.

```typescript
/**
 * Checks if the current user has permission for a given action.
 * @param user - The user object to check.
 * @param action - The action being requested.
 * @returns True if the user has the required permission.
 */
function hasPermission(user: User, action: string): boolean {
  return user.permissions.includes(action);
}
```

> This improves IntelliSense and auto-generates documentation if needed.

---

## ✅ Quick Summary

| Purpose | Pattern | Example |
|----------|----------|----------|
| **Boolean variable (state)** | adjective-like | `isReady`, `hasError` |
| **Boolean function (question)** | same prefixes + `()` | `isReadyToSubmit()` |
| **Validation (boolean)** | `validate` | `validateEmail()` |
| **Assertion / enforcement** | `check`, `assert`, `ensure` | `checkEmailValid()` |
| **Action** | verb phrase | `fetchUserData()` |
| **Type / Interface** | PascalCase noun | `UserProfile`, `ApiResponse<T>` |

---

**Rule of thumb:**
> - Variables are **answers**.  
> - Functions are **questions or actions**.  
> - `check` means **enforce it**, not **ask about it**.

---
