# Chapter 15 — Floating point

> Commits covered: `1e57f72`, `29de46a`, `cf9ceec`, `83f76eb`, `0ce1093`, `8ec1ebf`, `c6b3056`, `8b14859`, `e452cf7`, `ffea421`, `9bf9612`. Eleven commits — floating-point literals and types, the cast-table extension, NaN-correct comparison, scalar arithmetic on the XMM register file, the truth-value extension, the call/definition pair for floating-point parameters, default argument promotion, the variadic-floats consumer of §14.2's spill area, the constant-evaluator parallel, and `long double` collapsed to `double`.

Chapter 14 closed the integer side of chibicc's type system by doubling every kind for signedness, and it set up a register-save area whose floating-point half — eight `%xmm` slots — was being spilled but never read. Chapter 15 fills in the floating-point side. By the end of it, the codegen knows how to load a `double` literal, evaluate `2.3 + 3.8`, branch on `if (x)` for floating-point `x`, pass and return floats and doubles through the System V psABI's XMM-register convention, promote floats to doubles at variadic call sites, fold floating-point constant expressions at parse time, and accept `long double` as a synonym for `double`.

Eleven commits, eleven sections, no concept interlude. The chapter mapping flagged the IEEE 754 bit pattern and the System V XMM-register convention as possible interlude candidates. Both ended up in the prose: the bit-pattern walk lives inside §15.1 (where chibicc loads literals as integer immediates and bit-casts them into XMM), and the XMM-register convention lives inside §15.6 and §15.7, split across the call and definition sides because they are different mechanics. Pulling either out into its own interlude would have left the surrounding section without a body.

The eleven sections:

- **§15.1** — Floating-point literals (commit 139).
- **§15.2** — `float`, `double`, and the cast table (commit 140).
- **§15.3** — Floating-point comparisons and NaN-correct equality (commit 141).
- **§15.4** — Floating-point arithmetic (commit 142).
- **§15.5** — Floating-point in control flow (commit 143).
- **§15.6** — Calling a function that takes or returns floats (commit 144).
- **§15.7** — Defining a function that takes or returns floats (commit 145).
- **§15.8** — Default argument promotion (commit 146).
- **§15.9** — Variadic floats (commit 147).
- **§15.10** — Floating-point constant expressions (commit 148).
- **§15.11** — `long double` as `double` (commit 149).

The chapter follows `main` order. The calendar dates scatter as usual. `8ec1ebf` (calling) and `c6b3056` (defining) are dated late August 2020; `8b14859` and `e452cf7` (default promotion and variadic floats) are dated *April–May 2020* — earlier than most of the chapter; the rest cluster mid-September 2020. The chapter doesn't try to untangle them.

---

## 15.1 — Floating-point literals

> `git checkout 1e57f72d8adf15937856a3ca3ca0e16ccb37421e` — *Add floating-point constant*

Chibicc's tokenizer has been splitting integer literals out of the source since Chapter 1, and over Chapters 7 and 14 it learned about hex, octal, binary, and the `U`/`L`/`LL` suffixes. Floating-point literals — `1.0`, `3e8`, `0x1p0`, `.1E4f` — have been absent. This commit adds them, plus the two new type kinds (`TY_FLOAT` and `TY_DOUBLE`) that the literals will assume.

The tokenizer change is the larger half. The old `read_int_literal` did the entire numeric-literal job: read digits, infer a base, handle the suffixes, build a `TK_NUM` token. With floating-point in play, that's no longer enough — the tokenizer has to distinguish `123` (an integer) from `123.0` (a float) from `1e10` (also a float, no decimal point but an exponent) from `123u` (an integer with a `U` suffix). Rui's solution is a *try-parse-then-decide* dispatcher:

```c
static Token *read_number(char *start) {
  // Try to parse as an integer constant.
  Token *tok = read_int_literal(start);
  if (!strchr(".eEfF", start[tok->len]))
    return tok;

  // If it's not an integer, it must be a floating point constant.
  char *end;
  double val = strtod(start, &end);

  Type *ty;
  if (*end == 'f' || *end == 'F') {
    ty = ty_float;
    end++;
  } else if (*end == 'l' || *end == 'L') {
    ty = ty_double;
    end++;
  } else {
    ty = ty_double;
  }

  tok = new_token(TK_NUM, start, end);
  tok->fval = val;
  tok->ty = ty;
  return tok;
}
```

Read the integer first; then look at the byte just past the integer. If it's one of `. e E f F`, the whole literal is actually a float and we re-parse from the start with `strtod`. Otherwise it was a real integer and we hand back the token as-is. The tokenizer's main loop dispatches into `read_number` whenever it sees a digit *or* a leading dot followed by a digit:

```c
if (isdigit(*p) || (*p == '.' && isdigit(p[1]))) {
  cur = cur->next = read_number(p);
  p += cur->len;
  continue;
}
```

The leading-dot case (`.1`, `.5e2`) is a small but necessary extension: previously, a leading `.` would never have been the start of a number. Now it can be.

The `read_int_literal` body also loses one line:

```diff
-  if (isalnum(*p))
-    error_at(p, "invalid digit");
```

