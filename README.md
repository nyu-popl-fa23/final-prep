## Final Exam Preparation

You can use the problems below to practice for the final exam. Some of
the practice problems are a bit more difficult than what you will
encounter in the exam. I do not expect you to be able to solve all of
these practice problems in the time that is allotted for the final
exam. There will be fewer questions in the exam, but the materials
covered will be similar. Finally, the exam will again include a
multiple choice problem that tests your general understanding of the
concepts.

As you will notice from the problems below, the final exam will focus
on the materials covered in the second half of the semester. However,
many concepts from the first half of the semester such as operational
semantics, abstract syntax trees, etc. will of course still be highly
relevant.

### Problem 1: Higher-Order Functions

1. Implement a Scala function

   ```scala
   def dropWhile[A](p: A => Boolean)(l: List[A]): List[A]
   ```

   that takes a predicate `p` and a list `l` and drops the longest prefix of
   elements of `l` that satisfies `p`. Example:

   ```scala
   scala> dropWhile(_ < 0)(List(-1, -2, 3, -5, 6))
   res0: List[Int] = List(3, -5, 6)
   ```
   
   1. implement `dropWhile` using recursion (i.e., without using
      `foldLeft`/`foldRight`). Also implement a tail-recursive version.

       <details><summary>Solution</summary>
       <p>
      
       ```scala
       // non-tail recursive version
       def dropWhile[A](p: A => Boolean)(l: List[A]): List[A] = l match
         case Nil => Nil
         case hd :: tl => if p(hd) then dropWhile(tl) else hd :: tl
         
       // tail-recursive version
       def dropWhile[A](p: A => Boolean)(l: List[A]): List[A] =
         /* invariant maintained by dw(lp, acc): 
            there exists a list lpp such that 
            1. l == lpp ++ acc.reverse ++ lp
            2. for all elements x in lpp, p(x) is true
            3. if acc.reverse == hd :: _ for some hd, then p(hd) is false
         */
         def dw(lp: List[A], acc: List[A]): List[A] = lp match
           case Nil => acc
           case hd :: tl => dw(lp, if acc == Nil && p(hd) then acc else hd :: acc)
         dw(l, Nil).reverse
       ```
       </p>
       </details>


   2. implement `dropWhile` using `foldLeft`/`foldRight`, whichever you find
      appropriate (though, make sure that you compute the correct result).
      
      <details><summary>Solution</summary>
      <p>
      
      ```scala
      def dropWhile[A](p: A => Boolean)(l: List[A]): List[A] =
        l.foldLeft(Nil: List[A]):
          (acc, hd) => if acc == Nil && p(hd) then acc else hd :: acc
        .foldLeft(Nil: List[A]):
          (acc, hd) => hd :: acc
      ```
      
      Note how the first `foldLeft` call can be directly obtained from the
      function `dw` by extracting the expression for the initial
      value `Nil` of the accumulator and the expression
      ```scala
      if acc == Nil && p(hd) then acc else hd :: acc
      ```
      for the update of the accumulator in the recursive step.

      The second `foldLeft` call implements `reverse`.

      </p>
      </details>

