# Next.js Interview Questions & Answers

---

## 1. What is Next.js?

Next.js is a React framework developed by Vercel for building full-stack web applications. It provides server-side rendering, static site generation, API routes, and many optimizations out of the box.

Key features:
- **File-based routing** — pages and routes defined by the file system
- **Multiple rendering strategies** — SSR, SSG, ISR, CSR
- **Server Components** — React Server Components by default (App Router)
- **API Routes / Route Handlers** — build backend APIs within the same project
- **Automatic code splitting** — each page loads only the JS it needs
- **Built-in optimizations** — Image, Font, Script, Link components
- **Middleware** — edge functions that run before requests
- **Full-stack capabilities** — Server Actions for form handling and mutations

---

## 2. What is the difference between the App Router and Pages Router?

| Feature | Pages Router (`pages/`) | App Router (`app/`) |
|---|---|---|
| Introduced | Next.js 1+ | Next.js 13+ |
| Components | Client Components by default | Server Components by default |
| Data fetching | `getServerSideProps`, `getStaticProps` | `fetch` in Server Components, `use` |
| Layouts | Manual (per-page) | Nested layouts (automatic) |
| Loading states | Manual | Built-in `loading.tsx` |
| Error handling | `_error.tsx` | Built-in `error.tsx` |
| Routing | File = route | Folder = route, `page.tsx` = UI |
| Streaming | Limited | Full support with Suspense |
| Server Actions | Not supported | Supported |
| Middleware | Supported | Supported |

```
// Pages Router
pages/
├── index.tsx          → /
├── about.tsx          → /about
├── posts/
│   ├── index.tsx      → /posts
│   └── [id].tsx       → /posts/:id
└── api/
    └── users.ts       → /api/users

// App Router
app/
├── page.tsx           → /
├── layout.tsx         → root layout
├── about/
│   └── page.tsx       → /about
├── posts/
│   ├── page.tsx       → /posts
│   └── [id]/
│       └── page.tsx   → /posts/:id
└── api/
    └── users/
        └── route.ts   → /api/users
```

The App Router is the recommended approach for new projects.

---

## 3. What are Server Components vs Client Components?

**Server Components** (default in App Router):
- Render on the server only
- Can directly access databases, file system, and server-side resources
- Cannot use hooks (`useState`, `useEffect`), browser APIs, or event handlers
- Zero JavaScript sent to the client
- Better performance and SEO

**Client Components:**
- Render on both server (SSR) and client (hydration)
- Can use hooks, browser APIs, and event handlers
- Marked with `'use client'` directive
- JavaScript is sent to the client for interactivity

```tsx
// Server Component (default — no directive needed)
async function ProductList() {
  const products = await db.product.findMany(); // direct DB access

  return (
    <ul>
      {products.map(p => (
        <li key={p.id}>{p.name} - ${p.price}</li>
      ))}
    </ul>
  );
}

// Client Component
'use client';

import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

**Composition pattern:**

```tsx
// Server Component wrapping Client Component
import Counter from './Counter'; // Client Component

async function Dashboard() {
  const stats = await getStats(); // server-side data fetch

  return (
    <div>
      <h1>Dashboard</h1>
      <p>Total users: {stats.users}</p>
      <Counter /> {/* interactive client component */}
    </div>
  );
}
```

**Rules:**
- Server Components can import Client Components
- Client Components cannot import Server Components (but can receive them as `children` prop)
- Mark a component as `'use client'` only when it needs interactivity

---

## 4. What are the rendering strategies in Next.js?

**Static Site Generation (SSG):**
- Pages are generated at build time
- Served from CDN, fastest option
- Default for pages without dynamic data

```tsx
// App Router — static by default
async function BlogPage() {
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache', // default — cached at build time
  });
  return <PostList posts={await posts.json()} />;
}

// Pages Router
export async function getStaticProps() {
  const posts = await fetchPosts();
  return { props: { posts } };
}
```

**Server-Side Rendering (SSR):**
- Pages are generated on every request
- Always fresh data

```tsx
// App Router — use no-store cache
async function DashboardPage() {
  const data = await fetch('https://api.example.com/stats', {
    cache: 'no-store', // fresh data on every request
  });
  return <Dashboard data={await data.json()} />;
}

// Pages Router
export async function getServerSideProps(context) {
  const data = await fetchDashboard();
  return { props: { data } };
}
```

**Incremental Static Regeneration (ISR):**
- Static pages that revalidate after a set time
- Best of both SSG and SSR

```tsx
// App Router — revalidate with time-based option
async function ProductPage() {
  const product = await fetch('https://api.example.com/product/1', {
    next: { revalidate: 60 }, // revalidate every 60 seconds
  });
  return <Product data={await product.json()} />;
}

// Pages Router
export async function getStaticProps() {
  const product = await fetchProduct();
  return { props: { product }, revalidate: 60 };
}
```

**Client-Side Rendering (CSR):**
- Data fetched in the browser after page load

```tsx
'use client';

import { useEffect, useState } from 'react';

