# Chapter 11 — All the operators

> Commits covered: `a4fea2b`, `01a94c0`, `47f1937`, `e8ca48c`, `7df934d`, `6b88bcb`, `46a96d6`, `daa7398`, `8644006`, `f30f781`, `29ed294`, `7963221`, `61a1055`, `6116cae`, `a4be55b`, `b3047f2`, `3c83dfd`, `044d9ae`, `d0c0cb7`, `447ee09`, `79f5de2`. Twenty-one commits — the second-largest single chapter in the book — finishing chibicc's C operator surface.

Through Chapter 10 the type system grew from three types to eight, the cast machinery landed, and the parser learned every shape of declaration syntax a working programmer is likely to write. What chibicc still didn't have was most of C's operators. Compound assignment (`+=`, `-=`, …). Increment and decrement (`++`, `--`). Modulo (`%`). The bitwise operators (`&`, `|`, `^`, `<<`, `>>`) and their compound-assigns. Logical not and bitwise not (`!`, `~`). Short-circuit `&&` and `||`. The ternary `?:`. And on the control-flow side, `goto`, labeled statements, `break`, `continue`, `switch`/`case`/`default`. The chapter that adds them is the second-largest in the book by commit count.

Twenty-one commits is too many for one section per commit. The chapter bundles aggressively. Operators that share a lowering shape — the `+=` family, the bitwise family, the shift family, the increment family — go in one section. The two `goto` commits go in one section. The `break` and `continue` commits go in one section. After bundling there are fifteen sections, plus a closer.

The fifteen sections:

- **§11.1** — For-loop locals (commit 76).
- **§11.2** — Compound assignment (commit 77).
- **§11.3** — Pre and post increment / decrement (commits 78, 79).
- **§11.4** — Number-literal bases (commit 80).
- **§11.5** — `!` and `~` (commits 81, 82).
- **§11.6** — `%` and `%=` (commit 83).
- **§11.7** — Bitwise `&`, `|`, `^`, `&=`, `|=`, `^=` (commit 84).
- **§11.8** — `&&` and `||` (commit 85).
- **§11.9** — Incomplete types: arrays, struct forward declarations (commits 86, 87, 88).
- **§11.10** — `goto` and labels (commits 89, 90).
- **§11.11** — `break` and `continue` (commits 91, 92).
- **§11.12** — `switch` / `case` / `default` (commit 93).
- **§11.13** — Shift operators `<<`, `>>`, `<<=`, `>>=` (commit 94).
- **§11.14** — `?:` (commit 95).
- **§11.15** — Constant expressions (commit 96).

Two themes run through the chapter. The first is *canonicalization at parse time*, named in §6.5 and tracked since. Compound assignment desugars `a += b` into a comma-with-pointer-temp. Pre-increment desugars `++x` into `x += 1`. Post-increment desugars `x++` into `(typeof x)((x += 1) - 1)`. Each compound-assignment-on-shifts and bitwise-compound-assign desugars the same way. The §8.5 generalized-lvalue comma extension was planted exactly for this; the loop closes here. The second is the *operator-family with shared codegen*: the bitwise family lowers through one parser idiom and three lines of x86-64 each (`and`/`or`/`xor`); the shift family through one idiom and `shl`/`sar`; the relational family was set up earlier and gets one new precedence layer below it. Reading any one operator in detail effectively reads all three.

The dates of these commits, like Chapters 7–10, scatter across the calendar. Roughly half are dated August 2019; the rest are between April and October 2020. The October-2020 cluster is the largest — `+=`, pre-increment, `%`, bitwise, `&&`/`||`, shifts. Rui clearly worked through the operator surface in one sitting late in the year; the August-2019 commits sit above and below them on `main` in chronological-relative-to-canonical-order tangles that the chapter doesn't try to untangle. Chapter follows `main` order, which is the order the chapter mapping pins.

A note on what's *not* a section. The chapter mapping flagged the possibility of a concept interlude on `goto` and structured programming. The prose for §11.10 turned out not to need it — the commit's mechanics carry the section, and there's no intuition gap to bridge. No interlude.

---

## 11.1 — For-loop locals

> `git checkout a4fea2ba3edeb8ab5a0812a09f14c2a771aa196c` — *Allow for-loops to define local variables*

Until this commit, the `for` parser read its init slot as an `expr-stmt`:

```c
node->init = expr_stmt(&tok, tok);
```

That handles `for (i = 0; …)` but not `for (int i = 0; …)`. C99 added the latter, and the change to support it is a five-line patch:

```diff
-    node->init = expr_stmt(&tok, tok);
+    enter_scope();
+
+    if (is_typename(tok)) {
+      Type *basety = declspec(&tok, tok, NULL);
+      node->init = declaration(&tok, tok, basety);
+    } else {
+      node->init = expr_stmt(&tok, tok);
+    }
```

with a matching `leave_scope()` after the body. The `for`'s init slot now opens a fresh block scope and uses `is_typename` to decide whether to call `declaration` or `expr_stmt`. The scope ensures that the `i` declared in `for (int i = 0; ...)` doesn't escape the loop:

```c
ASSERT(3, ({ int i=3; int j=0; for (int i=0; i<=10; i=i+1) j=j+i; i; }));
```

The outer `i` is `3`. The loop's local `i` shadows it for the duration of the loop and disappears when the loop ends. Without `enter_scope`/`leave_scope` around the `for` body, the inner `i` would have leaked out and the assertion would fail.

This is the chapter's quietest commit and the only one that doesn't add an operator or a control-flow construct. It lands here because most of the chapter's tests use `for` loops with locally-declared counters — `for (int i = 0; i < 10; i++)` is the C idiom — and writing those tests before `for` accepted local declarations would have been awkward. A small pre-factor for the test code that follows.

### Where we are

`for` loops can declare their counters. Block scope opens at the `for` and closes at the matching `}`. The `is_typename` predicate gets one more caller.

---

## 11.2 — Compound assignment

> `git checkout 01a94c04aa2b5a95ac4038bd0d6fd5334fcbf882` — *Add `+=`, `-=`, `*=` and `/=`*

The chapter's first canonicalization-at-parse-time addition since §9.5. The §8.5 generalized-lvalue comma extension was planted explicitly for this; the loop closes here.

C says `a += b` evaluates `a` exactly once and stores the result back to it. The naive lowering — `a = a + b` — evaluates `a` *twice*, which is wrong as soon as `a` has side effects: `*p++ += 1` would advance `p` twice. The standard C trick is to evaluate the *address* of `a` once into a temporary, then dereference twice:

```c
tmp = &a;
*tmp = *tmp + b;
```

Two statements that both produce `*tmp`. As a single expression with a comma operator:

```c
(tmp = &a, *tmp = *tmp + b)
```

The §8.5 commit generalized chibicc's comma operator to accept a *generalized lvalue* on the right — a comma-expression produces an lvalue if its right operand is one. The §8.5 prose said the extension was unused at the time and predicted a `+=`-style construct as the consumer. That consumer is `to_assign`:

```c
// Convert `A op= B` to `tmp = &A, *tmp = *tmp op B`
// where tmp is a fresh pointer variable.
static Node *to_assign(Node *binary) {
  add_type(binary->lhs);
  add_type(binary->rhs);
  Token *tok = binary->tok;

  Obj *var = new_lvar("", pointer_to(binary->lhs->ty));

  Node *expr1 = new_binary(ND_ASSIGN, new_var_node(var, tok),
                           new_unary(ND_ADDR, binary->lhs, tok), tok);

  Node *expr2 =
    new_binary(ND_ASSIGN,
               new_unary(ND_DEREF, new_var_node(var, tok), tok),
               new_binary(binary->kind,
                          new_unary(ND_DEREF, new_var_node(var, tok), tok),
                          binary->rhs,
                          tok),
               tok);

  return new_binary(ND_COMMA, expr1, expr2, tok);
}
```

Read it slowly. The input is a binary node — say, `ND_ADD` with `lhs = A` and `rhs = B`. `to_assign` allocates a fresh local variable named `""` (an unnameable identifier — the parser never reuses it) of type "pointer to A's type." It builds two assignment nodes:

- `expr1` is `tmp = &A`. The temporary now holds A's address.
- `expr2` is `*tmp = *tmp op B`. The first `*tmp` is the destination lvalue, the second `*tmp` is the loaded value, and `binary->kind` (the `op`) is preserved.

Then it returns `(expr1, expr2)` — a comma expression. The result of the whole expression is the value of `expr2`, which is the new value of `*tmp`, which is the new value of `A`. C says compound assignment evaluates to the new value of the left-hand side; this matches.

Why a *pointer* temporary rather than a value temporary? Because A might be an arbitrary lvalue — `*p++`, `s.field`, `arr[i+1]` — and the address machinery is what evaluates exactly once. Storing the *address* into the temporary, then dereferencing it twice, gets the side-effect-once semantics for free. The §8.5 generalized-lvalue extension is what lets `*tmp = *tmp + b` *be* an lvalue (so that the comma expression as a whole can sit in further expression contexts).

The parser side is four `if` branches in `assign`:

```c
if (equal(tok, "+="))
  return to_assign(new_add(node, assign(rest, tok->next), tok));

if (equal(tok, "-="))
  return to_assign(new_sub(node, assign(rest, tok->next), tok));

if (equal(tok, "*="))
  return to_assign(new_binary(ND_MUL, node, assign(rest, tok->next), tok));

if (equal(tok, "/="))
  return to_assign(new_binary(ND_DIV, node, assign(rest, tok->next), tok));
```

The grammar comment updates accordingly: `assign-op = "=" | "+=" | "-=" | "*=" | "/="`. The new tokens `+=`, `-=`, `*=`, `/=` are added to `read_punct`'s table.

Three small details are worth noting.

First, `+=` and `-=` use `new_add` and `new_sub` (the pointer-aware helpers from §6.5), not `new_binary(ND_ADD, …)`. That's because compound assignment to a pointer must scale the right-hand side by the element size: `int *p; p += 1;` advances `p` by `sizeof(int)`. `new_add` already encodes that rule.

Second, `*=` and `/=` use plain `new_binary` — multiplication and division on pointers is meaningless and won't typecheck, so there's no scaling to worry about.

Third, the temporary's name is `""`. `new_lvar` accepts that without complaint, and the empty name guarantees the identifier never collides with any C identifier. The temporary lives in `locals` like any other local; codegen treats it as a normal stack variable.

### The canonicalization count

