# Nock Combinator Calculus

<v-click>

Now, let's revisit Nock...
</v-click>

<v-clicks>

- A <b>noun</b> is the basic data structure in Nock. 
- A <b>noun</b> is an <b>atom</b> or a <b>cell</b>
- An <b>atom</b> is an arbitrarily large `unsigned integer`
- A <b>cell</b> is an ordered pair of nouns: `(noun, noun)`
- This definition is recursive and ultimately means that a <b>noun</b> is a finite size binary tree whose leaves are <b>atoms</b>
- <b>Atom</b> values can be interpreted as different types by Hoon (higher level programming language)
</v-clicks>

<v-click>

```rs
enum Noun {
    Atom(u64),          // although arbitrarily large in reality
    Cell(Noun, Noun)    // ignoring container for Noun
}
```
</v-click>

---

# Nock Combinator Calculus

Now, let's return to the Nock definition, starting with the operators...

<v-click>

```
nock(a)             *a
[a b c]             [a [b c]]
?[a b]              0
?a                  1
+[a b]              +[a b]
+a                  1 + a
=[a a]              0
=[a b]              1
/[1 a]              a
/[2 a b]            a
/[3 a b]            b
/[(a + a) b]        /[2 /[a b]]
/[(a + a + 1) b]    /[3 /[a b]]
/a                  /a
#[1 a b]            a
#[(a + a) b c]      #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  #[a [/[(a + a) c] b] c]
#a                  #a
```
</v-click>

---

# Nock Combinator Calculus: Operators

````md magic-move
```
nock(a)             *a
[a b c]             [a [b c]]
?[a b]              0
?a                  1
+[a b]              +[a b]
+a                  1 + a
=[a a]              0
=[a b]              1
/[1 a]              a
/[2 a b]            a
/[3 a b]            b
/[(a + a) b]        /[2 /[a b]]
/[(a + a + 1) b]    /[3 /[a b]]
/a                  /a
#[1 a b]            a
#[(a + a) b c]      #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  #[a [/[(a + a) c] b] c]
#a                  #a
```