function UserProfile() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch('/api/user').then(r => r.json()).then(setUser);
  }, []);

  if (!user) return <Skeleton />;
  return <Profile user={user} />;
}
```

---

## 5. What is the file-based routing system?

Next.js App Router uses folders and special files to define routes.

**Special files:**

| File | Purpose |
|---|---|
| `page.tsx` | UI for the route (makes route publicly accessible) |
| `layout.tsx` | Shared layout wrapping children (persists across navigation) |
| `loading.tsx` | Loading UI (wraps page in Suspense) |
| `error.tsx` | Error boundary for the route |
| `not-found.tsx` | 404 UI for the route |
| `template.tsx` | Like layout but re-mounts on navigation |
| `route.ts` | API endpoint (Route Handler) |
| `default.tsx` | Fallback for parallel routes |

**Route patterns:**

```
app/
├── page.tsx                    → /
├── about/page.tsx              → /about
├── blog/
│   ├── page.tsx                → /blog
│   └── [slug]/page.tsx         → /blog/:slug (dynamic)
├── shop/
│   └── [...slug]/page.tsx      → /shop/* (catch-all)
├── docs/
│   └── [[...slug]]/page.tsx    → /docs, /docs/* (optional catch-all)
├── (marketing)/                → route group (no URL impact)
│   ├── about/page.tsx          → /about
│   └── contact/page.tsx        → /contact
├── @sidebar/                   → parallel route (named slot)
│   └── page.tsx
└── (.)photo/[id]/page.tsx      → intercepting route
```

---

## 6. What are Layouts in Next.js?

Layouts are UI that wraps child routes. They persist across navigations and don't re-render.

```tsx
// app/layout.tsx — Root layout (required)
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Navbar />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}

// app/dashboard/layout.tsx — Nested layout
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex">
      <Sidebar />
      <div className="flex-1">{children}</div>
    </div>
  );
}

// Layouts are nested automatically:
// / → RootLayout > page.tsx
// /dashboard → RootLayout > DashboardLayout > page.tsx
// /dashboard/settings → RootLayout > DashboardLayout > page.tsx
```

**Layout vs Template:**
- `layout.tsx` — persists state, doesn't re-mount between navigations
- `template.tsx` — re-mounts on every navigation (useful for enter/exit animations, resetting state)

---

## 7. What are `loading.tsx` and `error.tsx`?

**`loading.tsx`** — automatically wraps the page in a React Suspense boundary:

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/3 mb-4" />
      <div className="h-4 bg-gray-200 rounded w-full mb-2" />
      <div className="h-4 bg-gray-200 rounded w-2/3" />
    </div>
  );
}
```

**`error.tsx`** — catches errors in the route segment and its children:

```tsx
// app/dashboard/error.tsx
'use client'; // must be a Client Component

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

**`not-found.tsx`** — displayed when `notFound()` is called:

```tsx
// app/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>404 — Page Not Found</h2>
      <p>The page you are looking for does not exist.</p>
    </div>
  );
}
```

```tsx
import { notFound } from 'next/navigation';

async function UserPage({ params }: { params: { id: string } }) {
  const user = await getUser(params.id);
  if (!user) notFound();
  return <UserProfile user={user} />;
}
```

---

## 8. What are Route Handlers (API Routes)?

Route Handlers define API endpoints in the App Router using `route.ts` files.

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = searchParams.get('page') || '1';

  const users = await db.user.findMany({
    skip: (Number(page) - 1) * 10,
    take: 10,
  });

  return NextResponse.json({ data: users, page: Number(page) });
}

export async function POST(request: NextRequest) {
  const body = await request.json();

  const user = await db.user.create({ data: body });

  return NextResponse.json(user, { status: 201 });
}

// app/api/users/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.user.findUnique({ where: { id: params.id } });

  if (!user) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  return NextResponse.json(user);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await db.user.delete({ where: { id: params.id } });
  return new NextResponse(null, { status: 204 });
}

// Setting headers and cookies
export async function GET(request: NextRequest) {
  const response = NextResponse.json({ data: 'hello' });
  response.headers.set('X-Custom-Header', 'value');
  response.cookies.set('token', 'abc123', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 60 * 60 * 24,
  });
  return response;
}
```

---

## 9. What are Server Actions?

Server Actions are async functions that run on the server, used for form submissions and data mutations.

```tsx
// Inline Server Action (in Server Component)
async function TodoList() {
  const todos = await db.todo.findMany();

  async function addTodo(formData: FormData) {
    'use server';
    const title = formData.get('title') as string;
    await db.todo.create({ data: { title } });
    revalidatePath('/todos');
  }

  return (
    <div>
      <form action={addTodo}>
        <input name="title" placeholder="New todo" required />
        <button type="submit">Add</button>
      </form>
      <ul>
        {todos.map(t => <li key={t.id}>{t.title}</li>)}
      </ul>
    </div>
  );
}

// Separate actions file
// app/actions/users.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  await db.user.create({ data: { name, email } });

  revalidatePath('/users');
  redirect('/users');
}

export async function deleteUser(id: string) {
  await db.user.delete({ where: { id } });
  revalidatePath('/users');
}

// Using Server Actions in Client Components
'use client';

import { createUser } from '@/app/actions/users';
import { useFormStatus, useFormState } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create User'}
    </button>
  );
}

function CreateUserForm() {
  const [state, formAction] = useFormState(createUser, null);

  return (
    <form action={formAction}>
      <input name="name" required />
      <input name="email" type="email" required />
      <SubmitButton />
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```

---

## 10. What is data fetching in the App Router?

In the App Router, data is fetched directly in Server Components using `fetch` or any async function.

```tsx
// Fetch with caching options
async function ProductsPage() {
  // Static (cached indefinitely) — default
  const products = await fetch('https://api.example.com/products');

  // Revalidate every 60 seconds (ISR)
  const featured = await fetch('https://api.example.com/featured', {
    next: { revalidate: 60 },
  });

  // No cache (SSR — fresh on every request)
  const cart = await fetch('https://api.example.com/cart', {
    cache: 'no-store',
  });

  return <ProductList products={await products.json()} />;
}

// Direct database access (no fetch needed)
async function UsersPage() {
  const users = await prisma.user.findMany();
  return <UserList users={users} />;
}

// Parallel data fetching
async function Dashboard() {
  // Start both fetches simultaneously
  const statsPromise = getStats();
  const notificationsPromise = getNotifications();

  const [stats, notifications] = await Promise.all([
    statsPromise,
    notificationsPromise,
  ]);

  return (
    <div>
      <Stats data={stats} />
      <Notifications data={notifications} />
    </div>
  );
}

// Sequential data fetching (when one depends on another)
async function UserPosts({ userId }: { userId: string }) {
  const user = await getUser(userId);
  const posts = await getPosts(user.id); // depends on user
  return <PostList posts={posts} />;
}
```

---

## 11. What is caching in Next.js?

Next.js has multiple caching layers:

| Cache | What | Where | Invalidation |
|---|---|---|---|
| **Request Memoization** | Deduplicates identical `fetch` calls in a single render | Server | Automatic per request |
| **Data Cache** | Caches `fetch` results across requests | Server | `revalidate`, `revalidatePath`, `revalidateTag` |
| **Full Route Cache** | Caches rendered HTML and RSC payload | Server | Revalidation, redeployment |
| **Router Cache** | Caches RSC payload in the browser | Client | Time-based, `router.refresh()` |

```tsx
// Tag-based revalidation
async function ProductPage({ params }) {
  const product = await fetch(`https://api.example.com/products/${params.id}`, {
    next: { tags: ['products', `product-${params.id}`] },
  });

  return <Product data={await product.json()} />;
}

// Revalidate by tag (in Server Action or Route Handler)
import { revalidateTag, revalidatePath } from 'next/cache';

export async function updateProduct(id: string, data: any) {
  'use server';
  await db.product.update({ where: { id }, data });
  revalidateTag(`product-${id}`);   // revalidate specific product
  revalidateTag('products');          // revalidate all products
  revalidatePath('/products');        // revalidate the products page
}

// Opt out of caching
export const dynamic = 'force-dynamic'; // entire route
export const revalidate = 0;           // entire route
```

---

## 12. What is Middleware in Next.js?

Middleware runs before a request is processed and can modify the request, redirect, rewrite, or set headers.

```tsx
// middleware.ts (root of project)
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Authentication check
  const token = request.cookies.get('token')?.value;

  if (pathname.startsWith('/dashboard') && !token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Add custom headers
  const response = NextResponse.next();
  response.headers.set('X-Request-Id', crypto.randomUUID());

  // Rewrite (internal routing without URL change)
  if (pathname === '/old-page') {
    return NextResponse.rewrite(new URL('/new-page', request.url));
  }

  // Geolocation-based routing
  const country = request.geo?.country || 'US';
  if (pathname === '/' && country === 'DE') {
    return NextResponse.rewrite(new URL('/de', request.url));
  }

  return response;
}

// Configure which paths middleware runs on
export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

Middleware runs at the edge (not Node.js runtime), so it has limited API access. It cannot use Node.js APIs like `fs`.

---

## 13. What is the `next/image` component?

`next/image` provides automatic image optimization.

```tsx
import Image from 'next/image';

// Local image (auto-sized)
import heroImage from '@/public/hero.jpg';

<Image
  src={heroImage}
  alt="Hero banner"
  placeholder="blur"          // blur placeholder while loading
  priority                     // preload (for LCP images)
/>

// Remote image (must specify dimensions)
<Image
  src="https://example.com/photo.jpg"
  alt="Photo"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, 50vw"
/>

// Fill container
<div className="relative h-64">
  <Image
    src="/background.jpg"
    alt="Background"
    fill                       // fills parent container
    style={{ objectFit: 'cover' }}
    sizes="100vw"
  />
</div>
```

**Benefits:**
- Automatic format conversion (WebP, AVIF)
- Responsive images with `srcset`
- Lazy loading by default
- Prevents layout shift (requires dimensions)
- On-demand optimization (images are optimized when requested)

**Configure remote domains in `next.config.js`:**

```js
module.exports = {
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'example.com' },
      { protocol: 'https', hostname: '**.amazonaws.com' },
    ],
  },
};
```

---

## 14. What is the `next/link` component?

`next/link` enables client-side navigation with prefetching.

```tsx
import Link from 'next/link';

// Basic link
<Link href="/about">About</Link>

// Dynamic route
<Link href={`/posts/${post.id}`}>Read More</Link>

// With query params
<Link href={{ pathname: '/search', query: { q: 'nextjs' } }}>
  Search
</Link>

// Replace history (no back button entry)
<Link href="/dashboard" replace>Dashboard</Link>

// Disable prefetching
<Link href="/heavy-page" prefetch={false}>Heavy Page</Link>

// Scroll to top (default: true)
<Link href="/page" scroll={false}>Stay in position</Link>

// Active link pattern
'use client';
import { usePathname } from 'next/navigation';

function NavLink({ href, children }) {
  const pathname = usePathname();
  const isActive = pathname === href;

  return (
    <Link href={href} className={isActive ? 'text-blue-600 font-bold' : ''}>
      {children}
    </Link>
  );
}
```

`Link` automatically prefetches linked pages when they enter the viewport, making navigation near-instant.

---

## 15. What are `next/font` optimizations?

`next/font` self-hosts fonts with zero layout shift.

```tsx
// Google Fonts (downloaded at build time, no external requests)
import { Inter, Roboto_Mono } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  variable: '--font-roboto-mono',
});

// Root layout
export default function RootLayout({ children }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}

// Local fonts
import localFont from 'next/font/local';

const myFont = localFont({
  src: [
    { path: './fonts/MyFont-Regular.woff2', weight: '400', style: 'normal' },
    { path: './fonts/MyFont-Bold.woff2', weight: '700', style: 'normal' },
  ],
  variable: '--font-my-font',
});

// Use in CSS via variables
// font-family: var(--font-inter);
```

**Benefits:**
- No external network requests (self-hosted)
- Zero layout shift (`size-adjust` applied automatically)
- CSS `font-display` support
- Automatic subset optimization

---

## 16. What is Dynamic Rendering vs Static Rendering?

**Static Rendering (default):**
- Routes are rendered at build time or on first request (then cached)
- Same HTML served to all users
- Deployed to CDN

**Dynamic Rendering:**
- Routes are rendered on each request
- Used when content depends on request-time information (cookies, headers, search params)

```tsx
// This triggers dynamic rendering automatically:

// 1. Using dynamic functions
import { cookies, headers } from 'next/headers';

async function Page() {
  const cookieStore = cookies();
  const token = cookieStore.get('token');
  // ...
}

// 2. Using searchParams
async function Page({ searchParams }) {
  const query = searchParams.q; // triggers dynamic
}

// 3. Using cache: 'no-store' in fetch
const data = await fetch(url, { cache: 'no-store' });

// Force static or dynamic
export const dynamic = 'force-static';    // force static rendering
export const dynamic = 'force-dynamic';   // force dynamic rendering
export const revalidate = 60;             // ISR (revalidate every 60s)
export const revalidate = 0;              // always dynamic
```

---

## 17. What are Parallel Routes?

Parallel routes render multiple pages in the same layout simultaneously using named slots.

```
app/
├── layout.tsx
├── page.tsx
├── @analytics/
│   └── page.tsx
├── @team/
│   └── page.tsx
└── @revenue/
    └── page.tsx
```

```tsx
// app/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
  revenue,
}: {
  children: React.ReactNode;
  analytics: React.ReactNode;
  team: React.ReactNode;
  revenue: React.ReactNode;
}) {
  return (
    <div>
      {children}
      <div className="grid grid-cols-3 gap-4">
        {analytics}
        {team}
        {revenue}
      </div>
    </div>
  );
}
```

Use cases:
- Dashboard with multiple independent panels
- Modals that can be URL-driven
- Conditional rendering based on authentication state

---

## 18. What are Intercepting Routes?

Intercepting routes allow loading a route within the current layout while keeping the context of the current page (like modals).

```
app/
├── feed/
│   └── page.tsx
├── photo/
│   └── [id]/
│       └── page.tsx           → /photo/:id (full page)
└── feed/
    └── (.)photo/
        └── [id]/
            └── page.tsx       → intercepts /photo/:id when navigating from /feed
```

**Convention:**
- `(.)` — intercept same level
- `(..)` — intercept one level above
- `(..)(..)` — intercept two levels above
- `(...)` — intercept from root

```tsx
// app/feed/(.)photo/[id]/page.tsx — modal view
export default function PhotoModal({ params }: { params: { id: string } }) {
  return (
    <Modal>
      <Photo id={params.id} />
    </Modal>
  );
}

// app/photo/[id]/page.tsx — full page view (direct URL or refresh)
export default function PhotoPage({ params }: { params: { id: string } }) {
  return <Photo id={params.id} />;
}
```

This pattern is used for Instagram-like photo modals — clicking a photo opens a modal, but sharing the URL or refreshing shows the full page.

---

## 19. What are Route Groups?

Route groups organize routes without affecting the URL structure using parentheses `()`.

```
app/
├── (marketing)/
│   ├── layout.tsx          → marketing layout
│   ├── about/page.tsx      → /about
│   └── contact/page.tsx    → /contact
├── (shop)/
│   ├── layout.tsx          → shop layout
│   ├── products/page.tsx   → /products
│   └── cart/page.tsx       → /cart
└── (auth)/
    ├── layout.tsx          → auth layout (no navbar)
    ├── login/page.tsx      → /login
    └── register/page.tsx   → /register
```

Use cases:
- Different layouts for different sections (marketing vs app vs auth)
- Organizing related routes without adding URL segments
- Separating concerns in large applications

---

## 20. What is `generateStaticParams`?

`generateStaticParams` pre-generates dynamic routes at build time (replaces `getStaticPaths` from Pages Router).

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getAllPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return <article>{post.content}</article>;
}

// Nested dynamic routes
// app/products/[category]/[id]/page.tsx
export async function generateStaticParams() {
  const products = await getAllProducts();

  return products.map((product) => ({
    category: product.category,
    id: product.id,
  }));
}

// Control behavior for paths not generated at build time
export const dynamicParams = true;  // default — generate on demand
export const dynamicParams = false; // return 404 for unknown paths
```

---

## 21. What is `generateMetadata`?

`generateMetadata` creates dynamic metadata (title, description, Open Graph) for SEO.

```tsx
import { Metadata } from 'next';

// Static metadata
export const metadata: Metadata = {
  title: 'My App',
  description: 'A description of my app',
};

// Dynamic metadata
export async function generateMetadata({
  params,
}: {
  params: { id: string };
}): Promise<Metadata> {
  const product = await getProduct(params.id);

  return {
    title: product.name,
    description: product.description,
    openGraph: {
      title: product.name,
      description: product.description,
      images: [{ url: product.image, width: 1200, height: 630 }],
    },
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description,
    },
  };
}

// Template-based titles
// app/layout.tsx
export const metadata: Metadata = {
  title: {
    template: '%s | My App',    // child pages use this template
    default: 'My App',          // fallback
  },
};

// app/about/page.tsx
export const metadata: Metadata = {
  title: 'About',               // renders as "About | My App"
};
```

---

## 22. What is Streaming and Suspense in Next.js?

Streaming progressively sends HTML from the server, allowing parts of the page to render before all data is ready.

```tsx
import { Suspense } from 'react';

async function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Shows immediately */}
      <StaticContent />

      {/* Streams in when data is ready */}
      <Suspense fallback={<ChartSkeleton />}>
        <SlowChart />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <SlowTable />
      </Suspense>
    </div>
  );
}

