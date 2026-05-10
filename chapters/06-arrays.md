# Chapter 6 — Arrays

> Commits covered: `8b6395d`, `3ce1b2d`, `648646b`, `3e55caf`, `0b76634`. Five commits. The first four add arrays, multi-dimensional arrays, the subscript operator, and `sizeof`; the fifth merges the `Function` and `Obj` types in preparation for global variables.

Chapter 5 finished with chibicc compiling recursive `fib`. The compiler had functions, parameters, calls, and a type system small enough to fit into one struct definition: `int`, "pointer to something," and "function returning something." What it didn't have was anything you could put more than one of in a row. Every value occupied exactly eight bytes and lived at one specific frame slot. There was no way to allocate a contiguous block, no way to step through it with a subscript, and no way for the compiler to talk about how big anything was.

Chapter 6 fixes all four of those at once. The first commit introduces 1-D arrays, which forces three things into existence: a `TY_ARRAY` type kind, a `size` field on every type (so `Type` finally knows how big the things it describes are), and a small but conceptually large change in the codegen — when an array appears in an expression, its name evaluates to its address rather than its contents. That's the C language's "array-to-pointer decay" rule, and chibicc implements it as a single special case in a new `load` helper.

The other four commits build on the first.

1. `8b6395d` adds the type kind, the size machinery, and the decay rule.
2. `3ce1b2d` extends the array grammar to allow nesting (`int x[2][3]`).
3. `648646b` adds the `[]` operator. It's a parser-only commit: `x[y]` desugars to `*(x+y)`, and codegen never sees a subscript.
4. `3e55caf` adds `sizeof`. It's also parser-only: `sizeof e` evaluates `e`'s type, looks up its size, and emits an integer literal.
5. `0b76634` is a "no functional change" refactor: the `Function` and `Obj` types collapse into a single `Obj` that knows whether it's a function, a local, or (next chapter) a global.

The chapter has one concept interlude, on **array-to-pointer decay** — what C means by it, why the language is built around it, and how chibicc's two-line implementation actually captures the rule. It lands between §6.1 (where the rule first appears in the codegen) and §6.2 (where multi-dimensional arrays start to exercise it).

---

## 6.1 — One-dimensional arrays

> `git checkout 8b6395d0f2be4024bd7e7921157a6496951eb162` — *Add one dimensional arrays*

This is the chapter's heaviest commit, around 100 lines added across all five source files. After it, programs like

```c
int main() { int x[3]; *x=3; *(x+1)=4; *(x+2)=5; return *(x+1); }
```

compile and produce 4. The user can declare an array, write into successive elements via pointer arithmetic, and read them back. The `[]` operator doesn't exist yet — that's two commits away — so for now arrays are written and read entirely through `*(x + n)`.

Three pieces of machinery have to land in this commit: the array type itself, sizes that aren't all 8, and the array-to-pointer decay rule.

### The `Type` struct grows up

```diff
 typedef enum {
   TY_INT,
   TY_PTR,
   TY_FUNC,
+  TY_ARRAY,
 } TypeKind;

 struct Type {
   TypeKind kind;
-
-  // Pointer
+  int size;      // sizeof() value
+
+  // Pointer-to or array-of type. We intentionally use the same member
+  // to represent pointer/array duality in C.
+  ...
   Type *base;

   // Declaration
   Token *name;

+  // Array
+  int array_len;
+
   // Function type
```

Three additions, all overdue.

`TY_ARRAY` joins the type-kind enum. An array's element type goes in `base` — the same `base` field a pointer uses to record what it points at — and the comment block above the field is doing real work. C, by design, blurs arrays and pointers together; the language treats `int x[3]` as a thing that points at `int`s, almost everywhere. Storing both kinds' element type in the same `base` member is more than a code-saving trick. It's the data structure mirroring the language semantics.

The Chapter 4 prose flagged this already, in a forward-leaning way: the test `lhs->ty->base` was described as "the thing that points at something" rather than as a pointer-specific test. That description pays off now, because the same condition that detects pointers in `new_add` and `new_sub` will detect arrays automatically. We won't have to teach pointer arithmetic about arrays separately. (We'll see how this plays out at the end of this section.)

`size` is the new field that finally puts a number on a type. `int` is 8 (chibicc still uses 64-bit ints), pointers are 8 (any pointer is just an address, regardless of what it points at), and an array's size is element-size times length:

```c
Type *array_of(Type *base, int len) {
  Type *ty = calloc(1, sizeof(Type));
  ty->kind = TY_ARRAY;
  ty->size = base->size * len;
  ty->base = base;
  ty->array_len = len;
  return ty;
}
```

The recursion is what makes nesting work for free. An array of arrays is an array whose `base` is itself an array, and whose size is the (already-computed) size of the inner array times the outer length. We won't show this off until §6.2, but the construction is general enough that no further code is needed when the time comes.

`array_len` is the element count, separately stored so that `sizeof` can be computed without dividing `size` by `base->size`. (The two are redundant for arrays, but each is the natural answer to a different question.)

The other two `Type` constructors get `size` assignments too:

```diff
 Type *pointer_to(Type *base) {
   Type *ty = calloc(1, sizeof(Type));
   ty->kind = TY_PTR;
+  ty->size = 8;
   ty->base = base;
   return ty;
 }
```

and `ty_int` itself:

```diff
-Type *ty_int = &(Type){TY_INT};
+Type *ty_int = &(Type){TY_INT, 8};
```

