# Rust: Benchmarking for Fun and Profit!

<v-clicks>

- Rust's toolchain has another trick up its sleeve: the `cargo` package manager
- Some genius wrote a statistical benchmarking framework called <b>criterion</b>, based on the Haskell library of the same name
- We shall use it to get excellent performance data on our Nock Interpreter!
</v-clicks>

<v-click>

Let's see how we set it up...
</v-click>

---

# Rust: Benchmarking for Fun and Profit!

Assuming we already have a project created via `cargo new <project_name>`, open the `Cargo.toml` configuration file.

<v-clicks>

Add the following:

```toml
[dev-dependencies]
criterion = "0.5.1"

[[bench]]
name = "decrement"
harness = false
```

Then, from the root directory, create the benchmark file:

```sh
mkdir benches && touch benches/decrement.rs
```

Now, open it!

</v-clicks>

---

# Rust: Benchmarking for Fun and Profit!

Now, we add the following code

<v-click>

````md magic-move
```rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use nock_interpreter::Noun;

fn bench_nock_4k_box_decrement(c: &mut Criterion) {
    let formula = noun![8 [[1 0] [8 [[1 [6 [[5 [[0 7] [4 [0 6]]]] [[0 6] [9 [2 [[0 2] [[4 [0 6]] [0 7]]]]]]]]] [9 [2 [0 1]]]]]]];
    c.bench_function("nock_4k_box decrement", |b| {
        b.iter(|| {
            let subject = NounBox::atom(black_box(100));
            black_box(NounBox::tar(&subject, &formula))
        })
    });
}

criterion_group!(benches, bench_nock_4k_box_decrement);
criterion_main!(benches);
```

```rs{5-6}
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use nock_interpreter::Noun;

fn bench_nock_4k_box_decrement(c: &mut Criterion) {
    // This Nock Decrement Formula comes straight from Nock's Documentation
    let formula = noun![8 [[1 0] [8 [[1 [6 [[5 [[0 7] [4 [0 6]]]] [[0 6] [9 [2 [[0 2] [[4 [0 6]] [0 7]]]]]]]]] [9 [2 [0 1]]]]]]];
    c.bench_function("nock_4k_box decrement", |b| {
        b.iter(|| {
            let subject = NounBox::atom(black_box(100));
            black_box(NounBox::tar(&subject, &formula))
        })
    });
}

criterion_group!(benches, bench_nock_4k_box_decrement);
criterion_main!(benches);
```
````
</v-click>

---

# Rust: Benchmarking for Fun and Profit!

Let's skip ahead to the full naive implementation of `tar`(the Nock eval fn) using `Box`.

```rs{4-5,8-9,13-14,17-18}
pub fn tar(noun: &mut Noun) -> Noun {
    match noun {
        Noun::Atom(_) => panic!("tar cannot be performed on an atom"),
        Noun::Cell(subject, formula) => match &**formula {
            Noun::Cell(op, tail) => match (&**op, &**tail) {
                (distribute_cell @ Noun::Cell(..), d) => {
                    Noun::cell(
                        tar(&mut Noun::Cell(subject.clone(), Box::new(distribute_cell.clone()))),
                        tar(&mut Noun::Cell(subject.clone(), Box::new(d.clone())))
                    )
                },
                (Noun::Atom(2), Noun::Cell(b, c)) => tar(&mut Noun::cell(
                    tar(&mut Noun::Cell(subject.clone(), b.clone())),
                    tar(&mut Noun::Cell(subject.clone(), c.clone())))
                ),
                (Noun::Atom(5), Noun::Cell(b, c)) => tis(
                    &tar(&mut Noun::Cell(subject.clone(), b.clone())),
                    &tar(&mut Noun::Cell(subject.clone(), c.clone())),
                ),
```

---

# Rust: Benchmarking for Fun and Profit!

These deep clones have a serious performance penalty. I was benchmarking `~1.42 ms` for the decrement benchmark originally. Let's see how else we can do the same thing here...

