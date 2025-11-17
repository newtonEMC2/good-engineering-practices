# Next.js 16 Rendering Flow (RSC + Client Components) --- Fully Explained

This document gives a **clear, detailed, and beginner‑friendly walkthrough** of what happens from the moment a user loads a page in **Next.js 16** until the page becomes interactive and how navigation works afterward.

It explains:

* How Server Components run
* What the RSC Payload is
* How HTML is generated
* What happens in the browser
* How React updates the UI during navigation

---

# 1. User Requests a URL

When a user visits a page e.g.:

```http
GET /products/123
```

This request goes to the Next.js server.
There is *no JavaScript involved yet*. It is a plain HTTP request.

**Diagram:**

```
User Browser
    |
    |  GET /products/123
    v
Next.js Server
```

---

# 2. Next.js Identifies the Component Tree

Next.js figures out which components belong to this route:

* **Server Components**
  Default type. No "use client" at the top.

* **Client Components**
  Must start with "use client". These require browser-side JavaScript.

Next.js builds the full layout hierarchy:

```
layout
 └─ nested layout
      └─ page
           └─ components
```

Each part may have both Server and Client Components.

**Code example:**

```jsx
// app/products/[id]/page.jsx
import ProductDetails from './ProductDetails'; // Server Component
import AddToCartButton from './AddToCartButton'; // Client Component

export default function ProductPage({ params }) {
  return (
    <div>
      <ProductDetails id={params.id} />
      <AddToCartButton productId={params.id} />
    </div>
  );
}
```

---

# 3. Server Components Execute on the Server

Next.js now **runs all Server Components** on the server.

This is where:

* Database queries happen
* `fetch()` calls run
* Environment variables can be used
* Business logic executes

**Key points:**

* No JavaScript bundles are emitted for these components
* Server Components **never run in the browser**
* Server Components **do not output HTML directly at this stage**; instead, they output a structured data format for the client (RSC Payload)

### Static vs Dynamic Server Components

Next.js 16 uses a **new cache system**, so “static” no longer always means “build-time.” Instead:

## ✔️ Three types of Server Component rendering in Next.js 16

---

## 1. **Build-time Static Rendering (rare)**

Only happens when ALL of these are true:

* route has **no dynamic segments**
* no dynamic fetches
* no cookies/headers
* no searchParams reading
* no server actions

Example:

```jsx
// /about (fully static)
export default function About() {
  return <div>About us</div>;
}
```

➡️ Next.js **pre-renders HTML + RSC Payload at build time**.
➡️ CDN can serve it directly.

---

## 2. **Runtime Static Rendering (MOST COMMON)**

This is what “static Server Component” usually means in the **App Router**.

* Happens for routes like `/products/[id]`
* Component has no dynamic flags
* Fetches are cached (`cache: 'force-cache'`) or default

BUT:
➡️ Next.js **cannot** pre-render at build time because IDs are unknown.
➡️ So it renders the page **on first request**, generates the RSC Payload + HTML, and stores that in the **Next.js Persistent Cache**.

Then:

* Future requests for the same path use the cached version
* CDN can cache *after* the first render

Example:

```jsx
// /products/[id]
export const revalidate = 3600; // allow ISR-style caching
```

Flow:

```
First request  -> render on server -> store in server cache
Later requests -> served from Next.js cache or CDN
```

---

## 3. **Dynamic Rendering (SSR)**

If the page uses any dynamic features:

* `export const dynamic = 'force-dynamic'`
* `fetch(..., { cache: 'no-store' })`
* reading cookies or headers
* searchParams affecting logic
* server actions

Then:
➡️ page renders **on every request**, not cached.
➡️ CDN cannot cache (unless forced manually).

Example:

```jsx
export const dynamic = 'force-dynamic';
```

---

## 🌟 How this affects HTML generation in Step 3

When Next.js begins rendering the page:

### **Where do static fragments come from?**

* **Build-time static:** from the build output (disk/memory)
* **Runtime static:** from the **Next.js server cache**, if already cached

  * If first time -> Next.js renders them and stores them

### **Where do dynamic fragments come from?**

* Rendered fresh on the server for every request

### **Important:**

👉 **Next.js server NEVER pulls static fragments from CDN when assembling HTML.**
It only reads from:

* build output
* its own runtime cache

CDN participates **after HTML is generated**, not before.

---

Diagram:

```
Client requests /products/123
       |
       v
Next.js Server
   |-- Build-time static? --> read from build output
   |-- Runtime static? ----> fetch from Next.js cache OR generate on first request
   |-- Dynamic? -----------> generate fresh
       |
       v
Merge fragments --> HTML + RSC Payload
       |
       v
Send HTML to client (CDN may cache after)
```

Diagram::**

```
Client requests /products/123
       |
       v
Next.js Server
   |-- Read static SSG fragments from disk/memory
   |-- Render dynamic SSR components
   v
Merge static + dynamic --> Full HTML + RSC Payload
       |
       v
Send to Browser (can be cached on CDN afterwards)
```

