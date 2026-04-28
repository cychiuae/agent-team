# Go — behavioral test code

Use the standard `testing` package. Table-driven where multiple inputs share one behavior; `t.Run` for sub-tests so each case has its own name in the output.

## Naming

```
Test_<Subject>_<Behavior>
```

`<Subject>` is the function/type under test. `<Behavior>` describes what is being pinned. One behavior per name.

## Trace block

Place a comment block directly above each test:

```go
// TC-3 | R-billing-2 | edge
// Given a coupon expiring at exactly t=now
// When apply is called
// Then it returns ErrCouponExpired
```

## Single-case test

```go
func Test_Apply_RejectsExpiredCoupon(t *testing.T) {
    // TC-3 | R-billing-2 | edge

    // Given
    now := time.Date(2026, 1, 1, 0, 0, 0, 0, time.UTC)
    coupon := Coupon{Code: "SAVE10", ExpiresAt: now}
    svc := NewService(WithClock(func() time.Time { return now }))

    // When
    err := svc.Apply(context.Background(), coupon)

    // Then
    if !errors.Is(err, ErrCouponExpired) {
        t.Fatalf("got %v, want ErrCouponExpired", err)
    }
}
```

## Table-driven test

One row per TC. Each row gets its own `t.Run` name so failure output names the behavior.

```go
func Test_Apply_DiscountRules(t *testing.T) {
    // R-billing-2 | covers TC-1 (happy), TC-4 (edge), TC-5 (error)
    cases := []struct {
        name    string  // include "TC-#"
        tc      string
        input   Order
        want    Money
        wantErr error
    }{
        {
            name:  "TC-1 happy: 10% off applies to subtotal",
            input: Order{Subtotal: money(100), Coupon: "SAVE10"},
            want:  money(90),
        },
        {
            name:  "TC-4 edge: minimum order not met",
            input: Order{Subtotal: money(5), Coupon: "SAVE10"},
            want:  money(5),
        },
        {
            name:    "TC-5 error: unknown coupon code",
            input:   Order{Subtotal: money(100), Coupon: "WAT"},
            wantErr: ErrUnknownCoupon,
        },
    }
    for _, tt := range cases {
        t.Run(tt.name, func(t *testing.T) {
            // When
            got, err := Apply(tt.input)

            // Then
            if !errors.Is(err, tt.wantErr) {
                t.Fatalf("err: got %v, want %v", err, tt.wantErr)
            }
            if tt.wantErr == nil && got != tt.want {
                t.Fatalf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Run before implementation

Tests must compile. If the production type doesn't exist yet, define the minimum stub in the production package (empty function, sentinel error) so the test compiles and **fails on the assertion**, not at build time. The implementer fills the body.
