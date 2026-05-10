# Chapter 9 — Structs and unions

> Commits covered: `f814033`, `9443e4b`, `dfec115`, `e1e831e`, `f0a018a`, `11e3841`, `bef0543`. Seven commits — the introduction of `struct`, member alignment, local-variable alignment to match, named struct tags, the `->` operator as syntactic sugar, the addition of `union`, and finally struct (and union) assignment.

Chapter 8 closed with chibicc holding everything it needed for compound types: a real scope chain, debuggable output, in-language tests. None of those are *about* compound types, but each one ends up being load-bearing here. Block scope is the mechanism that lets a tag declared inside a block disappear at the closing brace. The C-form test files in `test/struct.c` and `test/union.c` are how the new behavior is exercised — twenty-some assertions per file, each a self-contained statement expression. And per-statement `.loc` directives mean that an error in struct member parsing still points at the right source line.

Chapter 9 introduces the first compound type. Before this chapter, every value chibicc knew how to handle fit in a single 8-byte register: ints, chars, pointers, and arrays-as-pointers. After this chapter, chibicc can model and pass around *aggregates* — values built from named members with their own types and offsets. The implementation grows alignment infrastructure, a second namespace (struct tags, parallel to ordinary variables), one new desugaring at parse time, and a codegen path that breaks the everything-fits-in-rax assumption for the first time.

The seven sections:

- **§9.1** — `struct` (commit 49).
- **§9.2** — Aligning struct members (commit 50).
- **§9.3** — Aligning local variables (commit 51).
- **§9.4** — Struct tags (commit 52).
- **§9.5** — The `->` operator (commit 53).
- **§9.6** — `union` (commit 54).
- **§9.7** — Struct assignment (commit 55).

There's no concept interlude. The chapter mapping doesn't call for one, and chibicc's alignment story is mechanical enough that an in-prose paragraph in §9.2 covers what a reader needs. The natural digression would be the System V ABI's struct layout rules, but chibicc does the bare minimum that produces the right answer for the test corpus and doesn't engage with the ABI's more elaborate cases (nested aggregates passed by value across function calls, bit-fields, packed structs). The prose mirrors that scope.

The chronological dates of these commits do not match their position on `main`. Three of them (`f814033`, `dfec115`, `bef0543` — the original struct introduction, the local-variable alignment, and struct assignment) are dated 2019 and 2020-04. The rest (member alignment, struct tags, `->`, union) are dated 2020-08 through 2020-10. Notably, `dfec115` (local-variable alignment, dated 2019-08-09) appears in canonical order *after* `9443e4b` (struct member alignment, dated 2020-08-30) — a year earlier in wall-clock time but a position later on `main`. The chapter follows `main` order, which is the order a reader will see when walking `git log` and the order the chapter mapping pins.

---

## 9.1 — `struct`

> `git checkout f814033d04c4cefdbcf8174d65011d484d69303c` — *Add struct*

This is the chapter's biggest single commit. About 150 lines of diff add a new `Type` kind, a new `Member` struct, a new node kind, a new keyword, three new parser functions, an `is_typename` extension, a `declspec` branch, a `postfix` branch, an `add_type` rule, and codegen for member access. After this commit, chibicc can declare and use anonymous structs:

```c
struct {int a; int b;} x;
x.a = 1;
x.b = 2;
return x.a + x.b;
```

There are no tags yet (those arrive in §9.4), no alignment (§9.2), no `->` (§9.5), no struct assignment (§9.7), and `int` is still 8 bytes (it stays 8 bytes until Chapter 10). What this commit does deliver is the structural scaffold that the next six commits build on.

### A new type kind and a `Member` chain

```c
typedef enum {
  TY_CHAR,
  TY_INT,
  TY_PTR,
  TY_FUNC,
  TY_ARRAY,
  TY_STRUCT,
} TypeKind;
```

`TY_STRUCT` joins the type enum. The `Type` struct gains one new field:

```c
// Struct
Member *members;
```

A pointer to the head of a singly linked list of `Member`s. Each `Member` holds the four pieces of information needed to find the value of `s.x` given the address of `s`:

```c
struct Member {
  Member *next;
  Type *ty;
  Token *name;
  int offset;
};
```

`ty` is the member's type, `name` is the token that declared it (kept around so error messages can point at the source), `offset` is the byte offset from the start of the containing struct, and `next` chains the members in declaration order. There's no symbol-table lookup structure here — member resolution is by linear scan of the `members` list, which is fine because chibicc isn't going to see structs with hundreds of members in the programs it cares about.

`Member` is forward-declared at the top of `chibicc.h`:

```c
typedef struct Member Member;
```

joining the existing `Type` and `Node` typedefs.

### Parsing a struct type

The grammar for `struct {...}` lives in three new functions:

```c
// struct-decl = "{" struct-members
static Type *struct_decl(Token **rest, Token *tok) {
  tok = skip(tok, "{");

  // Construct a struct object.
  Type *ty = calloc(1, sizeof(Type));
  ty->kind = TY_STRUCT;
  struct_members(rest, tok, ty);

  // Assign offsets within the struct to members.
  int offset = 0;
  for (Member *mem = ty->members; mem; mem = mem->next) {
    mem->offset = offset;
    offset += mem->ty->size;
  }
  ty->size = offset;

  return ty;
}
```

`struct_decl` is called by `declspec` when the lookahead token is `struct`. It allocates a fresh `TY_STRUCT` `Type`, parses the brace-enclosed member list into that type, then walks the resulting member list assigning offsets. Each member's offset is the running sum of preceding members' sizes, and the total struct size is the final running sum. There's no alignment yet — that's §9.2 — so a `struct {char a; int b;}` lays out as `a` at offset 0, `b` at offset 1, with total size 9. (This is in fact what the test asserts in this commit: `ASSERT(9, ({ struct {char a; int b;} x; sizeof(x); }));`. The next commit deletes that assertion and replaces it with the aligned answer.)

The member list itself is parsed by `struct_members`:

```c
// struct-members = (declspec declarator (","  declarator)* ";")*
static void struct_members(Token **rest, Token *tok, Type *ty) {
  Member head = {};
  Member *cur = &head;

  while (!equal(tok, "}")) {
    Type *basety = declspec(&tok, tok);
    int i = 0;

    while (!consume(&tok, tok, ";")) {
      if (i++)
        tok = skip(tok, ",");

      Member *mem = calloc(1, sizeof(Member));
      mem->ty = declarator(&tok, tok, basety);
      mem->name = mem->ty->name;
      cur = cur->next = mem;
    }
  }

  *rest = tok->next;
  ty->members = head.next;
}
```

The shape mirrors `declaration` from Chapter 6: an outer loop over declarations (each starts with a `declspec`), an inner loop over comma-separated declarators sharing that base type, with `;` as the terminator. The dummy-`head` sentinel pattern is the same one chibicc uses everywhere it builds a list at the trailing edge of a parse loop. Each `Member`'s `name` is just the declarator's name token — `declarator` already returns a `Type` with its `name` field set, and `struct_members` lifts it onto the `Member`.