**Key clarification:**

* **Initial HTML build:** Server uses **local prebuilt static fragments**.
* **CDN caching:** Happens **after** the HTML/RSC Payload is generated, not as the source for building it.

---

# 4. React Generates the RSC Payload (React Server Component Payload)

The **RSC Payload** is a special serialized data format that describes:

* The structure of the server-rendered component tree
* All props passed into components
* Slots where Client Components should appear
* Serialized data returned from the server
* Streaming instructions for React

Think of it like:

> "Here is the entire UI, but encoded as data instead of HTML."

**Example of simplified RSC Payload (JSON-like):**

```json
{
  "type": "div",
  "children": [
    {
      "type": "h1",
      "props": { "children": "Product Name" }
    },
    {
      "type": "p",
      "props": { "children": "Product description..." }
    },
    {
      "type": "ClientComponent",
      "name": "AddToCartButton",
      "props": { "productId": "123" }
    }
  ]
}
```

**Diagram:**

```
Server Components
      |
      v
  [RSC Payload]
      |
      v
Next.js HTML Generator
```

---

# 5. Next.js Uses the RSC Payload to Generate HTML

Now, Next.js converts the RSC Payload into:

### ✔️ Pre-rendered HTML

```html
<div>
  <h1>Product Name</h1>
  <p>Product description...</p>
  <div id="AddToCartButton-123"></div>
</div>
```

### ✔️ Embedded or streamed RSC Payload

```html
<script type="application/json" id="__RSC_PAYLOAD__">
  { /* JSON payload from previous step */ }
</script>
```

### ✔️ A Client Component JS manifest

```json
{
  "AddToCartButton": "/_next/static/chunks/AddToCartButton.js"
}
```

Key point:

👉 **HTML is generated here, not earlier.**

---

# 6. Browser Receives Fully Rendered HTML

The user immediately sees the full page.

* Server Components render as real HTML
* Client Components render as placeholders (HTML looks correct, but not interactive yet)

**Diagram:**

```
Browser
  HTML Rendered:
  <h1>Product Name</h1>
  <p>Product description...</p>
  <div id="AddToCartButton-123"></div>
```

---

# 7. Browser Loads Only Client Component JavaScript

The browser loads JS **only** for Client Components:

```js
import AddToCartButton from "/_next/static/chunks/AddToCartButton.js";
```

**Result:**

* Buttons
* Form inputs
* Interactive widgets
* Animations

Server Components remain server-only.

---

# 8. React Hydrates Client Components Using the RSC Payload

Hydration process:

* Reuses the already-rendered HTML
* Attaches React logic and event listeners
* Makes Client Components interactive

**Code example (Client Component):**

```jsx
"use client";

export default function AddToCartButton({ productId }) {
  const handleClick = () => alert(`Added ${productId} to cart`);
  return <button onClick={handleClick}>Add to Cart</button>;
}
```

**Diagram:**

```
HTML Placeholder      RSC Payload
      |                  |
      v                  v
  React attaches events --> Interactive Client Components
```

Once hydration finishes:

👉 **The page becomes fully interactive.**

---

# 9. User Navigates to Another Page (Client-Side Navigation)

Next.js **does NOT reload the page** on `<Link />` clicks:

1. Browser requests **only the RSC Payload for new route**
2. Server Components run for the new page
3. A new RSC Payload streams back to the browser

**Diagram:**

```
User Clicks <Link>
       |
       v
   Browser requests RSC Payload
       |
       v
   Server generates new RSC Payload
       |
       v
   React updates UI selectively
```

---

# 10. React Updates Only the Parts of the UI That Changed

React diffs new RSC Payload with current UI:

* Updates only changed components
* Re-hydrates any new Client Components

**Example:**

```jsx
// Navigated to /products/124
{
  "type": "div",
  "children": [
    {
      "type": "h1",
      "props": { "children": "Another Product" }
    },
    {
      "type": "p",
      "props": { "children": "Different description..." }
    }
  ]
}
```

Only `<h1>` and `<p>` are updated; old AddToCartButton remains interactive.

---

# Summary --- Simple Ordered List

1. User requests page
2. Load component tree
3. Run Server Components (Static from disk/memory + Dynamic)
4. Generate RSC Payload
5. Pre-render HTML
6. Browser receives HTML
7. Download Client JS bundles
8. Hydrate Client Components
9. Fetch new RSC Payload on navigation
10. React updates UI

**Final diagram of full flow:**

```
User Request
      |
      v
  Next.js Server
      |
      v
  Identify Component Tree
      |
      v
  Run Server Components (Static/Dynamic)
      |
      v
  Generate RSC Payload
      |
      v
  Generate HTML + Embed RSC Payload
      |
      v
  Browser Receives HTML
      |
      v
  Load Client JS & Hydrate
      |
      v
  Page Fully Interactive
      |
      v
Client-side Navigation -> Fetch new RSC Payload -> Update UI
```
