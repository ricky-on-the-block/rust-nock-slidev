# Rust: Defining the Noun

<v-click>

Let's grab our recursive definition from before:
</v-click>

<v-click>

```rs
enum Noun {
    Atom(u64),          // arbitrarily large in reality
    Cell(Noun, Noun)    // ignoring container for Noun
}
```
</v-click>

<v-click>

Rust `enum`s are much more <b>powerful</b> than C/C++:
</v-click>

<v-clicks>

- Similar to algebraic data types in functional languages, which gives us pattern matching
- Can contain data and methods, like C++ classes
- Each variant can have different types and amounts of data
- Can implement traits (similar to interfaces)
- Memory-efficient: size is the largest variant plus discriminant
- No separate "union" type in Rust; enums cover this use case
</v-clicks>

---

# Rust: Defining the Noun

<v-click>

Let's figure out our Noun container. We know we will need to heap allocate, because we can't know how large the Noun is in advance. So, our obvious types are:
</v-click>

<v-clicks>

- `Box`: Rust's equivalent to a `unique_ptr` in C++; heap-allocated with single ownership.
- `Rc` : Similar to `shared_ptr` in C++; heap-allocated with reference-counted shared ownership.
</v-clicks>

<v-clicks>

Since we expect to be performing pure functional operations, a single `Box` should suffice.

````md magic-move
```rs{3}
enum Noun {
    Atom(u64),
    Cell(Noun, Noun)
}
```

```rs{3}
enum Noun {
    Atom(u64),
    Cell(Box<Noun>, Box<Noun>)
}
```

```rs{1}
#[derive(Debug, Clone, PartialEq)]
enum Noun {
    Atom(u64),
    Cell(Box<Noun>, Box<Noun>)
}
```
````
</v-clicks>

<v-clicks>

- `Debug`: Automatically implements the `Debug` trait, enabling debug formatting with `{:?}`
- `Clone`: Adds the ability to create a deep copy of the type with the `clone()` method
- `PartialEq`: allows for equality comparisons with `==` and `!=`
</v-clicks>

---

# Rust: Noun Helper

```rs
enum Noun {
    Atom(u64),
    Cell(Box<Noun>, Box<Noun>)
}
```

<v-clicks>

With this definition, we can now start making Nouns. But, they get verbose quickly.

```rs
let complex_noun = Noun::Cell(
    Box::new(Noun::Atom(42)),
    Box::new(Noun::Cell(
        Box::new(Noun::Atom(23)),
        Box::new(Noun::Cell(
            Box::new(Noun::Atom(7)),
            Box::new(Noun::Atom(5))
        ))
    ))
);
```

</v-clicks>

---

# Rust: Introducing Macros

```rs
let complex_noun = Noun::Cell(
    Box::new(Noun::Atom(42)),
    Box::new(Noun::Cell(
        Box::new(Noun::Atom(23)),
        Box::new(Noun::Cell(
            Box::new(Noun::Atom(7)),
            Box::new(Noun::Atom(5))
        ))
    ))
);
```

We can make this a lot simpler by using Rust macros. But first, what's different about them?

<v-clicks>

- Rust macros can be pattern-based (declarative) or procedural (code generation)
- Rust macros operate on the abstract syntax tree (AST), not just text substitution
- Rust macros can create new items (functions, structs, etc.), C/C++ macros can't
- Rust macros are type-aware and can use Rust's type system
- Rust macros can be unit tested
</v-clicks>

---

# Rust: Building a Recursive Macro

Ideally, we could just define our Nouns the way Nock defines them:

```
[8 [[4 [0 1]] [0 3]]]
```

Let's try!

````md magic-move
```rs
macro_rules! noun {
    // Let's start with the base case of a single Atom: [42]
}
```

```rs{3-6}
macro_rules! noun {
    // Let's start with the base case of a single Atom: [42]
    // Matches a single literal number inside brackets [x], and expands it to Noun::Atom(x)
    [$num:literal] => {
        Noun::Atom($num)
    };
}
```

```rs{5}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };

    // Now, let's do the base case of a single Cell: [1 2]
}
```

```rs{6-9}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };

    // Now, let's do the base case of a single Cell: [1 2]
    // Matches two literal numbers inside brackets [x y], and builds the Cell
    [$num1:literal $num2:literal] => {
        Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::atom($num2)))
    };
}
```

```rs{8}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };

    // Rule 2: Match two literals (simple cell)
    [$num1:literal $num2:literal] => { Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::atom($num2))) };

    // Now, let's handle a recursive structure on the right: [x [a [b c]]] => recurse on [a [b c]]
}
```