This means struct-member parsing reuses the entire declarator machinery from Chapter 6. A member of pointer type, array type, or both works because `declarator` already handles those:

```c
struct {char a[3]; char b[5];} x;
```

is parsed by calling `declarator` for `a[3]` and again for `b[5]`, each returning a `Type` whose kind is `TY_ARRAY` and whose size is `3` or `5`. The struct's total size is then `3 + 5 = 8`. This composability is the payoff of the Chapter 6 `declarator` rewrite: when a new context needs to parse a typed name, it can call the existing function and get pointer-of-array-of-pointer-of-int for free.

The third new function is the member lookup:

```c
static Member *get_struct_member(Type *ty, Token *tok) {
  for (Member *mem = ty->members; mem; mem = mem->next)
    if (mem->name->len == tok->len &&
        !strncmp(mem->name->loc, tok->loc, tok->len))
      return mem;
  error_tok(tok, "no such member");
}
```

Linear scan over the `members` list, comparing the requested name token against each member's name token. On miss, an error pointing at the access site.

### Parsing a member access

`postfix` gets a new branch. Until now it parsed `primary ("[" expr "]")*` — a primary expression followed by zero or more array index expressions. The new grammar is `primary ("[" expr "]" | "." ident)*`, with `.` parsed by calling a small helper:

```c
static Node *struct_ref(Node *lhs, Token *tok) {
  add_type(lhs);
  if (lhs->ty->kind != TY_STRUCT)
    error_tok(lhs->tok, "not a struct");

  Node *node = new_unary(ND_MEMBER, lhs, tok);
  node->member = get_struct_member(lhs->ty, tok);
  return node;
}
```

`struct_ref` takes the parsed lhs (whatever `s` was in `s.x`) and the member-name token, runs `add_type` on the lhs to make sure its type is computed, checks that it's actually a struct, then wraps the lhs in a new `ND_MEMBER` node tagged with the resolved `Member *`.

The `Node` struct gains one field to hold that pointer:

```c
// Struct member access
Member *member;
```

So an `ND_MEMBER` node carries its struct expression as the standard `lhs` (because `ND_MEMBER` is constructed via `new_unary`), and the resolved member as a separate `member` field. The token saved on the node is the member name token, useful for error messages downstream.

The `postfix` loop also gets converted from a `while` to a `for (;;)` with `continue` at the bottom of each branch:

```c
for (;;) {
  if (equal(tok, "[")) {
    // x[y] is short for *(x+y)
    Token *start = tok;
    Node *idx = expr(&tok, tok->next);
    tok = skip(tok, "]");
    node = new_unary(ND_DEREF, new_add(node, idx, start), start);
    continue;
  }

  if (equal(tok, ".")) {
    node = struct_ref(node, tok->next);
    tok = tok->next->next;
    continue;
  }

  *rest = tok;
  return node;
}
```

The shape is "loop forever, advancing through whichever postfix operator appears next, until none does and we return." This generalizes — when `->` arrives in §9.5, it'll fit as another `if (equal(tok, "->"))` branch in the same loop without disturbing the rest. Chibicc has been favoring this loop shape whenever a postfix grammar starts gaining alternatives.

### Codegen

Two-line addition in `gen_addr`:

```c
case ND_MEMBER:
  gen_addr(node->lhs);
  println("  add $%d, %%rax", node->member->offset);
  return;
```

To compute the address of `s.x`, compute the address of `s` (the lhs), then add the byte offset of `x`. That's it. `gen_addr` puts the result in `%rax`, which is the convention everywhere in chibicc's codegen.

And one line in `gen_expr`:

```c
case ND_VAR:
case ND_MEMBER:
  gen_addr(node);
  load(node->ty);
  return;
```

`ND_MEMBER` joins `ND_VAR` in the "compute the address, then load the value at that address" path. This works because `ND_MEMBER` is an lvalue — anywhere a variable name appears, a member access can appear instead, and the same load mechanism applies. The compute-address-then-load trick is exactly the same as for `ND_DEREF`, just with a different address-computation path.

The `add_type` rule for `ND_MEMBER` says the member access has the member's type:

```c
case ND_MEMBER:
  node->ty = node->member->ty;
  return;
```

So `s.x` where `x` is `int` has type `int`, `s.p` where `p` is `char *` has type `char *`, and so on.

### `declspec` and `is_typename` learn `struct`

The grammar entry point for type specifiers gains a `struct` branch:

```diff
-// declspec = "char" | "int"
+// declspec = "char" | "int" | struct-decl
 static Type *declspec(Token **rest, Token *tok) {
   if (equal(tok, "char")) {
     *rest = tok->next;
     return ty_char;
   }
 
-  *rest = skip(tok, "int");
-  return ty_int;
+  if (equal(tok, "int")) {
+    *rest = tok->next;
+    return ty_int;
+  }
+
+  if (equal(tok, "struct"))
+    return struct_decl(rest, tok->next);
+
+  error_tok(tok, "typename expected");
 }
```

The diff also restructures the existing `int` case from `*rest = skip(tok, "int")` to a guarded `if`, so all three cases have parallel shape. The fall-through becomes an explicit error instead of an implicit `skip` failure — small idiomatic improvement that comes with the rewrite.

`is_typename` learns `struct` too:

```diff
 static bool is_typename(Token *tok) {
-  return equal(tok, "char") || equal(tok, "int");
+  return equal(tok, "char") || equal(tok, "int") || equal(tok, "struct");
 }
```

This is the helper Chapter 7 §7.2 introduced; it grows by one keyword each time chibicc adds a type-introducing token. After this commit it's three. After §9.6 it'll be four. After Chapter 10 it'll be many more.

And the tokenizer's keyword list gains `struct`:

```c
static char *kw[] = {
  "return", "if", "else", "for", "while", "int", "sizeof", "char",
  "struct",
};
```

One word, one new entry. The tokenizer already handles arbitrary identifiers as `TK_IDENT`; the keyword pass converts ones that match.

### Tests

The new file `test/struct.c` exercises every shape the commit can handle. Twenty-three assertions in declaration order roughly cover: basic member access (`x.a`, `x.b`), char-and-int mixed members, arrays of structs (`x[2].b`), structs of arrays (`x.a[0]`), nested structs (`x.a.b`), and `sizeof` of various member combinations. The `sizeof` tests at the bottom encode the no-alignment-yet layout: `struct {char a; int b;} x; sizeof(x);` is `9`, not `16`. The next commit changes that.

### Where we are

Chibicc has anonymous structs. They can be declared in any place a `declspec` runs, members can be accessed with `.`, and member access is an lvalue (so `x.a = 1` works). The `Member` chain hangs off the `Type`, member offsets are the running sum of preceding sizes (no alignment), and member access codegen is the same address-then-load pattern as variable access. Three pieces of the surrounding language stay missing — tags, alignment, struct assignment — and the next four commits fill them in one by one, with `->` and `union` along the way.

---

## 9.2 — Aligning struct members

> `git checkout 9443e4b8bc587b670f9b448b03842530cd355760` — *Align struct members*

