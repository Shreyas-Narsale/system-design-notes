Design Patterns in Go
=====================

Strategy Pattern
----------------

When to Use
-----------

- You have multiple algorithms for a specific task.
- You want to switch algorithms dynamically at runtime.
- You want to avoid hardcoding multiple ways of doing something and swap behavior dynamically.

Example Scenarios
-----------------

- Payment gateways (PayPal, Stripe, Cash)
- Sorting algorithms (QuickSort, MergeSort)
- Logging strategies (Console, File, Remote)
- Rate-limiting strategies in APIs (Token Bucket, Leaky Bucket)

Components
----------

- Strategy Interface: defines the behavior/algorithm.
- Concrete Strategies: implement the Strategy interface.
- Context: holds a reference to a Strategy and delegates work to it.

Go Implementation
-----------------

Strategy Interface:

```go
package main

import "fmt"

// Strategy interface defines a common behavior.
type PaymentStrategy interface {
    Pay(amount float64)
}
```

Concrete Strategies:

```go
// CreditCard payment strategy.
type CreditCardPayment struct{}

func (c *CreditCardPayment) Pay(amount float64) {
    fmt.Printf("Paid %.2f using Credit Card\n", amount)
}

// PayPal payment strategy.
type PayPalPayment struct{}

func (p *PayPalPayment) Pay(amount float64) {
    fmt.Printf("Paid %.2f using PayPal\n", amount)
}

// Bitcoin payment strategy.
type BitcoinPayment struct{}

func (b *BitcoinPayment) Pay(amount float64) {
    fmt.Printf("Paid %.2f using Bitcoin\n", amount)
}
```

Context:

```go
// Context holds a reference to a PaymentStrategy.
type PaymentContext struct {
    strategy PaymentStrategy
}

// SetStrategy allows changing strategy at runtime.
func (c *PaymentContext) SetStrategy(strategy PaymentStrategy) {
    c.strategy = strategy
}

// Pay delegates the work to the strategy.
func (c *PaymentContext) Pay(amount float64) {
    c.strategy.Pay(amount)
}
```

Repository Architecture
-----------------------

```
internal/
├── payment/
│   ├── strategy.go         # Strategy interface
│   ├── credit_card.go      # Concrete strategy
│   ├── paypal.go           # Concrete strategy
│   ├── bitcoin.go          # Concrete strategy
│   └── context.go          # Context struct
└── order/
    └── order.go
```

Advantages
----------

- Change behavior dynamically.
- Each algorithm is separate and testable.