```rs{9-13}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };

    // Rule 2: Match two literals (simple cell)
    [$num1:literal $num2:literal] => { Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::atom($num2))) };

    // Now, let's handle a recursive structure on the right: [x [a [b c]]] => recurse on [a [b c]]
    // $left:tt matches any single token tree
    // [$($right:tt)+] matches one or more token trees inside brackets, like [a [b c]]
    [$left:tt [$($right:tt)+]] => {
        Noun::Cell(Box::new(noun![$left]), Box::new(noun![$($right)+]))
    };
}
```

```rs{6-7}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };
    // Rule 2: Match two literals (simple cell)
    [$num1:literal $num2:literal] => { Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::atom($num2))) };
    // Rule 3: Match a single token on the left and a nested structure on the right
    [$left:tt [$($right:tt)+]] => { Noun::Cell(Box::new(noun![$left]), Box::new(noun![$($right)+])) };
}
```

```rs{9}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };
    // Rule 2: Match two literals (simple cell)
    [$num1:literal $num2:literal] => { Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::atom($num2))) };
    // Rule 3: Match a single token on the left and a nested structure on the right
    [$left:tt [$($right:tt)+]] => { Noun::Cell(Box::new(noun![$left]), Box::new(noun![$($right)+])) };

    // Now, let's handle a recursive structure on the left: [[a [b c]] x] => recurse on [a [b c]]
}
```

```rs{5-9}
macro_rules! noun {
    // ... prev macro rules 1-3 ... 

    // Now, let's handle a recursive structure on the left: [[a [b c]] x] => recurse on [a [b c]]
    // [$($left:tt)+] matches one or more token trees inside brackets, like [a [b c]]
    // $right:tt matches any single token tree
    [[$($left:tt)+] $right:tt] => {
        Noun::Cell(Box::new(noun![$($left)+]), Box::new(noun![$right]))
    };
}
```

```rs{8-9}
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };
    // Rule 2: Match two literals (simple cell)
    [$num1:literal $num2:literal] => { Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::atom($num2))) };
    // Rule 3: Match a single token on the left and a nested structure on the right
    [$left:tt [$($right:tt)+]] => { Noun::Cell(Box::new(noun![$left]), Box::new(noun![$($right)+])) };
    // Rule 4: Match a nested structure on the left and a single token on the right
    [[$($left:tt)+] $right:tt] => { Noun::Cell(Box::new(noun![$($left)+]), Box::new(noun![$right])) };
}
```

```rs{4}
macro_rules! noun {
    // ... prev macro rules 1-4 ... 

    // Now, let's handle a recursive structure on both sides: [[a [b c]] [x [y z]] => recurse on [a [b c]] and [x [y z]]
}
```

```rs{5-9}
macro_rules! noun {
    // ... prev macro rules 1-4 ... 

    // Now, let's handle a recursive structure on both sides: [[a [b c]] [x [y z]] => recurse on [a [b c]] and [x [y z]]
    // [$($left:tt)+] matches one or more token trees inside brackets, like [a [b c]]
    // [$($right:tt)+] matches one or more token trees inside brackets, like [x [y z]]
    [[$($left:tt)+] [$($right:tt)+]] => {
        Noun::Cell(Box::new(noun![$($left)+]), Box::new(noun![$($right)+]))
    };
}
```

```rs
macro_rules! noun {
    // Rule 1: Match a single literal (base case for atoms)
    [$num:literal] => { Noun::Atom($num) };
    // Rule 2: Match two literals (simple cell)
    [$num1:literal $num2:literal] => { Noun::Cell(Box::new(Noun::Atom($num1)), Box::new(Noun::Atom($num2))) };
    // Rule 3: Match two nested structures
    [[$($left:tt)+] [$($right:tt)+]] => { Noun::Cell(Box::new(noun![$($left)+]), Box::new(noun![$($right)+])) };
    // Rule 4: Match a nested structure on the left and a single token on the right
    [[$($left:tt)+] $right:tt] => { Noun::Cell(Box::new(noun![$($left)+]), Box::new(noun![$right])) };
    // Rule 5: Match a single token on the left and a nested structure on the right
    [$left:tt [$($right:tt)+]] => { Noun::Cell(Box::new(noun![$left]), Box::new(noun![$($right)+])) };
}
```
````

---

# Rust: Using Our Macro

Awesome, now we can instantiate nouns more easily.

````md magic-move
```rs
let complex_noun = Noun::Cell(
    Box::new(Noun::Atom(42)),
    Box::new(Noun::Cell(
        Box::new(Noun::Atom(23)),
        Box::new(Noun::Cell(
            Box::new(Noun::Atom(7)),
            Box::new(Noun::Atom(5))
        ))
    ))
);
```
```rs
let complex_noun = noun![42 [23 [7 5]]];
```
````

<v-click>

Now that creating nouns is a lot easier, let's circle back on those Nock operators...

</v-click>