// SlowChart fetches its own data (no waterfall)
async function SlowChart() {
  const data = await fetch('https://api.example.com/chart-data');
  return <Chart data={await data.json()} />;
}

async function SlowTable() {
  const data = await fetch('https://api.example.com/table-data');
  return <DataTable data={await data.json()} />;
}
```

**Benefits:**
- Users see content immediately (no blank page)
- Slow data sources don't block the entire page
- `loading.tsx` is syntactic sugar for Suspense at the route level
- Improves TTFB and FCP metrics

---

## 23. What is the `next.config.js` file?

`next.config.js` configures Next.js behavior.

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Rewrites (proxy, URL mapping)
  async rewrites() {
    return [
      { source: '/api/:path*', destination: 'https://api.example.com/:path*' },
    ];
  },

  // Redirects
  async redirects() {
    return [
      { source: '/old-blog/:slug', destination: '/blog/:slug', permanent: true },
    ];
  },

  // Headers
  async headers() {
    return [
      {
        source: '/api/:path*',
        headers: [
          { key: 'Access-Control-Allow-Origin', value: '*' },
        ],
      },
    ];
  },

  // Images
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: '**.example.com' },
    ],
    formats: ['image/avif', 'image/webp'],
  },

  // Environment variables (public)
  env: {
    CUSTOM_VAR: 'value',
  },

  // Experimental features
  experimental: {
    serverActions: { bodySizeLimit: '2mb' },
    ppr: true, // Partial Prerendering
  },

  // Webpack customization
  webpack: (config, { isServer }) => {
    // Custom webpack config
    return config;
  },

  // Output mode
  output: 'standalone', // for Docker deployments

  // Base path
  basePath: '/app',

  // Internationalization
  i18n: {
    locales: ['en', 'es', 'fr'],
    defaultLocale: 'en',
  },
};

module.exports = nextConfig;
```