The previous commit's `struct {char a; int b;} x` had `a` at offset 0 and `b` at offset 1, giving the struct a total size of 9 bytes. That isn't how C works. C says each member must be at an offset that's a multiple of its type's *alignment requirement*, the struct's alignment is the largest alignment of any member, and the struct's size is rounded up to a multiple of its own alignment. With `int` alignment 8 (chibicc still uses 8-byte ints at this point), the layout has to be `a` at offset 0, then padding bytes 1 through 7, then `b` at offset 8, with the total size rounded up to 16. After this commit, that's what chibicc produces.

The implementation gains an `align` field on every `Type`, an `align_to` helper exported out of codegen, and a few lines added to `struct_decl` to apply the rules.

### `Type` gains an alignment field

```diff
 struct Type {
   TypeKind kind;
   int size;      // sizeof() value
+  int align;     // alignment
```

Until this commit, alignment was implicit in the type — `int` was always 8-byte-aligned because that's what the System V ABI says, but chibicc never had to write that down because struct layout didn't ask. Now it does, so the answer goes on the `Type` struct beside `size`.

Every constructor for a `Type` has to set `align`. The two static type instances and three constructor functions all get updated:

```c
Type *ty_char = &(Type){TY_CHAR, 1, 1};
Type *ty_int = &(Type){TY_INT, 8, 8};

static Type *new_type(TypeKind kind, int size, int align) {
  Type *ty = calloc(1, sizeof(Type));
  ty->kind = kind;
  ty->size = size;
  ty->align = align;
  return ty;
}
```

`ty_char` is `{kind=TY_CHAR, size=1, align=1}`; `ty_int` is `{kind=TY_INT, size=8, align=8}`. The new factory `new_type` centralizes the three-field pattern, and the existing `pointer_to`, `array_of`, and (later) other constructors are rewritten to call it:

```c
Type *pointer_to(Type *base) {
  Type *ty = new_type(TY_PTR, 8, 8);
  ty->base = base;
  return ty;
}

Type *array_of(Type *base, int len) {
  Type *ty = new_type(TY_ARRAY, base->size * len, base->align);
  ty->base = base;
  ty->array_len = len;
  return ty;
}
```

A pointer is always size 8, align 8 — that's the System V ABI for x86-64. An array's size is `base->size * len`, and crucially its alignment is *the base type's alignment*: `int [10]` has alignment 8 because each `int` element has alignment 8, even though the whole array is 80 bytes long. This is how alignment propagates through array types: when a struct has an `int [10]` member, the struct picks up alignment 8 from the array (which got it from the int).

`func_type` doesn't get an alignment because function types don't appear at runtime in any way that asks the question — `TY_FUNC` is a parse-time-only kind, and `add_type`'s consumers won't try to compute the size or alignment of one.

### `align_to` becomes shared

The `align_to` helper has been in `codegen.c` since Chapter 5, used to round the function's total stack size up to a multiple of 16 (the System V ABI's stack-alignment requirement for function calls). It's the same arithmetic the parser now needs, so it gets exported:

```diff
-static int align_to(int n, int align) {
+int align_to(int n, int align) {
   return (n + align - 1) / align * align;
 }
```

The `static` comes off and a declaration is added to `chibicc.h`:

```c
int align_to(int n, int align);
```

The arithmetic is the standard branchless round-up: `(n + align - 1) / align * align` rounds `n` up to the next multiple of `align`. For `n=5, align=8`, this is `(5+7)/8*8 = 12/8*8 = 1*8 = 8`. For `n=11, align=8`, it's `(11+7)/8*8 = 18/8*8 = 2*8 = 16`. The function comment in the source spells out exactly those two examples, which is a nice touch — a reader who isn't sure they remember what the formula does can verify against the commented examples.

### `struct_decl` applies the rules

The offset-assignment loop in `struct_decl` grows three additions:

```diff
   // Assign offsets within the struct to members.
+  ty->align = 1;
   int offset = 0;
   for (Member *mem = ty->members; mem; mem = mem->next) {
+    offset = align_to(offset, mem->ty->align);
     mem->offset = offset;
     offset += mem->ty->size;
+
+    if (ty->align < mem->ty->align)
+      ty->align = mem->ty->align;
   }
-  ty->size = offset;
+  ty->size = align_to(offset, ty->align);
```

Three changes encoding the three rules. The struct's alignment starts at 1 — the alignment of an empty struct, also the lower bound for any nonempty one. Before stamping each member's offset, the running offset is rounded up to that member's alignment, so a member of alignment 8 lands at the next multiple of 8. While walking, the struct's own alignment is updated to the maximum of any member's alignment. After the loop, the struct's size is the running offset rounded up to the struct's own alignment.

For `struct {char a; int b;}`:

- `align` starts at 1.
- Member `a`: alignment 1, offset rounded up to 0 (no change), `a->offset = 0`, offset becomes 1, `align` stays at 1.
- Member `b`: alignment 8, offset rounded up to 8, `b->offset = 8`, offset becomes 16, `align` becomes 8.
- `size = align_to(16, 8) = 16`.

For `struct {int a; char b;}`:

- `align` starts at 1.
- Member `a`: alignment 8, offset rounded up to 0, `a->offset = 0`, offset becomes 8, `align` becomes 8.
- Member `b`: alignment 1, offset rounded up to 8 (no change), `b->offset = 8`, offset becomes 9, `align` stays at 8.
- `size = align_to(9, 8) = 16`.

The last step is what produces *trailing padding* — the seven bytes after `b` that bring the struct's size up to a multiple of its own alignment. Trailing padding matters for arrays of the struct: an `array_of(struct {int a; char b;}, 2)` needs each element to be at an offset that's a multiple of the struct's alignment, which only works if the struct's size is itself a multiple of that alignment. The single `align_to(offset, ty->align)` at the end does the work, and the assertion `ASSERT(16, ({ struct {int a; char b;} x; sizeof(x); }));` in the test file confirms it.

### One codegen consequence

`codegen.c` gets exactly one functional change beyond the `static` strip:

```diff
-  if (ty->kind == TY_ARRAY) {
+  if (ty->kind == TY_ARRAY) {
```

There isn't actually a codegen change — the diff hunk is the export of `align_to` and nothing else. Member loads work correctly already because `gen_addr` for `ND_MEMBER` adds `mem->offset` to the struct's address, and now those offsets are properly aligned. The byte-offset arithmetic in the assembly doesn't care whether the offsets are aligned or not, but the resulting addresses are now suitable for `mov`s of any width.

### Where we are

Every `Type` has an `align` field. Struct layout follows the C rules: each member at the next address satisfying its alignment, the struct itself aligned to the maximum of its members, the struct's size rounded up. Chibicc still has 8-byte `int`, so a struct with mixed `char` and `int` has the same large layout as one of all-`int`s, but the alignment infrastructure is now in place. When `int` becomes 32-bit in Chapter 10, the same code will produce the four-byte alignment without further modification.

---

## 9.3 — Aligning local variables

> `git checkout dfec1157b41bb86c8cb66eee0b0cbdb9dcccb6f4` — *Align local variables*