````md magic-move
```rs{1,4-5,8-9,13-14,17-18}
pub fn tar(noun: &mut Noun) -> Noun {
    match noun {
        Noun::Atom(_) => panic!("tar cannot be performed on an atom"),
        Noun::Cell(subject, formula) => match &**formula {
            Noun::Cell(op, tail) => match (&**op, &**tail) {
                (distribute_cell @ Noun::Cell(..), d) => {
                    Noun::cell(
                        tar(&mut Noun::Cell(subject.clone(), Box::new(distribute_cell.clone()))),
                        tar(&mut Noun::Cell(subject.clone(), Box::new(d.clone())))
                    )
                },
                (Noun::Atom(2), Noun::Cell(b, c)) => tar(&mut Noun::cell(
                    tar(&mut Noun::Cell(subject.clone(), b.clone())),
                    tar(&mut Noun::Cell(subject.clone(), c.clone())))
                ),
                (Noun::Atom(5), Noun::Cell(b, c)) => tis(
                    &tar(&mut Noun::Cell(subject.clone(), b.clone())),
                    &tar(&mut Noun::Cell(subject.clone(), c.clone())),
                )
            }
        }
    }
}
```

```rs{1,4,6,9,12}
pub fn tar(subject: &Noun, formula: &Noun) -> Noun {
    match formula {
        Noun::Atom(_) => panic!("tar cannot be performed on an atom"),
        Noun::Cell(op, tail) => match (op.as_ref(), tail.as_ref()) {
            (distribute_cell @ Noun::Cell(..), d) => {
                Noun::cell(Self::tar(subject, distribute_cell), Self::tar(subject, d))
            }
            (Noun::Atom(2), Noun::Cell(b, c)) => {
                Self::tar(&Self::tar(subject, b), &Self::tar(subject, c))
            }
            (Noun::Atom(5), Noun::Cell(b, c)) => {
                Self::tis(&Self::tar(subject, b), &Self::tar(subject, c))
            }
        }
    }
}
```

```rs
// Original fn defn:
pub fn tar(noun: &mut Noun) -> Noun
// Optimized fn defn:
pub fn tar(subject: &Noun, formula: &Noun) -> Noun
```
````

<v-clicks>

- This simple change unlocked the ability to pass borrowed immutable references most of the time
- This allowed me to remove 41 of 44 total deep `clone()` calls from the `fn tar`
- The benchmark went from `~1.42 ms` to `~350 μs`, a <b>4x Speedup</b>
- The naive impl had serious performance issues due to being pigeon-holed by the borrow checker
- When forced to re-construct a `Cell` for the recursive call, there is no way to compile without cloning the outer `Box` first
- This is where Rust can inhibit you vs something like C++ without practice
</v-clicks>

---

# Rust: Benchmarking for Fun and Profit!

<v-clicks>

- So our box implementation now runs in `~350 μs`
- But, we still have 3 clones in the recursive pattern matching of `tar`
- We can swap `Box` for a reference counted pointer instead: `Rc`
- This would make it so any necessary clones due to borrowing are just refcount increments
</v-clicks>

<v-click>

````md magic-move
```rs{4}
// Original Noun Implementation
enum Noun {
    Atom(u64),
    Cell(Box<Noun>, Box<Noun>)
}
```

```rs{4}
// Optimized Noun Implementation
enum Noun {
    Atom(u64),
    Cell(Rc<Noun>, Rc<Noun>)
}
```
````
</v-click>

<v-clicks>

- With this simple change, we now run the <b>decrement</b> benchmark in `~50 μs`
- The naive solution ran in `~1.42 ms`
- Our speedup from naive to optimized, then, is: `~28x`. Not bad.
- All of this hinges on understanding Rust's borrowing rules
- There are more advanced rules on lifetimes and ownership, which is different than borrowing
</v-clicks>

---

# Rust: Lessons

<v-clicks>

- Rust is not necessarily quick to learn coming from C++
- It's different in ways that don't fit your existing mental models
- Rust's toolchain is awesome: package management, testing, and building are all easy
- Benchmarking is extremely easy
- Memory safety and thread safety are great guarantees, but performance may take a big hit if you don't know idiomatic Rust yet
</v-clicks>

---

# Rust: Recommendations

<v-clicks>

- There is a new book targeted at C++/C Devs, called <b>Effective Rust</b>, which pays homage to the <b>Effective C++</b> books you likely know
- Because there are many fewer <i>footguns</i> in Rust than C++, it does not have the same format
- Rather, it helps accelerate C++ to Rust transtition
- That would be my recommendation for anyone interested in getting better with Rust
</v-clicks>