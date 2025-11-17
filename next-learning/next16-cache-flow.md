# Next.js 16 Post App with Cache Examples

This document demonstrates all caching scenarios in Next.js 16.0.3 using the new `cache` API, including partial UI caching and full SSG pages.

---

## **1. Cached Utility Functions**

```ts
// lib/cache.ts
import { cache } from 'react';

// Global cached function: never recomputes
export const getServerStartTime = cache(() => new Date().toISOString());

// Argument-based cache: caches per postId
export const fetchPost = cache(async (postId: string) => {
  const res = await fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`);
  return res.json();
});

// Cached list of posts with revalidation
export const fetchPosts = cache(async () => {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts', {
    next: { revalidate: 30 }, // Revalidate every 30s
  });
  return res.json();
});
```

---

## **2. Server Component — Home Page**

```ts
// app/page.tsx
import { fetchPosts, getServerStartTime } from '../lib/cache';
import ClientPost from './ClientPost';

export default async function HomePage() {
  const startTime = getServerStartTime();
  const posts = await fetchPosts();

  return (
    <div>
      <h1>Next.js 16 Post App</h1>
      <p>Server started at: {startTime}</p>

      <h2>Posts List (cached + revalidate 30s)</h2>
      <ul>
        {posts.slice(0, 5).map(post => (
          <li key={post.id}>
            <a href={`/post/${post.id}`}>{post.title}</a>
          </li>
        ))}
      </ul>

      <h2>Client Components (cached per post)</h2>
      {posts.slice(0, 2).map(post => (
        <ClientPost key={post.id} postId={post.id.toString()} />
      ))}
    </div>
  );
}
```

---

## **3. Server Component — Post Page**

```ts
// app/post/[postId]/page.tsx
import { fetchPost } from '../../../lib/cache';

export default async function PostPage({ params }: { params: { postId: string } }) {
  const post = await fetchPost(params.postId);

  return (
    <div>
      <h1>Post {post.id}</h1>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </div>
  );
}
```

---

## **4. Client Component — Fetch Per Post**

```ts
// app/ClientPost.tsx
'use client';
import { use } from 'react';
import { fetchPost } from './lib/cache';

export default function ClientPost({ postId }: { postId: string }) {
  const post = use(fetchPost(postId));

  return (
    <div style={{ border: '1px solid #ccc', padding: '0.5rem', margin: '0.5rem 0' }}>
      <h3>Post {post.id} (Client)</h3>
      <p>{post.title}</p>
    </div>
  );
}
```

---

## **5. Nested Cached Calls**

```ts
// lib/nestedCache.ts
import { cache } from 'react';
import { fetchPosts } from './cache';

export const fetchTopPosts = cache(async () => {
  const posts = await fetchPosts();
  return posts.slice(0, 3); // Cached globally + depends on fetchPosts
});
```

Usage in a server component:

```ts
import { fetchTopPosts } from '../lib/nestedCache';

export default async function TopPosts() {
  const topPosts = await fetchTopPosts();

  return (
    <div>
      <h2>Top 3 Posts (Nested Cache)</h2>
      <ul>
        {topPosts.map(p => (
          <li key={p.id}>{p.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## **6. Partial UI Caching Example**

```ts
// app/components/CachedPosts.tsx
import { cache } from 'react';
import { fetchPosts } from '../lib/cache';

const getCachedPosts = cache(async () => {
  const posts = await fetchPosts();
  return posts.slice(0, 5);
});

export default async function CachedPosts() {
  const posts = await getCachedPosts();

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

```ts
// Usage in a dynamic page
export default async function Page() {
  const time = new Date().toISOString();

  return (
    <div>
      <p>Dynamic server time: {time}</p>
      <CachedPosts /> {/* cached UI part */}
    </div>
  );
}
```

---

## **7. Full Page SSG Example**

```ts
// app/ssg-page.tsx
import { fetchPosts } from '../lib/cache';

// Force full SSG
export const dynamic = 'force-static';

export default async function SSGPage() {
  const posts = await fetchPosts();

  return (
    <div>
      <h1>SSG Posts Page</h1>
      <ul>
        {posts.slice(0, 10).map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

* `dynamic = 'force-static'` ensures **build-time generation**.
* Cached data is read at build-time. You can combine with `revalidate` for ISR.

---

## **8. Caching Flow Diagram**

```mermaid
flowchart TD
    subgraph Server
        A[Server Start] --> B[getServerStartTime (global cache)]
        C[fetchPosts (cache + revalidate)]
        D[fetchPost(postId) (arg cache)]
        E[fetchTopPosts (nested cache)]
        F[CachedPosts component] --> C
    end

    subgraph Client
        G[ClientPost component] -->|use(fetchPost)| D
    end

    A --> B
    B --> C
    C --> E
    E --> G
    C --> F
```

---

## **9. Summary Table**

| Cache Scenario             | Implementation                           | Scope                     | Example                 |
| -------------------------- | ---------------------------------------- | ------------------------- | ----------------------- |
| Global memoization         | `cache(fn)`                              | Server-wide               | `getServerStartTime`    |
| Argument-based             | `cache(fn(arg))`                         | Server-wide per arg       | `fetchPost(postId)`     |
| Stale-while-revalidate     | `fetch(url, { next: { revalidate: X }})` | Global                    | `fetchPosts()`          |
| Client-side suspense cache | `use(fetchPost(postId))`                 | Per request               | `ClientPost` component  |
| Nested cache               | `cache(fn)` calling cached functions     | Global                    | `fetchTopPosts`         |
| Partial UI cache           | `cache()` inside component               | Server-wide for component | `CachedPosts` component |
| Full SSG page              | `export const dynamic = 'force-static'`  | Build-time HTML           | `SSGPage`               |

---

This `.md` file now includes **all caching scenarios in Next.js 16.0.3**, with examples of server component caching, client suspense cache, nested caches, partial UI cache, and fully static pages.
