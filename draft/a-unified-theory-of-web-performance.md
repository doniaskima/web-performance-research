# A Unified Theory of Web Performance

- Page Load
- Post-Load interactions

## Page Load

### How browsers work

1. Retrieve and parse HTML to build the DOM
2. Load External Resources: JS and CSS
   - Parse CSS and build CSSOM
   - Execute JavaScript code
3. Render the page

Some things to remember:

- HTML + Client rendering: everything is blocked for users, meaning users can't interact with the page.
- FCP: First Contentful Paint — First content after the white screen
- TTI: Time To Interactive — The point at which layout has stabilized, key webfonts are visible, and the main thread is available enough to handle user input

The application loading time is proportional to the number and the size of the downloaded files, when the application is loaded.

### Strategies

The cost of JavaScript is not only the time it takes to load your bundle. The time to parse and execute your JavaScript is just as crucial.

- Removing unused code
- Compressing the code
- Compressing images
- Compressing fonts
- Route/code splitting
- Measuring performance
- Priorities: Load the core experience immediately, then enhancements, and then the extras.

### Metrics & Measuring

- Not all sites and visitors are the same: Web performance is a distribution
  - A static blog may need no JavaScript whatsoever whereas a graphics editor uses quite a lot of it.
  - A 4G visitor may deal with only 50ms of latency and a 2G user could suffer through an entire second per round trip.
- When thinking about distribution:
  - Mobile connection: 2G, 3G, 4G, 5G
  - Country/location
  - Phone device: iPhone 5, Samsung, etc
  - Operating System
  - Browser
- Not all metrics are equally important
  - Common metrics: TTI, FID, LCP, TBT, CLS, Speed Index, custom metrics
  - For Netflix TV UI, key input responsiveness, memory usage and TTI are more critical, and for Wikipedia, first/last visual changes and CPU time spent metrics are more important.
- Measurements
  - theory
    - 0.1 second is about the limit for having the user feel that the system is reacting instantaneously, meaning that no special feedback is necessary except to display the result.
    - 1.0 second is about the limit for the user's flow of thought to stay uninterrupted, even though the user will notice the delay. Normally, no special feedback is necessary during delays of more than 0.1 but less than 1.0 second, but the user does lose the feeling of operating directly on the data.
    - 10 seconds is about the limit for keeping the user's attention focused on the dialogue. For longer delays, users will want to perform other tasks while waiting for the computer to finish, so they should be given feedback indicating when the computer expects to be done. Feedback during the delay is especially important if the response time is likely to be highly variable, since users will then not know what to expect.
  - 100-millisecond response time, 60 fps: any longer than 100ms, the user perceives the app as laggy
  - FID < 100ms, LCP < 2.5s, TTI < 5s on 3G, Critical file size budget < 170KB (gzipped)
  - a budget of 170KB JavaScript gzipped already would take up to 1s to parse and compile on a mid-range phone.
- adaptive media
  - offline: placeholder text
  - 2G - low resolution image: ~30kb
  - 3G - high resolution image: ~180kb
  - 4G - HD video: ~1.5mb
- code splitting
  - page/route-based code-splitting
  - above the fold based code splitting
  - code split not visible components: modal/dialog, tab content
  - in addition to code-splitting via dynamic imports, [we could] also use code-splitting at the package level, where each imported node modules get put into a chunk based on its package’s name.
- Optimizing JavaScript for the main thread
  - Using Web Workers
    - Web Worker can’t manipulate the DOM (doesn't have acces to the dom, document, window, parent)
    - can use three different workers: shared workers, service workers, and dedicated workers
    - can be used to prefetch data and improve data loading time

### Core Web Vitals

- Largest Contentful Paint (LCP) < 2.5 sec.
  - The main reason for a low LCP score is usually images.
  - maximum theoretical image size is only around 144KB.
  - responsive images and preloading critical images early matter
- First Input Delay (FID) < 100ms.
  - Measures the responsiveness of the UI
  - The goal is to stay within 50–100ms for every interaction
  - Minimize the work on the main thread
  - Important to find Long Tasks: blocks the main thread for >50ms
    - code-split a bundle into multiple chunks
    - reduce JavaScript execution time
    - optimize data-fetching
    - defer script execution of third-parties
    - move JavaScript to the background thread with Web workers
    - use progressive hydration to reduce rehydration costs in SPAs.
- Cumulative Layout Shift (CLS) < 0.1.
  - Measures visual stability

## Post-load interactions

WIP

## Rendering Architectures

- server side rendering (SSR)
  - generate the page on runtime
  - FP (First Paint) & FCP (First Contentful Paint) are fast: rendering on the server to avoid sending lots of JavaScript to the client
  - Reducing the sie of JavaScript sent to the client, it makes TTI (Time to Interactive) faster as the browser needs to parse, evaluate, and execute less JavaScript
- SSR with hydration: rendering on the server (SSR) with client side rendering (CSR)
  - The primary downside of SSR with rehydration is that it can have a significant negative impact on Time To Interactive
  - Hydration: SSR’d pages often look deceptively loaded and interactive, but can’t actually respond to input until the client-side JS is executed and event handlers have been attached.
    - parsing and execution costs a lot in hydration
- static rendering
  - render HTML at build time
  - no ssr overhead (what are the SSR overhead here?)
  - only for static content
  - fast First Paint, First Contentful Paint and Time To Interactive
- pre-rendering
  - render HTML at build time
  - no ssr overhead (what are the SSR overhead here?)
  - only for static content
  - fast First Paint, First Contentful Paint and Time To Interactive
- client side rendering (CSR)
- streaming SSR
  - allows you to send HTML in chunks that the browser can progressively render as it's received.
- progressive hydration
  - component-level approach
  - entire page rendered server-side
  - pieces of the page are booted-up in order of priority
  - attach some event handlers to the DOM
  - make the page interactive faster
    - it doesn't need to hydrate all the pieces/components in the page
    - using the profiler: you can the cost of hydration (`hydrate`) and the difference between progressive hydration and no use of progressive hydration
  - partial hydration is an extension of the progressive hydration

| —         | SSR                                                             | Static SSR                                                      | SSR with Hydration                                           | CSR with Prerendering                            | CSR                                         |
| --------- | --------------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------- |
| Overview  | The client requests pages and the server responds with the HTML | Page is prerendered to static HTML at build time. JS is removed | The server prerenders pages. The app is booted on the client | Page is prerendered to static HTML at build time | Rendering and booting is done on the client |
| Rendering | Dynamic HTML                                                    | Static HTML                                                     | Dynamic HTML + JS/DOM                                        | Partial Static HTML, then JS/DOM                 | JS/DOM                                      |
| Pros      | Fast FCP, TTI                                                   | Fast TTFB, FCP, and TTI                                         | Flexible                                                     | Fast TTFB, flexible                              | Fast TTFB, flexible                         |
| Cons      | Slow TTFB, Inflexible                                           | Inflexible, Hydration                                           | Slow TTFB, TTI                                               | TTI > FCP                                        | TTI >>> FCP                                 |

### How it works

Client request HTML to the server

- ssr: build the page on runtime
- pre-rendering: the page was built at build time -> the server respond almost immediately

### Other: draft

- People are paying more attention
- Lighthouse was a big push to make web perf more visible
- Metrics are more attached to UX
- Improvements in the browser vendors
- Improvements in the ecosystem: tooling, literacy
- Test, test, test. Measure, Measure, Measure.
  - JS Profiling API, LongTasks
- It's not a fixed solution. There's no framework