A four-line diff. With struct types now carrying nontrivial alignment, the stack slots that hold local struct variables have to be aligned too — otherwise a struct on the stack might land on an odd byte and any operation that expected its alignment would be wrong.

The change is in `assign_lvar_offsets`:

```diff
   int offset = 0;
   for (Obj *var = fn->locals; var; var = var->next) {
     offset += var->ty->size;
+    offset = align_to(offset, var->ty->align);
     var->offset = -offset;
   }
   fn->stack_size = align_to(offset, 16);
```

One line. After accumulating the variable's size into the running offset, round the offset up to the variable's alignment. Because chibicc allocates locals downward from `%rbp` (the offsets are negated when assigned as `var->offset`), this means each local's stack slot is at the rounded-up position. The function's total `stack_size` is then rounded up to 16 as before — the System V ABI's stack-alignment requirement for the prologue's `sub $N, %rsp`.

A subtle point: the rounding happens *after* adding the size, not before. So the layout is "bump the offset by the variable's size, then align the result." This means the variable is placed at `-offset` *after* alignment, which is below where it would have been without alignment. For a struct of size 16 and alignment 8 following an `int` of size 8 and alignment 8:

- Start: `offset = 0`.
- `int x`: `offset += 8 = 8`, `align_to(8, 8) = 8`, `x->offset = -8`.
- `struct {int a; int b;} s`: `offset += 16 = 24`, `align_to(24, 8) = 24`, `s->offset = -24`.

`s` lives in the range `-24` to `-9` from `%rbp`, and `s.a` (at member offset 0) is at `-24`, `s.b` (at member offset 8) is at `-16`. Both are 8-byte-aligned addresses, which is what the struct's alignment requires.

The new tests in `test/variable.c` confirm the layout:

```c
ASSERT(15, ({ int x; int y; char z; char *a=&y; char *b=&z; b-a; }));
ASSERT(1, ({ int x; char y; int z; char *a=&y; char *b=&z; b-a; }));
```

The first test has three locals: `int x`, `int y`, `char z`. After this commit their stack slots are 8-byte aligned where their type requires it. `&y` and `&z` are computed as `%rbp` plus their respective offsets, and the difference `b - a` reflects the actual byte-distance between them. The expected value `15` decodes the layout: `y` is 8 bytes after `x`, `z` is allocated at `align_to(8+8+1, 1)` from the start, but stack growth is downward, so the difference comes out positive. The exact arithmetic depends on how `assign_lvar_offsets` assigns negatives — the test value is what chibicc actually produces, which is what makes it a regression test for the alignment behavior.

The second test does the same with the order rearranged: `int x; char y; int z`. The expected difference between `&z` and `&y` is `1`, because `z` follows `y` immediately in stack-growth-order with no padding for the char — but the *next* `int` would have to round up. The test isn't checking the padding in front of `z`; it's checking that the stack offsets are computed by the new size-then-align rule rather than the old size-only rule. The values `15` and `1` are just witnesses that the layout matches the rule.

This commit is a small thing that the previous commit makes necessary. Without member alignment, all stack offsets were already multiples of 8 because every type was a multiple of 8 in size. With the new mixed sizes (char being 1, char arrays being non-multiples of 8), the stack would have unaligned slots if `assign_lvar_offsets` didn't follow up. One line of code closes the loop.

### Where we are

Stack slots for local variables respect each variable's type alignment. The combination of §9.2 (member alignment within a struct) and §9.3 (alignment of the struct's stack slot) means a struct local sits at an aligned address and each of its members sits at an aligned offset from there. The alignment story is complete for the types chibicc currently supports.

---

## 9.4 — Struct tags

> `git checkout e1e831ea3ee46ed7d4c975822f418d60d3050e1b` — *Support struct tags*

Until this commit, every struct type is anonymous and every use is a fresh declaration. There's no way to write

```c
struct Point { int x, y; };
struct Point p;
```

because the second `struct Point` has no way to find the first. This commit adds C's struct *tag* mechanism — a name attached to a struct type at definition time, looked up by name at use time. The implementation introduces a second namespace alongside the existing variable namespace, both tracked per scope.

### A second namespace per scope

```c
// Scope for struct tags
typedef struct TagScope TagScope;
struct TagScope {
  TagScope *next;
  char *name;
  Type *ty;
};

// Represents a block scope.
typedef struct Scope Scope;
struct Scope {
  Scope *next;

  // C has two block scopes; one is for variables and the other is
  // for struct tags.
  VarScope *vars;
  TagScope *tags;
};
```

`TagScope` mirrors `VarScope` exactly: a name, a thing the name refers to (here a `Type *`, where `VarScope` stored an `Obj *`), and a `next` pointer for the linked list. The `Scope` struct gains a `tags` field beside `vars`. The two namespaces share the same scope chain — there's still one `scope` cursor, one `enter_scope`/`leave_scope` pair, one nested-list shape — but each `Scope` now holds two parallel lists, one for ordinary identifiers and one for tags.

This matches what the C standard says: the language has multiple namespaces, and ordinary identifiers (variable names, function names, typedef names) are in one, while struct/union/enum tags are in another. The same name can be both: `int Point;` and `struct Point { int x; };` can coexist without conflict, because the variable-namespace lookup and the tag-namespace lookup walk different lists. The new tests file demonstrates this:

```c
ASSERT(3, ({ struct t {int x;}; int t=1; struct t y; y.x=2; t+y.x; }));
```

`struct t` is the tag, `int t = 1` is the variable. They share the name `t`, but the variable lookup finds the int and the tag lookup finds the struct. `t + y.x` is `1 + 2 = 3`.

### Lookup and registration helpers

The lookup mirrors `find_var`:

```c
static Type *find_tag(Token *tok) {
  for (Scope *sc = scope; sc; sc = sc->next)
    for (TagScope *sc2 = sc->tags; sc2; sc2 = sc2->next)
      if (equal(tok, sc2->name))
        return sc2->ty;
  return NULL;
}
```

Same nested-list walk as variable lookup, just over `tags` instead of `vars`. Innermost scope first, first match wins, returns NULL on miss.

Registration mirrors `push_scope`:

```c
static void push_tag_scope(Token *tok, Type *ty) {
  TagScope *sc = calloc(1, sizeof(TagScope));
  sc->name = strndup(tok->loc, tok->len);
  sc->ty = ty;
  sc->next = scope->tags;
  scope->tags = sc;
}
```

Allocate a `TagScope`, copy the name out of the source buffer with `strndup` (the token's `loc` points into the source, which is fine for the lifetime of the parse but the tag has to outlive the token's view), prepend to the current scope's `tags` list. Same shape, different field.

### `struct_decl` learns the four cases

The grammar changes from `struct-decl = "{" struct-members` to `struct-decl = ident? "{" struct-members`:

```c
// struct-decl = ident? "{" struct-members
static Type *struct_decl(Token **rest, Token *tok) {
  // Read a struct tag.
  Token *tag = NULL;
  if (tok->kind == TK_IDENT) {
    tag = tok;
    tok = tok->next;
  }

  if (tag && !equal(tok, "{")) {
    Type *ty = find_tag(tag);
    if (!ty)
      error_tok(tag, "unknown struct type");
    *rest = tok;
    return ty;
  }

  // Construct a struct object.
  Type *ty = calloc(1, sizeof(Type));
  ty->kind = TY_STRUCT;
  struct_members(rest, tok->next, ty);
  ty->align = 1;
  ...
  // Register the struct type if a name was given.
  if (tag)
    push_tag_scope(tag, ty);
  return ty;
}
```

