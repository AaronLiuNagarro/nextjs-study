# Rendering
## Fundamentals
- Rendering Environments
: The Environments your application code can be executed in: the server and the client
- The Request-Response Lifecycle that's initiated when a user visits or interacts with your application.
- The Network Boundary that separates server and client code.
- your application code flows in one direction: from the server to the client.

### Request-Response Lifecycle
1. User Action: The user interacts with a web application. This could be clicking a link, submitting a form, or typing a URL directly into the browser's address bar.
2. HTTP Request: The client sends an HTTP request to the server that contains necessary information about what resources are being requested, what method is being used (e.g. `GET`, `POST`), and additional data if necessary.
3. Server: The server processes the request and responds with the appropriate resources. This process may take a couple of steps like routing, fetching data, etc.
4. HTTP Response: After processing the request, the server sends an HTTP response back to the client. This response contains a status code (which tells the client whether the request was successful or not) and requested resources (e.g. HTML, CSS, JavaScript, static assets, etc).
5. Client: The client parses the resources to render the user interface.
6. User Action: Once the user interface is rendered, the user can interact with it, and the whole process starts again.

### Network Boundary
- a conceptual line that separates the different environments
- the client and the server, or the server and the data store
- the client module graph and the server module graph
- `use client` and `use server`


## Server Components
### Benefits of Server Rendering
- Data Fetching
- Security
- Caching
- Bundle Sizes
- Initial Page Load and [First Contentful Paint (FCP)](https://web.dev/articles/fcp)
- Search Engine Optimization and Social Network Shareability
- Streaming

### How are Server Components rendered?
1. The rendering work is split into chunks
2. Each chunk is rendered in two steps
   1. React renders Server Components into a special data format called the React Server Component Payload (RSC Payload).
   2. Next.js uses the RSC Payload and Client Component JavaScript instructions to render HTML on the server.
3. On the client
   1. The HTML is used to immediately show a fast non-interactive preview of the route - this is for the initial page load only.
   2. The React Server Components Payload is used to reconcile the Client and Server Component trees, and update the DOM.
   3. The JavaScript instructions are used to hydrate Client Components and make the application interactive.

### Server Rendering Strategies
- Static Rendering (Default)
  - routes are rendered at build time, or in the background after data revalidation
  - The result is cached and can be pushed to a CDN
  - It's useful when a route has data that is not personalized to the user and can be known at build time, such as a static blog post or a product page.
  - This optimization allows you to share the result of the rendering work between users and server requests.
- Dynamic Rendering
  - routes are rendered for each user at request time.
  - is useful when a route has data that is personalized to the user
  - is useful when a router has information that can only be known at request time, such as cookies or the URL's search params
  - During rendering, if a dynamic function or uncached data request is discovered, Next.js will switch to dynamically rendering the whole route
  - Dynamic Functions: rely on information that can only be known at request time
    - `cookies()` and `headers()`
    - `useSearchParams()` (IMPORTANT: use it in a `<Suspense/>` boundary, [Example](https://nextjs.org/docs/app/api-reference/functions/use-search-params#static-rendering))
    - `searchParams`


| Dynamic Functions | Data       | Route                |
| :---------------- | :--------- | :------------------- |
| No                | Cached     | Statically Rendered  |
| Yes               | Cached     | Dynamically Rendered |
| No                | Not Cached | Dynamically Rendered |
| Yes               | Not Cached | Dynamically Rendered |

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fstatic-and-dynamic-routes.png&w=3840&q=75&dpl=dpl_Fpx7kMYuAx67q69KZVM7W7w2kwd5)

- Streaming
  - enables you to progressively render UI from the server
  - allows the user to see parts of the page immediately, before the entire content has finished rendering
  - is built into the Next.js App Router by default

## Client Components
- `use client`, all other modules imported into it, including child components, are considered part of the client bundle
- You can keep code on the server even though it's theoretically nested inside Client Components by interleaving Client and Server Components and Server Actions.

## Composition Patterns
[a quick summary of the different use cases for Server and Client Components](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns#when-to-use-server-and-client-components)

### Server Component Patterns
- Sharing data between components
  - (React Context is not available on the server)
  - use `fetch` or React's `cache` function to fetch the same data in the components that need it, because React automatically [memoize](https://nextjs.org/docs/app/building-your-application/caching#request-memoization) data requests
- Keeping Server-only Code out of the Client Environment
  - an environment variable not prefixed with `NEXT_PUBLIC` would be replaced with an empty string to client environment
  - `server-only` package could give other developers a build-time error if they ever accidentally import one of these modules into a Client Component
  - `client-only` marks modules that contain client-only code(eg. code that accesses the window object)
- Using Third-party Packages and Providers
  - wrap third-party components that rely on client-only features in your own Client Components
    ```ts
      'use client'

      import { Carousel } from 'acme-carousel'

      export default Carousel
    ```
  - you can only create your context and render its provider inside of a Client Component
  - You should render providers as deep as possible in the tree. This makes it easier for Next.js to optimize the static parts of your Server Components
  - an example of how to configure esbuild to include the `"use client"` directive: [React Wrap Balancer](https://github.com/shuding/react-wrap-balancer/blob/main/tsup.config.ts#L10-L13)

### Client Components Patterns
- Moving Client Components Down the Tree
- Passing props from Server to Client Components (Serialization)
  - need to be serializable by React
  - if not, fetch data on the client with a third party library or on the server via a Route Handler.
- Interleaving Server and Client Components
  - When a new request is made to the server, all Server Components are rendered first, including those nested inside Client Components.
  - Since Client Components are rendered after Server Components, you cannot import a Server Component into a Client Component module (since it would require a new request back to the server). Instead, you can pass a Server Component as props to a Client Component
  - lift content up / move state down (https://juejin.cn/post/7169464362904748062)
  - lift state down

