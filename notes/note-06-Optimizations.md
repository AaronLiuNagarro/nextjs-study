# Optimizations
## Core Web Vitals
- LCP: [Largest Contentful Paint (LCP)](https://almanac.httparchive.org/en/2022/performance#largest-contentful-paint-lcp)
- FID: [First Input Delay](https://nextjs.org/learn-pages-router/seo/web-performance/fid)
- CLS: [Cumulative Layout Shift](https://nextjs.org/learn-pages-router/seo/web-performance/cls)
- [Others](https://nextjs.org/docs/app/building-your-application/optimizing/analytics#web-vitals)

## Built-in Components
- Images <-- `<img>`
- Link  <-- `<a>`
- Scripts  <-- `<script>`
## Metadata
- better SEO
- allows you to customize how your content is presented on social media
- helping you create a more engaging and consistent user experience across various platforms
- be used to define your application metadata for improved SEO and web shareability
- by adding `<head>` elements
- only supported in Server Components
- how to add it
  - Config-based Metadata
  - File-based Metadata
- Static Metadata
  ```ts
  import type { Metadata } from 'next'
  export const metadata: Metadata = {
    title: '...',
    description: '...',
  }

  export default function Page() {}
  ```
- Dynamic Metadata
  - by `generateMetadata` to `fetch` metadata
  - Next.js will wait for data fetching inside generateMetadata to complete before streaming UI to the client.
- File-based metadata
  - favicon.ico, apple-icon.jpg, and icon.jpg
  - opengraph-image.jpg and twitter-image.jpg
  - robots.txt
  - sitemap.xml
- Behavior
  - File-based metadata has the higher priority and will override any config-based metadata.
  - Default Fields
    - meta charset tag
    - meta viewport tag (can be overwrote)
  - Ordering: root -> segment
  - Merging
    - multiple segments in the same route are shallowly merged together
    - same metadata will be overwritten by the last segment to define them
    - good practice is to pull the same fields out into a separate variable

- Dynamic Image Generation: `ImageResponse`
  - to generate dynamic images using JSX and CSS
  - useful for creating social media images such as Open Graph images, Twitter cards, and more
  - uses the [Edge Runtime](https://nextjs.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes#edge-runtime), the default Node.js runtime will not work
  - integrates well with other Next.js APIs
  - supports common CSS properties ([full list of supported CSS properties](https://github.com/vercel/satori#css))( (e.g. `display: grid`) will not work)
  - Maximum bundle size of 500KB.
  - Only `ttf`, `otf`, and `woff` font formats are supported. To maximize the font parsing speed, `ttf` or `otf` are preferred over `woff`
  - Samples: [Vercel OG Playground](https://og-playground.vercel.app/)

- [JSON-LD](https://nextjs.org/docs/app/building-your-application/optimizing/metadata#json-ld)
  - a format for structured data that can be used by search engines to understand your content
  - [schema-dts](https://www.npmjs.com/package/schema-dts)

## Static Assets
- `public` folder under root directory
- be referenced by your code starting from the base URL (`/`).

## Images
  - Size Optimization
  - Visual Stability
  - Faster Page Loads
    - native browser lazy loading
    - optional blur-up placeholders
  - Asset Flexibility
  - Domains & [remotePatterns](https://nextjs.org/docs/app/api-reference/components/image#remotepatterns)
  - Priority

## Fonts
- built-in automatic self-hosting

## Script
- `next/script`, `<Script src="xx/xx.js" />`
- fetched when accessed by the user
- the script will only load once
- Loading Strategy
  - beforeInteractive
  - afterInteractive (default)
  - lazyOnload
  - worker (experimental)
- Executing Additional Code (for Client Component)
  - onLoad
  - onReady
  - onError

## Lazy Loading
- improve the initial loading performance
- defer loading of Client Components and imported libraries
- implementation
  - Dynamic Imports
  - `React.lazy()`
- applies to Client Components
- SSR by default
- disable SSR: 
  ```ts
  const ComponentC = dynamic(() => import('../components/C'), {
    ssr: false
  })
  ```
- load External libraries on demand with `import`
  ```ts
  'use client'
  // ...
  // Dynamically load fuse.js
  const Fuse = (await import('fuse.js')).default
  ```
- Adding a custom loading component
  ```ts
  import dynamic from 'next/dynamic'

  const WithCustomLoading = dynamic(
    () => import('../components/WithCustomLoading'),{
      loading: () => <p>Loading...</p>,
    }
  )

  export default function Page() {
    return (
      <div>
        {/* The loading component will be rendered while  <WithCustomLoading/> is loading */}
        <WithCustomLoading />
      </div>
    )
  }
  ```
- Importing Named Exports
  ```js
  // components/hello.js
  'use client'

  export function Hello() {
    return <p>Hello!</p>
  }

  // app/page.js
  import dynamic from 'next/dynamic'

  const ClientComponent = dynamic(() =>
    import('../components/hello').then((mod) => mod.Hello)
  )
  ```

## Analytics and Monitoring
- Next.js has built-in support for measuring and reporting performance metrics
- `useReportWebVitals` 
- [managed service](https://vercel.com/analytics?utm_source=next-site&utm_medium=docs&utm_campaign=next-website)
- [Web Vitals](https://web.dev/vitals/)
  ```ts
  'use client'

  import { useReportWebVitals } from 'next/web-vitals'

  export function WebVitals() {
    useReportWebVitals((metric) => {
      switch (metric.name) {
        case 'FCP': {
          // handle FCP results
        }
        case 'LCP': {
          // handle LCP results
        }
        // ...
      }
    })
  }
  ```
- [sending results to Google Analytics](https://github.com/GoogleChrome/web-vitals#send-the-results-to-google-analytics)

## OpenTelemetry
- it's experimental
- explicitly opt-in: `experimental.instrumentationHook = true;` in `next.config.js`
- [Official OpenTelemetry docs](https://opentelemetry.io/docs/)
- Next.js supports OpenTelemetry instrumentation out of the box, but don't support for edge or client side code