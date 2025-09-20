# Patterns & Playbook (living doc)

> Goal: think faster, break problems cleanly, avoid repeating the same mistakes.

## Decomposition template

```
SPEC:
- Inputs: …
- Outputs: …
- Constraints/edge cases: …

PLAN (3–5 steps):
1) …
2) …
3) …

TEST (quick):
- given … expect …

DONE WHEN:
- [ ] condition A
- [ ] condition B
```

---

## Language patterns

### Closures (capture + factory)
```js
function makeCounter(start = 0) {
  let n = start;
  return { next: () => ++n, current: () => n };
}
const c = makeCounter();
c.next(); // 1
```
**Use when:** you need private state without classes.  
**Pitfall:** capturing loop variables incorrectly—prefer `for (const x of …)`.

### `this` vs arrow
- Arrow functions **don’t** rebind `this`. Great for callbacks in methods.
- Use `function` when you want dynamic `this` (e.g., event handlers on objects).

### Immutability snippet
```js
const next = prev => ({ ...prev, updatedAt: Date.now() });
```

---

## DOM & Events

### Delegation pattern
```js
document.body.addEventListener('click', (e) => {
  const btn = e.target.closest('[data-action]');
  if (!btn) return;
  const action = btn.dataset.action;
  actions[action]?.(e);
});
```
**Why:** fewer listeners, works with dynamic elements.

### Accessible button from a div (only if necessary)
- Add `role="button"`, `tabindex="0"`, handle `Enter`/`Space`, and label it.

---

## Async/fetch

### Safe fetch with timeout + abort
```js
export async function fetchJson(url, { signal, timeout = 8000 } = {}) {
  const ctrl = new AbortController();
  const id = setTimeout(() => ctrl.abort(), timeout);
  try {
    const res = await fetch(url, { signal: signal ?? ctrl.signal });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } finally {
    clearTimeout(id);
  }
}
```

### Retry with backoff (once)
```js
async function withRetry(fn, times = 1) {
  try { return await fn(); }
  catch (e) {
    if (times <= 0) throw e;
    await new Promise(r => setTimeout(r, 500));
    return withRetry(fn, times - 1);
  }
}
```

---

## Performance UX

### Debounce & throttle
```js
export const debounce = (fn, ms = 300) => {
  let t;
  return (...args) => { clearTimeout(t); t = setTimeout(() => fn(...args), ms); };
};

export const throttle = (fn, ms = 200) => {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= ms) { last = now; fn(...args); }
  };
};
```

### List rendering keys (React or vanilla keyed map)
- Keys must be **stable** and **unique**. Don’t use array index for reordering lists.

---

## Testing checklist

- Unit: pure functions first (no DOM, no network).  
- Integration: API routes with **supertest**.  
- E2E: one happy-path per feature (create/edit/delete).  
- Always test: error states, empty states, slow network.

**Given-When-Then snippet**
```js
test('creates a todo', async () => {
  // Given an empty list
  // When I add "buy milk"
  // Then I see it in the DOM and storage
});
```

---

## Backend patterns

### Express error handling
```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status ?? 500).json({ error: err.message ?? 'Server error' });
});
```

### Zod validation
```ts
const Note = z.object({
  id: z.string().uuid().optional(),
  title: z.string().min(1),
  body: z.string().default(''),
});
```

---

## Edge case checklist (run this mentally)

- Empty input / whitespace only  
- Very long input  
- Network fail / slow / duplicate clicks  
- Refresh mid-action / back button  
- Timezone / locale formatting (dates)  
- Accessibility: focus order, keyboard only, labels

---

## Pitfall log (I won’t repeat these)

- ✅ Forgetting to cancel in-flight fetches on rapid input → **use AbortController**  
- ✅ Re-render loops from state updates in effects → **add deps carefully**  
- ✅ Using array index as key in lists → **use stable IDs**