Compound literal syntax — we initialize the second field implicitly by position. (`func_type` doesn't get a size; a function type isn't a value, you can't take `sizeof` of one, and the `0` it ends up with is fine because the field is never consulted for `TY_FUNC`.)

### The hardcoded `8`s finally fall

This is the moment Chapter 4's foreshadowing pays off. The pointer-arithmetic helpers in `parse.c` get a small but earned change:

```diff
   // ptr + num
-  rhs = new_binary(ND_MUL, rhs, new_num(8, tok), tok);
+  rhs = new_binary(ND_MUL, rhs, new_num(lhs->ty->base->size, tok), tok);
   return new_binary(ND_ADD, lhs, rhs, tok);
 }
```

and the same change in two more places in `new_sub` (for `ptr - num` and `ptr - ptr`). The literal 8 — flagged as a temporary back in §4.3 — is gone, replaced by a per-type lookup. `lhs->ty->base->size` reads as "the size of one thing that this pointer points at." For `int *`, that's 8 (still). For `int (*)[3]`, that's 24 — three ints. For an array, it's the element size, because arrays and pointers share the `base` field.

This is the change Chapter 4 forecast and Chapter 5 re-flagged. We're not going to dwell on it; the previous chapters did the explanation, this one shows the line. From this commit on, pointer arithmetic on any type works correctly because the size lookup goes through the type system instead of being hardcoded.

A frame allocator change rides along:

```diff
 static void assign_lvar_offsets(Function *prog) {
   for (Function *fn = prog; fn; fn = fn->next) {
     int offset = 0;
     for (Obj *var = fn->locals; var; var = var->next) {
-      offset += 8;
+      offset += var->ty->size;
       var->offset = -offset;
     }
     fn->stack_size = align_to(offset, 16);
   }
 }
```

Each local now consumes its own size in the frame, not a fixed 8. An `int x[3]` reserves 24 bytes; the next variable's offset is `-24` lower than `x`'s. The 16-byte alignment of the per-function `stack_size` is unchanged — it's still rounded up at the end — but the per-variable arithmetic is type-aware.

This is a quiet but important moment. Until this commit, every variable in chibicc was the same size, so the frame allocator could be a counter that incremented by 8. The first time a chibicc program declares `int x[3]`, the allocator has to think; the change is one line, and it's been earned by adding `size` to `Type`.

### The parser learns the `[N]` declarator suffix

```diff
-// type-suffix = ("(" func-params? ")")?
-// func-params = param ("," param)*
+// func-params = (param ("," param)*)? ")"
 // param       = declspec declarator
-static Type *type_suffix(Token **rest, Token *tok, Type *ty) {
-  if (equal(tok, "(")) {
-    tok = tok->next;
+static Type *func_params(Token **rest, Token *tok, Type *ty) {
+  Type head = {};
+  Type *cur = &head;
+
+  while (!equal(tok, ")")) {
+    if (cur != &head)
+      tok = skip(tok, ",");
+    Type *basety = declspec(&tok, tok);
+    Type *ty = declarator(&tok, tok, basety);
+    cur = cur->next = copy_type(ty);
+  }

-    Type head = {};
-    Type *cur = &head;
+  ty = func_type(ty);
+  ty->params = head.next;
+  *rest = tok->next;
+  return ty;
+}

-    while (!equal(tok, ")")) {
-      if (cur != &head)
-        tok = skip(tok, ",");
-      Type *basety = declspec(&tok, tok);
-      Type *ty = declarator(&tok, tok, basety);
-      cur = cur->next = copy_type(ty);
-    }
+// type-suffix = "(" func-params
+//             | "[" num "]"
+//             | ε
+static Type *type_suffix(Token **rest, Token *tok, Type *ty) {
+  if (equal(tok, "("))
+    return func_params(rest, tok->next, ty);

-    ty = func_type(ty);
-    ty->params = head.next;
-    *rest = tok->next;
-    return ty;
+  if (equal(tok, "[")) {
+    int sz = get_number(tok->next);
+    *rest = skip(tok->next->next, "]");
+    return array_of(ty, sz);
   }
```

Two structural moves at once. The function-parameter parsing is extracted into its own `func_params` helper so `type_suffix` itself can dispatch on what follows the identifier. After the refactor, `type_suffix` is a three-way switch: a `(` means function parameters; a `[` means an array dimension; anything else means we're done with the suffix.

The `[N]` form is small: read a number, skip the `]`, wrap the type in `array_of`. The `get_number` helper is new and shaped like `get_ident`:

```c
static int get_number(Token *tok) {
  if (tok->kind != TK_NUM)
    error_tok(tok, "expected a number");
  return tok->val;
}
```

Chibicc's array dimensions are compile-time integer constants. Not expressions, not constant expressions involving operators or `sizeof`, just a single numeric literal. C99 has variable-length arrays, but chibicc isn't going there for many chapters. `int x[5]` is legal; `int x[5+1]` is not.

The grammar comment is worth pausing on. `type-suffix = "(" func-params | "[" num "]" | ε` is a flat three-way alternation in this commit. The next commit, `3ce1b2d`, will make the array case recurse — `[N]` followed by another `type-suffix` — to allow `int x[2][3]`. We'll see that in §6.2.

### Codegen: `load` and `store` get extracted, and `load` learns about arrays

The codegen changes look like a refactor — `mov (%rax), %rax` is extracted into a `load` helper, and `pop %rdi; mov %rax, (%rdi)` into a `store` helper:

```diff
+// Load a value from where %rax is pointing to.
+static void load(Type *ty) {
+  if (ty->kind == TY_ARRAY) {
+    // If it is an array, do not attempt to load a value to the
+    // register because in general we can't load an entire array to a
+    // register. As a result, the result of an evaluation of an array
+    // becomes not the array itself but the address of the array.
+    // This is where "array is automatically converted to a pointer to
+    // the first element of the array in C" occurs.
+    return;
+  }
+
+  printf("  mov (%%rax), %%rax\n");
+}
+
+// Store %rax to an address that the stack top is pointing to.
+static void store(void) {
+  pop("%rdi");
+  printf("  mov %%rax, (%%rdi)\n");
+}
```

Then both `ND_VAR` and `ND_DEREF` call `load(node->ty)` instead of emitting the `mov` directly, and `ND_ASSIGN` calls `store()`. So far this looks like cleanup. But look at `load`'s body. It has a special case for arrays: *don't load*. If the type is `TY_ARRAY`, the function returns without emitting any instruction — leaving whatever is in `%rax` (the address of the array, which `gen_addr` just put there) untouched.

This is the entire mechanism by which arrays decay to pointers in chibicc. When the compiler emits code for `x` (an array variable), it walks the same path as any other variable: `gen_addr` computes `lea offset(%rbp), %rax`, then `load(node->ty)` is called. For an `int`, `load` emits `mov (%rax), %rax` — that's the second step that turns "address of x" into "value of x." For an array, `load` skips the second step, and "address of x" stays in `%rax`. The expression's value *is* the address of the array.

In C terms: `x` and `&x[0]` produce the same register state, because chibicc never goes through the "load the value of x" step for an array. There's no value to load. The address is the result.

Rui's source comment captures the intuition: "in general we can't load an entire array to a register." A 24-byte array won't fit in `%rax`; even for a 1-element array it would be wrong to load just the first element. The decay rule matches what the hardware can actually do — registers hold scalars, so an array-typed expression's "value" has to be its address.

### `add_type` learns three new cases

The type-deriver in `type.c` gains decay-aware logic in three places. First, the `ND_ADDR` case:

```diff
   case ND_ADDR:
-    node->ty = pointer_to(node->lhs->ty);
+    if (node->lhs->ty->kind == TY_ARRAY)
+      node->ty = pointer_to(node->lhs->ty->base);
+    else
+      node->ty = pointer_to(node->lhs->ty);
     return;
```

`&x` where `x` is `int[3]` should produce `int *`, not `int (*)[3]`. The C standard's rule: when you take the address of an array, you get a pointer to the first element, not a pointer to the whole array. Chibicc handles this by stripping one level off the type before wrapping it in `pointer_to`. (Real C distinguishes `int *` from `int (*)[3]` and they produce the same address but have different types; chibicc collapses them to the former. This will eventually need fixing for proper multi-dimensional array semantics, but it works for the tests we have.)

Second, the `ND_DEREF` case:

```diff
   case ND_DEREF:
-    if (node->lhs->ty->kind != TY_PTR)
+    if (!node->lhs->ty->base)
       error_tok(node->tok, "invalid pointer dereference");
     node->ty = node->lhs->ty->base;
     return;
```

The check changes from "is it a pointer?" to "does it have a base?" — and arrays have a base, so they're now dereferenceable. This is the same `lhs->ty->base` trick from `new_add`, generalized to work uniformly for both pointers and arrays. After this change, `*x` where `x` is an `int[3]` produces an `int`-typed expression, exactly like `*p` where `p` is `int *`.

Third, the `ND_ASSIGN` case grows a guard:

```diff
   case ND_ASSIGN:
+    if (node->lhs->ty->kind == TY_ARRAY)
+      error_tok(node->lhs->tok, "not an lvalue");
     node->ty = node->lhs->ty;
     return;
```

You can't assign to an array. `int x[3]; int y[3]; x = y;` is illegal in C — the array name is not an lvalue, only its elements are. Chibicc enforces this by special-casing arrays in `ND_ASSIGN`'s type-deriver. The error message (`"not an lvalue"`) is technically a misnomer — an array name is an lvalue in C's terms, just not a *modifiable* one — but the spirit is right.

A small structural move comes with this: the `ND_ASSIGN` case used to share its body with `ND_ADD`/`ND_SUB`/`ND_MUL`/`ND_DIV`/`ND_NEG` (a fall-through case list). It splits out so the array check can run before `node->ty = node->lhs->ty`.

### Tests

```diff
+assert 3 'int main() { int x[2]; int *y=&x; *y=3; return *x; }'
+
+assert 3 'int main() { int x[3]; *x=3; *(x+1)=4; *(x+2)=5; return *x; }'
+assert 4 'int main() { int x[3]; *x=3; *(x+1)=4; *(x+2)=5; return *(x+1); }'
+assert 5 'int main() { int x[3]; *x=3; *(x+1)=4; *(x+2)=5; return *(x+2); }'
```

Four tests. The first one is the most interesting — it's exercising decay directly. `int *y = &x` takes the address of `x` (which decays to `int *`, by the `ND_ADDR` rule), and assigns it to a regular pointer. Then `*y = 3` writes through the pointer, and `*x` reads through the array — both end up reading the same memory location (the first element of `x`), so the test returns 3.

The other three are pointer-arithmetic-through-array-name tests. `*(x+1)` works because `x` has type `int[2]` (well, `int[3]` in those tests), `x` decays to `int *`, `x + 1` adds `sizeof(int) = 8` bytes (not 1) thanks to the now-de-hardcoded `new_add`, and the resulting address points at the second element. Writing to it stores 4 there; reading from it gets 4 back.

These tests are doing a lot of work for their length. Every line exercises decay, pointer arithmetic, and the new size-aware frame allocator. If any one of those were broken, the tests would fail in different ways.

### Where we are

Arrays exist. `int x[N]` declares a contiguous block of `N` `int`s in the frame; `x` evaluates to the address of the first element thanks to `load`'s array special case; pointer arithmetic on `x` scales correctly thanks to the now-typed `new_add`; and you can write through `*x`, `*(x+1)`, etc., to fill the elements one at a time.

We've also accidentally landed two pieces of machinery that the rest of the chapter is going to lean on. The `size` field on `Type` is what `sizeof` is going to read in §6.4. The `array_of` constructor's recursive shape (an array's base can be another array) is what §6.2 is going to exercise to allow nesting. Neither needs new code; both fall out of doing §6.1 right.

Before the next commit gives us multi-dimensional arrays, let's pause to look at the decay rule itself, since it's the trick that makes this whole chapter possible.

---

## Concept interlude — Array-to-pointer decay

In C, the expression `x` — where `x` was declared as `int x[3]` — has type `int *` and evaluates to `&x[0]`, almost everywhere. The "almost" is doing a lot of work in that sentence, and the pattern is so pervasive in C that the language is sometimes described as having "no real arrays, only declarators that happen to allocate space." That's an exaggeration, but a useful one.

The official rule, from C11 §6.3.2.1: "an expression that has type 'array of type' is converted to an expression with type 'pointer to type' that points to the initial element of the array object." There are exceptions: when the array is the operand of `sizeof`, when it's the operand of unary `&`, when it's a string literal used to initialize an array, and (in C11) when it's the operand of `_Alignof`. Everywhere else, an array name in expression context decays to a pointer to its first element.

This is not just a syntactic shortcut. It's the reason C can pass arrays to functions cheaply: a parameter declared `int a[]` is silently rewritten to `int *a`, and the call site passes the address of the first element. It's the reason `arr[i]` and `*(arr + i)` are interchangeable: the subscript operator is defined in terms of pointer arithmetic on the decayed array. It's the reason `strlen("hello")` works: the string literal is an array of `char`, but in the function-call context it decays to `char *`. The rule is woven into so many corners of C that any compiler has to get it right or fail in lots of small ways simultaneously.

### What chibicc actually has to do

Implementing decay sounds intricate, but chibicc gets away with two observations.

**Observation one: in expression context, the natural code-generation path for an lvalue is already "compute the address, then optionally load."** Every variable read is two steps in chibicc's codegen — `gen_addr` puts the variable's address in `%rax`, then a `mov (%rax), %rax` reads through it. The "optionally" is the new thing. For scalars, both steps run; for arrays, only the first one does. If we skip the load for `TY_ARRAY`, the result of the expression is automatically the address. No further code change is needed; the rest of the pipeline naturally consumes "an address in `%rax`" as if it were a pointer's value.

That's `load`'s array special case in §6.1. One `if` statement implements decay for every site where an array name appears in an expression: variable reads, dereferences, anything that flows into the standard `load` path.

**Observation two: the `&` and `sizeof` exceptions can be handled at the type-derivation level.** For `&`, chibicc adjusts `add_type`'s `ND_ADDR` case to strip one level off arrays before wrapping in `pointer_to`. So `&x` where `x` is `int[3]` produces a `pointer_to(int)` rather than `pointer_to(array_of(int, 3))`. The codegen, meanwhile, doesn't change: `&` of an array still goes through `gen_addr`, which leaves the array's address in `%rax`. The address is the same; only the type differs.

For `sizeof` (we'll see this in §6.4), the exception handles itself almost trivially: `sizeof` reads the operand's *type*, not its value. Since chibicc preserves the array type up until the moment it would `load` (and `sizeof` doesn't go through `load`), the operand type for `sizeof(x)` is still `int[3]`, with `size` 24, and that's what `sizeof` returns.

### What chibicc doesn't yet capture

Real C makes a finer distinction than chibicc currently does. The type `int (*p)[3]` (a pointer to an array of three ints) is not the same as `int *p` (a pointer to int), even though `&x` for an `int x[3]` produces something that *could* legally be either. C says it produces the latter — the rule strips the array down to its element type — but the former type is reachable through nested declarators and through array-of-array element pointers.

Chibicc, today, has no syntax for `int (*p)[3]` — its declarator parser doesn't yet handle parenthesized declarators. That's a Chapter 10 concern. For now, chibicc's ability to write a nested-pointer-to-array type is zero, and the simplification "any address-of-array is just address-of-first-element" is sufficient. When Chapter 10 lands the full declarator zoo, the distinction will start to matter, and `add_type`'s `ND_ADDR` case will get a follow-up. We're noting the limitation here so that when it gets fixed, the foreshadowing is in place.

The other corner chibicc doesn't address is array decay across function boundaries. C's rule that `void f(int a[])` is rewritten to `void f(int *a)` exists because the language wants array parameters to be cheap (a single pointer, not a copy of the array). Chibicc doesn't have any types in its parameter list besides `int` and pointer, so the question doesn't arise yet — there's no way to write `int a[]` as a parameter type, because `[N]` is the only array suffix the parser accepts. Globals (Chapter 7) and full declarator syntax (Chapter 10) will eventually expose this.

### Decay as a small implementation

The point of this interlude is that C's decay rule, which can sound like a tangle of exceptions, becomes very simple in chibicc's implementation. Two if-statements (`load`'s `TY_ARRAY` skip and `add_type`'s `ND_ADDR` strip), one shared `base` field that arrays and pointers cohabit, and the rule falls out almost on its own. Multi-dimensional arrays in §6.2 will exercise this, and we'll see decay applied recursively — `x` of type `int[2][3]` decays to `int (*)[3]` (or rather, in chibicc's simplification, to `int *`), and `*x` of type `int[3]` decays in turn to `int *`. Each `load` skipping each `TY_ARRAY` works the same way at every level.

The interlude is shorter than Chapter 5's calling-convention one because there's less to say. Decay isn't a contract between two parties; it's a single rule the compiler applies to its own AST, and the rule has a small implementation. We just have to know it's there, because §6.2 and §6.3 are about to lean on it without further comment.

---

## 6.2 — Arrays of arrays

> `git checkout 3ce1b2d067164f754dcb4216c193dc98e164b3ce` — *Add arrays of arrays*

The smallest commit in the chapter. Five lines of parser change, seven new tests. After it, `int x[2][3]` parses as "array of 2 arrays of 3 ints," and the compiler handles it correctly.

### The grammar grows one symbol

```diff
 // type-suffix = "(" func-params
-//             | "[" num "]"
+//             | "[" num "]" type-suffix
 //             | ε
 static Type *type_suffix(Token **rest, Token *tok, Type *ty) {
   if (equal(tok, "("))
     return func_params(rest, tok->next, ty);

   if (equal(tok, "[")) {
     int sz = get_number(tok->next);
-    *rest = skip(tok->next->next, "]");
+    tok = skip(tok->next->next, "]");
+    ty = type_suffix(rest, tok, ty);
     return array_of(ty, sz);
   }

   *rest = tok;
   return ty;
 }
```

Two lines change. After consuming `[N]`, instead of returning, the parser recurses into `type_suffix` to consume any further `[M]` suffixes. The result: `int x[2][3]` parses as

1. `declspec` consumes `int` → `ty = int`.
2. `declarator` consumes `*`s (none) and the identifier `x`.
3. `type_suffix` sees `[`, reads `2`, consumes `]`, then recurses.
4. Recursive `type_suffix` sees `[`, reads `3`, consumes `]`, then recurses.
5. Recursive-recursive `type_suffix` sees `;`, falls through to `ε`, returns `ty = int`.
6. Back in step 4: `array_of(int, 3) → int[3]`, returned.
7. Back in step 3: `array_of(int[3], 2) → int[2][3]`, returned.

The order matters. C's array-dimension layout reads outside-in but builds inside-out: `int x[2][3]` is "an array of 2, where each element is an array of 3 ints." The recursion above is what implements that — the innermost `type_suffix` returns first, becoming the base, and the outer `array_of` wraps it. The total size, computed by `array_of`, is `int_size × 3 × 2 = 48`, and `assign_lvar_offsets` reserves 48 bytes in the frame.

That's the whole code change. The tests do the rest of the work.

### Tests as documentation

```diff
+assert 0 'int main() { int x[2][3]; int *y=x; *y=0; return **x; }'
+assert 1 'int main() { int x[2][3]; int *y=x; *(y+1)=1; return *(*x+1); }'
+assert 2 'int main() { int x[2][3]; int *y=x; *(y+2)=2; return *(*x+2); }'
+assert 3 'int main() { int x[2][3]; int *y=x; *(y+3)=3; return **(x+1); }'
+assert 4 'int main() { int x[2][3]; int *y=x; *(y+4)=4; return *(*(x+1)+1); }'
+assert 5 'int main() { int x[2][3]; int *y=x; *(y+5)=5; return *(*(x+1)+2); }'
```

Six tests, each exercising a different addressing form into the same 6-element 2-D layout. Read them as a table:

| Test | Index | Address | Read-back form |
|---|---|---|---|
| 0 | `[0][0]` | `y+0` | `**x` |
| 1 | `[0][1]` | `y+1` | `*(*x+1)` |
| 2 | `[0][2]` | `y+2` | `*(*x+2)` |
| 3 | `[1][0]` | `y+3` | `**(x+1)` |
| 4 | `[1][1]` | `y+4` | `*(*(x+1)+1)` |
| 5 | `[1][2]` | `y+5` | `*(*(x+1)+2)` |

The `int *y = x` line is doing something subtle. `x` has type `int[2][3]`; assigning it to an `int *` triggers two levels of decay. First, `x` (as an expression) decays from `int[2][3]` to "pointer to first element," which in chibicc is `int *`. (C would say "pointer to `int[3]`," but chibicc's decay simplification collapses to `int *`.) Then the `int *y` assignment binds the result. After this, `y` and `x` point at the same byte — the first byte of the 48-byte block — but with different types. `y + 1` advances by 8 bytes (sizeof int, because `y` is `int *`). `x + 1` advances by 24 bytes (sizeof one row, because `x`'s base is `int[3]` — *its* size is 24).

That's why the read-back forms differ. `y+5` lands on byte 40, the start of the sixth integer. `*(x+1)` lands on byte 24, the start of the second row, and `*(x+1)+2` advances within that row to byte 24+16 = byte 40. Same byte, two different paths.

The tests confirm that the size-aware pointer arithmetic from §6.1 works at every level of nesting. `x + 1` scales by `sizeof(int[3]) = 24` because `array_of(int, 3)->size` is 24 because `int->size` is 8 — the sizes propagate up through the constructor recursion, and the arithmetic uses them automatically.

### Where we are

The compiler handles arbitrarily-nested array types with no further code changes. A `int x[5][6][7][8]` would also work; chibicc just hasn't been tested that deep. The next commit reaches for ergonomics: writing `*(*(x+1)+2)` to read `x[1][2]` is correct but eye-watering. Time for the `[]` operator.

---

## 6.3 — The `[]` operator

> `git checkout 648646bba704745274fcd4fef3b7029c7f7e0fcd` — *Add `[]` operator*

After this commit, `x[1][2]` works as a synonym for `*(*(x+1)+2)`. The codegen never sees a "subscript" node; the parser quietly rewrites every `[]` into a dereference of an addition. Twenty lines added to `parse.c`, thirteen new tests.

### A new grammar level: `postfix`

The grammar grows one production:

```diff
 // unary = ("+" | "-" | "*" | "&") unary
-//       | primary
+//       | postfix
 static Node *unary(Token **rest, Token *tok) {
   ...
-  return primary(rest, tok);
+  return postfix(rest, tok);
 }
+
+// postfix = primary ("[" expr "]")*
+static Node *postfix(Token **rest, Token *tok) {
+  Node *node = primary(&tok, tok);
+
+  while (equal(tok, "[")) {
+    // x[y] is short for *(x+y)
+    Token *start = tok;
+    Node *idx = expr(&tok, tok->next);
+    tok = skip(tok, "]");
+    node = new_unary(ND_DEREF, new_add(node, idx, start), start);
+  }
+  *rest = tok;
+  return node;
+}
```

`postfix` slots into the grammar between `unary` and `primary`. Its job: parse a primary expression, then optionally consume any number of `[expr]` suffixes. Each suffix wraps the current node in `*(node + idx)`, which is exactly what C says `node[idx]` means.

The `while` loop is what makes chained subscripts work. `x[1][2]` parses as:

1. `primary` returns `x`.
2. First iteration: `[1]` is consumed; node becomes `*(x+1)`.
3. Second iteration: `[2]` is consumed; node becomes `*(*(x+1)+2)`.
4. No more `[`, loop exits.

The result is the same AST a user would have written by hand with explicit dereferences. Codegen has nothing new to learn — it sees the `ND_DEREF`, `ND_ADD`, `ND_VAR`, `ND_NUM` shape it has handled since Chapter 4.

### A pattern worth naming

This is the moment to name something the book has been pointing at for a while. Chibicc has a recurring discipline: when there are multiple surface syntaxes that mean the same thing, the parser collapses them to a single shape and the codegen sees only the canonical form. We've now seen four instances of this *canonicalization-at-parse-time*:

1. `>` and `>=` are rewritten as `<` and `<=` with operands swapped (Chapter 3, §3.4).
2. `while (e) s` is rewritten as `for (; e; ) s` — a degenerate for-loop with no init or increment (Chapter 3, §3.9).
3. `p + n` (where `p` is a pointer and `n` is an integer) is rewritten as `p + (n * sizeof(*p))` (Chapter 4, §4.3).
4. `x[y]` is rewritten as `*(x + y)` (this section).

Plus a near-miss in Chapter 4: initializers are rewritten to declaration-plus-assignment (`int x = 3;` becomes `int x; x = 3;`).

The pattern's effect is always the same: the codegen handles a smaller language than the parser accepts. Add support for `>` to the parser and it costs nothing in codegen, because the codegen already handles `<`. Add `[]` and the codegen doesn't change. This is the principle that lets chibicc keep its codegen short — about 200 lines at this commit, not much more than that even at the end of the book — while the parser grows to handle most of C.

The effect compounds. By Chapter 12, when chibicc supports compound initializers, struct member access via `->`, the `+=` family of compound assignments, and a half-dozen other surface forms, the codegen still won't have grown proportionally. Each new surface form gets desugared on its way in, and the codegen sees only the basic operators.

There's a dual cost: the parser knows about more than syntax. It has to know that `x[y]` *means* dereference-of-add, not just "parse-and-emit-an-ND_SUBSCRIPT." That's an ergonomic trade-off — the parser does more work; the rest of the compiler does less. Over the book, Rui consistently picks the parser-does-more side. He's said elsewhere that he likes the codegen to stay simple because debugging codegen is harder than debugging the parser. Canonicalize on the way in, and the bug-hunting surface area at the back end stays manageable.

### `2[x]` works for free

The most fun test in this commit:

```sh
assert 5 'int main() { int x[3]; *x=3; x[1]=4; 2[x]=5; return *(x+2); }'
```

`2[x]` is legal C. It expands to `*(2 + x)`, and because `+` is commutative on integers and pointers, `2 + x` and `x + 2` are the same value. Many C programmers have never written `2[x]` and never will; some C textbooks treat it as a curiosity worth teaching once. Chibicc supports it not because of any conscious decision — the test confirms it works, but Rui didn't add code to make it work — but because of how `new_add` is built.

Recall §4.3: `new_add` checks both operand types, swaps them if the pointer is on the right, and then scales the integer. `2[x]` after the parser builds the AST is `*(2 + x)`. When type derivation reaches the `+`, the swap-canonicalization moves `x` to the left and the AST becomes `*(x + 2)`. From the codegen's perspective, `2[x]` and `x[2]` produce identical assembly.

This kind of accidental correctness is a small joy of building on canonicalization. The parser desugars `[]` to `+`-and-`*`, and `+` already swaps to put pointers on the left. Two transformations compose to give correct behavior on a syntactic case nobody wrote code for.

### Tests for nested subscripts

```diff
+assert 0 'int main() { int x[2][3]; int *y=x; y[0]=0; return x[0][0]; }'
+assert 1 'int main() { int x[2][3]; int *y=x; y[1]=1; return x[0][1]; }'
+assert 2 'int main() { int x[2][3]; int *y=x; y[2]=2; return x[0][2]; }'
+assert 3 'int main() { int x[2][3]; int *y=x; y[3]=3; return x[1][0]; }'
+assert 4 'int main() { int x[2][3]; int *y=x; y[4]=4; return x[1][1]; }'
+assert 5 'int main() { int x[2][3]; int *y=x; y[5]=5; return x[1][2]; }'
```

These six are exactly the tests from §6.2 with the `*(...)` and `*(*(...)+...)` rewritten as `[...]` subscripts. The `y[0..5]` writes use the flat-pointer view; the `x[i][j]` reads use the 2-D view. Every test passes, confirming that single-level and double-level subscripts both go through the desugaring correctly.

### Where we are

The user can write idiomatic array code: `x[i]` for 1-D, `x[i][j]` for 2-D, with the natural read-it-left-to-right precedence falling out of the recursive grammar. The compiler still has no notion of array bounds; `x[100]` for an `int x[3]` will quietly compute an address 800 bytes past `x` and read or write whatever happens to be there. This is consistent with C and with chibicc's general lack-of-runtime-checks posture.

The other thing the user still can't do is ask the compiler "how big is `x`?" Pointer arithmetic uses size internally; the user can't see those sizes. The next commit fixes that.

---

## 6.4 — `sizeof`

> `git checkout 3e55cafef80f0fc9d74bb06ea174de4b53e2ef94` — *Add sizeof*

A small commit that's bigger in concept than in code. After it, `sizeof e` returns the byte-size of `e`'s type as a compile-time constant. The implementation is six new lines of parser, three lines of tokenizer, and twelve new tests. Codegen doesn't change at all.

### `sizeof` as a primary

```diff
-// primary = "(" expr ")" | ident func-args? | num
+// primary = "(" expr ")" | "sizeof" unary | ident func-args? | num
 static Node *primary(Token **rest, Token *tok) {
   if (equal(tok, "(")) {
     ...
   }

+  if (equal(tok, "sizeof")) {
+    Node *node = unary(rest, tok->next);
+    add_type(node);
+    return new_num(node->ty->size, tok);
+  }
+
   if (tok->kind == TK_IDENT) {
```

Five lines. We see `sizeof`, we parse the operand as a unary expression (matching C's precedence rule that `sizeof` binds tighter than most operators), we run `add_type` to fill in the operand's `ty`, and we return an `ND_NUM` with the type's `size` as its value. The original `sizeof` token's location is the resulting node's `tok`.

That's it. `sizeof` is one more instance of the canonicalization pattern, but a particularly clean one: the operand AST is *thrown away*. We compute its type just to read the size, then we replace the whole subtree with a numeric literal. The codegen will see `mov $48, %rax` or `mov $24, %rax`; it has no idea a `sizeof` was ever there.

This is also why `sizeof(x = 2)` doesn't run the assignment:

```sh
assert 1 'int main() { int x=1; sizeof(x=2); return x; }'
```

The test asserts that `x` is still 1 after the `sizeof` expression. The `x = 2` is parsed and type-checked — the parser builds the AST for it — but the `new_num` replaces the whole tree before any of it reaches the codegen. C says `sizeof` operands are not evaluated (with one exception we'll get to), and chibicc gets that for free by not running them.

The "exception" in real C is `sizeof` of a variable-length array, which has to be evaluated to know its size. Chibicc doesn't have VLAs and won't for many chapters, so the exception doesn't apply. The simpler "sizeof never evaluates" is sufficient.

### `sizeof` as a keyword

```diff
 static bool is_keyword(Token *tok) {
-  static char *kw[] = {"return", "if", "else", "for", "while", "int"};
+  static char *kw[] = {
+    "return", "if", "else", "for", "while", "int", "sizeof",
+  };

   for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
     if (equal(tok, kw[i]))
```

The `kw[]` list grows by one entry. As before in §3.7, §3.9, and §4.4, this is the only thing the tokenizer has to learn — the second pass over the token stream (`convert_keywords`) reclassifies any identifier matching one of these as `TK_KEYWORD`, and the parser dispatches on string match against the keyword text.

There's a micro-amusement worth noting: `is_keyword` itself uses `sizeof` in its loop bound (`sizeof(kw) / sizeof(*kw)`, the standard C "size of the array divided by size of one element" idiom). This file is being compiled by GCC, not by chibicc, so the `sizeof` here works the way `sizeof` always has. But once chibicc can self-compile (much later in the book), this very loop will be a chibicc-compiled use of `sizeof` — Rui's compiler self-applying its own implementation.

### Tests

```diff
+assert 8 'int main() { int x; return sizeof(x); }'
+assert 8 'int main() { int x; return sizeof x; }'
+assert 8 'int main() { int *x; return sizeof(x); }'
+assert 32 'int main() { int x[4]; return sizeof(x); }'
+assert 96 'int main() { int x[3][4]; return sizeof(x); }'
+assert 32 'int main() { int x[3][4]; return sizeof(*x); }'
+assert 8 'int main() { int x[3][4]; return sizeof(**x); }'
+assert 9 'int main() { int x[3][4]; return sizeof(**x) + 1; }'
+assert 9 'int main() { int x[3][4]; return sizeof **x + 1; }'
+assert 8 'int main() { int x[3][4]; return sizeof(**x + 1); }'
+assert 8 'int main() { int x=1; return sizeof(x=2); }'
+assert 1 'int main() { int x=1; sizeof(x=2); return x; }'
```

Twelve tests. The first three exercise the basic types: `int` is 8, `int *` is 8, parens are optional. The next three walk down a 2-D array: `int x[3][4]` is 96 bytes, `*x` (one row) is 32 bytes, `**x` (one element) is 8 bytes. This is the array-decay rule in action — `*x` produces a value of type `int[4]`, whose size is 32, which is what `sizeof` reports. Without the decay rule applied carefully, `*x` would either fail to type or produce the wrong size.

The next two pairs exercise precedence and grouping. `sizeof(**x) + 1` is `(sizeof(**x)) + 1 = 8 + 1 = 9`; `sizeof **x + 1` parses as `sizeof(**x) + 1` because `sizeof` binds tighter than `+`, so it's also 9. But `sizeof(**x + 1)` is `sizeof(int + int)` (the parens force `**x + 1` to be evaluated as one expression) and it's 8 — the size of the resulting `int`, not the size of one element plus one byte.

The last two exercise the no-evaluation rule: `sizeof(x = 2)` produces 8 (the size of `int`) without changing `x`. The proof is the second test: after the `sizeof`, `x` is still 1. Both tests pass because the parser calls `add_type` to derive the operand's type (which doesn't run the side effects) and then discards the operand AST entirely.

### Where we are

`sizeof` works for any type chibicc currently has — `int`, pointer types, array types, and combinations. The result is always a compile-time integer constant, exactly as C requires for non-VLA operands. The user can now write portable-ish code like

```c
int x[100];
int n = sizeof(x) / sizeof(*x);
```

to get the array length, and chibicc will compute `800 / 8 = 100` at parse time.

The chapter is almost done. One commit remains, and it adds no features — it's a refactor that prepares for the next chapter. We close out arrays with `Function` and `Obj` becoming the same type.

---

## 6.5 — Merging `Function` with `Obj`

> `git checkout 0b7663481d0513067e0c0af04765b8578ae2a498` — *Merge Function with Var*

The commit message says "No functional change," and that's true at the level of compiled outputs. The diff is sixty lines, all of it shape-shifting: `Function` ceases to exist as a separate struct, `Obj` absorbs its fields, the `globals` list joins `locals` as a parser-side state variable, and `parse` returns an `Obj *` instead of a `Function *`.

Why now, in the middle of a chapter about arrays? Because the next chapter — globals, characters, strings — needs a unified representation for "thing with a name and a type that lives at some address." A function and a global variable have a lot in common: both have a name, a type, an address that gets resolved at link time, and a `.globl` directive in the output. Trying to keep them as separate structs would mean duplicating logic. The clean path is to acknowledge what they have in common, fold it into one struct, and let the code branch on the differences.

We flagged the convergence at the end of Chapter 5 §5.4: `fn->params` was already a *prefix* of `fn->locals`, meaning every parameter was already an `Obj`. The two structs were drifting toward each other. This commit is the meeting point.

### `Obj` absorbs `Function`

```diff
-// Local variable
+// Variable or function
 typedef struct Obj Obj;
 struct Obj {
   Obj *next;
-  char *name; // Variable name
-  Type *ty;   // Type
-  int offset; // Offset from RBP
-};
+  char *name;    // Variable name
+  Type *ty;      // Type
+  bool is_local; // local or global/function

-// Function
-typedef struct Function Function;
-struct Function {
-  Function *next;
-  char *name;
-  Obj *params;
+  // Local variable
+  int offset;

+  // Global variable or function
+  bool is_function;
+
+  // Function
+  Obj *params;
   Node *body;
   Obj *locals;
   int stack_size;
 };
```

The new `Obj` is a sum-type-by-flag. Two booleans (`is_local` and `is_function`) discriminate three kinds: local variable (`is_local && !is_function`), global variable (`!is_local && !is_function`), and function (`!is_local && is_function`). The fields used by each kind are commented as such — `offset` for locals, `params`/`body`/`locals`/`stack_size` for functions, with globals using only the common `name` and `ty`.

This is the "wide struct, branch on a tag" pattern, the same shape `Node` uses for its fifteen-or-so node kinds. It's not a hierarchy and it's not a tagged union (in the C-`union` sense) — it's a struct that holds every field anybody could want, and the discipline of "only read fields that match the kind" is enforced by code reviews and convention rather than by the type system. Chibicc has used this pattern consistently since Chapter 1 (when the `Node` struct first appeared with a long list of mostly-unused fields), and this commit applies it to the next-larger object kind.

The footprint cost is small. An `Obj` for a local variable has `is_function = false`, `params = NULL`, `body = NULL`, `locals = NULL`, `stack_size = 0` — five wasted fields, totaling about 36 bytes per local. There are typically a handful of locals per function, so this is a wasted few hundred bytes for a compiler input of any realistic size. Negligible.

### `parse` returns a single list

```diff
-Function *parse(Token *tok);
+Obj *parse(Token *tok);
```

The list returned by `parse` is now a flat list of `Obj`s — functions and (next chapter) global variables interleaved in source order. Codegen and `assign_lvar_offsets` will both filter by `is_function`:

```diff
 static void assign_lvar_offsets(Function *prog) {
   for (Function *fn = prog; fn; fn = fn->next) {
+    if (!fn->is_function)
+      continue;
+
     int offset = 0;
     ...
   }
 }
```

```diff
   for (Function *fn = prog; fn; fn = fn->next) {
+    if (!fn->is_function)
+      continue;
+
     printf("  .globl %s\n", fn->name);
+    printf("  .text\n");
     printf("%s:\n", fn->name);
```

Codegen's outer loop walks the unified list, skips non-function entries, and emits one `.globl` + body per function. The new `printf("  .text\n");` directive places each function in the `.text` section — assembly's name for the executable-instruction segment. This matters because §7.1 will introduce globals, which go in `.data`. Without explicit section directives, the assembler defaults to whichever was last; mixing functions and globals would break that default. The `.text` line is here, in this "no functional change" commit, because the next commit needs it.

Both filters are quietly preparing for §7's globals. At this commit, every `Obj` in the list happens to have `is_function = true` — the parser only produces functions for now — so the filter never actually skips anything. The structural change is in place, the semantic change comes next.

### `parse`'s shape pre-factors for globals

```diff
-static Function *function(Token **rest, Token *tok) {
-  Type *ty = declspec(&tok, tok);
-  ty = declarator(&tok, tok, ty);
+static Token *function(Token *tok, Type *basety) {
+  Type *ty = declarator(&tok, tok, basety);

-  locals = NULL;
-
-  Function *fn = calloc(1, sizeof(Function));
-  fn->name = get_ident(ty->name);
+  Obj *fn = new_gvar(get_ident(ty->name), ty);
+  fn->is_function = true;

+  locals = NULL;
   create_param_lvars(ty->params);
   fn->params = locals;
 
   tok = skip(tok, "{");
-  fn->body = compound_stmt(rest, tok);
+  fn->body = compound_stmt(&tok, tok);
   fn->locals = locals;
-  return fn;
+  return tok;
 }

-// program = function-definition*
-Function *parse(Token *tok) {
-  Function head = {};
-  Function *cur = &head;
+// program = (function-definition | global-variable)*
+Obj *parse(Token *tok) {
+  globals = NULL;

-  while (tok->kind != TK_EOF)
-    cur = cur->next = function(&tok, tok);
-  return head.next;
+  while (tok->kind != TK_EOF) {
+    Type *basety = declspec(&tok, tok);
+    tok = function(tok, basety);
+  }
+  return globals;
 }
```

Three reshapings sit inside this diff.

**`function` no longer calls `declspec`.** It's now `parse`'s job to consume the leading `int`, and `function` receives the resulting `basety` as a parameter. The reason: `parse` will (in §7.1) need to *look* at the basety and at the tokens that follow to decide whether the next thing is a function (whose name is followed by `(`) or a global variable (whose name is followed by `=` or `;` or `[`). To make that decision, `parse` has to consume the `declspec` first; `function` can't be the one to do it.

**`function` returns `Token *` instead of `Function *`.** The Obj it produces is no longer the return value — it's been registered with `new_gvar`, which prepends it to the `globals` list. `function` now just returns the updated token cursor. This is a minor stylistic change but matters for the next commit, where a `global_variable` parser will follow the same shape: parse, register, return tokens.

**`parse` builds the list via `globals`, not via head-and-cursor.** The function-definition list used to be built explicitly in `parse` with the standard sentinel-and-cursor idiom. Now the *side effect* of `function` (registering an Obj via `new_gvar`) is what builds the list, and `parse` just returns the head pointer. This reuses the same idiom that's been present for locals since Chapter 3 — the parser walks a global mutable list, the caller snapshots the result. The commit makes globals work the same way locals always have.

`new_gvar` and `new_var` are new helpers:

```c
static Obj *new_var(char *name, Type *ty) {
  Obj *var = calloc(1, sizeof(Obj));
  var->name = name;
  var->ty = ty;
  return var;
}

static Obj *new_lvar(char *name, Type *ty) {
  Obj *var = new_var(name, ty);
  var->is_local = true;
  var->next = locals;
  locals = var;
  return var;
}

static Obj *new_gvar(char *name, Type *ty) {
  Obj *var = new_var(name, ty);
  var->next = globals;
  globals = var;
  return var;
}
```

`new_var` is the common allocator. `new_lvar` adds the local flag and prepends to `locals`. `new_gvar` (currently unused by anything except `function`) prepends to `globals`. The locality flag is set in the constructor that creates the local; for globals, the flag stays at its zero-initialized default (false). The same separation will let global *variables* (next chapter) be created by `new_gvar` without confusion.

### `locals` becomes static

```diff
-Obj *locals;
+static Obj *locals;
+static Obj *globals;
```

A small but real change. `locals` was a non-`static` global, callable from any translation unit. With the introduction of `globals`, both lists become parser-private — they're scaffolding for `parse`'s internal state, not part of the parser's public interface. The compiler doesn't need to access them from `codegen.c` or `main.c`; it only needs the parsed `Obj *prog` returned by `parse`.

This is the kind of thing one only notices when a project is mid-cleanup. Through Chapters 3, 4, and 5, `locals` was a quiet linkage leak — visible to other files but never used by them. The merge of `Function` and `Obj` was the natural moment to fix it.

### `main.c` updates

```diff
+  // Tokenize and parse.
   Token *tok = tokenize(argv[1]);
-  Function *prog = parse(tok);
+  Obj *prog = parse(tok);
```

Two-line change: type rename, plus a comment. The comment foreshadows that `main.c` is going to grow more phases in future chapters (separate parse-emit phases, command-line flags, file I/O). For now, "tokenize and parse" is one phase.

### Why now and not later

A reader might reasonably ask: why fold these structs *now*, before the chapter that actually uses globals? Why not when the first global variable lands?

The answer is that globals can't be added cleanly to a system where `Function *` is the parse output. The parser would either have to return two lists (functions and variables) or wrap them in a discriminated union, both of which require more code than the merged-struct approach. Doing the merge as a separate "no functional change" commit lets Rui decouple the structural change from the feature change. The diff in this commit is purely refactoring — every test that passed before still passes after, and the commit can be reviewed in isolation. The next commit (Chapter 7 §7.1) will then be small: just the parser dispatch on "function or global var" and a codegen path for `.data`-section emission.

This is a discipline worth pointing at because it'll recur. Several times over the rest of the book, Rui will introduce a "no functional change" refactor right before the feature commit it enables. The pattern is: see the structural shape the next feature needs, do the structural change first, then add the feature on top. It keeps any single commit small enough to review in one sitting.

### Where we are

`Obj` is now the chibicc compiler's universal name-and-type record. Local variables, function parameters, and (about to be) global variables and functions all share the same representation, with boolean flags discriminating them. The parser produces a single `Obj *` list; codegen filters that list for functions; and the next chapter will add a parallel filter for globals. The `locals`/`globals` split inside the parser foreshadows the symbol-table organization we'll need when scopes get more complicated (Chapter 8, where blocks introduce inner scopes, and Chapter 13, where `static` and `extern` distinguish linkage from storage).

The chapter ends here. Five commits, four of them additive (arrays, multi-dim arrays, `[]`, `sizeof`) and one of them setting up the next chapter's machinery. The compiler has gained sizes, the canonicalization-at-parse-time pattern has gotten a name, and `Function` has been retired into `Obj`.

---

## Recap

| Commit | What it added |
|---|---|
| `8b6395d` | `TY_ARRAY`, `array_of`, `Type.size`, `Type.array_len`; size-aware `new_add`/`new_sub`/`assign_lvar_offsets`; `load`/`store` helpers with array-decay special case; `&` and `*` of arrays handled via `add_type` |
| `3ce1b2d` | Recursive `type_suffix` so `int x[2][3]` parses as array-of-array |
| `648646b` | `postfix` grammar level; `[]` desugars to `*( + )`; canonicalization-at-parse-time formally named |
| `3e55caf` | `sizeof` keyword; `primary` returns `new_num(node->ty->size)`; non-evaluating semantics fall out for free |
| `0b76634` | `Function` merged into `Obj`; `is_local`/`is_function` flags; `parse` returns `Obj *`; `new_gvar`; `.text` directive added; pre-factor for next chapter's globals |

Five commits, two of them substantial (the first and the last). The chapter's center of gravity is the first — `8b6395d` adds three things at once (a new type kind, sizes, decay) and gives every later commit something to lean on. The middle three commits are short individually but each completes a piece of array support: nesting, ergonomic indexing, and introspection. The last commit is plumbing for Chapter 7.

Two patterns crystallized this chapter. Canonicalization-at-parse-time finally got named — four instances was enough — and the practice of doing pure-refactor commits right before feature commits got its own treatment in §6.5. Both are going to keep mattering, and the book is now in a position to point at them by name when they recur.

Chapter 7 turns to globals, characters, and strings. It's a long arc of twelve commits — about four times the per-chapter commit budget so far — covering global variables, the `char` type, string literals with their full escape-sequence vocabulary, statement expressions (a GCC extension chibicc supports), reading from a file, the `-o` and `--help` flags, and source comments. The `Obj` merge and the `.text` directive from this chapter both come into play immediately. The hardcoded "every value is 8 bytes" assumption in `argreg` finally has to break, because `char` is one byte and parameter passing has to choose register width based on type. And string literals will be the first time the compiler emits something to the `.data` section — the directive that landed in this chapter's last commit and didn't yet have a use.
