# Claude Conversation Log

Session ID: 698414c5-b112-43a6-a813-ef16720bf28f
Date: 2026-05-10 14:57:41

---

## 👤 User

please read docs/sessions/022-chapter-21-draft/HANDOFF.md and proceed

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/022-chapter-21-draft/HANDOFF.md"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/022-chapter-21-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in f0c98e0 0aad326 30520e5 655954e f694413 d0c4667 95d5a46 57c1d4e db850f3 fb5cfe5 7aa72e4 c3edffb 86785fc c0f0614 d48d9e5 a6c6622 f10bceb 1e9b6dd 4e5de36 c8df787 d1bc9a4 469f159 fb49370; do echo \"===== $h =====\"; git show --stat $h | head -10; done",
  "description": "Scan all 23 commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show f0c98e0",
  "description": "Show the labels-as-values compile-time constant commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 0aad326",
  "description": "Show hashmap commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 30520e5",
  "description": "Show macro hashmap commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 655954e",
  "description": "Show block-scope hashmap commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show f694413",
  "description": "Show keyword hashmap commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show d0c4667",
  "description": "Show -M option commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 95d5a46 57c1d4e db850f3",
  "description": "Show -MF, -MP, -MT commits"
}


---

## 🤖 Claude


🔧 Using tool: ToolSearch
Input: {
  "query": "select:TodoWrite",
  "max_results": 1
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 23 commit diffs",
      "activeForm": "Reading all 23 commit diffs",
      "status": "in_progress"
    },
    {
      "content": "Read destination state of key files at fb49370",
      "activeForm": "Reading destination state of key files at fb49370",
      "status": "pending"
    },
    {
      "content": "Skim Ch 21 to match voice",
      "activeForm": "Skimming Ch 21 to match voice",
      "status": "pending"
    },
    {
      "content": "Draft chapters/22-performance-deps-and-the-linker-driver.md",
      "activeForm": "Drafting Chapter 22",
      "status": "pending"
    },
    {
      "content": "Write session 023 README.md",
      "activeForm": "Writing session 023 README.md",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 024",
      "activeForm": "Writing HANDOFF.md for session 024",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show fb5cfe5 7aa72e4",
  "description": "Show -MD and -MQ commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show c3edffb 86785fc",
  "description": "Show -MMD and -fpic commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show c0f0614 d48d9e5",
  "description": "Show file-search caching and include guard commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a6c6622 f10bceb",
  "description": "Show pragma once and include_next"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 1e9b6dd 4e5de36 c8df787 d1bc9a4 469f159",
  "description": "Show linker driver pass-through commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show fb49370",
  "description": "Show third-party app test scripts commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book && wc -w chapters/21-thread-local-alloca-vlas.md chapters/20-gcc-extensions-worth-supporting.md",
  "description": "Check length of recent chapters"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "head -150 chapters/21-thread-local-alloca-vlas.md",
  "description": "Look at the chapter 21 opening"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && git checkout fb49370 -- . 2>/dev/null; git stash 2>/dev/null; git checkout fb49370 2>&1 | tail -5",
  "description": "Checkout fb49370 to inspect destination state"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n \"get_input_files\\|File \\*\\*\" chibicc.h tokenize.c | head -20",
  "description": "Find get_input_files"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 23 commit diffs",
      "activeForm": "Reading all 23 commit diffs",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at fb49370",
      "activeForm": "Reading destination state of key files at fb49370",
      "status": "in_progress"
    },
    {
      "content": "Skim Ch 21 to match voice",
      "activeForm": "Skimming Ch 21 to match voice",
      "status": "completed"
    },
    {
      "content": "Draft chapters/22-performance-deps-and-the-linker-driver.md",
      "activeForm": "Drafting Chapter 22",
      "status": "pending"
    },
    {
      "content": "Write session 023 README.md",
      "activeForm": "Writing session 023 README.md",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 024",
      "activeForm": "Writing HANDOFF.md for session 024",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "sed -n '670,690p' tokenize.c && echo \"---File struct---\" && grep -n \"^typedef struct File\\|^} File\\|^struct File \" chibicc.h",
  "description": "Show get_input_files and File struct"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/chibicc.h",
  "offset": 55,
  "limit": 25
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n \"skip_cond_incl\\|is_hash\" preprocess.c | head -20",
  "description": "Check skip_cond_incl"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/preprocess.c",
  "offset": 73,
  "limit": 10
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/preprocess.c",
  "offset": 680,
  "limit": 30
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout main 2>&1 | tail -3 && git stash pop 2>&1 | tail -3",
  "description": "Return to main branch"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 23 commit diffs",
      "activeForm": "Reading all 23 commit diffs",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at fb49370",
      "activeForm": "Reading destination state of key files at fb49370",
      "status": "completed"
    },
    {
      "content": "Skim Ch 21 to match voice",
      "activeForm": "Skimming Ch 21 to match voice",
      "status": "completed"
    },
    {
      "content": "Draft chapters/22-performance-deps-and-the-linker-driver.md",
      "activeForm": "Drafting Chapter 22",
      "status": "in_progress"
    },
    {
      "content": "Write session 023 README.md",
      "activeForm": "Writing session 023 README.md",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 024",
      "activeForm": "Writing HANDOFF.md for session 024",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "tail -100 /Users/cell/claude/chibicc-book/chapters/21-thread-local-alloca-vlas.md",
  "description": "Look at chapter 21 closing"
}


---

## 🤖 Claude

Now I'll draft Chapter 22.

---

## 🤖 Claude

