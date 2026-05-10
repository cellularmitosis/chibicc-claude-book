# Chapter 22 — Performance, dependency files, and the linker driver

> Commits covered: `f0c98e0`, `0aad326`, `30520e5`, `655954e`, `f694413`, `d0c4667`, `95d5a46`, `57c1d4e`, `db850f3`, `fb5cfe5`, `7aa72e4`, `c3edffb`, `86785fc`, `c0f0614`, `d48d9e5`, `a6c6622`, `f10bceb`, `1e9b6dd`, `4e5de36`, `c8df787`, `d1bc9a4`, `469f159`, `fb49370`. Twenty-three commits — the longest-running chapter so far. The arc is mixed: one small parser tweak that lets labels-as-values be used in static initializers, the introduction of a from-scratch open-addressing hashmap (and three places that immediately start using it), the seven-commit `-M` family that lets chibicc emit Makefile-shaped dependency rules, `-fpic`/`-fPIC` (which actually changes codegen — this chapter's biggest surprise), file-search caching that uses the new hashmap, three include-handling improvements (include-guard optimization, `#pragma once`, `#include_next`), five linker-driver pass-throughs (`-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker`), and a small harness of shell scripts that build real-world third-party programs with chibicc.

Two threads run through the chapter. One is performance: the macro table, the block-scope variable lookup, the keyword check, and the include-search path were all linear scans, and at the codebase's current size they're starting to show up in profiles. Rui's answer is to write a single hashmap and use it everywhere a linear scan was hiding. The hashmap is the most reusable piece of data structure code in the entire compiler.

The other thread is real-world buildability. Chibicc needs to produce dependency files so `make` can know which headers a `.c` file pulls in. It needs `-static`, `-shared`, `-L`, `-Wl,`, and `-Xlinker` so build systems can drive it the way they drive `gcc`. It needs include-guard optimization so the preprocessor doesn't tokenize the same header dozens of times. And it needs `-fpic`/`-fPIC` so shared libraries built by chibicc actually link. The chapter closes with the harness: shell scripts that try to build git, libpng, sqlite, and tinycc. By the end of the chapter chibicc can compile real programs that real people use.

Seven sections from twenty-three commits.

- **§22.1** — Labels-as-values as a compile-time constant (commit 284).
- **§22.2** — The string hashmap (commit 285).
- **§22.3** — Three hashmap users: macros, scopes, keywords (commits 286–288).
- **§22.4** — The `-M` family (commits 289–295).
- **§22.5** — `-fpic`/`-fPIC` and the file-search cache (commits 296–297).
- **§22.6** — Include-guard optimization, `#pragma once`, `#include_next` (commits 298–300).
- **§22.7** — Linker-driver pass-throughs and the third-party harness (commits 301–306).

The chapter follows `main` order. Calendar dates scatter widely across August, September, and October 2020 — the `-M` flags were drafted in mid-August but the `-MMD` polish came in mid-September, and several commits land in early October. As before, the prose walks `main` order without commenting on the dates except where they matter for a dependency between commits.

---

## 22.1 — Labels-as-values as a compile-time constant

> `git checkout f0c98e0d590ffae286a8a4847c91212c734be8e3` — *[GNU] Treat labels-as-values as compile-time constant*

Chapter 21 §21.6.10 added `&&label` as an expression usable inside function bodies. What that commit didn't address was the trick that motivates labels-as-values in the first place: storing label addresses in a static array so a piece of code can dispatch by indexing.

```c
static void *jump_table[] = {&&l1, &&l2, &&l3};
goto *jump_table[i];
```

This is the canonical use pattern — interpreters use it to dispatch on opcode, lexers use it to dispatch on character class. For it to work, `&&l1`, `&&l2`, `&&l3` need to be compile-time constants suitable for static initialization, the same way `&global_var` is. The commit teaches `eval2` and `eval_rval` about `ND_LABEL_VAL`.

### A new pointer level on `Relocation`

The first surprise is that `Relocation`'s `label` field grows a level of indirection:

```c
struct Relocation {
  Relocation *next;
  int offset;
-  char *label;
+  char **label;
  long addend;
};
```

And the call site in `emit_data` learns to dereference:

```c
println("  .quad %s%+ld", *rel->label, rel->addend);
```

Why? A label's `unique_label` (the assembler symbol like `.L..3`) is generated lazily — `assign_lvar_offsets` walks each function and stamps a unique label name into `Node->unique_label` only when codegen needs it. At the time `eval2` runs over a global initializer, the function bodies haven't been codegen'd yet, so the label string isn't filled in. By storing a pointer-to-pointer, the `Relocation` captures the *address of the slot* in `Node->unique_label`. By the time `emit_data` reads `*rel->label`, codegen has filled in the slot, and the dereference reads the real string.

The same indirection covers global variables — `Obj->name` is set at parse time, but using `**` uniformly lets the relocation machinery treat both kinds of symbol identically. The two existing `eval2` cases that take label addresses (`ND_VAR` for globals and `ND_ADDR` of a deref-chain) change their stored expression to `&node->var->name`, and the new arm stores `&node->unique_label` for `ND_LABEL_VAL`.

### The `eval2` arm

```c
case ND_LABEL_VAL:
  *label = &node->unique_label;
  return 0;
```

Three lines. The `eval2` machinery already knew how to record "this initializer is a relocation against symbol *X*, with addend *N*." All this commit had to do was teach it that label-value nodes are also valid sources of relocations, and arrange for the symbol slot to be readable at emit time even though the unique-label string isn't filled in yet.

`eval_rval` is updated for symmetry — its `ND_VAR` case also moves from `*label = node->var->name` to `*label = &node->var->name`. The double-pointer convention is now uniform across the eval-quartet.

The `is_const_expr` predicate (the fifth eval-quartet member from Ch 21) doesn't need an `ND_LABEL_VAL` arm; the existing structural recursion already returns true on labels-as-values once `eval2` accepts them.

### The test

`test/control.c` grows three new cases:

```c
ASSERT(3, ({ static void *p[]={&&v41,&&v42,&&v43}; int i=0; goto *p[0];
             v41:i++; v42:i++; v43:i++; i; }));
```

The `static` storage-class is the load-bearing piece. Chapter 21's labels-as-values would have rejected this initializer (it's not a function-local automatic, so its initializer must be a compile-time constant). After this commit, the static array of three label addresses elaborates correctly — each element gets a relocation against the corresponding `.L` label.

**Where we are.** Labels-as-values are now usable in static initializers — the canonical jump-table pattern works. The `Relocation` mechanism now stores a pointer-to-pointer for the symbol name, accommodating labels (whose unique-label strings are generated lazily during codegen) and globals (whose names are set at parse time) through the same channel.

---

## 22.2 — The string hashmap

> `git checkout 0aad326f3550b3d4c499d4078fcc65cc2dbf7626` — *Add string hashmap*

This commit adds a single new file, `hashmap.c`. The header gets a `HashMap` and `HashEntry` type plus six function declarations. There are no callers yet — the hashmap is added in isolation, with a built-in self-test wired up to the driver as `-hashmap-test`. The next three commits convert linear-search structures one at a time.

The data structure is a from-scratch open-addressing hashmap with linear probing, FNV-1a hashing, and tombstone-based deletion. It's about 165 lines including the test. The simplest hashmap a competent C programmer writes, and that's not a coincidence — Rui doesn't reach for a fancier data structure when a plain one will do.

### The two types

```c
typedef struct {
  char *key;
  int keylen;
  void *val;
} HashEntry;

typedef struct {
  HashEntry *buckets;
  int capacity;
  int used;
} HashMap;
```

`HashEntry` carries a key (a `(pointer, length)` pair so non-NUL-terminated tokens can be used directly), a value pointer, and that's it. `HashMap` carries the bucket array, the bucket array size, and a count of used entries.

The decision to store keys as `(pointer, length)` rather than a NUL-terminated `char *` is what makes the hashmap fit chibicc's tokenizer cleanly. Tokens point into the source buffer; a token has a `loc` and `len` rather than a NUL-terminated string. With the `keylen` field, `find_macro` can hand the hashmap `tok->loc` and `tok->len` directly without copying out a temporary. The non-`2` variants of the API treat their input as `strlen`-terminated; the `2`-suffixed variants take an explicit length.