The §10 closer left the count at six (five strict desugarings plus the `({...})` delegation). Compound assignment is the seventh. The handoff predicted "probably several at once"; we'll see in §11.3 (pre/post-increment, two more) and §11.6 (`%=`, the same machinery) and §11.7 (`&=`/`|=`/`^=`) and §11.13 (`<<=`/`>>=`). The chapter conservatively counts compound-assign-via-comma as *one* mechanism — every `op=` operator routes through the same `to_assign` — so the increment is one, not many. By the end of the chapter the count will be eight (compound assignment + pre/post-increment).

### Where we are

`a += b`, `a -= b`, `a *= b`, `a /= b` work for any C lvalue. They evaluate `a` once. The §8.5 generalized-lvalue comma is the load-bearing mechanism. `to_assign` will get more callers in §11.3, §11.6, §11.7, §11.13.

---

## 11.3 — Pre and post increment / decrement

> `git checkout 47f19371f75db9029ea1b8b3783624fb7838d2db` — *Add pre `++` and `--`*
> `git checkout e8ca48cf41f5f3113cadfb23acfedad7b9fa2e63` — *Add post `++` and `--`*

Two commits, two desugarings, both routing through `to_assign`. Bundled because they share the same trick.

### Pre `++` and `--`

`++x` means "increment `x` by 1, then yield the new value of `x`." Which is exactly `x += 1`. Likewise `--x` is `x -= 1`. The parser hooks into `unary`:

```c
// Read ++i as i+=1
if (equal(tok, "++"))
  return to_assign(new_add(unary(rest, tok->next), new_num(1, tok), tok));

// Read --i as i-=1
if (equal(tok, "--"))
  return to_assign(new_sub(unary(rest, tok->next), new_num(1, tok), tok));
```

That's it. No new node kind, no codegen. Once parsed, `++i` is indistinguishable from `i += 1`. The `new_add` call handles pointer arithmetic correctly — `int *p; ++p;` advances by `sizeof(int)`, because `new_add` scales — and `to_assign` does the side-effect-once dance with the pointer temporary.

The grammar comment grows one line:

```diff
 // unary = ("+" | "-" | "*" | "&") cast
+//       | ("++" | "--") unary
 //       | postfix
```

A subtle point: pre-increment is parsed as a `unary` operator, not a `postfix` one. That's the C precedence table — `++x` and `x++` sit at different rungs. And pre-increment recurses into `unary` (so `++*p` works), where compound-assignment recurses into `assign`.

The tokens `++` and `--` are added to `read_punct`'s table.

### Post `++` and `--`

Post-increment is fiddlier. `x++` returns the *old* value of `x`, but still increments `x`. The standard trick is:

```
x++ ≡ (typeof x)((x += 1) - 1)
```

The expression `(x += 1)` evaluates to the new value of `x` (per §11.2 — compound assignment yields the new value). Subtracting `1` gives the old value back. The cast preserves the original type (since the subtraction may produce a wider type via the usual arithmetic conversion).

In code:

```c
// Convert A++ to `(typeof A)((A += 1) - 1)`
static Node *new_inc_dec(Node *node, Token *tok, int addend) {
  add_type(node);
  return new_cast(new_add(to_assign(new_add(node, new_num(addend, tok), tok)),
                          new_num(-addend, tok), tok),
                  node->ty);
}
```

The function takes an `addend` of `+1` (for `++`) or `-1` (for `--`):

- The inner `new_add(node, new_num(addend), tok)` builds `node + addend`.
- `to_assign` desugars that into `(tmp = &node, *tmp = *tmp + addend)` — this is the side-effect-once compound-assign.
- The outer `new_add(_, new_num(-addend), tok)` subtracts the addend back.
- `new_cast` restores the original type.

Why is this in `new_inc_dec` rather than a one-off in `postfix`? Because `++` and `--` only differ by sign; one helper generates both. And why is the outer add called `new_add` rather than `new_sub`? Because the addend is `-1` for `++` (subtract `-1` to undo `+1`) — the math is the same, the sign of the constant flips. Consistency with `new_add` matters here because for pointer increments, `new_add` knows to scale by element size; we want the same scaling on the way back.

The hook lives in `postfix`:

```diff
 // postfix = primary ("[" expr "]" | "." ident | "->" ident)*
+// postfix = primary ("[" expr "]" | "." ident | "->" ident | "++" | "--")*
 static Node *postfix(Token **rest, Token *tok) {
   ...
+    if (equal(tok, "++")) {
+      node = new_inc_dec(node, tok, 1);
+      tok = tok->next;
+      continue;
+    }
+
+    if (equal(tok, "--")) {
+      node = new_inc_dec(node, tok, -1);
+      tok = tok->next;
+      continue;
+    }
```

`postfix` is the iterative loop that handles `[]`, `.`, `->`, and now `++`/`--`. The order matters — `a[i]++` parses left-associatively: first the `a[i]` part is consumed by `postfix`'s subscript branch, then the `++` is consumed by `postfix`'s post-increment branch.

The tests for post-increment pin down the side-effect ordering:

```c
ASSERT(2, ({ int i=2; i++; }));      // statement value is the old i
ASSERT(3, ({ int i=2; i++; i; }));   // i is now 3
ASSERT(0, ({ int a[3]; a[0]=0; a[1]=1; a[2]=2; int *p=a+1; (*p++)--; a[0]; }));
```

The third assertion is the side-effect-ordering torture test. `*p++` evaluates as `*(p++)` — the `++` has higher precedence than `*` in postfix position. So `(*p++)--`: dereference `p` (which is `&a[1]`, value `1`), advance `p` (now `&a[2]`), then post-decrement what was at the original location (`a[1]` becomes `0`). After the expression, `a[0]` is still `0`, `a[1]` is `0`, `a[2]` is `2`, and `p` points at `a[2]`. The first assertion checks `a[0]`, which is `0` — meaning the post-decrement landed on `a[1]`, not `a[0]`. Both `++` and `--` had to advance the right pointer the right number of times.

### Date-vs-position note

Commit 78 (pre-increment) is dated 2020-10-07, but commit 79 (post-increment) is dated 2020-04-13. On `main`, 78 comes first. So the *ordering* on `main` says "pre-increment → post-increment," but the *chronological* order is the opposite. This isn't unusual for chibicc — Chapter 10's intro flagged the same pattern — and probably reflects Rui rewriting commits during the 2020-10 cleanup. The chapter follows `main` order, which is the order the book pins.

### Where we are

`++x`, `--x`, `x++`, `x--` work on integers and pointers. None of them needs a new node kind; both desugar to compound assignment plus, for the postfix forms, a cast back to the original type. The canonicalization count is eight: §11.2's compound-assign mechanism plus pre/post-increment as a single mechanism on top of it. The §8.5 comma extension is now load-bearing for ten operators, with more to come.

---

## 11.4 — Number-literal bases

> `git checkout 7df934d2b63727d67d1c054975893930fa6aff44` — *Add hexadecimal, octal and binary number literals*

A tokenizer change. Until this commit, every numeric literal was decimal. C has hex (`0x10`), octal (`077`), and — as a GCC extension long since adopted by Clang and standardized by C23 — binary (`0b101`). Rui adds all three in one commit by factoring the integer-reading logic into a helper:

```c
static Token *read_int_literal(char *start) {
  char *p = start;

  int base = 10;
  if (!strncasecmp(p, "0x", 2) && isalnum(p[2])) {
    p += 2;
    base = 16;
  } else if (!strncasecmp(p, "0b", 2) && isalnum(p[2])) {
    p += 2;
    base = 2;
  } else if (*p == '0') {
    base = 8;
  }

  long val = strtoul(p, &p, base);
  if (isalnum(*p))
    error_at(p, "invalid digit");

  Token *tok = new_token(TK_NUM, start, p);
  tok->val = val;
  return tok;
}
```

`strncasecmp` (case-insensitive prefix compare) recognizes `0x`/`0X` and `0b`/`0B`. The `isalnum(p[2])` guard prevents `0x` followed by nothing from misclassifying — without it, `0x` alone would be a malformed hex literal that `strtoul` would silently accept as zero. With the guard, the lexer falls through to octal (which then sees `0` followed by `x`, errors at `x`, and reports "invalid digit").

The leading-`0` octal rule is a C tradition that has bitten generations of programmers writing `0123` to mean one hundred twenty-three. (`077` is sixty-three.) chibicc inherits the rule without comment; the test pins it down:

```c
ASSERT(511, 0777);
```

`0777` in octal is `7*64 + 7*8 + 7` = `511`. If `0777` were decimal, the test would fail.

`strtoul` does the actual digit-by-digit parsing for any base. The trailing `isalnum(*p)` check catches the case where parsing stops mid-token because of a non-digit — `0xZ` or `123abc` — and reports an error at the offending character.

The caller in `tokenize` becomes a one-liner:

```diff
     // Numeric literal
     if (isdigit(*p)) {
-      cur = cur->next = new_token(TK_NUM, p, p);
-      char *q = p;
-      cur->val = strtoul(p, &p, 10);
-      cur->len = p - q;
+      cur = cur->next = read_int_literal(p);
+      p += cur->len;
       continue;
     }
```

A small `strings.h` include is added to `chibicc.h` because `strncasecmp` lives there on most systems (POSIX puts it in `strings.h`, not `string.h`).

### Where we are

Hex, octal, and binary literals tokenize correctly. `0xff`, `0xFF`, `0XFF` are all the same. `0b1011` and `0B1011` are the same. `077` is octal. The token's `val` field is `int64_t`-wide (per §10.2), so all these literals fit.

---

## 11.5 — `!` and `~`

> `git checkout 6b88bcb306ef80b65d7f99c081ba83283b4ffac5` — *Add `!` operator*
> `git checkout 46a96d6862e4c1317ff48df69391fd98a1ae5e3d` — *Add `~` operator*

Two unary operators, two minimal AST nodes, two minimal codegen sequences. Bundled because the prose for one essentially repeats the prose for the other.

### `!` — logical not

`!x` yields `1` if `x` is zero, `0` otherwise. The result type is `int`. The codegen is the same idiom we saw in §10.12 for the `_Bool` cast: compare against zero, set the byte if equal, zero-extend.

```c
case ND_NOT:
  gen_expr(node->lhs);
  println("  cmp $0, %%rax");
  println("  sete %%al");
  println("  movzx %%al, %%rax");
  return;
```

`cmp $0, %rax` sets the zero flag if `%rax` holds zero. `sete %al` writes 1 into the low byte of `%rax` if the zero flag is set, 0 otherwise. `movzx %al, %rax` clears the upper bits.