The integer parser used to throw an error if anything alphanumeric followed the digits. That error has to go now, because something like `123e10` would otherwise fail before the dispatcher had a chance to see the `e` and re-route to `strtod`. The check moves up into the dispatcher (whatever `read_int_literal` doesn't consume must be one of `. e E f F` for the float path to apply, or the float path falls through and the leftover bytes will be picked up by the next token).

The suffix handling at the end of `read_number` mirrors the integer suffix logic from Chapter 14 §14.6, but smaller: `f` or `F` makes the literal a `float`, `l` or `L` makes it a `double` (chibicc has no `long double` until §15.11, where it becomes another alias for `double`), and a literal with no suffix defaults to `double`.

The token's value is stored in a new `fval` field on `Token`:

```c
struct Token {
  TokenKind kind; // Token kind
  Token *next;    // Next token
  int64_t val;    // If kind is TK_NUM, its value
  double fval;    // If kind is TK_NUM, its value
  // ...
};
```

`val` and `fval` coexist: `val` for integer tokens, `fval` for floating-point tokens. The token's `ty` (which already exists, since Chapter 14 §14.6) tells callers which to read.

The two new type kinds, `TY_FLOAT` and `TY_DOUBLE`, are added to the `TypeKind` enum, with their global instances:

```c
typedef enum {
  TY_VOID,
  TY_BOOL,
  TY_CHAR,
  TY_SHORT,
  TY_INT,
  TY_LONG,
  TY_FLOAT,
  TY_DOUBLE,
  // ...
} TypeKind;

Type *ty_float = &(Type){TY_FLOAT, 4, 4};
Type *ty_double = &(Type){TY_DOUBLE, 8, 8};
```

Sizes are 4 and 8, alignments are 4 and 8 — IEEE 754 single and double precision. Note that `ty_float` and `ty_double` are *not* declared with `is_unsigned` set; the `is_unsigned` flag from Chapter 14 §14.5 is irrelevant for floats. (Floats are signed in a sense, but it's a different sense — the sign is encoded in the bit pattern, not in a flag on the type. The flag on `Type` is for the *integer* signedness axis.) An `is_flonum` predicate joins `is_integer` in `type.c`:

```c
bool is_flonum(Type *ty) {
  return ty->kind == TY_FLOAT || ty->kind == TY_DOUBLE;
}
```

The parser's `primary` learns that a `TK_NUM` token might be a float and copies `tok->fval` into the node's new `fval` field:

```c
if (tok->kind == TK_NUM) {
  Node *node;
  if (is_flonum(tok->ty)) {
    node = new_node(ND_NUM, tok);
    node->fval = tok->fval;
  } else {
    node = new_num(tok->val, tok);
  }
  node->ty = tok->ty;
  *rest = tok->next;
  return node;
}
```

`Node` gains a `double fval` field alongside the existing `int64_t val`. Like `Token`, the two coexist — the type tells you which to read.

The codegen for `ND_NUM` learns the floating-point arms:

```c
case ND_NUM: {
  union { float f32; double f64; uint32_t u32; uint64_t u64; } u;

  switch (node->ty->kind) {
  case TY_FLOAT:
    u.f32 = node->fval;
    println("  mov $%u, %%eax  # float %f", u.u32, node->fval);
    println("  movq %%rax, %%xmm0");
    return;
  case TY_DOUBLE:
    u.f64 = node->fval;
    println("  mov $%lu, %%rax  # double %f", u.u64, node->fval);
    println("  movq %%rax, %%xmm0");
    return;
  }

  println("  mov $%ld, %%rax", node->val);
  return;
}
```

This is the section's first surprise. Floats are loaded *as integer immediates* and then bit-cast into the XMM register file by way of `%rax`. The `union { float f32; ...; uint32_t u32; ...; }` is chibicc-the-compiler reading the IEEE 754 bit pattern of `fval` (a host `double`), then printing that bit pattern as a hexadecimal integer in the assembly output. The assembler in turn sees `mov $0x4014666666666666, %rax` (or whatever the bit pattern is), then `movq %rax, %xmm0` transfers the 64-bit value into the bottom half of `%xmm0` without any reinterpretation. The final value in `%xmm0` is the same bits as the source `double` — IEEE 754 says so.

This means floating-point literals are *not* backed by anonymous globals, despite the Chapter 13 §13.4 anonymous-global pattern being available. Rui chose the simpler bit-pattern-as-immediate approach. There's a small cost to it: the assembly is bigger than it would be with a `.data` blob (a 64-bit immediate plus a 7-byte `movq` is more bytes of encoded instruction than a `movsd label(%rip), %xmm0`), but the codegen is local — no `Relocation`, no anonymous-global allocation, no `.data` section emission. For a compiler that doesn't optimize, that's the right trade.

The `# float %f` and `# double %f` are assembler comments. They serve no purpose at runtime; they're for humans reading the generated assembly to see the floating-point value at a glance, instead of decoding a hex bit pattern in their head.

`get_common_type` doesn't change in this commit (it gets float arms in §15.3, where they first matter). And the cast machinery is untouched — no float casts work yet. That's all §15.2.

The test file pins a few literal shapes:

```c
0.0;
1.0;
3e+8;
0x10.1p0;
.1E4f;

ASSERT(4, sizeof(8f));
ASSERT(4, sizeof(0.3F));
ASSERT(8, sizeof(0.));
ASSERT(8, sizeof(.0));
ASSERT(8, sizeof(5.l));
ASSERT(8, sizeof(2.0L));
```

The first five are *expression statements* whose values are discarded — they exist to verify the tokenizer doesn't barf on each shape. `0.0` (decimal-with-fraction), `1.0` (decimal-with-fraction), `3e+8` (exponent without dot), `0x10.1p0` (hex-with-fraction-and-binary-exponent), and `.1E4f` (leading-dot + uppercase exponent + `f` suffix). The size assertions confirm that suffix handling produces the right type — `8f` is `float` (4 bytes), `0.` is `double` (8 bytes), `5.l` and `2.0L` are also `double` (chibicc's `l`/`L` doesn't make `long double` here; that's §15.11).

### Where we are

The tokenizer recognizes floating-point literals and tags each with `ty_float` or `ty_double`. The parser copies `fval` from token to node. The codegen for `ND_NUM` loads the float as an integer immediate and bit-casts it into `%xmm0`. Two new `Type` kinds (`TY_FLOAT` and `TY_DOUBLE`) and one new predicate (`is_flonum`) join the system. Nothing else uses the new types yet — not casts, not arithmetic, not comparison, not control flow, not function calls. Everything other than `ND_NUM` will hit the float path and fall off the end of one switch or another. §15.2 starts repairing that.

---

## 15.2 — `float`, `double`, and the cast table

> `git checkout 29de46aed47e5308db9a0aef6e13610dea8fb389` — *Add "float" and "double" local variables and casts*

§15.1 stood up the types. This commit makes them usable: `float` and `double` parse as type specifiers, can be declared as variables, can be loaded from and stored to memory, and can be cast to and from every existing integer type. The cast table from Chapter 14 §14.5 grows from 8×8 to 10×10. The codegen's `load` and `store` learn an XMM-aware path. The tokenizer adds `float` and `double` to its keyword list. And `declspec`'s bit-flag enum grows two new specifiers.

Take the parser side first. The `is_typename` array gains `"float"` and `"double"` (Chapter 9 §9.x's predicate, repeatedly extended). The tokenizer's `is_keyword` array gains the same two strings. And `declspec`'s enum grows by two:

```c
enum {
  VOID     = 1 << 0,
  BOOL     = 1 << 2,
  CHAR     = 1 << 4,
  SHORT    = 1 << 6,
  INT      = 1 << 8,
  LONG     = 1 << 10,
  FLOAT    = 1 << 12,
  DOUBLE   = 1 << 14,
  OTHER    = 1 << 16,
  SIGNED   = 1 << 17,
  UNSIGNED = 1 << 18,
};
```

`FLOAT` and `DOUBLE` slot in between the integer kinds and `OTHER`, with their own bit positions. The shift is significant: the bit positions get spread further apart so each "doublable" specifier has room to be repeated (`long long`, `short short` would conflict, but only `LONG` is actually doublable; the spacing reserves headroom). `OTHER`, `SIGNED`, and `UNSIGNED` move up correspondingly.

Two new switch arms in `declspec`'s case dispatcher resolve the new specifiers to their types:

```c
case FLOAT:
  ty = ty_float;
  break;
case DOUBLE:
  ty = ty_double;
  break;
```

These are stand-alone — they don't combine with anything (until §15.11, where `LONG + DOUBLE` joins). `signed float` or `unsigned double` would resolve through `default: error_tok(...)` and reject; the standard says both are ill-formed.

Now the codegen. `load` and `store` learn that XMM types use different instructions:

```c
static void load(Type *ty) {
  switch (ty->kind) {
  case TY_ARRAY:
  case TY_STRUCT:
  case TY_UNION:
    return;
  case TY_FLOAT:
    println("  movss (%%rax), %%xmm0");
    return;
  case TY_DOUBLE:
    println("  movsd (%%rax), %%xmm0");
    return;
  }
  // ... integer load ...
}

static void store(Type *ty) {
  pop("%rdi");
  switch (ty->kind) {
  case TY_STRUCT:
  case TY_UNION:
    // ... struct/union memcpy ...
    return;
  case TY_FLOAT:
    println("  movss %%xmm0, (%%rdi)");
    return;
  case TY_DOUBLE:
    println("  movsd %%xmm0, (%%rdi)");
    return;
  }
  // ... integer store ...
}
```

`movss` is *move scalar single*, transferring 32 bits between memory and the bottom of an XMM register. `movsd` is *move scalar double*, 64 bits. The XMM register file is the same `%xmm0`–`%xmm15` set the integer codegen has been ignoring; floats live in the bottom 32 or 64 bits. The integer registers are entirely separate, and `mov` between memory and `%rax` cannot transfer to `%xmm0` directly — that's why §15.1's literal codegen had to bounce through `%rax` and then `movq` to `%xmm0`.

The `load`/`store` switch-statement reshape is also worth noting. The pre-§15.2 code was a single `if` for the array/struct/union early return; now it's a `switch` with each kind as a `case`, fall-through to the integer paths. The reshape was needed to add the float/double cases as siblings of the existing early-return cases without nesting. (A typical refactor-as-part-of-the-feature, not a separate pre-factor.)

Now the cast table. Chapter 14 §14.5 set up an 8×8 matrix indexed by `getTypeId`:

```c
enum { I8, I16, I32, I64, U8, U16, U32, U64 };
```

This commit extends the enum by two and the table by two rows and two columns:

```c
enum { I8, I16, I32, I64, U8, U16, U32, U64, F32, F64 };
```

`F32` is `TY_FLOAT`, `F64` is `TY_DOUBLE`. The dispatcher fills in the new arms of `getTypeId`:

```c
case TY_FLOAT:
  return F32;
case TY_DOUBLE:
  return F64;
```

The full 10×10 table:

```c
static char *cast_table[][10] = {
  // i8   i16     i32     i64     u8     u16     u32     u64     f32     f64
  {NULL,  NULL,   NULL,   i32i64, i32u8, i32u16, NULL,   i32i64, i32f32, i32f64}, // i8
  {i32i8, NULL,   NULL,   i32i64, i32u8, i32u16, NULL,   i32i64, i32f32, i32f64}, // i16
  {i32i8, i32i16, NULL,   i32i64, i32u8, i32u16, NULL,   i32i64, i32f32, i32f64}, // i32
  {i32i8, i32i16, NULL,   NULL,   i32u8, i32u16, NULL,   NULL,   i64f32, i64f64}, // i64

  {i32i8, NULL,   NULL,   i32i64, NULL,  NULL,   NULL,   i32i64, i32f32, i32f64}, // u8
  {i32i8, i32i16, NULL,   i32i64, i32u8, NULL,   NULL,   i32i64, i32f32, i32f64}, // u16
  {i32i8, i32i16, NULL,   u32i64, i32u8, i32u16, NULL,   u32i64, u32f32, u32f64}, // u32
  {i32i8, i32i16, NULL,   NULL,   i32u8, i32u16, NULL,   NULL,   u64f32, u64f64}, // u64

  {f32i8, f32i16, f32i32, f32i64, f32u8, f32u16, f32u32, f32u64, NULL,   f32f64}, // f32
  {f64i8, f64i16, f64i32, f64i64, f64u8, f64u16, f64u32, f64u64, f64f32, NULL},   // f64
};
```

The new conversion strings — eighteen of them, every cell of the float row and float column that isn't the diagonal — are individual instruction strings declared next to the existing integer ones:

```c
static char i32f32[] = "cvtsi2ssl %eax, %xmm0";
static char i32f64[] = "cvtsi2sdl %eax, %xmm0";
static char i64f32[] = "cvtsi2ssq %rax, %xmm0";
static char i64f64[] = "cvtsi2sdq %rax, %xmm0";
static char f32f64[] = "cvtss2sd %xmm0, %xmm0";
static char f64f32[] = "cvtsd2ss %xmm0, %xmm0";
static char f32i32[] = "cvttss2sil %xmm0, %eax";
static char f64i64[] = "cvttsd2siq %xmm0, %rax";
// ... and twelve more ...
```

The instruction names follow a regular pattern. `cvtsi2ss` is *convert scalar integer to scalar single*; `cvttss2si` is *convert with truncation scalar single to scalar integer* (the second `t` is for truncation — round toward zero rather than toward nearest). The `l`/`q` suffix selects 32-bit or 64-bit operand width on the integer side, and the `ss`/`sd` selects float or double on the floating-point side.

A few of the cells are subtle. The `u64f32` and `u64f64` paths handle the *unsigned 64-bit to floating-point* case, which is harder than the others because the x86 `cvtsi2ss`/`cvtsi2sd` instructions interpret their integer operand as *signed*. Casting `(double)(unsigned long)0xFFFFFFFFFFFFFFFFull` should produce `1.844…e19`, but the signed-interpretation instruction would produce `-1.0`. The fix is a small inlined runtime check:

```c
static char u64f64[] =
  "test %rax,%rax; js 1f; pxor %xmm0,%xmm0; cvtsi2sd %rax,%xmm0; jmp 2f; "
  "1: mov %rax,%rdi; and $1,%eax; pxor %xmm0,%xmm0; shr %rdi; "
  "or %rax,%rdi; cvtsi2sd %rdi,%xmm0; addsd %xmm0,%xmm0; 2:";
```

Test the high bit: if it's clear, the value is positive in signed interpretation too, and we use `cvtsi2sd` directly. If the high bit is set, we shift right by one, OR in the saved low bit, do the conversion, and double the result. This is a textbook unsigned-to-floating-point dance; it's expressed inline in the table-string form because that's where every other conversion lives. (A `cvtsi2sd` for the pure-signed half of the same case — `i64f64` — is a single instruction. The unsigned cell carries the burden because the underlying machine instruction doesn't.)

The `_Bool` cast arm in the `cast` function — Chapter 14 §14.5's contribution — also gets a float arm that wasn't visible in the diff because `cmp_zero` (which the `_Bool` cast uses) is what gets extended in §15.5. We'll get to that two sections from now.

Once the table is in place, the existing `cast(from, to)` function — which looks up `cast_table[t1][t2]` and emits the resulting instruction string — handles all the float conversions without code change. The cast machinery from Chapter 10 was designed to extend by adding to the table; this is exactly that extension.

The test file `test/cast.c` and the new `test/float.c` between them exercise every cell of the new table:

```c
ASSERT(0, (_Bool)0.0);
ASSERT(1, (_Bool)0.1);
ASSERT(3, (char)3.0);
ASSERT(1000, (short)1000.3);
ASSERT(3, (int)3.99);
ASSERT(2000000000000000, (long)2e15);
ASSERT(3, (float)3.5);
ASSERT(5, (double)(float)5.5);
ASSERT(3, (float)3);
ASSERT(3, (double)3);
```

`test/float.c` adds the full integer-to-float-and-back round-trip for every signedness and width. The very last assertion is the worth-noting one:

```c
ASSERT(-2147483648, (double)(unsigned long)(long)-1);
```

`(long)-1` is the bit pattern `0xFFFFFFFFFFFFFFFF`. Reinterpreted as `unsigned long`, it's `1.844…e19`. Cast to `double`, it's the floating-point representation of that — losing precision because `double` has only 52 mantissa bits. Cast back to `int` (the implicit type of the assertion), it's the truncated integer value. The fact that the test passes — the round-trip lands on a specific known integer — pins down the `u64f64` cast string above.

### Where we are

`float` and `double` are first-class types: declarable as variables, loadable from memory with `movss`/`movsd`, storable back, and convertible to and from every existing integer kind. The cast table from Chapter 14 §14.5 grows from 8×8 to 10×10 with eighteen new instruction strings. The `_Bool` cast on a float doesn't yet work (it routes through `cmp_zero`, which doesn't know about floats until §15.5). Float arithmetic doesn't work yet either — that's §15.4. But the variables and the casts are in.