```{1,2}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a
[a b c]             =>  [a [b c]]
?[a b]              =>  0
?a                  =>  1
+[a b]              =>  +[a b]
+a                  =>  1 + a
=[a a]              =>  0
=[a b]              =>  1
/[1 a]              =>  a
/[2 a b]            =>  a
/[3 a b]            =>  b
/[(a + a) b]        =>  /[2 /[a b]]
/[(a + a + 1) b]    =>  /[3 /[a b]]
/a                  =>  /a
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{3}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]
?[a b]              =>  0
?a                  =>  1
+[a b]              =>  +[a b]
+a                  =>  1 + a
=[a a]              =>  0
=[a b]              =>  1
/[1 a]              =>  a
/[2 a b]            =>  a
/[3 a b]            =>  b
/[(a + a) b]        =>  /[2 /[a b]]
/[(a + a + 1) b]    =>  /[3 /[a b]]
/a                  =>  /a
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{4}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
?[a b]              =>  0
?a                  =>  1
+[a b]              =>  +[a b]
+a                  =>  1 + a
=[a a]              =>  0
=[a b]              =>  1
/[1 a]              =>  a
/[2 a b]            =>  a
/[3 a b]            =>  b
/[(a + a) b]        =>  /[2 /[a b]]
/[(a + a + 1) b]    =>  /[3 /[a b]]
/a                  =>  /a
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{5-7}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
// ? Operator: 'wut' - tests if cell or atom
?[a b]              =>  0                               // Test if cell: 0; true in Nock
?a                  =>  1                               // Test if atom: 1; false in Nock
+[a b]              =>  +[a b]
+a                  =>  1 + a
=[a a]              =>  0
=[a b]              =>  1
/[1 a]              =>  a
/[2 a b]            =>  a
/[3 a b]            =>  b
/[(a + a) b]        =>  /[2 /[a b]]
/[(a + a + 1) b]    =>  /[3 /[a b]]
/a                  =>  /a
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{7-9}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
?[a b]              =>  0                               // Test if cell: 0; true in Nock
?a                  =>  1                               // Test if atom: 1; false in Nock
// + Operator 'lus' - increments noun
+[a b]              =>  +[a b]                          // Increment cell: error
+a                  =>  1 + a                           // Increment atom
=[a a]              =>  0
=[a b]              =>  1
/[1 a]              =>  a
/[2 a b]            =>  a
/[3 a b]            =>  b
/[(a + a) b]        =>  /[2 /[a b]]
/[(a + a + 1) b]    =>  /[3 /[a b]]
/a                  =>  /a
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{9-11}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
?[a b]              =>  0                               // Test if cell: 0; true in Nock
?a                  =>  1                               // Test if atom: 1; false in Nock
+[a b]              =>  +[a b]                          // Increment cell: error
+a                  =>  1 + a                           // Increment atom
// = Operator 'tis' - tests equality                    
=[a a]              =>  0                               // Test equality: 0; true in Nock
=[a b]              =>  1                               // Test equality: 1; false in Nock
/[1 a]              =>  a
/[2 a b]            =>  a
/[3 a b]            =>  b
/[(a + a) b]        =>  /[2 /[a b]]
/[(a + a + 1) b]    =>  /[3 /[a b]]
/a                  =>  /a
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{11-17}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
?[a b]              =>  0                               // Test if cell: 0; true in Nock
?a                  =>  1                               // Test if atom: 1; false in Nock
+[a b]              =>  +[a b]                          // Increment cell: error
+a                  =>  1 + a                           // Increment atom                  
=[a a]              =>  0                               // Test equality: 0; true in Nock
=[a b]              =>  1                               // Test equality: 1; false in Nock
// / Operator 'tis' - address tree
/[1 a]              =>  a                               // Address 1: identity
/[2 a b]            =>  a                               // Address 2: head of cell
/[3 a b]            =>  b                               // Address 3: tail of cell
/[(a + a) b]        =>  /[2 /[a b]]                     // Even address: descend head
/[(a + a + 1) b]    =>  /[3 /[a b]]                     // Odd address: descend tail
/a                  =>  /a                              // Address atom: error
#[1 a b]            =>  a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]
#a                  =>  #a
```

```{17-21}
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
?[a b]              =>  0                               // Test if cell: 0; true in Nock
?a                  =>  1                               // Test if atom: 1; false in Nock
+[a b]              =>  +[a b]                          // Increment cell: error
+a                  =>  1 + a                           // Increment atom                  
=[a a]              =>  0                               // Test equality: 0; true in Nock
=[a b]              =>  1                               // Test equality: 1; false in Nock
/[1 a]              =>  a                               // Address 1: identity
/[2 a b]            =>  a                               // Address 2: head of cell
/[3 a b]            =>  b                               // Address 3: tail of cell
/[(a + a) b]        =>  /[2 /[a b]]                     // Even address: descend head
/[(a + a + 1) b]    =>  /[3 /[a b]]                     // Odd address: descend tail
/a                  =>  /a                              // Address atom: error
// # Operator 'hax' - edit tree
#[1 a b]            =>  a                               // Edit at address 1: replace b with a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]     // Edit at even address
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]         // Edit at odd address
#a                  =>  #a                              // Edit atom: error
```

```
Reduction Rule      =>  Reduction Result
----------------------------------------
nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
[a b c]             =>  [a [b c]]                       // Triple notation is right-associative
?[a b]              =>  0                               // Test if cell: 0; true in Nock
?a                  =>  1                               // Test if atom: 1; false in Nock
+[a b]              =>  +[a b]                          // Increment cell: error
+a                  =>  1 + a                           // Increment atom                  
=[a a]              =>  0                               // Test equality: 0; true in Nock
=[a b]              =>  1                               // Test equality: 1; false in Nock
/[1 a]              =>  a                               // Address 1: identity
/[2 a b]            =>  a                               // Address 2: head of cell
/[3 a b]            =>  b                               // Address 3: tail of cell
/[(a + a) b]        =>  /[2 /[a b]]                     // Even address: descend head
/[(a + a + 1) b]    =>  /[3 /[a b]]                     // Odd address: descend tail
/a                  =>  /a                              // Address atom: error
#[1 a b]            =>  a                               // Edit at address 1: replace b with a
#[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]     // Edit at even address
#[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]         // Edit at odd address
#a                  =>  #a                              // Edit atom: error
```