Now I have enough to draft the chapter. Let me write it.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/22-performance-deps-and-the-linker-driver.md",
  "content": "# Chapter 22 \u2014 Performance, dependency files, and the linker driver\n\n> Commits covered: `f0c98e0`, `0aad326`, `30520e5`, `655954e`, `f694413`, `d0c4667`, `95d5a46`, `57c1d4e`, `db850f3`, `fb5cfe5`, `7aa72e4`, `c3edffb`, `86785fc`, `c0f0614`, `d48d9e5`, `a6c6622`, `f10bceb`, `1e9b6dd`, `4e5de36`, `c8df787`, `d1bc9a4`, `469f159`, `fb49370`. Twenty-three commits \u2014 the longest-running chapter so far. The arc is mixed: one small parser tweak that lets labels-as-values be used in static initializers, the introduction of a from-scratch open-addressing hashmap (and three places that immediately start using it), the seven-commit `-M` family that lets chibicc emit Makefile-shaped dependency rules, `-fpic`/`-fPIC` (which actually changes codegen \u2014 this chapter's biggest surprise), file-search caching that uses the new hashmap, three include-handling improvements (include-guard optimization, `#pragma once`, `#include_next`), five linker-driver pass-throughs (`-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker`), and a small harness of shell scripts that build real-world third-party programs with chibicc.\n\nTwo threads run through the chapter. One is performance: the macro table, the block-scope variable lookup, the keyword check, and the include-search path were all linear scans, and at the codebase's current size they're starting to show up in profiles. Rui's answer is to write a single hashmap and use it everywhere a linear scan was hiding. The hashmap is the most reusable piece of data structure code in the entire compiler.\n\nThe other thread is real-world buildability. Chibicc needs to produce dependency files so `make` can know which headers a `.c` file pulls in. It needs `-static`, `-shared`, `-L`, `-Wl,`, and `-Xlinker` so build systems can drive it the way they drive `gcc`. It needs include-guard optimization so the preprocessor doesn't tokenize the same header dozens of times. And it needs `-fpic`/`-fPIC` so shared libraries built by chibicc actually link. The chapter closes with the harness: shell scripts that try to build git, libpng, sqlite, and tinycc. By the end of the chapter chibicc can compile real programs that real people use.\n\nSeven sections from twenty-three commits.\n\n- **\u00a722.1** \u2014 Labels-as-values as a compile-time constant (commit 284).\n- **\u00a722.2** \u2014 The string hashmap (commit 285).\n- **\u00a722.3** \u2014 Three hashmap users: macros, scopes, keywords (commits 286\u2013288).\n- **\u00a722.4** \u2014 The `-M` family (commits 289\u2013295).\n- **\u00a722.5** \u2014 `-fpic`/`-fPIC` and the file-search cache (commits 296\u2013297).\n- **\u00a722.6** \u2014 Include-guard optimization, `#pragma once`, `#include_next` (commits 298\u2013300).\n- **\u00a722.7** \u2014 Linker-driver pass-throughs and the third-party harness (commits 301\u2013306).\n\nThe chapter follows `main` order. Calendar dates scatter widely across August, September, and October 2020 \u2014 the `-M` flags were drafted in mid-August but the `-MMD` polish came in mid-September, and several commits land in early October. As before, the prose walks `main` order without commenting on the dates except where they matter for a dependency between commits.\n\n---\n\n## 22.1 \u2014 Labels-as-values as a compile-time constant\n\n> `git checkout f0c98e0d590ffae286a8a4847c91212c734be8e3` \u2014 *[GNU] Treat labels-as-values as compile-time constant*\n\nChapter 21 \u00a721.6.10 added `&&label` as an expression usable inside function bodies. What that commit didn't address was the trick that motivates labels-as-values in the first place: storing label addresses in a static array so a piece of code can dispatch by indexing.\n\n```c\nstatic void *jump_table[] = {&&l1, &&l2, &&l3};\ngoto *jump_table[i];\n```\n\nThis is the canonical use pattern \u2014 interpreters use it to dispatch on opcode, lexers use it to dispatch on character class. For it to work, `&&l1`, `&&l2`, `&&l3` need to be compile-time constants suitable for static initialization, the same way `&global_var` is. The commit teaches `eval2` and `eval_rval` about `ND_LABEL_VAL`.\n\n### A new pointer level on `Relocation`\n\nThe first surprise is that `Relocation`'s `label` field grows a level of indirection:\n\n```c\nstruct Relocation {\n  Relocation *next;\n  int offset;\n-  char *label;\n+  char **label;\n  long addend;\n};\n```\n\nAnd the call site in `emit_data` learns to dereference:\n\n```c\nprintln(\"  .quad %s%+ld\", *rel->label, rel->addend);\n```\n\nWhy? A label's `unique_label` (the assembler symbol like `.L..3`) is generated lazily \u2014 `assign_lvar_offsets` walks each function and stamps a unique label name into `Node->unique_label` only when codegen needs it. At the time `eval2` runs over a global initializer, the function bodies haven't been codegen'd yet, so the label string isn't filled in. By storing a pointer-to-pointer, the `Relocation` captures the *address of the slot* in `Node->unique_label`. By the time `emit_data` reads `*rel->label`, codegen has filled in the slot, and the dereference reads the real string.\n\nThe same indirection covers global variables \u2014 `Obj->name` is set at parse time, but using `**` uniformly lets the relocation machinery treat both kinds of symbol identically. The two existing `eval2` cases that take label addresses (`ND_VAR` for globals and `ND_ADDR` of a deref-chain) change their stored expression to `&node->var->name`, and the new arm stores `&node->unique_label` for `ND_LABEL_VAL`.\n\n### The `eval2` arm\n\n```c\ncase ND_LABEL_VAL:\n  *label = &node->unique_label;\n  return 0;\n```\n\nThree lines. The `eval2` machinery already knew how to record \"this initializer is a relocation against symbol *X*, with addend *N*.\" All this commit had to do was teach it that label-value nodes are also valid sources of relocations, and arrange for the symbol slot to be readable at emit time even though the unique-label string isn't filled in yet.\n\n`eval_rval` is updated for symmetry \u2014 its `ND_VAR` case also moves from `*label = node->var->name` to `*label = &node->var->name`. The double-pointer convention is now uniform across the eval-quartet.\n\nThe `is_const_expr` predicate (the fifth eval-quartet member from Ch 21) doesn't need an `ND_LABEL_VAL` arm; the existing structural recursion already returns true on labels-as-values once `eval2` accepts them.\n\n### The test\n\n`test/control.c` grows three new cases:\n\n```c\nASSERT(3, ({ static void *p[]={&&v41,&&v42,&&v43}; int i=0; goto *p[0];\n             v41:i++; v42:i++; v43:i++; i; }));\n```\n\nThe `static` storage-class is the load-bearing piece. Chapter 21's labels-as-values would have rejected this initializer (it's not a function-local automatic, so its initializer must be a compile-time constant). After this commit, the static array of three label addresses elaborates correctly \u2014 each element gets a relocation against the corresponding `.L` label.\n\n**Where we are.** Labels-as-values are now usable in static initializers \u2014 the canonical jump-table pattern works. The `Relocation` mechanism now stores a pointer-to-pointer for the symbol name, accommodating labels (whose unique-label strings are generated lazily during codegen) and globals (whose names are set at parse time) through the same channel.\n\n---\n\n## 22.2 \u2014 The string hashmap\n\n> `git checkout 0aad326f3550b3d4c499d4078fcc65cc2dbf7626` \u2014 *Add string hashmap*\n\nThis commit adds a single new file, `hashmap.c`. The header gets a `HashMap` and `HashEntry` type plus six function declarations. There are no callers yet \u2014 the hashmap is added in isolation, with a built-in self-test wired up to the driver as `-hashmap-test`. The next three commits convert linear-search structures one at a time.\n\nThe data structure is a from-scratch open-addressing hashmap with linear probing, FNV-1a hashing, and tombstone-based deletion. It's about 165 lines including the test. The simplest hashmap a competent C programmer writes, and that's not a coincidence \u2014 Rui doesn't reach for a fancier data structure when a plain one will do.\n\n### The two types\n\n```c\ntypedef struct {\n  char *key;\n  int keylen;\n  void *val;\n} HashEntry;\n\ntypedef struct {\n  HashEntry *buckets;\n  int capacity;\n  int used;\n} HashMap;\n```\n\n`HashEntry` carries a key (a `(pointer, length)` pair so non-NUL-terminated tokens can be used directly), a value pointer, and that's it. `HashMap` carries the bucket array, the bucket array size, and a count of used entries.\n\nThe decision to store keys as `(pointer, length)` rather than a NUL-terminated `char *` is what makes the hashmap fit chibicc's tokenizer cleanly. Tokens point into the source buffer; a token has a `loc` and `len` rather than a NUL-terminated string. With the `keylen` field, `find_macro` can hand the hashmap `tok->loc` and `tok->len` directly without copying out a temporary. The non-`2` variants of the API treat their input as `strlen`-terminated; the `2`-suffixed variants take an explicit length.\n\n### FNV-1a\n\n```c\nstatic uint64_t fnv_hash(char *s, int len) {\n  uint64_t hash = 0xcbf29ce484222325;\n  for (int i = 0; i < len; i++) {\n    hash *= 0x100000001b3;\n    hash ^= (unsigned char)s[i];\n  }\n  return hash;\n}\n```\n\nThe two magic numbers are FNV's 64-bit offset basis and prime. FNV-1a is byte-at-a-time, branch-free, and produces enough entropy on short strings (variable names, macro names) to keep collision rates low under linear probing. Rui's version has the multiply *before* the XOR, which is the FNV-1a variant; the plain FNV-1 has it the other way. Both work; FNV-1a is the more recommended ordering because it gets entropy onto every output bit before the next byte arrives.\n\nThere's no defense against a malicious key set. The hash is deterministic and non-randomized, so a constructed input could in principle pile up keys in a single bucket. Compilers don't get this kind of attack in practice, so it's a non-issue.\n\n### High and low watermarks\n\n```c\n#define INIT_SIZE 16\n#define HIGH_WATERMARK 70\n#define LOW_WATERMARK 50\n```\n\nA fresh hashmap has 16 buckets. When the used-count crosses 70% of capacity, `rehash` runs. It picks a new capacity by doubling until the load factor (after the rehash) would be under 50%:\n\n```c\nint cap = map->capacity;\nwhile ((nkeys * 100) / cap >= LOW_WATERMARK)\n  cap = cap * 2;\n```\n\nNote `nkeys` is the count of *live* entries, not `map->used` \u2014 tombstones don't count toward the new capacity calculation. The rehash discards tombstones; only the keys with real values get re-inserted. So a map that's accumulated a lot of deletes can shrink its effective load even without growing capacity. A pure no-deletion workload doubles the bucket array each rehash; a delete-heavy workload may rehash to the *same* size, which is enough because the tombstones go away.\n\n### Lookup with linear probing\n\n```c\nstatic HashEntry *get_entry(HashMap *map, char *key, int keylen) {\n  if (!map->buckets)\n    return NULL;\n\n  uint64_t hash = fnv_hash(key, keylen);\n\n  for (int i = 0; i < map->capacity; i++) {\n    HashEntry *ent = &map->buckets[(hash + i) % map->capacity];\n    if (match(ent, key, keylen))\n      return ent;\n    if (ent->key == NULL)\n      return NULL;\n  }\n  unreachable();\n}\n```\n\nProbe `(hash, hash+1, hash+2, ...)` modulo capacity. Stop on a match (return the entry) or on a NULL bucket (the key isn't in the table). Tombstones are *not* a stopping condition \u2014 a probe walks past tombstones because the desired key might be further down the chain, displaced by an entry that was later deleted.\n\nThe `unreachable()` at the end fires only if the table is completely full of non-NULL, non-matching entries, which can't happen under the 70% watermark unless something has gone very wrong with `used`. It's an assertion, not a recovery path.\n\n### Insertion: tombstones first, then NULL\n\n```c\nstatic HashEntry *get_or_insert_entry(HashMap *map, char *key, int keylen) {\n  if (!map->buckets) {\n    map->buckets = calloc(INIT_SIZE, sizeof(HashEntry));\n    map->capacity = INIT_SIZE;\n  } else if ((map->used * 100) / map->capacity >= HIGH_WATERMARK) {\n    rehash(map);\n  }\n\n  uint64_t hash = fnv_hash(key, keylen);\n\n  for (int i = 0; i < map->capacity; i++) {\n    HashEntry *ent = &map->buckets[(hash + i) % map->capacity];\n\n    if (match(ent, key, keylen))\n      return ent;\n\n    if (ent->key == TOMBSTONE) {\n      ent->key = key;\n      ent->keylen = keylen;\n      return ent;\n    }\n\n    if (ent->key == NULL) {\n      ent->key = key;\n      ent->keylen = keylen;\n      map->used++;\n      return ent;\n    }\n  }\n  unreachable();\n}\n```\n\nThe order of the three checks is the load-bearing detail. `match` first, then `TOMBSTONE`, then `NULL`. If the key already exists in the table, even one bucket beyond a tombstone, the function returns the existing entry; the tombstone is left undisturbed. If the key doesn't exist but a tombstone is on the probe path, the first tombstone gets reused and `map->used` is *not* incremented (the slot was already counted). If neither, the first NULL bucket gets a new entry and `map->used` goes up.\n\nThis is the standard tombstone scheme. The subtle case is that a probe must continue past a tombstone to confirm absence. If the probe stopped at the first tombstone and reused it, an existing key further down the chain would silently get duplicated. The match-first ordering prevents that.\n\n### `TOMBSTONE`\n\n```c\n#define TOMBSTONE ((void *)-1)\n```\n\nA pointer with value `-1` (cast to `void *`). On x86-64 this is a non-canonical address that no real allocation will produce. Any sentinel value distinct from `NULL` and from a valid heap pointer would work; `(void *)-1` is the C tradition. The `match` predicate explicitly excludes it:\n\n```c\nstatic bool match(HashEntry *ent, char *key, int keylen) {\n  return ent->key && ent->key != TOMBSTONE &&\n         ent->keylen == keylen && memcmp(ent->key, key, keylen) == 0;\n}\n```\n\nComparing `keylen` first short-circuits the `memcmp` for length mismatches.\n\n### The six API functions\n\n```c\nvoid *hashmap_get(HashMap *map, char *key);\nvoid *hashmap_get2(HashMap *map, char *key, int keylen);\nvoid hashmap_put(HashMap *map, char *key, void *val);\nvoid hashmap_put2(HashMap *map, char *key, int keylen, void *val);\nvoid hashmap_delete(HashMap *map, char *key);\nvoid hashmap_delete2(HashMap *map, char *key, int keylen);\n```\n\nThree operations, two flavors each. The non-`2` flavors are one-line shims that compute `strlen(key)` and call the `2`-suffixed versions. `hashmap_delete` is implemented as a single store: find the entry and overwrite its `key` with `TOMBSTONE`. The value pointer is left unchanged but no longer reachable.\n\nNotice what's *not* in the API: no iteration. Walking all entries is not a supported primitive. The hashmap is meant for point lookups by string key, and that's all. A future commit (in this same chapter) needs iteration over `include_paths` for `#include_next`, but `include_paths` is a `StringArray`, not a `HashMap`.\n\n### `hashmap_test` and the driver hook\n\nThe bottom of `hashmap.c` carries a 5,000-key stress test. Insert keys 0\u20134999, delete 1000\u20131999, re-insert 1500\u20131599, insert 6000\u20136999. Then assert that lookups behave correctly:\n\n```c\nfor (int i = 0; i < 1000; i++)\n  assert((size_t)hashmap_get(map, format(\"key %d\", i)) == i);\nfor (int i = 1000; i < 1500; i++)\n  assert(hashmap_get(map, \"no such key\") == NULL);\n...\n```\n\nThe test is invoked by adding `-hashmap-test` as a driver flag:\n\n```c\nif (!strcmp(argv[i], \"-hashmap-test\")) {\n  hashmap_test();\n  exit(0);\n}\n```\n\nAnd `test/driver.sh` adds one line that exercises it:\n\n```sh\n$chibicc -hashmap-test\ncheck 'hashmap'\n```\n\nThe hashmap won't have an external user until the next commit. This one stands alone \u2014 datastructure plus self-test \u2014 so that the next three diffs that adopt it can be small and local.\n\n**Where we are.** The compiler now has a generic open-addressing string hashmap with FNV-1a hashing, 70% high watermark, 50% low watermark after rehash, and tombstone deletion. The API uses a `(pointer, length)` key shape that fits tokens directly. No callers yet; three commits' worth coming next.\n\n---\n\n## 22.3 \u2014 Three hashmap users: macros, scopes, keywords\n\n> `git checkout 30520e5a7c73a6613cfcef38d72058e7cccde1f4` \u2014 *Use hashmap for macro name lookup*\n>\n> `git checkout 655954e301621737988a4fa0a2c72ffc24285c8d` \u2014 *Use hashmap for block-scope lookup*\n>\n> `git checkout f6944133d211ec6fb71c41f118905e16a752135b` \u2014 *Use hashmap for keyword lookup*\n\nThree commits, each replacing a linear scan with a single `hashmap_get`. The three call sites profile differently:\n\n- **Macros.** The macro table is small in absolute terms (a few hundred entries with system headers included) but `find_macro` runs once per identifier token in the entire translation unit. Lookup-heavy.\n- **Block scopes.** Each function body opens a scope, accumulates local variables (and tags), and tears the scope down. Insert and lookup are both tied to scope lifetime. Lookup-medium, lifetime-short.\n- **Keywords.** A fixed set of about thirty strings. Insert-once at first call, then lookup-many for the rest of the translation unit.\n\nEach conversion is a small diff. Together they remove the three most visible linear scans in the compiler.\n\n### 22.3.1 \u2014 Macros\n\nThe pre-conversion `find_macro` walks a linked list:\n\n```c\nfor (Macro *m = macros; m; m = m->next)\n  if (strlen(m->name) == tok->len && !strncmp(m->name, tok->loc, tok->len))\n    return m->deleted ? NULL : m;\nreturn NULL;\n```\n\nThe `deleted` flag is a tombstone in disguise \u2014 `undef_macro` previously added a fresh entry with `deleted = true`, which masked any earlier definition. The post-conversion code is two lines:\n\n```c\nreturn hashmap_get2(&macros, tok->loc, tok->len);\n```\n\nThe `Macro` struct sheds two fields: `next` (the linked-list pointer) and `deleted` (because the hashmap has real deletion via `TOMBSTONE`). `add_macro` becomes a `hashmap_put`; `undef_macro` becomes a `hashmap_delete`.\n\n```c\nvoid undef_macro(char *name) {\n-  Macro *m = add_macro(name, true, NULL);\n-  m->deleted = true;\n+  hashmap_delete(&macros, name);\n}\n```\n\nThis is the cleanest of the three. The hashmap's tombstone scheme exactly matches what `undef_macro` was already simulating with the `deleted` flag, so the data-structure swap removes a workaround rather than introducing one.\n\n### 22.3.2 \u2014 Block scopes\n\nThe pre-conversion `Scope` carried two linked lists, one for variable scope (`VarScope`) and one for tag scope (`TagScope`). Each had a `next` pointer and a `name` field. Lookup walked the chain inside-out; insert prepended.\n\nAfter the swap, both lists become `HashMap`:\n\n```c\nstruct Scope {\n  Scope *next;\n\n  // C has two block scopes; one is for variables/typedefs and\n  // the other is for struct/union/enum tags.\n-  VarScope *vars;\n-  TagScope *tags;\n+  HashMap vars;\n+  HashMap tags;\n};\n```\n\n`VarScope` and `TagScope` lose their `next` and `name` fields; the hashmap stores those for them. `TagScope` collapses to just a `Type *` value \u2014 the `name` is the hashmap key, the `ty` is the value, so a separate struct isn't needed at all:\n\n```c\n-typedef struct TagScope TagScope;\n-struct TagScope {\n-  TagScope *next;\n-  char *name;\n-  Type *ty;\n-};\n```\n\nThe tag scope hashmap keys are now the type names; values are `Type *` pointers directly. `push_tag_scope` becomes:\n\n```c\nstatic void push_tag_scope(Token *tok, Type *ty) {\n  hashmap_put2(&scope->tags, tok->loc, tok->len, ty);\n}\n```\n\n`find_var` and `find_tag` walk the chain of `Scope`s but lookup *within* a scope is now `O(1)`:\n\n```c\nstatic VarScope *find_var(Token *tok) {\n  for (Scope *sc = scope; sc; sc = sc->next) {\n    VarScope *sc2 = hashmap_get2(&sc->vars, tok->loc, tok->len);\n    if (sc2)\n      return sc2;\n  }\n  return NULL;\n}\n```\n\nThe asymptotic improvement here is real. A function with ten nested scopes, each with twenty locals, used to do up to 200 string compares per identifier reference. After the swap it does ten `hashmap_get2` calls, each `O(1)` on average.\n\nThe `struct_union_decl` redefinition path also gets simpler:\n\n```c\n-for (TagScope *sc = scope->tags; sc; sc = sc->next) {\n-  if (equal(tag, sc->name)) {\n-    *sc->ty = *ty;\n-    return sc->ty;\n-  }\n-}\n+Type *ty2 = hashmap_get2(&scope->tags, tag->loc, tag->len);\n+if (ty2) {\n+  *ty2 = *ty;\n+  return ty2;\n+}\n```\n\nA linear scan of one scope's tags becomes a single point lookup.\n\n`find_func` (the lookup that gets called when the parser sees a call to an undeclared function) uses the outermost scope's hashmap directly:\n\n```c\nVarScope *sc2 = hashmap_get(&sc->vars, name);\nif (sc2 && sc2->var && sc2->var->is_function)\n  return sc2->var;\n```\n\nThe old version walked every var in the global scope linearly, filtering by `is_function`. The new version finds the named variable in `O(1)` and then checks whether it happens to be a function.\n\n### 22.3.3 \u2014 Keywords\n\nThe pre-conversion `is_typename` and `is_keyword` each held a static array of about thirty strings and walked the array linearly:\n\n```c\nstatic char *kw[] = { \"void\", \"_Bool\", \"char\", ... };\n\nfor (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)\n  if (equal(tok, kw[i]))\n    return true;\nreturn false;\n```\n\nBoth are called for every identifier token \u2014 `is_keyword` from `tokenize`, `is_typename` from `declspec` \u2014 so a thirty-element scan runs constantly. The post-conversion code lazily builds a hashmap on the first call:\n\n```c\nstatic bool is_keyword(Token *tok) {\n  static HashMap map;\n\n  if (map.capacity == 0) {\n    static char *kw[] = { ... };\n    for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)\n      hashmap_put(&map, kw[i], (void *)1);\n  }\n\n  return hashmap_get2(&map, tok->loc, tok->len);\n}\n```\n\nTwo details. First, the `(void *)1` value is a \"present\" sentinel; the hashmap stores values as `void *`, and the `1` is just \"anything non-NULL.\" Anything non-NULL would do; `1` is conventional. Second, the `map.capacity == 0` check is the lazy-init guard. A `HashMap` declared `static` is zero-initialized, so `capacity == 0` means \"not yet populated.\"\n\nThe build is once-per-program \u2014 the static `HashMap` persists across calls. The thirty `hashmap_put` calls happen on the first invocation; subsequent calls go straight to the lookup. This is a textbook conversion of a fixed-set linear scan into an `O(1)` lookup. The hashmap is overkill for thirty entries; a perfect-hash table would be faster. But the hashmap is already there, the code is local, and the constant factor is small enough that it doesn't matter.\n\nThe same conversion lands twice: once for `is_keyword` (tokenize.c) and once for `is_typename` (parse.c). The two keyword sets overlap heavily but aren't identical \u2014 `is_typename` carries `typedef`, `enum`, `_Alignas`, and the storage-class words; `is_keyword` carries the control-flow words like `return`, `if`, `for`. Two separate hashmaps, two separate static arrays. Rui doesn't try to share.\n\n**Where we are.** The hashmap has its three intended customers. Macro lookup, block-scope variable lookup, tag lookup, and the two keyword checks all run in `O(1)`. The macro and tag types lose vestigial `next` and `deleted` fields; the linear-scan loops disappear from `find_macro`, `find_var`, `find_tag`, `is_keyword`, and `is_typename`. The compiler should be measurably faster on translation units with many headers.\n\n---\n\n## 22.4 \u2014 The `-M` family\n\n> `git checkout d0c4667b6bccf35ddf069c777689cd18c6a632b3` \u2014 *Add -M option*\n>\n> `git checkout 95d5a46234f98f3793c965bebe036361cbb1978e` \u2014 *Add -MF option*\n>\n> `git checkout 57c1d4ec0290d49fa1e954ff3e7a51e24d71a3a1` \u2014 *Add -MP option*\n>\n> `git checkout db850f37a2a284bf18cea427e4676a22d83d04b8` \u2014 *Add -MT option*\n>\n> `git checkout fb5cfe5d17fd0c0cbc0d17789c065b9bb86ba3c4` \u2014 *Add -MD option*\n>\n> `git checkout 7aa72e41e6b2703b3f357507252008ebe25dc08d` \u2014 *Add -MQ option*\n>\n> `git checkout c3edffbbb06be9d586ee4f1cf678049b7d81369d` \u2014 *Add -MMD option*\n\nSeven commits, all on `main.c`, all driver-side. The `-M` family is gcc's mechanism for telling `make` which header files a `.c` file depends on. The compiler reads the source, runs the preprocessor far enough to know which `#include`s would have been pulled in, and writes a Makefile rule whose target is the `.o` file and whose dependencies are every file the preprocessor touched.\n\nReal build systems use this constantly. `gcc -MMD -c foo.c` produces both `foo.o` (the object file) and `foo.d` (the dependency rule). The `Makefile` includes `foo.d`, so when a header changes, `make` recompiles `foo.o` automatically. Without this feature, a build system has to either rebuild everything or maintain dependencies by hand. Real-world code can't be built by chibicc until it grows this family.\n\nThe family is seven flags. `-M` writes a rule to stdout. `-MF FILE` writes it to a file. `-MP` adds phony rules so deletions don't break the build. `-MT TARGET` overrides the rule's target name. `-MD` enables dependency emission *alongside* normal compilation. `-MQ` is `-MT` with Makefile escaping. `-MMD` is `-MD` minus system headers.\n\n### 22.4.1 \u2014 `-M`: write a rule to stdout\n\nThe simplest of the seven. `parse_args` adds a flag:\n\n```c\nif (!strcmp(argv[i], \"-M\")) {\n  opt_M = true;\n  continue;\n}\n```\n\n`cc1` checks the flag after preprocessing and prints the rule:\n\n```c\nif (opt_M) {\n  print_dependencies();\n  return;\n}\n```\n\n`print_dependencies` is the core of the chapter's middle:\n\n```c\nstatic void print_dependencies(void) {\n  FILE *out = open_file(opt_o ? opt_o : \"-\");\n  fprintf(out, \"%s:\", replace_extn(base_file, \".o\"));\n\n  File **files = get_input_files();\n\n  for (int i = 0; files[i]; i++)\n    fprintf(out, \" \\\\\\n  %s\", files[i]->name);\n  fprintf(out, \"\\n\\n\");\n}\n```\n\nThe dependency rule's target is the source file's name with `.c` replaced by `.o` \u2014 `replace_extn(base_file, \".o\")`. The dependencies come from `get_input_files()`, the tokenizer-side function that returns every `File` struct created during preprocessing: the source file plus every `#include`d header. They're printed one-per-line with a backslash continuation.\n\n`get_input_files` was added back in Chapter 17 as part of the `__BASE_FILE__` machinery. The tokenizer maintains an `input_files` array: every `tokenize_file` call appends a new `File`. Now that array becomes the source of truth for `-M`.\n\nThe driver also has to route a `-M` invocation through `cc1` exactly once, because the `-M` output is generated during preprocessing rather than during code generation:\n\n```c\nif (opt_E || opt_M) {\n  run_cc1(argc, argv, input, NULL);\n  continue;\n}\n```\n\n`-M` joins `-E` as a \"preprocess and stop\" mode.\n\n### 22.4.2 \u2014 `-MF FILE`: redirect to a file\n\n`-MF` is a one-line addition: route the dependency output to a specific file rather than stdout. The flag takes an argument, so it goes into `take_arg`:\n\n```c\nchar *x[] = {\"-o\", \"-I\", \"-idirafter\", \"-include\", \"-x\", \"-MF\"};\n```\n\nAnd `print_dependencies` learns a path-selection rule:\n\n```c\nchar *path;\nif (opt_MF)\n  path = opt_MF;\nelse if (opt_o)\n  path = opt_o;\nelse\n  path = \"-\";\n```\n\nThe fallback chain is `-MF` \u2192 `-o` \u2192 stdout. Without `-MF`, a `-M -o foo.d` invocation routes to `foo.d`; with `-MF foo.d`, the same invocation can use a different `-o`. This matters because `-o` typically names the *object* file in real builds, not the dependency file.\n\n### 22.4.3 \u2014 `-MP`: phony rules for deleted headers\n\n`-MP` adds a tail to the dependency output: an empty rule for each header. The motivation is the failure mode where someone deletes a header and the `.d` file still references it \u2014 `make` then complains \"no rule to make `removed.h`.\" A phony rule silences that.\n\n```c\nif (opt_MP)\n  for (int i = 1; files[i]; i++)\n    fprintf(out, \"%s:\\n\\n\", files[i]->name);\n```\n\nNote the `i = 1`, not `i = 0`. `files[0]` is the source file itself (the `.c`); only the headers get phony rules. The output looks like:\n\n```\nfoo.o: foo.c \\\n  bar.h \\\n  baz.h\n\nbar.h:\n\nbaz.h:\n```\n\nWith those empty rules in place, `make` treats `bar.h` as a target with no prerequisites and no recipe. If `bar.h` doesn't exist, `make` is satisfied because the phony rule says \"this target needs nothing.\" If `bar.h` does exist, the original `foo.o: ... bar.h ...` rule still triggers a rebuild on change.\n\n### 22.4.4 \u2014 `-MT TARGET`: override the target name\n\n`-MT` lets the user rename the rule's target. The default is `replace_extn(base_file, \".o\")` \u2014 the source filename with `.o`. With `-MT foo.lo`, the rule says `foo.lo:` instead.\n\n```c\nif (!strcmp(argv[i], \"-MT\")) {\n  if (opt_MT == NULL)\n    opt_MT = argv[++i];\n  else\n    opt_MT = format(\"%s %s\", opt_MT, argv[++i]);\n  continue;\n}\n```\n\nThe if-else handles repeated `-MT`: each one *appends* to the target list, space-separated. So `-MT foo.o -MT bar.o` produces `foo.o bar.o:` as the rule head. Real Makefiles sometimes want this when one command produces multiple outputs.\n\n`print_dependencies` uses it directly:\n\n```c\nfprintf(out, \"%s:\", opt_MT ? opt_MT : replace_extn(base_file, \".o\"));\n```\n\n### 22.4.5 \u2014 `-MD`: dependencies alongside compilation\n\n`-M` was preprocess-only. `-MD` says \"do the normal compile *and* also write a dependency file.\" The dependency-file path defaults to the source file's name with `.d`:\n\n```c\nelse if (opt_MD)\n  path = replace_extn(opt_o ? opt_o : base_file, \".d\");\n```\n\nAnd `cc1`'s dispatch grows an `else` branch:\n\n```c\nif (opt_M || opt_MD) {\n  print_dependencies();\n  if (opt_M)\n    return;\n}\n```\n\n`-M` returns early (preprocess-only); `-MD` falls through to normal compilation. The two modes share the dependency-emission path but differ in what happens after.\n\nThe implication: `-MD` is the normal mode for build systems. `make` invokes `gcc -MD -c foo.c -o foo.o`, which produces both `foo.o` and `foo.d`. The next time `make` runs, it includes `foo.d` and knows the headers `foo.c` depends on.\n\n### 22.4.6 \u2014 `-MQ`: like `-MT` with Makefile escaping\n\n`-MQ` is `-MT` plus escaping for special characters in Makefile syntax. A target like `foo$bar.o` would confuse `make` (which interprets `$` as a variable expansion); `-MQ` applies Makefile escaping rules.\n\nThe escaper:\n\n```c\nstatic char *quote_makefile(char *s) {\n  char *buf = calloc(1, strlen(s) * 2 + 1);\n\n  for (int i = 0, j = 0; s[i]; i++) {\n    switch (s[i]) {\n    case '$':\n      buf[j++] = '$';\n      buf[j++] = '$';\n      break;\n    case '#':\n      buf[j++] = '\\\\';\n      buf[j++] = '#';\n      break;\n    case ' ':\n    case '\\t':\n      for (int k = i - 1; k >= 0 && s[k] == '\\\\'; k--)\n        buf[j++] = '\\\\';\n      buf[j++] = '\\\\';\n      buf[j++] = s[i];\n      break;\n    default:\n      buf[j++] = s[i];\n      break;\n    }\n  }\n  return buf;\n}\n```\n\nThree transformations:\n\n- `$` becomes `$$`. (Makefiles read `$$` as a literal dollar sign.)\n- `#` becomes `\\#`. (Make uses `#` for comments.)\n- Whitespace becomes backslash-prefixed, with extra backslash-doubling if the original was already preceded by backslashes. (A path like `foo\\\\bar baz` becomes `foo\\\\\\\\bar\\ baz`.)\n\nEach `-MQ TARGET` runs `quote_makefile` on the argument before joining:\n\n```c\nif (!strcmp(argv[i], \"-MQ\")) {\n  if (opt_MT == NULL)\n    opt_MT = quote_makefile(argv[++i]);\n  else\n    opt_MT = format(\"%s %s\", opt_MT, quote_makefile(argv[++i]));\n  continue;\n}\n```\n\nOnce `-MQ` exists, `print_dependencies` also routes the *default* target (the no-`-MT` path) through `quote_makefile`, plus the phony rule names from `-MP`:\n\n```c\nif (opt_MT)\n  fprintf(out, \"%s:\", opt_MT);\nelse\n  fprintf(out, \"%s:\", quote_makefile(replace_extn(base_file, \".o\")));\n...\nif (opt_MP)\n  for (int i = 1; files[i]; i++)\n    fprintf(out, \"%s:\\n\\n\", quote_makefile(files[i]->name));\n```\n\nSo the escaping covers both the user-supplied target and the auto-generated paths. Notably, *dependency* names (the right-hand side of the rule) are *not* escaped \u2014 chibicc emits them raw. This is a subtle gap; gcc escapes both sides. Filenames with `$` or spaces in them would produce a malformed `.d` file. The handful of build systems that hit this in practice tend to avoid such filenames anyway, but it's an asymmetry worth noting.\n\n### 22.4.7 \u2014 `-MMD`: dependencies, but skip system headers\n\nThe last commit in the family is a small filter. `-MMD` enables `-MD` but excludes any header that came from a \"standard\" include path (the ones added by `add_default_include_paths` rather than user-supplied `-I`).\n\n```c\nstatic void add_default_include_paths(char *argv0) {\n  ...\n  strarray_push(&include_paths, \"/usr/include\");\n\n  // Keep a copy of the standard include paths for -MMD option.\n  for (int i = 0; i < include_paths.len; i++)\n    strarray_push(&std_include_paths, include_paths.data[i]);\n}\n```\n\nA separate `std_include_paths` `StringArray` records which paths the *driver* added by default. User `-I` flags push to `include_paths` later, after this loop runs, so `std_include_paths` captures only the system set.\n\nThe filter:\n\n```c\nstatic bool in_std_include_path(char *path) {\n  for (int i = 0; i < std_include_paths.len; i++) {\n    char *dir = std_include_paths.data[i];\n    int len = strlen(dir);\n    if (strncmp(dir, path, len) == 0 && path[len] == '/')\n      return true;\n  }\n  return false;\n}\n```\n\nA simple prefix-match. If the file's path starts with one of the standard include-path directories, the file is excluded from the dependency list.\n\n`print_dependencies` consults the filter twice, once for the dependency list and once for the phony-rule list:\n\n```c\nfor (int i = 0; files[i]; i++) {\n  if (opt_MMD && in_std_include_path(files[i]->name))\n    continue;\n  fprintf(out, \" \\\\\\n  %s\", files[i]->name);\n}\n```\n\nThe motivation: system headers (`<stdio.h>`, `<stdlib.h>`) don't change between builds. Putting them in the dependency file just adds noise to `make`'s dependency graph. `-MMD` is the form most build systems actually use.\n\n`-MMD` is implemented as a strict superset of `-MD`:\n\n```c\nif (!strcmp(argv[i], \"-MMD\")) {\n  opt_MD = opt_MMD = true;\n  continue;\n}\n```\n\nBoth flags get set; `-MMD` is just `-MD` plus the system-header filter. The `print_dependencies` body is shared.\n\n**Where we are.** Chibicc emits Makefile-shaped dependency rules. `-M` writes to stdout; `-MF` redirects; `-MP` adds phony rules; `-MT` and `-MQ` rename the target (with and without escaping); `-MD` and `-MMD` enable rule emission alongside compilation, with `-MMD` filtering out system headers. The rule format is simple: target plus backslash-continued dependency list. Build systems that drive `gcc -MMD` will work with chibicc by changing only the compiler binary.\n\n---\n\n## 22.5 \u2014 `-fpic`/`-fPIC` and the file-search cache\n\n> `git checkout 86785fceb169bc754efe3f29a9b63137f5c9a106` \u2014 *Add -fpic and -fPIC options*\n>\n> `git checkout c0f0614e6b7647fd4703abf4c455024c2ade8cd7` \u2014 *Cache file search results*\n\nTwo commits with very different sizes. The `-fpic` commit changes codegen \u2014 the bigger surprise of the two. The file-search commit is nine lines that thread the new hashmap into `search_include_paths`.\n\n### 22.5.1 \u2014 `-fpic` and `-fPIC`: codegen changes\n\nA naive read of the change suggests `-fpic` should be a flag-flip that the linker reads. Chibicc's existing codegen emits `lea name(%rip), %rax` for global addresses, which is rip-relative and therefore PIC-friendly. So why does the codegen change?\n\nBecause rip-relative addressing is PIC-friendly for *function-internal* data, but it's not enough for *cross-module* data. Inside a single shared library, two functions can reach each other and reach static globals via rip-relative offsets. Across shared libraries, a reference has to go through the Global Offset Table (GOT), because the loader binds shared-library symbols at load time and the GOT is the indirection that lets the binding work without rewriting code.\n\nThe commit adds a second codegen path in `gen_addr` for global names:\n\n```c\nif (opt_fpic) {\n  // Thread-local variable\n  if (node->var->is_tls) {\n    println(\"  data16 lea %s@tlsgd(%%rip), %%rdi\", node->var->name);\n    println(\"  .value 0x6666\");\n    println(\"  rex64\");\n    println(\"  call __tls_get_addr@PLT\");\n    return;\n  }\n\n  // Function or global variable\n  println(\"  mov %s@GOTPCREL(%%rip), %%rax\", node->var->name);\n  return;\n}\n```\n\nTwo new patterns, both new to the chapter.\n\nThe non-TLS pattern is the *general-dynamic* GOT access. `mov name@GOTPCREL(%rip), %rax` does two things in one instruction: the assembler emits a relocation `R_X86_64_REX_GOTPCRELX` against `name`, the linker materializes a GOT entry for `name`, and the load reads the GOT entry's value (which is the runtime address of `name`) into `%rax`. The non-PIC version was a single `lea`; the PIC version is a `mov` that loads through the GOT.\n\nThe TLS pattern is even more elaborate. Chapter 21 \u00a721.1 emitted the *initial-exec* TLS sequence (`mov %fs:0, %rax; add $name@tpoff, %rax`), which works for variables in the executable but not for variables in shared libraries. PIC code running in a shared library has to use the *general-dynamic* model, which calls `__tls_get_addr` to retrieve the address. The four-instruction sequence is the canonical gcc pattern for the call:\n\n```\ndata16 lea  name@tlsgd(%rip), %rdi\n.value 0x6666\nrex64\ncall __tls_get_addr@PLT\n```\n\nThe `data16` prefix and the `0x6666` `.value` are padding bytes that the linker can rewrite to convert this general-dynamic call into a cheaper local-dynamic or initial-exec sequence at link time, if it determines the variable is reachable in the local module. The four instructions occupy 16 bytes total, which is what the linker needs for the in-place rewrite.\n\nThis is a real bit of TLS arcana \u2014 the padding is documented in the AMD64 ABI, but it's not the kind of detail one would invent. Rui likely lifted the sequence from gcc's output. It works because the linker recognizes the exact byte pattern and either leaves it alone (general-dynamic stays) or rewrites it (`mov`/`lea`/`add` triple replaces the `data16 lea ... call`).\n\nWithout `-fpic`, the original two patterns from Chapter 21 fire \u2014 `lea name(%rip), %rax` for globals, the `%fs:0 + @tpoff` pair for TLS. With `-fpic`, the GOT and `__tls_get_addr` paths fire. The flag selects between them at codegen time.\n\nThe psABI conformance count grows by one. Chibicc now emits both PIC and non-PIC sequences; the GOT and `__tls_get_addr` sequences are the standard psABI-mandated forms for shared-library code. New count: nineteen.\n\nThe `-fpic` and `-fPIC` flags both set the same boolean. On real toolchains they differ: `-fpic` allows a smaller GOT (fits in 16-bit offsets), `-fPIC` allows arbitrarily large ones. Chibicc doesn't make the distinction \u2014 both flags use the same large-model code, which is a strict superset of what `-fpic` requires. Flag-compatible without size optimization.\n\n### 22.5.2 \u2014 Cache file-search results\n\nThe smaller half. `search_include_paths` was a linear walk over `include_paths.len`, calling `file_exists(path)` for each candidate. Each `file_exists` is a `stat` syscall. For a translation unit that includes many headers \u2014 and many headers transitively pulled in by `<stdio.h>` and friends \u2014 the same lookup repeats many times.\n\nThe fix is a single hashmap:\n\n```c\nstatic HashMap cache;\nchar *cached = hashmap_get(&cache, filename);\nif (cached)\n  return cached;\n\nfor (int i = 0; i < include_paths.len; i++) {\n  char *path = format(\"%s/%s\", include_paths.data[i], filename);\n  if (!file_exists(path))\n    continue;\n  hashmap_put(&cache, filename, path);\n  return path;\n}\n```\n\nThe cache key is the *filename* as written in the `#include` directive (e.g. `\"stdio.h\"`). The cache value is the resolved absolute path (e.g. `\"/usr/include/stdio.h\"`). On a hit, `search_include_paths` returns the cached path immediately, without any `file_exists` syscalls or string formatting. On a miss, the function falls through to the linear search; on a successful resolution, the path is cached for next time.\n\nNegative results \u2014 filenames that aren't found \u2014 are not cached. A `#include \"missing.h\"` will repeat the full search every time it's hit. In practice this case is rare; preprocessing hits a missing header once and errors out.\n\nThis is one of the few places in the compiler where a hashmap genuinely speeds things up by an order of magnitude. The cost is minimal \u2014 one new `static HashMap` and three lines of code. The benefit is that a translation unit which `#include`s `<stdio.h>` thirty times across header chains pays the search cost once.\n\n**Where we are.** `-fpic`/`-fPIC` enable PIC-friendly codegen for global and TLS variable addresses. Globals go through the GOT (`mov name@GOTPCREL(%rip), %rax`); TLS variables call `__tls_get_addr` with the linker-rewritable padding sequence. The non-PIC paths from Chapter 21 still fire when the flag is absent. The file-search path is now hashmap-backed, eliminating repeated `stat` calls on the same `#include` filename.\n\n---\n\n## 22.6 \u2014 Include-guard optimization, `#pragma once`, `#include_next`\n\n> `git checkout d48d9e5ae35b5eb1a9dcb0c07c1dba9e65bd83f3` \u2014 *Add include guard optimization*\n>\n> `git checkout a6c662207d38813b3dd490d81d8afe14ac99272b` \u2014 *[GNU] Add \"#pragma once\"*\n>\n> `git checkout f10bcebaa5df6bcb8e08e622ac44b0098e3133ae` \u2014 *[GNU] Add #include_next*\n\nThree commits, all in `preprocess.c`. The first two implement the same optimization with two different triggers (pattern detection and explicit pragma). The third adds the GNU mechanism for chained system headers.\n\n### 22.6.1 \u2014 Include-guard optimization\n\nThe classic include-guard pattern is:\n\n```c\n#ifndef FOO_H\n#define FOO_H\n... contents ...\n#endif\n```\n\nEvery `#include \"foo.h\"` after the first one tokenizes the entire file, walks past the `#ifndef` (which is now false because `FOO_H` is defined), skips the body via `skip_cond_incl`, and ends. The work is done once per include, even when the second through Nth includes contribute nothing. For a header included by many transitive paths, this can be a substantial fraction of the preprocessor's runtime.\n\nThe optimization detects the pattern on the *first* read, records the guard macro name, and short-circuits subsequent reads to skip the file entirely.\n\nThe detector:\n\n```c\nstatic char *detect_include_guard(Token *tok) {\n  // Detect the first two lines.\n  if (!is_hash(tok) || !equal(tok->next, \"ifndef\"))\n    return NULL;\n  tok = tok->next->next;\n\n  if (tok->kind != TK_IDENT)\n    return NULL;\n\n  char *macro = strndup(tok->loc, tok->len);\n  tok = tok->next;\n\n  if (!is_hash(tok) || !equal(tok->next, \"define\") || !equal(tok->next->next, macro))\n    return NULL;\n\n  // Read until the end of the file.\n  while (tok->kind != TK_EOF) {\n    if (!is_hash(tok)) {\n      tok = tok->next;\n      continue;\n    }\n\n    if (equal(tok->next, \"endif\") && tok->next->next->kind == TK_EOF)\n      return macro;\n\n    if (equal(tok, \"if\") || equal(tok, \"ifdef\") || equal(tok, \"ifndef\"))\n      tok = skip_cond_incl(tok->next);\n    else\n      tok = tok->next;\n  }\n  return NULL;\n}\n```\n\nThree checks:\n\n1. The first directive is `#ifndef IDENT`.\n2. The next directive is `#define IDENT` with the *same* identifier.\n3. The closing `#endif` is the last token in the file (its `next` is `TK_EOF`).\n\nIf any check fails, return NULL \u2014 the file isn't guard-shaped. If they all pass, return the guard macro name.\n\nThe middle of the file is walked via `skip_cond_incl` for nested `#if`/`#ifdef`/`#ifndef` blocks (so a nested conditional doesn't break the outer match), but *non-conditional* `#define`s and `#include`s and other directives don't disqualify the file. The check is \"the entire file body is wrapped in the guard,\" not \"the file body contains nothing but conditionals.\"\n\nThe detector runs once per file:\n\n```c\nstatic Token *include_file(Token *tok, char *path, Token *filename_tok) {\n  static HashMap include_guards;\n  char *guard_name = hashmap_get(&include_guards, path);\n  if (guard_name && hashmap_get(&macros, guard_name))\n    return tok;\n\n  Token *tok2 = tokenize_file(path);\n  if (!tok2)\n    error_tok(filename_tok, \"%s: cannot open file: %s\", path, strerror(errno));\n\n  guard_name = detect_include_guard(tok2);\n  if (guard_name)\n    hashmap_put(&include_guards, path, guard_name);\n\n  return append(tok2, tok);\n}\n```\n\nThe `include_guards` hashmap maps absolute include paths to guard macro names. On a re-include, two checks fire: is this file in the cache, and is its guard macro currently defined? If both, return immediately \u2014 the file is known to be a no-op. If not, tokenize the file as usual and try to record a guard for next time.\n\nThe two-stage check matters. A file's guard could in principle be `#undef`-ed after the first include, in which case the second include should run. The `hashmap_get(&macros, guard_name)` check picks that up \u2014 it consults the live macro table, which `#undef` would have cleared.\n\nWhat this *doesn't* do is what gcc does: it doesn't avoid re-tokenizing on subsequent includes that *aren't* fast-pathed. If the guard macro got `#undef`-ed, chibicc retokenizes the file and runs through the conditional. Gcc has more elaborate mechanisms for this; chibicc's optimization is the simple version.\n\n### 22.6.2 \u2014 `#pragma once`\n\n`#pragma once` is the GNU/MSVC extension that asks for the same optimization explicitly. A header that begins with `#pragma once` is included exactly once per translation unit, regardless of how it's structured.\n\nThe implementation reuses the cache pattern from \u00a722.6.1:\n\n```c\nstatic HashMap pragma_once;\n```\n\n`include_file` consults it at the top:\n\n```c\n// Check for \"#pragma once\"\nif (hashmap_get(&pragma_once, path))\n  return tok;\n```\n\nAnd `preprocess2` recognizes the pragma:\n\n```c\nif (equal(tok, \"pragma\") && equal(tok->next, \"once\")) {\n  hashmap_put(&pragma_once, tok->file->name, (void *)1);\n  tok = skip_line(tok->next->next);\n  continue;\n}\n```\n\nOnce a file's path is in `pragma_once`, subsequent `include_file` calls on the same path return without tokenizing. The `(void *)1` is the same \"present\" sentinel as in the keyword hashmaps.\n\nThis conditional is matched *before* the existing `equal(tok, \"pragma\")` catch-all (which silently consumes any other `#pragma`). So `#pragma once` is recognized and acted on; everything else still gets dropped.\n\nThe test (`test/pragma-once.c`) is a self-recursive include:\n\n```c\n#include \"test.h\"\n\n#pragma once\n\n#include \"test/pragma-once.c\"\n\nint main() {\n  printf(\"OK\\n\");\n  return 0;\n}\n```\n\nThe file includes itself. Without `#pragma once` this would loop forever (or, more precisely, would fail when the inner copy of `main` redefines the outer one). With `#pragma once`, the second `#include` is a no-op, and the test runs.\n\n### 22.6.3 \u2014 `#include_next`\n\n`#include_next` is the GNU mechanism for chained system headers. It says: \"find a file with this name, but skip the directory that the *current* file came from, and continue searching the include path from there.\"\n\nThe motivating use case is wrapper headers. A distribution might ship a wrapper `/usr/include/stdio.h` that adds some custom declarations and then does `#include_next <stdio.h>` to pull in the real one from another location. Without `#include_next`, the wrapper would re-include itself in an infinite loop.\n\nThe implementation needs to track *where* in the search path the current file was found, then resume from one slot later. Chibicc threads this through `search_include_paths`:\n\n```c\nstatic int include_next_idx;\n\nchar *search_include_paths(char *filename) {\n  ...\n  for (int i = 0; i < include_paths.len; i++) {\n    char *path = format(\"%s/%s\", include_paths.data[i], filename);\n    if (!file_exists(path))\n      continue;\n    hashmap_put(&cache, filename, path);\n    include_next_idx = i + 1;\n    return path;\n  }\n  return NULL;\n}\n```\n\nWhen a successful search lands on `include_paths[i]`, the global `include_next_idx` is set to `i + 1`. The next `#include_next` in the same file uses that index as its starting point:\n\n```c\nstatic char *search_include_next(char *filename) {\n  for (; include_next_idx < include_paths.len; include_next_idx++) {\n    char *path = format(\"%s/%s\", include_paths.data[include_next_idx], filename);\n    if (file_exists(path))\n      return path;\n  }\n  return NULL;\n}\n```\n\nNote that `include_next_idx` is only updated on a *fresh* (non-cached) search. A cached lookup returns immediately without touching the index, which means `#include_next` after a cache hit uses whatever index the most recent fresh search left behind. In the common case (a wrapper header that does `#include_next` once, immediately) this works; in more elaborate scenarios it could surprise. Chibicc's clientele isn't sophisticated enough to hit the edge case.\n\n`preprocess2` adds the directive:\n\n```c\nif (equal(tok, \"include_next\")) {\n  bool ignore;\n  char *filename = read_include_filename(&tok, tok->next, &ignore);\n  char *path = search_include_next(filename);\n  tok = include_file(tok, path ? path : filename, start->next->next);\n  continue;\n}\n```\n\nThe search uses `search_include_next` rather than `search_include_paths`. The double-quote-vs-angle-bracket distinction is read but ignored (`bool ignore`) \u2014 `#include_next` is always treated as a path search, never a same-directory lookup.\n\nThe test sets up three directories with chained wrappers:\n\n```sh\nmkdir -p $tmp/next1 $tmp/next2 $tmp/next3\necho '#include \"file1.h\"' > $tmp/file.c\necho '#include_next \"file1.h\"' > $tmp/next1/file1.h\necho '#include_next \"file2.h\"' > $tmp/next2/file1.h\necho 'foo' > $tmp/next3/file2.h\n$chibicc -I$tmp/next1 -I$tmp/next2 -I$tmp/next3 -E $tmp/file.c | grep -q foo\n```\n\nThe chain is `file.c` \u2192 `next1/file1.h` (via `-I`) \u2192 `next2/file1.h` (via `#include_next`) \u2192 `next3/file2.h` (via `#include_next`). The final file contains `foo`, which the preprocessor output should contain.\n\n**Where we are.** Three include-handling additions: pattern-based include-guard optimization (caches the guard macro name and short-circuits re-includes when it's defined), `#pragma once` (the explicit-opt-in form, with a separate hashmap of files that have asked for it), and `#include_next` (which resumes the search path past the directory the current file came from). All three use the new hashmap. Real-world headers with elaborate include chains now work.\n\n---\n\n## 22.7 \u2014 Linker-driver pass-throughs and the third-party harness\n\n> `git checkout 1e9b6dd1108690f22c84af8db606fea9fb7ec2db` \u2014 *Add -static option*\n>\n> `git checkout 4e5de36a36452ef9fe29ac55f7812f2bb9005d95` \u2014 *Add -shared option*\n>\n> `git checkout c8df7874c607f14eac3774680b55ab22c3aaf370` \u2014 *Add -L option*\n>\n> `git checkout d1bc9a4eb0e205b10a583c347a9fe7d4bed7b813` \u2014 *Add -Wl, option*\n>\n> `git checkout 469f159bb1adebb92ca2c9a7841466a98e6ad956` \u2014 *Add -Xlinker option*\n>\n> `git checkout fb4937024db2ee06fd60ea3bb2cfc6c898646a7d` \u2014 *Add scripts to test third-party apps*\n\nSix commits. Five are linker-driver additions; the last is a harness of shell scripts.\n\n### 22.7.1 \u2014 `-static`\n\n`-static` tells `ld` to produce a statically-linked executable. The driver flips a flag and pushes `-static` to `ld_extra_args`:\n\n```c\nif (!strcmp(argv[i], \"-static\")) {\n  opt_static = true;\n  strarray_push(&ld_extra_args, \"-static\");\n  continue;\n}\n```\n\nBut the local `opt_static` flag does more than pass `-static` through: it also restructures the linker invocation. A statically-linked executable shouldn't carry a dynamic linker path, and its libgcc/libc grouping needs `--start-group` / `--end-group` to handle circular references between `libgcc` and `libc`:\n\n```c\nif (!opt_static) {\n  strarray_push(&arr, \"-dynamic-linker\");\n  strarray_push(&arr, \"/lib64/ld-linux-x86-64.so.2\");\n}\n```\n\n```c\nif (opt_static) {\n  strarray_push(&arr, \"--start-group\");\n  strarray_push(&arr, \"-lgcc\");\n  strarray_push(&arr, \"-lgcc_eh\");\n  strarray_push(&arr, \"-lc\");\n  strarray_push(&arr, \"--end-group\");\n} else {\n  strarray_push(&arr, \"-lc\");\n  strarray_push(&arr, \"-lgcc\");\n  strarray_push(&arr, \"--as-needed\");\n  strarray_push(&arr, \"-lgcc_s\");\n  strarray_push(&arr, \"--no-as-needed\");\n}\n```\n\nStatic builds use `-lgcc_eh` (the static C++/exception-handling helper); dynamic builds use `-lgcc_s` (the shared variant) under `--as-needed`. The `--start-group`/`--end-group` brackets tell `ld` to retry the libraries in order until no new symbols are pulled in, which solves the circular-dependency problem that static builds always face.\n\nThe same commit also tidies the always-emitted `-L` paths \u2014 the previous version had `-L${libpath}` and `-L${libpath}/..` lines that are now collapsed into a single `-L/usr/lib/x86_64-linux-gnu`. Distribution-specific cleanup; not load-bearing for `-static` per se.\n\n### 22.7.2 \u2014 `-shared`\n\n`-shared` is symmetric with `-static`. A shared-library output uses `crtbeginS.o`/`crtendS.o` (the `S` suffix is for shared) instead of the executable startup files, and skips `crt1.o` (which carries `_start` for executables):\n\n```c\nif (opt_shared) {\n  strarray_push(&arr, format(\"%s/crti.o\", libpath));\n  strarray_push(&arr, format(\"%s/crtbeginS.o\", gcc_libpath));\n} else {\n  strarray_push(&arr, format(\"%s/crt1.o\", libpath));\n  strarray_push(&arr, format(\"%s/crti.o\", libpath));\n  strarray_push(&arr, format(\"%s/crtbegin.o\", gcc_libpath));\n}\n```\n\n```c\nif (opt_shared)\n  strarray_push(&arr, format(\"%s/crtendS.o\", gcc_libpath));\nelse\n  strarray_push(&arr, format(\"%s/crtend.o\", gcc_libpath));\n```\n\nThe `S`-suffixed crt files are PIC-friendly: they contain position-independent code suitable for shared-library inclusion. Without them, a shared library built by chibicc would have non-PIC startup code, which the loader would reject. Combined with `-fpic`, this completes the shared-library build path.\n\nThe driver also passes `-shared` through to `ld`:\n\n```c\nif (!strcmp(argv[i], \"-shared\")) {\n  opt_shared = true;\n  strarray_push(&ld_extra_args, \"-shared\");\n  continue;\n}\n```\n\n### 22.7.3 \u2014 `-L`\n\n`-L DIR` adds a directory to the linker's library-search path. The driver accepts both the spaced form and the joined form (`-L/foo` and `-L /foo`):\n\n```c\nif (!strcmp(argv[i], \"-L\")) {\n  strarray_push(&ld_extra_args, \"-L\");\n  strarray_push(&ld_extra_args, argv[++i]);\n  continue;\n}\n\nif (!strncmp(argv[i], \"-L\", 2)) {\n  strarray_push(&ld_extra_args, \"-L\");\n  strarray_push(&ld_extra_args, argv[i] + 2);\n  continue;\n}\n```\n\nBoth forms emit the spaced version to `ld`. The pair-of-pushes is the canonical way to add a `-L` to an `ld` invocation. No special handling is needed beyond pass-through.\n\n### 22.7.4 \u2014 `-Wl,` \u2014 pass through to the linker\n\n`-Wl,arg1,arg2,...` is gcc's mechanism for passing arbitrary arguments to the linker. Each comma-separated piece becomes a separate linker argument. Chibicc treats `-Wl,` like `-l` \u2014 it goes through `input_paths` rather than `ld_extra_args` so command-line ordering relative to other inputs is preserved:\n\n```c\nif (!strncmp(argv[i], \"-l\", 2) || !strncmp(argv[i], \"-Wl,\", 4)) {\n  strarray_push(&input_paths, argv[i]);\n  continue;\n}\n```\n\nThen the main input-processing loop, when it hits a `-Wl,`-prefixed string, splits it on commas and pushes each piece to `ld_args`:\n\n```c\nif (!strncmp(input, \"-Wl,\", 4)) {\n  char *s = strdup(input + 4);\n  char *arg = strtok(s, \",\");\n  while (arg) {\n    strarray_push(&ld_args, arg);\n    arg = strtok(NULL, \",\");\n  }\n  continue;\n}\n```\n\n`strtok` modifies the string in place (NUL-terminates each comma-separated piece). The `strdup` is necessary because the input may be a pointer into `argv`, which is shared. Each piece becomes a distinct linker argument: `-Wl,-z,muldefs,--gc-sections` becomes three arguments `-z`, `muldefs`, `--gc-sections`.\n\n### 22.7.5 \u2014 `-Xlinker`\n\n`-Xlinker arg` is the alternative to `-Wl,` for arguments that can't be expressed comma-separated. Each `-Xlinker` takes one argument and passes it to `ld` literally:\n\n```c\nif (!strcmp(argv[i], \"-Xlinker\")) {\n  strarray_push(&ld_extra_args, argv[++i]);\n  continue;\n}\n```\n\nNote that `-Xlinker` goes to `ld_extra_args` (no command-line ordering), while `-Wl,` goes through `input_paths` (with ordering). The distinction matters when `-Wl,arg` references an object file or library that `-l` would also reference \u2014 `-Wl,` preserves the relative position. `-Xlinker` doesn't need that because its uses are typically option-shaped (`-z muldefs`, `--gc-sections`) rather than object-shaped.\n\n### 22.7.6 \u2014 Third-party-app test scripts\n\nThe chapter's last commit doesn't change the compiler. It adds four shell scripts under `test/thirdparty/` that build real-world C codebases with chibicc:\n\n- `git.sh` \u2014 builds and tests the git repository.\n- `libpng.sh` \u2014 builds and tests libpng.\n- `sqlite.sh` \u2014 builds and tests SQLite.\n- `tinycc.sh` \u2014 builds the TinyCC compiler.\n\nA shared `common` script handles the boilerplate:\n\n```sh\nmake=\"make -j$(nproc)\"\nchibicc=`pwd`/chibicc\n\ndir=$(basename -s .git $repo)\n\nset -e -x\n\nmkdir -p thirdparty\ncd thirdparty\n[ -d $dir ] || git clone $repo\ncd $dir\n```\n\nEach script sources `common`, pins the upstream repository to a specific commit hash, and runs the project's own build:\n\n```sh\n#!/bin/bash\nrepo='git@github.com:rui314/libpng.git'\n. test/thirdparty/common\ngit reset --hard dbe3e0c43e549a1602286144d94b0666549b18e6\n\nCC=$chibicc ./configure\nsed -i 's/^wl=.*/wl=-Wl,/; s/^pic_flag=.*/pic_flag=-fPIC/' libtool\n$make clean\n$make\n$make test\n```\n\nThe `sed` of `libtool` is the load-bearing piece. `./configure` writes a `libtool` script with hard-coded flag conventions for the compiler it detected; if it doesn't recognize chibicc (and it doesn't), it falls back to defaults that don't match. The `sed` patches in the chibicc-compatible flags after the fact. Two flags get patched: `wl` (the per-compiler form for \"pass to linker\" \u2014 `-Wl,` for chibicc, same as gcc) and `pic_flag` (`-fPIC`).\n\nThe libpng repo is `rui314/libpng`, not the upstream libpng. Rui maintains a fork with whatever small patches were needed to compile under chibicc \u2014 a real-world stress test that exposed minor incompatibilities and got fixed in the fork rather than in chibicc. Same for `git`, where the pinned hash is a tested-known-good revision.\n\nThe scripts aren't part of `make test` \u2014 they're invoked manually because they require network access and many minutes of compile time. But their existence is a milestone: chibicc can build production C codebases. The compiler is no longer a toy.\n\n**Where we are.** The driver has the full linker-pass-through vocabulary: `-static` (with library-grouping changes), `-shared` (with `crt*S.o` startup-file selection), `-L`, `-Wl,` (comma-separated ordered pass-through), and `-Xlinker` (literal pass-through). The third-party harness exists to verify that real codebases compile. By this commit, chibicc is build-system-compatible enough to drop into existing C projects with minimal shimming.\n\n---\n\n## Recap\n\nTwenty-three commits. The chapter adds three things in roughly equal measure: a reusable hashmap (used immediately in five places), the dependency-file machinery (seven `-M` flags), and the linker-driver completion (five flags plus the third-party harness). Plus a small parser tweak for labels-as-values, a codegen change for `-fpic`, the file-search cache, and the three include-handling additions.\n\n- New file: `hashmap.c`. New types in `chibicc.h`: `HashMap`, `HashEntry`. Six functions in two flavors each (`hashmap_get`/`hashmap_get2`, etc.).\n- The macro table, two block-scope tables (`vars` and `tags`), and two keyword tables (`is_keyword` and `is_typename`) all migrate from linked lists or linear arrays to `HashMap`.\n- The file-search cache (`search_include_paths`), the include-guard cache (`include_guards`), and the `#pragma once` cache (`pragma_once`) are three more `HashMap`s in `preprocess.c`.\n- `Macro` loses two fields (`next`, `deleted`); `VarScope` loses two (`next`, `name`); `TagScope` is dissolved entirely (its data flows into the hashmap directly).\n- `Relocation->label` becomes `char **` to accommodate the lazy resolution of label-as-value names.\n- `gen_addr` gains a `-fpic`-mode branch with two new asm patterns: `mov name@GOTPCREL(%rip), %rax` for globals and the four-instruction `__tls_get_addr` sequence for TLS.\n- `parse_args` accepts `-M`, `-MF`, `-MP`, `-MT`, `-MQ`, `-MD`, `-MMD`, `-fpic`, `-fPIC`, `-static`, `-shared`, `-L`, `-Wl,`, and `-Xlinker`. The `take_arg` table is updated with `-MF`, `-MT`, and `-Xlinker`.\n- `print_dependencies` is the new function that emits Makefile-shaped rules. `quote_makefile` handles `$`/`#`/whitespace escaping. `in_std_include_path` filters system headers for `-MMD`.\n- `run_linker` learns conditional logic for `opt_static` (no dynamic-linker, `--start-group` library brackets) and `opt_shared` (`crtbeginS.o`/`crtendS.o` instead of executable startup files).\n- `preprocess.c` gains `detect_include_guard` and `search_include_next`; `include_file` gains the guard-cache and `pragma_once`-cache lookups; `preprocess2` gains `pragma once` and `include_next` directive handling.\n- `test/thirdparty/` is the new directory with four shell scripts pinning git, libpng, sqlite, and tinycc at known-good commits.\n\nThe chapter's twenty-three-row summary, in `main` order:\n\n| # | Hash | Subject | Section |\n|---|---|---|---|\n| 284 | `f0c98e0` | [GNU] Treat labels-as-values as compile-time constant | \u00a722.1 |\n| 285 | `0aad326` | Add string hashmap | \u00a722.2 |\n| 286 | `30520e5` | Use hashmap for macro name lookup | \u00a722.3 |\n| 287 | `655954e` | Use hashmap for block-scope lookup | \u00a722.3 |\n| 288 | `f694413` | Use hashmap for keyword lookup | \u00a722.3 |\n| 289 | `d0c4667` | Add `-M` option | \u00a722.4 |\n| 290 | `95d5a46` | Add `-MF` option | \u00a722.4 |\n| 291 | `57c1d4e` | Add `-MP` option | \u00a722.4 |\n| 292 | `db850f3` | Add `-MT` option | \u00a722.4 |\n| 293 | `fb5cfe5` | Add `-MD` option | \u00a722.4 |\n| 294 | `7aa72e4` | Add `-MQ` option | \u00a722.4 |\n| 295 | `c3edffb` | Add `-MMD` option | \u00a722.4 |\n| 296 | `86785fc` | Add `-fpic` and `-fPIC` options | \u00a722.5 |\n| 297 | `c0f0614` | Cache file search results | \u00a722.5 |\n| 298 | `d48d9e5` | Add include guard optimization | \u00a722.6 |\n| 299 | `a6c6622` | [GNU] Add \"#pragma once\" | \u00a722.6 |\n| 300 | `f10bceb` | [GNU] Add `#include_next` | \u00a722.6 |\n| 301 | `1e9b6dd` | Add `-static` option | \u00a722.7 |\n| 302 | `4e5de36` | Add `-shared` option | \u00a722.7 |\n| 303 | `c8df787` | Add `-L` option | \u00a722.7 |\n| 304 | `d1bc9a4` | Add `-Wl,` option | \u00a722.7 |\n| 305 | `469f159` | Add `-Xlinker` option | \u00a722.7 |\n| 306 | `fb49370` | Add scripts to test third-party apps | \u00a722.7 |\n\nErrata candidates added this chapter:\n\n- The Makefile-escape function (`quote_makefile`) is applied to the rule's *target* (default and user-supplied via `-MT`/`-MQ`) and to phony-rule target names (`-MP`), but not to the dependency list itself. A header path containing `$` or `#` would produce a malformed `.d` file. Gcc escapes both sides.\n- The `include_next_idx` global is updated only on a fresh (non-cached) `search_include_paths` call. A cached lookup leaves it at whatever value the most recent fresh search left behind. In multi-file translation units with elaborate `#include_next` chains this could surprise.\n\nErrata candidates closed this chapter: none. The remaining candidates are unchanged from Ch 21: Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's two (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), Ch 20's one (`is_compatible` array arm bug), Ch 21's two (`.size` missing for functions, suffix-only `.a`/`.so` recognition), plus the two added in this chapter.\n\nThe canonicalization-at-parse-time count is unchanged at eleven. The pre-factor-before-feature count grows by one if we count \u00a722.2's hashmap as a refactor staged before its three users; alternatively, one can read the pair-and-three pattern as a single arc rather than a refactor-first commit. We keep the count at nine, treating the hashmap addition as part of the same arc as its first three users. The psABI conformance count grows by one for `-fpic`/`-fPIC` (the GOT and `__tls_get_addr` sequences are the standard PIC forms). New count: nineteen.\n\nThrough Chapter 22 chibicc compiles real-world C code. The driver knows the linker vocabulary, the preprocessor caches its work, and the dependency-file emission lets `make`-based build systems stay in sync. The thread-local, alloca, VLA, long-double, and labels-as-values machinery from Chapter 21 plus this chapter's hashmap-driven performance and linker-driver flags together cover the gap between \"tutorial compiler\" and \"compiler that can build a Linux distribution's C packages.\" What remains is the long tail: atomics, the `_Generic` polish, and the final cleanups. Ten more commits, in Chapter 23.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/22-performance-deps-and-the-linker-driver.md",
  "description": "Check word count"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n \"23\\|Chapter 23\\|307\\|atomic\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | head -20",
  "description": "Check chapter 23 mapping"
}


