# `react-router-dom`

## 6.12.1

### Patch Changes

- Adjust feature detection of `React.startTransition` to fix webpack + react 17 compilation error ([#10569](https://github.com/remix-run/react-router/pull/10569))
- Updated dependencies:
  - `react-router@6.12.1`

## 6.12.0

### Minor Changes

- Wrap internal router state updates with `React.startTransition` if it exists ([#10438](https://github.com/remix-run/react-router/pull/10438))

### Patch Changes

- Re-throw `DOMException` (`DataCloneError`) when attempting to perform a `PUSH` navigation with non-serializable state. ([#10427](https://github.com/remix-run/react-router/pull/10427))
- Updated dependencies:
  - `@remix-run/router@1.6.3`
  - `react-router@6.12.0`

## 6.11.2

### Patch Changes

- Export `SetURLSearchParams` type ([#10444](https://github.com/remix-run/react-router/pull/10444))
- Updated dependencies:
  - `react-router@6.11.2`
  - `@remix-run/router@1.6.2`

## 6.11.1

### Patch Changes

- Updated dependencies:
  - `react-router@6.11.1`
  - `@remix-run/router@1.6.1`

## 6.11.0

### Minor Changes

- Enable `basename` support in `useFetcher` ([#10336](https://github.com/remix-run/react-router/pull/10336))
  - If you were previously working around this issue by manually prepending the `basename` then you will need to remove the manually prepended `basename` from your `fetcher` calls (`fetcher.load('/basename/route') -> fetcher.load('/route')`)

### Patch Changes

- Fix inadvertent re-renders when using `Component` instead of `element` on a route definition ([#10287](https://github.com/remix-run/react-router/pull/10287))
- Fail gracefully on `<Link to="//">` and other invalid URL values ([#10367](https://github.com/remix-run/react-router/pull/10367))
- Switched from `useSyncExternalStore` to `useState` for internal `@remix-run/router` router state syncing in `<RouterProvider>`. We found some [subtle bugs](https://codesandbox.io/s/use-sync-external-store-loop-9g7b81) where router state updates got propagated _before_ other normal `useState` updates, which could lead to footguns in `useEffect` calls. ([#10377](https://github.com/remix-run/react-router/pull/10377), [#10409](https://github.com/remix-run/react-router/pull/10409))
- Add static prop to `StaticRouterProvider`'s internal `Router` component ([#10401](https://github.com/remix-run/react-router/pull/10401))
- When using a `RouterProvider`, `useNavigate`/`useSubmit`/`fetcher.submit` are now stable across location changes, since we can handle relative routing via the `@remix-run/router` instance and get rid of our dependence on `useLocation()`. When using `BrowserRouter`, these hooks remain unstable across location changes because they still rely on `useLocation()`. ([#10336](https://github.com/remix-run/react-router/pull/10336))
- Updated dependencies:
  - `react-router@6.11.0`
  - `@remix-run/router@1.6.0`

## 6.10.0

### Minor Changes

- Added support for [**Future Flags**](https://reactrouter.com/en/main/guides/api-development-strategy) in React Router. The first flag being introduced is `future.v7_normalizeFormMethod` which will normalize the exposed `useNavigation()/useFetcher()` `formMethod` fields as uppercase HTTP methods to align with the `fetch()` behavior. ([#10207](https://github.com/remix-run/react-router/pull/10207))

  - When `future.v7_normalizeFormMethod === false` (default v6 behavior),
    - `useNavigation().formMethod` is lowercase
    - `useFetcher().formMethod` is lowercase
  - When `future.v7_normalizeFormMethod === true`:
    - `useNavigation().formMethod` is uppercase
    - `useFetcher().formMethod` is uppercase

### Patch Changes

- Fix `createStaticHandler` to also check for `ErrorBoundary` on routes in addition to `errorElement` ([#10190](https://github.com/remix-run/react-router/pull/10190))
- Updated dependencies:
  - `@remix-run/router@1.5.0`
  - `react-router@6.10.0`

## 6.9.0

### Minor Changes

- React Router now supports an alternative way to define your route `element` and `errorElement` fields as React Components instead of React Elements. You can instead pass a React Component to the new `Component` and `ErrorBoundary` fields if you choose. There is no functional difference between the two, so use whichever approach you prefer 😀. You shouldn't be defining both, but if you do `Component`/`ErrorBoundary` will "win". ([#10045](https://github.com/remix-run/react-router/pull/10045))

  **Example JSON Syntax**

  ```jsx
  // Both of these work the same:
  const elementRoutes = [{
    path: '/',
    element: <Home />,
    errorElement: <HomeError />,
  }]

  const componentRoutes = [{
    path: '/',
    Component: Home,
    ErrorBoundary: HomeError,
  }]

  function Home() { ... }
  function HomeError() { ... }
  ```

  **Example JSX Syntax**

  ```jsx
  // Both of these work the same:
  const elementRoutes = createRoutesFromElements(
    <Route path='/' element={<Home />} errorElement={<HomeError /> } />
  );

  const componentRoutes = createRoutesFromElements(
    <Route path='/' Component={Home} ErrorBoundary={HomeError} />
  );

  function Home() { ... }
  function HomeError() { ... }
  ```

- **Introducing Lazy Route Modules!** ([#10045](https://github.com/remix-run/react-router/pull/10045))

  In order to keep your application bundles small and support code-splitting of your routes, we've introduced a new `lazy()` route property. This is an async function that resolves the non-route-matching portions of your route definition (`loader`, `action`, `element`/`Component`, `errorElement`/`ErrorBoundary`, `shouldRevalidate`, `handle`).

  Lazy routes are resolved on initial load and during the `loading` or `submitting` phase of a navigation or fetcher call. You cannot lazily define route-matching properties (`path`, `index`, `children`) since we only execute your lazy route functions after we've matched known routes.

  Your `lazy` functions will typically return the result of a dynamic import.

  ```jsx
  // In this example, we assume most folks land on the homepage so we include that
  // in our critical-path bundle, but then we lazily load modules for /a and /b so
  // they don't load until the user navigates to those routes
  let routes = createRoutesFromElements(
    <Route path="/" element={<Layout />}>
      <Route index element={<Home />} />
      <Route path="a" lazy={() => import("./a")} />
      <Route path="b" lazy={() => import("./b")} />
    </Route>
  );
  ```

  Then in your lazy route modules, export the properties you want defined for the route:

  ```jsx
  export async function loader({ request }) {
    let data = await fetchData(request);
    return json(data);
  }

  // Export a `Component` directly instead of needing to create a React Element from it
  export function Component() {
    let data = useLoaderData();

    return (
      <>
        <h1>You made it!</h1>
        <p>{data}</p>
      </>
    );
  }

  // Export an `ErrorBoundary` directly instead of needing to create a React Element from it
  export function ErrorBoundary() {
    let error = useRouteError();
    return isRouteErrorResponse(error) ? (
      <h1>
        {error.status} {error.statusText}
      </h1>
    ) : (
      <h1>{error.message || error}</h1>
    );
  }
  ```

  An example of this in action can be found in the [`examples/lazy-loading-router-provider`](https://github.com/remix-run/react-router/tree/main/examples/lazy-loading-router-provider) directory of the repository.

  🙌 Huge thanks to @rossipedia for the [Initial Proposal](https://github.com/remix-run/react-router/discussions/9826) and [POC Implementation](https://github.com/remix-run/react-router/pull/9830).

- Updated dependencies:
  - `react-router@6.9.0`
  - `@remix-run/router@1.4.0`

## 6.8.2

### Patch Changes

- Treat same-origin absolute URLs in `<Link to>` as external if they are outside of the router `basename` ([#10135](https://github.com/remix-run/react-router/pull/10135))
- Fix `useBlocker` to return `IDLE_BLOCKER` during SSR ([#10046](https://github.com/remix-run/react-router/pull/10046))
- Fix SSR of absolute `<Link to>` urls ([#10112](https://github.com/remix-run/react-router/pull/10112))
- Properly escape HTML characters in `StaticRouterProvider` serialized hydration data ([#10068](https://github.com/remix-run/react-router/pull/10068))
- Updated dependencies:
  - `@remix-run/router@1.3.3`
  - `react-router@6.8.2`

## 6.8.1

### Patch Changes

- Improved absolute url detection in `Link` component (now also supports `mailto:` urls) ([#9994](https://github.com/remix-run/react-router/pull/9994))
- Fix partial object (search or hash only) pathnames losing current path value ([#10029](https://github.com/remix-run/react-router/pull/10029))
- Updated dependencies:
  - `react-router@6.8.1`
  - `@remix-run/router@1.3.2`

## 6.8.0

### Minor Changes

- Support absolute URLs in `<Link to>`. If the URL is for the current origin, it will still do a client-side navigation. If the URL is for a different origin then it will do a fresh document request for the new origin. ([#9900](https://github.com/remix-run/react-router/pull/9900))

  ```tsx
  <Link to="https://neworigin.com/some/path">    {/* Document request */}
  <Link to="//neworigin.com/some/path">          {/* Document request */}
  <Link to="https://www.currentorigin.com/path"> {/* Client-side navigation */}
  ```

### Patch Changes

- Fix bug with search params removal via `useSearchParams` ([#9969](https://github.com/remix-run/react-router/pull/9969))
- Respect `preventScrollReset` on `<fetcher.Form>` ([#9963](https://github.com/remix-run/react-router/pull/9963))
- Fix navigation for hash routers on manual URL changes ([#9980](https://github.com/remix-run/react-router/pull/9980))
- Use `pagehide` instead of `beforeunload` for `<ScrollRestoration>`. This has better cross-browser support, specifically on Mobile Safari. ([#9945](https://github.com/remix-run/react-router/pull/9945))
- Updated dependencies:
  - `@remix-run/router@1.3.1`
  - `react-router@6.8.0`

## 6.7.0

### Minor Changes

- Add `unstable_useBlocker` hook for blocking navigations within the app's location origin ([#9709](https://github.com/remix-run/react-router/pull/9709))
- Add `unstable_usePrompt` hook for blocking navigations within the app's location origin ([#9932](https://github.com/remix-run/react-router/pull/9932))
- Add `preventScrollReset` prop to `<Form>` ([#9886](https://github.com/remix-run/react-router/pull/9886))

### Patch Changes

- Added pass-through event listener options argument to `useBeforeUnload` ([#9709](https://github.com/remix-run/react-router/pull/9709))
- Streamline jsdom bug workaround in tests ([#9824](https://github.com/remix-run/react-router/pull/9824))
- Updated dependencies:
  - `@remix-run/router@1.3.0`
  - `react-router@6.7.0`

## 6.6.2

### Patch Changes

- Ensure `useId` consistency during SSR ([#9805](https://github.com/remix-run/react-router/pull/9805))
- Updated dependencies:
  - `react-router@6.6.2`

## 6.6.1

### Patch Changes

- Updated dependencies:
  - `@remix-run/router@1.2.1`
  - `react-router@6.6.1`

## 6.6.0

### Minor Changes

- Add `useBeforeUnload()` hook ([#9664](https://github.com/remix-run/react-router/pull/9664))
- Remove `unstable_` prefix from `createStaticHandler`/`createStaticRouter`/`StaticRouterProvider` ([#9738](https://github.com/remix-run/react-router/pull/9738))

### Patch Changes

- Proper hydration of `Error` objects from `StaticRouterProvider` ([#9664](https://github.com/remix-run/react-router/pull/9664))
- Support uppercase `<Form method>` and `useSubmit` method values ([#9664](https://github.com/remix-run/react-router/pull/9664))
- Skip initial scroll restoration for SSR apps with `hydrationData` ([#9664](https://github.com/remix-run/react-router/pull/9664))
- Fix `<button formmethod>` form submission overriddes ([#9664](https://github.com/remix-run/react-router/pull/9664))
- Updated dependencies:
  - `@remix-run/router@1.2.0`
  - `react-router@6.6.0`

## 6.5.0

### Patch Changes

- Updated dependencies:
  - `react-router@6.5.0`
  - `@remix-run/router@1.1.0`

## 6.4.5

### Patch Changes

- Updated dependencies:
  - `@remix-run/router@1.0.5`
  - `react-router@6.4.5`

## 6.4.4

### Patch Changes

- Fix issues with encoded characters in `NavLink` and descendant `<Routes>` ([#9589](https://github.com/remix-run/react-router/pull/9589), [#9647](https://github.com/remix-run/react-router/pull/9647))
- Properly serialize/deserialize `ErrorResponse` instances when using built-in hydration ([#9593](https://github.com/remix-run/react-router/pull/9593))
- Support `basename` in static data routers ([#9591](https://github.com/remix-run/react-router/pull/9591))
- Updated dependencies:
  - `@remix-run/router@1.0.4`
  - `react-router@6.4.4`

## 6.4.3

### Patch Changes

- Fix hrefs generated for `createHashRouter` ([#9409](https://github.com/remix-run/react-router/pull/9409))
- fix encoding/matching issues with special chars ([#9477](https://github.com/remix-run/react-router/pull/9477), [#9496](https://github.com/remix-run/react-router/pull/9496))
- Properly support `index` routes with a `path` in `useResolvedPath` ([#9486](https://github.com/remix-run/react-router/pull/9486))
- Respect `relative=path` prop on `NavLink` ([#9453](https://github.com/remix-run/react-router/pull/9453))
- Fix `NavLink` behavior for root urls ([#9497](https://github.com/remix-run/react-router/pull/9497))
- Updated dependencies:
  - `@remix-run/router@1.0.3`
  - `react-router@6.4.3`

## 6.4.2

### Patch Changes

- Respect `basename` in `useFormAction` ([#9352](https://github.com/remix-run/react-router/pull/9352))
- Enhance console error messages for invalid usage of data router hooks ([#9311](https://github.com/remix-run/react-router/pull/9311))
- If an index route has children, it will result in a runtime error. We have strengthened our `RouteObject`/`RouteProps` types to surface the error in TypeScript. ([#9366](https://github.com/remix-run/react-router/pull/9366))
- Updated dependencies:
  - `react-router@6.4.2`
  - `@remix-run/router@1.0.2`

## 6.4.1

### Patch Changes

- Updated dependencies:
  - `react-router@6.4.1`
  - `@remix-run/router@1.0.1`

## 6.4.0

Whoa this is a big one! `6.4.0` brings all the data loading and mutation APIs over from Remix. Here's a quick high level overview, but it's recommended you go check out the [docs][rr-docs], especially the [feature overview][rr-feature-overview] and the [tutorial][rr-tutorial].

**New APIs**

- Create your router with `createMemoryRouter`/`createBrowserRouter`/`createHashRouter`
- Render your router with `<RouterProvider>`
- Load data with a Route `loader` and mutate with a Route `action`
- Handle errors with Route `errorElement`
- Submit data with the new `<Form>` component
- Perform in-page data loads and mutations with `useFetcher()`
- Defer non-critical data with `defer` and `Await`
- Manage scroll position with `<ScrollRestoration>`

**New Features**

- Perform path-relative navigations with `<Link relative="path">` (#9160)

**Bug Fixes**

- Path resolution is now trailing slash agnostic (#8861)
- `useLocation` returns the scoped location inside a `<Routes location>` component (#9094)
- respect the `<Link replace>` prop if it is defined (#8779)

**Updated Dependencies**

- `react-router@6.4.0`

[rr-docs]: https://reactrouter.com
[rr-feature-overview]: https://reactrouter.com/start/overview
[rr-tutorial]: https://reactrouter.com/start/tutorial