---

## 24. What are environment variables in Next.js?

```
# .env.local (not committed to git)
DATABASE_URL=postgres://localhost:5432/mydb
JWT_SECRET=supersecret

# NEXT_PUBLIC_ prefix makes variables available in the browser
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_GA_ID=G-12345
```

```tsx
// Server-side only (Server Components, Route Handlers, Server Actions)
const dbUrl = process.env.DATABASE_URL;
const secret = process.env.JWT_SECRET;

// Client-side (available in browser)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

**Loading order:** `.env` → `.env.local` → `.env.development` / `.env.production` → `.env.development.local` / `.env.production.local`

Variables without `NEXT_PUBLIC_` are only available on the server. They are replaced at build time, not runtime.

---

## 25. What is the `useRouter` hook?

```tsx
'use client';

import { useRouter, usePathname, useSearchParams, useParams } from 'next/navigation';

function NavigationExample() {
  const router = useRouter();
  const pathname = usePathname();           // current path
  const searchParams = useSearchParams();   // query params
  const params = useParams();               // dynamic route params

  // Navigation methods
  router.push('/dashboard');                // navigate (adds to history)
  router.replace('/dashboard');             // navigate (replaces history)
  router.back();                            // go back
  router.forward();                         // go forward
  router.refresh();                         // re-fetch server components
  router.prefetch('/about');                // prefetch a route

  // Reading values
  const page = searchParams.get('page');    // ?page=2
  const id = params.id;                     // /users/[id]

  // Building URLs with search params
  const createQueryString = (name: string, value: string) => {
    const params = new URLSearchParams(searchParams.toString());
    params.set(name, value);
    return params.toString();
  };

  return (
    <button onClick={() => router.push(`/search?${createQueryString('q', 'next')}`)}>
      Search
    </button>
  );
}
```

---

## 26. What is the `next/script` component?

`next/script` optimizes loading of third-party scripts.

```tsx
import Script from 'next/script';