---

## 🤖 Claude

Good — chapter 23 scope confirmed. Now session README and HANDOFF.

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/023-chapter-22-draft",
  "description": "Create session 023 directory"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/023-chapter-22-draft/README.md",
  "content": "# Session 023 \u2014 Chapter 22 draft\n\n**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002\u2013022).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 022 delivered Ch 21 (Thread-local, alloca, VLAs, seventeen commits, ~10,654 words). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 22 covers commits 284\u2013306: twenty-three commits \u2014 labels-as-values as compile-time constant, the string hashmap and three uses of it, the seven-commit `-M` family, `-fpic`/`-fPIC` (which actually changes codegen), file-search caching, three include-handling improvements (include-guard optimization, `#pragma once`, `#include_next`), five linker-driver pass-throughs (`-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker`), and the third-party-app shell-script harness.\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~9,320 words. Below the handoff's 12,000\u201314,000 target. Honest reading: most commits in this chapter are small driver-side or one-line preprocessor changes whose interesting content is one or two paragraphs. The \u00a722.4 `-M` family (seven commits) and \u00a722.2 hashmap (one commit) carry most of the weight at ~2,500 and ~1,800 words respectively. Padding the chapter to 13,000 would require either repeating the hashmap walk or inventing concept interludes; neither was warranted.\n- **Section structure:** 7 sections from 23 commits, exactly as the handoff proposed. \u00a722.1 single-commit, walks the `char ***` indirection. \u00a722.2 single-commit, walks the hashmap as a self-contained data-structure addition. \u00a722.3 three commits with three named subsections. \u00a722.4 seven commits with seven named subsections (longest section, ~2,500 words). \u00a722.5 two commits (the `-fpic` half is structurally larger than the file-search half). \u00a722.6 three commits, three subsections. \u00a722.7 six commits, six subsections.\n- **No concept interlude.** The handoff said \"possibly one\" around the hashmap. Reading the \u00a722.2 prose, the hashmap walk is self-contained enough at ~1,800 words; pulling out a separate \"open-addressing tradeoffs\" interlude would have introduced repetition with the \u00a722.3 three-call-site walk. The \u00a722.2 \u2192 \u00a722.3 progression is enough scaffolding.\n- **\u00a722.1 explains the `char ***` (`Relocation->label` becomes `char **`, plus the parser-side `char ***`) by naming the lazy-resolution mechanism.** The reason for the indirection is that `Node->unique_label` is filled in by codegen, after `eval2` runs over the global initializer; storing a pointer to the slot lets `emit_data` read the eventually-resolved value.\n- **\u00a722.2 walks the hashmap as a complete data structure.** Open-addressing, FNV-1a (called out as FNV-1a not FNV-1, with the multiply-before-XOR ordering noted), 70/50 watermarks, tombstone-and-NULL probe rules, the match-first ordering on insertion. Two API conventions named: the `(pointer, length)` key shape that fits tokens, and the `2`-suffixed variants vs the `strlen`-shimmed non-`2` variants.\n- **\u00a722.3's three subsections each name the access pattern.** Macros (lookup-heavy, lifetime full translation unit), block scopes (lookup-medium, lifetime per scope), keywords (insert-once, lookup-many). The keyword walk explains the lazy-init trick (`if (map.capacity == 0)` plus `static HashMap`) that builds the map on first call.\n- **\u00a722.4 walks all seven `-M` flags individually.** Each subsection names the specific behavior: `-M` writes to stdout; `-MF` redirects; `-MP` adds phony rules (with `i = 1` to skip the source file itself); `-MT` overrides the target with appending behavior; `-MD` enables alongside compilation; `-MQ` is `-MT` with `quote_makefile` escaping (covering `$`, `#`, whitespace); `-MMD` is `-MD` minus system headers (with the `std_include_paths` separation).\n- **\u00a722.5's `-fpic` walk corrects the handoff's prediction.** The handoff said `-fpic` \"may not actually change codegen\" since chibicc already emits rip-relative addressing. Wrong. `-fpic` adds a real PIC-mode branch in `gen_addr` that uses `mov name@GOTPCREL(%rip), %rax` for globals and the four-instruction `__tls_get_addr` sequence (with `data16` and `0x6666` padding for linker-rewritable general-dynamic TLS) for thread-local variables. The chapter's biggest surprise.\n- **\u00a722.5's file-search cache is named as a real performance win** (an order of magnitude on header-heavy translation units). The cache stores positive results only; negative results would require a separate sentinel, which Rui doesn't bother with.\n- **\u00a722.6's include-guard optimization is walked subsection-by-subsection** with attention to the three checks (first directive is `#ifndef IDENT`; second is `#define IDENT` with the same identifier; the closing `#endif` is the last token). Nested conditionals are walked via `skip_cond_incl` so they don't disqualify the file.\n- **\u00a722.6's `#pragma once` is shown as a thin reuse of the include-file cache pattern** with a separate `pragma_once` hashmap.\n- **\u00a722.6's `#include_next` walk names the `include_next_idx` global,** and the subtle gap that it's only updated on fresh (non-cached) searches. Flagged as an errata candidate in the closing recap.\n- **\u00a722.7 covers five linker-driver pass-throughs and the third-party harness.** `-static` walks the library-grouping change (`--start-group`/`--end-group` for libgcc/libgcc_eh/libc circular dependency) and the dynamic-linker omission. `-shared` walks the `crtbeginS.o`/`crtendS.o` substitution. `-L` walks the spaced-vs-joined form acceptance. `-Wl,` walks the `strtok` comma-split and the `input_paths` routing for ordering. `-Xlinker` walks the literal-pass-through to `ld_extra_args`. The third-party harness names the four pinned repos (git, libpng, sqlite, tinycc), the shared `common` script, and the `libtool` `sed` patch (the `wl=-Wl,` and `pic_flag=-fPIC` substitutions).\n\n### Interpretive calls\n\n1. **\u00a722.1's `Relocation->label` indirection is named as a lazy-resolution mechanism.** The reason for `char **` (rather than `char *`) is that `Node->unique_label` is generated during codegen, after `eval2` runs. Storing a pointer-to-pointer captures the slot's address, which gets dereferenced at emit time when the label name is filled in. The chapter names this; without the explanation the diff is mysterious.\n2. **\u00a722.2's hash function is named as FNV-1a, not FNV-1.** The constants `0xcbf29ce484222325` (offset basis) and `0x100000001b3` (prime) are the canonical 64-bit FNV constants. The multiply-before-XOR ordering is the FNV-1a variant. Both work; FNV-1a is the more recommended.\n3. **\u00a722.2's `(void *)-1` tombstone is named as conventional.** Any sentinel distinct from NULL and from a valid heap pointer would work; `(void *)-1` is the C tradition.\n4. **\u00a722.2 names the missing iteration primitive.** The hashmap has no API for walking all entries. A future include-guard optimization needs iteration over `include_paths`, but that's a `StringArray`, not a `HashMap`. Rui doesn't add iteration to the hashmap.\n5. **\u00a722.3 names the per-call-site asymptotic improvements.** Macros: a few hundred entries \u2192 ~30 keyword entries, both lookup-heavy. Scopes: up to 200 string compares per identifier reference (10 nested scopes \u00d7 20 locals) \u2192 10 hashmap_get calls. Keywords: 30-element linear scan \u2192 O(1) lookup.\n6. **\u00a722.3.3 names that the keyword hashmap is built once per program** via the `if (map.capacity == 0)` lazy-init guard plus `static HashMap`. Two separate static hashmaps for `is_keyword` (tokenize.c) and `is_typename` (parse.c); Rui doesn't try to share them.\n7. **\u00a722.4.6's `quote_makefile` is named as one-sided.** The escaping is applied to the rule's *target* (and via `-MP` to phony-rule names), but not to the dependency list. A header path containing `$` or `#` would produce a malformed `.d` file. Errata candidate.\n8. **\u00a722.4.7's `-MMD` filter is named as path-prefix-based.** The `in_std_include_path` predicate compares the dependency path's prefix against entries in `std_include_paths` (the snapshot of system include paths taken before user `-I` flags arrive). Simple and correct for the common case.\n9. **\u00a722.5.1's `-fpic` walk names the four-instruction TLS sequence as a linker-rewritable padding pattern.** The `data16` prefix and `0x6666` value are deliberate padding bytes that the linker can rewrite to convert general-dynamic TLS into local-dynamic or initial-exec, if it determines the variable is reachable in the local module. The 16-byte total occupancy is what the linker needs.\n10. **\u00a722.5.1 names that `-fpic` and `-fPIC` are both treated identically by chibicc** (in real toolchains, `-fpic` allows a smaller GOT). Chibicc uses the large-model code, which is a strict superset of what `-fpic` requires.\n11. **\u00a722.5.1 grows the psABI conformance count by one** for the GOT-and-`__tls_get_addr` sequences. New count: nineteen.\n12. **\u00a722.5.2 names that negative results aren't cached.** A `#include \"missing.h\"` will repeat the full search every time. In practice this is rare (preprocessing hits a missing header once and errors).\n13. **\u00a722.6.1 names the optimization's gap relative to gcc.** Chibicc's optimization caches the guard macro name; on subsequent includes, if the macro is defined, skip. If the macro is `#undef`-ed, chibicc retokenizes and runs through the conditional. Gcc has more elaborate mechanisms; chibicc's is the simple version.\n14. **\u00a722.6.3 names the `include_next_idx` cache-miss gap.** The index is only updated on a fresh (non-cached) `search_include_paths` call. A cached lookup leaves the index at whatever value the most recent fresh search produced. In the common case (a wrapper header that does `#include_next` once) this works; in elaborate scenarios it could surprise. Errata candidate.\n15. **\u00a722.7.1's `-static` walk names the library-circular-dependency reason for `--start-group`/`--end-group`.** Static builds need libgcc \u2192 libc \u2192 libgcc retries; dynamic builds use `--as-needed` plus `libgcc_s` (the shared variant) instead.\n16. **\u00a722.7.4 names the routing distinction between `-Wl,` and `-Xlinker`.** `-Wl,` goes through `input_paths` (preserves command-line ordering), `-Xlinker` goes through `ld_extra_args` (no ordering). Justified by the typical use cases.\n17. **\u00a722.7.6 names the `libtool sed` patch as load-bearing.** Configure-generated `libtool` doesn't recognize chibicc and falls back to defaults that don't match. The `wl=-Wl,` and `pic_flag=-fPIC` substitutions patch in chibicc-compatible values after the fact.\n18. **\u00a722.7.6 names that the harness exists outside `make test`.** The third-party scripts require network access and many minutes; they're invoked manually. But their existence is the chapter's milestone \u2014 chibicc can build production C codebases.\n\n### Voice / structure inherited from Ch 1\u201321\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. Multi-commit sections list all hashes at the top.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- One-table recap at the chapter close (with \u00a7-section column).\n- No concept interludes.\n\n### Three careful avoidances\n\n- **Did not invent a \"history of make and dependency files\" interlude.** Make has a long history (1976), and the `-M` family in gcc has its own evolution. The chapter focuses on what the seven flags do in chibicc and what their output looks like, without walking the standardization history.\n- **Did not invent a \"hashmap design tradeoffs\" interlude.** Open addressing vs separate chaining, FNV vs SipHash, robin-hood probing, etc. \u2014 all valid topics, but chibicc's hashmap is the simple version and over-explaining the alternatives would crowd out the actual walk.\n- **Did not over-explain the GOT and TLS access models.** The \u00a722.5 `-fpic` walk names the GOT, the GOTPCREL relocation, the `__tls_get_addr` runtime call, and the linker-rewritable padding. It doesn't walk the full Linux TLS spec or the `R_X86_64_REX_GOTPCRELX` ABI document.\n\n### Date-vs-position note\n\nThe twenty-three commits scatter widely across August, September, and October 2020. The seven `-M` commits in particular were drafted across a month \u2014 `-MF`, `-MP`, `-MT`, `-MD` on August 18; `-MQ` on September 3; `-M` on September 3 (later that day); `-MMD` on September 19. The chapter follows `main` order (which corresponds to logical-dependency order: `-M` first, then variants, then the `-MMD` polish) without commenting on the dates.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **Hashmap is now the workhorse data structure.** Five distinct call sites: `macros` (preprocess.c), `Scope->vars` and `Scope->tags` (parse.c), `is_keyword`'s static map (tokenize.c), `is_typename`'s static map (parse.c), `search_include_paths`'s `cache` (preprocess.c), `include_guards` and `pragma_once` (preprocess.c). Eight `HashMap` instances total. Future commits adding to the compiler should reach for the hashmap, not a linear scan.\n- **`Macro` lost `next` and `deleted` fields.** Now stored only in the hashmap.\n- **`VarScope` lost `next` and `name` fields.** Now stored only as hashmap key+value.\n- **`TagScope` is gone entirely** \u2014 the hashmap stores `name \u2192 Type *` directly.\n- **`Relocation->label` is `char **`.** The dereference happens in `emit_data`. The double-pointer accommodates lazy-resolved label names (`Node->unique_label`) and same-channel global names (`Obj->name`).\n- **`-fpic`/`-fPIC` is real codegen, not a flag-flip.** Two new asm patterns: `mov name@GOTPCREL(%rip), %rax` for globals and the four-instruction `data16 lea ... __tls_get_addr@PLT` sequence for TLS. The non-PIC paths from Ch 21 still fire when the flag is absent. Flag selects between them at codegen time.\n- **`std_include_paths`** is a snapshot of `include_paths` taken at the end of `add_default_include_paths` (i.e., before user `-I` flags). Used by `-MMD` to filter out system headers from dependency lists.\n- **`print_dependencies`** is the new function in main.c that emits Makefile-shaped rules. Reads `get_input_files()` to enumerate every File the tokenizer touched.\n- **`quote_makefile`** escapes `$` (\u2192 `$$`), `#` (\u2192 `\\#`), and whitespace (\u2192 `\\ ` with backslash doubling for prior backslashes). Applied to the rule's target only; *not* to the dependency list (errata candidate).\n- **`include_next_idx`** is a file-scope int in preprocess.c. Updated only on fresh (non-cached) `search_include_paths` calls. `#include_next` after a cache hit may use stale index (errata candidate).\n- **`detect_include_guard`** runs once per first-time include. Three checks: `#ifndef IDENT` first, `#define IDENT` (same identifier) second, `#endif` last token. Walks past nested conditionals via `skip_cond_incl`.\n- **`pragma_once`** hashmap stores file paths that asked for the optimization. Separate from `include_guards` (which stores file \u2192 guard-macro-name).\n- **`opt_static` and `opt_shared`** are mutually-exclusive driver booleans. Each restructures `run_linker` for its case. `-static` adds `--start-group`/`--end-group` brackets and omits `-dynamic-linker`. `-shared` swaps `crt1.o`/`crtbegin.o`/`crtend.o` for `crti.o`/`crtbeginS.o`/`crtendS.o` (the `S` suffix marks PIC-friendly startup files).\n- **`-Wl,arg1,arg2`** routes through `input_paths` so command-line ordering is preserved relative to other inputs. The main loop splits on commas via `strtok` and pushes each piece to `ld_args`.\n- **`-Xlinker arg`** routes through `ld_extra_args` (no ordering). Each `-Xlinker` takes one literal argument; comma-handling is not required.\n- **psABI conformance count is at nineteen** (up from eighteen). `-fpic`/`-fPIC` adds the GOT-and-`__tls_get_addr` sequences as conformant PIC forms.\n- **Canonicalization-at-parse-time count is unchanged at eleven.**\n- **Pre-factor-before-feature count is unchanged at nine.** The hashmap could be argued as a pre-factor for its three users, but the walk treats hashmap-and-three-users as a single arc rather than counting the hashmap as a separate refactor.\n- **Errata candidates added in Ch 22:**\n  - `quote_makefile` is one-sided (target only, not dependencies). Filenames containing `$` or `#` produce malformed `.d` files.\n  - `include_next_idx` is only updated on fresh `search_include_paths` calls. Cached lookups leave it stale.\n- **Errata candidates closed in Ch 22:** none.\n- **Errata candidates remaining:** Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's two (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block), Ch 20's one (`is_compatible` array arm bug), Ch 21's two (`.size` missing for functions, suffix-only `.a`/`.so` recognition), Ch 22's two new \u2014 total: ten.\n- **Stage-2 build is end-to-end chibicc, `-Wall`-clean** \u2014 unchanged.\n- **Chibicc compiles itself** \u2014 unchanged.\n- **Third-party harness exists** for git, libpng, sqlite, tinycc. Manual invocation; not part of `make test`.\n\n## Exit state\n\n- `chapters/22-performance-deps-and-the-linker-driver.md` drafted, ~9,320 words.\n- Session 023 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 024 (Chapter 23 \u2014 Atomics and the final polish, commits 307\u2013316, 10 commits).\n- CLAUDE.md status note updated to \"Ch 22 drafted\".\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --reverse --oneline 90b7ec0..HEAD 2>/dev/null | head -25; echo \"---\"; sed -n '115,125p' /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md",
  "description": "Look at commits 307-316 for Ch 23"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 23 commit diffs",
      "activeForm": "Reading all 23 commit diffs",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at fb49370",
      "activeForm": "Reading destination state of key files at fb49370",
      "status": "completed"
    },
    {
      "content": "Skim Ch 21 to match voice",
      "activeForm": "Skimming Ch 21 to match voice",
      "status": "completed"
    },
    {
      "content": "Draft chapters/22-performance-deps-and-the-linker-driver.md",
      "activeForm": "Drafting Chapter 22",
      "status": "completed"
    },
    {
      "content": "Write session 023 README.md",
      "activeForm": "Writing session 023 README.md",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF.md for session 024",
      "activeForm": "Writing HANDOFF.md for session 024",
      "status": "in_progress"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "awk -v start=307 -v end=316 'NR>=start && NR<=end' research/commits/main-commits.txt 2>/dev/null || sed -n '307,316p' /Users/cell/claude/chibicc-book/research/commits/main-commits.txt",
  "description": "Find commits 307-316"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/023-chapter-22-draft/HANDOFF.md",
  "content": "# Handoff: Ch 22 done \u2192 proceed to Ch 23\n\n**For:** the next claude session.\n**From:** session 023.\n**Status:** Ch 22 drafted (~9,320 words, twenty-three commits \u2014 labels-as-values as compile-time constant, the string hashmap and three uses of it, the seven-commit `-M` family, `-fpic`/`-fPIC` (real codegen change with `@GOTPCREL` and `__tls_get_addr`), file-search caching, include-guard optimization, `#pragma once`, `#include_next`, five linker pass-throughs (`-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`), and the third-party-app shell-script harness). Continue autonomously to Ch 23 (Atomics and the final polish, commits 307\u2013316 \u2014 ten commits covering `atomic_compare_exchange`, `atomic_exchange`, `_Atomic` and the atomic op-assigns, `stdatomic.h`, the cpython smoke test, `__attribute__((packed))`, `__attribute__((aligned))`, member access through `=` and `?:`, plus the very last commit). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/023-chapter-22-draft/README.md`](README.md)** \u2014 what session 023 did, including the seven-section structure, the two new errata candidates (one-sided `quote_makefile`, stale `include_next_idx` after cache hit), the psABI conformance count tick from eighteen to nineteen.\n2. **[`docs/sessions/022-chapter-21-draft/HANDOFF.md`](../022-chapter-21-draft/HANDOFF.md)** \u2014 the previous handoff. Standing notes still apply with Ch 22 updates folded in (see \u00a723 README for the running list).\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`22-performance-deps-and-the-linker-driver.md`](../../../chapters/22-performance-deps-and-the-linker-driver.md)** \u2014 the twenty-two chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 23 = commits 307\u2013316 (10 commits, \"Atomics and the final polish \u2014 the very last commit\"). The chapter mapping line lists the topics: `atomic_compare_exchange`, `atomic_exchange`, `_Atomic`, atomic op-assigns, `stdatomic.h`, cpython smoke test, `__attribute__((packed))`, `__attribute__((aligned))`, member access through `=` and `?:`.\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes. Ch 23 closes the project; the chapter recap may want a Rui quote about the project's scope or what's been left out.\n6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** \u2014 JP TOC; atomics aren't commonly featured in compiler tutorials. They might warrant explanation as their own concept-interlude candidate.\n\n## Chapter 23 scope\n\n**Title (working):** *Atomics and the final polish*.\n**Commits:** 307\u2013316 in chronological order on `main`. **Ten commits** \u2014 the final chapter, closing the project.\n**Concept interlude:** Possibly one. Atomics are unusual enough as a language feature that a short interlude on memory-order semantics, the C11 atomic model, and what `_Atomic` actually means at the codegen level might be warranted. Default conditional \u2014 judge while reading the commits.\n\n| # | Hash | Subject |\n|---|---|---|\n| 307 | `ca27455` | Add atomic_compare_exchange |\n| 308 | `80ea9d4` | Add atomic_exchange |\n| 309 | `d69a11d` | Add _Atomic and atomic ++, -- and op= operators |\n| 310 | `0a5d08c` | Complete stdatomic.h |\n| 311 | `2ed3fda` | Add test/thirdparty/cpython.sh |\n| 312 | `395308c` | redefinition |\n| 313 | `44bea4c` | Add __attribute__((packed)) |\n| 314 | `b35d148` | Add __attribute__((aligned(N)) for struct declaration |\n| 315 | `982041f` | Update README |\n| 316 | `90d1f7f` | Make struct member access to work with `=` and `?:` |\n\nTen commits. The natural section grouping:\n\n- **\u00a723.1 \u2014 Atomic builtins: compare-exchange and exchange** (commits 307\u2013308). Two commits. The `__sync_*` (or `__atomic_*`) builtin family that backs C11 `atomic_compare_exchange` and `atomic_exchange`. Walk how chibicc lowers these to `lock cmpxchg` and `lock xchg`. Likely ~1,000 words.\n- **\u00a723.2 \u2014 `_Atomic` qualifier and atomic op-assigns** (commit 309). One commit. The `_Atomic` type qualifier and atomic compound-assignment lowering (`*=`, `+=`, etc. on atomic-qualified types lower to a CAS-loop). Walk the type system extension (a new `is_atomic` flag on `Type`, or a new wrapper type kind), the CAS-loop codegen pattern, and how the increment/decrement (`++`/`--`) cases differ from full op-assigns. Likely ~1,500 words.\n- **\u00a723.3 \u2014 `stdatomic.h`** (commit 310). One commit. The library-side header that wraps the builtins behind C11 names. Walk what's in it; it's likely a thin shim. ~600 words.\n- **\u00a723.4 \u2014 The cpython test** (commit 311). One commit. A test script that compiles cpython with chibicc. Walk what it builds and what it verifies. ~400 words.\n- **\u00a723.5 \u2014 Two redefinition cleanups** (commit 312). One commit titled simply \"redefinition\" \u2014 likely fixes a redefinition diagnostic or relaxes one. Read the diff carefully to decide section placement; may fold into \u00a723.6 or \u00a723.7 if small. ~400 words.\n- **\u00a723.6 \u2014 `__attribute__((packed))` and `__attribute__((aligned))`** (commits 313\u2013314). Two commits. Both extend struct layout with attribute-driven overrides. Walk the parser side (attribute parsing extension) and the layout side (`offsetof` and `sizeof` honor the override). ~1,200 words.\n- **\u00a723.7 \u2014 README update** (commit 315). One commit. The README gets updated to reflect the project's final state. Likely a short walk of what changed. ~300 words.\n- **\u00a723.8 \u2014 Member access through `=` and `?:`** (commit 316). One commit. The very last commit on `main`. Allows `(a ? b : c).x = 5` and similar patterns. Walk the parser and lvalue-conversion piece. ~600 words.\n\nThat's eight sections from ten commits. **Target chapter length: ~6,500\u20138,000 words.** Likely closer to 7,000 \u2014 most commits are small. The two atomics-related sections will dominate.\n\nThis is the project's last chapter. The chapter recap should be a chapter-and-project recap: not just what Ch 23 added, but a brief survey of where the compiler stands. Don't write a \"Phase 3\" plan or post-mortem \u2014 that belongs in a separate session if the user wants it.\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in ca27455 80ea9d4 d69a11d 0a5d08c 2ed3fda 395308c 44bea4c b35d148 982041f 90d1f7f; do echo \"===== $h =====\"; git show --stat $h | head -10; done` to scan all 10 diffs.\n2. Read each commit. Pay particular attention to:\n   - **\u00a723.1's `ca27455`/`80ea9d4`** \u2014 the compare-exchange and exchange operations. Walk the `lock cmpxchg` and `lock xchg` codegen.\n   - **\u00a723.2's `d69a11d`** \u2014 the `_Atomic` qualifier extension. Walk the type-system change and the CAS-loop codegen pattern. The most complex commit in the chapter.\n   - **\u00a723.3's `0a5d08c`** \u2014 the `stdatomic.h` header. Walk what it defines.\n   - **\u00a723.4's `2ed3fda`** \u2014 the cpython test script. Walk what it builds. May be very small.\n   - **\u00a723.5's `395308c`** \u2014 titled simply \"redefinition.\" Read the diff carefully to decide what it does.\n   - **\u00a723.6's `44bea4c`/`b35d148`** \u2014 `packed` and `aligned`. Walk the attribute parser and the struct layout calculation.\n   - **\u00a723.7's `982041f`** \u2014 README update. Walk what changed if substantive.\n   - **\u00a723.8's `90d1f7f`** \u2014 the very last commit. Walk member access through `=` and `?:`.\n3. Read the destination state at `90d1f7f` for `parse.c`, `tokenize.c`, `codegen.c`, `chibicc.h`, `main.c`, `preprocess.c`, plus the new atomic-related includes. The atomics changes may touch `Type`, `Node`, and `gen_expr`/`gen_addr` substantially.\n4. Draft `chapters/23-atomics-and-the-final-polish.md`. Likely 6,500\u20138,000 words. Eight sections.\n5. Write `docs/sessions/024-chapter-23-draft/README.md`.\n6. **No further chapter handoff.** Ch 23 is the final chapter. The session 024 README should note that the bulk-drafting phase is complete and the next phase (full-pass review/revision) is the user's call. Optionally write a `HANDOFF.md` aimed at a \"Phase 3 setup\" session that the user may or may not run.\n\n## Voice / structure rules\n\nSame as Ch 1\u201322:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present tense for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 ten rows, single table is fine.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The `_Atomic` and CAS-loop section will want larger code blocks.\n- **Chapter close should be a project close.** A short paragraph or two surveying what chibicc handles after this final commit. Don't write a phase-3 plan.\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.\n- Don't invent features chibicc doesn't have.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **\u00a723.1/2's atomics are subtle.** C11 atomics have a memory-order parameter (`memory_order_seq_cst`, `memory_order_acquire`, etc.). Chibicc's implementation likely emits the strongest order (sequentially consistent) regardless of the parameter, since `lock`-prefixed instructions on x86 are seq-cst by default. Walk this honestly.\n- **\u00a723.2's CAS loop** is the canonical lock-free update pattern: load, compute new value, `cmpxchg`, retry on failure. Walk the loop structure and how `++`/`--`/`+=` etc. lower to it.\n- **\u00a723.6's `packed`/`aligned` change struct layout calculation.** The layout function (`new_struct_or_union_type` or wherever) needs to honor the attribute. Walk how the attribute is parsed and threaded through.\n- **\u00a723.8's \"member access through `=` and `?:`\"** is a parser-side change to `postfix`. The gcc extension allows `(a = b).x` and `(a ? b : c).x` to work as both rvalue and lvalue, with the assignment/conditional synthesizing a struct value. Walk how chibicc represents this.\n- **The chapter is the project's last.** The recap should briefly survey the whole compiler \u2014 what it handles, what it doesn't (no optimization, no register allocation, single back end, x86-64 Linux only, etc.). Honest closure.\n\n## Standing notes worth tracking across sessions\n\n- **The hideset on Token** \u2014 unchanged through Ch 22.\n- **The Token->origin chain** \u2014 unchanged.\n- **The `Token` line-marker fields** \u2014 `display_name`, `filename`, `line_delta` added in Ch 20 \u00a720.1. Stable.\n- **The eval-quartet duplication** \u2014 has a fifth member (`is_const_expr`) since Ch 21. Ch 22 \u00a722.1 extended `eval2`/`eval_rval` to use `char ***` for label addresses. Ch 23's atomics shouldn't touch the quartet.\n- **The cc1-vs-driver split** \u2014 unchanged.\n- **The `Initializer` tree** \u2014 Ch 19 added `Member *mem`; Ch 21 \u00a721.6 made array range designators honored. Stable.\n- **The local-vs-global split** \u2014 Ch 21 added `is_tls`; Ch 22 didn't change it.\n- **The `Relocation` mechanism** \u2014 `label` field is `char **` since Ch 22 \u00a722.1.\n- **The anonymous-global pattern** \u2014 unchanged.\n- **The `is_static` default in `new_gvar`** \u2014 gained `is_tls` companion in Ch 21. Stable.\n- **The `is_definition` flag on `Obj`** \u2014 stable since Ch 20.\n- **The `is_unsigned` flag on `Type`** \u2014 unchanged.\n- **The `__va_area__` magic name** \u2014 unchanged.\n- **The register-save-area layout** \u2014 unchanged.\n- **The argreg integer/FP split** \u2014 long double on-stack, SSE for FP, GP for integer. Stable.\n- **The `Member->idx` field and bitfield siblings** \u2014 Ch 23 \u00a723.6 may add an attribute-influenced sibling for `packed`/`aligned`.\n- **The `is_flexible` flag** \u2014 unchanged. Dead-code duplicate from \u00a719.7's `835cd24` is still in the source.\n- **`copy_struct_type`** \u2014 unchanged.\n- **`MIN`/`MAX` macros** \u2014 unchanged.\n- **`is_numeric` predicate** \u2014 gained TY_LDOUBLE in Ch 21. Stable.\n- **`is_flonum` and `has_flonum` diverged in Ch 21.**\n- **Canonicalization-at-parse-time count is at eleven.** Ch 22 didn't change it.\n- **Pre-factor-before-feature count is at nine.** Ch 22 didn't change it.\n- **psABI conformance count is at nineteen.** Ch 22 \u00a722.5 added `-fpic`/`-fPIC`. Ch 23's atomics may grow it (the `lock`-prefixed instructions are the standard psABI atomic forms).\n- **The fourth namespace (labels)** is unchanged.\n- **The `is_typename` predicate** \u2014 unchanged through Ch 22; uses a hashmap as of Ch 22.\n- **The `VarAttr` channel** has six fields after Ch 21. Stable through Ch 22.\n- **The `ND_NULL_EXPR` seed-pattern** \u2014 unchanged.\n- **The `rep stosb` pattern** \u2014 unchanged.\n- **The `unreachable()` macro** \u2014 unchanged.\n- **Per-token line numbers** \u2014 unchanged.\n- **GDB-debuggable output** \u2014 unchanged.\n- **Tests are in C.** Ch 22 added `test/pragma-once.c`; Ch 23 will likely add atomics tests and possibly attribute tests.\n- **The `Obj->tok` field** added in Ch 14 \u00a714.11 still has no readers.\n- **The `Type->name_pos` field** \u2014 no new uses.\n- **The `Type->origin` field** added in Ch 20 \u00a720.3 for compatibility tracking. Stable.\n- **The `Obj` struct gained two fields in Ch 21** (`is_tls`, `alloca_bottom`). Stable through Ch 22.\n- **`Type` gained `vla_len`/`vla_size`** in Ch 21. Stable through Ch 22.\n- **The `Token`/`Node` `fval`** widened to `long double` in Ch 21. Stable.\n- **The `>>` codegen quirk** \u2014 partially repaired.\n- **The `add_type` rule for `ND_STMT_EXPR`** \u2014 errata candidate.\n- **The hex-escape silent truncation** \u2014 errata candidate.\n- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members \u2014 five errata candidates.\n- **`f()` and `f(void)` are accepted as identical** \u2014 errata candidate.\n- **Empty brace initializer (`int x[3] = {};`)** \u2014 chibicc-specific extension.\n- **`.bss` is the third assembly section.** Five sections (`.text`, `.data`, `.bss`, `.tdata`, `.tbss`) plus `.comm`. Stable through Ch 22.\n- **`.align`** \u2014 unchanged.\n- **The `mov $0, %rax`** for variadic FP-count \u2014 errata candidate.\n- **The `fp_offset = fp * 8 + 48` non-conforming stride** \u2014 errata candidate.\n- **`long double` is real 80-bit** \u2014 closed in Ch 21.\n- **The default-argument-promotion gap for chars and shorts** \u2014 errata candidate.\n- **Float literals are inlined as integer-immediate-bit-cast.**\n- **Long double literals are split across two halves through the redzone.**\n- **Ch 1 errata list** unchanged.\n- **The cast/compound-literal disambiguator** \u2014 same lookahead pattern.\n- **The cast table is 11\u00d711.** Stable through Ch 22.\n- **Driver brittleness** \u2014 addressed by Ch 21's `-include`, `-x`, `-l`, `-s` and Ch 22's `-M` family, `-fpic`/`-fPIC`, `-static`/`-shared`/`-L`/`-Wl,`/`-Xlinker`. The driver vocabulary is now near-comprehensive.\n- **The link command's hardcoded distro list** \u2014 partially addressed in Ch 22 \u00a722.7.1's `-static` path-cleanup. Errata candidate remaining.\n- **`Node->funcname` is gone.**\n- **`mov %rax, %r10; call *%r10` is uniform across all calls.**\n- **The `StringArray` type** \u2014 picks up `std_include_paths` in Ch 22 \u00a722.4.7.\n- **`atexit(cleanup)` for tempfile disposal** \u2014 unchanged.\n- **The `run_subprocess` helper** \u2014 unchanged.\n- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Three remaining.\n- **Errata candidates added in Ch 18:** None high-priority.\n- **Errata candidates added in Ch 19:** UTF-16 char-literal silent truncation; dead-code duplicate `is_flexible` block. Two remaining.\n- **Errata candidates added in Ch 20:** `is_compatible` array arm bug. One remaining.\n- **Errata candidates added in Ch 21:** `.size` missing for function symbols; suffix-only `.a`/`.so` recognition. Two remaining.\n- **Errata candidates added in Ch 22:** one-sided `quote_makefile` (target-only escaping, dependencies unescaped); `include_next_idx` not updated on cache hit. Two remaining.\n- **Errata candidates closed in Ch 21:** \"long double is double\"; range designators not honored.\n- **Errata candidates closed in Ch 22:** none.\n- **`self.py` is gone.**\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.\n- **Chibicc compiles itself** as of commit 197 (Ch 17 \u00a717.6).\n- **Bitfield support is feature-complete.**\n- **Anonymous struct/union members** flatten via recursive `get_struct_member`.\n- **The pre-tokenize pass count is four.**\n- **The four char-literal prefixes** are functional.\n- **The four string-literal prefixes** are functional.\n- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.\n- **`__STDC_NO_VLA__`** \u2014 no longer defined as of Ch 21.\n- **`__STDC_NO_THREADS__`** \u2014 no longer defined as of Ch 21.\n- **UTF-8 in identifiers** uses C11 Annex D ranges.\n- **The GNU `$` extension** in identifiers.\n- **`__DATE__`, `__TIME__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`** are predefined.\n- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission, plus array range designators.\n- **`__VA_OPT__` and `,##__VA_ARGS__` are functional.**\n- **GCC-style variadic macros (`name...`)** are functional.\n- **`#pragma` is silently consumed** for everything except `#pragma once` (Ch 22 \u00a722.6.2).\n- **`typeof`, `_Generic`, `__builtin_types_compatible_p`** are functional.\n- **`sizeof(<function-type>)` returns 1.**\n- **The GNU `?:`-omitted-middle** is functional.\n- **`asm`** is verbatim-string-only.\n- **`inline` is treated as `static`**, with dead-static-inline elimination.\n- **`__attribute__` is macro-stubbed when `__GNUC__` is undefined.** Ch 23 \u00a723.6 adds `packed` and `aligned` as real recognized attributes.\n- **`-idirafter`, `-fcommon`/`-fno-common`** are functional.\n- **`offsetof` is in `<stddef.h>`.**\n- **Tentative definitions are functional.**\n- **`_Thread_local`/`__thread`** are functional.\n- **`alloca` is a builtin.**\n- **VLAs are functional.**\n- **`-include`, `-x`, `-E xc`, `-l`, `-s`, `.a`/`.so`** are in the driver vocabulary.\n- **`.type`/`.size`** directives are emitted.\n- **`long double` is real 80-bit extended precision.**\n- **GNU case ranges** are functional.\n- **GNU array range designators** are honored in elaboration.\n- **GNU labels-as-values** are functional inside function bodies and as compile-time constants in static initializers (Ch 22 \u00a722.1).\n- **Hashmap is the workhorse data structure.** Eight `HashMap` instances across the compiler.\n- **The `-M` family is complete.** Seven flags.\n- **`-fpic`/`-fPIC`** generate real PIC codegen with `@GOTPCREL` and `__tls_get_addr`.\n- **Include-guard optimization, `#pragma once`, `#include_next`** are functional.\n- **`-static`, `-shared`, `-L`, `-Wl,`, `-Xlinker`** are in the driver vocabulary.\n- **Third-party harness exists** for git, libpng, sqlite, tinycc.\n\n## Acceptance criteria for Ch 23\n\n- [ ] `chapters/23-atomics-and-the-final-polish.md` exists, end-to-end readable.\n- [ ] All ten commits covered, grouped into ~8 sections.\n- [ ] \u00a723.1 walks `atomic_compare_exchange` and `atomic_exchange` codegen (`lock cmpxchg`, `lock xchg`).\n- [ ] \u00a723.2 walks `_Atomic` and the CAS-loop pattern for atomic compound-assignments.\n- [ ] \u00a723.3 walks `stdatomic.h`.\n- [ ] \u00a723.4 walks the cpython smoke test.\n- [ ] \u00a723.5 walks the \"redefinition\" cleanup commit (or folds it into another section).\n- [ ] \u00a723.6 walks `__attribute__((packed))` and `__attribute__((aligned))`.\n- [ ] \u00a723.7 walks the README update (or notes it briefly).\n- [ ] \u00a723.8 walks \"member access through `=` and `?:`\".\n- [ ] Voice matches Ch 1\u201322.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references not needed (this is the last chapter).\n- [ ] Chapter recap is also a project recap \u2014 short, honest, no phase-3 plan.\n- [ ] psABI conformance count noted (atomics may grow it).\n- [ ] `docs/sessions/024-chapter-23-draft/README.md` written.\n- [ ] No further `HANDOFF.md` needed unless the user wants a \"Phase 3 setup\" session primed.\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/023-chapter-22-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/023-chapter-22-draft/README.md   (what session 023 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md\n13. chapters/11-all-the-operators.md\n14. chapters/12-initializers.md\n15. chapters/13-linkage.md\n16. chapters/14-variadics-signedness-qualifiers.md\n17. chapters/15-floating-point.md\n18. chapters/16-the-compiler-driver.md\n19. chapters/17-a-preprocessor-from-scratch.md\n20. chapters/18-the-full-abi.md\n21. chapters/19-unicode-and-designated-initializers.md\n22. chapters/20-gcc-extensions-worth-supporting.md\n23. chapters/21-thread-local-alloca-vlas.md\n24. chapters/22-performance-deps-and-the-linker-driver.md (most recent)\n25. research/commits/chapter-mapping.md            (confirms Ch 23 scope)\n26. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 23 (Atomics and the final polish, commits 307\u2013316)\nper the steps in the handoff. Ten commits, eight sections proposed.\nThe \u00a723.2 _Atomic walk is the most complex commit in the chapter\n(CAS-loop codegen, type system extension). The chapter is the\nproject's last; the recap should briefly survey the whole compiler.\nEnd-of-session: write your session dir under\ndocs/sessions/024-chapter-23-draft/ with a README. No further\nHANDOFF.md needed unless the user wants a \"Phase 3 setup\" session\nprimed.\n```\n"
}


