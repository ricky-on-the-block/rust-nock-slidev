# Nock: The Foundation of Urbit
<v-clicks>

- <b>Foundational Language:</b> Nock is a low-level, interpreted <i>functional</i> programming language. It serves as the "assembly language" for Hoon, a higher-level functional programming language in which Urbit is implemented
- <b>Combinator Calculus:</b> Nock is a specific implementation of combinator calculus, a formal system for functional computation
- <b>Simplicity:</b> Only 11 instructions and 6 operations, making it easy to implement and verify
- <b>Kelvin Versioning:</b> Unique versioning system counting down to 0K, aiming for a final, frozen state. Currently at version 4K, demonstrating commitment to long-term stability
- <b>Universality:</b> Turing complete, despite its simplicity
</v-clicks>

---
layout: two-cols
---

# Nock Operations

<div class="pr-12">
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
</div>

::right::

# Nock Instructions

```
*[a [b c] d]        [*[a b c] *[a d]]
*[a 0 b]            /[b a]
*[a 1 b]            b
*[a 2 b c]          *[*[a b] *[a c]]
*[a 3 b]            ?*[a b]
*[a 4 b]            +*[a b]
*[a 5 b c]          =[*[a b] *[a c]]
*[a 6 b c d]        *[a *[[c d] 0 *[[2 3] 0 *[a 4 4 b]]]]
*[a 7 b c]          *[*[a b] c]
*[a 8 b c]          *[[*[a b] a] c]
*[a 9 b c]          *[*[a c] 2 [0 1] 0 b]
*[a 10 [b c] d]     #[b *[a c] *[a d]]
*[a 11 [b c] d]     *[[*[a c] *[a d]] 0 3]
*[a 11 b c]         *[a c]
*a                  *a
```


<v-click>

This defines all of Nock. We'll come back to this...
</v-click>