The function now handles four shapes:

1. `struct {int a;}` — no tag, with members. Anonymous struct; same as before.
2. `struct Point {int x, y;}` — tag and members. Define-and-name; constructs the type, then registers it.
3. `struct Point` — tag only, no members. Reference an existing tag; calls `find_tag`.
4. `struct {int a;}` followed by nothing useful — the prior anonymous form, unchanged.

The decision tree: if there's an identifier, capture it as `tag`. If a tag is present and *isn't* followed by `{`, this is a reference — look up the tag, error if not found, return the existing type. Otherwise (tag followed by `{`, or no tag at all), fall through to the existing parse-the-members path. At the end, if a tag was given, register the new type.

Notice that the lookup-or-construct decision happens before parsing the body. The error message for a missing tag (`"unknown struct type"`) points at the tag token, which is the natural place a reader would look. And the registration step happens *after* the type is fully constructed — including offsets and (after §9.2) alignment — so the registered type is immediately usable.

### Block scoping for tags

Because tags live in the same `Scope` chain as variables, block scope works for them automatically:

```c
ASSERT(2, ({ struct t {char a[2];}; { struct t {char a[4];}; } struct t y; sizeof(y); }));
```

The outer scope declares `struct t` with `char a[2]` (size 2). A nested block declares its own `struct t` with `char a[4]` (size 4). At the closing brace of the inner block, `leave_scope` pops the inner `Scope`, and with it the inner `tags` list goes too — the outer `struct t` is visible again. The subsequent `struct t y` resolves to the outer one, and `sizeof(y)` is `2`.

This is the §8.1 mechanism cashing in for tags. Block scope was implemented as a chain of `Scope`s; adding tags to that chain is one extra field per scope and one extra inner loop in lookup, and shadowing falls out for free. None of `enter_scope`, `leave_scope`, or the scope-cursor logic had to change.

### Tags-without-bodies declarations

