# Rust: TDD in Rust

<v-clicks>

- Recall that our Nock definition tells us exactly what our operations need to do.
- This allows us to perform something analogous to BDD, but we won't go as far as Gherkin (the de-facto language for BDD).
- Let's make use of Rust's integrated testing features instead
</v-clicks>

<v-clicks>

Let's start with the <b>wut</b> operator:

```
Op:    Reduction Rule      =>  Reduction Result
-----------------------------------------------
wut:   ?[a b]              =>  0            // Test if cell: 0; true in Nock
       ?a                  =>  1            // Test if atom: 1; false in Nock
```
</v-clicks>

---

# Rust: TDD in Rust

Let's start with the <b>wut</b> operator:

```
Op:    Reduction Rule      =>  Reduction Result
-----------------------------------------------
wut:   ?[a b]              =>  0            // Test if cell: 0; true in Nock
       ?a                  =>  1            // Test if atom: 1; false in Nock
```

<v-clicks>

We can easily perform BDD/TDD while implementing this operator:

````md magic-move
```rs
// test wut on cell, should return 0
// test wut on atom, should return 1
```

```rs
#[test]
fn test_wut_on_cell() {
    let cell = noun![1 2];
    assert_eq!(Noun::wut(&cell), noun![0]);
}

// test wut on atom, should return 1
```

```rs
#[test]
fn test_wut_on_cell() {
    let cell = noun![1 2];
    assert_eq!(Noun::wut(&cell), noun![0]);
}

#[test]
fn test_wut_on_atom() {
    let atom = noun![42];
    assert_eq!(Noun::wut(&atom), noun![1]);
}
```
````
</v-clicks>

---

# Rust: TDD in Rust

```rs
#[test]
fn test_wut_on_cell() {
    let cell = noun![1 2];
    assert_eq!(Noun::wut(&cell), noun![0]);
}

#[test]
fn test_wut_on_atom() {
    let atom = noun![42];
    assert_eq!(Noun::wut(&atom), noun![1]);
}
```

<v-click>

Running tests in Rust is built into the toolchain. Simply run, `cargo test`. It honestly is that simple.
</v-click>

---

# Rust: TDD in Rust

Now, we have a failing test (red). Let's implement this basic functionality (green).

````md magic-move
```rs
#[test]
fn test_wut_on_cell() {
    let cell = noun![1 2];
    assert_eq!(Noun::wut(&cell), noun![0]);
}

#[test]
fn test_wut_on_atom() {
    let atom = noun![42];
    assert_eq!(Noun::wut(&atom), noun![1]);
}
```

```rs
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

#[test]
fn test_wut_on_cell() {
    let cell = noun![1 2];
    assert_eq!(Noun::wut(&cell), noun![0]);
}
#[test]
fn test_wut_on_atom() {
    let atom = noun![42];
    assert_eq!(Noun::wut(&atom), noun![1]);
}
```

```rs
// We have some new syntax, now
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{1-3}
// Recall Noun is an enum, and here we are attaching a function to it (similar to methods/classes)
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{2-4}
impl Noun {
    // this is public function that takes an immutable reference to Noun, and returns a Noun
    // in Rust, everything is immutable by default.
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{2-3}
impl Noun {
    // if we wanted a mutable reference, we would define it like so
    pub fn wut(noun: &mut Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{2-3}
impl Noun {
    // but, considering we are working on a pure functional combinator calculus, we will use immutable
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{2-3}
impl Noun {
    // &Noun is similar to `const Noun&` in C++, but there's a crucial difference:
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{2-5}
impl Noun {
    // &Noun is similar to `const Noun&` in C++, but there's a crucial difference:
    // The original owner can't modify the data while it's borrowed. This is enforced by:
    // The Borrow Checker in Rust, a compile-time memory safety guarantor
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{2}
impl Noun {
    // the borrow checker tracks borrows and ownership, but we'll come back to that...
    pub fn wut(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{3-7}
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        // notice the match, here. this is Pattern Matching.
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{3-8}
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        // notice the match, here. this is Pattern Matching.
        // the Rust compiler guarantees that all variants of a type are handled, here
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```

```rs{5,8}
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        // let's notice this. this is Pattern Matching.
        // the Rust compiler guarantees that all variants of a type are handled, here
        // if we, say, forgot to implement Atom, we would fail compilation
        match noun {
            Noun::Cell(_, _) => Noun::Atom(0),
            // COMPILE ERROR: Noun::Atom not handled
        }
    }
}

// ... tests below ...
```

```rs{7-8}
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        // let's notice this. this is Pattern Matching.
        // the Rust compiler guarantees that all variants of a type are handled, here
        match noun {
            Noun::Cell(_, _) => Noun::Atom(0),
            // We could cover a default case using _ and pass compilation again, but our tests would fail
            _ => todo!()
        }
    }
}

// ... tests below ...
```

```rs{3}
impl Noun {
    pub fn wut(noun: &Noun) -> Noun {
        // Pattern Matching will allow us to make quick work of Nock
        match noun {
            Noun::Atom(_) => Noun::Atom(1),
            Noun::Cell(_, _) => Noun::Atom(0),
        }
    }
}

// ... tests below ...
```
````