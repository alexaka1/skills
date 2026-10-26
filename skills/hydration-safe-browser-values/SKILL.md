---
name: hydration-safe-browser-values
description: Hydration-safe rendering of values only the browser knows (current time, timezone, locale, localStorage) in SSR or static React pages. Use when writing one, e.g. relative timestamps, local dates, stored preferences, or when one flickers after load or causes a hydration mismatch.
---

# Hydration-safe browser values

A server-rendered page paints what the server knew: the build time for a static page, and never the browser's timezone, locale, or localStorage. For a value that depends on any of these, the first paint is wrong and the value **flickers** once React hydrates. An inline `<script>` placed right after the element runs while the browser parses the HTML, so it paints the right value before anything is visible. React then has to start from that same value, or it reports a hydration mismatch.

## 1. Pick the layer

Take the first that fits:

- The value comes from the request (cookie, header, URL) and the page can render per request: resolve it on the server with `no-ui-flash`.
- Theme or dark mode: use the theme library's own pre-paint script (e.g. `next-themes`).
- Text the user typed before hydration: `hydration-safe-inputs`.
- Anything else time-dependent or browser-only: continue here.

## 2. Write the derivation once

Put the logic that turns inputs into the painted value in one module-scope arrow function. The script receives its source text, so the component and the script run the same code and cannot drift. Keep it self-contained: it may use its parameters and browser globals only.

```tsx
const secondHandRotation = (ms: number) => {
  const t = new Date(ms);
  return t.getSeconds() * 6 + t.getMilliseconds() * 0.006;
};
```

## 3. Render the script right after the element

```tsx
declare global {
  interface Window {
    __prehydration?: Record<string, number>;
  }
}

const paintSecondHand = (id: string, derive: (ms: number) => number) => {
  const el = document.getElementById(id);
  if (!el) return;
  const now = Date.now();
  (window.__prehydration ??= {})[id] = now;
  el.setAttribute("transform", `rotate(${derive(now)}, 50, 50)`);
};

function SecondHandScript({ id }: { id: string }) {
  const args = JSON.stringify(id).replace(/</g, "\\u003c");
  return (
    <script
      suppressHydrationWarning
      dangerouslySetInnerHTML={{
        __html: `(${paintSecondHand.toString()})(${args}, ${secondHandRotation.toString()})`,
      }}
    />
  );
}
```

- Both functions ship as source text, so an import or captured variable inside them becomes a `ReferenceError` in the page. Write them as arrow or `function` expressions; method shorthand does not stringify to a valid expression.
- Pass only ids and static constants as arguments, JSON-encoded with `<` escaped so no value can close the script tag.
- `suppressHydrationWarning` belongs on the `<script>` only: server and client bundles can stringify the functions differently, and only the server copy ever runs. On the painted element it would hide the bug this skill fixes.
- Render the script immediately after its element, in the same component. Next.js `instrumentation-client` bundles imports but runs after the whole document loads, which is too late for first paint.
- Under a CSP without `'unsafe-inline'`, the script needs a nonce (which makes the page render per request) or a hash of its exact text. Check the response headers.

## 4. Start React from the painted value

React's first client render must produce exactly what the script wrote. DOM patching alone still ends in a hydration mismatch; `vercel-react-best-practices`' `rendering-hydration-no-flicker` rule stops before this step.

- Inputs that hold still between script and hydration (localStorage, timezone, locale): read them again in a lazy `useState` initializer.
- Inputs that move (the current time): read what the script recorded, then delete it after mount so a client-side navigation back to the page starts fresh.

```tsx
function Clock() {
  const id = useId();
  const [now, setNow] = useState(() =>
    typeof window === "undefined" ? Date.now() : (window.__prehydration?.[id] ?? Date.now()),
  );

  useEffect(() => {
    delete window.__prehydration?.[id];
    const timer = setInterval(() => setNow(Date.now()), 50);
    return () => clearInterval(timer);
  }, [id]);

  return (
    <>
      <svg width="200" height="200" viewBox="0 0 100 100">
        <line id={id} x1="50" y1="50" x2="50" y2="10" stroke="red"
          transform={`rotate(${secondHandRotation(now)}, 50, 50)`} />
      </svg>
      <SecondHandScript id={id} />
    </>
  );
}
```

## 5. Verify

Done when all three hold:

- **Production build** (`next build && next start` or the framework's equivalent), framework JS bundles blocked via DevTools request blocking (e.g. `/_next/static/*`): the element shows the browser's value, not the server's.
- **Dev server**, bundles unblocked, hard reload: no visible jump and no hydration warning in the console.
- **Client-side navigation** away and back to the page: the value is current, and `window.__prehydration` no longer holds the id.