2. Implement a Scala function

   ```scala
   def countSat[A](p: A => Boolean)(l: List[A]): Int
   ```

   that counts the number of elements in `l` for which `p` returns true. Example:

   ```scala
   scala> countSat(_ < 0)(List(-1, -2, 3, -5, 6))
   res0: Int = 3
   
   scala> countSat(_ >= 0)(List(-1, -2, 3, -5, 6))
   res`: Int = 2
   ```
   
   Proceed as in part 1.

   <details><summary>Solution</summary>
   <p>
      
   ```scala
   // non-tail-recursive version
   def countSat[A](p: A => Boolean)(l: List[A]): Int = l match:
     case Nil => 0
     case hd :: tl =>
       (if p(hd) then 1 else 0) + countSat(p)(tl)

   // tail-recursive version
   def countSat[A](p: A => Boolean)(l: List[A]): Int =
     /* invariant maintained by cs(lp, acc): 
        there exists a list lpp such that l == lpp ++ lp and acc == countSat(p)(lpp)
      */
     def cs(lp: List[A], acc: Int): Int = lp match
        case Nil => acc
        case hd :: tl => cs(tl, (if p(hd) then 1 else 0) + acc)
     cs(l, 0)
   
   // version with foldLeft
   def countSat[A](p: A => Boolean)(l: List[A]): Int =
     l.foldLeft(0):
       (acc, hd) => (if p(hd) then 1 else 0) + acc
   
   ```
   </p>
   </details>

### Problem 2: Type Inference and Type Checking

For each of the programs below, answer the following questions:

a. Can you find concrete types for the missing parameter type
   annotations `t1`,...,`t5` such that the given program is well-typed
   according to the typing rules of JakartaScript? If yes, provide one
   such annotation. If no such type annotations exist, explain why. 

b. If your answer to (a) is "yes", are your chosen types the only
   annotations that work?  If not, give at least one other choice of
   annotations that also makes the program well-typed.

c. If your answer to (a) is "no", can the program be safely
   evaluated according to the operational semantics of JakartaScript? If
   yes, what value does the program evaluate to? If not, where does the
   evaluation get stuck?

Programs:

1. ```javascript
   const f = (x: t1) => x + x;
   f(3)
   ```
   
   <details><summary>Solution</summary>
   <p>
   
   From the call `f(3)` we can infer that `t1` must by of type
   `Num`. Using this annotation makes the program well-typed. There is
   no other annotation that also makes the program well-typed because of
   the call `f(3)`.
   </p>
   </details>

2. ```javascript
   const swap = (f: t1) => (x: t2) => (y: t3) => f(y)(x);
   const g = (x: t4) => (y: t5) => x;
   swap(g)
   ```

   <details><summary>Solution</summary>
   <p>
   
   From the function call `f(y)(x)` we can infer that any annotation
   that makes the program well-typed must satisfy the equation:

   `t1 = t3 => t2 => t`

   for some type `t`.

   From the definition of `g` in the const declaration we infer that `g` has
   the type `tg` defined by the equation:

   `tg = t4 => t5 => t4`

   From the call `swap(g)` we can now infer that the program is only well-typed if:

   `t1 = tg`

   Using the equations defining `t1` and `tg` we can infer that this is only true if

   `t3 = t4` and
   `t2 = t5` and
   `t = t4`

    Thus, any annotation that satisfies the following equations will make
    the program well-typed:

    `t3 = t4` and
    `t2 = t5` and
    `t1 = t3 => t2 => t4`

    We can choose for example:

    `t3 = t2 = t4 = t5 = Bool` and
    `t1 = Bool => Bool => Bool`

    or

    `t3 = t4 = Bool` and
    `t2 = t5 = Num`
    `t1 = Bool => Num => Bool`
   
   </p>
   </details>


3. ```javascript
   (c: t1) => {
     c.get = () => c.x;
     c.inc = () => { c.x = c.x + 1; return c; };
     return c;
   }
   ```

   <details><summary>Solution</summary>
   <p>
   
   From the field dereference operations on `c` we can infer that `t1` must be
   an object type that provides at least the fields `x`, `get`, and
   `inc`. Moreover, since all of these fields are used in field assignements, they must have
   mutability 'let'. The types of the fields can be inferred from the
   right-hand sides of the assignment expressions. Adding all this
   information together, we infer that any annotation `t1` that makes the
   program well-typed must satisfy the following equation:

   ```
   t1 = { let x: Num,
          let get: () => Num,
          let inc: () => t1 }
   ```

   Case 1: no subtyping.

   Note that `t1` occurs again as the result type of the function type
   that `inc` must satisfy. Recall that the type expressions of our
   type system are finite abstract syntax trees that are constructed using structural
   recursion. This means that there is no type `t1` that satisfies
   the above equation because we cannot construct a finite tree that
   contains itself as a proper subtree. The recursive equation defining t1 can only be satisfied by an infinite tree. It is therefore impossible to find a type annotation (in our type system) that makes the above
   program well-typed.
   
   **Remark**: to make the above program typable in this case, we can extend the type
   system with recursive types (which from a mathematical perspective can
   be viewed as infinite ASTs). Recursive types are a common extension
   found in most typed programming languages. They are, e.g., needed to
   describe the types of recursive data structures such as linked-lists,
   trees, etc.

   Case 2: subtyping.
   
   If we allow subtyping, then we can replace t1 on the right side of the equation with any of its supertypes. There are infinitely many such types that work. For instance:
   
   ```
   t1 = { let x: Num,
          let get: () => Num,
          let inc: () => t }
   ```
   
   where 
   
   ```
   t = {}
   ``` 
   or
   ```
   t = { let x: Num,
         let get: () => Num,
         let inc: () => {} }
   ```
   or
   ```
   t = { let x: Num,
         let get: () => Num,
         let inc: () => { let x: Num,
                          let get: () => Num,
                          let inc: () => {} }
       }
   ```
   and so forth.

   Since the program is already a value, it can be safely evaluated. The result is the program itself.
   </p>
   </details>

4. ```javascript
   (c: t1) => {
     c.get = () => c.x;
     c.inc = () => { c.x = c.x + 1; return undefined; };
     return c;
   }
   ```

   <details><summary>Solution</summary>
   <p>
   
   This example is almost identical to 3. This time we infer the
   following constraint for `t1`:

   ```
   t1 = { let x: Num,
          let get: () => Num,
          let inc: () => Undefined }
   ```

   The type expression on the right side of the equality has no type
   variables, so it is a valid type annotation that would make the
   program well-typed. If we consider subtyping, then this type is the
   greatest type with respect to subtyping that makes the program
   well-typed (i.e. it is the type that imposes the fewest constraints
   on the parameter `c`). However, any subtype of this type is also a
   valid annotation. For example, the following type for `t1` would
   also work:

   ```
   { let x: Num,
     let y: Num,
     let get: () => Num,      
     let inc: () => Undefined }
   ```
   </p>
   </details>


### Problem 3: Parameter Passing Modes

Consider the typed variant of JakartaScript that supports the
different parameter passing modes we discussed. For each of the
following programs, say whether the program is well-typed. If not,
provide a brief explanation of the type error.  If yes, compute the
value that the program evaluates to and provide a brief explanation
why you obtain that specific value.


1. ```javascript
   let y = 2;
   const f = (let x: Num) => (x = x * x, x);
   f(y) + y
   ```

   <details><summary>Solution</summary>
   <p>
   
   The program is well-typed. It evaluates to 6. The let parameter `x` is
   freshly allocated in memory for each call to `f`, so the assignment to `x` in the body of `f` does not
   modify the value of `y`. Hence, `f(y)` returns 4, and `y` is still 2 when `f` returns.
   </p>
   </details>


2. ```javascript
   let x = 1;
   const y = 2;
   const f = (name y: Num) => y - y;
   f(x = x + y) + x
   ```

   <details><summary>Solution</summary>
   <p>
   
   The program is well-typed. It evaluates to 3. Since the parameter `y` of
   `f` is a `name` parameter, each occurrence of `y` in the body of `f` will
   reevaluate the argument expression `x = x + y`. Thus, `f` returns `-2`, and
   the new value of `x` is `5`, when `f` returns.
   </p>
   </details>

3. ```javascript
   let y = 2;
   const f = (ref x: Num) => (x = x * x, x);
   f(y) + y
   ```

   <details><summary>Solution</summary>
   <p>
   
   The program is well-typed. It evaluates to 8. The program is
   similar to (a) except that `f`'s parameter `x` is now a `ref` parameter,
   which means that the assignment in `f`'s body also updates the value
   of `y` to 4. Thus, `f` returns `4` and the new value of `y` when `f` returns is also 4.
   </p>
   </details>

4. ```javascript
   const y = 2;
   const f = (ref x: Num) => (x = x * x, x);
   f(y) + y
   ```

   <details><summary>Solution</summary>
   <p>
   
   The program is not well-types since we are trying to call `f` (which
   has a `ref` parameter) with the const variable `y` (which is not
   assignable).
   
   </p>
   </details>

### Problem 4: Subtyping

1. Decide whether the following subtype relations hold

   a. `Num <: {let f: Num}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   b. `{let f: Num, const g: Bool} <: {let g: Bool, const f: Num}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   c. `{let f: Num} <: {let g: Bool, const f: Num}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   d. `{let g: Bool, const f: Num} <: {let f: Num}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   e. `{let f: Num} <: {const f: Num}`

      <details><summary>Solution</summary>
      <p>
      true
      </p>
      </details>

   f. `{const f: Num} <: {let f: Num}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   g. `{let f: Num} <: {let f: Any}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   h. `{let f: Num} <: {const f: Any}`

      <details><summary>Solution</summary>
      <p>
      true
      </p>
      </details>

   i. `{const f: Num} => {const f: Any} <: {const f: Any} => {const f: Num}`

      <details><summary>Solution</summary>
      <p>
      false
      </p>
      </details>

   j. `{const f: Any} => {const f: Num} <: {const f: Num} => {const f: Any}`

      <details><summary>Solution</summary>
      <p>
      true
      </p>
      </details>

1. Compute the joins and meets of the following pairs of types `(t1, t2)`. 
   Indicate the cases where the meet does not exist.

   a. `t1 = {let f: Bool}`
      `t2 = {const g: Bool}`

      <details><summary>Solution</summary>
      <p>
      
      `join = {}`
      `meet = {let f: Bool, const g: Bool}`
      </p>
      </details>

   b. `t1 = {let f: Num}`
      `t2 = {let f: Bool}`

      <details><summary>Solution</summary>
      <p>
      
      `join = {const f: Any}`
      `meet = Nothing`
      </p>
      </details>

   c. `t1 = {let f: Num, let h: Bool}`
      `t2 = {const f: Bool, const k: {let h: Bool} }`

      <details><summary>Solution</summary>
      <p>
      
      `join = {const f: Any}`
      `meet = Nothing`
      </p>
      </details>

   d. `t1 = {let f: Any, let h: Bool}`
      `t2 = {const f: Bool, const k: {let h: Bool} }`

      <details><summary>Solution</summary>
      <p>
      
      `join = {const f: Any}`
      `meet = Nothing`
      </p>
      </details>

   e. `t1 = {let f: Num} => {let f: Num}`
      `t2 = {const f: Any} => {const f: Bool}`

      <details><summary>Solution</summary>
      <p>
      
      `join = {let f: Num} => {const f: Any}`
      `meet = {const f: Any} => Nothing`
      </p>
      </details>

   f. `t1 = {let f: Num} => {let f: {const: g: Bool}}`
      `t2 = {const f: Bool} => {const f: Any}`

      <details><summary>Solution</summary>
      <p>
      
      `join = Nothing => {const f: Any}`
      `meet = {const f: Any} => {let f: {const g: Bool}}`
      </p>
      </details>


1. Suppose we extend our type systems for object types with a new kind
   of field mutability: a field `f` with mutability `sink` is only
   allowed to be written to, but not to be read from. For instance, the
   following function should be well-typed:

   ```javascript
   (x: { sink f: Num, const g: Num }) => (x.f = 3, x.g)
   ```

   On the other hand, the next function should not be well-typed
   because it tries to read from the `sink` field `f`:

   ```javascript
   (x: { sink f: Num, const g: Num }) => x.f
   ```

   a. Provide new typing rules for field dereference expressions and
      field assignment expressions that take into account the new kind
      of mutability (the semantics of `const` and `let` fields should
      remain unchanged).
   
      ```
               ?
      ------------------TypeFldDeref
      Gamma |- e.f : tau

                  ?
      ------------------------ TypeFldDeref
      Gamma |- e1.f = e2 : tau
      ```
      
      <details><summary>Solution</summary>
      <p>
      
      The following solution relies on the subtyping rules provided in part b.
      
      ```
      Gamma |- e : tau'  tau' <: { const f: tau } 
      --------------------------------------------TypeFldDeref
      Gamma |- e.f : tau


      Gamma |- e1 : tau'  Gamma |- e2 : tau   tau' <: { sink f: tau } 
      -----------------------------------------------------------------TypeFldDeref
      Gamma |- e1.f = e2 : tau
      ```
      </p>
      </details>

      
   b. We can think of a `let` field as a combination of a `sink`
      field and a `const` field. Hence, it makes sense to extend our
      subtyping relation with the ability to *demote* a `let` field to
      a `sink` field, similar to our existing rule `SubObjMut`, which demotes a `let` field to a `const` field:
   
      ```
      { mut_1 f_1: tau_1, ..., let f: tau, ..., mut_n f_n: tau_n } 
      <:
      { mut_1 f_1: tau_1, ..., sink f: tau, ..., mut_n f_n: tau_n } 
      ```
      
      We can also introduce a form of depth subtyping for `sink` fields:
      
      ```
                                 ?
      --------------------------------------------------------
      { mut_1 f_1: tau_1, ..., sink f: tau, ..., mut_n f_n: tau_n } 
      <:
      { mut_1 f_1: tau_1, ..., sink f: tau', ..., mut_n f_n: tau_n } 
      ```
      
      What is the correct premise for this rule? Is it `tau <: tau'`
      or `tau' <: tau`? Only one of these two choices is
      correct. Provide a program that shows how the incorrect choice
      would break the soundness of the type system.
      
      <details><summary>Solution</summary>
      <p>
      
      The correct premise is `tau' <: tau`. That is, `sink` fields are typed contravariantly.
      The premise `tau <: tau'` would not be sound as the following example demonstrates:
      
      ```javascript
      const fun = (x: {sink f: {}}) => x.f = {};
      const y = {let f: {const g: 0 }};
      fun(y);
      y.f.g
      ```
      
      The program would be well-typed. In particular, the inferred
      type for `y` would be `{let f: {const g: Num}}` which according to
      the incorrect premise, would be a subtype of `{sink f: {}}`. 
      Hence the call `fun(y)` is well-typed. However, when the
      program is evaluated, the execution will get stuck when
      evaluating the final field dereference operation on field `g`.
      </p> 
      </details>
      
### Problem 5: State Monad

Consider the following function `fib` that computes the `n`-th Fibonacci
number.

```scala
def fib(n: Int): Int = if n <= 1 then n else fib(n - 1) + fib(n - 2)
```

The implementation of `fib` is quite naive. Its running time is exponential in `n` due to the two recursive calls. If we expand the recursive call `fib(n - 1)` we obtain:

```scala
if (n - 1) <= 1 then n - 1 else fib(n - 2) + fib(n - 3)
```

Thus, the second recursive call `fib(n - 2)` recomputes a result that
has already been computed in the first recursive call `fib(n - 1)`. A
simple idea to improve the implementation is to memoize the already
computed values of `fib` in a cache. Before we do the recursive calls,
we check whether the value is already in the cache to avoid
unnecessary recomputation. The following code implements this idea:

```scala
type Cache = Map[Int, Int]

def fib(n: Int) = fibMemo(Map.empty, n)._2

def fibMemo(m: Cache, n: Int): (Cache, Int) =
  if n <= 1 then (n, m) 
  else 
    m.get(n).map: 
      t => (t, m)
    .getOrElse:
      val (r, mr) = fibMemo(m, n - 1)
      val (s, ms) = fibMemo(mr, n - 2)
      val t = r + s
      (ms + (n -> t), t)
```

1. Complete the following variant of this implementation that uses a
   state monad to hide the threading of the memoization cache:

   ```scala
   def fib(n: Int) = fibMemo(n)(Map.empty)._2

   def fibMemo(n: Int): State[Cache,Int] =
     if n <= 1 
     then State.insert(n) else
       for
         c <- State.read { m: Cache => m.get(n) }
         t <- c.map(State.insert[Cache,Int]).getOrElse:
           for
             ???
           yield ???
         _ <- State.write { m: Cache => ??? }
       yield t
   ```

   **Disclaimer**: Arguably, unlike for our implementation of the
   interpreter, the monadic version of the function `fibMemo` does not
   improve over the non-monadic version in terms of code
   clarity. Also, there is a much more efficient way to compute the
   `fib` function that does not rely on memoization (see part 2 of the
   exercise below). The sole purpose of this exercise is to practice
   writing monadic code and understanding how it works.

    <details><summary>Solution</summary>
    <p>

    Let's start with the second version of the `fib` function (the one that
    uses the cache for memorizing the intermediate results):

    ```scala
    type Cache = Map[Int, Int]

    def fib(n: Int) = fibMemo(Map.empty, n)._2

    def fibMemo(m: Cache, n: Int): (Cache, Int) = 
      if n <= 1
      then (m, n) 
      else
        m.get(n).map(t => (m, t)).getOrElse:
          val (mr, r) = fibMemo(m, n - 1)
          val (ms, s) = fibMemo(mr, n - 2)
          val t = r + s
          (ms + (n -> t), t)
    ```

    You can see that the `fibMemo` function takes a cache state `m` as
    input (to store the mappings from natural numbers `n` to their
    Fibonacci number `fib(n)` that have already been computed). Each
    time `fibMemo` is called, it first checks whether the value for the
    given `n` is already stored in the cache. If not, it computes a
    value `t` (which is `fib(n)`) recursively. The recursive computation
    will produce a new cache state ms that now contains all mappings
    `(m -> fib(m))` for values `m < n`. The last step is to add the
    mapping for `n` and its Fibonacci number `t` to the cache and return
    the final cache state and the value `t`.

    So, altogether `fibMemo` is a computation that takes a cache state and a
    number `n`, and returns an updated cache state and a new number which is
    `fib(n)`. In a first step, we can rewrite the `fibMemo` function by
    currying the input of the cache state `m`:

    ```scala
    def fibMemo(n: Int): Cache => (Cache, Int) = 
      m: Cache =>
        if n <= 1 
        then (m, n) 
        else
          m.get(n).map( t => (m, t) ).getOrElse:
            val (mr, r) = fibMemo(n - 1)(m)
            val (mr, s) = fibMemo(n - 2)(mr)
            val t = r + s
            (ms + (n -> t), t)
   ```
   
   We can modify this a bit more by taking the function abstraction in
   the body of `fibMemo`, `m: Cache => ... `, and pushing it inside of
   each branch of the if expression:

   ```scala
   def fibMemo(n: Int): Cache => (Int, Cache) =
     if n <= 1
     then m: Cache => (m, n)
     else m: Cache =>
       m.get(n).map(t => (m, t)).getOrElse:
         val (mr, r) = fibMemo(n - 1)(m)
         val (ms, s) = fibMemo(n - 2)(mr)
         val t = r + s
         (ms + (n -> t), t)
   ```
  
   Note that `fibMemo` is now a function that returns a function of
   type `Cache => (Int, Cache)`. That is, the return value is a
   function that takes an input cache state and returns an output
   cache state and a result value of type `Int`. The state monad is
   essentially a container for functions of this shape. So we can
   obtain a first (trivial) monadic version of the `fibMemo` function by
   wrapping the function abstractions in each branch in a state monad:

   ```scala
   def fibMemo(n: Int): State[Cache, Int] =
     if n <= 1
     then State(m: Cache => (m, n))
     else State{m: Cache =>
       m.get(n).map( t => (m, t) ).getOrElse:
         val (mr, r) = fibMemo(n - 1)(m)
         val (ms, s) = fibMemo(n - 2)(mr)
         val t = r + s
         (ms + (n -> t), t)
     }
   ```
   
   While this version is technically a correct solution, it does not
   yet take advantage of the actual composition operations provided by
   the state monad data structure. Recall that the `map` and `flatMap`
   functions of state monads allow us to compose multiple functions of
   the shape `(Cache => (Cache, Int))` together, producing new functions
   of this shape. The composition makes the result value and output
   state of the first function become the input value and input state
   of the next function, and so forth.

   For example, let's take a look at the recursive call

   `fibMemo(n - 1)`

   in the else branch. This call now returns a `State[Cache,Int]` which
   hols a function of type `Cache => (Cache, Int)`. Let's refer to this
   function as `f`. If we call `f` with the input cache state `m`, we get back
   the result values `(mr, r)`. Instead of calling `f` directly with `m`, we
   can first create a new function that takes an `m`, calls `f` with that `m`,
   then passes the resulting cache state `mr` to the function contained in
   `fibMemo(n - 2)` to obtain `(ms, s)` from which we then compute `(ms, r + s)`:

   ```scala 
   m: Mem =>
     val (r, mr) = f(m)
     val (s, ms) = fibMemo(n - 2)(mr)
     (r + s, ms)
   ```
   
   Note that this function is again of type `Cache => (Cache, Int)`. Let's
   call this function `g`.

   We can then express `g` like this:

   ```scala
   fibMemo(n - 1).flatMap( r => fibMemo(n - 2).map( s => r + s ) )`
   ```
   
   `fibMemo(n - 1)` gives us a state monad that contains the function
   `f`. The subsequent `flatMap` and `map` operations compose `f` with
   the second `fibMemo` call and the final addition operation to obtain
   a state monad that contains the function `g`. If we now call the
   apply method of that state monad with an input cache state `m`, we
   get back the value `r + s` and the memory state `ms`.

   Using the for expression syntax, we can now express composition of
   these functions more elegantly:

   ```scala
   for
     r <- fibMemo(n - 1)
     s <- fibMemo(n - 2)
   yield r + s
   ```
   
   This expression gives us again a state monad that contains the
   function `g` just like the explicit `flatMap` and `map` calls.

   Now the else branch in the monadic version of the `fibMemo` function
   is complicated by the actual cache look up, which we can implement
   using `State.read`, and the final update of the cache, which we can
   implement using `State.write`. Though, it again uses `flatMap` and `map`
   operations (hidden in `for` expressions) to compose these operations
   with the state monad that computes the function `g` above:

   ```scala
   def fibMemo(n: Int): State[Cache,Int] = 
     if n <= 1
     then State.insert(n) 
     else
       for
         c <- State.read(m: Cache => m.get(n))
         t <- c.map(State.insert[Cache,Int]).getOrElse: 
                for
                  r <- fibMemo(n - 1)
                  s <- fibMemo(n - 2)
                yield r + s
         _ <- State.write(m: Cache => m + (n -> t))
       yield t
   ```
   
   Note that the call `State.insert(n)` in the 'then' branch of the conditional
   expression is equivalent to the expression

   `State(m: Cache => (m, n))`

   which is exactly the expression that we had obtained from the
   curried version of `fibMemo` when we first transformed it into the
   monadic version.

   The value `c` inside of the `for` expression of the else branch is a
   value of type `Option[Int]`. It is `Some(t)` where `t` is `fib(n)` if this
   value was already cached in the input cache state `m`. Otherwise, `c` is
   `None`. In the case when `c` is `Some(t)`, the expression

   ```scala
   c.map(State.insert[Cache,Int])
   ```
   
   is equivalent to the expression

   ```scala
   Some(State {m: Cache => (m, t)})
   ```
   
   That is, we extract `t` from `c` and wrap it again in a state monad
   so that we can keep composing the remaining steps with `flatMap` calls. The subsequent call to `getOrElse` then yields just
   `State(m: Cache => (m, t))`.

   If `c` is `None`, the above expression evaluates to `None`, so the
   subsequent call to `getOrElse` evaluates to the argument passed to
   `getOrElse`, i.e. a function that recursively computes `fib(n)`,
   wrapped in a state monad.

   The actual `fib` function is then obtained from `fibMemo` by
   calling `fibMemo(n)`, which gives a state monad that contains the
   stateful `fib` computation.  Then we call the `apply` method of that
   state monad with an empty initial cache state to obtain the result
   value and output cache state. Finally, we project that pair to the
   result value, discarding the final cache state:

   ```scala
   def fib(n: Int) = fibMemo(n)(Map.empty)._2
   ```
   
   </p> 
   </details>

2. A simpler and even more efficient solution is to make the function
   `fib` tail-recursive so that it only requires a single recursive
   call. Complete the following code to obtain such a tail-recursive
   implementation. Your implementation should run in constant space and
   in time `O(n)`:

   ```scala
   def fib(n: Int) = fibTail(n, ???)

   def fibTail(n: Int, ???): Int =
     ???
   ```
   
   Hint: You may need to add more than one parameter to `fibTail` to
   accumulate intermediate results across recursive calls. If you have
   trouble coming up with a solution, then first think about how to
   implement the `fib` function using a single while loop. Then
   transform the loop into a tail-recursive function.
   
   <details><summary>Solution</summary>
   <p>

   ```scala
   def fib(n: Int) = fibTail(n, 0, 1, 0)

   def fibTail(n: Int, i: Int, r: Int, s: Int): Int =
     if i == n then s else fibTail(n, i + 1, r + s, r)
   ```

   The tail-recursive implementation of `fib` exploits the observation
   that we actually only need to memoize two values for each
   recursive step. That is, we introduce two accumulator arguments `r`
   and `s` to the recursive function such that we maintain the following
   invariant: in the `i`-th recursive call to `fibTail`, we have 
   
   `r == fib(i+1)` and `s == fib(i)`. 
   
   Thus, in the initial call when `i == 0` we must have `r == fib(0) == 0` and `s == fib(1) == 1`,
   and in the final call when `i == n` we have `s == fib(n)` for the initial
   value of `n`.

   </p> 
   </details>
