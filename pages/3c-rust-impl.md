# Rust: Speedrunning the Implementation

<v-clicks>

- Now, the basics are covered
- We can assume TDD is trivial to perform on the Nock Specification
- From here on, we will speed up the implementation, taking note of anything new and exciting in Rust
</v-clicks>

---

# Rust: Speedrunning the Implementation

<v-click>

````md magic-move
```rs
// Implement this operator
// lus:   +[a b] =>  +[a b]     // Increment cell: error
//        +a     =>  1 + a      // Increment atom              
```

```rs
// Implement this operator
// lus:   +[a b] =>  +[a b]     // Increment cell: error
//        +a     =>  1 + a      // Increment atom      
impl Noun {
    pub fn lus(noun: &Noun) -> Noun

    // ... other fn defs ...
}        
```

```rs
// Implement this operator
// lus:   +[a b] =>  +[a b]     // Increment cell: error
//        +a     =>  1 + a      // Increment atom      
impl Noun {
    pub fn lus(noun: &Noun) -> Noun {
        match noun {
            Noun::Atom(n) => Noun::Atom(n.wrapping_add(1)), // works for trivial impl
            Noun::Cell(_, _) => panic!("lus operation is not defined for cells, only atoms"),
        }
    }

    // ... other fn defs ...
}
```

```rs
// Implement this operator
// tis:   =[a a] =>  0          // Test equality: 0; true in Nock
//        =[a b] =>  1          // Test equality: 1; false in Nock
impl Noun {
    // ... other fn defs ...
}
```

```rs
// Implement this operator
// tis:   =[a a] =>  0          // Test equality: 0; true in Nock
//        =[a b] =>  1          // Test equality: 1; false in Nock
impl Noun {
    pub fn tis(noun1: &Noun, noun2: &Noun) -> Noun {
        if noun1 == noun2 {
            Noun::Atom(0)
        } else {
            Noun::Atom(1)
        }
    }

    // ... other fn defs ...
}
```

```rs{6-7}
// Implement this operator
// tis:   =[a a] =>  0          // Test equality: 0; true in Nock
//        =[a b] =>  1          // Test equality: 1; false in Nock
impl Noun {
    pub fn tis(noun1: &Noun, noun2: &Noun) -> Noun {
        // equality comes from Noun's #[derive(PartialEq)]
        if noun1 == noun2 {
            Noun::Atom(0)
        } else {
            Noun::Atom(1)
        }
    }

    // ... other fn defs ...
}
```

```rs
// Implement this operator
// fas:   /[1 a]           =>  a                // Address 1: identity
//        /[2 a b]         =>  a                // Address 2: head of cell
//        /[3 a b]         =>  b                // Address 3: tail of cell
//        /[(a + a) b]     =>  /[2 /[a b]]      // Even address: descend head
//        /[(a + a + 1) b] =>  /[3 /[a b]]      // Odd address: descend tail
//        /a               =>  /a               // Address atom: error
impl Noun {
    pub fn fas(address: &Noun, tree: &Noun) -> Noun
    // ... other fn defs ...
}
```

```rs{2,12-14}
// Implement this operator
// fas:   /[1 a]           =>  a                // Address 1: identity
//        /[2 a b]         =>  a                // Address 2: head of cell
//        /[3 a b]         =>  b                // Address 3: tail of cell
//        /[(a + a) b]     =>  /[2 /[a b]]      // Even address: descend head
//        /[(a + a + 1) b] =>  /[3 /[a b]]      // Odd address: descend tail
//        /a               =>  /a               // Address atom: error
impl Noun {
    pub fn fas(address: &Noun, tree: &Noun) -> Noun {
        match address {
            Noun::Atom(0) => panic!("fas operation does not support 0 address"),
            // This has to be deep cloned because of how Rust borrow checker works
            // We are borrowed here, and are not allowed to move in a borrow
            Noun::Atom(1) => tree.clone(),
            Noun::Atom(n) => todo!()
            Noun::Cell(..) => panic!("fas operation does not support cell address"),
        }
    }
    // ... other fn defs ...
}
```

```rs{1-2,11-20}
//        /[2 a b]         =>  a                // Address 2: head of cell
//        /[3 a b]         =>  b                // Address 3: tail of cell
//        /[(a + a) b]     =>  /[2 /[a b]]      // Even address: descend head
//        /[(a + a + 1) b] =>  /[3 /[a b]]      // Odd address: descend tail
//        /a               =>  /a               // Address atom: error
impl Noun {
    pub fn fas(address: &Noun, tree: &Noun) -> Noun {
        match address {
            Noun::Atom(0) => panic!("fas operation does not support 0 address"),
            Noun::Atom(1) => tree.clone(),
            Noun::Atom(n) if *n == 2 || *n == 3 => match tree {
                Noun::Cell(op, tail) => {
                    if *n == 2 {
                        *op.clone()
                    } else {
                        *tail.clone()
                    }
                }
                Noun::Atom(_) => panic!("fas operation found no child at this address"),
            },
            Noun::Atom(n) => todo!()
            Noun::Cell(..) => panic!("fas operation does not support cell address"),
        }
    }
    // ... other fn defs ...
}

```rs{1-2,11-20}
//        /[(a + a) b]     =>  /[2 /[a b]]      // Even address: descend head
//        /[(a + a + 1) b] =>  /[3 /[a b]]      // Odd address: descend tail
//        /a               =>  /a               // Address atom: error
impl Noun {
    pub fn fas(address: &Noun, tree: &Noun) -> Noun {
        match address {
            Noun::Atom(0) => panic!("fas operation does not support 0 address"),
            Noun::Atom(1) => tree.clone(),
            Noun::Atom(n) if *n == 2 || *n == 3 => match tree {
                Noun::Cell(op, tail) => {
                    if *n == 2 {
                        *op.clone()
                    } else {
                        *tail.clone()
                    }
                }
                Noun::Atom(_) => panic!("fas operation found no child at this address"),
            },
            Noun::Atom(n) => todo!()
            Noun::Cell(..) => panic!("fas operation does not support cell address"),
        }
    }
    // ... other fn defs ...
}
```

```rs{1-2,7-15}
//        /[(a + a) b]     =>  /[2 /[a b]]      // Even address: descend head
//        /[(a + a + 1) b] =>  /[3 /[a b]]      // Odd address: descend tail
impl Noun {
    pub fn fas(address: &Noun, tree: &Noun) -> Noun {
        match address {
            // .. other pattern matches ...
            Noun::Atom(n) if *n == 2 || *n == 3 => { ... } // handles base case of descent
            Noun::Atom(n) => match tree {
                // Use arithmetic to auto handle even or odd descent
                Noun::Cell(..) => Self::fas(
                    &Noun::Atom(2 + *n % 2),
                    &Self::fas(&Noun::Atom(*n / 2), tree),
                ),
                Noun::Atom(_) => panic!("fas operation found no child at this address"),
            },
        }
    }
    // ... other fn defs ...
}
```
````
</v-click>

---

# Rust: Speedrunning the Implementation

And you get the idea...

<v-clicks>

- Use Pattern Matching on inputs to determine which reduction rule to handle
- Use recursion when needed
- Try to limit deep cloning
- TRY TO LIMIT DEEP CLONING
</v-clicks>

<v-click>

That brings us to our next topic:
</v-click>

<v-clicks>

- Rust borrow checker can feel very limiting coming from C++ and C
- Getting used to the ownership, lifetimes, and borrowing rules takes time and practice
- I'll show you a real example of addressing performance issues that I had after my first naive implementation
</v-clicks>