The §10.12 `_Bool` cast emits the *opposite* sequence — `cmp $0, %rax; setne %al; movzb %al, %rax` — because casting to `_Bool` yields 1 for nonzero and 0 for zero. `!` and `(_Bool)` differ by exactly one letter in the asm: `sete` vs. `setne`. Two operators, mirror-image of each other.

The parser hooks into `unary` alongside the other unary operators:

```c
if (equal(tok, "!"))
  return new_unary(ND_NOT, cast(rest, tok->next), tok);
```

A new `ND_NOT` enum entry, an `add_type` arm setting `ty = ty_int`, and tests:

```c
ASSERT(0, !1);
ASSERT(1, !0);
ASSERT(1, !(char)0);
ASSERT(4, sizeof(!(char)0));    // result is int, sizeof(int) == 4
ASSERT(4, sizeof(!(long)0));    // result is int regardless of operand
```

The `sizeof` tests are the type-system check: `!`'s result is always `int`, regardless of operand width. The §10.10 usual-arithmetic-conversion machinery doesn't kick in because `!` is unary; `add_type` simply hardcodes `ty_int`.

### `~` — bitwise not

`~x` flips every bit. The result type matches the operand. The codegen is one instruction:

```c
case ND_BITNOT:
  gen_expr(node->lhs);
  println("  not %%rax");
  return;
```

`not %rax` is x86-64's bitwise complement of all 64 bits. For an `int` (32-bit) operand whose value lives in `%eax` with the upper bits undefined, this happens to work because `!=` and `==` against signed values behave the same whether or not the upper bits match the sign — and for storage, `mov %eax, mem` only writes the low four bytes anyway. (Strictly: `not %rax` flips the upper bits too, but they get masked off whenever the value is stored or compared as a 32-bit value.)

The parser:

```c
if (equal(tok, "~"))
  return new_unary(ND_BITNOT, cast(rest, tok->next), tok);
```

`add_type` sets the result type to the operand's type:

```c
case ND_BITNOT:
  node->ty = node->lhs->ty;
  return;
```

Tests:

```c
ASSERT(-1, ~0);
ASSERT(0, ~-1);
```

`~0` is all-bits-set, which interpreted as a signed `int` is `-1`. `~-1` is `~(0xFFFFFFFF)` = `0`. Both pin down two's-complement.

### The `cast` recursion

Both `!` and `~` recurse into `cast`, not `unary`. That matters for `!(int)x` and `~(int)x`: the cast operator in §10.9 sits at the `cast` level (lower precedence than `unary`), so `!` and `~` of a cast expression need to descend through `cast` first to consume the cast and only then reach the operand. If they recursed into `unary`, `!(int)x` would parse as `!(int)`, which is nonsense. Recursing into `cast` makes the precedence right.

This is why `unary`'s grammar comment now reads:

```
unary = ("+" | "-" | "*" | "&" | "!" | "~") cast
      | ("++" | "--") unary
      | postfix
```

The first arm (`+`/`-`/`*`/`&`/`!`/`~`) uses `cast`; the second arm (`++`/`--`) uses `unary`. The increment operators recurse into `unary` because `++++x` is illegal — you can't increment an rvalue — and routing through `unary` reaches `postfix`/`primary` to find an lvalue. The first arm's operators don't need an lvalue and so can take the cast operator below them.

### Where we are

`!` and `~` work on integers. `!`'s result is always `int`; `~`'s result preserves the operand type. Both nest correctly with the cast operator.

---

## 11.6 — `%` and `%=`

> `git checkout daa739817c58baa8dcd0c23bb403d27d5907abfb` — *Add `%` and `%=`*

Modulo. The parser arm in `mul`:

```c
if (equal(tok, "%")) {
  node = new_binary(ND_MOD, node, cast(&tok, tok->next), start);
  continue;
}
```

The compound-assign arm in `assign`:

```c
if (equal(tok, "%="))
  return to_assign(new_binary(ND_MOD, node, assign(rest, tok->next), tok));
```

`%=` routes through `to_assign` like all the other compound-assigns. New token `%=` in `read_punct`'s table.

The codegen is the chapter's first reuse of an existing instruction sequence with a one-line tweak:

```c
case ND_DIV:
case ND_MOD:
  if (node->lhs->ty->size == 8)
    println("  cqo");
  else
    println("  cdq");
  println("  idiv %s", di);

  if (node->kind == ND_MOD)
    println("  mov %%rdx, %%rax");
  return;
```

x86-64's `idiv` instruction performs signed integer division, taking the dividend in `%rdx:%rax` (a 128-bit pair) or, for the 32-bit form, in `%edx:%eax`. The `cqo`/`cdq` instructions sign-extend `%rax`/`%eax` into `%rdx`/`%edx` first — that's the prerequisite for `idiv`. After `idiv`, the *quotient* lives in `%rax`/`%eax` and the *remainder* lives in `%rdx`/`%edx`. For division, that's already the right answer. For modulo, one extra `mov %rdx, %rax` moves the remainder into the conventional result register.

`ND_DIV` and `ND_MOD` share a `case` arm (C-style fall-through), with a final `if (kind == ND_MOD)` selecting the right output. This is the kind of shared codegen that's easy to miss if you read `gen_expr` linearly — the `case` arm looks like it handles only `ND_DIV` until you notice the `if (kind == ND_MOD)`.

`%`'s `add_type` arm is added to the `usual_arith_conv` group with the other arithmetic operators:

```diff
   case ND_MUL:
   case ND_DIV:
+  case ND_MOD:
     usual_arith_conv(&node->lhs, &node->rhs);
     node->ty = node->lhs->ty;
     return;
```

So `(long)17 % 6` produces a `long`-typed result, exactly like `(long)17 / 6` would.

### Where we are

`%` and `%=` work on integers. They share `idiv` codegen with `/` and `/=`. Pointer modulo is meaningless and won't typecheck.

---

## 11.7 — Bitwise `&`, `|`, `^`, `&=`, `|=`, `^=`

> `git checkout 86440068b43d6f9c93fdb07c1c2279cbab579e73` — *Add `&`, `|`, `^`, `&=`, `|=` and `^=`*

Six operators in one commit. Three new precedence layers in the parser, three new node kinds, three new codegen arms (each one instruction), three new compound-assigns routed through `to_assign`. The bundling makes sense: `&`, `|`, `^` have the same shape, and once you've seen one you've seen all three.

### Parser

The C precedence chain says: `bitor` is below `logor` (above), above `bitxor`; `bitxor` is above `bitand`; `bitand` is above `equality`. Three new layers slot in. Each one looks identical:

```c
// bitor = bitxor ("|" bitxor)*
static Node *bitor(Token **rest, Token *tok) {
  Node *node = bitxor(&tok, tok);
  while (equal(tok, "|")) {
    Token *start = tok;
    node = new_binary(ND_BITOR, node, bitxor(&tok, tok->next), start);
  }
  *rest = tok;
  return node;
}

// bitxor = bitand ("^" bitand)*
static Node *bitxor(Token **rest, Token *tok) { /* same shape with ^ */ }

// bitand = equality ("&" equality)*
static Node *bitand(Token **rest, Token *tok) { /* same shape with & */ }
```

Three nearly-identical functions. `assign` now starts with `bitor` instead of `equality` — the precedence chain has grown.

The grammar comment reflects this:

```
assign    = bitor (assign-op assign)?
assign-op = "=" | "+=" | "-=" | "*=" | "/=" | "%=" | "&=" | "|=" | "^="
```

And three more arms in the long `if`-cascade in `assign`:

```c
if (equal(tok, "&="))
  return to_assign(new_binary(ND_BITAND, node, assign(rest, tok->next), tok));

if (equal(tok, "|="))
  return to_assign(new_binary(ND_BITOR, node, assign(rest, tok->next), tok));

if (equal(tok, "^="))
  return to_assign(new_binary(ND_BITXOR, node, assign(rest, tok->next), tok));
```

`&=`, `|=`, `^=` go through `to_assign` like the rest of the compound-assigns.

### Codegen

```c
case ND_BITAND:
  println("  and %%rdi, %%rax");
  return;
case ND_BITOR:
  println("  or %%rdi, %%rax");
  return;
case ND_BITXOR:
  println("  xor %%rdi, %%rax");
  return;
```

Three lines of x86-64. The `and`/`or`/`xor` instructions accept two register operands and write the result into the destination — exactly the same shape as `add` and `sub`. By the time we reach this `case`, the binary-operator scaffolding has already loaded the right-hand side into `%rdi` and the left-hand side into `%rax`; the single-instruction codegen just names the operation.

### `add_type`

The three operators join the `usual_arith_conv` group:

```diff
   case ND_MUL:
   case ND_DIV:
   case ND_MOD:
+  case ND_BITAND:
+  case ND_BITOR:
+  case ND_BITXOR:
     usual_arith_conv(&node->lhs, &node->rhs);
     node->ty = node->lhs->ty;
     return;
```

So `0xff & (long)0` produces a `long`, same as `0xff + (long)0`.

### Tests

```c
ASSERT(0, 0&1);
ASSERT(3, 7&3);
ASSERT(10, -1&10);
ASSERT(0b10011, 0b10000|0b00011);
ASSERT(0b110100, 0b111000^0b001100);
ASSERT(2, ({ int i=6; i&=3; i; }));
ASSERT(7, ({ int i=6; i|=3; i; }));
ASSERT(10, ({ int i=15; i^=5; i; }));
```

The `0b110100 == 0b111000 ^ 0b001100` test exercises the §11.4 binary literals (without §11.4's tokenizer, the test wouldn't compile) and the bitwise codegen at the same time. Six-binary-digit literals as a way of writing tests is an idiom that pays for itself the moment the language has both.

### Where we are

`&`, `|`, `^` and their compound-assign forms work on integers. The parser's precedence chain now has three more layers. The compound-assign machinery has six tenants and counting.

---

## 11.8 — `&&` and `||`

> `git checkout f30f78175c1fd50c8cdd132ca804573ae0d18453` — *Add `&&` and `||`*

The first commit in the chapter that doesn't desugar. `&&` and `||` short-circuit: `a && b` doesn't evaluate `b` if `a` is zero, and `a || b` doesn't evaluate `b` if `a` is nonzero. That ordering can't be expressed by a desugaring into another binary operator — both operands of an unconditional binary node would always be evaluated. So `ND_LOGAND` and `ND_LOGOR` get their own AST kinds and their own codegen with branches and labels.

### Parser

Two more precedence layers, slotting in above `bitor` and below `assign`/`conditional`:

```c
// logor = logand ("||" logand)*
static Node *logor(Token **rest, Token *tok) {
  Node *node = logand(&tok, tok);
  while (equal(tok, "||")) {
    Token *start = tok;
    node = new_binary(ND_LOGOR, node, logand(&tok, tok->next), start);
  }
  *rest = tok;
  return node;
}

// logand = bitor ("&&" bitor)*
static Node *logand(Token **rest, Token *tok) { /* same shape with && */ }
```

`assign`'s entry point changes from `bitor` to `logor`. The grammar comment updates accordingly.

### Codegen

The interesting part. `&&` short-circuits to false: if the left operand is zero, the result is zero without evaluating the right operand.

```c
case ND_LOGAND: {
  int c = count();
  gen_expr(node->lhs);
  println("  cmp $0, %%rax");
  println("  je .L.false.%d", c);
  gen_expr(node->rhs);
  println("  cmp $0, %%rax");
  println("  je .L.false.%d", c);
  println("  mov $1, %%rax");
  println("  jmp .L.end.%d", c);
  println(".L.false.%d:", c);
  println("  mov $0, %%rax");
  println(".L.end.%d:", c);
  return;
}
```

Reading sequentially: evaluate the left. Compare to zero; if equal, jump to `.L.false`. Otherwise evaluate the right. Compare to zero; if equal, jump to `.L.false`. Both operands were nonzero — load `1` into `%rax` and jump to `.L.end`. `.L.false` loads `0` into `%rax`. `.L.end` is the merge point. The result is `1` (both true) or `0` (either false), and at most one of the two operands has been evaluated to a nonzero result.

`||` is the mirror image:

```c
case ND_LOGOR: {
  int c = count();
  gen_expr(node->lhs);
  println("  cmp $0, %%rax");
  println("  jne .L.true.%d", c);
  gen_expr(node->rhs);
  println("  cmp $0, %%rax");
  println("  jne .L.true.%d", c);
  println("  mov $0, %%rax");
  println("  jmp .L.end.%d", c);
  println(".L.true.%d:", c);
  println("  mov $1, %%rax");
  println(".L.end.%d:", c);
  return;
}
```

If the left is nonzero, jump to `.L.true` and yield `1`. Otherwise evaluate the right. If nonzero, jump to `.L.true` and yield `1`. Otherwise yield `0`.

The labels are uniqued by `count()` (the same monotonic counter used by `if`/`for`/`while`), so nested `&&`/`||` don't collide. The result type is always `int`:

```diff
   case ND_NOT:
+  case ND_LOGOR:
+  case ND_LOGAND:
     node->ty = ty_int;
     return;
```

### A small subtlety

C says `a && b` and `a || b` produce `0` or `1` regardless of the actual operand values. `5 && 3` is `1`, not `3`. `5 || 0` is `1`, not `5`. The codegen pattern above produces exactly `0` or `1` because the `mov $1, %rax` and `mov $0, %rax` arms write literal constants. A naive translation that returned the second operand (something like `cmp; jz .end; mov rhs, %rax; .end:`) would be wrong by the standard. Rui's pattern follows the standard precisely.

The new tokens `&&` and `||` go into `read_punct`'s table.

### Where we are

`&&` and `||` short-circuit. They're the chapter's first AST nodes that emit branching codegen rather than straight-line code. The result is `0` or `1`, type `int`. The parser's precedence chain has grown to seven layers above `bitor`: `assign` → `conditional` (wait — that's coming in §11.14, not yet) → `logor` → `logand` → `bitor` → `bitxor` → `bitand` → `equality` → ….

---

## 11.9 — Incomplete types: arrays, struct forward declarations

> `git checkout 29ed294906ebc271c32a755e1aefc360df4d3863` — *Add a notion of an incomplete array type*
> `git checkout 79632219d0991aae83e1de3c56df7d664205c2b6` — *Decay an array to a pointer in the func param context*
> `git checkout 61a10551209a0d3770449862152e1b73b584d771` — *Add a notion of an incomplete struct type*

Three commits about *incompleteness* — types that have a name but not yet a known size. Bundled because the underlying trick (use `size = -1` as a sentinel) is the same. The middle commit is the most subtle of the three: function parameters declared as arrays *decay* to pointers, and chibicc has been getting that wrong implicitly until now.

### Incomplete arrays (commit 86)

C lets you write `int x[]` (no dimension) in some contexts: array-of-pointer cast types, function parameters, and global declarations later initialized. The type-suffix parser is refactored to admit empty `[]`:

```c
// array-dimensions = num? "]" type-suffix
static Type *array_dimensions(Token **rest, Token *tok, Type *ty) {
  if (equal(tok, "]")) {
    ty = type_suffix(rest, tok->next, ty);
    return array_of(ty, -1);
  }

  int sz = get_number(tok);
  tok = skip(tok->next, "]");
  ty = type_suffix(rest, tok, ty);
  return array_of(ty, sz);
}
```

If `[` is immediately followed by `]`, the array is *incomplete* — `array_of(ty, -1)`. The `-1` is a sentinel meaning "size unknown." `array_of`, when handed `-1`, sets the type's `size` field to `-1` too (a signed `int` size accommodates this).

`declaration` rejects incomplete-typed variables outright:

```c
Type *ty = declarator(&tok, tok, basety);
if (ty->size < 0)
  error_tok(tok, "variable has incomplete type");
```

C says you can declare `int x[]` only in specific contexts; chibicc takes a stricter line and rejects it everywhere except where the parser explicitly allows it. The places that *do* allow it after this commit are abstract-declarator contexts like `sizeof(int(*)[])` (where the `[]` is between parens for the abstract function-pointer-shape, not a real declaration) and parameter contexts (next commit).

The test:

```c
ASSERT(8, sizeof(int(*)[10]));
ASSERT(8, sizeof(int(*)[][10]));
```

`sizeof(int(*)[][10])` — pointer to array-of-array-of-int with the outer dimension unknown — is `8` because pointers are 8 bytes regardless of the pointee's size or completeness.

The same commit threads a `Token *tok` field into `Member` (anticipating better error messages for bad struct member references) but doesn't otherwise wire it up.

### Function-parameter array decay (commit 87)

C has a special rule for function parameters. `int f(int x[])` and `int f(int *x)` declare the *same function*. The C standard calls it "adjustment of parameters" — array-typed parameters are silently rewritten to pointer-typed parameters at function-declaration parse time. Chibicc has been getting this wrong implicitly: an `int x[]` parameter would have type `array of int`, the parameter slot would be sized as a (broken) array, and the calling convention would be inconsistent with what real C compilers do.

The fix is in `func_params`:

```c
Type *ty2 = declspec(&tok, tok, NULL);
ty2 = declarator(&tok, tok, ty2);

// "array of T" is converted to "pointer to T" only in the parameter
// context. For example, *argv[] is converted to **argv by this.
if (ty2->kind == TY_ARRAY) {
  Token *name = ty2->name;
  ty2 = pointer_to(ty2->base);
  ty2->name = name;
}

cur = cur->next = copy_type(ty2);
```