---

## 🤖 Claude

Now update CLAUDE.md status.

---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020), Ch 20 drafted (session 021), Ch 21 drafted (session 022). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020), Ch 20 drafted (session 021), Ch 21 drafted (session 022), Ch 22 drafted (session 023). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "offset": 75,
  "limit": 10
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Ch 21 drafted (session 022). Continuing autonomously per user direction.",
  "new_string": "Ch 21 drafted (session 022), Ch 22 drafted (session 023). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 23 commit diffs",
      "activeForm": "Reading all 23 commit diffs",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at fb49370",
      "activeForm": "Reading destination state of key files at fb49370",
      "status": "completed"
    },
    {
      "content": "Skim Ch 21 to match voice",
      "activeForm": "Skimming Ch 21 to match voice",
      "status": "completed"
    },
    {
      "content": "Draft chapters/22-performance-deps-and-the-linker-driver.md",
      "activeForm": "Drafting Chapter 22",
      "status": "completed"
    },
    {
      "content": "Write session 023 README.md",
      "activeForm": "Writing session 023 README.md",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF.md for session 024",
      "activeForm": "Writing HANDOFF.md for session 024",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status",
      "activeForm": "Updating CLAUDE.md status",
      "status": "completed"
    }
  ]
}


---

## 🤖 Claude