### FNV-1a

```c
static uint64_t fnv_hash(char *s, int len) {
  uint64_t hash = 0xcbf29ce484222325;
  for (int i = 0; i < len; i++) {
    hash *= 0x100000001b3;
    hash ^= (unsigned char)s[i];
  }
  return hash;
}
```

The two magic numbers are FNV's 64-bit offset basis and prime. FNV-1a is byte-at-a-time, branch-free, and produces enough entropy on short strings (variable names, macro names) to keep collision rates low under linear probing. Rui's version has the multiply *before* the XOR, which is the FNV-1a variant; the plain FNV-1 has it the other way. Both work; FNV-1a is the more recommended ordering because it gets entropy onto every output bit before the next byte arrives.

There's no defense against a malicious key set. The hash is deterministic and non-randomized, so a constructed input could in principle pile up keys in a single bucket. Compilers don't get this kind of attack in practice, so it's a non-issue.

### High and low watermarks

```c
#define INIT_SIZE 16
#define HIGH_WATERMARK 70
#define LOW_WATERMARK 50
```

A fresh hashmap has 16 buckets. When the used-count crosses 70% of capacity, `rehash` runs. It picks a new capacity by doubling until the load factor (after the rehash) would be under 50%:

```c
int cap = map->capacity;
while ((nkeys * 100) / cap >= LOW_WATERMARK)
  cap = cap * 2;
```

Note `nkeys` is the count of *live* entries, not `map->used` — tombstones don't count toward the new capacity calculation. The rehash discards tombstones; only the keys with real values get re-inserted. So a map that's accumulated a lot of deletes can shrink its effective load even without growing capacity. A pure no-deletion workload doubles the bucket array each rehash; a delete-heavy workload may rehash to the *same* size, which is enough because the tombstones go away.

### Lookup with linear probing

```c
static HashEntry *get_entry(HashMap *map, char *key, int keylen) {
  if (!map->buckets)
    return NULL;

  uint64_t hash = fnv_hash(key, keylen);

  for (int i = 0; i < map->capacity; i++) {
    HashEntry *ent = &map->buckets[(hash + i) % map->capacity];
    if (match(ent, key, keylen))
      return ent;
    if (ent->key == NULL)
      return NULL;
  }
  unreachable();
}
```

