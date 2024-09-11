# Key Figures and Timeline

<div class="flex justify-between items-start w-full">
  <div class="flex flex-col items-center w-1/3">
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Moses_Sch%C3%B6nfinkel_1922_%28cropped%29.jpg/440px-Moses_Sch%C3%B6nfinkel_1922_%28cropped%29.jpg" class="w-32 h-32 rounded-full mb-4" />
    <h3 class="text-lg font-bold mb-2">Moses Schönfinkel</h3>
    <p class="text-center">1920s: Introduces combinatory logic</p>
  </div>
  <div class="flex flex-col items-center w-1/3">
    <img src="https://www.computerhope.com/people/pictures/haskell_curry.jpg" class="w-32 h-32 rounded-full mb-4" />
    <h3 class="text-lg font-bold mb-2">Haskell Curry</h3>
    <p class="text-center">1930s: Further develops combinatory logic</p>
  </div>
  <div class="flex flex-col items-center w-1/3">
    <img src="https://upload.wikimedia.org/wikipedia/en/a/a6/Alonzo_Church.jpg?20090824110835" class="w-32 h-32 rounded-full mb-4" />
    <h3 class="text-lg font-bold mb-2">Alonzo Church</h3>
    <p class="text-center">1936: Introduces lambda calculus</p>
  </div>
</div>

---

# Combinator Calculus: What Is It?

<v-clicks>

- <b>Combinator:</b> A function without free variables. `Sxyz = xz(yz)`: `(x, y, z)` are all bound by position to combinator `S`
- <b>Calculus:</b> A method of computation/calculation in special notation
- <b>Free Variables:</b> Lambda Calculus <i>can</i> have free variables: `λx. y+x`: `y` is a free variable because it is not bound to lambda abstraction `λ`
</v-clicks>

<v-clicks>

What does that mean?

```c++
// x is a bound variable
// y is analagous to a free variable (captured here, but accessible outside)
int main() {
    int y = 10;
    auto adder = [y](int x) {
        return x + y;
    };
    cout << adder(5) << endl; // Output: 15
}
```
</v-clicks>

---

# SKI Combinator Calculus

<v-click>

The most well-known combinator calculus is called SKI. It's very simple.
</v-click>

<v-clicks>

- <b>I (Identity):</b> `Ix = x`. Whatever argument I is applied to is returned
- <b>K (Constant):</b> `Kxy = x`. Always returns the 1st argument `x`, ignoring the tail `y`
- <b>S (Combination):</b> `Sxya = xz(yz)`. `(x, y)` are functions that take one argument, `a` is an argument. It first applies `y` to `a`, then applies `x` to `a`, and finally applies the result of `x` to `a` to the result of `y` to `a`.
</v-clicks>

<v-click>

## Takeaway
</v-click>

<v-clicks>

- These combinators together are turing complete
- Even with a minimal set of tools, we can achieve computational power!
</v-clicks>