---

## 15.3 — Floating-point comparisons and NaN-correct equality

> `git checkout cf9ceecb2f8cad2fb694b15c14ca1cf98e9524e7` — *Add flonum ==, !=, < and <=*

This commit lands `==`, `!=`, `<`, and `<=` for floats. The codegen is forty-nine lines but the section's substance is concentrated in two of them: the parity-flag check that distinguishes NaN-equality from real equality. Floating-point comparison on x86 sets *three* flags — zero, parity, and carry — and the standard requires that `NaN == anything` produce false, even `NaN == NaN`. Getting this right takes a small dance.

Start with the small things. Two new helpers, `pushf` and `popf`, push and pop the bottom 64 bits of `%xmm0` through the stack:

```c
static void pushf(void) {
  println("  sub $8, %%rsp");
  println("  movsd %%xmm0, (%%rsp)");
  depth++;
}

static void popf(char *arg) {
  println("  movsd (%%rsp), %s", arg);
  println("  add $8, %%rsp");
  depth--;
}
```

Symmetric to `push`/`pop` for the integer registers, but going through the XMM file. Both adjust `depth` (the chibicc-side stack-tracking counter from Chapter 5 used to keep 16-byte alignment at calls). `pushf` always pushes from `%xmm0`; `popf` pops to whichever XMM register the caller specifies (a string for now — §15.6 changes this to an integer).

The dispatch inside `gen_expr` for binary nodes whose left side is floating-point routes through a parallel block:

```c
if (is_flonum(node->lhs->ty)) {
  gen_expr(node->rhs);
  pushf();
  gen_expr(node->lhs);
  popf("%xmm1");

  char *sz = (node->lhs->ty->kind == TY_FLOAT) ? "ss" : "sd";

  switch (node->kind) {
  case ND_EQ:
  case ND_NE:
  case ND_LT:
  case ND_LE:
    println("  ucomi%s %%xmm0, %%xmm1", sz);

    if (node->kind == ND_EQ) {
      println("  sete %%al");
      println("  setnp %%dl");
      println("  and %%dl, %%al");
    } else if (node->kind == ND_NE) {
      println("  setne %%al");
      println("  setp %%dl");
      println("  or %%dl, %%al");
    } else if (node->kind == ND_LT) {
      println("  seta %%al");
    } else {
      println("  setae %%al");
    }

    println("  and $1, %%al");
    println("  movzb %%al, %%rax");
    return;
  }

  error_tok(node->tok, "invalid expression");
}
```

The shape mirrors the integer comparison: evaluate rhs, push, evaluate lhs, pop into a second register, then compare. The integer side pushes `%rax` and pops to `%rdi`; the float side pushes `%xmm0` and pops to `%xmm1`. `sz` picks the instruction-suffix variant (`ucomiss` for single, `ucomisd` for double).

`ucomiss` is *unordered compare scalar single*. It sets `%rflags` based on the comparison: ZF (zero flag) on equal, CF (carry flag) on less-than, and — the surprise — PF (parity flag) on unordered. Unordered means at least one of the operands is NaN. The IEEE 754 rule that `NaN == NaN` is false requires this third bit, because in ordinary x86 integer comparison there's no equivalent — integers are totally ordered, so two operands are either equal, less, or greater.

The `==` arm reads it directly:

```
sete %%al    # set %al if ZF is set (operands equal)
setnp %%dl   # set %dl if PF is clear (operands ordered)
and %%dl, %%al   # %al &= %dl
```

`==` is true only if the operands are equal *and* ordered. For `NaN == NaN`, ZF is set (the equality machinery flags them as bitwise-similar in some sense; what matters is what `ucomiss` does, which is set ZF and PF) — but PF is also set, so `setnp` clears `%dl`, and the AND clears `%al`. The result: `NaN == NaN` produces 0. Correct.

`!=` is the dual:

```
setne %%al   # set %al if ZF is clear
setp %%dl    # set %dl if PF is set (unordered)
or %%dl, %%al  # %al |= %dl
```

`!=` is true if the operands are unequal *or* unordered. `NaN != NaN` is true (PF is set, OR-ing in `%dl` produces 1).

`<` and `<=` are simpler:

```
seta %%al    # set if CF=0 and ZF=0
setae %%al   # set if CF=0
```

These are the *unsigned* comparison setters from Chapter 14 §14.5 (`seta` for "above", `setae` for "above-or-equal" — both unsigned). Why unsigned? Because `ucomi*` sets the integer flags such that the "less-than" answer comes out in the carry flag, the same place the unsigned integer comparison `cmp` puts it. A NaN operand sets PF *and* CF, so `seta` (which requires CF clear) and `setae` (which also requires CF clear) both correctly produce 0 — `NaN < anything` is false, `NaN <= anything` is false.

There's a subtlety in the operand order. The dispatch generates rhs into `%xmm0`, then pushes, then generates lhs (which lands in `%xmm0`), then pops rhs into `%xmm1`. After the dance, `%xmm0` holds *lhs* and `%xmm1` holds *rhs*. The `ucomi%s %%xmm0, %%xmm1` line then says — *in AT&T syntax* — "compare `%xmm1` against `%xmm0`": that is, "is `%xmm1` (which is rhs) less than `%xmm0` (which is lhs)?" `ucomi*` sets CF if the *first source* is less than the *second source*. So CF=1 means rhs < lhs, i.e. lhs > rhs. The `seta` instruction tests CF=0 and ZF=0, which means rhs >= lhs and rhs != lhs, i.e. lhs > rhs. Wait — the AST has lhs and rhs for `lhs < rhs`, but `seta` tests `lhs > rhs`. Re-read the dispatch: the code emits `seta` for `ND_LT`. The dispatch is doing `lhs < rhs` by computing `rhs > lhs` — operands swapped because of the push-pop dance. The same swap applies to `<=` (`setae` testing `rhs >= lhs`, equivalent to `lhs <= rhs`).

Last line of the float-comparison block:

```
and $1, %%al
movzb %%al, %%rax
```