Probe `(hash, hash+1, hash+2, ...)` modulo capacity. Stop on a match (return the entry) or on a NULL bucket (the key isn't in the table). Tombstones are *not* a stopping condition — a probe walks past tombstones because the desired key might be further down the chain, displaced by an entry that was later deleted.

The `unreachable()` at the end fires only if the table is completely full of non-NULL, non-matching entries, which can't happen under the 70% watermark unless something has gone very wrong with `used`. It's an assertion, not a recovery path.

### Insertion: tombstones first, then NULL

```c
static HashEntry *get_or_insert_entry(HashMap *map, char *key, int keylen) {
  if (!map->buckets) {
    map->buckets = calloc(INIT_SIZE, sizeof(HashEntry));
    map->capacity = INIT_SIZE;
  } else if ((map->used * 100) / map->capacity >= HIGH_WATERMARK) {
    rehash(map);
  }

  uint64_t hash = fnv_hash(key, keylen);

  for (int i = 0; i < map->capacity; i++) {
    HashEntry *ent = &map->buckets[(hash + i) % map->capacity];

    if (match(ent, key, keylen))
      return ent;

    if (ent->key == TOMBSTONE) {
      ent->key = key;
      ent->keylen = keylen;
      return ent;
    }

    if (ent->key == NULL) {
      ent->key = key;
      ent->keylen = keylen;
      map->used++;
      return ent;
    }
  }
  unreachable();
}
```

The order of the three checks is the load-bearing detail. `match` first, then `TOMBSTONE`, then `NULL`. If the key already exists in the table, even one bucket beyond a tombstone, the function returns the existing entry; the tombstone is left undisturbed. If the key doesn't exist but a tombstone is on the probe path, the first tombstone gets reused and `map->used` is *not* incremented (the slot was already counted). If neither, the first NULL bucket gets a new entry and `map->used` goes up.

This is the standard tombstone scheme. The subtle case is that a probe must continue past a tombstone to confirm absence. If the probe stopped at the first tombstone and reused it, an existing key further down the chain would silently get duplicated. The match-first ordering prevents that.

### `TOMBSTONE`

```c
#define TOMBSTONE ((void *)-1)
```

A pointer with value `-1` (cast to `void *`). On x86-64 this is a non-canonical address that no real allocation will produce. Any sentinel value distinct from `NULL` and from a valid heap pointer would work; `(void *)-1` is the C tradition. The `match` predicate explicitly excludes it:

```c
static bool match(HashEntry *ent, char *key, int keylen) {
  return ent->key && ent->key != TOMBSTONE &&
         ent->keylen == keylen && memcmp(ent->key, key, keylen) == 0;
}
```

Comparing `keylen` first short-circuits the `memcmp` for length mismatches.

### The six API functions

```c
void *hashmap_get(HashMap *map, char *key);
void *hashmap_get2(HashMap *map, char *key, int keylen);
void hashmap_put(HashMap *map, char *key, void *val);
void hashmap_put2(HashMap *map, char *key, int keylen, void *val);
void hashmap_delete(HashMap *map, char *key);
void hashmap_delete2(HashMap *map, char *key, int keylen);
```

Three operations, two flavors each. The non-`2` flavors are one-line shims that compute `strlen(key)` and call the `2`-suffixed versions. `hashmap_delete` is implemented as a single store: find the entry and overwrite its `key` with `TOMBSTONE`. The value pointer is left unchanged but no longer reachable.

Notice what's *not* in the API: no iteration. Walking all entries is not a supported primitive. The hashmap is meant for point lookups by string key, and that's all. A future commit (in this same chapter) needs iteration over `include_paths` for `#include_next`, but `include_paths` is a `StringArray`, not a `HashMap`.

### `hashmap_test` and the driver hook

The bottom of `hashmap.c` carries a 5,000-key stress test. Insert keys 0–4999, delete 1000–1999, re-insert 1500–1599, insert 6000–6999. Then assert that lookups behave correctly:

```c
for (int i = 0; i < 1000; i++)
  assert((size_t)hashmap_get(map, format("key %d", i)) == i);
for (int i = 1000; i < 1500; i++)
  assert(hashmap_get(map, "no such key") == NULL);
...
```

The test is invoked by adding `-hashmap-test` as a driver flag:

```c
if (!strcmp(argv[i], "-hashmap-test")) {
  hashmap_test();
  exit(0);
}
```

And `test/driver.sh` adds one line that exercises it:

```sh
$chibicc -hashmap-test
check 'hashmap'
```

The hashmap won't have an external user until the next commit. This one stands alone — datastructure plus self-test — so that the next three diffs that adopt it can be small and local.

**Where we are.** The compiler now has a generic open-addressing string hashmap with FNV-1a hashing, 70% high watermark, 50% low watermark after rehash, and tombstone deletion. The API uses a `(pointer, length)` key shape that fits tokens directly. No callers yet; three commits' worth coming next.

---

## 22.3 — Three hashmap users: macros, scopes, keywords

> `git checkout 30520e5a7c73a6613cfcef38d72058e7cccde1f4` — *Use hashmap for macro name lookup*
>
> `git checkout 655954e301621737988a4fa0a2c72ffc24285c8d` — *Use hashmap for block-scope lookup*
>
> `git checkout f6944133d211ec6fb71c41f118905e16a752135b` — *Use hashmap for keyword lookup*

Three commits, each replacing a linear scan with a single `hashmap_get`. The three call sites profile differently:

- **Macros.** The macro table is small in absolute terms (a few hundred entries with system headers included) but `find_macro` runs once per identifier token in the entire translation unit. Lookup-heavy.
- **Block scopes.** Each function body opens a scope, accumulates local variables (and tags), and tears the scope down. Insert and lookup are both tied to scope lifetime. Lookup-medium, lifetime-short.
- **Keywords.** A fixed set of about thirty strings. Insert-once at first call, then lookup-many for the rest of the translation unit.

Each conversion is a small diff. Together they remove the three most visible linear scans in the compiler.

### 22.3.1 — Macros

The pre-conversion `find_macro` walks a linked list:

```c
for (Macro *m = macros; m; m = m->next)
  if (strlen(m->name) == tok->len && !strncmp(m->name, tok->loc, tok->len))
    return m->deleted ? NULL : m;
return NULL;
```

The `deleted` flag is a tombstone in disguise — `undef_macro` previously added a fresh entry with `deleted = true`, which masked any earlier definition. The post-conversion code is two lines:

```c
return hashmap_get2(&macros, tok->loc, tok->len);
```

The `Macro` struct sheds two fields: `next` (the linked-list pointer) and `deleted` (because the hashmap has real deletion via `TOMBSTONE`). `add_macro` becomes a `hashmap_put`; `undef_macro` becomes a `hashmap_delete`.

```c
void undef_macro(char *name) {
-  Macro *m = add_macro(name, true, NULL);
-  m->deleted = true;
+  hashmap_delete(&macros, name);
}
```

This is the cleanest of the three. The hashmap's tombstone scheme exactly matches what `undef_macro` was already simulating with the `deleted` flag, so the data-structure swap removes a workaround rather than introducing one.

### 22.3.2 — Block scopes

The pre-conversion `Scope` carried two linked lists, one for variable scope (`VarScope`) and one for tag scope (`TagScope`). Each had a `next` pointer and a `name` field. Lookup walked the chain inside-out; insert prepended.

After the swap, both lists become `HashMap`:

```c
struct Scope {
  Scope *next;

  // C has two block scopes; one is for variables/typedefs and
  // the other is for struct/union/enum tags.
-  VarScope *vars;
-  TagScope *tags;
+  HashMap vars;
+  HashMap tags;
};
```

`VarScope` and `TagScope` lose their `next` and `name` fields; the hashmap stores those for them. `TagScope` collapses to just a `Type *` value — the `name` is the hashmap key, the `ty` is the value, so a separate struct isn't needed at all:

```c
-typedef struct TagScope TagScope;
-struct TagScope {
-  TagScope *next;
-  char *name;
-  Type *ty;
-};
```

The tag scope hashmap keys are now the type names; values are `Type *` pointers directly. `push_tag_scope` becomes:

```c
static void push_tag_scope(Token *tok, Type *ty) {
  hashmap_put2(&scope->tags, tok->loc, tok->len, ty);
}
```

`find_var` and `find_tag` walk the chain of `Scope`s but lookup *within* a scope is now `O(1)`:

```c
static VarScope *find_var(Token *tok) {
  for (Scope *sc = scope; sc; sc = sc->next) {
    VarScope *sc2 = hashmap_get2(&sc->vars, tok->loc, tok->len);
    if (sc2)
      return sc2;
  }
  return NULL;
}
```

The asymptotic improvement here is real. A function with ten nested scopes, each with twenty locals, used to do up to 200 string compares per identifier reference. After the swap it does ten `hashmap_get2` calls, each `O(1)` on average.

The `struct_union_decl` redefinition path also gets simpler:

```c
-for (TagScope *sc = scope->tags; sc; sc = sc->next) {
-  if (equal(tag, sc->name)) {
-    *sc->ty = *ty;
-    return sc->ty;
-  }
-}
+Type *ty2 = hashmap_get2(&scope->tags, tag->loc, tag->len);
+if (ty2) {
+  *ty2 = *ty;
+  return ty2;
+}
```

A linear scan of one scope's tags becomes a single point lookup.

`find_func` (the lookup that gets called when the parser sees a call to an undeclared function) uses the outermost scope's hashmap directly:

```c
VarScope *sc2 = hashmap_get(&sc->vars, name);
if (sc2 && sc2->var && sc2->var->is_function)
  return sc2->var;
```

The old version walked every var in the global scope linearly, filtering by `is_function`. The new version finds the named variable in `O(1)` and then checks whether it happens to be a function.

### 22.3.3 — Keywords

The pre-conversion `is_typename` and `is_keyword` each held a static array of about thirty strings and walked the array linearly:

```c
static char *kw[] = { "void", "_Bool", "char", ... };

for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
  if (equal(tok, kw[i]))
    return true;
return false;
```

Both are called for every identifier token — `is_keyword` from `tokenize`, `is_typename` from `declspec` — so a thirty-element scan runs constantly. The post-conversion code lazily builds a hashmap on the first call:

```c
static bool is_keyword(Token *tok) {
  static HashMap map;

  if (map.capacity == 0) {
    static char *kw[] = { ... };
    for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)
      hashmap_put(&map, kw[i], (void *)1);
  }

  return hashmap_get2(&map, tok->loc, tok->len);
}
```

Two details. First, the `(void *)1` value is a "present" sentinel; the hashmap stores values as `void *`, and the `1` is just "anything non-NULL." Anything non-NULL would do; `1` is conventional. Second, the `map.capacity == 0` check is the lazy-init guard. A `HashMap` declared `static` is zero-initialized, so `capacity == 0` means "not yet populated."

The build is once-per-program — the static `HashMap` persists across calls. The thirty `hashmap_put` calls happen on the first invocation; subsequent calls go straight to the lookup. This is a textbook conversion of a fixed-set linear scan into an `O(1)` lookup. The hashmap is overkill for thirty entries; a perfect-hash table would be faster. But the hashmap is already there, the code is local, and the constant factor is small enough that it doesn't matter.

The same conversion lands twice: once for `is_keyword` (tokenize.c) and once for `is_typename` (parse.c). The two keyword sets overlap heavily but aren't identical — `is_typename` carries `typedef`, `enum`, `_Alignas`, and the storage-class words; `is_keyword` carries the control-flow words like `return`, `if`, `for`. Two separate hashmaps, two separate static arrays. Rui doesn't try to share.

**Where we are.** The hashmap has its three intended customers. Macro lookup, block-scope variable lookup, tag lookup, and the two keyword checks all run in `O(1)`. The macro and tag types lose vestigial `next` and `deleted` fields; the linear-scan loops disappear from `find_macro`, `find_var`, `find_tag`, `is_keyword`, and `is_typename`. The compiler should be measurably faster on translation units with many headers.

---

## 22.4 — The `-M` family

> `git checkout d0c4667b6bccf35ddf069c777689cd18c6a632b3` — *Add -M option*
>
> `git checkout 95d5a46234f98f3793c965bebe036361cbb1978e` — *Add -MF option*
>
> `git checkout 57c1d4ec0290d49fa1e954ff3e7a51e24d71a3a1` — *Add -MP option*
>
> `git checkout db850f37a2a284bf18cea427e4676a22d83d04b8` — *Add -MT option*
>
> `git checkout fb5cfe5d17fd0c0cbc0d17789c065b9bb86ba3c4` — *Add -MD option*
>
> `git checkout 7aa72e41e6b2703b3f357507252008ebe25dc08d` — *Add -MQ option*
>
> `git checkout c3edffbbb06be9d586ee4f1cf678049b7d81369d` — *Add -MMD option*

Seven commits, all on `main.c`, all driver-side. The `-M` family is gcc's mechanism for telling `make` which header files a `.c` file depends on. The compiler reads the source, runs the preprocessor far enough to know which `#include`s would have been pulled in, and writes a Makefile rule whose target is the `.o` file and whose dependencies are every file the preprocessor touched.

Real build systems use this constantly. `gcc -MMD -c foo.c` produces both `foo.o` (the object file) and `foo.d` (the dependency rule). The `Makefile` includes `foo.d`, so when a header changes, `make` recompiles `foo.o` automatically. Without this feature, a build system has to either rebuild everything or maintain dependencies by hand. Real-world code can't be built by chibicc until it grows this family.

The family is seven flags. `-M` writes a rule to stdout. `-MF FILE` writes it to a file. `-MP` adds phony rules so deletions don't break the build. `-MT TARGET` overrides the rule's target name. `-MD` enables dependency emission *alongside* normal compilation. `-MQ` is `-MT` with Makefile escaping. `-MMD` is `-MD` minus system headers.

### 22.4.1 — `-M`: write a rule to stdout

The simplest of the seven. `parse_args` adds a flag:

```c
if (!strcmp(argv[i], "-M")) {
  opt_M = true;
  continue;
}
```

`cc1` checks the flag after preprocessing and prints the rule:

```c
if (opt_M) {
  print_dependencies();
  return;
}
```

`print_dependencies` is the core of the chapter's middle:

```c
static void print_dependencies(void) {
  FILE *out = open_file(opt_o ? opt_o : "-");
  fprintf(out, "%s:", replace_extn(base_file, ".o"));

  File **files = get_input_files();

  for (int i = 0; files[i]; i++)
    fprintf(out, " \\\n  %s", files[i]->name);
  fprintf(out, "\n\n");
}
```

The dependency rule's target is the source file's name with `.c` replaced by `.o` — `replace_extn(base_file, ".o")`. The dependencies come from `get_input_files()`, the tokenizer-side function that returns every `File` struct created during preprocessing: the source file plus every `#include`d header. They're printed one-per-line with a backslash continuation.

`get_input_files` was added back in Chapter 17 as part of the `__BASE_FILE__` machinery. The tokenizer maintains an `input_files` array: every `tokenize_file` call appends a new `File`. Now that array becomes the source of truth for `-M`.

The driver also has to route a `-M` invocation through `cc1` exactly once, because the `-M` output is generated during preprocessing rather than during code generation:

```c
if (opt_E || opt_M) {
  run_cc1(argc, argv, input, NULL);
  continue;
}
```

`-M` joins `-E` as a "preprocess and stop" mode.

### 22.4.2 — `-MF FILE`: redirect to a file

`-MF` is a one-line addition: route the dependency output to a specific file rather than stdout. The flag takes an argument, so it goes into `take_arg`:

```c
char *x[] = {"-o", "-I", "-idirafter", "-include", "-x", "-MF"};
```

And `print_dependencies` learns a path-selection rule:

```c
char *path;
if (opt_MF)
  path = opt_MF;
else if (opt_o)
  path = opt_o;
else
  path = "-";
```

The fallback chain is `-MF` → `-o` → stdout. Without `-MF`, a `-M -o foo.d` invocation routes to `foo.d`; with `-MF foo.d`, the same invocation can use a different `-o`. This matters because `-o` typically names the *object* file in real builds, not the dependency file.

### 22.4.3 — `-MP`: phony rules for deleted headers

`-MP` adds a tail to the dependency output: an empty rule for each header. The motivation is the failure mode where someone deletes a header and the `.d` file still references it — `make` then complains "no rule to make `removed.h`." A phony rule silences that.

```c
if (opt_MP)
  for (int i = 1; files[i]; i++)
    fprintf(out, "%s:\n\n", files[i]->name);
```

Note the `i = 1`, not `i = 0`. `files[0]` is the source file itself (the `.c`); only the headers get phony rules. The output looks like:

```
foo.o: foo.c \
  bar.h \
  baz.h

bar.h:

baz.h:
```

With those empty rules in place, `make` treats `bar.h` as a target with no prerequisites and no recipe. If `bar.h` doesn't exist, `make` is satisfied because the phony rule says "this target needs nothing." If `bar.h` does exist, the original `foo.o: ... bar.h ...` rule still triggers a rebuild on change.

### 22.4.4 — `-MT TARGET`: override the target name

`-MT` lets the user rename the rule's target. The default is `replace_extn(base_file, ".o")` — the source filename with `.o`. With `-MT foo.lo`, the rule says `foo.lo:` instead.

```c
if (!strcmp(argv[i], "-MT")) {
  if (opt_MT == NULL)
    opt_MT = argv[++i];
  else
    opt_MT = format("%s %s", opt_MT, argv[++i]);
  continue;
}
```

The if-else handles repeated `-MT`: each one *appends* to the target list, space-separated. So `-MT foo.o -MT bar.o` produces `foo.o bar.o:` as the rule head. Real Makefiles sometimes want this when one command produces multiple outputs.

`print_dependencies` uses it directly:

```c
fprintf(out, "%s:", opt_MT ? opt_MT : replace_extn(base_file, ".o"));
```

### 22.4.5 — `-MD`: dependencies alongside compilation

`-M` was preprocess-only. `-MD` says "do the normal compile *and* also write a dependency file." The dependency-file path defaults to the source file's name with `.d`:

```c
else if (opt_MD)
  path = replace_extn(opt_o ? opt_o : base_file, ".d");
```

And `cc1`'s dispatch grows an `else` branch:

```c
if (opt_M || opt_MD) {
  print_dependencies();
  if (opt_M)
    return;
}
```

`-M` returns early (preprocess-only); `-MD` falls through to normal compilation. The two modes share the dependency-emission path but differ in what happens after.

The implication: `-MD` is the normal mode for build systems. `make` invokes `gcc -MD -c foo.c -o foo.o`, which produces both `foo.o` and `foo.d`. The next time `make` runs, it includes `foo.d` and knows the headers `foo.c` depends on.

### 22.4.6 — `-MQ`: like `-MT` with Makefile escaping

`-MQ` is `-MT` plus escaping for special characters in Makefile syntax. A target like `foo$bar.o` would confuse `make` (which interprets `$` as a variable expansion); `-MQ` applies Makefile escaping rules.

The escaper:

```c
static char *quote_makefile(char *s) {
  char *buf = calloc(1, strlen(s) * 2 + 1);

  for (int i = 0, j = 0; s[i]; i++) {
    switch (s[i]) {
    case '$':
      buf[j++] = '$';
      buf[j++] = '$';
      break;
    case '#':
      buf[j++] = '\\';
      buf[j++] = '#';
      break;
    case ' ':
    case '\t':
      for (int k = i - 1; k >= 0 && s[k] == '\\'; k--)
        buf[j++] = '\\';
      buf[j++] = '\\';
      buf[j++] = s[i];
      break;
    default:
      buf[j++] = s[i];
      break;
    }
  }
  return buf;
}
```

Three transformations:

- `$` becomes `$$`. (Makefiles read `$$` as a literal dollar sign.)
- `#` becomes `\#`. (Make uses `#` for comments.)
- Whitespace becomes backslash-prefixed, with extra backslash-doubling if the original was already preceded by backslashes. (A path like `foo\\bar baz` becomes `foo\\\\bar\ baz`.)

Each `-MQ TARGET` runs `quote_makefile` on the argument before joining:

```c
if (!strcmp(argv[i], "-MQ")) {
  if (opt_MT == NULL)
    opt_MT = quote_makefile(argv[++i]);
  else
    opt_MT = format("%s %s", opt_MT, quote_makefile(argv[++i]));
  continue;
}
```

Once `-MQ` exists, `print_dependencies` also routes the *default* target (the no-`-MT` path) through `quote_makefile`, plus the phony rule names from `-MP`:

```c
if (opt_MT)
  fprintf(out, "%s:", opt_MT);
else
  fprintf(out, "%s:", quote_makefile(replace_extn(base_file, ".o")));
...
if (opt_MP)
  for (int i = 1; files[i]; i++)
    fprintf(out, "%s:\n\n", quote_makefile(files[i]->name));
```

So the escaping covers both the user-supplied target and the auto-generated paths. Notably, *dependency* names (the right-hand side of the rule) are *not* escaped — chibicc emits them raw. This is a subtle gap; gcc escapes both sides. Filenames with `$` or spaces in them would produce a malformed `.d` file. The handful of build systems that hit this in practice tend to avoid such filenames anyway, but it's an asymmetry worth noting.

### 22.4.7 — `-MMD`: dependencies, but skip system headers

The last commit in the family is a small filter. `-MMD` enables `-MD` but excludes any header that came from a "standard" include path (the ones added by `add_default_include_paths` rather than user-supplied `-I`).

```c
static void add_default_include_paths(char *argv0) {
  ...
  strarray_push(&include_paths, "/usr/include");

  // Keep a copy of the standard include paths for -MMD option.
  for (int i = 0; i < include_paths.len; i++)
    strarray_push(&std_include_paths, include_paths.data[i]);
}
```

A separate `std_include_paths` `StringArray` records which paths the *driver* added by default. User `-I` flags push to `include_paths` later, after this loop runs, so `std_include_paths` captures only the system set.

The filter:

```c
static bool in_std_include_path(char *path) {
  for (int i = 0; i < std_include_paths.len; i++) {
    char *dir = std_include_paths.data[i];
    int len = strlen(dir);
    if (strncmp(dir, path, len) == 0 && path[len] == '/')
      return true;
  }
  return false;
}
```

A simple prefix-match. If the file's path starts with one of the standard include-path directories, the file is excluded from the dependency list.

`print_dependencies` consults the filter twice, once for the dependency list and once for the phony-rule list:

```c
for (int i = 0; files[i]; i++) {
  if (opt_MMD && in_std_include_path(files[i]->name))
    continue;
  fprintf(out, " \\\n  %s", files[i]->name);
}
```

The motivation: system headers (`<stdio.h>`, `<stdlib.h>`) don't change between builds. Putting them in the dependency file just adds noise to `make`'s dependency graph. `-MMD` is the form most build systems actually use.

`-MMD` is implemented as a strict superset of `-MD`:

```c
if (!strcmp(argv[i], "-MMD")) {
  opt_MD = opt_MMD = true;
  continue;
}
```

Both flags get set; `-MMD` is just `-MD` plus the system-header filter. The `print_dependencies` body is shared.

**Where we are.** Chibicc emits Makefile-shaped dependency rules. `-M` writes to stdout; `-MF` redirects; `-MP` adds phony rules; `-MT` and `-MQ` rename the target (with and without escaping); `-MD` and `-MMD` enable rule emission alongside compilation, with `-MMD` filtering out system headers. The rule format is simple: target plus backslash-continued dependency list. Build systems that drive `gcc -MMD` will work with chibicc by changing only the compiler binary.

---

## 22.5 — `-fpic`/`-fPIC` and the file-search cache

> `git checkout 86785fceb169bc754efe3f29a9b63137f5c9a106` — *Add -fpic and -fPIC options*
>
> `git checkout c0f0614e6b7647fd4703abf4c455024c2ade8cd7` — *Cache file search results*

Two commits with very different sizes. The `-fpic` commit changes codegen — the bigger surprise of the two. The file-search commit is nine lines that thread the new hashmap into `search_include_paths`.

### 22.5.1 — `-fpic` and `-fPIC`: codegen changes

A naive read of the change suggests `-fpic` should be a flag-flip that the linker reads. Chibicc's existing codegen emits `lea name(%rip), %rax` for global addresses, which is rip-relative and therefore PIC-friendly. So why does the codegen change?

Because rip-relative addressing is PIC-friendly for *function-internal* data, but it's not enough for *cross-module* data. Inside a single shared library, two functions can reach each other and reach static globals via rip-relative offsets. Across shared libraries, a reference has to go through the Global Offset Table (GOT), because the loader binds shared-library symbols at load time and the GOT is the indirection that lets the binding work without rewriting code.

The commit adds a second codegen path in `gen_addr` for global names:

```c
if (opt_fpic) {
  // Thread-local variable
  if (node->var->is_tls) {
    println("  data16 lea %s@tlsgd(%%rip), %%rdi", node->var->name);
    println("  .value 0x6666");
    println("  rex64");
    println("  call __tls_get_addr@PLT");
    return;
  }

  // Function or global variable
  println("  mov %s@GOTPCREL(%%rip), %%rax", node->var->name);
  return;
}
```

Two new patterns, both new to the chapter.

The non-TLS pattern is the *general-dynamic* GOT access. `mov name@GOTPCREL(%rip), %rax` does two things in one instruction: the assembler emits a relocation `R_X86_64_REX_GOTPCRELX` against `name`, the linker materializes a GOT entry for `name`, and the load reads the GOT entry's value (which is the runtime address of `name`) into `%rax`. The non-PIC version was a single `lea`; the PIC version is a `mov` that loads through the GOT.

The TLS pattern is even more elaborate. Chapter 21 §21.1 emitted the *initial-exec* TLS sequence (`mov %fs:0, %rax; add $name@tpoff, %rax`), which works for variables in the executable but not for variables in shared libraries. PIC code running in a shared library has to use the *general-dynamic* model, which calls `__tls_get_addr` to retrieve the address. The four-instruction sequence is the canonical gcc pattern for the call:

```
data16 lea  name@tlsgd(%rip), %rdi
.value 0x6666
rex64
call __tls_get_addr@PLT
```

The `data16` prefix and the `0x6666` `.value` are padding bytes that the linker can rewrite to convert this general-dynamic call into a cheaper local-dynamic or initial-exec sequence at link time, if it determines the variable is reachable in the local module. The four instructions occupy 16 bytes total, which is what the linker needs for the in-place rewrite.

This is a real bit of TLS arcana — the padding is documented in the AMD64 ABI, but it's not the kind of detail one would invent. Rui likely lifted the sequence from gcc's output. It works because the linker recognizes the exact byte pattern and either leaves it alone (general-dynamic stays) or rewrites it (`mov`/`lea`/`add` triple replaces the `data16 lea ... call`).

Without `-fpic`, the original two patterns from Chapter 21 fire — `lea name(%rip), %rax` for globals, the `%fs:0 + @tpoff` pair for TLS. With `-fpic`, the GOT and `__tls_get_addr` paths fire. The flag selects between them at codegen time.

The psABI conformance count grows by one. Chibicc now emits both PIC and non-PIC sequences; the GOT and `__tls_get_addr` sequences are the standard psABI-mandated forms for shared-library code. New count: nineteen.

The `-fpic` and `-fPIC` flags both set the same boolean. On real toolchains they differ: `-fpic` allows a smaller GOT (fits in 16-bit offsets), `-fPIC` allows arbitrarily large ones. Chibicc doesn't make the distinction — both flags use the same large-model code, which is a strict superset of what `-fpic` requires. Flag-compatible without size optimization.

### 22.5.2 — Cache file-search results

The smaller half. `search_include_paths` was a linear walk over `include_paths.len`, calling `file_exists(path)` for each candidate. Each `file_exists` is a `stat` syscall. For a translation unit that includes many headers — and many headers transitively pulled in by `<stdio.h>` and friends — the same lookup repeats many times.

The fix is a single hashmap:

```c
static HashMap cache;
char *cached = hashmap_get(&cache, filename);
if (cached)
  return cached;

for (int i = 0; i < include_paths.len; i++) {
  char *path = format("%s/%s", include_paths.data[i], filename);
  if (!file_exists(path))
    continue;
  hashmap_put(&cache, filename, path);
  return path;
}
```

The cache key is the *filename* as written in the `#include` directive (e.g. `"stdio.h"`). The cache value is the resolved absolute path (e.g. `"/usr/include/stdio.h"`). On a hit, `search_include_paths` returns the cached path immediately, without any `file_exists` syscalls or string formatting. On a miss, the function falls through to the linear search; on a successful resolution, the path is cached for next time.

Negative results — filenames that aren't found — are not cached. A `#include "missing.h"` will repeat the full search every time it's hit. In practice this case is rare; preprocessing hits a missing header once and errors out.

This is one of the few places in the compiler where a hashmap genuinely speeds things up by an order of magnitude. The cost is minimal — one new `static HashMap` and three lines of code. The benefit is that a translation unit which `#include`s `<stdio.h>` thirty times across header chains pays the search cost once.

**Where we are.** `-fpic`/`-fPIC` enable PIC-friendly codegen for global and TLS variable addresses. Globals go through the GOT (`mov name@GOTPCREL(%rip), %rax`); TLS variables call `__tls_get_addr` with the linker-rewritable padding sequence. The non-PIC paths from Chapter 21 still fire when the flag is absent. The file-search path is now hashmap-backed, eliminating repeated `stat` calls on the same `#include` filename.

---

## 22.6 — Include-guard optimization, `#pragma once`, `#include_next`

> `git checkout d48d9e5ae35b5eb1a9dcb0c07c1dba9e65bd83f3` — *Add include guard optimization*
>
> `git checkout a6c662207d38813b3dd490d81d8afe14ac99272b` — *[GNU] Add "#pragma once"*
>
> `git checkout f10bcebaa5df6bcb8e08e622ac44b0098e3133ae` — *[GNU] Add #include_next*

Three commits, all in `preprocess.c`. The first two implement the same optimization with two different triggers (pattern detection and explicit pragma). The third adds the GNU mechanism for chained system headers.

### 22.6.1 — Include-guard optimization

The classic include-guard pattern is:

```c
#ifndef FOO_H
#define FOO_H
... contents ...
#endif
```

Every `#include "foo.h"` after the first one tokenizes the entire file, walks past the `#ifndef` (which is now false because `FOO_H` is defined), skips the body via `skip_cond_incl`, and ends. The work is done once per include, even when the second through Nth includes contribute nothing. For a header included by many transitive paths, this can be a substantial fraction of the preprocessor's runtime.

The optimization detects the pattern on the *first* read, records the guard macro name, and short-circuits subsequent reads to skip the file entirely.

The detector:

```c
static char *detect_include_guard(Token *tok) {
  // Detect the first two lines.
  if (!is_hash(tok) || !equal(tok->next, "ifndef"))
    return NULL;
  tok = tok->next->next;

  if (tok->kind != TK_IDENT)
    return NULL;

  char *macro = strndup(tok->loc, tok->len);
  tok = tok->next;

  if (!is_hash(tok) || !equal(tok->next, "define") || !equal(tok->next->next, macro))
    return NULL;

  // Read until the end of the file.
  while (tok->kind != TK_EOF) {
    if (!is_hash(tok)) {
      tok = tok->next;
      continue;
    }

    if (equal(tok->next, "endif") && tok->next->next->kind == TK_EOF)
      return macro;

    if (equal(tok, "if") || equal(tok, "ifdef") || equal(tok, "ifndef"))
      tok = skip_cond_incl(tok->next);
    else
      tok = tok->next;
  }
  return NULL;
}
```

Three checks:

1. The first directive is `#ifndef IDENT`.
2. The next directive is `#define IDENT` with the *same* identifier.
3. The closing `#endif` is the last token in the file (its `next` is `TK_EOF`).

If any check fails, return NULL — the file isn't guard-shaped. If they all pass, return the guard macro name.

The middle of the file is walked via `skip_cond_incl` for nested `#if`/`#ifdef`/`#ifndef` blocks (so a nested conditional doesn't break the outer match), but *non-conditional* `#define`s and `#include`s and other directives don't disqualify the file. The check is "the entire file body is wrapped in the guard," not "the file body contains nothing but conditionals."

The detector runs once per file:

```c
static Token *include_file(Token *tok, char *path, Token *filename_tok) {
  static HashMap include_guards;
  char *guard_name = hashmap_get(&include_guards, path);
  if (guard_name && hashmap_get(&macros, guard_name))
    return tok;

  Token *tok2 = tokenize_file(path);
  if (!tok2)
    error_tok(filename_tok, "%s: cannot open file: %s", path, strerror(errno));

  guard_name = detect_include_guard(tok2);
  if (guard_name)
    hashmap_put(&include_guards, path, guard_name);

  return append(tok2, tok);
}
```

The `include_guards` hashmap maps absolute include paths to guard macro names. On a re-include, two checks fire: is this file in the cache, and is its guard macro currently defined? If both, return immediately — the file is known to be a no-op. If not, tokenize the file as usual and try to record a guard for next time.

The two-stage check matters. A file's guard could in principle be `#undef`-ed after the first include, in which case the second include should run. The `hashmap_get(&macros, guard_name)` check picks that up — it consults the live macro table, which `#undef` would have cleared.

What this *doesn't* do is what gcc does: it doesn't avoid re-tokenizing on subsequent includes that *aren't* fast-pathed. If the guard macro got `#undef`-ed, chibicc retokenizes the file and runs through the conditional. Gcc has more elaborate mechanisms for this; chibicc's optimization is the simple version.

### 22.6.2 — `#pragma once`

`#pragma once` is the GNU/MSVC extension that asks for the same optimization explicitly. A header that begins with `#pragma once` is included exactly once per translation unit, regardless of how it's structured.

The implementation reuses the cache pattern from §22.6.1:

```c
static HashMap pragma_once;
```

`include_file` consults it at the top:

```c
// Check for "#pragma once"
if (hashmap_get(&pragma_once, path))
  return tok;
```

And `preprocess2` recognizes the pragma:

```c
if (equal(tok, "pragma") && equal(tok->next, "once")) {
  hashmap_put(&pragma_once, tok->file->name, (void *)1);
  tok = skip_line(tok->next->next);
  continue;
}
```

Once a file's path is in `pragma_once`, subsequent `include_file` calls on the same path return without tokenizing. The `(void *)1` is the same "present" sentinel as in the keyword hashmaps.

This conditional is matched *before* the existing `equal(tok, "pragma")` catch-all (which silently consumes any other `#pragma`). So `#pragma once` is recognized and acted on; everything else still gets dropped.

The test (`test/pragma-once.c`) is a self-recursive include:

```c
#include "test.h"

#pragma once

#include "test/pragma-once.c"

int main() {
  printf("OK\n");
  return 0;
}
```

The file includes itself. Without `#pragma once` this would loop forever (or, more precisely, would fail when the inner copy of `main` redefines the outer one). With `#pragma once`, the second `#include` is a no-op, and the test runs.

### 22.6.3 — `#include_next`

`#include_next` is the GNU mechanism for chained system headers. It says: "find a file with this name, but skip the directory that the *current* file came from, and continue searching the include path from there."

The motivating use case is wrapper headers. A distribution might ship a wrapper `/usr/include/stdio.h` that adds some custom declarations and then does `#include_next <stdio.h>` to pull in the real one from another location. Without `#include_next`, the wrapper would re-include itself in an infinite loop.

The implementation needs to track *where* in the search path the current file was found, then resume from one slot later. Chibicc threads this through `search_include_paths`:

```c
static int include_next_idx;

char *search_include_paths(char *filename) {
  ...
  for (int i = 0; i < include_paths.len; i++) {
    char *path = format("%s/%s", include_paths.data[i], filename);
    if (!file_exists(path))
      continue;
    hashmap_put(&cache, filename, path);
    include_next_idx = i + 1;
    return path;
  }
  return NULL;
}
```

When a successful search lands on `include_paths[i]`, the global `include_next_idx` is set to `i + 1`. The next `#include_next` in the same file uses that index as its starting point:

```c
static char *search_include_next(char *filename) {
  for (; include_next_idx < include_paths.len; include_next_idx++) {
    char *path = format("%s/%s", include_paths.data[include_next_idx], filename);
    if (file_exists(path))
      return path;
  }
  return NULL;
}
```

Note that `include_next_idx` is only updated on a *fresh* (non-cached) search. A cached lookup returns immediately without touching the index, which means `#include_next` after a cache hit uses whatever index the most recent fresh search left behind. In the common case (a wrapper header that does `#include_next` once, immediately) this works; in more elaborate scenarios it could surprise. Chibicc's clientele isn't sophisticated enough to hit the edge case.

`preprocess2` adds the directive:

```c
if (equal(tok, "include_next")) {
  bool ignore;
  char *filename = read_include_filename(&tok, tok->next, &ignore);
  char *path = search_include_next(filename);
  tok = include_file(tok, path ? path : filename, start->next->next);
  continue;
}
```

The search uses `search_include_next` rather than `search_include_paths`. The double-quote-vs-angle-bracket distinction is read but ignored (`bool ignore`) — `#include_next` is always treated as a path search, never a same-directory lookup.

The test sets up three directories with chained wrappers:

```sh
mkdir -p $tmp/next1 $tmp/next2 $tmp/next3
echo '#include "file1.h"' > $tmp/file.c
echo '#include_next "file1.h"' > $tmp/next1/file1.h
echo '#include_next "file2.h"' > $tmp/next2/file1.h
echo 'foo' > $tmp/next3/file2.h
$chibicc -I$tmp/next1 -I$tmp/next2 -I$tmp/next3 -E $tmp/file.c | grep -q foo
```

The chain is `file.c` → `next1/file1.h` (via `-I`) → `next2/file1.h` (via `#include_next`) → `next3/file2.h` (via `#include_next`). The final file contains `foo`, which the preprocessor output should contain.

**Where we are.** Three include-handling additions: pattern-based include-guard optimization (caches the guard macro name and short-circuits re-includes when it's defined), `#pragma once` (the explicit-opt-in form, with a separate hashmap of files that have asked for it), and `#include_next` (which resumes the search path past the directory the current file came from). All three use the new hashmap. Real-world headers with elaborate include chains now work.

---

## 22.7 — Linker-driver pass-throughs and the third-party harness

> `git checkout 1e9b6dd1108690f22c84af8db606fea9fb7ec2db` — *Add -static option*
>
> `git checkout 4e5de36a36452ef9fe29ac55f7812f2bb9005d95` — *Add -shared option*
>
> `git checkout c8df7874c607f14eac3774680b55ab22c3aaf370` — *Add -L option*
>
> `git checkout d1bc9a4eb0e205b10a583c347a9fe7d4bed7b813` — *Add -Wl, option*
>
> `git checkout 469f159bb1adebb92ca2c9a7841466a98e6ad956` — *Add -Xlinker option*
>
> `git checkout fb4937024db2ee06fd60ea3bb2cfc6c898646a7d` — *Add scripts to test third-party apps*

Six commits. Five are linker-driver additions; the last is a harness of shell scripts.

### 22.7.1 — `-static`

`-static` tells `ld` to produce a statically-linked executable. The driver flips a flag and pushes `-static` to `ld_extra_args`:

```c
if (!strcmp(argv[i], "-static")) {
  opt_static = true;
  strarray_push(&ld_extra_args, "-static");
  continue;
}
```

But the local `opt_static` flag does more than pass `-static` through: it also restructures the linker invocation. A statically-linked executable shouldn't carry a dynamic linker path, and its libgcc/libc grouping needs `--start-group` / `--end-group` to handle circular references between `libgcc` and `libc`:

```c
if (!opt_static) {
  strarray_push(&arr, "-dynamic-linker");
  strarray_push(&arr, "/lib64/ld-linux-x86-64.so.2");
}
```

```c
if (opt_static) {
  strarray_push(&arr, "--start-group");
  strarray_push(&arr, "-lgcc");
  strarray_push(&arr, "-lgcc_eh");
  strarray_push(&arr, "-lc");
  strarray_push(&arr, "--end-group");
} else {
  strarray_push(&arr, "-lc");
  strarray_push(&arr, "-lgcc");
  strarray_push(&arr, "--as-needed");
  strarray_push(&arr, "-lgcc_s");
  strarray_push(&arr, "--no-as-needed");
}
```

Static builds use `-lgcc_eh` (the static C++/exception-handling helper); dynamic builds use `-lgcc_s` (the shared variant) under `--as-needed`. The `--start-group`/`--end-group` brackets tell `ld` to retry the libraries in order until no new symbols are pulled in, which solves the circular-dependency problem that static builds always face.

The same commit also tidies the always-emitted `-L` paths — the previous version had `-L${libpath}` and `-L${libpath}/..` lines that are now collapsed into a single `-L/usr/lib/x86_64-linux-gnu`. Distribution-specific cleanup; not load-bearing for `-static` per se.

### 22.7.2 — `-shared`

`-shared` is symmetric with `-static`. A shared-library output uses `crtbeginS.o`/`crtendS.o` (the `S` suffix is for shared) instead of the executable startup files, and skips `crt1.o` (which carries `_start` for executables):

```c
if (opt_shared) {
  strarray_push(&arr, format("%s/crti.o", libpath));
  strarray_push(&arr, format("%s/crtbeginS.o", gcc_libpath));
} else {
  strarray_push(&arr, format("%s/crt1.o", libpath));
  strarray_push(&arr, format("%s/crti.o", libpath));
  strarray_push(&arr, format("%s/crtbegin.o", gcc_libpath));
}
```

```c
if (opt_shared)
  strarray_push(&arr, format("%s/crtendS.o", gcc_libpath));
else
  strarray_push(&arr, format("%s/crtend.o", gcc_libpath));
```

The `S`-suffixed crt files are PIC-friendly: they contain position-independent code suitable for shared-library inclusion. Without them, a shared library built by chibicc would have non-PIC startup code, which the loader would reject. Combined with `-fpic`, this completes the shared-library build path.

The driver also passes `-shared` through to `ld`:

```c
if (!strcmp(argv[i], "-shared")) {
  opt_shared = true;
  strarray_push(&ld_extra_args, "-shared");
  continue;
}
```

### 22.7.3 — `-L`

`-L DIR` adds a directory to the linker's library-search path. The driver accepts both the spaced form and the joined form (`-L/foo` and `-L /foo`):

```c
if (!strcmp(argv[i], "-L")) {
  strarray_push(&ld_extra_args, "-L");
  strarray_push(&ld_extra_args, argv[++i]);
  continue;
}

if (!strncmp(argv[i], "-L", 2)) {
  strarray_push(&ld_extra_args, "-L");
  strarray_push(&ld_extra_args, argv[i] + 2);
  continue;
}
```

Both forms emit the spaced version to `ld`. The pair-of-pushes is the canonical way to add a `-L` to an `ld` invocation. No special handling is needed beyond pass-through.

### 22.7.4 — `-Wl,` — pass through to the linker

`-Wl,arg1,arg2,...` is gcc's mechanism for passing arbitrary arguments to the linker. Each comma-separated piece becomes a separate linker argument. Chibicc treats `-Wl,` like `-l` — it goes through `input_paths` rather than `ld_extra_args` so command-line ordering relative to other inputs is preserved:

```c
if (!strncmp(argv[i], "-l", 2) || !strncmp(argv[i], "-Wl,", 4)) {
  strarray_push(&input_paths, argv[i]);
  continue;
}
```

Then the main input-processing loop, when it hits a `-Wl,`-prefixed string, splits it on commas and pushes each piece to `ld_args`:

```c
if (!strncmp(input, "-Wl,", 4)) {
  char *s = strdup(input + 4);
  char *arg = strtok(s, ",");
  while (arg) {
    strarray_push(&ld_args, arg);
    arg = strtok(NULL, ",");
  }
  continue;
}
```

`strtok` modifies the string in place (NUL-terminates each comma-separated piece). The `strdup` is necessary because the input may be a pointer into `argv`, which is shared. Each piece becomes a distinct linker argument: `-Wl,-z,muldefs,--gc-sections` becomes three arguments `-z`, `muldefs`, `--gc-sections`.

### 22.7.5 — `-Xlinker`

`-Xlinker arg` is the alternative to `-Wl,` for arguments that can't be expressed comma-separated. Each `-Xlinker` takes one argument and passes it to `ld` literally:

```c
if (!strcmp(argv[i], "-Xlinker")) {
  strarray_push(&ld_extra_args, argv[++i]);
  continue;
}
```

Note that `-Xlinker` goes to `ld_extra_args` (no command-line ordering), while `-Wl,` goes through `input_paths` (with ordering). The distinction matters when `-Wl,arg` references an object file or library that `-l` would also reference — `-Wl,` preserves the relative position. `-Xlinker` doesn't need that because its uses are typically option-shaped (`-z muldefs`, `--gc-sections`) rather than object-shaped.

### 22.7.6 — Third-party-app test scripts

The chapter's last commit doesn't change the compiler. It adds four shell scripts under `test/thirdparty/` that build real-world C codebases with chibicc:

- `git.sh` — builds and tests the git repository.
- `libpng.sh` — builds and tests libpng.
- `sqlite.sh` — builds and tests SQLite.
- `tinycc.sh` — builds the TinyCC compiler.

A shared `common` script handles the boilerplate:

```sh
make="make -j$(nproc)"
chibicc=`pwd`/chibicc

dir=$(basename -s .git $repo)

set -e -x

mkdir -p thirdparty
cd thirdparty
[ -d $dir ] || git clone $repo
cd $dir
```

Each script sources `common`, pins the upstream repository to a specific commit hash, and runs the project's own build:

```sh
#!/bin/bash
repo='git@github.com:rui314/libpng.git'
. test/thirdparty/common
git reset --hard dbe3e0c43e549a1602286144d94b0666549b18e6

CC=$chibicc ./configure
sed -i 's/^wl=.*/wl=-Wl,/; s/^pic_flag=.*/pic_flag=-fPIC/' libtool
$make clean
$make
$make test
```

The `sed` of `libtool` is the load-bearing piece. `./configure` writes a `libtool` script with hard-coded flag conventions for the compiler it detected; if it doesn't recognize chibicc (and it doesn't), it falls back to defaults that don't match. The `sed` patches in the chibicc-compatible flags after the fact. Two flags get patched: `wl` (the per-compiler form for "pass to linker" — `-Wl,` for chibicc, same as gcc) and `pic_flag` (`-fPIC`).

The libpng repo is `rui314/libpng`, not the upstream libpng. Rui maintains a fork with whatever small patches were needed to compile under chibicc — a real-world stress test that exposed minor incompatibilities and got fixed in the fork rather than in chibicc. Same for `git`, where the pinned hash is a tested-known-good revision.

The scripts aren't part of `make test` — they're invoked manually because they require network access and many minutes of compile time. But their existence is a milestone: chibicc can build production C codebases. The compiler is no longer a toy.

**Where we are.** The driver has the full linker-pass-through vocabulary: `-static` (with library-grouping changes), `-shared` (with `crt*S.o` startup-file selection), `-L`, `-Wl,` (comma-separated ordered pass-through), and `-Xlinker` (literal pass-through). The third-party harness exists to verify that real codebases compile. By this commit, chibicc is build-system-compatible enough to drop into existing C projects with minimal shimming.

---

## Recap

Twenty-three commits. The chapter adds three things in roughly equal measure: a reusable hashmap (used immediately in five places), the dependency-file machinery (seven `-M` flags), and the linker-driver completion (five flags plus the third-party harness). Plus a small parser tweak for labels-as-values, a codegen change for `-fpic`, the file-search cache, and the three include-handling additions.

- New file: `hashmap.c`. New types in `chibicc.h`: `HashMap`, `HashEntry`. Six functions in two flavors each (`hashmap_get`/`hashmap_get2`, etc.).
- The macro table, two block-scope tables (`vars` and `tags`), and two keyword tables (`is_keyword` and `is_typename`) all migrate from linked lists or linear arrays to `HashMap`.
- The file-search cache (`search_include_paths`), the include-guard cache (`include_guards`), and the `#pragma once` cache (`pragma_once`) are three more `HashMap`s in `preprocess.c`.
- `Macro` loses two fields (`next`, `deleted`); `VarScope` loses two (`next`, `name`); `TagScope` is dissolved entirely (its data flows into the hashmap directly).
- `Relocation->label` becomes `char **` to accommodate the lazy resolution of label-as-value names.
- `gen_addr` gains a `-fpic`-mode branch with two new asm patterns: `mov name@GOTPCREL(%rip), %rax` for globals and the four-instruction `__tls_get_addr` sequence for TLS.
- `parse_args` accepts `-M`, `-MF`, `-MP`, `-MT`, `-MQ`, `-MD`, `-MMD`, `-fpic`, `-fPIC`, `-static`, `-shared`, `-L`, `-Wl,`, and `-Xlinker`. The `take_arg` table is updated with `-MF`, `-MT`, and `-Xlinker`.
- `print_dependencies` is the new function that emits Makefile-shaped rules. `quote_makefile` handles `$`/`#`/whitespace escaping. `in_std_include_path` filters system headers for `-MMD`.
- `run_linker` learns conditional logic for `opt_static` (no dynamic-linker, `--start-group` library brackets) and `opt_shared` (`crtbeginS.o`/`crtendS.o` instead of executable startup files).
- `preprocess.c` gains `detect_include_guard` and `search_include_next`; `include_file` gains the guard-cache and `pragma_once`-cache lookups; `preprocess2` gains `pragma once` and `include_next` directive handling.
- `test/thirdparty/` is the new directory with four shell scripts pinning git, libpng, sqlite, and tinycc at known-good commits.

The chapter's twenty-three-row summary, in `main` order:

| # | Hash | Subject | Section |
|---|---|---|---|
| 284 | `f0c98e0` | [GNU] Treat labels-as-values as compile-time constant | §22.1 |
| 285 | `0aad326` | Add string hashmap | §22.2 |
| 286 | `30520e5` | Use hashmap for macro name lookup | §22.3 |
| 287 | `655954e` | Use hashmap for block-scope lookup | §22.3 |
| 288 | `f694413` | Use hashmap for keyword lookup | §22.3 |
| 289 | `d0c4667` | Add `-M` option | §22.4 |
| 290 | `95d5a46` | Add `-MF` option | §22.4 |
| 291 | `57c1d4e` | Add `-MP` option | §22.4 |
| 292 | `db850f3` | Add `-MT` option | §22.4 |
| 293 | `fb5cfe5` | Add `-MD` option | §22.4 |
| 294 | `7aa72e4` | Add `-MQ` option | §22.4 |
| 295 | `c3edffb` | Add `-MMD` option | §22.4 |
| 296 | `86785fc` | Add `-fpic` and `-fPIC` options | §22.5 |
| 297 | `c0f0614` | Cache file search results | §22.5 |
| 298 | `d48d9e5` | Add include guard optimization | §22.6 |
| 299 | `a6c6622` | [GNU] Add "#pragma once" | §22.6 |
| 300 | `f10bceb` | [GNU] Add `#include_next` | §22.6 |
| 301 | `1e9b6dd` | Add `-static` option | §22.7 |
| 302 | `4e5de36` | Add `-shared` option | §22.7 |
| 303 | `c8df787` | Add `-L` option | §22.7 |
| 304 | `d1bc9a4` | Add `-Wl,` option | §22.7 |
| 305 | `469f159` | Add `-Xlinker` option | §22.7 |
| 306 | `fb49370` | Add scripts to test third-party apps | §22.7 |

Errata candidates added this chapter:

- The Makefile-escape function (`quote_makefile`) is applied to the rule's *target* (default and user-supplied via `-MT`/`-MQ`) and to phony-rule target names (`-MP`), but not to the dependency list itself. A header path containing `$` or `#` would produce a malformed `.d` file. Gcc escapes both sides.
- The `include_next_idx` global is updated only on a fresh (non-cached) `search_include_paths` call. A cached lookup leaves it at whatever value the most recent fresh search left behind. In multi-file translation units with elaborate `#include_next` chains this could surprise.

Errata candidates closed this chapter: none. The remaining candidates are unchanged from Ch 21: Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's two (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), Ch 20's one (`is_compatible` array arm bug), Ch 21's two (`.size` missing for functions, suffix-only `.a`/`.so` recognition), plus the two added in this chapter.

The canonicalization-at-parse-time count is unchanged at eleven. The pre-factor-before-feature count grows by one if we count §22.2's hashmap as a refactor staged before its three users; alternatively, one can read the pair-and-three pattern as a single arc rather than a refactor-first commit. We keep the count at nine, treating the hashmap addition as part of the same arc as its first three users. The psABI conformance count grows by one for `-fpic`/`-fPIC` (the GOT and `__tls_get_addr` sequences are the standard PIC forms). New count: nineteen.

Through Chapter 22 chibicc compiles real-world C code. The driver knows the linker vocabulary, the preprocessor caches its work, and the dependency-file emission lets `make`-based build systems stay in sync. The thread-local, alloca, VLA, long-double, and labels-as-values machinery from Chapter 21 plus this chapter's hashmap-driven performance and linker-driver flags together cover the gap between "tutorial compiler" and "compiler that can build a Linux distribution's C packages." What remains is the long tail: atomics, the `_Generic` polish, and the final cleanups. Ten more commits, in Chapter 23.
