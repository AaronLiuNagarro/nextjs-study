# Why NextJs
https://react.dev/learn/start-a-new-react-project  
 The Next.js team has agreed to collaborate with React in researching, developing, integrating, and testing framework-agnostic bleeding-edge React features like React Server Components.

# Getting start
1. Resources
   1. Official document online: https://nextjs.org/docs
   2. Samples codes: https://github.com/vercel/next.js/tree/canary/examples
   3. Sample preview: https://nextjs-app-directory-beige.vercel.app/
2. Next.js:
   1. a React framework
   2. for building full-stack web applications
   3. use React Components to build user interfaces
   4. use Next.js for additional features and optimizations
3. Main features
   - Routing
   - Rendering
   - Data fetching
   - ...
# Routing
## Define routes
- Next.js uses **file-system** routing, which means the routes in your application are determined by how you structure your files.
  - [File conventions](https://nextjs.org/docs/app/building-your-application/routing#file-conventions)
  - only the contents returned by `page.js` or `route.js` are publicly addressable.(even though route structure is defined through folders, a route is not publicly accessible until a `page.js` or `route.js` file is added to a route segment)
  - Private folders: _folderName
  - You can create URL segments that start with an underscore by prefixing the folder name with %5F (the URL-encoded form of an underscore): `%5FfolderName`.
  - [Router groups by brackets: (folderName)](https://nextjs.org/docs/app/building-your-application/routing/route-groups)
  - [Parallel route Slots: @folder](https://nextjs.org/docs/app/building-your-application/routing/parallel-routes#convention)
  - [Intercepting Routes: (..)](https://nextjs.org/docs/app/building-your-application/routing/intercepting-routes)
  - [Module Path Aliases](https://nextjs.org/docs/app/building-your-application/configuring/absolute-imports-and-module-aliases#module-aliases)
  - choose a strategy that works for you and your team and be **consistent** across the project
- App Router introduced by Next.js v13, Pages Router is the original Next.js router
- The App Router takes priority over the Pages Router
- Components inside app are React Server Components by default

## Pages & Layouts
- A page is UI that is unique to a route
- A layout is UI that is shared between multiple pages, On navigation,
  - layouts preserve state
  - remain interactive
  - do not re-render
  - Layouts can also be nested
- Pages and Layouts are Server Components by default
- Passing data between a parent layout and its children is not possible. However, you can fetch the same data in a route more than once, and React will automatically dedupe the requests without affecting performance. ([Request Memoization](https://nextjs.org/docs/app/building-your-application/caching#request-memoization))
- `useSelectedLayoutSegment` only returns the segment one level down. To return all active segments, see `useSelectedLayoutSegments`
- Root layout
  - must contain html and body tags
  - [built-in SEO support](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
  - can not be set to a Client Component
- Nesting Layouts cannot contain html and body tags
- Templates create a new instance for each of their children on navigation, DOM elements are recreated, state is not preserved, and effects are re-synchronized

## Linking and Navigating
1. `<Link>`'s prefetching behavior (It's different for static and dynamic routes)
   - Static Routes: prefetch defaults to true. The entire route is prefetched and cached.
   - Dynamic Routes: prefetch default to automatic. Only the shared layout down until the first loading.js file is prefetched and cached for 30s. This reduces the cost of fetching an entire dynamic route, and it means you can show an instant loading state for better visual feedback to users.  
> You can disable prefetching by setting the prefetch prop to false.  
> Prefetching is not enabled in development, only in production.

> Hard navigation:  
This means the browser reloads the page and resets React state such as useState hooks in your app and browser state such as the user's scroll position or focused element.  
Soft navigation:  
However, in Next.js, the App Router uses soft navigation. This means React only renders the segments that have changed while preserving React and browser state, and there is no full page reload.

2. [Router Cache](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#caching-data) on navigation, the cache is reused as much as possible
3. Partial rendering means only the route segments that change on navigation re-render on the client, and any shared segments are preserved.

## Dynamic Routes
- `[folderName]`
- generateStaticParams: smart retrieval of data
- catch-all Segments: `[...folderName]`
- Optional Catch-all Segments: `[[...folderName]]`

## Loading UI and Streaming
- React Suspense
- Use the `loading.js` convention for route segments (layouts and pages) as Next.js optimizes this functionality.
- SSR
  1. First, all data for a given page is fetched on the server.
  2. The server then renders the HTML for the page.
  3. The HTML, CSS, and JavaScript for the page are sent to the client.
  4. A non-interactive user interface is shown using the generated HTML, and CSS.
  5. Finally, React hydrates the user interface to make it interactive.

- Streaming
  - allows you to break down the page's HTML into smaller chunks and progressively send those chunks from the server to the client.
  - SEO: Next.js will wait for data fetching inside `generateMetadata` to complete before streaming UI to the client. This guarantees the first part of a streamed response includes `<head>` tags
  - Status code is `200` first and we can deal with errors within the streamed content itself.Since the response headers have already been sent to the client, the status code of the response cannot be updated

## Error Handling
- `error.js` automatically creates a `React Error Boundary`
- Errors bubble up to the nearest parent error boundary
- An `error.js` boundary will not handle errors thrown in a `layout.js` or `template.js` component in the same segment because the error boundary is nested inside that layout's component.
- To handle errors within a specific layout or template, place an `error.js` file in the layouts parent segment.
- The root `app/error.js` boundary does not catch errors thrown in the root `app/layout.js` or `app/template.js` component. Use `app/global-error.js` instead.
- `app/global-error.js` must define its own `<html>` and `<body>` tags.

## Parallel Routes
- `@folder`
- The `children` prop is an implicit slot that does not need to be mapped to a folder. This means `app/page.js` is equivalent to `app/@children/page.js`.
- Unmatched Routes
  - `default.js`
  - On navigation, Next.js will render the slot's previously active state, even if it doesn't match the current URL.
  - On reload, Next.js will first try to render the unmatched slot's `default.js` file. If that's not available, a 404 gets rendered.

- Render modals
  - To ensure that the contents of the modal don't get rendered when it's not active, you can create a `default.js` file that returns `null`.
  - dismiss the modal by calling `router.back()` or by using a `Link` component
  - use a catch-all route to dismiss a modal when navigate elsewhere: `[...catchAll]`
  - `Catch-all` routes take precedence over `default.js`.

- Conditional Routes

## Intercepting Routes
- allows you to load a route from another part of your application within the current layout.
- It's useful when you want to display the content of a route without the user switching to a different context
- Convention
  - `(.)`: same level
  - `(..)`: one level above
  - `(..)(..)`: two levels above
  - `(...)` match segments from the root app directory

## Route Handlers
create custom request handlers

- only available inside the app directory
- do not need to use API Routes and Route Handlers together
- `app/api/route.ts`
- There cannot be a `route.js` file at the same route as `page.js`
- `NextRequest` and `NextResponse`
- cached by default for `GET` method
- can opt out of caching by
  - Using the `Request` object with the GET method
  - Using any of the other HTTP methods.
  - Using `Dynamic Functions` like cookies and headers
  - The [Segment Config Options](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#segment-config-options) manually specifies dynamic mode
- Revalidating Cached Data
  - `next.revalidate` option:
```ts
  const res = await fetch('https://data.mongodb-api.com/...', {
    next: { revalidate: 60 }, // Revalidate every 60 seconds
  })
```
  - `revalidate` segment config option: 
  ```ts
  export const revalidate = 60
  ```

### Dynamic Functions
- Cookies
```ts
import { cookies } from 'next/headers'

export async function GET(request: Request) {
  const cookieStore = cookies()
  const token = cookieStore.get('token')

  return new Response('Hello, Next.js!', {
    status: 200,
    headers: { 'Set-Cookie': `token=${token.value}` },
  })
}
```
or
```ts
import { type NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  const token = request.cookies.get('token')
}
```

- Headers
```ts
import { headers } from 'next/headers'
 
export async function GET(request: Request) {
  const headersList = headers()
  const referer = headersList.get('referer')
 
  return new Response('Hello, Next.js!', {
    status: 200,
    headers: { referer: referer },
  })
}
```
or
```ts
import { type NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  const requestHeaders = new Headers(request.headers)
}
```

### Redirects
```ts
import { redirect } from 'next/navigation'
 
export async function GET(request: Request) {
  redirect('https://nextjs.org/')
}
```

### [Streaming](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#streaming)

### runtime
'nodejs' is the default, set it to `edge`:
```ts
export const runtime = 'edge'
```
> The Next.js [Edge Runtime](https://nextjs.org/docs/pages/api-reference/edge) is based on standard Web APIs, is a subset of available Node.js APIs
- [Runtime Differences](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes)

|                            | Node   | Serverless | Edge             |
| :------------------------- | :----- | :--------- | :--------------- |
| Cold Boot                  | /      | Normal     | Low              |
| HTTP Streaming             | Yes    | Yes        | Yes              |
| IO                         | All    | All        | fetch            |
| Scalability                | /      | High       | Highest          |
| Security                   | Normal | High       | High             |
| Latency                    | Normal | Low        | Lowest           |
| npm Packages               | All    | All        | A smaller subset |
| Static Rendering           | Yes    | Yes        | No               |
| Dynamic Rendering          | Yes    | Yes        | Yes              |
| Data Revalidation w/ fetch | Yes    | Yes        | Yes              |

## Middleware
- allows you to run code before a request is completed
- runs before cached content and routes are matched
- will be invoked for every route in your project
### Matching Paths
- execution order
   1. `headers` from `next.config.js`
   2. `redirects` from `next.config.js`
   3. Middleware (`rewrites`, `redirects`, etc.)
   4. `beforeFiles` (`rewrites`) from `next.config.js`
   5. Filesystem routes (`public/`, `_next/static/`, `pages/`, `app/`, etc.)
   6. `afterFiles` (`rewrites`) from `next.config.js`
   7. Dynamic Routes (`/blog/[slug]`)
   8. `fallback` (`rewrites`) from `next.config.js`

- two ways to define which paths Middleware will run on:
  - Custom matcher config
    1. MUST start with `/`
    2. Can include named parameters: `/about/:path` matches `/about/a` and `/about/b` but not `/about/a/c`
    3. Can have modifiers on named parameters (starting with `:`): `/about/:path*` matches `/about/a/b/c` because `*` is zero or more. `?` is zero or one and `+` one or more
    4. Can use regular expression enclosed in parenthesis: `/about/(.*)` is the same as `/about/:path*`
  - Conditional statements
```ts
export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }
}
```

### NextResponse
allows you to:
- `redirect` the incoming request to a different URL
- `rewrite` the response by displaying a given URL
- Set request headers for API Routes, `getServerSideProps`, and `rewrite` destinations
- Set response cookies
- Set response headers

## Internationalization
- Routing can be internationalized by either the sub-path (`/fr/products`) or domain (`my-site.fr/products`)
- redirect the user based on the locale inside `Middleware`
- ensure all special files inside `app/` are nested under `app/[lang]`
```ts
// app/[lang]/page.js
// app/[lang]/layout.js

// You now have access to the current locale
// e.g. /en-US/products -> `lang` is "en-US"
export default async function Page({ params: { lang } }) {
  return ...
}
```