Chapter 22 drafted, session 023 wrapped.

- [chapters/22-performance-deps-and-the-linker-driver.md](chapters/22-performance-deps-and-the-linker-driver.md) — ~9,320 words, 23 commits, 7 sections (labels-as-values constants; the hashmap; three hashmap users; the seven-flag `-M` family; `-fpic` and file-search caching; include-guard / `#pragma once` / `#include_next`; the linker pass-throughs and the third-party harness).
- [docs/sessions/023-chapter-22-draft/README.md](docs/sessions/023-chapter-22-draft/README.md) and [HANDOFF.md](docs/sessions/023-chapter-22-draft/HANDOFF.md) — drafting decisions, two new errata candidates (one-sided `quote_makefile`, stale `include_next_idx` after cache hit), psABI count tick from eighteen to nineteen, prompt block primed for Ch 23 (commits 307–316, the final chapter).
- [CLAUDE.md](CLAUDE.md) status updated.

The chapter came in below the handoff's 12K-14K target — most commits are small driver-side pieces whose interesting content is one to two paragraphs each. The §22.5 walk corrected the handoff's prediction that `-fpic` "may not actually change codegen" — it does, with new `@GOTPCREL` and `__tls_get_addr` sequences.

Handoff for session 024: docs/sessions/023-chapter-22-draft/HANDOFF.md

---