The shape `struct Point;` (declare a tag with no body and don't use it) isn't actually exercised here — the parser would treat it as a `declspec` followed by no declarators, which `declaration` would reject for the wrong reason. But the construct `struct t {int a, b;}; struct t y;` (declare-and-don't-use the tag, then later use it) works because the test file confirms it:

```c
ASSERT(16, ({ struct t {int a; int b;}; struct t y; sizeof(y); }));
```

The first line is `struct t {int a; int b;};` — a `declspec` (resulting in the struct type, registered under tag `t`) followed by a single `;` because `declaration` allows zero declarators. The second line is `struct t y;` — a fresh `declspec` that takes the tag-lookup path, returning the existing type, then `y` as the declarator. The two lines share a type by name.

### One wart, one missing check

The C standard has struct tags and union tags in *separate* namespaces — `struct Foo` and `union Foo` are unrelated names. Chibicc puts them both in the same `tags` list (this becomes apparent in §9.6, when union joins struct). Mixing the two would let a `struct Foo` definition shadow a previous `union Foo`, which is incorrect by the standard. None of the test programs exercise the conflict, so chibicc's relaxation is invisible in practice, but it's a wart for the eventual errata appendix.

The other missing check is the same one that affected variables in §8.1: two `struct t` definitions in the same scope produce two `TagScope` entries, with the second shadowing the first by virtue of being prepended. The standard says the second is a redeclaration error. Chibicc doesn't care.

### Where we are

Struct types can be named with tags. A tag introduced in a scope is visible until that scope closes, can be referenced by `struct <tag>` in later declarations, and can be shadowed by a nested-scope tag of the same name. The mechanism is a parallel `tags` field on `Scope` plus a `find_tag`/`push_tag_scope` pair that mirrors the variable namespace. C's two-namespaces shape is now expressible in chibicc — almost. Struct and union tags share a list (incorrect by the standard), and same-scope redeclarations are silently shadowed. Both are errata-appendix candidates.

---

## 9.5 — The `->` operator

> `git checkout f0a018a7d6f5e3847d7e66e324c5f71a55c8b5ef` — *Add `->` operator*

The smallest commit in the chapter. Three files touched, seventeen net lines added. The `->` operator is C's syntactic sugar for "dereference and then take a member": `p->x` means `(*p).x`. Chibicc implements it by literally rewriting `p->x` into `(*p).x` at parse time — a parse-time desugaring that produces the same AST a hand-written `(*p).x` would.

### Tokenizer side

`->` is two characters, so the punctuator reader has to know about it. Until this commit, multi-character punctuators were a fixed list of comparison operators handled by an `||` chain:

```diff
 static int read_punct(char *p) {
-  if (startswith(p, "==") || startswith(p, "!=") ||
-      startswith(p, "<=") || startswith(p, ">="))
-    return 2;
+  static char *kw[] = {"==", "!=", "<=", ">=", "->"};
+
+  for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
+    if (startswith(p, kw[i]))
+      return strlen(kw[i]);

   return ispunct(*p) ? 1 : 0;
 }
```

The `||` chain becomes a table-driven loop. Adding `->` to the table is one new entry; adding any future multi-character punctuator (`>>`, `<<`, `&&`, `||`, the compound-assignment `+=` family from Chapter 11) would now be a one-line change to the table. The `strlen(kw[i])` return makes the table support different-length punctuators, which the previous form didn't need but the new form does — every entry happens to be length 2 here, but the structure generalizes.

### Parser side

One new branch in the `postfix` loop:

```c
if (equal(tok, "->")) {
  // x->y is short for (*x).y
  node = new_unary(ND_DEREF, node, tok);
  node = struct_ref(node, tok->next);
  tok = tok->next->next;
  continue;
}
```

Three lines of body. Wrap the current node in an `ND_DEREF` (turning `p` into `*p`), then call `struct_ref` to add the member access (turning `*p` into `(*p).y`), then advance past the `->` and the member-name tokens. The result is an AST that's structurally identical to what `(*p).y` would have produced: an `ND_MEMBER` whose `lhs` is an `ND_DEREF` whose `lhs` is `p`.

This is the chapter's **canonicalization-at-parse-time** instance, the sixth across the book so far. The previous five, named together in Chapter 6 §6.1 and then extended once in Chapter 7:

- Chapter 3 §3.4: `>` and `>=` are rewritten as `<` and `<=` with operands swapped.
- Chapter 3 §3.9: `while (e) s` is rewritten as `for (; e; ) s`.
- Chapter 4 §4.3: `p + n` (pointer plus integer) is rewritten as `p + (n * sizeof(*p))`.
- Chapter 6 §6.1: `x[y]` is rewritten as `*(x + y)`.
- Chapter 7 §7.5: `({ ... ; expr })` parses through `compound_stmt` and is tagged `ND_STMT_EXPR` — a *delegation* variant rather than a strict desugaring (the AST shape is novel, but the parser borrows the shape of the form it most resembles).

The `->` instance fits as a desugaring: it rewrites surface form A (`p->y`) into the AST shape that surface form B (`(*p).y`) already produces, with no new node kind. There's no `ND_ARROW` in chibicc; the symbol exists at the token level and disappears in parsing. So the breakdown across six instances is five strict desugarings and one delegation.

The pattern is by now firmly part of chibicc's vocabulary. Whenever the surface syntax says one thing that's equivalent to another already-supported shape, chibicc chooses parsing over codegen — the new construct gets a few lines in the parser that build the existing AST shape, and codegen, type checking, and `add_type` all stay unchanged. The next likely place this shows up is Chapter 11's `+=` family, which lowers to something like `tmp = &lhs, *tmp = *tmp + rhs` using the generalized-lvalue comma extension from Chapter 8 §8.5.

### Tests

Two tests in `test/struct.c`:

```c
ASSERT(3, ({ struct t {char a;} x; struct t *y = &x; x.a=3; y->a; }));
ASSERT(3, ({ struct t {char a;} x; struct t *y = &x; y->a=3; x.a; }));
```

The first reads through the pointer: `x.a` is set to 3, then `y->a` (which is `(*y).a`, which is `(*&x).a`, which is `x.a`) reads back 3. The second writes through the pointer: `y->a = 3` writes to `(*y).a` which is `x.a`, and the subsequent `x.a` reads back 3. Both confirm that `->` is indistinguishable from `(*p).` in observable behavior, which is what the parser-level desugaring guarantees.

### Where we are

`->` exists as a punctuator and as a postfix operator. The implementation is sixteen lines of new code (one entry in the punctuator table, one branch in the postfix loop) and zero lines of new codegen. The desugaring pattern keeps the AST surface area smaller — chibicc carries one member-access node kind, not two — and the codegen for `*` and `.` does double duty without changes.

---

## 9.6 — `union`

> `git checkout 11e3841832697c8ba4a1d68f5daa05045f70a716` — *Add union*

C's `union` is a struct's distorted twin: same syntax for declaring the body, same `.` and `->` for member access, but every member starts at offset 0 and the size is the maximum member size, not the sum. Adding it to chibicc is a refactor exercise — most of what the struct parser does is also what the union parser needs, so the commit factors out the shared logic into a `struct_union_decl` helper and adds a thin `union_decl` alongside the existing `struct_decl`.

### A new keyword and a new type kind

```diff
   TY_STRUCT,
+  TY_UNION,
 } TypeKind;
```

```c
"struct", "union",
```

`TY_UNION` joins the type enum, `union` joins the keyword list. Two one-line changes.

`is_typename` learns `union`:

```diff
 static bool is_typename(Token *tok) {
-  return equal(tok, "char") || equal(tok, "int") || equal(tok, "struct");
+  return equal(tok, "char") || equal(tok, "int") || equal(tok, "struct") ||
+         equal(tok, "union");
 }
```

And `declspec` gets a fourth branch:

```diff
   if (equal(tok, "struct"))
     return struct_decl(rest, tok->next);
 
+  if (equal(tok, "union"))
+    return union_decl(rest, tok->next);
+
   error_tok(tok, "typename expected");
```

### Refactoring `struct_decl`

The previous `struct_decl` did three things: parse the optional tag and members (or look up the tag if no body was given), assign offsets and compute alignment for the resulting type, and register the tag if one was given. The first and third are common to struct and union; the second is type-specific. The commit factors out the shared parts:

```c
// struct-union-decl = ident? ("{" struct-members)?
static Type *struct_union_decl(Token **rest, Token *tok) {
  // Read a tag.
  Token *tag = NULL;
  if (tok->kind == TK_IDENT) {
    tag = tok;
    tok = tok->next;
  }

  if (tag && !equal(tok, "{")) {
    Type *ty = find_tag(tag);
    if (!ty)
      error_tok(tag, "unknown struct type");
    *rest = tok;
    return ty;
  }

  // Construct a struct object.
  Type *ty = calloc(1, sizeof(Type));
  ty->kind = TY_STRUCT;
  struct_members(rest, tok->next, ty);
  ty->align = 1;

  // Register the struct type if a name was given.
  if (tag)
    push_tag_scope(tag, ty);
  return ty;
}
```

`struct_union_decl` returns a type that's been parsed and tag-registered, with `kind = TY_STRUCT` (provisionally) and members in place — but no offsets or sizes computed. The two specific decl functions take that result and finish it off:

```c
// struct-decl = struct-union-decl
static Type *struct_decl(Token **rest, Token *tok) {
  Type *ty = struct_union_decl(rest, tok);
  ty->kind = TY_STRUCT;

  // Assign offsets within the struct to members.
  int offset = 0;
  for (Member *mem = ty->members; mem; mem = mem->next) {
    offset = align_to(offset, mem->ty->align);
    mem->offset = offset;
    offset += mem->ty->size;

    if (ty->align < mem->ty->align)
      ty->align = mem->ty->align;
  }
  ty->size = align_to(offset, ty->align);
  return ty;
}

// union-decl = struct-union-decl
static Type *union_decl(Token **rest, Token *tok) {
  Type *ty = struct_union_decl(rest, tok);
  ty->kind = TY_UNION;

  // If union, we don't have to assign offsets because they
  // are already initialized to zero. We need to compute the
  // alignment and the size though.
  for (Member *mem = ty->members; mem; mem = mem->next) {
    if (ty->align < mem->ty->align)
      ty->align = mem->ty->align;
    if (ty->size < mem->ty->size)
      ty->size = mem->ty->size;
  }
  ty->size = align_to(ty->size, ty->align);
  return ty;
}
```

`struct_decl` is what was already there, minus the parts now in the helper. `union_decl` is the parallel for unions: walk the members, take the max alignment and the max size, round the size up to the alignment. The `kind = TY_UNION` overwrite at the top is what re-tags the type after `struct_union_decl` returned it as `TY_STRUCT`. (The over-and-back is a small wart — `struct_union_decl` could take a kind parameter, or could just default to leaving it unset for the caller to fill in. Rui's choice is the path-of-least-keystrokes.)

The union's per-member work is two `if`s. Members don't get `offset` assignments because they're all zero already from the `calloc` in `struct_members`'s `Member` allocation. Alignment and size each track the max. The trailing `align_to(ty->size, ty->align)` rounds the size up to the alignment, just as for structs — a `union {int a; char b[6];}` has size 6 from `b` but rounds up to 8 from `a`'s alignment.

The test cases:

```c
ASSERT(8, ({ union { int a; char b[6]; } x; sizeof(x); }));
```

Members are `int a` (size 8, align 8) and `char b[6]` (size 6, align 1). Max size 8, max align 8, `align_to(8, 8) = 8`. Result: 8. The alignment rounding doesn't actually change anything here; the int's alignment is what dominates.

```c
ASSERT(3, ({ union { int a; char b[4]; } x; x.a = 515; x.b[0]; }));
ASSERT(2, ({ union { int a; char b[4]; } x; x.a = 515; x.b[1]; }));
ASSERT(0, ({ union { int a; char b[4]; } x; x.a = 515; x.b[2]; }));
ASSERT(0, ({ union { int a; char b[4]; } x; x.a = 515; x.b[3]; }));
```

The member-aliasing tests. `x.a = 515` writes the int 515 to offset 0 of the union. `515` in hex is `0x203`, which in little-endian (x86-64) byte order is `03 02 00 00 00 00 00 00`. Reading the bytes through `x.b` recovers them: `b[0] = 3`, `b[1] = 2`, `b[2] = 0`, `b[3] = 0`. The first four bytes are the active range; the last four (which `b` doesn't reach because it's only 4 bytes long) would also be zero. This is the canonical "type-pun via union" pattern, and chibicc gets it right because both members start at offset 0 and `gen_addr` for `ND_MEMBER` adds that offset (zero) to the union's address.

### `struct_ref` accepts unions

The lvalue-or-error check in `struct_ref` widens:

```diff
 static Node *struct_ref(Node *lhs, Token *tok) {
   add_type(lhs);
-  if (lhs->ty->kind != TY_STRUCT)
-    error_tok(lhs->tok, "not a struct");
+  if (lhs->ty->kind != TY_STRUCT && lhs->ty->kind != TY_UNION)
+    error_tok(lhs->tok, "not a struct nor a union");
```

The `&&` chain admits union types. Otherwise unchanged: union member access works through the same `ND_MEMBER` node, the same `gen_addr` path (add the member's offset, which for a union is always zero), the same `gen_expr` load. The `.` operator works for unions because `postfix` doesn't care about the lhs's type until `struct_ref` is called — and `struct_ref` now allows it. Same for `->`, which calls `struct_ref` after wrapping in `ND_DEREF`.

### One namespace, not two

Tag-scope holds both struct and union tags. The `TagScope` comment changes:

```diff
-// Scope for struct tags
+// Scope for struct or union tags
```

But the data structure is the same `tags` chain on each `Scope`. C99 has *separate* namespaces for struct tags and union tags — `struct Foo` and `union Foo` are unrelated names — and chibicc puts them both in the same list. This is the wart §9.4 promised. None of the test programs exercise the conflict, so it's invisible in practice, but it'll go in the errata appendix. The fix would be a parallel `unions` field on `Scope` or a `kind` field on `TagScope`; the cost is small and the benefit (correctness) is real, but Rui doesn't take it.

### Where we are

`union` works. The shape is "struct minus offsets," which factors cleanly: `struct_union_decl` does the shared parsing, `struct_decl` and `union_decl` differ only in offset/size logic. Member access works identically because `gen_addr` for `ND_MEMBER` always adds an offset (zero or otherwise) to the aggregate's address. The first compound type with non-trivial layout overlap is now in chibicc's vocabulary, and the parser refactor leaves the door open for whatever the next aggregate kind would be (`enum`, in Chapter 10's neighborhood).

---

## 9.7 — Struct assignment

> `git checkout bef05432c9d3289636ed1d360ca9b863a0698dc7` — *Add struct assignment*

The last gap. Until this commit, `s1 = s2` (where both are structs) doesn't work. The codegen's `gen_expr` for `ND_ASSIGN` calls `gen_addr(lhs)` to push the destination address, then `gen_expr(rhs)` to put the value in `%rax`, then `store(ty)` to write the value. But for a struct, "the value in `%rax`" is meaningless — a struct doesn't fit in one register. Chibicc has been silently producing nonsense for struct-to-struct assignment, which the test suite hadn't exercised because no existing test wrote `s1 = s2`.

This commit fixes it by changing two things in `codegen.c`. First, `load` is taught to short-circuit on struct and union types, just as it already did for arrays. Second, `store` is taught to handle struct and union types with a byte-by-byte copy loop instead of a single `mov`. Together these implement struct assignment as "copy `sizeof(struct)` bytes from the source address to the destination address."

### `load` skips struct and union

```diff
 static void load(Type *ty) {
-  if (ty->kind == TY_ARRAY) {
+  if (ty->kind == TY_ARRAY || ty->kind == TY_STRUCT || ty->kind == TY_UNION) {
     // If it is an array, do not attempt to load a value to the
     // register because in general we can't load an entire array to a
     // register. As a result, the result of an evaluation of an array
     // becomes not the array itself but the address of the array.
     // This is where "array is automatically converted to a pointer to
     // the first element of the array in C" occurs.
     return;
   }
```

For arrays, this short-circuit was the encoding of "an array decays to a pointer to its first element" — when chibicc evaluates an array-typed expression, the result in `%rax` is the array's address, not its contents. Structs and unions get the same treatment, but for a different reason: they don't decay to anything, but they don't fit in a register either. Leaving the address in `%rax` instead of trying to load is the only sensible behavior.

The consequence: `gen_expr` of a struct-typed `ND_VAR` or `ND_MEMBER` produces the struct's address, not its bytes. Anything downstream that expected to consume "the value" has to know to read from that address instead. For struct assignment, that's exactly what `store` does.

### `store` does a byte loop for struct and union

```diff
 static void store(Type *ty) {
   pop("%rdi");
 
+  if (ty->kind == TY_STRUCT || ty->kind == TY_UNION) {
+    for (int i = 0; i < ty->size; i++) {
+      println("  mov %d(%%rax), %%r8b", i);
+      println("  mov %%r8b, %d(%%rdi)", i);
+    }
+    return;
+  }
+
   if (ty->size == 1)
     println("  mov %%al, (%%rdi)");
   else
     println("  mov %%rax, (%%rdi)");
 }
```

`store`'s contract is "write the value in `%rax` to the address on the stack." The pop pulls the destination address into `%rdi`. For an integer or pointer, the existing one-instruction `mov` writes the register's contents to the address. For a struct, the contents-of-`%rax` is a *source address* (because `load` short-circuited), so the new branch loops over the struct's bytes and copies each one from `(%rax + i)` to `(%rdi + i)`.

The instruction pair is `mov %d(%%rax), %%r8b` (read one byte from offset i of the source) and `mov %%r8b, %d(%%rdi)` (write that byte to offset i of the destination). `%r8b` is the low 8 bits of the `%r8` register — chibicc uses it as a scratch byte register because it's not part of the SysV calling convention's first-six argument registers (none of which are in use during a `store`) and because the assembler accepts it as the byte form. Each iteration emits two assembly instructions, so the assembly grows by `2 * size` lines per struct assignment. For a small struct of 16 bytes, that's 32 lines of `mov`s — verbose, but correct.

A real compiler would do this differently: `rep movsb` is one x86 instruction that copies `%rcx` bytes from `%rsi` to `%rdi`, and a vector load/store pair (`movdqa`) handles 16 bytes at a time on aligned addresses. Both would shrink the assembly and the runtime cost. Chibicc takes the simple path. The byte-by-byte loop has the merit of being obviously correct and producing the same result for any struct size or alignment, with no special cases. For the test programs chibicc cares about (small structs in throwaway test functions), the performance doesn't matter.

### How the pieces fit at `ND_ASSIGN`

The codegen for `ND_ASSIGN` itself is unchanged:

```c
case ND_ASSIGN:
  gen_addr(node->lhs);
  push();
  gen_expr(node->rhs);
  store(node->ty);
  return;
```

For `s1 = s2` with both as structs:

1. `gen_addr(s1)` puts `&s1` in `%rax`.
2. `push()` pushes `%rax` onto the stack.
3. `gen_expr(s2)` runs. `s2` is `ND_VAR`, so `gen_addr(s2)` puts `&s2` in `%rax`, then `load` is called — but `load` sees `TY_STRUCT` and returns without emitting anything. `%rax` still holds `&s2`.
4. `store(struct_type)` pops the destination from the stack into `%rdi`. Now `%rax = &s2`, `%rdi = &s1`. `store` sees `TY_STRUCT` and emits the byte-by-byte copy loop, transferring `sizeof(s1)` bytes from `(%rax)` to `(%rdi)`.

Three pieces had to align: `gen_addr` already worked for struct lvalues (via the existing `ND_VAR` and `ND_MEMBER` paths); `load` had to be taught to leave the address in `%rax` for struct types; `store` had to be taught to interpret `%rax` as a source address rather than a value when the type is struct or union. The two changes in this commit are exactly the second and third pieces.

The same path works for `*p = s` (where `p` is a struct pointer): `gen_addr` of `ND_DEREF` calls `gen_expr` of the pointer and leaves its value in `%rax`, which is `&*p` — the destination address. The push/expr/store sequence proceeds identically, and the byte-copy loop runs at the right destination. The new tests exercise this:

```c
ASSERT(7, ({ struct t {int a,b;}; struct t x; x.a=7; struct t y; struct t *z=&y; *z=x; y.a; }));
```

`*z = x` is a struct assignment via a pointer dereference. The destination `gen_addr` resolves to `gen_expr(z)` (which loads `&y` into `%rax`, then push); the source resolves to `gen_addr(x)` (which loads `&x` into `%rax`); `store` does the byte copy. After the assignment, `y.a` is `7` because the bytes of `x` (which had `a = 7`) are now in `y`.

And the same path works for unions:

```c
ASSERT(3, ({ union {int a,b;} x,y; x.a=3; y.a=5; y=x; y.a; }));
```

`y = x` for unions does the same byte-copy. Because `load` and `store` both check for `TY_UNION` alongside `TY_STRUCT`, the union case rides on the same code paths. The result is `3`, the value of `x.a` copied byte-for-byte into `y`.

### Where we are

Struct and union assignment work. The implementation is two case-additions and one byte-loop, total ten lines of diff. The combined effect of `load` short-circuiting and `store` byte-copying is that every struct-typed assignment is a memcpy, regardless of where the source and destination came from — local-to-local, member-to-member, dereferenced-pointer-to-local. The simplicity of the approach (no rep-movsb, no vector loads, no alignment checks) keeps the codegen path uniform at the cost of verbose output. For chibicc, that's the right trade.

---

## Recap

| Commit | What it added |
|---|---|
| `f814033` | The `struct` keyword and the `TY_STRUCT` type kind; the `Member` struct with `name`, `ty`, `offset`, `next`; `struct_decl` and `struct_members` parsers; `get_struct_member` linear-scan lookup; `struct_ref` building `ND_MEMBER` nodes; `postfix` learns the `.` operator; `gen_addr` and `gen_expr` for `ND_MEMBER`; `add_type` rule (member's type); `is_typename` learns `struct` |
| `9443e4b` | The `align` field on `Type`; `align_to` exported from codegen; `new_type` factory centralizing `kind`/`size`/`align` initialization; `pointer_to` and `array_of` rewritten through `new_type`; `struct_decl` enforces member alignment, struct-self alignment, and trailing-padding rules |
| `dfec115` | `assign_lvar_offsets` rounds each local's stack offset up to the local's alignment; one-line addition matching the now-meaningful alignment story |
| `e1e831e` | `TagScope` struct paralleling `VarScope`; `tags` field on `Scope` paralleling `vars`; `find_tag` and `push_tag_scope` paralleling the variable functions; `struct_decl` learns optional tag, define-and-name, reference-only, and reference-error paths; struct tag namespace parallel to ordinary identifiers, sharing the same scope chain |
| `f0a018a` | The `->` punctuator (table-driven `read_punct` rewrite); `postfix` branch desugaring `p->y` into `(*p).y` at parse time; sixth canonicalization-at-parse-time instance |
| `11e3841` | The `union` keyword and the `TY_UNION` type kind; `struct_union_decl` factored out of `struct_decl`; new `union_decl` computing max-alignment and max-size with offsets all zero; `is_typename` learns `union`; `struct_ref` widens to accept union lhs; `TagScope` comment expanded (struct and union tags share the namespace) |
| `bef0543` | `load` short-circuits for `TY_STRUCT` and `TY_UNION`, leaving the source address in `%rax`; `store` byte-loops over `ty->size` for `TY_STRUCT` and `TY_UNION`, copying each byte from `(%rax)` to `(%rdi)` via `%r8b`; struct-to-struct, member-to-member, and pointer-deref-to-local assignment all work |

Seven commits — the largest bundle so far. Five of them (the `struct` introduction, member alignment, struct tags, `union`, struct assignment) are substantive; two (local alignment, `->`) are small. The chapter's center of gravity is in §9.1 (the most code, most concepts), §9.4 (the namespace work), and §9.7 (the byte-copy that breaks the everything-fits-in-rax assumption). The other four sections each lift one specific limitation that would have been awkward to leave in.

Several threads from earlier chapters meet here. The Chapter 6 declarator machinery is reused in `struct_members` without modification — the same function that parses `int *a[3]` as a function parameter parses it as a struct member. The Chapter 8 §8.1 scope chain is reused for tags by adding one extra field per scope, with `enter_scope`, `leave_scope`, and the lookup loop shape all unchanged. The canonicalization-at-parse-time pattern named across Chapters 6–7 picks up its sixth instance with `->`, and the pre-factor-before-feature pattern doesn't gain a new instance but the `struct_union_decl` refactor in §9.6 *does* — Rui factors the shared decl logic out the same commit that introduces unions, rather than splitting into a pre-factor and a feature commit, but the impulse is the same.

Two errata candidates accumulated. Struct and union tags share a namespace where C99 requires them to be separate (§9.4, §9.6). And the redeclaration-in-same-scope check missing since Chapter 8 §8.1 is missing here too — declaring `struct t {...}` twice in the same block produces two `TagScope` entries, with the second silently winning. Both are worth a paragraph in the eventual errata appendix.

The compiler now has compound types, with member access by `.` and `->`, named struct and union tags, proper member and stack-slot alignment, and full-size struct/union assignment by byte copy. The next chapter (Filling out the type system, the largest commit count of the book) revisits the type infrastructure built across Chapters 1–9 and brings it closer to C's actual type story: `int` shrinks to 32 bits, `short` and `long` arrive, type checking grows usual-arithmetic-conversions logic, and the parser learns enough of the C declarator grammar to handle nested cases that have been quietly unhandled until now.
