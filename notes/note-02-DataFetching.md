# Data Fetching
## four ways you can fetch data
    1. On the server, with `fetch`
    2. On the server, with third-party libraries
    3. On the client, via a Route Handler
    4. On the client, with third-party libraries.

## Fetching Data on the Server with fetch
- Next.js extends the native `fetch` Web API to allow you to configure the caching and revalidating behavior for each fetch request on the server
- React extends `fetch` to automatically memoize fetch requests while rendering a React component tree.
- You can use `fetch` with `async`/`await` in
  -  Server Components
  -  Route Handlers
  -  Server Actions

### Caching Data
`fetch` requests that use the `POST` method are also automatically cached. Unless it's inside a `Route Handler` that uses the POST method, then it will not be cached.

### Revalidating Data
purging the Data Cache and re-fetching the latest data.  
Cached data can be revalidated in two ways:
- Time-based revalidation
```ts
fetch('https://...', { next: { revalidate: 3600 } })
// or
export const revalidate = 3600 // layout.js | page.js
```
- [On-demand revalidation](https://nextjs.org/docs/app/building-your-application/caching#on-demand-revalidation)

> Error handling and revalidation: If an error is thrown while attempting to revalidate data, the last successfully generated data will continue to be served from the cache. On the next subsequent request, Next.js will retry revalidating the data.

### [Opting out of Data Caching](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#opting-out-of-data-caching)
fetch requests are not cached if:

- The `cache: 'no-store'` is added to `fetch` requests.
- The `revalidate: 0` option is added to individual `fetch` requests.
- The `fetch` request is inside a Router Handler that uses the `POST` method.
- The `fetch` request comes after the usage of `headers` or `cookies`.
- The `const dynamic = 'force-dynamic'` route segment option is used.
- The `fetchCache` route segment option is configured to skip cache by default.
- The `fetch` request uses `Authorization` or `Cookie` headers and there's an uncached request above it in the component tree.

> we recommend configuring the caching behavior of each fetch request individually.

## Fetching data on the Server with third-party libraries
- The React `cache` function is used to memoize data requests.

## Fetching Data on the Client with Route Handlers
If you need to fetch data in a client component, you can call a Route Handler from the client. Route Handlers execute on the server and return the data to the client. This is useful when you don't want to expose sensitive information to the client, such as API tokens.
> you can fetch the data directly inside the Server Component.

## Fetching Data on the Client with third-party libraries

## Data Fetching Patterns
1. Whenever possible, we recommend fetching data on the server
2. Fetching Data Where It's Needed
3. Streaming
4. Parallel and Sequential Data Fetching
5. Preloading Data
6. Using React `cache`,`server-only`, and the Preload Pattern

## Forms and Mutations
### How Server Actions Work
- Server Actions can be defined in Server Components
- Server Actions can be called from Client Components.
- Forms calling Server Actions from Server Components can function without JavaScript.
- Forms calling Server Actions from Client Components will queue submissions if JavaScript isn't loaded yet, prioritizing client hydration.

### Revalidating Cached Data
```ts
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'

// invalidate an entire route segment
revalidatePath('/')

// invalidate a specific data fetch with a cache tag
revalidateTag('posts')
```

### Redirecting
### Form Validation
- basic HTML form validation:
  - `required`
  - `type="email"`
- more advanced server-side validation
  - a schema validation library like [zod](https://zod.dev/)
```ts
import { z } from 'zod'
 
const schema = z.object({
  // ...
})
 
export default async function submit(formData: FormData) {
  const parsed = schema.parse({
    id: formData.get('id'),
  })
  // ...
}
```

### Displaying Loading State
`useFormStatus`, can only be used as a child of a form element using a Server Action

### Error Handling
Server Actions can also return `serializable objects` like: `return { message: 'Failed to create' }`,  
Then, from a Client Component,
```ts
'use client'
import { useFormState } from 'react-dom'

export function AddForm() {
    const [state] = useFormState();
     return (<form>
        ...
        {state?.message}
     </form>)
}
```

### Optimistic Updates
Use `useOptimistic` to optimistically update the UI before the Server Action finishes, rather than waiting for the response