```
Op:    Reduction Rule      =>  Reduction Result
----------------------------------------
-      [a b c]             =>  [a [b c]]                       // Triple notation is right-associative
tar:   nock(a)             =>  *a                              // Evaluate noun 'a', *a for shorthand (e.g. "begin")
wut:   ?[a b]              =>  0                               // Test if cell: 0; true in Nock
       ?a                  =>  1                               // Test if atom: 1; false in Nock
lus:   +[a b]              =>  +[a b]                          // Increment cell: error
       +a                  =>  1 + a                           // Increment atom                  
tis:   =[a a]              =>  0                               // Test equality: 0; true in Nock
       =[a b]              =>  1                               // Test equality: 1; false in Nock
fas:   /[1 a]              =>  a                               // Address 1: identity
       /[2 a b]            =>  a                               // Address 2: head of cell
       /[3 a b]            =>  b                               // Address 3: tail of cell
       /[(a + a) b]        =>  /[2 /[a b]]                     // Even address: descend head
       /[(a + a + 1) b]    =>  /[3 /[a b]]                     // Odd address: descend tail
       /a                  =>  /a                              // Address atom: error
hax:   #[1 a b]            =>  a                               // Edit at address 1: replace b with a
       #[(a + a) b c]      =>  #[a [b /[(a + a + 1) c]] c]     // Edit at even address
       #[(a + a + 1) b c]  =>  #[a [/[(a + a) c] b] c]         // Edit at odd address
       #a                  =>  #a                              // Edit atom: error
```
````

---

# Nock: What About Performance?

<v-click>

Here are our operators:
</v-click>

<v-clicks>

1. `?` - "wut" - This is the cell test operator.
2. `+` - "lus" - This is the increment operator.
3. `=` - "tis" - This is the equality test operator.
4. `/` - "fas" - This is the slot (address) operator.
5. `#` - "hax" - This is the edit operator.
6. `*` - "tar" - This is the Nock evaluation operator.
</v-clicks>

<v-click>

Notice:
</v-click>

<v-clicks>

- Our only arithmetic operator is increment.
- What about add, subtract, multiply, divide?
- While they can all be implemented using our set of operators, that's not exactly efficient.
</v-clicks>

---

# Nock: Introducing Jets

<v-click>

What Are Jets?
</v-click>

<v-clicks>

- Jets are the most important contribution to this design space by the Urbit Project
- Jets are high-performance native code implementations of Nock functions
- They serve as optimized shortcuts for commonly used or computationally intensive operations
</v-clicks>

<v-click>

How Jets Work
</v-click>

<v-clicks>

- Jets provide identical results to their Nock counterparts but execute much faster
- The Nock interpreter can dynamically replace slow Nock code with equivalent jet code
</v-clicks>

---

# Nock: Making A Jet

<v-clicks>

1. Define the Nock function to be optimized
2. Implement a native code version (the jet) that produces identical results (C, Rust, etc)
3. Register the jet with the Nock runtime, associating it with its Nock equivalent
4. The runtime automatically uses the jet when the corresponding Nock code is encountered
</v-clicks>

---

# Nock: Making A Jet

Nock code for increment: `[6 [5 [0 7] [0 6]] [0 6] 0 2]`

<v-click>

Jet implementation in C:
```c
u3_noun u3qa_add(u3_atom a, u3_atom b) {
  if (_(u3a_is_cat(a)) && _(u3a_is_cat(b))) {
    c3_d c = ((c3_d) a) + ((c3_d) b);
    return u3i_words(1, &c);
  }
  else {
    mpz_t a_mp, b_mp, c_mp;

    u3r_mp(a_mp, a);
    u3r_mp(b_mp, b);
    mpz_init(c_mp);
    
    mpz_add(c_mp, a_mp, b_mp);

    return u3i_mp(c_mp);
  }
}
```
</v-click>

This jet drastically improves performance for the common operation of incrementing a number.