— mask everything but the bottom bit (in case `setp` left a higher bit, which it doesn't, but the code is defensive), then zero-extend to `%rax`. Result: `%rax` is 0 or 1.

The `error_tok(node->tok, "invalid expression")` at the bottom is a sentinel: if the dispatcher reached this point and the node kind wasn't `ND_EQ`/`ND_NE`/`ND_LT`/`ND_LE`, *something* in the parser routed a non-comparison node here through the float path. It can't happen in well-formed code today — `ND_ADD` and friends will fall into the float arm too once §15.4 lands, but as of *this commit* only comparisons are handled, so anything else is a bug.

`type.c`'s `get_common_type` learns the float promotion rule:

```c
if (ty1->kind == TY_DOUBLE || ty2->kind == TY_DOUBLE)
  return ty_double;
if (ty1->kind == TY_FLOAT || ty2->kind == TY_FLOAT)
  return ty_float;
```

— at the top of the function, before the integer promotion logic. The C standard's *usual arithmetic conversions* say that if either operand is a `double`, both become `double`; otherwise if either is a `float`, both become `float`; otherwise integer rules apply. Chibicc encodes the cascade as two early returns at the top of the function.

The test file pins the comparisons:

```c
ASSERT(1, 2e3==2e3);
ASSERT(0, 2e3==2e5);
ASSERT(1, 2.0==2);
ASSERT(0, 5.1<5);
ASSERT(0, 5.0<5);
ASSERT(1, 4.9<5);
ASSERT(0, 5.1<=5);
ASSERT(1, 5.0<=5);
ASSERT(1, 4.9<=5);
```

Note `2.0==2`: the `2` is an `int`, the `2.0` is a `double`. `get_common_type` promotes the int to double, the comparison runs as double-vs-double, and 2.0 == 2 is true. The mixed-type compare is the smallest test that exercises the new `get_common_type` arm.

The variants with `f` suffix exercise the `float` path:

```c
ASSERT(1, 2e3f==2e3);
// ...
```

Here `2e3f` is `float` and `2e3` is `double`. The common type is `double` (the higher-precision one); both get cast up before comparison.

NaN-equality is tested in the next commit (§15.4), since constructing a NaN literal requires `0.0/0.0`, which needs floating-point division.

### Where we are

`==`, `!=`, `<`, and `<=` work on floats with NaN-correct semantics. The dispatch in `gen_expr` adds an early branch on `is_flonum(node->lhs->ty)` that handles all four comparisons through `ucomiss`/`ucomisd`. The parity-flag dance distinguishes "equal-and-ordered" from "ZF-set-because-NaN-vs-NaN." `get_common_type` learns the float promotion cascade. Float arithmetic is still missing (§15.4); the float branch emits `error_tok("invalid expression")` for any non-comparison, but that error path goes away the moment §15.4 fills in the arithmetic arms.

---

## 15.4 — Floating-point arithmetic

> `git checkout 83f76ebb66712a2560b2993e92265b574b1ab7ed` — *Add flonum +, -, * and /*

The four arithmetic operators land in the float branch added by §15.3. The codegen is small — four new switch arms — because the comparison dispatch already set up the operand evaluation, the push-pop dance, and the size suffix. The arithmetic operators just need to emit the right scalar instruction:

```c
switch (node->kind) {
case ND_ADD:
  println("  add%s %%xmm1, %%xmm0", sz);
  return;
case ND_SUB:
  println("  sub%s %%xmm1, %%xmm0", sz);
  return;
case ND_MUL:
  println("  mul%s %%xmm1, %%xmm0", sz);
  return;
case ND_DIV:
  println("  div%s %%xmm1, %%xmm0", sz);
  return;
case ND_EQ:
// ... unchanged ...
}
```

`addss`, `subss`, `mulss`, `divss` are *add/sub/mul/div scalar single*; the `sd` variants are double. Each takes two XMM operands (in AT&T syntax, source first, destination second), reading the bottom 32 or 64 bits and writing the result back to the destination's bottom slot. Unlike integer division (which needs `idiv`/`div` with implicit `%rdx:%rax` operand placement), float division is regular: `divss %xmm1, %xmm0` divides `%xmm0` by `%xmm1` and stores the quotient in `%xmm0`. No setup, no special registers.

Negation needs its own arm earlier in the same function. The integer codegen `case ND_NEG: ... println("  neg %%rax");` doesn't work on XMM — `neg` is an integer instruction. The float codegen does the negation by XOR-ing the sign bit:

```c
case ND_NEG:
  gen_expr(node->lhs);

  switch (node->ty->kind) {
  case TY_FLOAT:
    println("  mov $1, %%rax");
    println("  shl $31, %%rax");
    println("  movq %%rax, %%xmm1");
    println("  xorps %%xmm1, %%xmm0");
    return;
  case TY_DOUBLE:
    println("  mov $1, %%rax");
    println("  shl $63, %%rax");
    println("  movq %%rax, %%xmm1");
    println("  xorpd %%xmm1, %%xmm0");
    return;
  }

  println("  neg %%rax");
  return;
```

Construct the bit pattern `0x80000000` (for float) or `0x8000000000000000` (for double) — that's the IEEE 754 sign-bit-set, magnitude-zero pattern, equal to `-0.0`. Move it through `%rax` into `%xmm1`. Then XOR `%xmm1` into `%xmm0`. The XOR flips the sign bit and leaves the rest of the bits unchanged: `+x` becomes `-x` (and `-x` becomes `+x`, including for NaN, which negates while preserving its NaN-ness). `xorps` and `xorpd` are *XOR packed single* and *XOR packed double*; technically packed-128-bit operations, but the bit-flipping work happens entirely in the bottom 32 or 64 bits where the scalar lives, and the rest is don't-care. (`xorss`/`xorsd` don't exist in the SSE instruction set; the packed forms are what's available.)

A pair of helper-function additions on the parser side widen the dispatch. `new_add` and `new_sub` (the pointer-arithmetic-aware versions added in Chapter 4) used to gate on `is_integer`; this commit changes the gate to `is_numeric`:

```c
// num + num
- if (is_integer(lhs->ty) && is_integer(rhs->ty))
+ if (is_numeric(lhs->ty) && is_numeric(rhs->ty))
  return new_binary(ND_ADD, lhs, rhs, tok);
```

And `is_numeric` is added to `type.c`:

```c
bool is_numeric(Type *ty) {
  return is_integer(ty) || is_flonum(ty);
}
```

The check is *only* on `ND_ADD` and `ND_SUB`, the two operators that have pointer-arithmetic alternative semantics. `ND_MUL` and `ND_DIV` go through `new_binary` directly without any predicate check — multiplying or dividing a pointer by a number isn't a thing in C. The lack of guard for `ND_MUL` and `ND_DIV` means that, before this commit, `1.0 * 2.0` would already have parsed (the parser doesn't care about the operand type), but the codegen would have hit the integer arm and produced wrong code (multiplying the bit-pattern as if it were an integer through `%rax`). Now the codegen's float branch catches it and routes correctly.

Bitwise operators don't get a float arm. The C standard says `1.5 & 2.5` is ill-typed; chibicc doesn't bother to check this — it simply doesn't reach the bitwise arm because `is_flonum(node->lhs->ty)` routes float-LHS to the float branch first, where bitwise opcodes aren't handled and the trailing `error_tok("invalid expression")` fires. The error message is generic but the location (`node->tok`) is correct.

There's also a small unrelated cleanup elsewhere in `gen_expr` that ships with this commit:

```diff
- println("  and %%rdi, %%rax");
+ println("  and %s, %s", di, ax);
```

This is the bitwise `&`, `|`, `^` arms switching from hardcoded `%rdi`/`%rax` to width-selected `di`/`ax` (the variables that pick the right register width based on operand type — set up in Chapter 14 §14.5 for the unsigned-arithmetic split). This is technically an unsigned-arithmetic completion, not a floating-point change; it slipped in here as part of the same edit. The bitwise operators previously emitted 64-bit forms regardless of operand width; now they pick `%edi`/`%eax` for 32-bit operands. (The Chapter 11 §11.13 codegen quirk discussion noted this kind of overshoot; this commit closes one more case of it.)

The test file gets a row of arithmetic assertions plus a row of NaN-comparison assertions:

```c
ASSERT(6, 2.3+3.8);
ASSERT(-1, 2.3-3.8);
ASSERT(-3, -3.8);
ASSERT(13, 3.3*4);
ASSERT(2, 5.0/2);

ASSERT(0, 0.0/0.0 == 0.0/0.0);
ASSERT(1, 0.0/0.0 != 0.0/0.0);

ASSERT(0, 0.0/0.0 < 0);
ASSERT(0, 0.0/0.0 <= 0);
ASSERT(0, 0.0/0.0 > 0);
ASSERT(0, 0.0/0.0 >= 0);
```

`0.0/0.0` is the standard NaN literal. The first two assertions verify the §15.3 NaN-correct equality dance: `NaN == NaN` is false (0), `NaN != NaN` is true (1). The next four exercise NaN's incomparability: `NaN < 0`, `NaN <= 0`, `NaN > 0`, `NaN >= 0` are all false. (`>` and `>=` aren't directly emitted as separate instructions; the parser swaps the operands and routes through `ND_LT`/`ND_LE`. The `>` and `>=` tests in the assertion list still verify the dance, just through the swapped path.)

The `sizeof` tests pin the type-promotion rules:

```c
ASSERT(4, sizeof(1f+2));
ASSERT(8, sizeof(1.0+2));
```

`1f+2` is `float + int`; `get_common_type` promotes int to float, result is `float`, size 4. `1.0+2` is `double + int`; result is `double`, size 8.

### Where we are

Floating-point `+`, `-`, `*`, `/`, and unary `-` all work, on both `float` and `double`. The instruction selection is straightforward — the SSE scalar instructions match the C operators one-to-one — but the unary-minus dance through the sign bit is a small reminder that floating-point arithmetic on x86 isn't quite the same shape as integer. `is_numeric` joins `is_integer` and `is_flonum` as the predicate that the parser's pointer-arithmetic-aware `new_add`/`new_sub` use to gate the regular numeric path. The bitwise operators stayed integer-only (correctly, by C semantics).

---

## 15.5 — Floating-point in control flow

> `git checkout 0ce109302715f8186b90671a53517a63a2741022` — *Handle flonum for if, while, do, !, ?:, || and &&*

Most of the chapters since 4 have, at some point, hit a wall where a feature works in arithmetic context but not in control-flow context. Chapter 4's `int x; if (x) ...` is the original; Chapter 14 §14.4's unsigned types worked the same way. Chapter 15 has a smaller version of this wall: floating-point variables declared, arithmetic working, comparisons working — but `if (x)` for `double x` doesn't work, because the conditional codegen does `cmp $0, %rax`, which compares the *integer* register. With `x` evaluated into `%xmm0`, `%rax` holds whatever stale value it had before, and the conditional reads garbage.

The fix is in `cmp_zero`, the helper from Chapter 14 §14.5:

```c
static void cmp_zero(Type *ty) {
  switch (ty->kind) {
  case TY_FLOAT:
    println("  xorps %%xmm1, %%xmm1");
    println("  ucomiss %%xmm1, %%xmm0");
    return;
  case TY_DOUBLE:
    println("  xorpd %%xmm1, %%xmm1");
    println("  ucomisd %%xmm1, %%xmm0");
    return;
  }

  if (is_integer(ty) && ty->size <= 4)
    println("  cmp $0, %%eax");
  else
    println("  cmp $0, %%rax");
}
```

The float arms zero `%xmm1` (with `xorps`/`xorpd` self-XOR — the standard zero-an-XMM-register idiom), then compare `%xmm0` against `%xmm1` with `ucomiss`/`ucomisd`. The flags come out the same way as in §15.3's comparison dance: ZF set on equal, PF set on unordered (NaN). For control-flow purposes, the zero check is "is this register equal to zero?" — equivalent to "is `%xmm0` == 0.0?". A NaN in `%xmm0` produces ZF=0 and PF=1, which makes `je` (jump if zero) *not jump*: `if (NaN) ...` enters the then branch, which matches C's "NaN is truthy" rule (anything not equal to zero counts as true).

Actually, there's a wrinkle. The standard says `if (x)` is equivalent to `if (x != 0)` — and `NaN != 0` is true. But the codegen's `cmp_zero` followed by `je .L.else.NN` says "jump to else if equal to zero." `je` tests ZF; `ucomi*` sets ZF on equal. NaN sets ZF=0 (because `NaN == 0.0` is false). So `je` doesn't jump — and the then branch runs. That's the right behavior. The PF bit doesn't actually need to be tested for the control-flow case because the desired answer ("treat NaN as truthy") happens to fall out of testing ZF alone.

The callers of `cmp_zero` get switched from hardcoded `cmp $0, %%rax` to the helper. There are seven of them, in this commit:

```diff
case ND_COND: {
  // ...
- println("  cmp $0, %%rax");
+ cmp_zero(node->cond->ty);
  println("  je .L.else.%d", c);

case ND_NOT:
  gen_expr(node->lhs);
- println("  cmp $0, %%rax");
+ cmp_zero(node->lhs->ty);
  println("  sete %%al");

case ND_LOGAND: { ... two of them ... }
case ND_LOGOR:  { ... two of them ... }

case ND_IF: {
- println("  cmp $0, %%rax");
+ cmp_zero(node->cond->ty);

case ND_FOR (and the while/for-cond): {
- println("  cmp $0, %%rax");
+ cmp_zero(node->cond->ty);

case ND_DO: {
- println("  cmp $0, %%rax");
+ cmp_zero(node->cond->ty);
```

Seven sites. They were previously emitting integer comparisons; now they all consult the type and emit the right one. The mechanical sweep is what makes the section work — touching all seven places at once means there's no half-converted state where some control-flow shapes work for floats and others don't.

Note that this also affects `_Bool` casts. The `_Bool` cast arm in `cast()` (Chapter 14 §14.5) calls `cmp_zero` followed by `setne %al`. With `cmp_zero` now float-aware, `(_Bool)0.0` and `(_Bool)0.1` produce 0 and 1 respectively, which is what §15.2's test file already asserts. The `_Bool` test for floats was already in place; this commit makes it pass.

The test file's new entries cover the seven sites:

```c
ASSERT(0, 0.0 && 0.0);
ASSERT(0, 0.0 && 0.1);
ASSERT(0, 0.3 && 0.0);
ASSERT(1, 0.3 && 0.5);
ASSERT(0, 0.0 || 0.0);
ASSERT(1, 0.0 || 0.1);
ASSERT(1, 0.3 || 0.0);
ASSERT(1, 0.3 || 0.5);
ASSERT(5, ({ int x; if (0.0) x=3; else x=5; x; }));
ASSERT(3, ({ int x; if (0.1) x=3; else x=5; x; }));
ASSERT(5, ({ int x=5; if (0.0) x=3; x; }));
ASSERT(3, ({ int x=5; if (0.1) x=3; x; }));
ASSERT(10, ({ double i=10.0; int j=0; for (; i; i--, j++); j; }));
ASSERT(10, ({ double i=10.0; int j=0; do j++; while(--i); j; }));

ASSERT(0, !3.);
ASSERT(1, !0.);
ASSERT(0, !3.f);
ASSERT(1, !0.f);

ASSERT(5, 0.0 ? 3 : 5);
ASSERT(3, 1.2 ? 3 : 5);
```

The `for` and `do-while` tests are the most interesting: a floating-point loop counter `i` driven down by `i--` until `cmp_zero(double)` reports zero. This tests both the conditional-branch arm and the `--` decrement (which falls through to the float arithmetic from §15.4).

### Where we are

`cmp_zero` is the truth-value helper for control flow; it now handles floats. All seven control-flow shapes — `if`, `while`, `do`, `for`-cond, `?:`, `!`, and the short-circuit `&&`/`||` — work for floats and doubles, with NaN treated as truthy (which is the C standard's answer). The `_Bool` cast falls into the same machinery and now works for floats. The chapter still hasn't touched function calls (§15.6), function definitions (§15.7), or any of the variadic plumbing (§15.8 onward).

---

## 15.6 — Calling a function that takes or returns floats

> `git checkout 8ec1ebf176b88522fc4ec3980d20c78e13fdd526` — *Allow to call a function that takes/returns flonums*

Floating-point parameters in the System V x86-64 psABI use a separate register file from integer parameters. The first eight floating-point arguments go into `%xmm0` through `%xmm7`; integer arguments use `%rdi` through `%r9`. The two register files are independent: a function `void f(int a, double b, int c, double d)` puts `a`/`c` into `%rdi`/`%rsi` and `b`/`d` into `%xmm0`/`%xmm1`. Each side is consumed in declaration order, but the GP and FP slots advance independently. Returns work the same way: `%rax` for integer/pointer returns, `%xmm0` for floating-point returns.

This commit teaches chibicc's call-site codegen the parallel-FP-track. The change is concentrated in `gen_expr`'s `ND_FUNCALL` arm, with two helpers refactored.

`popf` becomes integer-indexed instead of string-indexed:

```diff
-static void popf(char *arg) {
-  println("  movsd (%%rsp), %s", arg);
+static void popf(int reg) {
+  println("  movsd (%%rsp), %%xmm%d", reg);
  println("  add $8, %%rsp");
  depth--;
}
```

Previously `popf` took a string like `"%xmm1"`; now it takes a register number and constructs the operand. The change is so that the call-site loop can iterate `popf(0)`, `popf(1)`, `popf(2)`, … instead of building register-name strings on the fly. The §15.3 caller (`popf("%xmm1")`) updates accordingly to `popf(1)`.

A new helper `push_args` recursively pushes arguments in *reverse* order:

```c
static void push_args(Node *args) {
  if (args) {
    push_args(args->next);

    gen_expr(args);
    if (is_flonum(args->ty))
      pushf();
    else
      push();
  }
}
```

The recursion goes to the end of the argument list before pushing anything; on the way back up, it pushes args from last to first. The result on the stack is: first arg on top, last arg on bottom. Reversing the order matters because the previous integer-only code did the reverse with an explicit-counter approach (`for (Node *arg = ...; ...; ) { gen_expr; push; nargs++; }` then `for (int i = nargs - 1; i >= 0; i--) pop(argreg64[i]);`), which only worked when all args used the same register file. With two register files, the index-arithmetic gets ugly (you'd need separate counters and separate index passes). The recursive approach side-steps the bookkeeping by pushing in reverse-source order and popping in source order, with each pop choosing its own register file.

The funcall arm itself is now small:

```c
case ND_FUNCALL: {
  push_args(node->args);

  int gp = 0, fp = 0;
  for (Node *arg = node->args; arg; arg = arg->next) {
    if (is_flonum(arg->ty))
      popf(fp++);
    else
      pop(argreg64[gp++]);
  }

  if (depth % 2 == 0) {
    println("  call %s", node->funcname);
  } else {
    // ... 16-byte-alignment workaround from Ch 13 §13.8 ...
  }
  // ...
}
```

`push_args` walks the arg list in reverse order and pushes each — onto the stack in the regular spot for integers, into a `pushf`/`movsd` slot for floats. The forward walk then pops each into its register: `gp` indexes into `argreg64` for integers, `fp` is the XMM register number for floats. Each side advances independently, so a call like `f(int, double, int, double)` produces `pop %rdi; popf 0; pop %rsi; popf 1` — the two trackers running in parallel.

The integer-side `argreg64` array is reused as-is; there's no `argreg_xmm` table (the XMM registers are just `%xmm0` through `%xmm7`, and `popf(n)` builds the name on the fly). Eight registers on each side; the call breaks (silently miscompiles) if a call has more than six integer or more than eight FP arguments. The integer side has carried this limitation since Chapter 5 §5.4. The FP side joins it now.

The `mov $0, %rax` line that used to be emitted right before each `call` (Chapter 5 §5.1's variadic-safety zero) was already in the dispatch for integer calls; this commit moves it to a slightly different relative position within the funcall arm but keeps it. The zero is still emitted before every call regardless of whether floats are passed — chibicc still doesn't track XMM-register count at call sites for setting `%al` correctly for variadic callees. (For non-variadic callees, the zero is harmless. For variadic callees with FP arguments, the zero is *wrong* — the ABI requires `%al` to hold the count of XMM registers used. §15.9 doesn't fix this either; it's a chibicc limitation that means calls from chibicc-compiled code into variadic functions with FP arguments may not work correctly, and the test suite avoids this case.)

For the return side, no codegen change is needed in this commit. Returns go through `%xmm0` for floats and `%rax` for integers, and the caller-side already knows how to read each. The dispatch in `gen_expr`'s post-`call` handling reads the return value from whichever register the function's return type indicates — which has been pinned down since §15.2's `load`/`store` rewrite. (`movss`/`movsd` for float-typed return reads, the existing integer load otherwise.)

`test/common` (the host-compiled side that chibicc-compiled tests link against) gains two simple helpers:

```c
float add_float(float x, float y) {
  return x + y;
}

double add_double(double x, double y) {
  return x + y;
}
```

And the test file calls them:

```c
double add_double(double x, double y);
float add_float(float x, float y);

// ...

ASSERT(6, add_float(2.3, 3.8));
ASSERT(6, add_double(2.3, 3.8));
```

The two args go into `%xmm0` and `%xmm1`; the return comes back in `%xmm0`. The caller-side code emitted by chibicc has to set both up correctly; the test depends on byte-for-byte ABI conformance with what the host compiler (which built `add_float` and `add_double`) expects.

### Where we are

The call site can pass any mix of integer and floating-point arguments through the System V psABI's two register files, with a recursive `push_args` providing the right reversal. Float returns work because `load` already routes through XMM. The variadic-safety `%al = 0` is still always emitted, which means variadic calls with FP arguments are still broken; chibicc doesn't compute the XMM-count at call sites. The callee side — defining a function that uses XMM args — is §15.7.

---

## 15.7 — Defining a function that takes or returns floats

> `git checkout c6b30568b407e7b60b6fc2929801669434e4f91a` — *Allow to define a function that takes/returns flonums*

The callee side is the prologue's "save passed-by-register arguments to the stack" loop. Chapter 5 §5.x set up the integer-only version: walk the function's parameters in order, copy each from `argreg{8,16,32,64}` to a stack slot. With FP parameters in play, that loop has to consult each parameter's type and dispatch to the right register file.

Two changes. First, a parallel `store_fp` helper next to the existing `store_gp`:

```c
static void store_fp(int r, int offset, int sz) {
  switch (sz) {
  case 4:
    println("  movss %%xmm%d, %d(%%rbp)", r, offset);
    return;
  case 8:
    println("  movsd %%xmm%d, %d(%%rbp)", r, offset);
    return;
  }
  unreachable();
}
```

`store_fp` takes a register number (0–7, indexing `%xmm0`–`%xmm7`), a stack offset (relative to `%rbp`), and a size (4 for `float`, 8 for `double`). It emits `movss` or `movsd` accordingly. The `unreachable()` arm is for cases that shouldn't happen — a future `long double` would hit it, but §15.11 makes `long double` an alias for `double` (size 8), so the trap stays unreachable. The shape is identical to `store_gp`'s 1/2/4/8 dispatcher; together they handle all the parameter-spill cases.

Second, the prologue's parameter-spill loop runs two counters in parallel:

```diff
-int i = 0;
-for (Obj *var = fn->params; var; var = var->next)
-  store_gp(i++, var->offset, var->ty->size);
+int gp = 0, fp = 0;
+for (Obj *var = fn->params; var; var = var->next) {
+  if (is_flonum(var->ty))
+    store_fp(fp++, var->offset, var->ty->size);
+  else
+    store_gp(gp++, var->offset, var->ty->size);
+}
```

Each parameter is spilled to its own slot (`var->offset`, computed during parsing, the same way as for integer parameters since Chapter 5). The choice between `store_fp` and `store_gp` depends on the parameter's type. The two counters advance independently — exactly like the call-site code in §15.6. A function `void f(int a, double b, int c, double d)` spills `%rdi`→`a` (gp=0), `%xmm0`→`b` (fp=0), `%rsi`→`c` (gp=1), `%xmm1`→`d` (fp=1).

The return side needs no codegen change here either. The function body's `return expr;` evaluates `expr` and leaves it in either `%rax` or `%xmm0` depending on type — chibicc's existing `gen_expr` already gets that right because the float arms from §15.2 onward route through XMM registers. The function's ABI return-register is therefore *whatever register `gen_expr` wrote to* for the return expression, which matches the psABI for the function's declared return type.

The test file gets two callee-side helpers:

```c
float add_float3(float x, float y, float z) {
  return x + y + z;
}

double add_double3(double x, double y, double z) {
  return x + y + z;
}
```

— and the assertions:

```c
ASSERT(7, add_float3(2.5, 2.5, 2.5));
ASSERT(7, add_double3(2.5, 2.5, 2.5));
```

These three-argument forms exercise three XMM-register parameters (`%xmm0`, `%xmm1`, `%xmm2`), each spilled to a stack slot, then read back through `movss`/`movsd` for arithmetic. The arithmetic chain `x + y + z` produces a final value in `%xmm0`, which is the function's return register. Round-trip from caller (§15.6) to callee (this section), through three argument slots, through two arithmetic operations, back through the return register.

The integration with §14.2's variadic spill area is also worth noting. The variadic prologue spills *all* of `%rdi`–`%r9` and `%xmm0`–`%xmm7` to the save area, regardless of how many fixed arguments were declared. The non-variadic prologue (the one this section just modified) spills only the *declared* parameters. The two paths don't conflict — `fn->va_area` is `NULL` for non-variadic functions, so the variadic spill block is skipped, and only `store_gp`/`store_fp` for the declared params runs.

§15.9 will fold the variadic-spill prologue into the same loop framework, so a variadic function with FP parameters spills its declared FP params first (eating into the XMM register count) and then runs the variadic spill for the rest.

### Where we are

Functions defined in chibicc-compiled code can take and return `float` and `double` parameters through the System V psABI's two-register-file convention. The prologue spills declared float params from `%xmm0`–`%xmm7` to stack slots via the new `store_fp` helper; the return side reuses `gen_expr`'s float arms. The pair (caller-side §15.6 + callee-side §15.7) is the structural mirror of Chapter 14's §14.1/§14.2 pair for variadic calls. The next three commits — §15.8, §15.9, §15.10 — handle the corner cases: default argument promotion, variadic floats, and constant-expression folding.

---

## 15.8 — Default argument promotion

> `git checkout 8b14859f63a8389882bdb9330de592a112affa18` — *Implement default argument promotion for float*

C has a rule that when a function is called with arguments that aren't covered by a prototype — either because the prototype is missing (K&R style) or because the argument is in the variadic tail — the argument types undergo *default argument promotion*. The rule is: any argument of type `char`, `short`, or any integer narrower than `int` is promoted to `int`; and any argument of type `float` is promoted to `double`. The motivation is variadic-call ABI predictability: `printf` doesn't know the type of `%d`/`%f`/`%s` arguments at compile time, but the ABI guarantees that all integer arguments are at least `int`-wide and all floating-point arguments are `double`-wide. `va_arg(ap, double)` is well-defined; `va_arg(ap, float)` would not be.

Chibicc's call-site code (§15.6) doesn't promote anything — a `float` argument passed to a variadic function would land in `%xmm0` as a `float` (4 bytes), and the variadic callee would `va_arg(ap, double)` it, reading 8 bytes — garbage in the upper 4. This commit is the four-line fix:

```c
} else if (arg->ty->kind == TY_FLOAT) {
  // If parameter type is omitted (e.g. in "..."), float
  // arguments are promoted to double.
  arg = new_cast(arg, ty_double);
}
```

The `else if` plugs into the existing `funcall` argument-walk, where `param_ty` is the parser's pointer through the prototype. When `param_ty` is NULL — meaning we've walked off the end of the prototype's fixed parameters and into the variadic tail — `funcall` previously did nothing (no cast, no check). Now it inserts an `(double)` cast on `float` arguments.

The reason the fix is so small is that the existing parser machinery already handles all the surrounding work. `new_cast(arg, ty_double)` wraps the argument node in `ND_CAST`, and the existing cast codegen (Chapter 10 §10.x, extended in §15.2) handles the float-to-double conversion via `cvtss2sd`. From the funcall side, the argument is now a `double` like any other, and the §15.6 code routes it to `%xmm0`/`%xmm1`/etc. through the normal FP register file.

Note what *isn't* here: integer promotion. The standard says `char` and `short` arguments to variadic calls also get promoted (to `int`). Chibicc's funcall doesn't do that — the test suite doesn't depend on it (calling `printf("%d", (char)42)` works because the call-site ABI for `int` and `char` happens to land in the same register and the upper bits are don't-care for `%d`'s read), so the rule isn't enforced. Errata candidate.

The C-standard wording is also slightly broader than what the chibicc code checks. The standard says default promotion applies to any *unprototyped* argument — including arguments to `f()` shapes (K&R-style declarations with no prototype). Chibicc's §14.3 turns `f()` into variadic, which means `param_ty` for those calls walks off the end immediately on the first arg, and the float promotion fires. So the chibicc behavior is correct for the cases that the test suite exercises (variadic and `f()`), even if the structural argument is "all unprototyped args" rather than chibicc's "anything past the fixed prototype."

The test pins one case:

```c
ASSERT(0, ({ char buf[100]; sprintf(buf, "%.1f", (float)3.5); strcmp(buf, "3.5"); }));
```

`(float)3.5` is explicitly `float`. `sprintf` is variadic. Without the promotion, `%xmm0` would hold a `float`-precision 3.5 (the bit pattern `0x40600000`), and glibc's `sprintf` would `va_arg(ap, double)` it, reading 8 bytes from the `va_list`'s FP slot — but the slot holds the 4-byte float bit pattern in the bottom and 4 bytes of garbage in the top. The result would be a wildly-incorrect printed value. With the promotion, the float gets cast to `double` (`cvtss2sd`) before hitting `%xmm0`, the double bit pattern lands in 8 bytes of slot, and `va_arg(ap, double)` reads it correctly.

### Where we are

A `float` argument to a variadic call is now promoted to `double` before hitting `%xmm0`. Three lines of code, one explicit cast insertion. Integer promotions for `char`/`short` are *not* implemented; the test suite doesn't exercise the cases where they would matter. The fix is C-standard-required and bug-for-bug compatible with what real compilers emit at the call site.

---

## 15.9 — Variadic floats

> `git checkout e452cf721511dbf0d7f8c8f469f2dd67d8a5ee93` — *Support variadic function with floating-point parameters*

Chapter 14 §14.2 set up the register-save area for variadic functions: 136 bytes, with a 24-byte header followed by 48 bytes of GP slots and 64 bytes of XMM slots. The prologue spilled all six GP registers and all eight XMM registers unconditionally. The four-field `__va_elem` header was populated with `gp_offset = 8 × (number of fixed integer params)`, `fp_offset = 0`, `overflow_arg_area = ...`, `reg_save_area = ...`.

The `fp_offset = 0` was a placeholder. The System V psABI says `fp_offset` should start at the byte offset where the first unused FP register slot begins — meaning *past* the GP region (which is 48 bytes) plus 8 bytes per fixed FP parameter that has already been consumed. With `fp_offset = 0`, glibc's `va_arg(ap, double)` would read from `reg_save_area + 0`, which is the GP slot for `%rdi` — entirely wrong. The placeholder was harmless only because no test until this commit exercised `va_arg(ap, double)`.

This commit replaces the placeholder. The prologue's variadic-spill block, in `emit_text`:

```diff
 if (fn->va_area) {
-  int gp = 0;
-  for (Obj *var = fn->params; var; var = var->next)
-    gp++;
+  int gp = 0, fp = 0;
+  for (Obj *var = fn->params; var; var = var->next) {
+    if (is_flonum(var->ty))
+      fp++;
+    else
+      gp++;
+  }
+
   int off = fn->va_area->offset;

   // va_elem
   println("  movl $%d, %d(%%rbp)", gp * 8, off);
-  println("  movl $0, %d(%%rbp)", off + 4);
+  println("  movl $%d, %d(%%rbp)", fp * 8 + 48, off + 4);
   println("  movq %%rbp, %d(%%rbp)", off + 16);
   println("  addq $%d, %d(%%rbp)", off + 24, off + 16);
```

Two changes. First, the loop that counts fixed parameters splits — fixed integer parameters are counted as `gp`, fixed floating-point parameters as `fp` — using `is_flonum(var->ty)` to decide. Second, the `fp_offset` field is now `fp * 8 + 48` instead of `0`. The `+48` skips past the 48-byte GP region; the `fp * 8` skips the FP slots already consumed by fixed FP parameters.

The 8-byte stride here is worth pausing on. The System V ABI specifies *16-byte* slots in the FP region (each XMM register is 128 bits, and the layout of `va_list` assumes 16-byte stride for `va_arg` walking). Chibicc's spill area uses 8-byte slots (the prologue's `movsd %xmm0, off+72(%rbp)` writes 8 bytes, not 16). This is a non-conformity that's been present since §14.2 and that this commit doesn't fix. It works for the tests in the suite because each variadic call passes only one or zero FP arguments — so glibc reads a single `double` from `reg_save_area + fp_offset`, finds the chibicc-written 8 bytes there, and is done. A test passing two or more FP arguments through a variadic call into glibc would expose the bug. None do.

The `gp` counter computation is also worth examining. It used to start at 0 and increment for every parameter (because all parameters were assumed to be integer, before §15.6). Now it increments only for non-flonum parameters — meaning the count accurately reflects the number of integer registers consumed by fixed parameters. The `gp_offset = gp * 8` line that follows is therefore correct after this change, where it would have been off-by-one-per-FP-fixed-param before. (The §14.2 prose noted that `gp_offset` starts at `8 × (number of fixed integer parameters)`; the fix is bringing the implementation into line with the prose.)

The test:

```c
ASSERT(0, ({ char buf[100]; fmt(buf, "%.1f", (float)3.5); strcmp(buf, "3.5"); }));
```

`fmt` is the chibicc-compiled variadic function from §14.2 (which copies `__va_area__` into a user-declared `va_list` and hands it to glibc's `vsprintf`). The trailing `(float)3.5` argument is float-promoted to double by §15.8's call-site cast, then passed in `%xmm0`. `fmt`'s prologue spills `%xmm0` to the FP region of `__va_area__` (offset off+72 in the spill area). `vsprintf` walks the `va_list`, uses `fp_offset = 48` (because `fmt` has zero fixed FP parameters), reads 8 bytes from `reg_save_area + 48` — and finds the spilled double. Correct.

This is the loop closing on §14.2's spill area. The XMM half is now consumed.

### Where we are

The `fp_offset` field of the variadic header is finally set correctly. A variadic function defined in chibicc-compiled code can receive floating-point arguments through the spill area, and a `va_arg(ap, double)` walk via glibc's macros reads them correctly (as long as there's only one FP argument, owing to chibicc's 8-byte-vs-ABI-16-byte stride mismatch). The §14.2 spill area's XMM half is no longer dead code. The chapter's variadic story is closed.

---

## 15.10 — Floating-point constant expressions

> `git checkout ffea4219b1f4ebe7c06cecc6c221cb0aab3a03ea` — *Add flonum constant expression*

The constant-expression evaluator (`eval`/`eval2`/`eval_rval` from Chapter 12 §12.7, extended in Chapter 14 §14.9 for unsigned arms) returns `int64_t`. With float constants in play, that's not enough — folding `1.5 + 2.5` into the right value (`4.0`, not `4`) needs a double-returning evaluator. This commit adds a parallel `eval_double` and a small dispatcher to route between the two.

The forward declaration lands at the top of `parse.c`:

```c
static double eval_double(Node *node);
```

`eval2` — the existing `int64_t`-returning evaluator — gets a one-line dispatch at the top:

```c
static int64_t eval2(Node *node, char **label) {
  add_type(node);

  if (is_flonum(node->ty))
    return eval_double(node);
  // ... existing integer arms ...
}
```

If the type of the node is floating-point, hand off to `eval_double`. The cast back to `int64_t` happens implicitly: `eval2`'s return type narrows the double via C's `(int64_t)d` conversion, which is the same semantic the runtime cast machinery (Chapter 10) implements. So `eval((int)1.5)` returns `1` — the parse-time fold matches the runtime cast.

`eval_double` is a parallel switch over the same node kinds, returning `double`:

```c
static double eval_double(Node *node) {
  add_type(node);

  if (is_integer(node->ty)) {
    if (node->ty->is_unsigned)
      return (unsigned long)eval(node);
    return eval(node);
  }

  switch (node->kind) {
  case ND_ADD:
    return eval_double(node->lhs) + eval_double(node->rhs);
  case ND_SUB:
    return eval_double(node->lhs) - eval_double(node->rhs);
  case ND_MUL:
    return eval_double(node->lhs) * eval_double(node->rhs);
  case ND_DIV:
    return eval_double(node->lhs) / eval_double(node->rhs);
  case ND_NEG:
    return -eval_double(node->lhs);
  case ND_COND:
    return eval_double(node->cond) ? eval_double(node->then) : eval_double(node->els);
  case ND_COMMA:
    return eval_double(node->rhs);
  case ND_CAST:
    if (is_flonum(node->lhs->ty))
      return eval_double(node->lhs);
    return eval(node->lhs);
  case ND_NUM:
    return node->fval;
  }

  error_tok(node->tok, "not a compile-time constant");
}
```

The shape mirrors `eval2`. The integer-typed dispatch at the top — if the *node*'s type is integer — uses the integer evaluator (`eval`) and returns the result as a double. If the integer is unsigned, the `(unsigned long)` cast preserves the wraparound semantics that Chapter 14 §14.9 set up. The cast arm checks the operand's type: if the operand is float, recurse into `eval_double`; if it's integer, route to `eval` and let the implicit `int64_t → double` conversion happen.

Notice the operators that *aren't* in this switch: `ND_MOD`, `ND_BITAND`, `ND_BITOR`, `ND_BITXOR`, `ND_SHL`, `ND_SHR`, `ND_LT`, `ND_LE`, `ND_EQ`, `ND_NE`, `ND_LOGAND`, `ND_LOGOR`, `ND_NOT`. The bitwise and shift operators don't apply to floats. The comparisons return `int`, so they go through `eval`/`eval2`, which folds them as integer-valued (with operand evaluation routed back through `eval_double` if the operands are float). The logical operators do the same. `ND_MOD` would be the only float-applicable arm missing — and chibicc's parser doesn't accept `%` on floats (the C standard agrees, `%` is integer-only), so it's correct to omit.

The `write_gvar_data` function (Chapter 12 §12.7's global initializer writer) gets two new top-of-function arms:

```c
if (ty->kind == TY_FLOAT) {
  *(float *)(buf + offset) = eval_double(init->expr);
  return cur;
}

if (ty->kind == TY_DOUBLE) {
  *(double *)(buf + offset) = eval_double(init->expr);
  return cur;
}
```

This is what makes file-scope `float g = 1.5;` work: the initializer is evaluated via `eval_double` at parse time, and the resulting double is written into the `.data` blob (truncated to `float` if the target type is `float`). The bit pattern then ends up in the assembly output as raw bytes via `emit_data`'s existing `.byte` emission. No new `Relocation` is needed — the initializer is a pure constant, no symbolic reference, no fixup at link time.

The test file:

```c
float g40 = 1.5;
double g41 = 0.0 ? 55 : (0, 1 + 1 * 5.0 / 2 * (double)2 * (int)2.0);

// in main():
ASSERT(1, g40==1.5);
ASSERT(1, g41==11);
```

`g40 = 1.5` is the simplest case: the literal folds to itself, written as 4 bytes of `0x3FC00000`. `g41` is a torture test: ternary, comma, mixed integer-and-float arithmetic, two casts, all in one expression. The folded value is `11.0`, and the assertion `g41 == 11` (with `11` an int, promoted to double in the comparison) passes.

### Where we are

The constant-expression evaluator has a parallel `eval_double` that handles floating-point folding. `eval2` dispatches into it when the top-level node is float-typed. Global initializers `float x = ...;` and `double x = ...;` write the right bit pattern into `.data` at parse time. The integer constant-expression evaluator and the float one share node-kind handling but use different return types. The chapter's last big mechanism is in.

---

## 15.11 — `long double` as `double`

> `git checkout 9bf96124ba1e0cb95f491bd0c91d4e9c7a9850da` — *Add "long double" as an alias for "double"*

The chapter's closer is one line in `declspec`'s switch:

```diff
 case DOUBLE:
+case LONG + DOUBLE:
   ty = ty_double;
   break;
```

The `LONG` and `DOUBLE` bits in the declspec counter combine via addition (the same machinery from Chapter 14 §14.5), so `long double` lights both bits, and the switch resolves the combined value to `ty_double`. There's no `ty_long_double` — chibicc collapses `long double` into `double` outright.

Real C distinguishes `long double` from `double`. On x86-64 Linux, `long double` is 80-bit extended precision (sometimes called `long double` or `x87 extended`); `sizeof(long double)` is 16 (with padding). Chibicc's `long double` is just `double` — `sizeof(long double)` is 8, and there's no extended-precision arithmetic. The difference matters for numerical code that depends on the extra mantissa bits, but the chibicc test suite doesn't exercise such code.

This is a chibicc-specific aliasing — a deliberate simplification, not a parsing oversight. Implementing `long double` properly would require: a new `TY_LDOUBLE` kind, the x87 instruction set (`fld`/`fstp`/`faddp` and friends — chibicc avoids the x87 stack altogether), separate calling conventions (the System V ABI passes `long double` on the stack, not in XMM), and a separate evaluator path in §15.10. The cost would be substantial; the benefit would be exercising features no chibicc-compiled test depends on. Errata candidate.

The test:

```c
ASSERT(8, sizeof(long double));
```

— eight, not sixteen, because chibicc says so.

### Where we are

`long double` parses as `double`. One line in `declspec`. The chapter is over.

---

## Where the chapter leaves us

Eleven commits, eleven sections. The floating-point story arrives complete enough to compile real test programs that pass floats around: literals, locals, arithmetic, comparison, control flow, function calls and returns, default argument promotion, variadic floats, constant expressions, and `long double`. The only deliberate approximation is the `long double = double` collapse.

| Commit | Topic |
|---|---|
| `1e57f72` | Floating-point literals. New `TY_FLOAT`/`TY_DOUBLE` kinds, `is_flonum` predicate, `Token->fval`, `Node->fval`, `read_number` dispatcher with `strtod` fallback. Codegen loads literals as integer immediates bit-cast into XMM via `%rax`. No anonymous-global pattern. |
| `29de46a` | `float`/`double` locals and casts. `load`/`store` learn `movss`/`movsd`. Cast table grows from 8×8 to 10×10 with eighteen new conversion strings, including the unsigned-64-to-double dance. |
| `cf9ceec` | Float comparison and NaN-correct equality. `pushf`/`popf` helpers. `ucomiss`/`ucomisd` set ZF/CF/PF; the `==` and `!=` arms test `setp`/`setnp` to reject NaN-vs-NaN as equal. `<` and `<=` reuse the unsigned setters (`seta`/`setae`) because of how `ucomi*` writes CF. `get_common_type` learns the float promotion cascade. |
| `83f76eb` | Float arithmetic. `addss`/`subss`/`mulss`/`divss` (and `sd` variants). Unary `-` flips the sign bit via `xorps`/`xorpd`. `is_numeric` predicate gates `new_add`/`new_sub`. Bitwise width-fix slips in. |
| `0ce1093` | Float in control flow. `cmp_zero` learns float arms: `xorps`/`xorpd` to zero `%xmm1`, `ucomiss`/`ucomisd` to compare. Seven call-site `cmp $0, %rax`s switch to `cmp_zero(node->ty)`. NaN is truthy by virtue of how `ucomi*` writes ZF. |
| `8ec1ebf` | Float function calls. Recursive `push_args` reverses argument order. `popf` becomes integer-indexed. The funcall arm walks the args in source order, with `gp`/`fp` counters running independently into `%rdi`–`%r9` and `%xmm0`–`%xmm7`. |
| `c6b3056` | Float function definitions. `store_fp` joins `store_gp`. The prologue's parameter-spill loop uses `is_flonum` to dispatch to the right helper. Returns work because `gen_expr` already routes floats through XMM. |
| `8b14859` | Default argument promotion. Three lines in `funcall`: float arguments past the prototype's fixed parameters get a `(double)` cast. Variadic calls and `f()`-as-variadic both benefit. Integer promotions (`char`/`short` to `int`) not implemented; errata candidate. |
| `e452cf7` | Variadic floats. The `fp_offset` placeholder in §14.2's spill block becomes `fp * 8 + 48`. The `gp` counter splits into `gp` and `fp`, each counting only its register file. The §14.2 XMM half of the save area finally has a reader. |
| `ffea421` | Float constant expressions. Parallel `eval_double` next to `eval`/`eval2`. `eval2` dispatches by `is_flonum(node->ty)`. `write_gvar_data` writes float/double initializers via `eval_double` at parse time. |
| `9bf9612` | `long double` as `double`. One switch arm: `case LONG + DOUBLE: ty = ty_double;`. No extended precision; errata candidate. |

Three structural moves carry forward.

The first is the *floating-point register file as a parallel track*. Chibicc's argument-passing machinery, which had been single-track since Chapter 5, splits into two parallel tracks in §15.6 (call site) and §15.7 (definition). The two counters (`gp` and `fp`) advance independently, indexing into separate register files (`argreg64` for integer, `%xmm0`–`%xmm7` for FP), with each parameter dispatched to one or the other based on `is_flonum(arg->ty)`. The pattern of "counter per register file, dispatch by predicate" is small and clean; future register-file additions (a hypothetical SSE-vector or AVX track) would follow the same shape. The same pattern shows up again in §15.9, where the variadic-spill prologue counts fixed integer and fixed FP parameters separately.

The second is the *NaN-aware comparison via the parity flag*. The integer comparison codegen (since Chapter 1) tested ZF and CF; the float comparison adds PF. The dance — `setnp %dl; and %dl, %al` — is small but it's the substance of §15.3. The NaN-correctness extends through control flow (§15.5's `cmp_zero` doesn't need PF because the desired NaN-as-truthy behavior happens to fall out of testing ZF alone, but it would have been wrong in the other direction), and through the variadic path (§15.9's `va_arg(ap, double)` reads bit patterns that can include NaN, which `vsprintf` formats as `"nan"`). NaN handling is a structural piece of how floats fit into a language whose runtime semantics were designed for integers, and chibicc gets it right at the comparison-instruction level.

The third is the *psABI conformance thread continues*. Chapter 13 §13.8/§13.9 plus Chapter 14 §14.1/§14.2/§14.8 had given five corrections. Chapter 15 adds three more: §15.6 and §15.7 implement the two-register-file calling convention end-to-end; §15.8 implements the default-argument-promotion rule that the ABI relies on for variadic FP arguments; §15.9 fixes `fp_offset` so glibc can walk the spill area's FP half. The thread now stands at eight corrections. A still-unfixed item is `fp * 8 + 48` versus the ABI's 16-byte stride — chibicc's variadic FP parameters work in the one-argument case the test suite uses, but a multi-FP-argument variadic call into glibc would expose the stride mismatch. Errata candidate.

The chapter doesn't add to the *canonicalization-at-parse-time* count (still eight). The chapter *does* add one to the *pre-factor-before-feature* count: the `load`/`store` reshape in §15.2 from `if` to `switch` is a refactor done as part of the float feature, but the recursive `push_args` in §15.6 is a deeper rework that replaces a prior counter-based approach with a tree-walk approach so that two register files can be handled — that's a pre-factor in spirit even if it landed in the same commit as the feature it enables. Current count: seven. The chapter doesn't add a fifth namespace. The chapter doesn't add to `VarAttr` (the channel stays at four fields — same forecast as Chapter 14, this time correct). The chapter does extend `is_typename` by two (`float`, `double`).

A few standing notes carried forward to Chapter 16:

- The `is_unsigned` flag on `Type` is irrelevant to float types. Chapter 14's flag-on-existing-kind pattern is for the integer signedness axis only. New axes (float, in this chapter; thread-local or atomic in future hypothetical chapters) get new kinds.
- The `register-save area` is fully consumed now. The 64-byte FP region is populated by §14.2's prologue and read by `va_arg(ap, double)` walks via §15.9's `fp_offset`. The 8-byte-per-slot stride is non-conforming (the ABI specifies 16-byte) but works for the single-FP-argument case the test suite exercises.
- The `__va_area__` magic local-variable name continues to be the chibicc-specific user-side hook for `va_start`. No new uses added in Chapter 15; same trick from Chapter 14.
- The `cast_table` is now 10×10. Future axis additions (a hypothetical `_Decimal32`, or the unimplemented true `long double`) would extend it again. The flag-or-kind decision for new axes follows the §14.5 precedent: orthogonal axes use a flag; new dimensions get new kinds.
- The constant-expression evaluator (`eval`/`eval2`/`eval_rval`/`eval_double`) is now a quartet. Chapter 14 added unsigned arithmetic; Chapter 15 added float arithmetic. The dispatch is by `is_flonum(node->ty)` at the top of `eval2`. Future axis additions would fit the same shape.
- The argreg track for FP is `%xmm0`–`%xmm7`; for integer it's `argreg64` (still six wide). Future variadic-stack-overflow handling would need to populate `overflow_arg_area` from caller-side stack pushes, which chibicc doesn't yet do — the "more than 6 integer args silently miscompiles" remains; "more than 8 FP args silently miscompiles" joins it. Errata candidates.
- The `is_numeric` predicate joins `is_integer` and `is_flonum` as a parser-side type discriminator. `is_numeric` is used by `new_add` and `new_sub`'s pointer-arithmetic gate; the other two are used widely by codegen and type-system code.
- The `eval_double` evaluator handles `ND_ADD`, `ND_SUB`, `ND_MUL`, `ND_DIV`, `ND_NEG`, `ND_COND`, `ND_COMMA`, `ND_CAST`, `ND_NUM` and falls through (via `is_integer(node->ty)` check) to `eval` for integer-typed nodes. It does *not* handle the bitwise/shift operators (correctly), nor `ND_MOD` (correctly — `%` is integer-only).
- The `mov $0, %rax` (Chapter 5 §5.1) is still emitted before every call, including calls into variadic functions with FP arguments. For non-variadic calls, harmless. For variadic calls with FP arguments, technically wrong (`%al` should hold the count of XMM registers used, not zero) — but glibc's variadic functions tolerate `%al = 0` when their format string doesn't ask for FP arguments. The §15.8 + §15.9 path works because the test suite's only variadic-FP-call is `fmt(buf, "%.1f", (float)3.5)` and glibc reads from the spill area regardless of `%al`. Errata candidate, lower priority.
- The `Initializer` tree (Chapter 12) is unchanged in Chapter 15. Float initializers reuse the existing tree shape via `write_gvar_data`'s new `TY_FLOAT`/`TY_DOUBLE` arms.
- The anonymous-global pattern (Chapter 13 §13.4) was *not* used for float literals. Inline integer-immediate-as-bit-pattern is the chosen approach. Worth tracking as a precedent: when a literal can be encoded as a 64-bit immediate plus a bit-cast, chibicc inlines; when it can't (string literals can't, since they're variable-length), the anonymous global pattern applies.
- The `unreachable()` macro lives in `chibicc.h`. Chapter 15 adds one new caller: `store_fp`'s default arm, which is statically unreachable today (only sizes 4 and 8 ever appear) but defensive against a future `long double` that doesn't alias `double`.

Forward references for Chapter 16 (the compiler driver, commits 150–157):

- The driver is the executable wrapper around the compiler proper. Today, `chibicc input.c` produces an assembly file (or, with the test harness, an object via the host `cc`). Chapter 16 introduces a real driver that handles `-o`, `-S`, `-c`, `-E`, multiple input files, and the assembly-and-link pipeline. The compiler proper's interface won't change much; the driver wraps it.
- Eight commits, mostly driver-shape changes. Not a deep walk through codegen or parsing; more about how chibicc presents itself to the outside world. Forecast: shorter chapter than 15, possibly ~6,000–8,000 words, depending on how much of the driver mechanism is genuinely novel.
