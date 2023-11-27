# Caching in Next.js
- caching rendering work
- caching data requests
- different caching mechanisms and their purpose:

| Mechanism           | What                       | Where  | Purpose                                         | Duration                        |
| :------------------ | :------------------------- | :----- | :---------------------------------------------- | :------------------------------ |
| Request Memoization | Return values of functions | Server | Re-use data in a React Component tree           | Per-request lifecycle           |
| Data Cache          | Data                       | Server | Store data across user requests and deployments | Persistent (can be revalidated) |
| Full Route Cache    | HTML and RSC payload       | Server | Reduce rendering cost and improve performance   | Persistent (can be revalidated) |
| Router Cache        | RSC Payload                | Client | Reduce server requests on navigation            | User session or time-based      |

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fcaching-overview.png&w=3840&q=75&dpl=dpl_Fpx7kMYuAx67q69KZVM7W7w2kwd5)

## Request Memoization
- Request memoization is a React feature, not a Next.js feature
- Memoization only applies to the GET method in fetch requests.
- Memoization only applies to the React Component tree, this means:
    - It applies to fetch requests in generateMetadata, generateStaticParams, Layouts, Pages, and other Server Components.
    - It doesn't apply to fetch requests in Route Handlers as they are not a part of the React component tree.
- For cases where fetch is not suitable (e.g. some database clients, CMS clients, or GraphQL clients), you can use the React cache function to memoize functions.
- to opt out of memoization in `fetch` requests: pass an `AbortController` signal to the request.
```ts
const { signal } = new AbortController();
fetch(url, { signal })
```

## Data Cache
- In the browser: a request and the browser's HTTP cache
- in Next.js: a server-side request and the servers Data Cache.
- Whether the data is cached or uncached, the requests are always memoized to avoid making duplicate requests for the same data during **a React render pass**
- Differences between the Data Cache and Request Memoization:
  - Data Cache is persistent across incoming requests and deployments
  - memoization only lasts the lifetime of a request
  - With memoization, we reduce the number of duplicate requests in the same render pass
  - With the Data Cache, we reduce the number of requests made to our origin data source
- Revalidating:
```ts
// Revalidate at most every hour
fetch('https://...', { next: { revalidate: 3600 } })

// manually revalidate data and re-render the route segments below a specific path in a single operation
revalidatePath('/')

// Cache data with a tag
fetch(`https://...`, { next: { tags: ['a', 'b', 'c'] } })

// Revalidate entries with a specific tag
revalidateTag('a')
```
- Opting out:
```ts
// Opt out of caching for an individual `fetch` request
fetch(`https://...`, { cache: 'no-store' })

// Opt out of caching for all data requests in the route segment
export const dynamic = 'force-dynamic'
```

## Full Route Cache
1. React Rendering on the Server
   - The rendering work is split into chunks: by individual routes segments and Suspense boundaries
   - Each chunk is rendered in two steps:
     - React renders Server Components into a special data format, optimized for streaming, called the React Server Component Payload.
     - Next.js uses the React Server Component Payload and Client Component JavaScript instructions to render HTML on the server.
2. Next.js Caching on the Server (Full Route Cache)
   ![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Ffull-route-cache.png&w=3840&q=75&dpl=dpl_Fpx7kMYuAx67q69KZVM7W7w2kwd5)
3. React Hydration and Reconciliation on the Client
   1. The HTML is used to immediately show a fast non-interactive initial preview of the Client and Server Components.
   2. The React Server Components Payload is used to reconcile the Client and rendered Server Component trees, and update the DOM.
   3. The JavaScript instructions are used to hydrate Client Components and make the application interactive.
4. Next.js Caching on the Client (Router Cache)
5. Subsequent Navigations

- [Static and Dynamic Rendering](https://nextjs.org/docs/app/building-your-application/rendering/server-components#server-rendering-strategies)
![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fstatic-and-dynamic-routes.png&w=3840&q=75&dpl=dpl_Fpx7kMYuAx67q69KZVM7W7w2kwd5)

- Duration: persistent
- Invalidation
  - Revalidating Data
  - Redeploying
- Opting out
  - Using a Dynamic Function
  - Using the `dynamic = 'force-dynamic'` or `revalidate = 0` route segment config options
  - Opting out of the Data Cache

## Router Cache
- Prefetch Cache refers to the prefetched route segments
- Client-side Cache refers to the whole Router cache, which includes both visited and prefetched segments
- Router Cache: in-memory client-side cache
  - stores the React Server Component Payload
  - split by individual route segments
  - for the duration of a user session

- Difference between the Router Cache and Full Route Cache:
  - Router Cache:
    - stores the React Server Component Payload
    - in the browser
    - for the duration of a user session
    - applies to both statically and dynamically rendered routes
  - Full Route Cache:
    - persistently stores the React Server Component Payload and HTML
    - on the server
    - across multiple user requests
    - only caches statically rendered routes

- Router Cache Duration
  - it's cleared on page refresh
  - Automatic Invalidation Period
    - Dynamically Rendered: 30 seconds
    - Statically Rendered: 5 minutes
- Invalidation
  - In a Server Action
    - Revalidating data on-demand by path with (`revalidatePath`) or by cache tag with (`revalidateTag`)
    - Using `cookies.set` or `cookies.delete` invalidates the Router Cache to prevent routes that use cookies from becoming stale (e.g. authentication).
  - Calling `router.refresh` will invalidate the Router Cache and make a new request to the server for the current route.
- Opting out
  - not possible to opt out of the Router Cache
  - can opt out of prefetching by setting the `prefetch` prop of the `<Link>` component to false.
  - but Visited routes will still be cached

## Cache Interactions
- Data Cache and Full Route Cache
  - Revalidating or opting out of the Data Cache will invalidate the Full Route Cache
  - but Invalidating or opting out of the Full Route Cache **does not** affect the Data Cache
- Data Cache and Client-side Router cache
  - Revalidating the Data Cache in a Route Handler **will not** immediately invalidate the Router Cache as the Router Handler isn't tied to a specific route
  - To immediately invalidate the Data Cache and Router cache, you can use `revalidatePath` or `revalidateTag` in a Server Action.

## APIs
```ts
// Cached by default. `force-cache` is the default option and can be ommitted.
fetch(`https://...`, { cache: 'force-cache' })

// Opt out of caching
fetch(`https://...`, { cache: 'no-store' })

// Revalidate at most after 1 hour
fetch(`https://...`, { next: { revalidate: 3600 } })
```

- Dynamic Functions
  - `cookies`
  - `headers`
  - `useSearchParams`
  - `searchParams`

```ts
// disable caching at request time, only paths provided by `generateStaticParams` will be served, and other routes will 404 or match
export const dynamicParams = false
```

- React cache function
```ts
import { cache } from 'react'
import db from '@/lib/db'

export const getItem = cache(async (id: string) => {
  const item = await db.item.findUnique({ id })
  return item
})
```