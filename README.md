### The State Design Pattern

In Rust, the State design pattern is a behavioral pattern that allows an object
to change its behavior when its internal state changes.

Because Rust does not support traditional object-oriented inheritance,
developers generally choose between three distinct approaches to manage states:
the Classical OOP-style (Trait-based) approach, the Enum-based state machine
approach, and the idiomatic Typestate pattern approach.

### Comparison of State Approaches in Rust

| Approach | Where Transitions Happen | Safety Level | Idiomatic? |
|---|---|---|---|
| Classical Trait | Runtime (Box<dyn State>) | Runtime checks required | No (un-idiomatic Rust) |
| Enum Machine | Runtime (match statements) | Safe, but handles error states | Yes, for dynamic states |
| Typestate | Compile-time (Type changes) | Maximum (Impossible to misbehave) | Highly Idiomatic |

### UML Diagrams

#### Enum-based State Machine (`src/enum-based.rs`) — State at runtime

```
┌──────────────────────┐
│       Cart           │
├──────────────────────┤
│ - state: CartState   │         ┌───────────────────────────┐
├──────────────────────┤         │      CartState (enum)     │
│ + new() -> Self      │         ├───────────────────────────┤
│ + add_item(&mut      │───────▶│ ◇ Empty                  │
│   self, String)      │  holds  │ ◇ Filled { items: Vec    │
│   -> Result<(),…>    │         │            <String> }     │
│ + checkout(&mut      │         │ ◇ Paid { invoice_id:     │
│   self, String)      │         │           String }        │
│   -> Result<(),…>    │         └───────────────────────────┘
└──────────────────────┘

Transitions (checked at runtime, return Result):
   Empty ──add_item──▶ Filled ──add_item──▶ Filled ──checkout──▶ Paid
     ▲                                            │
     └──────────────── add_item ✗ ────────────────┘      (runtime Err)
```

#### Typestate Pattern (`src/typestate.rs`) — State at compile time

```
┌──────────────────────────────┐
│          Cart<State>         │
├──────────────────────────────┤
│ - state: State               │
└──────────────────────────────┘
           ▲ generic over
           │
┌───────────────┐   ┌─────────────────────┐   ┌───────────────────────┐
│     Empty     │   │       Filled        │   │         Paid          │
├───────────────┤   ├─────────────────────┤   ├───────────────────────┤
│ (no fields)   │   │ - items: Vec<String>│   │ - invoice_id: String  │
└───────────────┘   └─────────────────────┘   └───────────────────────┘

impl Cart<Empty>:
  + new() -> Self                                    (self consumed)
  + add_item(self, String) -> Cart<Filled>  ─────────────────────▶┐
                                                                   │
impl Cart<Filled>:                                                 │
  + checkout(self, String) -> Cart<Paid>  ───────────────────▶┐   │
                                                               │   │
                                ┌──────────────────────────────┘   │
                                ▼                                  │
                        ┌───────────────┐                          │
                        │ Cart<Filled>  │◀────────────────────────┘
                        └───────────────┘

State machine (enforced by the compiler):
   Cart<Empty> ──add_item──▶ Cart<Filled> ──checkout──▶ Cart<Paid>
                                 ✓ invalid calls fail at COMPILE time
```
