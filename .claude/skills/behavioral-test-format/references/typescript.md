# TypeScript — behavioral test code

Use the project's existing runner (`vitest` or `jest`). Group with `describe`, name behaviors with `it`.

## Naming

```
describe('<Subject>', () => {
  it('<behavior in plain English>', () => { ... })
})
```

One behavior per `it`. The `it` text reads like a sentence: "rejects an expired coupon", not "test1".

## Trace block

A leading comment or the `it` description carries the trace IDs:

```ts
// TC-3 | R-billing-2 | edge
it('rejects a coupon expiring at exactly now', () => { ... })
```

## Single-case test

```ts
import { describe, it, expect } from 'vitest'

describe('Service.apply', () => {
  // TC-3 | R-billing-2 | edge
  it('rejects a coupon expiring at exactly now', () => {
    // Given
    const now = new Date('2026-01-01T00:00:00Z')
    const coupon = { code: 'SAVE10', expiresAt: now }
    const svc = new Service({ clock: () => now })

    // When / Then
    expect(() => svc.apply(coupon)).toThrow(CouponExpiredError)
  })
})
```

## Table-driven via `it.each`

One row per TC; include `TC-#` in the description.

```ts
describe('apply — discount rules (R-billing-2)', () => {
  it.each([
    { id: 'TC-1 happy', subtotal: 100, coupon: 'SAVE10', want: 90,  err: null },
    { id: 'TC-4 edge',  subtotal:   5, coupon: 'SAVE10', want:  5,  err: null },
    { id: 'TC-5 error', subtotal: 100, coupon: 'WAT',    want: null, err: UnknownCouponError },
  ])('$id', ({ subtotal, coupon, want, err }) => {
    const order = { subtotal, coupon }

    if (err) {
      expect(() => apply(order)).toThrow(err)
    } else {
      expect(apply(order)).toBe(want)
    }
  })
})
```

## Run before implementation

The test file must type-check and compile. If a production symbol doesn't exist yet, add the minimum stub (empty function, `class Service {}`, exported error class) in the production module so the import resolves. Tests then **fail on the assertion** (red), not at compile.

## Don'ts

- No `it.skip` / `it.todo` to dodge red.
- No assertions on `vi.fn()` / `jest.fn()` call counts as the only check — that's an internals assertion.