After the type is fully assembled, if it's an array type, it's replaced with `pointer_to(base)`. The name token (carrying the parameter's identifier) is preserved across the swap. The C identity `int x[]` ≡ `int *x` now holds.

The test:

```c
int param_decay(int x[]) { return x[0]; }
ASSERT(3, ({ int x[2]; x[0]=3; param_decay(x); }));
```

The caller passes a 2-element array. The callee declares the parameter as `int x[]`, which after decay is `int *x`. The function reads `x[0]`. The behavior matches what `int *x` would have done, including the calling convention (the array decays to a pointer at the call site, which is exactly what C standard rules say happens for array-typed expressions when they appear in non-`sizeof`, non-`&` contexts).

### Incomplete struct types (commit 88)

The third incompleteness commit. C lets you declare `struct foo *bar;` without `struct foo` having a body — the *struct tag* enters the tag namespace, and the pointer is well-formed because pointers don't need to know their pointee's size. Later, when `struct foo { ... }` is defined, the previously-incomplete tag is filled in. This is the standard *forward declaration* idiom.

`struct_union_decl` is rewritten to handle three cases:

1. **Tag with body** (the original case). Declare and define the struct in one shot.
2. **Tag without body, no prior declaration**. Register the tag with size `-1`. Future references can refer to it; the size will be set when a definition arrives.
3. **Tag without body, prior declaration**. Look it up and return the existing type.

The new code:

```c
if (tag && !equal(tok, "{")) {
  *rest = tok;

  Type *ty = find_tag(tag);
  if (ty)
    return ty;

  ty = struct_type();
  ty->size = -1;
  push_tag_scope(tag, ty);
  return ty;
}

tok = skip(tok, "{");

// Construct a struct object.
Type *ty = struct_type();
struct_members(rest, tok, ty);

if (tag) {
  // If this is a redefinition, overwrite a previous type.
  // Otherwise, register the struct type.
  for (TagScope *sc = scope->tags; sc; sc = sc->next) {
    if (equal(tag, sc->name)) {
      *sc->ty = *ty;
      return sc->ty;
    }
  }

  push_tag_scope(tag, ty);
}

return ty;
```

A new helper `struct_type()` in `type.c` allocates a fresh struct type with `size = 0` and `align = 1`. The forward-declaration arm overwrites `size` to `-1` (incomplete). The body-defining arm fills in members and computes size and alignment. If the tag was previously incomplete, the body-defining arm finds it in `scope->tags` and *mutates the existing type in place* (`*sc->ty = *ty;`) — so any pointers declared with the incomplete tag now see the completed type without re-walking.

`struct_decl` and `union_decl` add an early-return for incomplete types:

```c
if (ty->size < 0)
  return ty;
```

Otherwise the offset-assignment loop would index into a missing members list and crash.

The test pin:

```c
ASSERT(8, ({ struct foo *bar; sizeof(bar); }));
ASSERT(4, ({ struct T *foo; struct T {int x;}; sizeof(struct T); }));
ASSERT(1, ({ struct T { struct T *next; int x; } a; struct T b; b.x=1; a.next=&b; a.next->x; }));
ASSERT(4, ({ typedef struct T T; struct T { int x; }; sizeof(T); }));
```

The first asserts that `struct foo *bar` is well-formed even though `struct foo` has no body — `bar` is a pointer, and `sizeof(bar) == 8`. The second asserts that a forward-declared `struct T` can be later defined and used. The third is the *recursive struct* idiom — `struct T` contains a pointer to itself — which only works because the inner `struct T *next` references the tag while it's still incomplete (mid-definition). The fourth combines `typedef struct T T;` with a later `struct T { int x; };` definition; the `typedef`-bound `T` and the tag-bound `struct T` share the underlying type, so the `T` typedef sees the completed `int x` member.

### Where we are

Three forms of incompleteness work: incomplete arrays (`int x[]`), array-to-pointer decay in parameters, and incomplete (forward-declared) struct types. The `size < 0` sentinel is the unifying marker. Forward-declared structs can be later defined; a recursive struct can hold a pointer to itself.

---

## 11.10 — `goto` and labels

> `git checkout 6116cae4c4b98ef9ed55736f3a6c1d872de97767` — *Add `goto` and labeled statement*
> `git checkout a4be55b333c9f712c334aac81e7ef4e076c2bc9b` — *Resolve conflict between labels and typedefs*

Two commits. The first adds `goto label;` and labeled statements. The second resolves a parser conflict that the first commit's grammar created.

### `goto` and labels (commit 89)

A function can have any number of labels and any number of `goto`s referring to them. A `goto` may name a label that is *defined later in the function* — forward jumps are common. So a single-pass parser can't resolve `goto` targets to addresses on the fly; it has to make a second pass after parsing the entire function. Rui's structure:

```c
// Lists of all goto statements and labels in the curent function.
static Node *gotos;
static Node *labels;
```

Two function-scoped globals. As the parser walks, every `goto` adds itself to `gotos`, every label adds itself to `labels`. Both lists chain through a `goto_next` field on `Node`. (Reusing the field name across both flavors is mildly cute; the names match because both are linked-list nexts, not because there's a runtime relationship.)

Parsing a `goto`:

```c
if (equal(tok, "goto")) {
  Node *node = new_node(ND_GOTO, tok);
  node->label = get_ident(tok->next);
  node->goto_next = gotos;
  gotos = node;
  *rest = skip(tok->next->next, ";");
  return node;
}
```

The node stores the *label name* (as a C string) into `node->label` and pushes itself onto the `gotos` list. The `unique_label` field — which codegen will use — is left null at this point.

Parsing a labeled statement:

```c
if (tok->kind == TK_IDENT && equal(tok->next, ":")) {
  Node *node = new_node(ND_LABEL, tok);
  node->label = strndup(tok->loc, tok->len);
  node->unique_label = new_unique_name();
  node->lhs = stmt(rest, tok->next->next);
  node->goto_next = labels;
  labels = node;
  return node;
}
```

The label gets a *unique* asm-level name (via `new_unique_name`, which produces `.L..NNNN` strings) and stores both the source-level label string and the unique asm name. The labeled statement wraps the actual statement after the colon — `foo: x++;` parses as `ND_LABEL("foo") -> ND_EXPR_STMT(x++)`.

After the function body is parsed, `resolve_goto_labels` matches gotos to labels:

```c
static void resolve_goto_labels(void) {
  for (Node *x = gotos; x; x = x->goto_next) {
    for (Node *y = labels; y; y = y->goto_next) {
      if (!strcmp(x->label, y->label)) {
        x->unique_label = y->unique_label;
        break;
      }
    }

    if (x->unique_label == NULL)
      error_tok(x->tok->next, "use of undeclared label");
  }

  gotos = labels = NULL;
}
```

A nested loop, O(gotos × labels). For every `goto`, scan the labels for a match; if none is found, error. After resolution every `goto` node's `unique_label` points at the label's unique asm name.

`function()` calls `resolve_goto_labels` after parsing the body and before the function returns. The static lists are zeroed out, ready for the next function.

Codegen is a one-liner per node kind:

```c
case ND_GOTO:
  println("  jmp %s", node->unique_label);
  return;
case ND_LABEL:
  println("%s:", node->unique_label);
  gen_stmt(node->lhs);
  return;
```

`jmp <label>` is unconditional jump. Emitting `<label>:` plants the label at that program point. The unique-naming dance is what keeps two functions' identically-named labels from colliding in the assembly output.

### Labels and the namespace question

Labels are a *fourth namespace* in C. Variables and typedef names share one (the `vars` chain in chibicc). Struct/union/enum tags share another (the `tags` chain). Members within each struct/union live in a per-struct namespace (handled by linear search in `find_member`). Labels are the fourth — they belong to a *function*, not a block, and they're stored in a function-private list (`gotos`/`labels`) rather than in any `Scope`.

The C standard's reasoning for separating labels: a `goto` from inside a nested `for` to a label in the enclosing function body must work, regardless of how blocks nest. If labels were block-scoped, `goto` would have to know about scope nesting. Putting them at function scope keeps `goto`'s semantics simple at the cost of one more namespace.

### The label-vs-typedef conflict (commit 90)

Commit 89's grammar has a parsing problem. Consider:

```c
typedef int x;
x: ;
```

The `compound_stmt` parser asks "does this statement start a declaration or a statement?" Its current criterion is `is_typename`. After §10.6, `is_typename` returns *true* for `x` (because `x` is a typedef name). So `compound_stmt` calls `declaration`, which expects something like `x var;` or `x var = 5;`. But the actual next tokens are `x : ;` — a label. `declaration` errors.

The C standard says: when an identifier is followed by `:`, it's a label, regardless of whether the identifier is also a typedef name. Labels don't conflict with typedef names because they live in different namespaces — but the parser, having only token-by-token lookahead, has to disambiguate before committing.

The fix is one line:

```diff
   while (!equal(tok, "}")) {
-    if (is_typename(tok)) {
+    if (is_typename(tok) && !equal(tok->next, ":")) {
       VarAttr attr = {};
       Type *basety = declspec(&tok, tok, &attr);
       ...
```

`compound_stmt` now peeks one token further: if the typename-token is followed by `:`, it's a label, fall through to `stmt`. The disambiguation is local to `compound_stmt`.

The test:

```c
ASSERT(1, ({ typedef int foo; goto foo; foo:; 1; }));
```

`typedef int foo` makes `foo` a typedef name. `goto foo` jumps to a label named `foo`. `foo: ;` is the labeled statement. The two `foo`s are in different namespaces and don't collide. Without the lookahead fix, `foo: ;` would be parsed as a (broken) declaration.

### A note on the parser-side hack

The §10.6 prose called the typedef-handling change in `is_typename` "the standard C lexer-versus-parser hack." Adding the `!equal(tok->next, ":")` lookahead is a refinement of that same hack: when the symbol-table-driven `is_typename` test gives a precise but standalone answer, you sometimes need *two* tokens of context to disambiguate the syntax. C's grammar isn't context-free in the formal sense; chibicc's parser handles the context-sensitivity locally, with one-token lookahead at the trouble spots.

This is the third lookahead-by-probe instance the chapter has crossed. §11.10's label-vs-typedef is the latest in a family that includes §7.1 (`int x = 5;` vs. `int x[5];` in `declarator`), §10.3 (nested declarators), §10.7 (abstract declarators), and §10.6 (`is_typename` itself).

### Where we are

`goto` and labeled statements work. Labels live in their own per-function namespace, separate from the four-or-so namespaces in `Scope`. The label-vs-typedef parse conflict is resolved with one-token lookahead in `compound_stmt`. The `keyword` `goto` joins the lexer's table.

---

## 11.11 — `break` and `continue`

> `git checkout b3047f2317b74f19fb44dfe5e577d586d93dfa3c` — *Add break statement*
> `git checkout 3c83dfd8af045ae6923d4ccb3a3a5a50f4012346` — *Add continue statement*

Two commits, same shape. Both add a control-flow keyword that jumps to a label, and the label is established by the surrounding loop. Bundled.

### `break` (commit 91)

`break` in a `for`, `while`, or `switch` jumps to the statement immediately after the construct. Each of those constructs gets a `brk_label` field on its `Node`, generated as a unique asm name when the construct is parsed.

A new global `static char *brk_label;` in `parse.c` tracks the *currently active* break label. The parser saves and restores it around each loop:

```c
char *brk = brk_label;
brk_label = node->brk_label = new_unique_name();
... parse the loop body ...
brk_label = brk;
```

Save the outer label, install the new one, parse the body (which may contain inner `break`s), restore. This is how nested loops work — the inner `break` sees the inner loop's label, the outer `break` sees the outer.

Parsing a `break`:

```c
if (equal(tok, "break")) {
  if (!brk_label)
    error_tok(tok, "stray break");
  Node *node = new_node(ND_GOTO, tok);
  node->unique_label = brk_label;
  *rest = skip(tok->next, ";");
  return node;
}
```

A `break` is parsed as an `ND_GOTO` node — the same node kind as user-written `goto`. The `unique_label` is set directly to the active break label; no lookup or resolution needed. Reusing `ND_GOTO` rather than introducing `ND_BREAK` keeps codegen trivial: `case ND_GOTO: jmp <unique_label>` already works for both.

The codegen for `for`/`while` changes one line: instead of `je .L.end.%d`, it emits `je %s` with the saved `brk_label`:

```diff
-      println("  je  .L.end.%d", c);
+      println("  je %s", node->brk_label);
   ...
-    println(".L.end.%d:", c);
+    println("%s:", node->brk_label);
```

The label is now *named* (the unique-name-generator's `.L..NNNN`) rather than positional.

### `continue` (commit 92)

`continue` jumps to the loop's *post-body* point — for `for`, that's just before the increment expression; for `while`, just before the condition recheck. A second per-loop label and a parallel global track this:

```c
static char *cont_label;
```

The save/restore ritual happens for both labels in `for` and `while`:

```c
char *brk = brk_label;
char *cont = cont_label;
brk_label = node->brk_label = new_unique_name();
cont_label = node->cont_label = new_unique_name();
... parse body ...
brk_label = brk;
cont_label = cont;
```

Parsing a `continue`:

```c
if (equal(tok, "continue")) {
  if (!cont_label)
    error_tok(tok, "stray continue");
  Node *node = new_node(ND_GOTO, tok);
  node->unique_label = cont_label;
  *rest = skip(tok->next, ";");
  return node;
}
```

Same shape as `break`. Same `ND_GOTO` reuse.

Codegen for `for` plants the continue-label between the body and the increment:

```c
gen_stmt(node->then);
println("%s:", node->cont_label);
if (node->inc)
  gen_expr(node->inc);
println("  jmp .L.begin.%d", c);
```

`continue` jumps to `cont_label`, which lands at the start of the increment expression. The increment runs, then the `jmp` to `.L.begin.%d` re-enters the condition check.

### Stray break / stray continue

Both forms guard against stray usage — `break` outside any loop or switch, `continue` outside any loop. The check is:

```c
if (!brk_label)
  error_tok(tok, "stray break");
```

If the parser hasn't entered any construct that sets `brk_label`, the global is null, and the error fires. The error catches typos and copy-paste mishaps and is the only constraint chibicc places on `break`/`continue` (it doesn't, e.g., enforce that `continue` can't appear directly inside a `switch` body — C says that's legal if the switch is itself inside a loop and the continue refers to the enclosing loop, and chibicc's parser lets it through naturally because `cont_label` is set by the surrounding loop).

### Where we are

`break` and `continue` work in `for` and `while`. They reuse `ND_GOTO` codegen. Stray usage errors. Both keywords go into the lexer's table.

---

## 11.12 — `switch` / `case` / `default`

> `git checkout 044d9ae07ba700c52d8342e4eee26e07eea11619` — *Add switch-case*

The chapter's most substantial control-flow commit. `switch (e) { case k1: …; case k2: …; default: …; }` evaluates `e`, jumps to the matching `case` label, falls through to subsequent cases unless a `break` interrupts, and falls through to `default` if no case matches.

### Parser

A new global tracks the currently-being-parsed switch:

```c
// Points to a node representing a switch if we are parsing
// a switch statement. Otherwise, NULL.
static Node *current_switch;
```

The switch's parse:

```c
if (equal(tok, "switch")) {
  Node *node = new_node(ND_SWITCH, tok);
  tok = skip(tok->next, "(");
  node->cond = expr(&tok, tok);
  tok = skip(tok, ")");

  Node *sw = current_switch;
  current_switch = node;

  char *brk = brk_label;
  brk_label = node->brk_label = new_unique_name();

  node->then = stmt(rest, tok);

  current_switch = sw;
  brk_label = brk;
  return node;
}
```

Save-install-parse-restore for both `current_switch` and `brk_label`. Note that no `cont_label` is set: `continue` inside a `switch` refers to the enclosing loop, not the switch. The `Node` has a `cond` (the switch-expression), a `then` (the body), and a `brk_label` that `break` inside will jump to.

`case` and `default` are parsed within `stmt`:

```c
if (equal(tok, "case")) {
  if (!current_switch)
    error_tok(tok, "stray case");
  int val = get_number(tok->next);

  Node *node = new_node(ND_CASE, tok);
  tok = skip(tok->next->next, ":");
  node->label = new_unique_name();
  node->lhs = stmt(rest, tok);
  node->val = val;
  node->case_next = current_switch->case_next;
  current_switch->case_next = node;
  return node;
}

if (equal(tok, "default")) {
  if (!current_switch)
    error_tok(tok, "stray default");

  Node *node = new_node(ND_CASE, tok);
  tok = skip(tok->next, ":");
  node->label = new_unique_name();
  node->lhs = stmt(rest, tok);
  current_switch->default_case = node;
  return node;
}
```

Each `case` parses its constant value (right now, via `get_number` — *one decimal integer*; this is temporary, replaced in §11.15), gets a unique asm-level label, parses the trailing statement, and links itself onto `current_switch->case_next`. The list grows in reverse order; codegen walks it accordingly.

`default` is a `case` without a value — it's stored on `current_switch->default_case` directly rather than threaded onto `case_next`.

### Codegen

`switch` codegen is comparison-and-jump (no jump table, no balanced-tree optimization — chibicc keeps it simple):

```c
case ND_SWITCH:
  gen_expr(node->cond);

  for (Node *n = node->case_next; n; n = n->case_next) {
    char *reg = (node->cond->ty->size == 8) ? "%rax" : "%eax";
    println("  cmp $%ld, %s", n->val, reg);
    println("  je %s", n->label);
  }

  if (node->default_case)
    println("  jmp %s", node->default_case->label);

  println("  jmp %s", node->brk_label);
  gen_stmt(node->then);
  println("%s:", node->brk_label);
  return;
case ND_CASE:
  println("%s:", node->label);
  gen_stmt(node->lhs);
  return;
```

The dispatch sequence:

1. Evaluate `cond` into `%rax`.
2. For each case: compare its value against `%eax` or `%rax` (per the cond type's size — 8-byte cond uses 64-bit compare, 4-byte cond uses 32-bit compare), jump to the matching case label.
3. After all cases, if there's a `default`, unconditionally jump to it. Otherwise, unconditionally jump to the break label (i.e., past the body).
4. Generate the body statements. The case labels embedded in the body act as targets for the `je`s above.
5. Plant the break label at the end.

The body codegen is straight-line — execution falls through from case to case unless a `break` interrupts, exactly matching C's fallthrough semantics. The `case`/`default` labels in the body are just labels; control reaches them via the dispatch jumps at the top.

The 32-bit vs. 64-bit register selection uses the cond's *size*, not its sign. The test pins this down:

```c
ASSERT(3, ({ int i=0; switch(-1) { case 0xffffffff: i=3; break; } i; }));
```

The cond is `int`-typed (4 bytes). The case value `0xffffffff` is parsed as a positive constant `4294967295`, but when stored in the case node's `val` (an `int64_t`) and then compared with a `cmp $0xffffffff, %eax` instruction, the 32-bit comparison interprets the immediate as a sign-extended 32-bit value — which matches `-1`. So `switch(-1)` matches `case 0xffffffff`. The `int`-vs-`long` choice of comparison register is what makes this work; if the codegen always used `%rax`, the comparison would be against `0x00000000ffffffff`, which doesn't equal sign-extended `-1`.

### The temporary `get_number`

This commit's `case` parser uses `get_number`, which accepts only a single integer token. That means `case 5+2:` would fail (the `+` isn't expected after the number). Real C says case values are *constant expressions* — they may involve arithmetic, sizeof, and integer casts. Rui's commit-93 implementation knows it's incomplete and writes the simpler version; commit 96 (§11.15) replaces `get_number` with a real constant evaluator.

This pattern — write a *temporary* hack, replace it later in the same chapter — is unusual for chibicc. Most simplifications stick around. Tracking it as a chapter-internal scaffolding pattern: the §11.15 commit removes `get_number` from `parse.c` entirely and replaces every caller (the `case` value here, the `enum` constant assignment, the array-dimension parser) with `const_expr`.

### Tests

```c
ASSERT(5, ({ int i=0; switch(0) { case 0:i=5;break; case 1:i=6;break; case 2:i=7;break; } i; }));
ASSERT(0, ({ int i=0; switch(3) { case 0:i=5;break; case 1:i=6;break; case 2:i=7;break; } i; }));
ASSERT(5, ({ int i=0; switch(0) { case 0:i=5;break; default:i=7; } i; }));
ASSERT(7, ({ int i=0; switch(1) { case 0:i=5;break; default:i=7; } i; }));
ASSERT(2, ({ int i=0; switch(1) { case 0: 0; case 1: 0; case 2: 0; i=2; } i; }));
ASSERT(0, ({ int i=0; switch(3) { case 0: 0; case 1: 0; case 2: 0; i=2; } i; }));
```

The fifth pin tests *fallthrough*: with no `break`s, `switch(1)` lands on `case 1`, falls through `case 2`, and reaches `i=2`. The sixth tests *no match*: `switch(3)` finds no case, so without `default` the body is skipped entirely.

`switch`, `case`, `default` join the lexer's keyword table.

### Where we are

`switch`, `case`, and `default` work for integer cases. Comparison-and-jump dispatch. Fallthrough is the default; `break` interrupts. Case values are parsed via the temporary `get_number`, replaced by `const_expr` in §11.15.

---

## 11.13 — Shift operators `<<`, `>>`, `<<=`, `>>=`

> `git checkout d0c0cb74b21f431c62f7eeb8dbc0d6e14c1eff14` — *Add `<<`, `>>`, `<<=` and `>>=`*

Four operators in one commit. The parser sprouts a new precedence layer between `relational` and `add`; codegen gains two arms; the compound-assigns route through `to_assign`.

### Parser

`shift` slots between `relational` and `add`:

```c
// shift = add ("<<" add | ">>" add)*
static Node *shift(Token **rest, Token *tok) {
  Node *node = add(&tok, tok);

  for (;;) {
    Token *start = tok;

    if (equal(tok, "<<")) {
      node = new_binary(ND_SHL, node, add(&tok, tok->next), start);
      continue;
    }

    if (equal(tok, ">>")) {
      node = new_binary(ND_SHR, node, add(&tok, tok->next), start);
      continue;
    }

    *rest = tok;
    return node;
  }
}
```

`relational` updates to call `shift` instead of `add`, so `a < b << 2` parses as `a < (b << 2)` — the C precedence rule (shifts bind tighter than relational).

The compound-assign arms in `assign`:

```c
if (equal(tok, "<<="))
  return to_assign(new_binary(ND_SHL, node, assign(rest, tok->next), tok));

if (equal(tok, ">>="))
  return to_assign(new_binary(ND_SHR, node, assign(rest, tok->next), tok));
```

Same pattern as the rest of the compound-assign family.

### Codegen

```c
case ND_SHL:
  println("  mov %%rdi, %%rcx");
  println("  shl %%cl, %s", ax);
  return;
case ND_SHR:
  println("  mov %%rdi, %%rcx");
  if (node->ty->size == 8)
    println("  sar %%cl, %s", ax);
  else
    println("  sar %%cl, %s", ax);
  return;
```

x86-64's variable-shift instructions (`shl` / `shr` / `sar`) require the shift count in `%cl` (the low byte of `%rcx`). Both arms move the right-hand side from `%rdi` (where the binary-operator scaffolding placed it) into `%rcx`, then emit the shift.

`shl` is *shift left*. `sar` is *shift arithmetic right* — preserves the sign bit. The `>>` operator uses `sar` because chibicc only has signed integer types; `sar` does sign-extension during the shift, which is the C-defined behavior for signed `>>`. (When unsigned types arrive in Chapter 14, the `>>` codegen will branch on operand signedness — `sar` for signed, `shr` (logical) for unsigned.)

The `if (node->ty->size == 8)` branch in `ND_SHR` is interesting: both arms emit the same instruction. The branch was probably meant to dispatch to a different opcode but ended up identical; perhaps Rui intended `sar` for 8-byte and a different mnemonic for 4-byte and didn't finish the differentiation. The `ax` macro at the top of `gen_expr` selects `%eax` or `%rax` based on `node->ty->size`, so the operand width is correct either way — the size-of-shift is determined by the destination register, not the opcode.

### `add_type`

```diff
   case ND_BITNOT:
+  case ND_SHL:
+  case ND_SHR:
     node->ty = node->lhs->ty;
     return;
```

Shift operators take the left operand's type, *not* `usual_arith_conv`. C says the left operand of a shift is integer-promoted but the right operand is independent — the result type is the (promoted) left operand's type. Chibicc's simpler rule says "the result is whatever the left side is," which is close enough for chibicc's integer surface (no `signed`/`unsigned` distinction yet).

### Tests

```c
ASSERT(1, 1<<0);
ASSERT(8, 1<<3);
ASSERT(10, 5<<1);
ASSERT(2, 5>>1);
ASSERT(-1, -1>>1);                 // arithmetic right shift preserves sign
ASSERT(1, ({ int i=1; i<<=0; i; }));
ASSERT(8, ({ int i=1; i<<=3; i; }));
ASSERT(-1, ({ int i=-1; i>>=1; i; }));
```

`-1 >> 1` is `-1` (all bits stay set under arithmetic shift). `-1 >>= 1` does the same with a compound-assign.

Three new tokens: `<<`, `>>`, `<<=`, `>>=`. The three-character `<<=` and `>>=` are placed *first* in `read_punct`'s table, before the two-character `<<` and `>>`, so the longest-match-first rule picks `<<=` over `<<` followed by `=`:

```diff
-    "==", "!=", "<=", ">=", "->", "+=", "-=", "*=", "/=", "++", "--",
-    "%=", "&=", "|=", "^=", "&&", "||",
+    "<<=", ">>=", "==", "!=", "<=", ">=", "->", "+=",
+    "-=", "*=", "/=", "++", "--", "%=", "&=", "|=", "^=", "&&",
+    "||", "<<", ">>",
```

`read_punct` walks the table in order and returns on the first match, so reordering matters: `<<=` is tried before `<<`, otherwise `i<<=2` would tokenize as `<<` followed by `=2` and parse as `i << (=2)`, which is broken in two different ways.

### Where we are

`<<`, `>>`, `<<=`, `>>=` work on integers. `>>` is arithmetic right shift (sign-preserving). The parser's precedence chain has another layer; the longest-match tokenizer table is hand-ordered.

---

## 11.14 — `?:`

> `git checkout 447ee098c51f6f615ef560b35d429f32f0cb5a35` — *Add `?:` operator*

The conditional operator. Three operands, branching codegen, usual arithmetic conversion between the second and third operands. A new precedence layer between `assign` and `logor`.

### Parser

```c
// conditional = logor ("?" expr ":" conditional)?
static Node *conditional(Token **rest, Token *tok) {
  Node *cond = logor(&tok, tok);

  if (!equal(tok, "?")) {
    *rest = tok;
    return cond;
  }

  Node *node = new_node(ND_COND, tok);
  node->cond = cond;
  node->then = expr(&tok, tok->next);
  tok = skip(tok, ":");
  node->els = conditional(rest, tok);
  return node;
}
```

`assign` now calls `conditional` instead of `logor`. The grammar comment changes:

```diff
-// assign = logor (assign-op assign)?
+// assign = conditional (assign-op assign)?
```

The middle operand (`then`) is parsed with `expr` (so `a ? b, c : d` parses as `a ? (b, c) : d`). The else-arm (`els`) is parsed with `conditional` itself, making `?:` *right-associative*: `a ? b : c ? d : e` parses as `a ? b : (c ? d : e)`.

Right-associativity matches C's grammar exactly. The other recursive operator at this level — assignment — is also right-associative (`a = b = c` is `a = (b = c)`); no surprise that `?:` follows the same pattern.

### Codegen

```c
case ND_COND: {
  int c = count();
  gen_expr(node->cond);
  println("  cmp $0, %%rax");
  println("  je .L.else.%d", c);
  gen_expr(node->then);
  println("  jmp .L.end.%d", c);
  println(".L.else.%d:", c);
  gen_expr(node->els);
  println(".L.end.%d:", c);
  return;
}
```

Standard if-then-else codegen: evaluate cond, branch on zero, emit then arm, jump past else, plant else label, emit else arm, plant end label. The result of whichever arm executed lives in `%rax` after this block. Same shape as the §3.5-ish `if`-statement codegen, but at expression level.

### `add_type`

The interesting part. C says `?:`'s result type is determined by *applying the usual arithmetic conversion to the second and third operands*. If either is `void`, the result is `void` (with no conversion).

```c
case ND_COND:
  if (node->then->ty->kind == TY_VOID || node->els->ty->kind == TY_VOID) {
    node->ty = ty_void;
  } else {
    usual_arith_conv(&node->then, &node->els);
    node->ty = node->then->ty;
  }
  return;
```

The `usual_arith_conv` call is the §10.10 helper. It sees the two operands, finds their common type, and inserts `ND_CAST` nodes around each so they're both that common type. Then the result type is whichever common type they ended up at.

Tests:

```c
ASSERT(2, 0?1:2);
ASSERT(4, sizeof(0?1:2));            // both int, common type int (4 bytes)
ASSERT(8, sizeof(0?(long)1:(long)2)); // both long, common type long (8 bytes)
ASSERT(-1, 0?(long)-2:-1);            // long and int, common type long, result -1 cast to long is -1
ASSERT(-1, 0?-2:(long)-1);            // int and long, same — result is (long)-1
1 ? -2 : (void)-1;                    // void in else arm, no conversion, result type void
```

The last one — `1 ? -2 : (void)-1` — is the `void` arm. C says this is legal as long as the result is used in a void context (e.g., as a statement-expression with no semantics). The test exercises just that.

### The canonicalization-at-parse-time question

`?:` *could* have been canonicalized — for instance, by lowering to a series of comma operators with side-effecting assignments and a result variable. Rui chose not to; `?:` ships as a new node kind with explicit branching codegen. The canonicalization count remains at eight (compound-assign + pre/post-increment).

The reason `?:` doesn't canonicalize is clear: there's no way to express short-circuiting evaluation in chibicc's existing AST without either branching codegen (which is what `ND_COND` provides) or runtime tests-and-branches encoded as a complicated comma-with-assignment lowering that wouldn't be simpler. The §11.8 prose for `&&`/`||` reached the same conclusion.

### Where we are

`?:` works. It short-circuits — only one arm is evaluated. `usual_arith_conv` chooses the common type. `void`-typed arms produce a `void`-typed result.

---

## 11.15 — Constant expressions

> `git checkout 79f5de21eb706ea5486fd682a83ffbde7e4d16a9` — *Add constant expression*

The chapter's closer, and a small but load-bearing piece of machinery. C requires *constant expressions* in several contexts — array dimensions, `case` values, enum-constant assignments, bit-field widths (later). Until this commit, chibicc handled these by parsing only a single integer literal via `get_number`. That's incomplete: real C allows arithmetic, sizeof, casts, even `&&`/`||` inside constant expressions.

The fix is a small evaluator that walks the AST after parsing and folds it down to an `int64_t`:

```c
// Evaluate a given node as a constant expression.
static int64_t eval(Node *node) {
  add_type(node);

  switch (node->kind) {
  case ND_ADD: return eval(node->lhs) + eval(node->rhs);
  case ND_SUB: return eval(node->lhs) - eval(node->rhs);
  case ND_MUL: return eval(node->lhs) * eval(node->rhs);
  case ND_DIV: return eval(node->lhs) / eval(node->rhs);
  case ND_NEG: return -eval(node->lhs);
  case ND_MOD: return eval(node->lhs) % eval(node->rhs);
  case ND_BITAND: return eval(node->lhs) & eval(node->rhs);
  case ND_BITOR:  return eval(node->lhs) | eval(node->rhs);
  case ND_BITXOR: return eval(node->lhs) ^ eval(node->rhs);
  case ND_SHL:    return eval(node->lhs) << eval(node->rhs);
  case ND_SHR:    return eval(node->lhs) >> eval(node->rhs);
  case ND_EQ:     return eval(node->lhs) == eval(node->rhs);
  case ND_NE:     return eval(node->lhs) != eval(node->rhs);
  case ND_LT:     return eval(node->lhs) <  eval(node->rhs);
  case ND_LE:     return eval(node->lhs) <= eval(node->rhs);
  case ND_COND:   return eval(node->cond) ? eval(node->then) : eval(node->els);
  case ND_COMMA:  return eval(node->rhs);
  case ND_NOT:    return !eval(node->lhs);
  case ND_BITNOT: return ~eval(node->lhs);
  case ND_LOGAND: return eval(node->lhs) && eval(node->rhs);
  case ND_LOGOR:  return eval(node->lhs) || eval(node->rhs);
  case ND_CAST:
    if (is_integer(node->ty)) {
      switch (node->ty->size) {
      case 1: return (uint8_t)eval(node->lhs);
      case 2: return (uint16_t)eval(node->lhs);
      case 4: return (uint32_t)eval(node->lhs);
      }
    }
    return eval(node->lhs);
  case ND_NUM:
    return node->val;
  }

  error_tok(node->tok, "not a compile-time constant");
}
```

A switch over every integer-arithmetic AST node kind. Each arm recursively `eval`s its children, applies the C operator, and returns the integer result. The `default` arm errors: a `case` value of `x + 1` where `x` is a runtime variable hits `ND_VAR`, which has no arm, and reports "not a compile-time constant."

The `ND_CAST` arm is the only one with size-specific logic. Casting to `char` (size 1) masks to `uint8_t`; to `short` (size 2) masks to `uint16_t`; to `int` (size 4) masks to `uint32_t`. The rationale: the C standard says casting an integer to a smaller integer takes the value modulo `2^N` where `N` is the bit-width of the target. Using `uint8_t`/`uint16_t`/`uint32_t` and letting C's implicit conversion to `int64_t` handle the sign extension does this in one C cast each. (`(long)0xfffff` and casting to `int` doesn't change the chibicc-host-compiler's behavior — both are 8 bytes — so the size-8 case has no arm.)

The `ND_COMMA` arm returns `eval(rhs)`, ignoring `lhs`. For a constant expression, the left side of a comma must itself be constant (else `eval(lhs)` would error inside the recursion), but its value is discarded in C's comma-operator semantics. This implementation order matters: chibicc's `eval` doesn't actually call `eval(lhs)` here — it returns the right side directly, skipping the left's evaluation entirely. That's slightly looser than C requires (the left side might not even be a constant expression and the test would still pass), but it's harmless because chibicc only calls `eval` from contexts where the *whole* expression is required to be constant, and the front-end already accepts only constant-shaped subtrees.

### `const_expr`

The wrapper that callers use:

```c
static int64_t const_expr(Token **rest, Token *tok) {
  Node *node = conditional(rest, tok);
  return eval(node);
}
```

Parse a `conditional`-level expression (which excludes assignment — assignments aren't constant), then evaluate it. The `conditional` entry point is exactly right: the C grammar says constant expressions are conditional-expressions, and conditional sits below assign.

`get_number` is removed entirely. The three callers — array-dimensions, case-values, and enum-constant assignments — all switch to `const_expr`:

```diff
-    int sz = get_number(tok);
-    tok = skip(tok->next, "]");
+    int sz = const_expr(&tok, tok);
+    tok = skip(tok, "]");
   ...
-      val = get_number(tok->next);
-      tok = tok->next->next;
+      val = const_expr(&tok, tok->next);
   ...
-    int val = get_number(tok->next);
-
     Node *node = new_node(ND_CASE, tok);
-    tok = skip(tok->next->next, ":");
+    int val = const_expr(&tok, tok->next);
+    tok = skip(tok, ":");
```

The grammar comments reflect the change: `array-dimensions = const-expr? "]" type-suffix`, `"case" const-expr ":" stmt`.

### Tests

`test/constexpr.c` is a new file with thirty-plus assertions, exercising every operator that `eval` supports, in every context that calls `const_expr`:

```c
ASSERT(10, ({ enum { ten=1+2+3+4 }; ten; }));    // arithmetic in enum constant
ASSERT(1, ({ int i=0; switch(3) { case 5-2+0*3: i++; } i; }));  // arithmetic in case
ASSERT(8, ({ int x[1+1]; sizeof(x); }));         // arithmetic in array dim
ASSERT(0b100, ({ char x[0b110&0b101]; sizeof(x); }));   // bitwise
ASSERT(2, ({ char x[1?2:3]; sizeof(x); }));      // ?: in array dim
ASSERT(15, ({ char x[(char)0xffffff0f]; sizeof(x); })); // cast to char masks
ASSERT(0x10f, ({ char x[(short)0xffff010f]; sizeof(x); })); // cast to short masks
ASSERT(4, ({ char x[(int)0xfffffffffff+5]; sizeof(x); })); // cast to int masks
ASSERT(8, ({ char x[(int*)0+2]; sizeof(x); }));    // pointer arithmetic in const expr
```

The pointer-arithmetic test is the only one that exercises `eval`'s implicit treatment of `ND_ADD`-with-pointer-children. There's no pointer-specific arm in `eval`; the expression `(int*)0 + 2` is parsed as `(int*)0 + 2`, which `add_type` and `new_add` would have transformed into `(int*)0 + 2*sizeof(int)` (= `+8`) at parse time. By the time `eval` sees the AST, the multiplication has already happened, and `ND_ADD`'s arithmetic arm just adds `0 + 8 = 8`. The test pins this down: `char x[(int*)0+2]` produces a `char[8]`.

### Forward references

The `eval` function will get more callers in coming chapters. Chapter 12 (initializers) will use it for compile-time constant initializers — `int x = 1 + 2;` evaluates `1 + 2` at parse time and stores `3` in the global's initial value. Chapter 13's `extern` doesn't directly use it, but `const` and storage-class qualifiers in coming chapters often do.

This commit's `eval` is the entire constant-expression implementation that ships through the next several chapters. It's small (one switch, thirty arms), lives in `parse.c`, and has no friends elsewhere in the codebase.

### Where we are

`const_expr` and `eval` exist. Constant expressions work in array dimensions, `case` values, and enum-constant assignments. The temporary `get_number` from §11.12 is replaced. The chapter closes.

---

## Recap

Twenty-one commits, fifteen sections. The net change is large: chibicc gains every C operator that wasn't already in place — compound assignment, increment/decrement, modulo, bitwise, shifts, logical, conditional — and every common control-flow construct beyond `if`/`for`/`while` — `goto`, labeled statements, `break`, `continue`, `switch`/`case`/`default`. By the end of the chapter the language is much closer to recognizable C.

Per the Chapter 10 closer's prediction, the chapter splits the recap into two themed tables.

| Commit | Topic |
|---|---|
| `a4fea2b` | `for` accepts a local declaration in the init slot. New scope opens at the `for`. |
| `01a94c0` | `+=`, `-=`, `*=`, `/=`. Lowered to `(tmp = &A, *tmp = *tmp op B)` via the §8.5 generalized-lvalue comma extension. The §8.5 prediction closes here. |
| `47f1937` | Pre `++` and `--`. Lowered to `i += 1` / `i -= 1`. |
| `e8ca48c` | Post `++` and `--`. Lowered to `(typeof i)((i += 1) - 1)`. |
| `7df934d` | Hex (`0x`), octal (leading `0`), binary (`0b`) literals. Tokenizer change. |
| `6b88bcb` | `!`. `cmp $0; sete; movzx`. Result type `int`. |
| `46a96d6` | `~`. `not %rax`. Result type matches operand. |
| `daa7398` | `%` and `%=`. Reuses `idiv` codegen with one extra `mov %rdx, %rax`. |
| `8644006` | `&`, `|`, `^` and three compound-assigns. Three new precedence layers. |
| `f30f781` | `&&` and `||`. Short-circuit via labels. Result `0` or `1`, type `int`. |

| Commit | Topic |
|---|---|
| `29ed294` | Incomplete arrays: `int x[]`. `array_of(ty, -1)` sentinel. Rejected outside specific contexts. |
| `7963221` | Function-parameter array decay: `int f(int x[])` ≡ `int f(int *x)`. |
| `61a1055` | Forward-declared structs: `struct foo *bar;` valid even without `struct foo`'s body. |
| `6116cae` | `goto` and labeled statements. Labels are a function-scoped fourth namespace. Two-pass label resolution. |
| `a4be55b` | `compound_stmt` peeks `tok->next == ":"` to disambiguate label-vs-typedef. |
| `b3047f2` | `break`. Reuses `ND_GOTO`. Save-install-restore of `brk_label` around each loop. |
| `3c83dfd` | `continue`. Same pattern with a parallel `cont_label`. |
| `044d9ae` | `switch`/`case`/`default`. Comparison-and-jump dispatch. Temporary `get_number` for case values, replaced by §11.15. |
| `d0c0cb7` | `<<`, `>>`, `<<=`, `>>=`. Single-instruction codegen via `shl`/`sar`. Longest-match tokenizer reordering. |
| `447ee09` | `?:`. New `ND_COND` with branching codegen. `usual_arith_conv` between then and else. |
| `79f5de2` | `eval` and `const_expr`. AST-walking integer-folding evaluator. Replaces `get_number` everywhere. |

Three structural shifts deserve repeating.

The first is the *closure of the §8.5 prediction*. The Chapter 8 prose said the comma operator's generalized-lvalue extension was unused at the time and predicted a `+=`-style construct as its consumer. §11.2 closed the loop. From there, every compound-assign operator (`+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`) and every increment operator (`++`, `--` in both pre and post forms) routed through `to_assign` and so through the comma extension. By the end of the chapter, twelve C operators canonicalize through that one mechanism. The §8.5 commit's worth — small at the time — is now load-bearing.

The second is the *fourth-namespace addition*. After Chapter 9's struct/union member namespace and Chapter 10's typedef-and-enum sharing of `vars`, labels are the fourth namespace — function-scoped, separate from any block. The label-vs-typedef parser conflict (§11.10) resolves via one-token lookahead in `compound_stmt`. The pattern continues a small theme from Chapters 7, 9, 10: when chibicc's parser hits a place where C's grammar isn't context-free, the answer is local lookahead, not a separate symbol-table mechanism.

The third is the *constant-expression evaluator*. `eval` (§11.15) is small — one switch, thirty arms, eighty-odd lines. But it's the load-bearing piece of three Chapter 12 commits, the bit-field-width handling that comes later, and several more. Until §11.15, chibicc could only handle constant-shaped contexts where a single integer literal sufficed. After §11.15, full constant expressions work. The chapter closes by setting up Chapter 12.

A few standing notes carried forward to Chapter 12:

- The *canonicalization-at-parse-time* count was six at the end of Chapter 10. This chapter adds two: compound-assign-via-comma (the §11.2 mechanism, which §11.6, §11.7, §11.13 reuse — counted as one) and pre/post-increment-via-compound-assign (§11.3 — counted as one). The new count is eight.
- The *pre-factor-before-feature* count remains at four. §11.1 (`for` locals) is a small enabler for tests that follow but the chapter doesn't formally count it as a pre-factor: the feature it enables is a *style of test*, not a future commit.
- The *lookahead-by-probe* family adds §11.10 (label-vs-typedef) — the fourth instance after §10.3 (nested declarators), §10.7 (abstract declarators), and §10.6 (`is_typename`).
- The *ND_GOTO node reuse* for `break` and `continue` is a small cute trick: by parsing them as `ND_GOTO` with a pre-set `unique_label`, no codegen change is needed.
- The *temporary scaffolding pattern*: §11.12 used `get_number` for case values, knowing it would be replaced; §11.15 made the replacement. This is the chapter's only example of "ship a scaffold, swap it later." Most chibicc simplifications stay.
- The *everything-fits-in-rax* invariant continues to hold for all new operators. Each binary-operator codegen leaves its result in `%rax` (or `%eax` for narrow types). `&&` and `||` and `?:` produce `0` or `1` (or one arm's value, for `?:`) in `%rax`.
- The *argreg 8/16/32/64 split* from §10.1 and §10.2 is unchanged.
- The *cast machinery* from §10.9–10.12 is the load-bearing piece for `?:`, the shift result type, and the const-expr cast-folding.
- The *tag namespace* from Chapter 9 §9.4 + Chapter 10 §10.14 is unchanged.
- The *VarAttr channel* (`is_typedef` and `is_static`) is unchanged. Chapter 13 will add `is_extern`.
- The *struct forward-declaration mutation in place* (`*sc->ty = *ty;`) in §11.9 is the first time chibicc's parser modifies an already-registered tag's type. Watch for this when Chapter 12 introduces flexible array members and Chapter 13 introduces global initializers.

Forward references for Chapter 12:

- `eval` will be called for compile-time-constant initializers — `int x = 1 + 2;` at file scope.
- The §11.9 incomplete-array machinery feeds into Chapter 12's array initializers (`int x[] = {1, 2, 3};` deduces size from initializer count).
- The §11.9 incomplete-struct machinery interacts with Chapter 12's struct initializers (`struct point p = {1, 2};`).
- Chapter 12 is the densest arc in the compiler at nineteen commits; expect substantial pre-factoring inside the chapter.
