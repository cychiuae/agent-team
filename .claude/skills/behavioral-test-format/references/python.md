# Python — behavioral test code

Use `pytest`. Use `@pytest.mark.parametrize` when multiple inputs share one behavior. Use fixtures for setup; never mutate global state.

## Naming

```
def test_<subject>_<behavior>(...): ...
```

`<subject>` is the function/method under test. `<behavior>` describes what is being pinned. One behavior per name.

## Trace block

Use the docstring directly under the function as the trace block:

```python
def test_apply_rejects_expired_coupon():
    """TC-3 | R-billing-2 | edge

    Given a coupon expiring at exactly t=now
    When apply is called
    Then it raises CouponExpiredError.
    """
    ...
```

## Single-case test

```python
def test_apply_rejects_expired_coupon(frozen_clock):
    """TC-3 | R-billing-2 | edge"""
    # Given
    coupon = Coupon(code="SAVE10", expires_at=frozen_clock.now)
    svc = Service(clock=frozen_clock)

    # When / Then
    with pytest.raises(CouponExpiredError):
        svc.apply(coupon)
```

## Parametrized test

One row per TC. Include `TC-#` in the `id` so failure output names the behavior.

```python
@pytest.mark.parametrize(
    "subtotal, coupon, expected, raises",
    [
        pytest.param(100, "SAVE10", 90, None, id="TC-1 happy: 10% off"),
        pytest.param(  5, "SAVE10",  5, None, id="TC-4 edge: below minimum"),
        pytest.param(100, "WAT",  None, UnknownCouponError,
                     id="TC-5 error: unknown coupon"),
    ],
)
def test_apply_discount_rules(subtotal, coupon, expected, raises):
    """R-billing-2 — covers TC-1, TC-4, TC-5."""
    order = Order(subtotal=subtotal, coupon=coupon)

    if raises is not None:
        with pytest.raises(raises):
            apply(order)
    else:
        assert apply(order) == expected
```

## Run before implementation

Tests must collect cleanly. If a production symbol doesn't exist yet, add the minimum stub (empty function, `class Service: pass`, sentinel exception) in the production module so the file imports. Tests then **fail on the assertion** (red), not at collection.

## Don'ts

- No `pytest.skip` / `xfail` to dodge red.
- No `monkeypatch` against private attributes — that asserts on internals.
- No `assert mock.foo.called` — that's an internals check.