// Loading strategies
<Script src="https://analytics.com/script.js" strategy="afterInteractive" />
// afterInteractive — default, loads after page is interactive
// beforeInteractive — loads before page hydrates (critical scripts)
// lazyOnload — loads during idle time
// worker — loads in a web worker (experimental)

// Inline scripts
<Script id="schema" type="application/ld+json" strategy="afterInteractive">
  {JSON.stringify({
    '@context': 'https://schema.org',
    '@type': 'WebSite',
    name: 'My Site',
  })}
</Script>

// Event handlers
<Script
  src="https://maps.googleapis.com/maps/api/js"
  onLoad={() => console.log('Google Maps loaded')}
  onError={() => console.error('Failed to load Google Maps')}
  onReady={() => console.log('Script ready')}
/>
```

---

## 27. What is Partial Prerendering (PPR)?

PPR (experimental) combines static and dynamic rendering in a single route. The static shell is served immediately from CDN, and dynamic content streams in.

```tsx
// next.config.js
module.exports = {
  experimental: { ppr: true },
};

// Static shell is prerendered, dynamic parts stream in
export default function ProductPage({ params }) {
  return (
    <div>
      {/* Static — prerendered at build time */}
      <Header />
      <ProductDetails id={params.id} />

      {/* Dynamic — streams in when ready */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews id={params.id} /> {/* fetches from DB per request */}
      </Suspense>

      <Suspense fallback={<CartSkeleton />}>
        <CartWidget /> {/* reads cookies — always dynamic */}
      </Suspense>

      {/* Static */}
      <Footer />
    </div>
  );
}
```

PPR combines the speed of static serving with the freshness of dynamic rendering.

---

## 28. What is internationalization (i18n) in Next.js?

**App Router approach:**

```
app/
└── [lang]/
    ├── layout.tsx
    ├── page.tsx
    └── about/
        └── page.tsx
```

```tsx
// app/[lang]/layout.tsx
export async function generateStaticParams() {
  return [{ lang: 'en' }, { lang: 'es' }, { lang: 'fr' }];
}

export default function LangLayout({
  children,
  params: { lang },
}: {
  children: React.ReactNode;
  params: { lang: string };
}) {
  return <html lang={lang}><body>{children}</body></html>;
}

// Dictionary loader
const dictionaries = {
  en: () => import('./dictionaries/en.json').then(m => m.default),
  es: () => import('./dictionaries/es.json').then(m => m.default),
};

async function getDictionary(lang: string) {
  return dictionaries[lang]();
}

// app/[lang]/page.tsx
export default async function Home({ params: { lang } }) {
  const dict = await getDictionary(lang);
  return <h1>{dict.welcome}</h1>;
}

// middleware.ts — detect and redirect to locale
import { match } from '@formatjs/intl-localematcher';
import Negotiator from 'negotiator';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const pathnameHasLocale = locales.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`,
  );

  if (pathnameHasLocale) return;

  const locale = getLocale(request); // detect from Accept-Language header
  request.nextUrl.pathname = `/${locale}${pathname}`;
  return NextResponse.redirect(request.nextUrl);
}
```

---

## 29. What are the authentication patterns in Next.js?

```tsx
// 1. Middleware-based auth (edge — runs before page loads)
// middleware.ts
export function middleware(request: NextRequest) {
  const token = request.cookies.get('session')?.value;

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

// 2. Server Component auth check
import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';

async function DashboardPage() {
  const session = await getSession(cookies());

  if (!session) redirect('/login');

  return <Dashboard user={session.user} />;
}

// 3. NextAuth.js (Auth.js) integration
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [GitHub],
});

// app/api/auth/[...nextauth]/route.ts
export { handlers as GET, handlers as POST } from '@/auth';

// Server Component
import { auth } from '@/auth';

async function Page() {
  const session = await auth();
  if (!session) return <SignIn />;
  return <p>Welcome {session.user.name}</p>;
}

// 4. Layout-level protection
async function ProtectedLayout({ children }) {
  const session = await auth();
  if (!session) redirect('/login');

  return (
    <div>
      <Navbar user={session.user} />
      {children}
    </div>
  );
}
```

---

## 30. What is the difference between `redirect` and `rewrite` in Next.js?

```tsx
// Redirect — changes the URL in the browser (301/302)
import { redirect } from 'next/navigation';

async function Page() {
  const user = await getUser();
  if (!user) redirect('/login'); // 307 temporary redirect
}

// permanent redirect
import { permanentRedirect } from 'next/navigation';
permanentRedirect('/new-url'); // 308 permanent redirect

// In next.config.js
async redirects() {
  return [
    {
      source: '/old-path',
      destination: '/new-path',
      permanent: true, // 308
    },
  ];
}

// Rewrite — serves content from a different path WITHOUT changing the URL
async rewrites() {
  return [
    {
      source: '/api/:path*',
      destination: 'https://backend.example.com/:path*', // proxy API
    },
    {
      source: '/blog',
      destination: '/news', // serve /news content at /blog URL
    },
  ];
}
```

**Key difference:** Redirects change the URL in the browser. Rewrites serve different content at the same URL (useful for proxying, A/B testing, and URL masking).

---

## 31. What is `next/og` (Open Graph Image Generation)?

`next/og` generates dynamic Open Graph images at the edge.

```tsx
// app/api/og/route.tsx
import { ImageResponse } from 'next/og';

export const runtime = 'edge';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get('title') || 'My Site';

  return new ImageResponse(
    (
      <div
        style={{
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          backgroundColor: '#1a1a2e',
          color: 'white',
          fontSize: 48,
          fontWeight: 'bold',
        }}
      >
        {title}
      </div>
    ),
    { width: 1200, height: 630 },
  );
}

// Use in metadata
export async function generateMetadata({ params }) {
  return {
    openGraph: {
      images: [`/api/og?title=${encodeURIComponent('My Post Title')}`],
    },
  };
}
```

---

## 32. What are the Edge and Node.js runtimes?

Next.js supports two server runtimes:

| Feature | Node.js Runtime | Edge Runtime |
|---|---|---|
| Cold start | Slower | Near-instant |
| APIs available | Full Node.js | Limited Web APIs |
| Streaming | Yes | Yes |
| Data fetching | All methods | `fetch` only |
| npm packages | All | Must be edge-compatible |
| Location | Region-specific | Globally distributed |

```tsx
// Set runtime per route
export const runtime = 'edge';     // Edge Runtime
export const runtime = 'nodejs';   // Node.js Runtime (default)

// Middleware always runs on Edge
// Route Handlers can use either
// Server Components use Node.js by default
```

Use Edge for:
- Middleware (auth, redirects, A/B testing)
- Simple API responses
- Low-latency requirements

Use Node.js for:
- Database access
- File system operations
- Heavy computation
- Packages requiring Node.js APIs

---

## 33. What is `next export` / Static Export?

Static Export generates a fully static site that can be deployed without a Node.js server.

```js
// next.config.js
module.exports = {
  output: 'export',
};
```

**Limitations:**
- No Server Components with dynamic rendering
- No API Routes
- No ISR
- No Middleware
- No `next/image` optimization (needs loader)
- No dynamic routes without `generateStaticParams`

**Supported:**
- Static rendering (SSG)
- Client-side rendering
- Image optimization with external loaders
- Static API (JSON files)

Use for: documentation sites, marketing pages, blogs with pre-built content.

---

## 34. How do you handle forms in Next.js?

```tsx
// Server Action approach (recommended)
// app/contact/page.tsx
import { redirect } from 'next/navigation';

async function submitForm(formData: FormData) {
  'use server';

  const name = formData.get('name');
  const email = formData.get('email');
  const message = formData.get('message');

  // Validate
  if (!name || !email) {
    return { error: 'Name and email are required' };
  }

  // Save to database
  await db.contact.create({
    data: { name, email, message },
  });

  redirect('/contact/success');
}

export default function ContactPage() {
  return (
    <form action={submitForm}>
      <input name="name" required />
      <input name="email" type="email" required />
      <textarea name="message" />
      <SubmitButton />
    </form>
  );
}

// Client Component with validation and loading states
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { submitForm } from './actions';

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Sending...' : 'Send'}</button>;
}

export default function ContactForm() {
  const [state, formAction] = useFormState(submitForm, { error: null });

  return (
    <form action={formAction}>
      <input name="name" required />
      <input name="email" type="email" required />
      {state?.error && <p className="text-red-500">{state.error}</p>}
      <SubmitButton />
    </form>
  );
}
```

---

## 35. What are common performance optimizations in Next.js?

1. **Use Server Components** — reduce client-side JavaScript
2. **Streaming with Suspense** — don't block the entire page on slow data
3. **Image optimization** — use `next/image` with `sizes` and `priority`
4. **Font optimization** — use `next/font` for zero layout shift
5. **Route segment caching** — configure `revalidate` appropriately
6. **Parallel data fetching** — use `Promise.all` for independent fetches
7. **Dynamic imports** — lazy load heavy components

```tsx
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('./Chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // only load on client
});
```

8. **Prefetching** — `<Link>` auto-prefetches, use `router.prefetch` for programmatic
9. **Bundle analysis:**

```bash
npm install @next/bundle-analyzer
```

```js
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});
module.exports = withBundleAnalyzer(nextConfig);
```

10. **Use `loading.tsx`** — instant feedback while data loads

---

## 36. What is the `unstable_cache` function?

`unstable_cache` caches the result of expensive computations or database queries.

```tsx
import { unstable_cache } from 'next/cache';

const getCachedUser = unstable_cache(
  async (id: string) => {
    return await db.user.findUnique({ where: { id } });
  },
  ['user-cache'],          // cache key prefix
  {
    revalidate: 3600,      // revalidate every hour
    tags: ['users'],       // for on-demand revalidation
  },
);

// Usage
async function UserPage({ params }) {
  const user = await getCachedUser(params.id);
  return <Profile user={user} />;
}

// Invalidate
import { revalidateTag } from 'next/cache';
revalidateTag('users');
```

---

## 37. What are Turbopack and its benefits?

Turbopack is Next.js's Rust-based bundler (successor to Webpack for development).

```bash
# Use Turbopack for dev server
next dev --turbo
```

**Benefits:**
- Up to 10x faster cold starts than Webpack
- Up to 700x faster updates than Webpack
- Incremental computation (only rebuilds what changed)
- Built in Rust for native performance
- Same configuration as Webpack (gradual migration)

Turbopack is currently available for development mode. Production builds still use Webpack.

---

## 38. What is `instrumentation.ts`?

`instrumentation.ts` runs code when the Next.js server starts, useful for monitoring and tracing setup.

```tsx
// instrumentation.ts (root of project)
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    // Node.js runtime initialization
    const { initTracing } = await import('./lib/tracing');
    initTracing();
  }

  if (process.env.NEXT_RUNTIME === 'edge') {
    // Edge runtime initialization
  }
}
```

```js
// next.config.js
module.exports = {
  experimental: {
    instrumentationHook: true,
  },
};
```

Use cases: OpenTelemetry setup, Sentry initialization, database connection pooling, custom logging.

---

## 39. What is the `robots.ts` and `sitemap.ts` convention?

Next.js can generate `robots.txt` and `sitemap.xml` dynamically.

```tsx
// app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      { userAgent: '*', allow: '/', disallow: '/admin/' },
      { userAgent: 'Googlebot', allow: '/' },
    ],
    sitemap: 'https://example.com/sitemap.xml',
  };
}

// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPosts();

  const postEntries = posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly' as const,
    priority: 0.8,
  }));

  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    ...postEntries,
  ];
}
```

---

## 40. What is the `opengraph-image` convention?

Next.js can generate OG images using special file conventions.

```
app/
├── opengraph-image.tsx      → /opengraph-image (auto-linked in metadata)
├── twitter-image.tsx
├── icon.tsx
└── apple-icon.tsx
```

```tsx
// app/opengraph-image.tsx
import { ImageResponse } from 'next/og';

export const size = { width: 1200, height: 630 };
export const contentType = 'image/png';

export default function OGImage() {
  return new ImageResponse(
    (
      <div style={{
        width: '100%', height: '100%',
        display: 'flex', alignItems: 'center', justifyContent: 'center',
        background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
        color: 'white', fontSize: 64, fontWeight: 'bold',
      }}>
        My Website
      </div>
    ),
    size,
  );
}
```

---

## 41. What is the difference between `cookies()` and `headers()`?

```tsx
import { cookies, headers } from 'next/headers';

// Read cookies (Server Component / Route Handler)
async function Page() {
  const cookieStore = cookies();
  const token = cookieStore.get('session-token')?.value;
  const allCookies = cookieStore.getAll();
  const hasToken = cookieStore.has('session-token');
}

// Set cookies (Server Action / Route Handler only)
async function login() {
  'use server';
  cookies().set('session-token', token, {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7, // 1 week
    path: '/',
  });
}

// Read headers
async function Page() {
  const headersList = headers();
  const userAgent = headersList.get('user-agent');
  const referer = headersList.get('referer');
  const authorization = headersList.get('authorization');
}
```

Both `cookies()` and `headers()` trigger dynamic rendering — the route cannot be statically generated.

---

## 42. What is `revalidatePath` vs `revalidateTag`?

```tsx
import { revalidatePath, revalidateTag } from 'next/cache';

// revalidatePath — revalidate a specific route
export async function updatePost(id: string, data: any) {
  'use server';
  await db.post.update({ where: { id }, data });

  revalidatePath('/blog');              // revalidate blog listing
  revalidatePath(`/blog/${id}`);        // revalidate specific post
  revalidatePath('/blog', 'layout');    // revalidate including layout
  revalidatePath('/', 'layout');        // revalidate everything
}

// revalidateTag — revalidate by cache tag
async function ProductPage({ params }) {
  const product = await fetch(`/api/products/${params.id}`, {
    next: { tags: ['products', `product-${params.id}`] },
  });
}

export async function updateProduct(id: string, data: any) {
  'use server';
  await db.product.update({ where: { id }, data });

  revalidateTag(`product-${id}`);  // revalidate just this product
  revalidateTag('products');        // revalidate all products
}
```

Use `revalidateTag` for granular control. Use `revalidatePath` when you want to revalidate entire routes.

---

## 43. What is `next/headers` and how does it relate to caching?

Using dynamic functions from `next/headers` (`cookies()`, `headers()`) opts a route out of static rendering.

```tsx
// This page CANNOT be statically rendered
import { cookies } from 'next/headers';

async function Page() {
  const theme = cookies().get('theme')?.value;
  return <div className={theme}>Content</div>;
}

// This page CAN be statically rendered
async function Page() {
  const data = await fetch('https://api.example.com/posts');
  return <PostList posts={await data.json()} />;
}

// Partial dynamic rendering with Suspense
async function Page() {
  return (
    <div>
      <StaticContent />
      <Suspense fallback={<Loading />}>
        <DynamicContent /> {/* this part uses cookies() */}
      </Suspense>
    </div>
  );
}
```

---

## 44. What is the deployment strategy for Next.js?

| Platform | Features |
|---|---|
| **Vercel** | Zero-config, edge functions, analytics, preview deployments |
| **Docker** | Self-hosted, `output: 'standalone'` mode |
| **Static Export** | Any static hosting (S3, Cloudflare Pages, GitHub Pages) |
| **Node.js Server** | Self-hosted with `next start` |
| **AWS (Amplify/Lambda)** | Serverless deployment |

```dockerfile
# Docker deployment with standalone output
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

EXPOSE 3000
CMD ["node", "server.js"]
```

```js
// next.config.js for Docker
module.exports = {
  output: 'standalone',
};
```

---

## 45. What is testing in Next.js?

```tsx
// Component testing with React Testing Library
import { render, screen } from '@testing-library/react';
import Page from '@/app/page';

test('renders home page', () => {
  render(<Page />);
  expect(screen.getByText('Welcome')).toBeInTheDocument();
});

// Testing Server Components
import { render } from '@testing-library/react';

// Server Components are async — need to await
test('renders server component', async () => {
  const Component = await Page({ params: { id: '1' } });
  render(Component);
  expect(screen.getByText('User 1')).toBeInTheDocument();
});

// API Route testing
import { GET } from '@/app/api/users/route';
import { NextRequest } from 'next/server';

test('GET /api/users returns users', async () => {
  const request = new NextRequest('http://localhost/api/users');
  const response = await GET(request);
  const data = await response.json();

  expect(response.status).toBe(200);
  expect(data).toHaveLength(3);
});

// E2E with Playwright
import { test, expect } from '@playwright/test';

test('navigation flow', async ({ page }) => {
  await page.goto('/');
  await page.click('text=Sign In');
  await page.fill('input[name="email"]', 'test@example.com');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```

---

## 46. What are common Next.js project structures?

```
src/
├── app/                      # App Router routes
│   ├── (auth)/               # Auth route group
│   │   ├── login/
│   │   └── register/
│   ├── (main)/               # Main app route group
│   │   ├── dashboard/
│   │   └── settings/
│   ├── api/                  # Route Handlers
│   ├── layout.tsx
│   └── page.tsx
├── components/               # Shared components
│   ├── ui/                   # Generic UI components
│   └── features/             # Feature-specific components
├── lib/                      # Utility functions, configs
│   ├── db.ts                 # Database client
│   ├── auth.ts               # Auth utilities
│   └── utils.ts
├── actions/                  # Server Actions
├── hooks/                    # Custom React hooks
├── types/                    # TypeScript types
├── styles/                   # Global styles
└── middleware.ts             # Middleware
```

---

## 47. What is the `useOptimistic` hook?

`useOptimistic` provides optimistic UI updates while Server Actions are pending.

```tsx
'use client';

import { useOptimistic } from 'react';
import { addTodo } from '@/app/actions';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: string) => [
      ...state,
      { id: `temp-${Date.now()}`, title: newTodo, pending: true },
    ],
  );

  async function handleSubmit(formData: FormData) {
    const title = formData.get('title') as string;
    addOptimisticTodo(title);    // immediately update UI
    await addTodo(formData);      // server action runs in background
  }

  return (
    <div>
      <form action={handleSubmit}>
        <input name="title" required />
        <button type="submit">Add</button>
      </form>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
            {todo.title}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 48. What are common caching mistakes in Next.js?

1. **Not understanding default caching** — `fetch` is cached by default in Server Components
2. **Using `cache: 'no-store'` everywhere** — defeats the purpose of Next.js optimizations
3. **Forgetting to revalidate** — stale data after mutations
4. **Not using `revalidateTag`** — using `revalidatePath('/')` instead of targeted invalidation
5. **Mixing dynamic functions with static pages** — `cookies()` or `headers()` opts out of static rendering
6. **Not understanding Router Cache** — client-side cache persists across navigations
7. **Forgetting `router.refresh()`** — needed to clear client-side Router Cache after mutations
8. **Over-revalidating** — revalidating everything instead of specific data

---

## 49. What are common Next.js anti-patterns?

1. **Using `'use client'` on every component** — defeats Server Components benefits
2. **Fetching data in Client Components when Server Components work** — unnecessary client-side JS
3. **Not using `loading.tsx`** — blank screens while data loads
4. **Sequential data fetching** — use `Promise.all` for independent requests
5. **Not using `next/image`** — missing out on automatic optimization
6. **Placing all logic in API routes** — Server Components and Server Actions can access data directly
7. **Not using route groups** — messy layout structure
8. **Importing Server Components in Client Components** — breaks the component boundary
9. **Not configuring `remotePatterns` for images** — runtime errors for external images
10. **Using Pages Router patterns in App Router** — `getServerSideProps` in App Router code

---

## 50. What are the key differences between Next.js versions?

| Version | Key Features |
|---|---|
| **Next.js 12** | Middleware, Rust compiler (SWC), React 18 support |
| **Next.js 13** | App Router (beta), Server Components, Turbopack (alpha) |
| **Next.js 14** | Server Actions (stable), Turbopack (dev stable), Partial Prerendering (preview) |
| **Next.js 15** | React 19 support, async request APIs, `unstable_after`, enhanced caching |

Key trends:
- Shift from Pages Router to App Router
- Server Components as the default
- Server Actions replacing API routes for mutations
- Edge computing for middleware and lightweight handlers
- Progressive enhancement with streaming and Suspense
- Built-in SEO tools (metadata API, `robots.ts`, `sitemap.ts`)
