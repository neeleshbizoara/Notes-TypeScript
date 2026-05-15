# Next.js Cheatsheet

## Routing

### Dynamic Routes
* Use `[slug]` for single, predictable URL segments.
* Use `[...slug]` for multiple/nested segments.

```typescript
// Dynamic Routes
// app/blog/[slug]/page.tsx

// Catch-all Routes
// app/blog/[...slug]/page.tsx
```

---

## Prevent Accidental Leakage of Sensitive Data
* Use `removeConsole` to exclude console logs in production.
* This keeps specific console methods like errors.

```javascript
// Next.config.js
compiler: {
  removeConsole: {
    exclude: ["error"]
  }
}
```

---

## Async APIs
* In Next.js, `params`, `searchParams`, `cookies()`, and `headers()` must now be awaited.

```typescript
// Async APIs
import { cookies, headers } from 'next/headers';

export async function Page({
  params,
  searchParams
}) {
  const { id } = await params;
  const { query } = await searchParams;
  const cookieStore = await cookies();
  const headersList = await headers();
}
```

---

## after()
* Run server-side code in the background.
* Use `after()` to execute expensive operations without blocking the response.

```typescript
// after()
import { after } from 'next/server';

after(() => {
  try {
    await processJob();
  } catch (error) {
    console.error(error);
  }
});
```

---

## Server Components vs Actions
* Server Components never re-render on the client.
* Server Actions get exposed as API endpoints.
* **Server Components:** Useful for `GET` requests.
* **Server Actions:** Useful for `POST`/`PUT`/`DELETE` requests.

---

## Cache Components
* Next.js 15 makes components dynamic by default.
* Use `"use cache"` directive to opt-in to caching for pages, components, or functions.

```typescript
// Use Cache Directive
"use cache";

export async function getData() {
  // This function will be cached
  const data = await fetchData();
  return data;
}
```

---

## Turbopack
* Turbopack is now the default bundler for new Next.js projects.
* It offers faster builds and improved Fast Refresh.

```bash
# Turbopack Bundler
next dev --turbo
```

---

## Tag the Cache for Better Performance
* Tag your cache so you can delete it later.
* Use tags to selectively invalidate specific cached data.
* This avoids expensive deletions of the entire cache.

```typescript
// Tag the Cache
import { cache } from 'next/cache';

export async function getStudent(id) {
  "use cache";
  const res = await fetchStudent(id);
  
  // Tag the cache
  tags: [`xp-info:student:${studentId}`]
}
```

---

## Image Optimization
* Always use `<Image />` instead of `<img>`.
* Provide `width` and `height` to prevent layout shift.
* Use `priority` for above-the-fold images.

```typescript
// Image Component
<Image
  src="/hero.png"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  priority={true}
/>
```

---

## "use cache" - Cache Components - Private
* Allows you to use runtime APIs like cookies, headers, or search params and cache them.

```typescript
// Use Cache Directive
export async function getRecommendations(products: string[]) {
  "use cache";
  // This function will be cached and shared across all users
  return getPersonalizedRecommendations(products);
}
```

---

## Move Repeated Components to the Layout
* `layout.tsx` is a great place to pull repeated components.
* These components are used on every page.
* This way, components render once and cache for the entire application.

```typescript
// Move Repeated Components to the Layout
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <Sidebar />
      <main>{children}</main>
    </>
  );
}
```

---

## Clear the Tagged Cache
* Delete tagged cache selectively.
* Use `revalidateTag()` to bust specific cache tags without clearing the entire page.
* This is more efficient than clearing the entire cache.

```typescript
// Clear the Cache
import { revalidateTag } from 'next/cache';

if (studentId) {
  revalidateTag(`course-progress:student:${studentId}`);
  revalidateTag(`xp-info:student:${studentId}`);
  revalidateTag(`streaks:student:${studentId}`);
}
```

---

## Security Best Practices
* Avoid `NEXT_PUBLIC_` for sensitive vars.
* Use `"server-only"` to prevent server code exposure.
* Add authentication to exposed Server Action endpoints.

```typescript
// Security
import 'server-only';
```

---

## "use cache" - Cache Components - Remote
* Enables caching of remote data when you use dynamic data.

```typescript
// Use Cache Directive
async function getProduct(productId: string) {
  "use cache";
  // Fetch data from a database or remote API
  return db.products.get(productId);
}
```

---

## Server Actions Usage
* Use form action for traditional form submissions.
* Await `serverAction()` for programmatic calls with additional logic.

```typescript
// Server Actions
<form action={serverAction}>
  <button type="submit">Submit</button>
</form>

// Programmatic call
await serverAction();
```

---

## useOptimistic()
* Show optimistic state of completion while expensive work is done on the server for better UX.

```typescript
// useOptimistic()
const [initialState, setInitialState] = useState({
  isCompleted: false,
  isProcessing: false
});

const [optimisticState, addOptimistic] = useOptimistic(
  initialState,
  (state, update) => ({
    ...state,
    ...update
  })
);
```

---

## Data Sanitization
* Use Zod for form validation and sanitization.
* Define strict schemas for your data.
* Validate both client and server side.

```typescript
// Zod Schema
const userSchema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
  password: z.string().min(8)
});
```

---

## Improved Caching APIs
* `revalidateTag()` now requires a cache life profile as the second argument for stale-while-revalidate behavior.

```typescript
// Revalidate Tag
import { revalidateTag } from 'next/cache';

export async function updateProducts() {
  revalidateTag('products', '1h');
}
```

---

## Instant Updates
* Use `"use server"` directive to mark server-only functions.
* Implement update logic.
* Call `revalidatePath` to ensure consistent data.

```typescript
// Instant Updates
"use server";

async function updateCart() {
  // Update cart logic
  revalidatePath('/cart');
}
```

---

## Server-side Redirects
* Use redirect to navigate to a new path.
* Place outside try-catch blocks as redirect throws an error so the Router can route.

```typescript
// Redirects
import { redirect } from "next/navigation";

try {
  // logic
} catch (error) {
  // handle error
}
redirect("/new-path");
```
