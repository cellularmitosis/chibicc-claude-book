# Claude Conversation Log

Session ID: 850b11eb-4bb0-455a-905b-fc8175f2126e
Date: 2026-05-10 01:34:13

---

## 👤 User

please find the latest handoff doc and proceed

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "ls -la /Users/cell/claude/chibicc-book/docs/sessions/",
  "description": "List session directories"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "ls /Users/cell/claude/chibicc-book/docs/sessions/010-chapter-09-draft/",
  "description": "List session 010 contents"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/010-chapter-09-draft/HANDOFF.md"
}


---

## 🤖 Claude

The handoff is for Chapter 10 — *Filling out the type system*, commits 56–75. Twenty commits, the largest chapter. The user wants autonomous progress. Let me set up tracking and start by reading the foundational context.

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
      "content": "Read session 010 README and prior chapter for voice",
      "activeForm": "Reading session 010 README and prior chapter",
      "status": "in_progress"
    },
    {
      "content": "Scan all 20 commits (56-75) via git show --stat",
      "activeForm": "Scanning 20 commits",
      "status": "pending"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading commit diffs in detail",
      "status": "pending"
    },
    {
      "content": "Read destination state of key files at commit 736232f",
      "activeForm": "Reading destination state files",
      "status": "pending"
    },
    {
      "content": "Draft chapters/10-filling-out-the-type-system.md",
      "activeForm": "Drafting Chapter 10",
      "status": "pending"
    },
    {
      "content": "Write session 011 README",
      "activeForm": "Writing session 011 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 012 (Chapter 11)",
      "activeForm": "Writing handoff for session 012",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/010-chapter-09-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/09-structs-and-unions.md",
  "offset": 1,
  "limit": 100
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/09-structs-and-unions.md /Users/cell/claude/chibicc-book/chapters/08-scopes-and-source-locations.md /Users/cell/claude/chibicc-book/chapters/07-globals-characters-strings.md",
  "description": "Word counts of recent chapters"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 010 README and prior chapter for voice",
      "activeForm": "Reading session 010 README and prior chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 20 commits (56-75) via git show --stat",
      "activeForm": "Scanning 20 commits",
      "status": "in_progress"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading commit diffs in detail",
      "status": "pending"
    },
    {
      "content": "Read destination state of key files at commit 736232f",
      "activeForm": "Reading destination state files",
      "status": "pending"
    },
    {
      "content": "Draft chapters/10-filling-out-the-type-system.md",
      "activeForm": "Drafting Chapter 10",
      "status": "pending"
    },
    {
      "content": "Write session 011 README",
      "activeForm": "Writing session 011 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 012 (Chapter 11)",
      "activeForm": "Writing handoff for session 012",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in 5831eda 43c2f08 9d48eef a817b23 74e3acc 8c3503b 287906a f46370e a6b82da 67543ea cb81a37 cfc4fa9 8b430a6 9e211cb 818352a fdc80bc 44bba96 aa0accc 48ba265 736232f; do echo \"===== $h =====\"; git show --stat $h | head -8; done",
  "description": "Scan all 20 commit stats"
}


---

## 🤖 Claude

Now I have a sense. Let me read the actual diffs in batches.

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 5831eda",
  "description": "Show commit 56 diff"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show cb81a37",
  "description": "Show commit 66 \u2014 32-bit registers"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 43c2f08",
  "description": "Show commit 57 \u2014 long type"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 9d48eef",
  "description": "Show commit 58 \u2014 short type"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a817b23",
  "description": "Show commit 59 \u2014 nested type declarators"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 74e3acc",
  "description": "Show commit 60 \u2014 function declaration"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 8c3503b",
  "description": "Show commit 61 \u2014 void type"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 287906a",
  "description": "Show commit 62 \u2014 complex type declarations"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 010 README and prior chapter for voice",
      "activeForm": "Reading session 010 README and prior chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 20 commits (56-75) via git show --stat",
      "activeForm": "Scanning 20 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading commit diffs in detail",
      "status": "in_progress"
    },
    {
      "content": "Read destination state of key files at commit 736232f",
      "activeForm": "Reading destination state files",
      "status": "pending"
    },
    {
      "content": "Draft chapters/10-filling-out-the-type-system.md",
      "activeForm": "Drafting Chapter 10",
      "status": "pending"
    },
    {
      "content": "Write session 011 README",
      "activeForm": "Writing session 011 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 012 (Chapter 11)",
      "activeForm": "Writing handoff for session 012",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show f46370e",
  "description": "Show commit 63 \u2014 long long alias"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a6b82da",
  "description": "Show commit 64 \u2014 typedef"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 67543ea cfc4fa9",
  "description": "Show commits 65 (sizeof typename) and 67 (cast)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 8b430a6",
  "description": "Show commit 68 \u2014 usual arithmetic conversion"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 9e211cb 818352a fdc80bc",
  "description": "Show commits 69, 70, 71"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 44bba96 aa0accc",
  "description": "Show commits 72 (_Bool) and 73 (char literal)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 48ba265",
  "description": "Show commit 74 \u2014 enum"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 736232f",
  "description": "Show commit 75 \u2014 file-scope functions"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout 736232f -- chibicc.h type.c codegen.c 2>&1 | head -5; ls -la chibicc.h type.c codegen.c",
  "description": "Checkout destination state"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/type.c"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/codegen.c"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout main -- chibicc.h type.c codegen.c parse.c tokenize.c 2>&1; git status",
  "description": "Restore main branch state"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book && git status",
  "description": "Verify book repo state"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -A 30 \"Chapter 10\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | head -40",
  "description": "Check chapter mapping for Ch 10"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -B 1 -A 5 -i \"type\\|cast\\|typedef\\|int\\|conversion\" /Users/cell/claude/chibicc-book/research/notes/quotes-rui.md | head -80",
  "description": "Find relevant Rui quotes"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -B 1 -A 3 \"JP\\|japanese\\|declaration\" /Users/cell/claude/chibicc-book/research/notes/japanese-book-toc.md 2>/dev/null | head -50",
  "description": "Check JP book TOC for type-declaration treatment"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -B 2 -A 30 \"\u578b\u306e\u69cb\u6587\\|type syntax\\|declaration reading\" /Users/cell/claude/chibicc-book/research/notes/japanese-book-toc.md 2>/dev/null",
  "description": "Look for JP book type-syntax detail"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 010 README and prior chapter for voice",
      "activeForm": "Reading session 010 README and prior chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 20 commits (56-75) via git show --stat",
      "activeForm": "Scanning 20 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading commit diffs in detail",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at commit 736232f",
      "activeForm": "Reading destination state files",
      "status": "completed"
    },
    {
      "content": "Draft chapters/10-filling-out-the-type-system.md",
      "activeForm": "Drafting Chapter 10",
      "status": "in_progress"
    },
    {
      "content": "Write session 011 README",
      "activeForm": "Writing session 011 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 012 (Chapter 11)",
      "activeForm": "Writing handoff for session 012",
      "status": "pending"
    }
  ]
}


---

## 👤 User

Continue from where you left off.

---

## 🤖 Claude

No response requested.

---

## 👤 User

status?

---

## 🤖 Claude

Status: working through Chapter 10 — *Filling out the type system* (commits 56–75, the 20-commit chapter, largest in the book).

**Done:**
- Read HANDOFF.md, session 010 README, recent chapter prose for voice
- Scanned all 20 commit stats
- Read every commit's full diff in detail (5831eda → 736232f)
- Read destination state of `type.c`, `codegen.c` at commit 736232f
- Cross-checked chapter mapping and JP book TOC for the concept interlude
- Decided on 15-section structure plus a "How to read a C type declaration" interlude between §10.2 and §10.3, targeting ~14k words

**Key findings worth flagging now:**
- Commit 56 (`int` → 32-bit) is *not* a clean pre-factor — it ships `argreg32`, `movsxd`, `mov %eax`, and `store_gp` in the same commit. Commit 66 is a *separate* arithmetic-codegen change (using `%eax`/`%edi` when both operands fit in 32 bits). The handoff's framing of §10.1 as "the codegen catch-up doesn't happen until commit 66" is slightly off and I'll correct it in prose.
- Commit 74 (`enum`) puts enum tags in the same `tags` namespace as struct/union — this is **C-correct** (C99 §6.2.3: struct/union/enum tags share one namespace per scope). The Ch 9 errata note that "struct and union share namespace" treated this as a wart, which was wrong. I'll quietly correct course in §10.14 without re-litigating Ch 9.
- Commit 57 also widens `Token.val` and `Node.val` to `int64_t` — worth noting in §10.2.

**Next:** drafting the chapter prose (§10.1 onward), then session 011 README, then HANDOFF.md for session 012 (Ch 11 — All the operators, commits 76–96).

Continuing now.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/10-filling-out-the-type-system.md",
  "content": "# Chapter 10 \u2014 Filling out the type system\n\n> Commits covered: `5831eda`, `43c2f08`, `9d48eef`, `a817b23`, `74e3acc`, `8c3503b`, `287906a`, `f46370e`, `a6b82da`, `67543ea`, `cb81a37`, `cfc4fa9`, `8b430a6`, `9e211cb`, `818352a`, `fdc80bc`, `44bba96`, `aa0accc`, `48ba265`, `736232f`. Twenty commits \u2014 the largest single chapter in the book \u2014 turning chibicc's three-type universe (`char`, `int`, `pointer`) into something a working C programmer would recognize.\n\nThrough Chapter 9 chibicc has known three integer types and one pointer type. `char` was 1 byte, `int` was 8, every pointer was 8, and arrays were the same shape as pointers when it mattered. Codegen took those sizes for granted: `mov`, `lea`, `add %rdi, %rax` \u2014 one register width, one set of instructions, no narrowing or widening anywhere.\n\nThat was always a placeholder. Real C programs declare `long` and `short` and `void` and `_Bool`. They write `(int) x` to convert. They put types behind `typedef` names. They write `static` on file-scope functions and they expect `int` to be 32 bits, not 64. Twenty commits in `main`'s middle stretch take chibicc from the placeholder to something close to right.\n\nThe chapter is long because the work is broad. New keywords; new `Type` constants; a `declspec` rewrite that handles `long int` and `int long` and `long long int` interchangeably; a parser hook for nested declarators like `int (*x)[3]`; a parser hook for `typedef`; a parser hook for `enum`; a `cast` AST node; a `cast_table` in codegen; the *usual arithmetic conversion*; argument and return-value conversion at function boundaries; 32-bit register variants. Each commit is small. Their interleaving is the chapter's main pedagogical challenge.\n\nThe fifteen sections (plus one concept interlude):\n\n- **\u00a710.1** \u2014 `int` becomes 32 bits (commit 56).\n- **\u00a710.2** \u2014 `long`, `short`, `void` (commits 57, 58, 61).\n- **Concept interlude** \u2014 How to read a C type declaration.\n- **\u00a710.3** \u2014 Nested declarators (commit 59).\n- **\u00a710.4** \u2014 Function declarations (commit 60).\n- **\u00a710.5** \u2014 Complex type declarations (commits 62, 63).\n- **\u00a710.6** \u2014 `typedef` (commit 64).\n- **\u00a710.7** \u2014 `sizeof(typename)` (commit 65).\n- **\u00a710.8** \u2014 32-bit register arithmetic (commit 66).\n- **\u00a710.9** \u2014 Casts (commit 67).\n- **\u00a710.10** \u2014 The usual arithmetic conversion (commit 68).\n- **\u00a710.11** \u2014 Function-call type conversions (commits 69, 70, 71).\n- **\u00a710.12** \u2014 `_Bool` (commit 72).\n- **\u00a710.13** \u2014 Character literals (commit 73).\n- **\u00a710.14** \u2014 `enum` (commit 74).\n- **\u00a710.15** \u2014 File-scope `static` functions (commit 75).\n\nThe dates of these commits do not match their position on `main`. Roughly seven of them (`a817b23`, `8c3503b`, `287906a`, `67543ea`, `cfc4fa9`, `aa0accc`, `48ba265`) are dated August 2019; the rest fall between March and September 2020. Notably the `int` size change (`5831eda`, dated 2020-09-06) sits in `main` order *before* a string of August-2019 commits, which means in chronological terms the August-2019 commits were originally written when `int` was still 8 bytes and were rewritten \u2014 or at least re-tested \u2014 once Rui changed the size. The chapter follows `main` order, which is the order the chapter mapping pins.\n\nA note on the concept interlude. Rui's existing Japanese book has a dedicated chapter (#11) on reading C type declarations. The chapter mapping calls for an interlude in this position drawing on that material. Our interlude is shorter than a full chapter and sits between \u00a710.2 and \u00a710.3 \u2014 directly before the nested-declarator commit, which is where a reader first needs to read declarations like `char (*x)[3]` and not just `int x`.\n\n---\n\n## 10.1 \u2014 `int` becomes 32 bits\n\n> `git checkout 5831edaab3eb6d56126c08f01f5639222602f7e5` \u2014 *Change size of int from 8 to 4*\n\nThe smallest commit of the chapter, and the one that pre-conditions all the rest. One number changes in `type.c`:\n\n```c\n-Type *ty_int = &(Type){TY_INT, 8, 8};\n+Type *ty_int = &(Type){TY_INT, 4, 4};\n```\n\n`int` is now 4 bytes wide and 4-byte aligned. Everything chibicc does with `int` immediately gets narrower: a `sizeof(int)` returns 4, an `int x[4]` is 16 bytes on the stack, the offsets `assign_lvar_offsets` hands out tighten up, the `align_to` calls in `struct_decl` start producing different layouts. The test suite catches all of it, and the diff is full of test edits \u2014 `sizeof(x)` going from 8 to 4, struct sizes going from 16 to 8, the gap between `&y` and `&z` in `int x; int y; char z;` changing from 15 to 7.\n\nBut the change isn't only a constant. Until this commit, every value chibicc loaded or stored fit in a 64-bit register, full stop. With `int` at 4 bytes, that's no longer true. A value at an `int`-typed address is 4 bytes wide, and reading it with `mov (%rax), %rax` would pull 4 bytes of garbage along with the 4 bytes that matter. Storing back with `mov %rax, (%rdi)` would overwrite 4 bytes of whatever lives next to the variable.\n\nSo `load` and `store` grow a 4-byte branch:\n\n```diff\n   if (ty->size == 1)\n     println(\"  movsbq (%%rax), %%rax\");\n+  else if (ty->size == 4)\n+    println(\"  movsxd (%%rax), %%rax\");\n   else\n     println(\"  mov (%%rax), %%rax\");\n```\n\n```diff\n   if (ty->size == 1)\n     println(\"  mov %%al, (%%rdi)\");\n+  else if (ty->size == 4)\n+    println(\"  mov %%eax, (%%rdi)\");\n   else\n     println(\"  mov %%rax, (%%rdi)\");\n```\n\n`movsxd` (\"move sign-extending double to quad\") reads 4 bytes from memory, sign-extends them into 64 bits, and writes the result into `%rax`. `mov %eax, (%rdi)` writes the low 4 bytes of `%rax` to memory. Both are exactly what the C standard's \"load an `int`\" and \"store an `int`\" should compile to on x86-64.\n\nThe commit also generalizes a small bit of the function prologue. Until now the parameter-saving loop hard-coded a two-way branch on size \u2014 1 byte (use `argreg8`) versus everything else (use `argreg64`):\n\n```c\nfor (Obj *var = fn->params; var; var = var->next) {\n  if (var->ty->size == 1)\n    println(\"  mov %s, %d(%%rbp)\", argreg8[i++], var->offset);\n  else\n    println(\"  mov %s, %d(%%rbp)\", argreg64[i++], var->offset);\n}\n```\n\nAdding a 4-byte case to that branch would have made the loop start to look ugly. Rui factors it out into a helper, `store_gp`, that switches on size and picks the right register from the right table:\n\n```c\nstatic char *argreg32[] = {\"%edi\", \"%esi\", \"%edx\", \"%ecx\", \"%r8d\", \"%r9d\"};\n\nstatic void store_gp(int r, int offset, int sz) {\n  switch (sz) {\n  case 1:\n    println(\"  mov %s, %d(%%rbp)\", argreg8[r], offset);\n    return;\n  case 4:\n    println(\"  mov %s, %d(%%rbp)\", argreg32[r], offset);\n    return;\n  case 8:\n    println(\"  mov %s, %d(%%rbp)\", argreg64[r], offset);\n    return;\n  }\n  unreachable();\n}\n```\n\nThe prologue becomes a one-liner:\n\n```c\nint i = 0;\nfor (Obj *var = fn->params; var; var = var->next)\n  store_gp(i++, var->offset, var->ty->size);\n```\n\nTwo things are worth pausing on. First, the `unreachable()` macro is new. It's defined in `chibicc.h` and exists for exactly this purpose \u2014 to mark a switch arm that the surrounding logic guarantees is never taken:\n\n```c\n#define unreachable() \\\n  error(\"internal error at %s:%d\", __FILE__, __LINE__)\n```\n\nCalling `unreachable()` aborts the compile with a file-and-line internal-error message. It documents *why* the function only handles three sizes (chibicc only has 1-, 4-, and 8-byte parameter types right now), and gives the future a noisy crash if a fourth size sneaks in without `store_gp` being updated. The `unreachable()` helper will get more callers as the type system grows.\n\nSecond, the new `argreg32` table foreshadows \u00a710.8. The lower-32-bit System V argument registers are `%edi`, `%esi`, `%edx`, `%ecx`, `%r8d`, `%r9d` \u2014 the same six slots as `argreg64`, named by their 32-bit aliases. With `argreg32` declared, all the codegen paths that need to write a 32-bit argument to memory have what they need; the broader 32-bit register usage in `gen_expr` arrives ten commits later.\n\nA subtlety: this commit changes the *size* but not the *codegen* of int arithmetic. After this commit, `int x = 1; int y = 2; x + y;` still computes through 64-bit registers \u2014 `add %rdi, %rax` adds full quadwords. The narrow load and narrow store mean the `int` operand is correctly read and written, but the arithmetic in between uses the high bits of `%rax`, where sign-extension from `movsxd` makes the answer agree with what a 32-bit `add` would have produced anyway. The test for that property is in `test/usualconv.c`, which won't land for another twelve commits \u2014 but the property already holds.\n\nThe chapter calls this kind of thing *pre-factor before feature*, a pattern named in \u00a76.5 (`new_add` / `new_sub`) and used in \u00a77.6 (`read_file`'s trailing-newline guarantee) and \u00a78.3 (per-token line numbers). This is the fourth instance, and it's unusually long-running: the codegen catch-up that finally exploits the smaller-than-rax `int` doesn't fully arrive until commit 66, ten commits later. In the meantime, every commit between here and there can assume `sizeof(int) == 4`.\n\n### Where we are\n\n`int` has its standard width. Loads and stores narrow correctly. The parameter-saving prologue uses a size-dispatch helper. The arithmetic codegen still pretends the world is 64-bit, and will keep pretending until \u00a710.8.\n\n---\n\n## 10.2 \u2014 `long`, `short`, `void`\n\n> `git checkout 43c2f0829f7d4ec3b96132b9964a778ff816b2eb` \u2014 *Add long type*\n> `git checkout 9d48eef58b964551350fe0c1f641a57f5da40529` \u2014 *Add short type*\n> `git checkout 8c3503bb94bd6b2d57e1f979d9fc1d84383b2961` \u2014 *Add void type*\n\nThree commits, the same shape repeated three times. Each one adds a new keyword, a new `TypeKind` enum entry, a new global `ty_X` constant with the right size and alignment, a new `is_keyword` line, a new `is_typename` line, a new `declspec` branch, a new `is_integer` line where applicable, and a test or two. Bundled as one section because once you've seen `long`, you've effectively seen `short` and `void`.\n\n### `long` (commit 57)\n\n`long` is the largest of the three. Eight bytes wide, eight-byte aligned, and a kind of \"default\" return type for anything `add_type` doesn't have a better answer for:\n\n```c\nType *ty_long = &(Type){TY_LONG, 8, 8};\n```\n\n```diff\n typedef enum {\n   TY_CHAR,\n   TY_INT,\n+  TY_LONG,\n   TY_PTR,\n   TY_FUNC,\n   TY_ARRAY,\n```\n\nThe parser's `declspec` grows a branch matching the existing `int` and `char` branches:\n\n```c\nif (equal(tok, \"long\")) {\n  *rest = tok->next;\n  return ty_long;\n}\n```\n\n`is_typename` learns the new keyword. The tokenizer's `is_keyword` table picks up both `short` and `long` in one go, even though `short`'s parser support won't land until the next commit:\n\n```diff\n   static char *kw[] = {\n     \"return\", \"if\", \"else\", \"for\", \"while\", \"int\", \"sizeof\", \"char\",\n-    \"struct\", \"union\",\n+    \"struct\", \"union\", \"short\", \"long\",\n   };\n```\n\nAdding `short` to the lexer one commit early is a small inconsistency that doesn't break anything: until the next commit, `short` will tokenize as a keyword and `declspec` will fail to match it, producing a parse error that talks about an unexpected token rather than an unrecognized identifier. Probably faster than coming back to the lexer twice.\n\n`long` is also the first commit where chibicc lifts the width of literal integer values to 64 bits. The token's `val` field, the AST's `val` field, and the parser's `new_num` parameter all change from `int` to `int64_t`:\n\n```diff\n struct Token {\n   ...\n-  int val;        // If kind is TK_NUM, its value\n+  int64_t val;    // If kind is TK_NUM, its value\n   ...\n };\n```\n\n```diff\n-static Node *new_num(int val, Token *tok) {\n+static Node *new_num(int64_t val, Token *tok) {\n```\n\nThe codegen emits `mov $%ld` instead of `mov $%d`. None of chibicc's existing programs use 64-bit literals, but since pointer arithmetic, `sizeof`, and `long`-typed expressions are about to start producing them, the widening is preventive.\n\n`add_type` makes `long` the type of function calls and a few other \"we don't know what else this would be\" cases:\n\n```diff\n   case ND_NE:\n   case ND_LT:\n   case ND_LE:\n   case ND_NUM:\n   case ND_FUNCALL:\n-    node->ty = ty_int;\n+    node->ty = ty_long;\n     return;\n```\n\nThat's a coarse rule and several of these arms are revisited in \u00a710.10 (`ND_EQ`/`ND_NE`/`ND_LT`/`ND_LE` get split out and given `ty_int`, `ND_NUM` gets an `int`-or-`long` decision based on the value). For now, `long` is doubling as both the new explicit type and the catch-all category for any expression whose type chibicc doesn't bother to compute.\n\n### `short` (commit 58)\n\nTwo bytes wide, two-byte aligned. The `TY_SHORT` enum entry, the `ty_short` global, the `declspec` branch, the `is_integer` extension. Codegen grows a 2-byte branch in `load`, `store`, and `store_gp`, mirroring the 4-byte branches that `int` introduced two commits ago:\n\n```c\nstatic char *argreg16[] = {\"%di\", \"%si\", \"%dx\", \"%cx\", \"%r8w\", \"%r9w\"};\n```\n\n```diff\n   if (ty->size == 1)\n     println(\"  movsbq (%%rax), %%rax\");\n+  else if (ty->size == 2)\n+    println(\"  movswq (%%rax), %%rax\");\n   else if (ty->size == 4)\n     println(\"  movsxd (%%rax), %%rax\");\n```\n\n`movswq` is \"move sign-extending word to quad\" \u2014 read 2 bytes, sign-extend to 8, deposit in `%rax`. Symmetric with `movsbq` for `char` and `movsxd` for `int`.\n\nAfter this commit, `argreg8`, `argreg16`, `argreg32`, `argreg64` exist as a complete set, and the eight/sixteen/thirty-two/sixty-four split foreshadowed in \u00a77.2 is now the actual structure of the parameter-passing code. (At the time of \u00a77.2, only `argreg8` and `argreg64` existed; we predicted an `argreg32` would arrive when `int` got smaller. It did, in commit 56. The `argreg16` for `short` closes the loop.)\n\n### `void` (commit 61)\n\nSet aside that `void` is one byte wide and one-byte aligned (`Type *ty_void = &(Type){TY_VOID, 1, 1};`). That field setting is meaningless because `sizeof(void)` is a compile error in standard C, and chibicc's parser refuses `void` variables outright. The `1, 1` is just to make sure `void` doesn't accidentally crash something that asks for its size. The commit adds an explicit error in `declaration` for `void`-typed variables:\n\n```c\nType *ty = declarator(&tok, tok, basety);\nif (ty->kind == TY_VOID)\n  error_tok(tok, \"variable declared void\");\n```\n\nAnd one in `add_type` for void-pointer dereferences:\n\n```c\ncase ND_DEREF:\n  if (!node->lhs->ty->base)\n    error_tok(node->tok, \"invalid pointer dereference\");\n  if (node->lhs->ty->base->kind == TY_VOID)\n    error_tok(node->tok, \"dereferencing a void pointer\");\n  node->ty = node->lhs->ty->base;\n  return;\n```\n\nThe void-pointer rule is what makes `void *` useful: you can declare it, you can take addresses with it, you can cast it back to a usable pointer type, but you can't dereference it directly. The test for the commit is one bracketed line in `test/variable.c`:\n\n```c\n{ void *x; }\n```\n\nA declaration that exists only to confirm `void *` parses and doesn't trip the \"variable declared void\" error.\n\nThe `void` commit also restructures `is_typename`. After three additions it had become a long disjunctive `equal(tok, ...)` chain; Rui replaces it with a small static table and a loop:\n\n```diff\n static bool is_typename(Token *tok) {\n-  return equal(tok, \"char\") || equal(tok, \"short\") || equal(tok, \"int\") ||\n-         equal(tok, \"long\") || equal(tok, \"struct\") || equal(tok, \"union\");\n+  static char *kw[] = {\n+    \"void\", \"char\", \"short\", \"int\", \"long\", \"struct\", \"union\",\n+  };\n+\n+  for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)\n+    if (equal(tok, kw[i]))\n+      return true;\n+  return false;\n }\n```\n\nThe change is mechanical, but it pays off: every subsequent commit in this chapter that adds a new type-keyword adds it as one entry to `kw[]`, instead of as one more `||`-clause to a growing chain. `typedef`, `_Bool`, `enum`, `static` will all walk in this way. (Eventually `is_typename` grows past pure keyword matching into a symbol-table lookup \u2014 that's commit 64.)\n\n### Where we are\n\n`long`, `short`, and `void` exist. The four-way size split (1, 2, 4, 8) is fully covered by `load`, `store`, and `store_gp`. The parameter-passing tables are complete. The `is_typename` helper has been refactored into a table-and-loop form that the rest of the chapter will extend. `void` exists as a non-storable type that can only appear behind a pointer or as a return type; the parser blocks `void x;` and the type-checker blocks `*p` for `void *p`.\n\nWhat's still wrong: `int x; int y;` at this point compiles to add operands using `add %rdi, %rax`, even though both operands are 4 bytes. The cleanup is in \u00a710.8.\n\n---\n\n## Concept interlude \u2014 How to read a C type declaration\n\nC's declaration syntax is famous for being hard to read. `int (*x)(int, int)` and `int *x[10]` and `int (*x)[10]` all *look* alike, and they all mean different things, and they all use the same five symbols (`*`, `[`, `]`, `(`, `)`) in slightly different positions. The hard part isn't the symbols, it's that the symbols don't compose left-to-right. They compose by following a path through the declaration that goes outward from the identifier.\n\nThe mnemonic for this is sometimes called *the spiral rule* or *the right-to-left-with-precedence rule* or *declaration follows use*. They're all the same observation: a C declaration is a copy of the *expression that uses the variable*, with the type written in front of it. If you can write the expression, you can write the declaration.\n\nTake `int *x[10]`. The expression that uses `x` is `*x[10]` \u2014 index `x` at position 10 (or thereabouts), then dereference. So `x` is a thing you can index, and the result of that indexing is a thing you can dereference. Indexing means it's an array. Dereferencing means the array's elements are pointers. Working out from the identifier:\n\n- `x` \u2014 an identifier\n- `x[10]` \u2014 an array of 10 of them\n- `*x[10]` \u2014 and each one is a pointer\n- `int *x[10]` \u2014 a pointer to `int`\n\nSo `x` is an array of 10 pointers to `int`.\n\nNow `int (*x)[10]`. The parentheses around `*x` change the precedence: the `*` binds before the `[10]`. The expression that uses `x` is `(*x)[10]` \u2014 dereference, then index. Working out:\n\n- `x` \u2014 an identifier\n- `*x` \u2014 dereference it (so `x` is a pointer)\n- `(*x)[10]` \u2014 index the result (so what `x` points to is an array)\n- `int (*x)[10]` \u2014 and each element is `int`\n\nSo `x` is a pointer to an array of 10 `int`s. Different from the previous declaration in every way that matters; the pointer is one level up, the array is one level down, and at the leaves there are integers, not pointers.\n\nThe third member of this family is `int (*x)(int, int)`. The expression is `(*x)(int, int)` \u2014 dereference, then call with two integer arguments. So `x` is a pointer to a function. Working out:\n\n- `x` \u2014 an identifier\n- `*x` \u2014 dereference it (so `x` is a pointer)\n- `(*x)(int, int)` \u2014 call with two `int`s (so what `x` points to is a function)\n- `int (*x)(int, int)` \u2014 and the function returns `int`\n\nSo `x` is a pointer to a function of two `int`s returning `int`. A function pointer.\n\nThe general procedure: *find the identifier, then read outward, applying operators in the order that an expression would*. Postfix operators (`[]` and `()`) bind tighter than the prefix `*`, and parentheses group things to override the default. When you reach the outermost layer, the leftmost type-specifier (`int`, `char`, etc.) tells you what the leaves are.\n\nThe classic Unix utility `cdecl` translates declarations to English using exactly this procedure, and reading its output is a perfectly reasonable way to learn the rules:\n\n```\n$ cdecl\n> explain int (*x)(int, int)\ndeclare x as pointer to function (int, int) returning int\n> explain char *(*foo[5])(int)\ndeclare foo as array 5 of pointer to function (int) returning pointer to char\n```\n\nFor chibicc, the rule simplifies because the parser builds the type *as it walks the declarator*, returning the final `Type *` once it reaches the closing edge. The walking is recursive: the declarator `(*x)[10]` is \"a `*` applied to whatever the parenthesized declarator parses to, with a `[10]` suffix on the result.\" That recursion is exactly the spiral, expressed as code. The next section is where the recursion appears in the parser.\n\nA short table of the worst offenders, with parses:\n\n| Declaration | Reading |\n|---|---|\n| `int x` | `x` is an `int`. |\n| `int *x` | `x` is a pointer to `int`. |\n| `int x[10]` | `x` is an array of 10 `int`s. |\n| `int *x[10]` | `x` is an array of 10 pointers to `int`. |\n| `int (*x)[10]` | `x` is a pointer to an array of 10 `int`s. |\n| `int (*x)(int)` | `x` is a pointer to a function of `int` returning `int`. |\n| `int *(*x)(int)` | `x` is a pointer to a function of `int` returning a pointer to `int`. |\n| `int (*x[5])(int)` | `x` is an array of 5 pointers to functions of `int` returning `int`. |\n| `int *(*(*x)(int))(char)` | `x` is a pointer to a function of `int` returning a pointer to a function of `char` returning a pointer to `int`. |\n\nThe last one is where most readers reach for `cdecl`.\n\n---\n\n## 10.3 \u2014 Nested declarators\n\n> `git checkout a817b23da3c6f39f22bc57c0a53169978d97d7fa` \u2014 *Add nested type declarators*\n\nEleven lines of parser change support every declaration form in the table above except the function-pointer rows. The key insight is recursive descent: when a declarator opens with `(`, parse a declarator inside it, then keep going.\n\nThrough commit 58, `declarator` looked like:\n\n```c\n// declarator = \"*\"* ident type-suffix\nstatic Type *declarator(Token **rest, Token *tok, Type *ty) {\n  while (consume(&tok, tok, \"*\"))\n    ty = pointer_to(ty);\n\n  if (tok->kind != TK_IDENT)\n    error_tok(tok, \"expected a variable name\");\n  ty = type_suffix(rest, tok->next, ty);\n  ty->name = tok;\n  return ty;\n}\n```\n\nA pile of `*`s, then an identifier, then a `type-suffix` (the optional array-size and function-parameters part). That handles `int x`, `int *x`, `int x[10]`, `int *x[10]`, `int x(int, int)` \u2014 but not anything with a parenthesized inner declarator.\n\nThe new version inserts one `if` arm:\n\n```diff\n // declarator = \"*\"* (\"(\" ident \")\" | \"(\" declarator \")\" | ident) type-suffix\n static Type *declarator(Token **rest, Token *tok, Type *ty) {\n   while (consume(&tok, tok, \"*\"))\n     ty = pointer_to(ty);\n\n+  if (equal(tok, \"(\")) {\n+    Token *start = tok;\n+    Type dummy = {};\n+    declarator(&tok, start->next, &dummy);\n+    tok = skip(tok, \")\");\n+    ty = type_suffix(rest, tok, ty);\n+    return declarator(&tok, start->next, ty);\n+  }\n+\n   if (tok->kind != TK_IDENT)\n     error_tok(tok, \"expected a variable name\");\n   ty = type_suffix(rest, tok->next, ty);\n   ty->name = tok;\n   return ty;\n }\n```\n\nThis is the recursion that makes the spiral rule work. Read it carefully because the structure is unusual: the parenthesized inner declarator is parsed *twice*, with a dummy type the first pass and the real type the second pass.\n\nTo see why, take `char (*x)[3]`. The base type passed in is `char`. Skip past the `(` and we're at `*x)[3]`. We *can't* directly call `declarator(... ty)` here, because that would build \"pointer to char\" first, then apply `[3]` to it \u2014 producing pointer-to-char-array, which is wrong. We want array-of-3-char first (apply the suffix that's *outside* the parens), then pointer (apply what's *inside*).\n\nSo the parser does it in two passes. Pass one: walk the inside of the parens with a dummy `Type` just to get the `tok` pointer past it. After pass one, `tok` points at `)`. Skip the `)`, run `type_suffix` (which sees `[3]` and wraps the base type into `array of 3 char`), then re-walk the inside of the parens with that array type \u2014 building the `*` to produce \"pointer to array of 3 char\". Done.\n\nThe double-walk is unusual but it's the smallest code change that gets the spiral right. An alternative would be to change `type_suffix` into something that returns a *function* (in the sense of \"an action to apply later\"), but C doesn't compose closures cleanly enough to make that idiomatic. Two passes with a dummy type is the chibicc-style answer.\n\nThe tests cover the main cases:\n\n```c\nASSERT(24, ({ char *x[3]; sizeof(x); }));\nASSERT(8, ({ char (*x)[3]; sizeof(x); }));\nASSERT(1, ({ char (x); sizeof(x); }));\nASSERT(3, ({ char (x)[3]; sizeof(x); }));\nASSERT(12, ({ char (x[3])[4]; sizeof(x); }));\nASSERT(4, ({ char (x[3])[4]; sizeof(x[0]); }));\nASSERT(3, ({ char *x[3]; char y; x[0]=&y; y=3; x[0][0]; }));\nASSERT(4, ({ char x[3]; char (*y)[3]=x; y[0][0]=4; y[0][0]; }));\n```\n\n`char *x[3]` is an array of 3 pointers (24 bytes \u2014 three 8-byte pointers). `char (*x)[3]` is a pointer to an array of 3 chars (8 bytes \u2014 one pointer). The first reading wraps pointer outside array; the second wraps array inside pointer. The parser disagrees with itself by exactly the amount the parentheses ask for.\n\n`char (x[3])[4]` is the spiral one: `x` is an identifier, `x[3]` is an array of 3, `(x[3])[4]` is an array of 3 of arrays of 4 \u2014 `char[3][4]` total, 12 bytes. Reading the outer `[4]` first (because of the parens-then-suffix pattern) and the inner `[3]` second (because that's what's *inside* the parens) gets the same answer as reading `char x[3][4]` straight.\n\nA note on what's still missing. This commit doesn't handle function-pointer declarators like `int (*f)(int, int)`. The parenthesized declarator in this commit handles \"pointer to (something)\" and \"array of (something)\", but not \"pointer to function (something)\" because `type_suffix` only recognizes `[`, not `(`. The complete handling \u2014 including function pointers \u2014 arrives in commit 62 (\u00a710.5), which rewrites `declspec` to handle `int long` and friends and tightens `declarator`'s suffix logic. For now, function-pointer declarations work only if you write them with a non-parenthesized identifier, which excludes the C-canonical form.\n\n### Where we are\n\nPointers can be wrapped around arrays, arrays around arrays, and the parentheses around parts of a declarator are honored. The double-walk-with-dummy-type trick will be reused two commits from now (in `abstract_declarator`, the version of `declarator` that takes no name) and is the chapter's main parser-side recursion idiom.\n\n---\n\n## 10.4 \u2014 Function declarations\n\n> `git checkout 74e3acc296d90d6d16ae70803196e967564fb16a` \u2014 *Add function declaration*\n\nEight lines, distributed across `chibicc.h`, `parse.c`, and `codegen.c`. The change splits the existing notion of \"function\" into two: a *function declaration* (`int foo(int, int);` \u2014 a name and a type, no body) and a *function definition* (the same, with a body).\n\nUntil now, `function()` in `parse.c` always read a body after parsing the prototype. The change introduces an `is_definition` flag on `Obj`:\n\n```diff\n // Global variable or function\n bool is_function;\n+bool is_definition;\n```\n\nAnd a small early-return in `function()`:\n\n```diff\n Obj *fn = new_gvar(get_ident(ty->name), ty);\n fn->is_function = true;\n+fn->is_definition = !consume(&tok, tok, \";\");\n+\n+if (!fn->is_definition)\n+  return tok;\n\n locals = NULL;\n enter_scope();\n```\n\nThe `consume` on `\";\"` is a semicolon-or-not check: if the prototype is followed by `;`, it's a declaration and we stop there. If not, the next token must be `{`, which means it's a definition and we proceed to parse the body as before.\n\nCodegen has to skip declarations:\n\n```diff\n for (Obj *fn = prog; fn; fn = fn->next) {\n-  if (!fn->is_function)\n+  if (!fn->is_function || !fn->is_definition)\n     continue;\n```\n\nThis is what makes `printf();` valid at the top of `test/test.h`:\n\n```diff\n #define ASSERT(x, y) assert(x, y, #y)\n+\n+int printf();\n```\n\nIt also makes the test harness `assert()` declarable without being defined in the same file (it's defined later in `test/common`). Until this commit, the test files leaned on the host C compiler to swallow undeclared `printf` and `assert` calls \u2014 chibicc would parse them, leaving the linker to pick up the implementations at link time, but the type information would be missing. With function declarations, those identifiers can be introduced with their types and chibicc has something to type-check against.\n\nTwo pieces don't land here. First, the parser doesn't yet *check* that calls match declarations \u2014 that arrives in \u00a710.11. Second, the parser doesn't yet *use* the declared parameter types to convert arguments \u2014 also \u00a710.11. For now, a forward declaration is little more than a placeholder; its value is mostly that it parses without erroring and that it gives the parser an `Obj` to point function calls at.\n\n### Where we are\n\n`int foo(int, int);` is valid syntax. The declaration creates an `Obj` with `is_function == true` and `is_definition == false`; codegen ignores it; the parser will be able to look up its type when commit 71 starts converting arguments. The split between declaration and definition is the precondition for everything that follows about function-call type-checking.\n\n---\n\n## 10.5 \u2014 Complex type declarations\n\n> `git checkout 287906abb85081b961e118bb80b30decb93fba6f` \u2014 *Handle complex type declarations correctly*\n> `git checkout f46370ef98adec5d3a840d69a6b34a03d80b0699` \u2014 *Add `long long` as an alias for `long`*\n\nTwo commits, the first of which is a substantial rework of `declspec`. The C standard says that the order of type-specifier keywords doesn't matter: `int long static const` and `static const long int` declare the same thing, and so does `long`, and so does `long int`, and so does `int long`. They're all aliases for the same type.\n\nThrough commit 61, `declspec` was a series of `equal(tok, \"...\")` arms that each consumed one keyword and returned. That's correct for one-keyword forms (`int`, `char`, `short`, `long`, `void`) but wrong for any combination \u2014 `long int x` would consume the `long` and return `ty_long`, leaving `int x` on the input, which `declarator` would then misparse as an `int` variable named `x`. The test suite gets away with not noticing because no test up to this point uses multi-keyword type specifiers.\n\nRui rewrites `declspec` to *count* keyword occurrences and look up the resulting tuple in a switch:\n\n```c\nstatic Type *declspec(Token **rest, Token *tok) {\n  enum {\n    VOID  = 1 << 0,\n    CHAR  = 1 << 2,\n    SHORT = 1 << 4,\n    INT   = 1 << 6,\n    LONG  = 1 << 8,\n    OTHER = 1 << 10,\n  };\n\n  Type *ty = ty_int;\n  int counter = 0;\n\n  while (is_typename(tok)) {\n    if (equal(tok, \"struct\") || equal(tok, \"union\")) {\n      if (equal(tok, \"struct\"))\n        ty = struct_decl(&tok, tok->next);\n      else\n        ty = union_decl(&tok, tok->next);\n      counter += OTHER;\n      continue;\n    }\n\n    if (equal(tok, \"void\"))       counter += VOID;\n    else if (equal(tok, \"char\"))  counter += CHAR;\n    else if (equal(tok, \"short\")) counter += SHORT;\n    else if (equal(tok, \"int\"))   counter += INT;\n    else if (equal(tok, \"long\"))  counter += LONG;\n    else unreachable();\n\n    switch (counter) {\n    case VOID:                ty = ty_void;  break;\n    case CHAR:                ty = ty_char;  break;\n    case SHORT:\n    case SHORT + INT:         ty = ty_short; break;\n    case INT:                 ty = ty_int;   break;\n    case LONG:\n    case LONG + INT:          ty = ty_long;  break;\n    default:\n      error_tok(tok, \"invalid type\");\n    }\n\n    tok = tok->next;\n  }\n\n  *rest = tok;\n  return ty;\n}\n```\n\nThe trick is the bit-field encoding. Each keyword gets two bits of counter (bits 0\u20131 for `VOID`, bits 2\u20133 for `CHAR`, etc.), so `LONG + INT` adds disjoint bits and produces a unique sum. The switch arms list every legal sum: `VOID` alone, `CHAR` alone, `SHORT` or `SHORT+INT`, `INT` alone, `LONG` or `LONG+INT`. Anything else \u2014 `LONG+CHAR`, `INT+CHAR`, two `SHORT`s \u2014 falls into the `default` arm and produces an error. Two bits per keyword leaves room for the second occurrence (`LONG+LONG` overflows into the third bit-pair, which the switch rejects as invalid for now and which the *next* commit explicitly accepts).\n\nThe Rui-style commentary is unusually expansive here \u2014 Rui is taking the time to explain what's going on:\n\n> The order of typenames in a type-specifier doesn't matter. For example, `int long static` means the same as `static long int`. That can also be written as `static long` because you can omit `int` if `long` or `short` are specified. However, something like `char int` is not a valid type specifier. We have to accept only a limited combinations of the typenames.\n\n> In this function, we count the number of occurrences of each typename while keeping the \"current\" type object that the typenames up until that point represent.\n\nThe `OTHER` counter \u2014 an artifact of the bit-field \u2014 gets added when a `struct` or `union` is found, and the bookkeeping ensures `struct` and a built-in type don't end up in the same `declspec`. The `is_typename` forward declaration at the top of the file is new because the new `declspec` calls it, and the existing `is_typename` is defined later.\n\nA new test file lands:\n\n```c\n// test/decl.c\nASSERT(1, ({ char x; sizeof(x); }));\nASSERT(2, ({ short int x; sizeof(x); }));\nASSERT(2, ({ int short x; sizeof(x); }));\nASSERT(4, ({ int x; sizeof(x); }));\nASSERT(8, ({ long int x; sizeof(x); }));\nASSERT(8, ({ int long x; sizeof(x); }));\n```\n\nOrder-insensitivity. `short int` and `int short` and `long int` and `int long` all parse and produce the right size.\n\n### `long long` (commit 63)\n\nTwo new switch arms accept `LONG + LONG` and `LONG + LONG + INT`:\n\n```diff\n case LONG:\n case LONG + INT:\n+case LONG + LONG:\n+case LONG + LONG + INT:\n   ty = ty_long;\n   break;\n```\n\n`long long` and `long long int` both alias `long`. That's not quite right by the standard \u2014 `long long` is supposed to be a distinct type guaranteed at least 64 bits \u2014 but on x86-64 Linux, `long` is already 64 bits and `long long` is also 64 bits, so the alias works. The test:\n\n```c\nASSERT(8, ({ long long x; sizeof(x); }));\n```\n\nThe bit-field encoding (two bits per keyword) is exactly large enough to distinguish `LONG+LONG` (which overflows to bit 9 set, bit 8 clear) from `INT+INT` (rejected) or `LONG+INT` (accepted, different type). If `long long long` were valid, the encoding would have to widen to three bits per keyword, but standard C tops out at `long long`.\n\n### Where we are\n\nThe parser handles every legal combination of type specifier keywords. `long int`, `int long`, `long long`, `long long int` all work. The `declspec` function is now a small state machine instead of a chain of one-keyword branches. Any new built-in keyword can be added by allocating two more bits and adding switch arms for its legal combinations. The next two type-keyword commits \u2014 `_Bool` (\u00a710.12) and `static` as a storage-class (\u00a710.15) \u2014 both use this hook.\n\n---\n\n## 10.6 \u2014 `typedef`\n\n> `git checkout a6b82da1ae9eefa44dada0baa885c283823ad59a` \u2014 *Add typedef*\n\n`typedef` introduces a new name for an existing type. The implementation is the chapter's first interesting symbol-table extension, and it forces a small structural change in how the parser knows whether a token starts a type.\n\nUntil now, `is_typename` answered the question \"is this a keyword that starts a type\" by comparing the token against a fixed list. With typedef, the answer is \"is this a keyword *or* is this an identifier that's been previously typedef'd in scope,\" and the second clause is a symbol-table lookup.\n\n### Same scope, two roles\n\nIn \u00a78.1 we built `Scope` as a chain with one variable list per frame; in \u00a79.4 we added a parallel `tags` list for struct tags. `VarScope` records \"this name is a variable; here's its `Obj`.\" With typedef, that record needs a second possible role: \"this name is a type alias; here's its `Type`.\"\n\nThe C standard says typedef names live in the same namespace as ordinary identifiers \u2014 variables, function names, enum constants. That means `typedef int t; int t = 3;` is *valid* C: the typedef name `t` and the `int` variable named `t` shadow each other, with the variable winning in the inner scope (because the variable's declaration redefines the binding). chibicc gets this right by sharing a single `VarScope` slot:\n\n```diff\n typedef struct VarScope VarScope;\n struct VarScope {\n   VarScope *next;\n   char *name;\n   Obj *var;\n+  Type *type_def;\n };\n```\n\nA `VarScope` entry now has three possible shapes: variable (`var` set, `type_def` null), typedef (`var` null, `type_def` set), or \u2014 much later, when enum constants land \u2014 enum constant. Lookup walks the chain and returns the entry; the caller decides which field to read.\n\n`find_var` changes return type from `Obj *` to `VarScope *` and stops dereferencing the inner `var`:\n\n```diff\n-static Obj *find_var(Token *tok) {\n+static VarScope *find_var(Token *tok) {\n   for (Scope *sc = scope; sc; sc = sc->next)\n     for (VarScope *sc2 = sc->vars; sc2; sc2 = sc2->next)\n       if (equal(tok, sc2->name))\n-        return sc2->var;\n+        return sc2;\n   return NULL;\n }\n```\n\nCallers are correspondingly updated. `primary` now distinguishes the variable case from the not-found-or-typedef case:\n\n```c\nVarScope *sc = find_var(tok);\nif (!sc || !sc->var)\n  error_tok(tok, \"undefined variable\");\n*rest = tok->next;\nreturn new_var_node(sc->var, tok);\n```\n\n### Storage-class specifiers\n\n`typedef` is a *storage-class specifier* in C grammar \u2014 it appears in the same position as `static`, `extern`, `auto`, `register` would, mixed in with type-specifier keywords. It can come before or after `int`, before or after `*`, anywhere in the `declspec` part of a declaration. To express that, Rui passes a new optional argument through `declspec`:\n\n```c\ntypedef struct {\n  bool is_typedef;\n} VarAttr;\n\nstatic Type *declspec(Token **rest, Token *tok, VarAttr *attr);\n```\n\nMost call sites pass `NULL` (struct member declarations, function parameter declarations \u2014 places where a storage-class specifier doesn't make sense). The two top-level call sites \u2014 `compound_stmt` and `parse` \u2014 pass a real `VarAttr` and read out `attr.is_typedef` after `declspec` returns:\n\n```c\nwhile (!equal(tok, \"}\")) {\n  if (is_typename(tok)) {\n    VarAttr attr = {};\n    Type *basety = declspec(&tok, tok, &attr);\n\n    if (attr.is_typedef) {\n      tok = parse_typedef(tok, basety);\n      continue;\n    }\n\n    cur = cur->next = declaration(&tok, tok, basety);\n  } else {\n    cur = cur->next = stmt(&tok, tok);\n  }\n  add_type(cur);\n}\n```\n\nThe `if (attr.is_typedef)` branch routes to `parse_typedef`, which reads one or more declarators and pushes each as a typedef name:\n\n```c\nstatic Token *parse_typedef(Token *tok, Type *basety) {\n  bool first = true;\n\n  while (!consume(&tok, tok, \";\")) {\n    if (!first)\n      tok = skip(tok, \",\");\n    first = false;\n\n    Type *ty = declarator(&tok, tok, basety);\n    push_scope(get_ident(ty->name))->type_def = ty;\n  }\n  return tok;\n}\n```\n\n`push_scope` was previously reading the variable into the new entry; that's now split. `push_scope` returns the `VarScope *`, and the caller fills in either `var` or `type_def`. `new_var` is updated:\n\n```diff\n-static VarScope *push_scope(char *name, Obj *var) {\n+static VarScope *push_scope(char *name) {\n   VarScope *sc = calloc(1, sizeof(VarScope));\n   sc->name = name;\n-  sc->var = var;\n   sc->next = scope->vars;\n   scope->vars = sc;\n   return sc;\n }\n```\n\n```diff\n static Obj *new_var(char *name, Type *ty) {\n   ...\n-  push_scope(name, var);\n+  push_scope(name)->var = var;\n   return var;\n }\n```\n\n### `is_typename` becomes a symbol-table lookup\n\nInside `declspec`, when we see an identifier (not a keyword), we have to ask: is this a typedef name? `find_typedef` is the helper:\n\n```c\nstatic Type *find_typedef(Token *tok) {\n  if (tok->kind == TK_IDENT) {\n    VarScope *sc = find_var(tok);\n    if (sc)\n      return sc->type_def;\n  }\n  return NULL;\n}\n```\n\nAnd `is_typename` extends to:\n\n```diff\n static bool is_typename(Token *tok) {\n   static char *kw[] = {\n     \"void\", \"char\", \"short\", \"int\", \"long\", \"struct\", \"union\",\n+    \"typedef\",\n   };\n\n   for (int i = 0; i < sizeof(kw) / sizeof(*kw); i++)\n     if (equal(tok, kw[i]))\n       return true;\n-  return false;\n+  return find_typedef(tok);\n }\n```\n\nThe \"return `find_typedef(tok)`\" line is the structural shift. Through commit 63, `is_typename` was a pure syntactic predicate \u2014 it looked at the token text and said yes or no. Now, whether a token starts a type *depends on context*: the same identifier can be a type name in one scope and a variable name in another. Every place the parser asks \"is this a declaration or a statement,\" it now asks the symbol table.\n\nThe `declspec` body grows a typedef-handling arm right at the top:\n\n```c\nwhile (is_typename(tok)) {\n  if (equal(tok, \"typedef\")) {\n    if (!attr)\n      error_tok(tok, \"storage class specifier is not allowed in this context\");\n    attr->is_typedef = true;\n    tok = tok->next;\n    continue;\n  }\n\n  Type *ty2 = find_typedef(tok);\n  if (equal(tok, \"struct\") || equal(tok, \"union\") || ty2) {\n    if (counter)\n      break;\n    if (equal(tok, \"struct\")) {\n      ty = struct_decl(&tok, tok->next);\n    } else if (equal(tok, \"union\")) {\n      ty = union_decl(&tok, tok->next);\n    } else {\n      ty = ty2;\n      tok = tok->next;\n    }\n    counter += OTHER;\n    continue;\n  }\n  ...\n}\n```\n\nA typedef name is treated like a `struct` or `union`: it's a \"user-defined type,\" it consumes the entire `declspec` slot (the `if (counter) break` early-out makes sure we don't try to combine `MyInt int`), and it bumps the `OTHER` counter so other type-specifier keywords get rejected.\n\n### Tests\n\n`test/typedef.c` is new:\n\n```c\ntypedef int MyInt, MyInt2[4];\ntypedef int;\n\nint main() {\n  ASSERT(1, ({ typedef int t; t x=1; x; }));\n  ASSERT(1, ({ typedef struct {int a;} t; t x; x.a=1; x.a; }));\n  ASSERT(1, ({ typedef int t; t t=1; t; }));\n  ASSERT(2, ({ typedef struct {int a;} t; { typedef int t; } t x; x.a=2; x.a; }));\n  ASSERT(4, ({ typedef t; t x; sizeof(x); }));\n  ASSERT(3, ({ MyInt x=3; x; }));\n  ASSERT(16, ({ MyInt2 x; sizeof(x); }));\n\n  printf(\"OK\\n\");\n  return 0;\n}\n```\n\nThe third test (`typedef int t; t t=1; t;`) is the C-grammar curiosity: `t` as a typedef and `t` as a variable name both exist, the variable shadows the typedef in its own scope, and reading `t` resolves to the variable. The fourth test is the block-scope version: an inner-scope typedef (`typedef int t;`) shadows an outer-scope typedef-of-struct, and the shadowing disappears at the closing brace, restoring the outer struct typedef just in time for `t x; x.a=2;`.\n\nThe two top-of-file declarations are also intentional. `typedef int MyInt, MyInt2[4];` shows that a single `typedef` can introduce multiple aliases (separated by commas), each with its own declarator on top of the same base type. `MyInt` is `int`, `MyInt2` is `int[4]`. And `typedef int;` \u2014 a typedef with no declarator at all \u2014 is allowed, vacuously, by the C grammar; it does nothing but parses cleanly.\n\n`test/common` and `test/test.h` files come along with `int printf();` style declarations, now that those work.\n\n### Where we are\n\n`typedef` works at file scope and inside blocks. Typedef names share the variable namespace, with shadowing handled by the existing scope chain. `is_typename` is now a context-sensitive predicate that consults the symbol table. The `VarScope` entry has gained a third field, and `VarAttr` is the new vehicle for storage-class specifiers; `static` will land in \u00a710.15 by walking through the same channel.\n\nThis is the chapter's biggest single structural change after `declspec`'s rewrite. Everything that follows can lean on it.\n\n---\n\n## 10.7 \u2014 `sizeof(typename)`\n\n> `git checkout 67543ea113c5cc2b15881e2bbb85ffd44feaef1f` \u2014 *Make sizeof to accept not only an expression but also a typename*\n\nC's `sizeof` operator takes either an expression or a parenthesized type name: `sizeof x` and `sizeof(int)` are both legal. Until now, chibicc supported only the expression form \u2014 `sizeof(int)` would fail because `int` isn't an expression. The fix is to introduce a *type-name* grammar rule (a declaration without an identifier) and add an alternative production for `sizeof`.\n\nThe C standard calls a name-less declaration an *abstract declarator*. It looks like a declarator with the identifier removed:\n\n| Concrete | Abstract |\n|---|---|\n| `int x` | `int` |\n| `int *x` | `int *` |\n| `int x[4]` | `int [4]` |\n| `int (*x)[4]` | `int (*)[4]` |\n\n`abstract_declarator` is a copy of `declarator` with the identifier-required check removed:\n\n```c\n// abstract-declarator = \"*\"* (\"(\" abstract-declarator \")\")? type-suffix\nstatic Type *abstract_declarator(Token **rest, Token *tok, Type *ty) {\n  while (equal(tok, \"*\")) {\n    ty = pointer_to(ty);\n    tok = tok->next;\n  }\n\n  if (equal(tok, \"(\")) {\n    Token *start = tok;\n    Type dummy = {};\n    abstract_declarator(&tok, start->next, &dummy);\n    tok = skip(tok, \")\");\n    ty = type_suffix(rest, tok, ty);\n    return abstract_declarator(&tok, start->next, ty);\n  }\n\n  return type_suffix(rest, tok, ty);\n}\n\n// type-name = declspec abstract-declarator\nstatic Type *typename(Token **rest, Token *tok) {\n  Type *ty = declspec(&tok, tok, NULL);\n  return abstract_declarator(rest, tok, ty);\n}\n```\n\nThe double-walk recursion is the same idiom as \u00a710.3, just without an identifier at the leaf.\n\n`primary` grows one new alternative:\n\n```diff\n // primary = \"(\" \"{\" stmt+ \"}\" \")\"\n //         | \"(\" expr \")\"\n+//         | \"sizeof\" \"(\" type-name \")\"\n //         | \"sizeof\" unary\n //         | ident func-args?\n //         | str\n //         | num\n```\n\n```c\nif (equal(tok, \"sizeof\") && equal(tok->next, \"(\") && is_typename(tok->next->next)) {\n  Type *ty = typename(&tok, tok->next->next);\n  *rest = skip(tok, \")\");\n  return new_num(ty->size, start);\n}\n```\n\nThe two-token lookahead \u2014 checking that the next two tokens are `(` and a typename \u2014 picks the parenthesized-type-name form. If those don't both match, the existing `sizeof unary` form kicks in, which still handles `sizeof x` and `sizeof(x)` (as a parenthesized expression).\n\n`test/sizeof.c` is new and exercises everything:\n\n```c\nASSERT(1, sizeof(char));\nASSERT(2, sizeof(short));\nASSERT(4, sizeof(int));\nASSERT(8, sizeof(long));\nASSERT(8, sizeof(char *));\nASSERT(8, sizeof(int **));\nASSERT(8, sizeof(int(*)[4]));\nASSERT(32, sizeof(int*[4]));\nASSERT(16, sizeof(int[4]));\nASSERT(48, sizeof(int[3][4]));\nASSERT(8, sizeof(struct {int a; int b;}));\n```\n\n`int(*)[4]` and `int*[4]` differ by exactly the parenthesization \u2014 pointer to array of 4 ints (8 bytes) versus array of 4 pointers to int (32 bytes). The spiral rule in action.\n\n### Where we are\n\n`sizeof` accepts both expressions and type names. The `abstract_declarator` and `typename` helpers exist; they will be reused immediately, in the next section, for cast expressions.\n\n---\n\n## 10.8 \u2014 32-bit register arithmetic\n\n> `git checkout cb81a379d9f7aef32fb1bbebd18f8618e1617a3f` \u2014 *Use 32 bit registers for char, short and int*\n\nThe codegen catch-up that \u00a710.1 set up. After this commit, when chibicc compiles `int a + int b`, the `add` instruction uses 32-bit registers: `add %edi, %eax`, not `add %rdi, %rax`. For `long`, the existing 64-bit forms remain.\n\nThe change is tightly local to one function. `gen_expr`'s arithmetic tail used to read `%rax` and `%rdi` as fixed strings:\n\n```c\ncase ND_ADD:\n  println(\"  add %%rdi, %%rax\");\n  return;\ncase ND_SUB:\n  println(\"  sub %%rdi, %%rax\");\n  return;\n```\n\nNow the operand widths are computed from the operand types and emitted as variable strings:\n\n```c\nchar *ax, *di;\n\nif (node->lhs->ty->kind == TY_LONG || node->lhs->ty->base) {\n  ax = \"%rax\";\n  di = \"%rdi\";\n} else {\n  ax = \"%eax\";\n  di = \"%edi\";\n}\n\nswitch (node->kind) {\ncase ND_ADD:\n  println(\"  add %s, %s\", di, ax);\n  return;\ncase ND_SUB:\n  println(\"  sub %s, %s\", di, ax);\n  return;\n...\n```\n\nThe condition is \"operand is `long` *or* a pointer (`base != NULL`).\" Pointers are 8 bytes wide and need 64-bit arithmetic. Everything else \u2014 `char`, `short`, `int`, `_Bool`, `enum` \u2014 fits in 32 bits, and 32-bit `add` is the right answer.\n\nWhy not 8- and 16-bit register arithmetic? Because the comment in `load` from \u00a710.10 says the upper half of `%eax` may contain garbage, but the *lower* half always contains a valid value, sign-extended from whatever width the source had. That is: when chibicc loads a `char`, it issues `movsbl`, which sign-extends the 8-bit value to 32 bits in `%eax`. When it loads a `short`, `movswl` sign-extends 16 to 32. And for `int`, the value is already 32 bits. So at the point of the `add`, the operand always occupies at least the full lower 32 bits of the register, and 32-bit `add` produces the correct sum-modulo-2^32. (The full sign-extension to 64 bits \u2014 what `movsxd` would do \u2014 isn't required for correctness inside the expression and would just be wasted work.)\n\n`ND_DIV` gets a special branch because the sign-extension required before `idiv` is different at 32 vs. 64 bits:\n\n```c\ncase ND_DIV:\n  if (node->lhs->ty->size == 8)\n    println(\"  cqo\");\n  else\n    println(\"  cdq\");\n  println(\"  idiv %s\", di);\n  return;\n```\n\n`cqo` (\"convert quad to oct\") sign-extends `%rax` into `%rdx:%rax` before `idiv`. `cdq` (\"convert double to quad\") sign-extends `%eax` into `%edx:%eax` before the 32-bit `idiv`. Picking the right one based on operand size keeps the dividend correctly extended.\n\n### What this commit *doesn't* do\n\nIt doesn't insert sign-extending casts when the two operands of a binary expression have different types. `int + long` still happens with the 64-bit code path because `node->lhs->ty->kind == TY_LONG` is true (assuming `add_type` set the type already). `short + long` works because `add_type` will eventually narrow `short` to the bottom 16 bits of `%eax` via `movswl` during `load`, and then commit 68's `usual_arith_conv` will insert an explicit `(long)short` cast, making the operand reach the `add` as a properly-extended long. The two pieces \u2014 *load with sign-extension* and *cast inserted by `add_type`* \u2014 combine to make mixed-width arithmetic work.\n\n`load` is unchanged here. It still emits `movsbq`/`movswq`/`movsxd` to load a 1-, 2-, or 4-byte value into `%rax` with sign-extension to 64 bits. The next codegen change (in commit 68) adjusts the `char`/`short` loads to extend only to 32 bits, which is the form we want now that the arithmetic code path uses `%eax`/`%edi`.\n\n### Where we are\n\n`int + int` finally compiles to `add %edi, %eax`, and `long + long` to `add %rdi, %rax`. Pointers stay on the 64-bit path. Division uses the right sign-extension for its operand width. The argreg-split foreshadowed in \u00a77.2 is now the operand-width split running through the entire arithmetic core of the compiler.\n\n---\n\n## 10.9 \u2014 Casts\n\n> `git checkout cfc4fa94c1eb17f37466571f74bbdfae03a6e11f` \u2014 *Add type cast*\n\nThe `(type) expr` syntax. Adds a new node kind, a new parser arm, a new codegen path, and an instruction-table for the narrowing/widening cases.\n\n### The AST node\n\nA cast is a unary operator: it takes one expression and produces a same-or-different-typed value. `ND_CAST` joins `NodeKind`:\n\n```diff\n   ND_STMT_EXPR, // Statement expression\n   ND_VAR,       // Variable\n   ND_NUM,       // Integer\n+  ND_CAST,      // Type cast\n } NodeKind;\n```\n\nA small constructor:\n\n```c\nstatic Node *new_cast(Node *expr, Type *ty) {\n  add_type(expr);\n\n  Node *node = calloc(1, sizeof(Node));\n  node->kind = ND_CAST;\n  node->tok = expr->tok;\n  node->lhs = expr;\n  node->ty = copy_type(ty);\n  return node;\n}\n```\n\n`ty` is copied because the cast's destination type may be modified independently of the source `Type` object \u2014 for example, by being assigned a different size or aligning differently. The `copy_type` call exists to make sure the cast doesn't share storage with whatever `Type *` was passed in.\n\n### Grammar\n\nA new precedence level wedges in between `unary` and `mul`. The grammar comment says it nicely:\n\n```c\n// cast = \"(\" type-name \")\" cast | unary\n```\n\nA cast is \"an opening paren followed by a type name followed by a closing paren followed by another cast\" \u2014 recursive, so `(int)(char)x` works \u2014 or, falling through, just a unary expression. The recursion is right-associative, which matches how casts compose in C.\n\nThe parser:\n\n```c\nstatic Node *cast(Token **rest, Token *tok) {\n  if (equal(tok, \"(\") && is_typename(tok->next)) {\n    Token *start = tok;\n    Type *ty = typename(&tok, tok->next);\n    tok = skip(tok, \")\");\n    Node *node = new_cast(cast(rest, tok), ty);\n    node->tok = start;\n    return node;\n  }\n\n  return unary(rest, tok);\n}\n```\n\n`is_typename(tok->next)` is the lookahead: a `(` followed by a typename means cast; a `(` followed by anything else means a parenthesized expression. Because `is_typename` consults the symbol table, `(t)x` parses as a cast iff `t` is a typedef in scope, and as a function call `(t)(x)` \u2014 wait, no, function calls in chibicc parse via `primary`'s `ident \"(\"` arm, not as `(expr)`. The ambiguity here is exactly the one C has: `(t)x` where `t` is an unknown identifier could in principle be either a cast or a parenthesized expression with a misplaced `x`. chibicc resolves it the standard way: if `t` is a typedef, cast; otherwise, fail at parse time when the inner expression doesn't end with `)`.\n\nThe grammar threads through every spot that previously called `unary`. Multiplication and `+`, `-`, `&`, `*` (deref) all now go through `cast`:\n\n```diff\n-// mul = unary (\"*\" unary | \"/\" unary)*\n+// mul = cast (\"*\" cast | \"/\" cast)*\n```\n\n```diff\n-// unary = (\"+\" | \"-\" | \"*\" | \"&\") unary\n+// unary = (\"+\" | \"-\" | \"*\" | \"&\") cast\n //       | postfix\n```\n\nThis is the standard C precedence: `(int)*p` is `(int)(*p)`, not `*((int)p)` \u2014 a cast binds *less* tightly than postfix and unary operators on its right side, but tighter than multiplicative. Adding a level between `unary` and `mul` and routing the unary-operator recursive calls through `cast` produces exactly that.\n\n### Codegen\n\nThe narrowing-and-widening table:\n\n```c\nenum { I8, I16, I32, I64 };\n\nstatic int getTypeId(Type *ty) {\n  switch (ty->kind) {\n  case TY_CHAR:  return I8;\n  case TY_SHORT: return I16;\n  case TY_INT:   return I32;\n  }\n  return I64;\n}\n\nstatic char i32i8[]  = \"movsbl %al, %eax\";\nstatic char i32i16[] = \"movswl %ax, %eax\";\nstatic char i32i64[] = \"movsxd %eax, %rax\";\n\nstatic char *cast_table[][10] = {\n  {NULL,  NULL,   NULL, i32i64}, // i8\n  {i32i8, NULL,   NULL, i32i64}, // i16\n  {i32i8, i32i16, NULL, i32i64}, // i32\n  {i32i8, i32i16, NULL, NULL},   // i64\n};\n\nstatic void cast(Type *from, Type *to) {\n  if (to->kind == TY_VOID)\n    return;\n\n  int t1 = getTypeId(from);\n  int t2 = getTypeId(to);\n  if (cast_table[t1][t2])\n    println(\"  %s\", cast_table[t1][t2]);\n}\n```\n\nRead this as a 4\u00d74 table indexed by (source size, destination size). The diagonal (i8\u2192i8, i16\u2192i16, etc.) is `NULL` \u2014 no instruction needed, the value is already the right shape. The columns \"down\" (going to a smaller type) are also `NULL` for sizes \u2264 4 because of the upper-bits-may-contain-garbage convention in `load`: chibicc never *guarantees* the upper 56 bits of an `%eax` holding a `char` are zero, but it also never *reads* them, because every consumer that wants 8 bits will use `%al` (not `%rax`). Truncation in chibicc is implicit.\n\nThe columns \"up\" (going to a larger type) need explicit sign-extension for char-or-short \u2192 32, and 32 \u2192 64. But the chart actually elides char-or-short \u2192 64: `t2 == 64` is reached via `i32i64` (`movsxd %eax, %rax`), and the surrounding load convention guarantees that when a `char` or `short` reaches a register, it's already been sign-extended to 32 bits via `movsbl` / `movswl` in `load`. So the 32 \u2192 64 step is the only widening explicitly emitted, and it composes with the loaded-as-int convention to produce the right answer for char \u2192 long and short \u2192 long without a special table entry.\n\nA subtlety: `cast_table[I64][I32]` is `NULL`, meaning \"cast from `long` to `int` emits nothing.\" That's because chibicc never zeroes the upper bits \u2014 the value remains in `%rax`, and a later `int`-sized load will read only the low 4 bytes from memory. For an `int x = (int) someLongVar;` the low 32 bits are written to `x`, and a subsequent read of `x` will sign-extend back to 64 bits via `movsxd`. Truncation followed by sign-extension is exactly the C semantics for narrowing-then-widening.\n\n`gen_expr` grows one switch arm:\n\n```c\ncase ND_CAST:\n  gen_expr(node->lhs);\n  cast(node->lhs->ty, node->ty);\n  return;\n```\n\n### Tests\n\n`test/cast.c` exercises the table:\n\n```c\nASSERT(131585, (int)8590066177);\nASSERT(513, (short)8590066177);\nASSERT(1, (char)8590066177);\nASSERT(1, (long)1);\nASSERT(0, (long)&*(int *)0);\nASSERT(513, ({ int x=512; *(char *)&x=1; x; }));\nASSERT(5, ({ int x=5; long y=(long)&x; *(int*)y; }));\n(void)1;\n```\n\n`8590066177` is `2^33 + 2`. Casting to `int` keeps the low 32 bits \u2014 `2 + (2^33 mod 2^32) = 2 + 0 = 131585`? Let me check: `2^33 = 8589934592`. `8590066177 - 8589934592 = 131585`. So `8590066177` low 32 bits are 131585. Casting to `short` keeps the low 16 bits: `131585 mod 65536 = 513`. Casting to `char` keeps the low 8 bits: `513 mod 256 = 1`. The diagonal arithmetic checks out.\n\n`(void)1` is the standard \"discard a value\" idiom. The codegen for `cast(int, void)` emits nothing \u2014 the table's `to->kind == TY_VOID` check is a fast exit at the top.\n\n### Where we are\n\nCasts are first-class. The narrowing-and-widening table is small (one helper per non-trivial transition); the implicit-truncation-on-narrowing convention saves several entries. The next commit uses casts as a building block for the *implicit* type conversions C inserts at every binary operator and assignment.\n\n---\n\n## 10.10 \u2014 The usual arithmetic conversion\n\n> `git checkout 8b430a6c5fd6d33a637f2c615f8e5ec59e7be30e` \u2014 *Implement usual arithmetic conversion*\n\nThe C standard's name for the rule that says \"if you add an `int` and a `long`, the `int` is first promoted to `long`, and the result is `long`.\" More generally: when a binary operator has two operands of different arithmetic types, the operands are converted to a common type *before* the operator runs, and the result has that common type.\n\nchibicc implements this by inserting `ND_CAST` nodes at parse-and-typecheck time. The mechanism is `add_type`: when the type-checker walks a `ND_ADD`, `ND_SUB`, `ND_MUL`, `ND_DIV`, `ND_EQ`, `ND_NE`, `ND_LT`, `ND_LE`, it computes a *common type* for the two operands and rewrites both children to be casts to that type.\n\n### `get_common_type` and `usual_arith_conv`\n\n```c\nstatic Type *get_common_type(Type *ty1, Type *ty2) {\n  if (ty1->base)\n    return pointer_to(ty1->base);\n  if (ty1->size == 8 || ty2->size == 8)\n    return ty_long;\n  return ty_int;\n}\n\nstatic void usual_arith_conv(Node **lhs, Node **rhs) {\n  Type *ty = get_common_type((*lhs)->ty, (*rhs)->ty);\n  *lhs = new_cast(*lhs, ty);\n  *rhs = new_cast(*rhs, ty);\n}\n```\n\nThe common-type rules, in chibicc's simplified form:\n- If the left operand is a pointer (`base != NULL`), the common type is \"pointer to that base.\" This is the asymmetry that makes pointer arithmetic work \u2014 `ptr + int` keeps the pointer's type, doesn't try to promote to common-numeric.\n- Otherwise, if either operand is 8 bytes, the common type is `long`.\n- Otherwise (both \u2264 4 bytes), the common type is `int`.\n\nThe third rule is the C standard's *integer promotion*: `char` and `short` are always promoted to at least `int` before any binary operator runs. chibicc's table makes this happen because `get_common_type` returns `ty_int` for any pair of small integers, and `usual_arith_conv` casts both operands to `int`.\n\nThe new `add_type` arms for the binary-arithmetic operators:\n\n```c\ncase ND_ADD:\ncase ND_SUB:\ncase ND_MUL:\ncase ND_DIV:\n  usual_arith_conv(&node->lhs, &node->rhs);\n  node->ty = node->lhs->ty;\n  return;\ncase ND_NEG: {\n  Type *ty = get_common_type(ty_int, node->lhs->ty);\n  node->lhs = new_cast(node->lhs, ty);\n  node->ty = ty;\n  return;\n}\ncase ND_EQ:\ncase ND_NE:\ncase ND_LT:\ncase ND_LE:\n  usual_arith_conv(&node->lhs, &node->rhs);\n  node->ty = ty_int;\n  return;\n```\n\n`ND_NEG` (unary minus) uses the same machinery with a synthetic left operand of `ty_int`, which forces the integer-promotion rule. `-x` for `char x` produces a value of type `int`, not `char`.\n\nThe comparison operators (`==`, `!=`, `<`, `<=`) get `usual_arith_conv` to convert the operands but then explicitly set the result type to `ty_int`. Comparisons return `int` in C even if their operands are `long`.\n\n### `ND_NUM`'s type becomes value-dependent\n\nPreviously, every `ND_NUM` had type `ty_long` (the catch-all from \u00a710.2). That's wasteful \u2014 `1 + 2` shouldn't produce a `long` result on a machine where `int` arithmetic is available. After this commit:\n\n```c\ncase ND_NUM:\n  node->ty = (node->val == (int)node->val) ? ty_int : ty_long;\n  return;\n```\n\nIf the literal value fits in 32 bits, its type is `int`; otherwise `long`. This means `1 + 2` at the AST level has both operands as `int`, `usual_arith_conv` keeps both as `int`, and the result is `int`. `1 + 2147483648` (which doesn't fit in `int`) types the right operand as `long`, and the rule promotes the left to `long`, with the result as `long`.\n\n### Pointer arithmetic gets `new_long`\n\nTwo existing callers of `new_num` in pointer arithmetic \u2014 the multiplications-by-element-size in `new_add` and `new_sub` \u2014 switch to a new helper that always emits a `long`-typed literal:\n\n```c\nstatic Node *new_long(int64_t val, Token *tok) {\n  Node *node = new_node(ND_NUM, tok);\n  node->val = val;\n  node->ty = ty_long;\n  return node;\n}\n```\n\n```diff\n   // ptr + num\n-  rhs = new_binary(ND_MUL, rhs, new_num(lhs->ty->base->size, tok), tok);\n+  rhs = new_binary(ND_MUL, rhs, new_long(lhs->ty->base->size, tok), tok);\n```\n\nThe reason: pointer arithmetic should happen in 64-bit because pointers are 64 bits. If the size were a 32-bit `ND_NUM` (because most struct sizes fit in `int`), `usual_arith_conv` of `(int) * (long)` would correctly promote it, but it's cleaner \u2014 and one fewer cast in the AST \u2014 to make the size literal a `long` from the start.\n\n### `ND_ASSIGN` gets implicit conversion\n\n```c\ncase ND_ASSIGN:\n  if (node->lhs->ty->kind == TY_ARRAY)\n    error_tok(node->lhs->tok, \"not an lvalue\");\n  if (node->lhs->ty->kind != TY_STRUCT)\n    node->rhs = new_cast(node->rhs, node->lhs->ty);\n  node->ty = node->lhs->ty;\n  return;\n```\n\n`int x = 1L;` now inserts an `ND_CAST` to `int` on the right-hand side. The `if (node->lhs->ty->kind != TY_STRUCT)` exclusion preserves the byte-loop struct assignment from \u00a79.7 \u2014 struct assignment doesn't go through scalar casts.\n\n### Codegen tweak: char/short load extends to int\n\n`load` changes its small-type widths from \"extend to quad\" to \"extend to long\":\n\n```diff\n   if (ty->size == 1)\n-    println(\"  movsbq (%%rax), %%rax\");\n+    println(\"  movsbl (%%rax), %%eax\");\n   else if (ty->size == 2)\n-    println(\"  movswq (%%rax), %%rax\");\n+    println(\"  movswl (%%rax), %%eax\");\n   else if (ty->size == 4)\n     println(\"  movsxd (%%rax), %%rax\");\n   else\n     println(\"  mov (%%rax), %%rax\");\n```\n\nThe change matches the convention \u00a710.8 set up: 8- and 16-bit values arrive in `%eax`, sign-extended to 32 bits, with the upper 32 bits of `%rax` undefined. The 32-bit arithmetic in `gen_expr` reads from `%eax` and ignores `%rax`'s upper half. When a wider type is needed, `add_type` will already have inserted a `(long)x` cast, which `cast()` codegens as `movsxd %eax, %rax` \u2014 extending to 64 bits exactly when needed.\n\nThe new comment in `load` documents the contract:\n\n```c\n// When we load a char or a short value to a register, we always\n// extend them to the size of int, so we can assume the lower half of\n// a register always contains a valid value. The upper half of a\n// register for char, short and int may contain garbage. When we load\n// a long value to a register, it simply occupies the entire register.\n```\n\n### Tests\n\n`test/usualconv.c` is new:\n\n```c\nASSERT((long)-5, -10 + (long)5);\nASSERT((long)-15, -10 - (long)5);\nASSERT((long)-50, -10 * (long)5);\nASSERT((long)-2, -10 / (long)5);\n\nASSERT(1, -2 < (long)-1);\nASSERT(1, -2 <= (long)-1);\nASSERT(0, -2 > (long)-1);\nASSERT(0, -2 >= (long)-1);\n\nASSERT(0, 2147483647 + 2147483647 + 2);\nASSERT((long)-1, ({ long x; x=-1; x; }));\n```\n\n`-2 < (long)-1` is the signedness test. With `int` and `long` operands, the rule promotes the `int` to `long` (sign-extending), and `(long)-2 < (long)-1` is true. Without the rule, the comparison would happen at 32 bits, where `-2` and `-1` are both negative and the signed comparison still works \u2014 but the *result type* would be wrong. The comparison result is `int` regardless.\n\n`2147483647 + 2147483647 + 2` is the overflow test. `2147483647` is `INT_MAX`. Both operands are `int`, so the addition runs at 32 bits, overflows, and `INT_MAX + INT_MAX + 2 == 0` (modulo 2^32, accounting for `INT_MIN` and the `+2`). The test asserts that.\n\n`({ long x; x=-1; x; })` reads `x` after assigning `-1` (an `int` literal) to it. The implicit conversion in `ND_ASSIGN` casts `-1` to `long`, sign-extending to all-1s, and reading back gets `(long)-1`.\n\nA new test is also added to `test/sizeof.c`:\n\n```c\nASSERT(8, sizeof(-10 + (long)5));\nASSERT(8, sizeof(-10 - (long)5));\nASSERT(8, sizeof(-10 * (long)5));\nASSERT(8, sizeof(-10 / (long)5));\n```\n\n`sizeof` of an arithmetic expression returns the size of the operator's *result type*. After this commit, `int + long` has type `long`, so `sizeof` is 8.\n\n### Where we are\n\nMixed-width integer arithmetic produces the right type and the right value. Comparisons return `int`. Assignment narrows or widens implicitly. The `usual_arith_conv` machinery is used in five places (the four binary arithmetic operators and the four comparison operators), and `ND_NEG` and `ND_ASSIGN` use the underlying `get_common_type` and `new_cast` helpers directly.\n\nA small interpretive note: chibicc doesn't implement the full C99 *integer promotion* rules. The standard's full rule promotes any `_Bool`, `char`, or `short` to `int` (or to `unsigned int` if the value won't fit in `int`); it also has rules for `unsigned int` versus `long`. chibicc's `get_common_type` covers the most common case (the `size == 8 ? long : int` ladder), but doesn't engage with signedness or with the `unsigned int` versus `long` rule because chibicc doesn't have unsigned integers yet. Those arrive in Chapter 14.\n\n---\n\n## 10.11 \u2014 Function-call type conversions\n\n> `git checkout 9e211cbf1d459babf035fd6b3407c2bd184cb639` \u2014 *Report an error on undefined/undeclared functions*\n> `git checkout 818352acc07d0a982076b4b49345b42be706f5e1` \u2014 *Handle return type conversion*\n> `git checkout fdc80bc6b5faa058b88d838332c71b7101712896` \u2014 *Handle function argument type conversion*\n\nThree commits, in `main` order. The first wires up the lookup; the second uses it to convert return values; the third uses it to convert arguments.\n\n### Looking up the function (commit 69)\n\nBefore this commit, `funcall` parsed an identifier and a comma-separated argument list, never asking what the identifier referred to. The result was that calling an undefined function produced a link-time error rather than a parse-time one, and there was no way to type-check arguments because the parser had no `Type` for the function.\n\nThe fix consults the symbol table:\n\n```c\nVarScope *sc = find_var(start);\nif (!sc)\n  error_tok(start, \"implicit declaration of a function\");\nif (!sc->var || sc->var->ty->kind != TY_FUNC)\n  error_tok(start, \"not a function\");\n\nType *ty = sc->var->ty->return_ty;\n```\n\nTwo error cases: `sc == NULL` (the name isn't in scope at all) and `sc->var == NULL || not a function type` (the name is in scope but isn't a function \u2014 for example, `int x; x();`). Both produce parse errors.\n\nThe return type is read from the looked-up function and stored on the `ND_FUNCALL` node:\n\n```c\n*rest = skip(tok, \")\");\n\nNode *node = new_node(ND_FUNCALL, start);\nnode->funcname = strndup(start->loc, start->len);\nnode->ty = ty;\nnode->args = head.next;\nreturn node;\n```\n\nThis is what makes calls into a function returning `long` actually have type `long` at the call site, instead of the \u00a710.2 default of `ty_long` for *all* calls. (At this point the default still happens to match \u2014 `ty` is taken from `return_ty`, but `add_type`'s `ND_FUNCALL` case still hard-codes `ty_long`, and that wins because it runs after this assignment. Commit 71 will change that.)\n\nThe arguments also get `add_type` called on them, eagerly:\n\n```c\nwhile (!equal(tok, \")\")) {\n  if (cur != &head)\n    tok = skip(tok, \",\");\n  cur = cur->next = assign(&tok, tok);\n  add_type(cur);\n}\n```\n\nWithout `add_type`, the next commit's argument-conversion code would have no types to read from. Calling `add_type` here is preemptive.\n\nA small but important tokenizer/runtime change: `error_at` and `error_tok` no longer call `exit(1)` from within `verror_at`. The exit moves to the wrappers:\n\n```diff\n static void verror_at(int line_no, char *loc, char *fmt, va_list ap) {\n   ...\n-  exit(1);\n }\n\n void error_at(char *loc, char *fmt, ...) {\n   ...\n   verror_at(line_no, loc, fmt, ap);\n+  exit(1);\n }\n\n void error_tok(Token *tok, char *fmt, ...) {\n   ...\n   verror_at(tok->line_no, tok->loc, fmt, ap);\n+  exit(1);\n }\n```\n\nThe motivation is to allow `verror_at` to be called for warnings (or non-fatal errors) without aborting the compile. Nothing in this commit calls it that way, but the structural separation is there. The `// Reports an error message in the following format and exit.` comment also loses its \"and exit\" because `verror_at` no longer does that.\n\nThe test harness's `test/test.h` gets the `assert` declaration:\n\n```diff\n #define ASSERT(x, y) assert(x, y, #y)\n\n+void assert(int expected, int actual, char *code);\n int printf();\n```\n\nWithout this, every test file that uses `ASSERT` would now fail with \"implicit declaration of a function\" because the harness lookup would miss `assert`.\n\n### Return type conversion (commit 70)\n\n`return e;` previously emitted code for `e` and jumped to the function epilogue, ignoring whether `e`'s type matched the function's return type. After this commit, the return value is implicitly cast to the return type:\n\n```c\nif (equal(tok, \"return\")) {\n  Node *node = new_node(ND_RETURN, tok);\n  Node *exp = expr(&tok, tok->next);\n  *rest = skip(tok, \";\");\n\n  add_type(exp);\n  node->lhs = new_cast(exp, current_fn->ty->return_ty);\n  return node;\n}\n```\n\nThe conversion uses `new_cast`, which means the same instruction-table from \u00a710.9 generates the right narrowing/widening. Inside `char int_to_char(int x) { return x; }`, the `return x;` becomes `return (char)x;` \u2014 the parser inserts an `ND_CAST(int \u2192 char)`, codegen does nothing (no instruction needed for narrowing), and the function returns the low 8 bits in `%al`.\n\nA static `current_fn` tracks which function is being parsed, so the return type is reachable from inside `stmt()`:\n\n```c\nstatic Obj *current_fn;\n```\n\nIt's set in `function()` after the prototype is parsed:\n\n```c\ncurrent_fn = fn;\n```\n\nThere's already a `current_fn` in `codegen.c` for emitting the right return-label name. The parser's `current_fn` is a separate variable with the same name, in a different translation unit. (Whether to merge them is a code-cleanliness question Rui doesn't engage with here.)\n\nThe test exercises both narrowing and address returns:\n\n```c\nint g1;\nint *g1_ptr() { return &g1; }\nchar int_to_char(int x) { return x; }\n\nASSERT(3, *g1_ptr());\nASSERT(5, int_to_char(261));\n```\n\n`int_to_char(261)` truncates: `261 mod 256 = 5`. `g1_ptr()` returns the address of a global, which the caller dereferences.\n\n### Argument type conversion (commit 71)\n\nThe third commit walks the function's parameter list (now reachable via `sc->var->ty->params`) in lock-step with the argument list, casting each argument to the declared parameter type:\n\n```c\nType *ty = sc->var->ty;\nType *param_ty = ty->params;\n\nNode head = {};\nNode *cur = &head;\n\nwhile (!equal(tok, \")\")) {\n  if (cur != &head)\n    tok = skip(tok, \",\");\n\n  Node *arg = assign(&tok, tok);\n  add_type(arg);\n\n  if (param_ty) {\n    if (param_ty->kind == TY_STRUCT || param_ty->kind == TY_UNION)\n      error_tok(arg->tok, \"passing struct or union is not supported yet\");\n    arg = new_cast(arg, param_ty);\n    param_ty = param_ty->next;\n  }\n\n  cur = cur->next = arg;\n}\n```\n\nThe `if (param_ty)` guards against argument-list overflow \u2014 calling `printf(\"hello\", \"world\", 1, 2, 3)` with `int printf()` has no declared parameters, so the cast loop is skipped for the extra arguments. (This is the silent miscompile of \"more than 6 arguments to a variadic function\" we noted in \u00a75.4 as an errata candidate; it's not addressed here.)\n\nThe `TY_STRUCT`/`TY_UNION` check is a polite refusal: chibicc still doesn't handle struct or union arguments. Errors at parse time rather than producing wrong code.\n\nA `func_ty` field is added to `Node` so that codegen can also see the function type at the call site, in addition to the return type:\n\n```diff\n   // Function call\n   char *funcname;\n+  Type *func_ty;\n   Node *args;\n```\n\n```c\nnode->func_ty = ty;\nnode->ty = ty->return_ty;\n```\n\n`func_ty` isn't used yet in codegen. It will become important when chibicc starts caring about variadic functions and floating-point arguments \u2014 for now it's a forward-reference plant.\n\nA new test:\n\n```c\nint div_long(long a, long b) {\n  return a / b;\n}\n\nASSERT(-5, div_long(-10, 2));\n```\n\n`div_long(-10, 2)` is a call where `-10` and `2` are both `int` literals (their types are `ty_int` per \u00a710.10's `ND_NUM` rule), and the parameters are `long`. The argument cast inserts `(long)-10` and `(long)2`. Without the cast, `gen_expr` for the arguments would push 32-bit values onto the stack, the function would read 64-bit values from `%rdi` and `%rsi`, and the upper 32 bits would be garbage \u2014 producing wrong arithmetic when divided.\n\n### Where we are\n\nFunction calls are type-checked. Arguments are cast to parameter types; return values are cast to the function's declared return type. Calling an undeclared function is now an error. Struct and union arguments are explicitly rejected. The `func_ty` plant is laid for later codegen work.\n\nWhat's still missing: chibicc still doesn't *check* that the argument count matches the parameter count. Calling `int_to_char(1, 2, 3)` silently drops the extra arguments. A real C compiler would error.\n\n---\n\n## 10.12 \u2014 `_Bool`\n\n> `git checkout 44bba965cbe3827be2b68651e541b33fa040bb72` \u2014 *Add _Bool type*\n\n`_Bool` is an integer type with a special conversion rule: assigning any non-zero value to a `_Bool` produces 1, not the low bit. `_Bool x = 256;` yields `x == 1`, in contrast to `char x = 256;` where `x == 0` (because `256 mod 256 == 0`). The rule is what makes `_Bool` a real boolean rather than a tiny integer.\n\nThe commit message lays it out:\n\n> _Bool isn't just a 1-bit integer because when you convert a value to bool, the result is 1 if the original value is non-zero. This is contrary to the other small integral types, e.g. char, as you can see below:\n>\n>   char x  = 256; // x is 0\n>   _Bool y = 256; // y is 1\n\n`TY_BOOL` joins the type enum, sized 1 byte and aligned 1 byte (matching the C standard, which says `sizeof(_Bool) == 1`):\n\n```c\nType *ty_bool = &(Type){TY_BOOL, 1, 1};\n```\n\n`is_integer` adds it to the list. The keyword is added to the lexer, to `is_typename`, and to `declspec`. The `declspec` switch adds one new arm:\n\n```c\ncase BOOL:\n  ty = ty_bool;\n  break;\n```\n\nThe two-bit-per-keyword encoding gets a small renumbering \u2014 `BOOL` slots in between `VOID` and `CHAR`, and the existing constants shift. That's fine because nothing outside `declspec` looks at the values.\n\nThe interesting part is the codegen. `cast` gets a special arm:\n\n```c\nif (to->kind == TY_BOOL) {\n  cmp_zero(from);\n  println(\"  setne %%al\");\n  println(\"  movzx %%al, %%eax\");\n  return;\n}\n```\n\n`cmp_zero` is a new helper that compares `%rax` (or `%eax`, depending on width) against zero:\n\n```c\nstatic void cmp_zero(Type *ty) {\n  if (is_integer(ty) && ty->size <= 4)\n    println(\"  cmp $0, %%eax\");\n  else\n    println(\"  cmp $0, %%rax\");\n}\n```\n\nThen `setne %al` sets `%al` to 1 if the comparison was non-zero (sign and zero flags), 0 otherwise \u2014 exactly the `_Bool` conversion. `movzx %al, %eax` zero-extends `%al` to `%eax`, producing a clean 32-bit `0` or `1` value (not sign-extended garbage).\n\nThat's the whole `_Bool` conversion: compare to zero, set on not-equal, zero-extend. Three instructions. The result lives in the low byte of `%eax`, gets stored as a 1-byte value (because `_Bool` is 1 byte), and reads back as 0 or 1.\n\n### Tests\n\n```c\nASSERT(0, ({ _Bool x=0; x; }));\nASSERT(1, ({ _Bool x=1; x; }));\nASSERT(1, ({ _Bool x=2; x; }));\nASSERT(1, (_Bool)1);\nASSERT(1, (_Bool)2);\nASSERT(0, (_Bool)(char)256);\n```\n\nThe third line is the canonical case: `_Bool x = 2` assigns `2`, the `ND_ASSIGN` from \u00a710.10 inserts a cast to `_Bool`, codegen runs `cmp $0; setne; movzx`, and `1` is what gets stored. `(_Bool)(char)256` is the composition: `(char)256 = 0`, then `(_Bool)0 = 0`. Two casts, two table lookups, the composition produces `0`.\n\nFunction tests for parameter and return types:\n\n```c\n_Bool bool_fn_add(_Bool x) { return x + 1; }\n_Bool bool_fn_sub(_Bool x) { return x - 1; }\n\nASSERT(1, bool_fn_add(3));\nASSERT(0, bool_fn_sub(3));\n```\n\n`bool_fn_add(3)` casts `3` to `_Bool` (yielding `1`), then `1 + 1 = 2` happens at `int` width (per the integer-promotion rule from \u00a710.10), then the return cast back to `_Bool` turns `2` into `1`. Two `_Bool` conversions in a single call.\n\n### Where we are\n\n`_Bool` is a real type. Casts to `_Bool` produce 0 or 1, never any other value. The conversion is three instructions in codegen. Everything else \u2014 parameter passing, return values, comparisons \u2014 falls out of the existing machinery from \u00a710.9 and \u00a710.10.\n\n---\n\n## 10.13 \u2014 Character literals\n\n> `git checkout aa0accc75e9358d313fef0a6d4005103e2ce25f5` \u2014 *Add character literal*\n\n`'a'` becomes a tokenizer change. The token's `kind` is `TK_NUM`, its `val` is the integer value of the character, and the parser doesn't know the difference between `'a'` and `97`. That's exactly what the C standard says.\n\n`read_char_literal`:\n\n```c\nstatic Token *read_char_literal(char *start) {\n  char *p = start + 1;\n  if (*p == '\\0')\n    error_at(start, \"unclosed char literal\");\n\n  char c;\n  if (*p == '\\\\')\n    c = read_escaped_char(&p, p + 1);\n  else\n    c = *p++;\n\n  char *end = strchr(p, '\\'');\n  if (!end)\n    error_at(p, \"unclosed char literal\");\n\n  Token *tok = new_token(TK_NUM, start, end + 1);\n  tok->val = c;\n  return tok;\n}\n```\n\nThe tokenizer recognizes `'` and dispatches:\n\n```c\nif (*p == '\\'') {\n  cur = cur->next = read_char_literal(p);\n  p += cur->len;\n  continue;\n}\n```\n\n`read_escaped_char` was added in \u00a77.4 for string literals. It already handles `\\n`, `\\t`, `\\\\`, `\\'`, `\\\"`, octal escapes, and hex escapes \u2014 which means the same machinery now powers character escapes inside `'...'`. `'\\n'` gets handed to `read_escaped_char(&p, p + 1)` and comes back as `10`.\n\nThe token's `val` is a `char`, which on x86-64 Linux is signed 8-bit. So `'\\x80'` (the byte `0x80`) is stored as `-128`, sign-extended through `int64_t val`, which tests:\n\n```c\nASSERT(97, 'a');\nASSERT(10, '\\n');\nASSERT(-128, '\\x80');\n```\n\nIn a strict C, the type of a character constant is `int`, not `char` \u2014 but the *value* is the integer value of the character interpreted as `char`. chibicc represents the value through the existing `TK_NUM` path and never has an opportunity to disagree.\n\n### Where we are\n\nCharacter literals work. They tokenize as `TK_NUM`, leveraging the existing escape-sequence handling from \u00a77.4. The parser is unchanged.\n\n---\n\n## 10.14 \u2014 `enum`\n\n> `git checkout 48ba2656fecc646ec4eb7f943fa94b02ed9725c7` \u2014 *Add enum*\n\n`enum` introduces a new kind of type and a way to define named integer constants. The implementation is the chapter's third interesting symbol-table extension (after typedef and after struct tags), and it puts every namespace question chibicc has ever asked into focus.\n\n### Type\n\n`TY_ENUM` joins the type enum:\n\n```diff\n typedef enum {\n   TY_VOID,\n   TY_BOOL,\n   TY_CHAR,\n   TY_SHORT,\n   TY_INT,\n   TY_LONG,\n+  TY_ENUM,\n   TY_PTR,\n   ...\n```\n\nA constructor:\n\n```c\nType *enum_type(void) {\n  return new_type(TY_ENUM, 4, 4);\n}\n```\n\nSized 4, aligned 4 \u2014 chibicc represents enum values as `int`s. Strict C says implementations may choose any integer type that fits all the values; chibicc just picks `int`. (`is_integer` is updated to include `TY_ENUM`, so enum values can be used in arithmetic and comparisons.)\n\n### Storage in the symbol table\n\nAn enum constant is bound to a name and a value. `VarScope` already has slots for variable, typedef name; it grows two more for enum:\n\n```diff\n struct VarScope {\n   VarScope *next;\n   char *name;\n   Obj *var;\n   Type *type_def;\n+  Type *enum_ty;\n+  int enum_val;\n };\n```\n\n`enum_ty` is the enum type the constant belongs to (so `enum Color { RED, GREEN, BLUE };` lets `RED` know its type is `enum Color`). `enum_val` is the integer value.\n\nEnum *tags* (the `Color` in `enum Color`) live in the `tags` namespace from \u00a79.4 \u2014 the same one that holds struct and union tags. C99 says struct, union, and enum tags share a single namespace per scope (\u00a76.2.3), and chibicc gets this right because all three reach the tag scope through `push_tag_scope` and `find_tag`. (The Ch 9 prose treated struct/union tag-sharing as an errata candidate; that was a misreading on our part \u2014 sharing the tag namespace across struct/union/enum is exactly what the standard requires. We'll quietly skip restating it as a wart in this section.)\n\n### `enum_specifier`\n\n```c\n// enum-specifier = ident? \"{\" enum-list? \"}\"\n//                | ident (\"{\" enum-list? \"}\")?\n//\n// enum-list      = ident (\"=\" num)? (\",\" ident (\"=\" num)?)*\nstatic Type *enum_specifier(Token **rest, Token *tok) {\n  Type *ty = enum_type();\n\n  Token *tag = NULL;\n  if (tok->kind == TK_IDENT) {\n    tag = tok;\n    tok = tok->next;\n  }\n\n  if (tag && !equal(tok, \"{\")) {\n    Type *ty = find_tag(tag);\n    if (!ty)\n      error_tok(tag, \"unknown enum type\");\n    if (ty->kind != TY_ENUM)\n      error_tok(tag, \"not an enum tag\");\n    *rest = tok;\n    return ty;\n  }\n\n  tok = skip(tok, \"{\");\n\n  int i = 0;\n  int val = 0;\n  while (!equal(tok, \"}\")) {\n    if (i++ > 0)\n      tok = skip(tok, \",\");\n\n    char *name = get_ident(tok);\n    tok = tok->next;\n\n    if (equal(tok, \"=\")) {\n      val = get_number(tok->next);\n      tok = tok->next->next;\n    }\n\n    VarScope *sc = push_scope(name);\n    sc->enum_ty = ty;\n    sc->enum_val = val++;\n  }\n\n  *rest = tok->next;\n\n  if (tag)\n    push_tag_scope(tag, ty);\n  return ty;\n}\n```\n\nTwo cases. With a `{` body, parse the enum list, push each name into the variable scope (with `enum_ty` and `enum_val` set), and optionally bind the tag name to the new type. Without a `{` body \u2014 just `enum Foo` \u2014 look up `Foo` in the tag scope, expect a `TY_ENUM`, return that. The `tag && !equal(tok, \"{\")` early-exit is the \"use existing tag\" path.\n\n`val` starts at zero and increments after each entry, with `=` overriding it for the next entry. So:\n\n```c\nenum { zero, five=5, three=3, four };\n```\n\nbinds `zero=0` (initial), `five=5` (override), `three=3` (override), `four=4` (incrementing past `three`). The test:\n\n```c\nASSERT(0, ({ enum { zero, five=5, three=3, four }; zero; }));\nASSERT(5, ({ enum { zero, five=5, three=3, four }; five; }));\nASSERT(3, ({ enum { zero, five=5, three=3, four }; three; }));\nASSERT(4, ({ enum { zero, five=5, three=3, four }; four; }));\n```\n\n### Wiring to `declspec` and `is_typename`\n\n`declspec` learns to dispatch on `enum`:\n\n```diff\n     Type *ty2 = find_typedef(tok);\n-    if (equal(tok, \"struct\") || equal(tok, \"union\") || ty2) {\n+    if (equal(tok, \"struct\") || equal(tok, \"union\") || equal(tok, \"enum\") || ty2) {\n       if (counter)\n         break;\n\n       if (equal(tok, \"struct\")) {\n         ty = struct_decl(&tok, tok->next);\n       } else if (equal(tok, \"union\")) {\n         ty = union_decl(&tok, tok->next);\n+      } else if (equal(tok, \"enum\")) {\n+        ty = enum_specifier(&tok, tok->next);\n       } else {\n         ty = ty2;\n         tok = tok->next;\n       }\n       ...\n```\n\n`is_typename` adds `\"enum\"` to its keyword list. The lexer adds it. Same as `typedef` (\u00a710.6), nothing structurally new.\n\n### Reading an enum constant\n\nIn `primary`, when the parser sees an identifier, it looks it up in `find_var`. The lookup may now return a `VarScope` whose `var` is null but whose `enum_ty` is set, meaning \"this is an enum constant.\" The new arm:\n\n```c\n// Variable or enum constant\nVarScope *sc = find_var(tok);\nif (!sc || (!sc->var && !sc->enum_ty))\n  error_tok(tok, \"undefined variable\");\n\nNode *node;\nif (sc->var)\n  node = new_var_node(sc->var, tok);\nelse\n  node = new_num(sc->enum_val, tok);\n\n*rest = tok->next;\nreturn node;\n```\n\nIf `sc->var` is set, the name is a variable \u2014 make an `ND_VAR` node. If `sc->enum_ty` is set, the name is an enum constant \u2014 make an `ND_NUM` node with the constant's value. Either way, the AST that comes out is one chibicc has been compiling for nine chapters.\n\n### Tests\n\n```c\nASSERT(0, ({ enum { zero, one, two }; zero; }));\nASSERT(1, ({ enum { zero, one, two }; one; }));\nASSERT(2, ({ enum { zero, one, two }; two; }));\nASSERT(5, ({ enum { five=5, six, seven }; five; }));\nASSERT(6, ({ enum { five=5, six, seven }; six; }));\nASSERT(4, ({ enum { zero, one, two } x; sizeof(x); }));\nASSERT(4, ({ enum t { zero, one, two }; enum t y; sizeof(y); }));\n```\n\n`sizeof(x)` for `enum { zero, one, two } x;` is 4 because `enum_type()` builds a 4-byte type. The two-statement `enum t { ... }; enum t y;` declares the tag in the first statement and uses it in the second.\n\n### Where we are\n\n`enum` works. Tags live with struct and union tags (correctly \u2014 all three share the C-standard's tag namespace). Constants live in the variable namespace, alongside ordinary identifiers and typedef names (also correctly). The `VarScope` entry has gained two more fields. The `declspec` rewrite from \u00a710.5 is the framework that enum slots into.\n\n---\n\n## 10.15 \u2014 File-scope `static` functions\n\n> `git checkout 736232f3d672dae9a1ddae800909204c17fbe37c` \u2014 *Support file-scope functions*\n\nThe `static` keyword on a function. In C, file-scope `static` means *internal linkage* \u2014 the function's name is not visible outside its translation unit, so the linker won't see it and won't conflict with similarly-named functions in other files.\n\nThe implementation rides through the same `VarAttr` channel as `typedef`:\n\n```diff\n typedef struct {\n   bool is_typedef;\n+  bool is_static;\n } VarAttr;\n```\n\n`declspec` handles both keywords symmetrically:\n\n```c\nif (equal(tok, \"typedef\") || equal(tok, \"static\")) {\n  if (!attr)\n    error_tok(tok, \"storage class specifier is not allowed in this context\");\n\n  if (equal(tok, \"typedef\"))\n    attr->is_typedef = true;\n  else\n    attr->is_static = true;\n\n  if (attr->is_typedef + attr->is_static > 1)\n    error_tok(tok, \"typedef and static may not be used together\");\n  tok = tok->next;\n  continue;\n}\n```\n\nThe `(is_typedef + is_static) > 1` check is a polite refusal: both flags can't be set at the same time, because the C standard says a single declaration can have at most one storage-class specifier. (`extern` is a separate flag added in Chapter 13; with three flags, the same check generalizes.) The keyword is added to the lexer and to `is_typename`.\n\n`function()` reads the static flag from the attribute:\n\n```diff\n-static Token *function(Token *tok, Type *basety) {\n+static Token *function(Token *tok, Type *basety, VarAttr *attr) {\n   Type *ty = declarator(&tok, tok, basety);\n\n   Obj *fn = new_gvar(get_ident(ty->name), ty);\n   fn->is_function = true;\n   fn->is_definition = !consume(&tok, tok, \";\");\n+  fn->is_static = attr->is_static;\n```\n\n`Obj` gets a third boolean:\n\n```diff\n   bool is_function;\n   bool is_definition;\n+  bool is_static;\n```\n\nAnd codegen emits `.local` instead of `.globl` for static functions:\n\n```c\nif (fn->is_static)\n  println(\"  .local %s\", fn->name);\nelse\n  println(\"  .globl %s\", fn->name);\n```\n\n`.globl` makes the symbol visible to the linker; `.local` does not. After this commit, `static int foo(){...}` produces an object file whose `foo` symbol is internal and won't collide with another `foo` defined in another file.\n\n### Tests\n\n`test/common` (the harness, already in C as of \u00a78.2) gets a static helper:\n\n```c\nstatic int static_fn() { return 5; }\n```\n\nAnd `test/function.c` gets a colliding one:\n\n```c\nstatic int static_fn() { return 3; }\n\nASSERT(3, static_fn());\n```\n\nBoth files compile and link cleanly. Each file's `static_fn` is internal; the names don't collide. The test calls the local one and expects `3`.\n\n### Where we are\n\n`static` works on file-scope functions. The `VarAttr` machinery generalizes one flag further. Linkage \u2014 which is mostly Chapter 13's domain \u2014 gets its first chibicc encounter here.\n\n---\n\n## Recap\n\nTwenty commits, fifteen sections, one concept interlude. The net change: chibicc's three integer types and one pointer type become eight integer types (`_Bool`, `char`, `short`, `int`, `long`, plus the `long long` alias and the `long int`/`int long` alternative spellings, plus `enum`). Function types acquire return and parameter conversion. A cast operator and an instruction-level cast table get the work right at runtime. A `typedef` machinery puts type-name aliasing in the symbol table. `static` on functions handles linkage. The parser handles every C declaration syntax a working programmer is likely to write.\n\nTwenty commits is too many to summarize as a single table \u2014 the chapter has too much ground per commit. Instead, two tables, by theme:\n\n| Commit | Topic |\n|---|---|\n| `5831eda` | `int` width changes from 8 to 4. Pre-factor for everything else; ships `argreg32`, `movsxd`, `mov %eax`, and `store_gp`. |\n| `43c2f08` | `long` type \u2014 8 bytes. Token/AST `val` widens to `int64_t`. |\n| `9d48eef` | `short` type \u2014 2 bytes. `argreg16` and the `movswq`/`mov %ax` paths. |\n| `a817b23` | Nested declarators: `int (*x)[3]` and friends. The double-walk-with-dummy-type idiom. |\n| `74e3acc` | Function declaration as separate from definition. `is_definition` flag on `Obj`. |\n| `8c3503b` | `void` type. Variable-declared-`void` and void-pointer-deref errors. `is_typename` becomes a table-and-loop. |\n| `287906a` | `declspec` rewrite handling `int long`, `long int`, etc. Bit-field counter + switch. |\n| `f46370e` | `long long` and `long long int` as aliases for `long`. Two new switch arms. |\n| `a6b82da` | `typedef`. `VarScope.type_def`, `VarAttr.is_typedef`, `parse_typedef`. `is_typename` becomes context-sensitive. |\n| `67543ea` | `sizeof(typename)`. `abstract_declarator` and `typename` helpers. |\n\n| Commit | Topic |\n|---|---|\n| `cb81a37` | 32-bit register arithmetic for non-`long`, non-pointer operands. `cqo`/`cdq` split for `idiv`. |\n| `cfc4fa9` | `(type) expr` casts. `ND_CAST` node, `cast_table` codegen, `enum { I8, I16, I32, I64 }`. |\n| `8b430a6` | Usual arithmetic conversion. `get_common_type`, `usual_arith_conv`, integer promotion at every binary operator. `ND_NUM` typed by value range. `load` extends to 32 bits. |\n| `9e211cb` | Function-call lookup. Errors for \"implicit declaration\" and \"not a function.\" `error_at` no longer exits from inside `verror_at`. |\n| `818352a` | Return type conversion. `current_fn` in the parser. |\n| `fdc80bc` | Argument type conversion. `Node.func_ty`. Struct/union arguments rejected. |\n| `44bba96` | `_Bool` type. Cast-to-bool emits `cmp $0; setne; movzx`. |\n| `aa0accc` | Character literal `'a'` via the existing escape-sequence machinery. |\n| `48ba265` | `enum` type. Tag namespace shared with struct/union; constants in the variable namespace. |\n| `736232f` | File-scope `static`. `VarAttr.is_static`, `Obj.is_static`, `.local` directive. |\n\nThree structural shifts deserve repeating:\n\nThe first is the *bit-field-counter `declspec`*. Between commits 62 and 63 the parser's type-specifier handler became a small state machine that accepts every legal combination of C type-specifier and storage-class keywords. Every type or storage-class addition for the rest of the book \u2014 `_Bool`, `enum`, `static`, `extern`, `signed`, `unsigned`, `const`, `volatile` \u2014 slots into this framework by allocating two more bits and adding switch arms.\n\nThe second is the *context-sensitive `is_typename`*. After `typedef`, the question \"does this token start a type\" depends on the symbol table. The parser's distinction between declaration and statement now consults scope. This is the standard C lexer-versus-parser hack \u2014 sometimes known as \"the lexer hack\" \u2014 except that chibicc puts it in the parser by making `is_typename` look up the identifier in the current scope chain. (Real C compilers usually do the same thing; the alternative, marking typedef tokens at lex time, requires the lexer to know about scope, which is uglier.)\n\nThe third is the *cast machinery*. Once `ND_CAST` and `new_cast` exist, the type-checker uses them to insert implicit conversions everywhere: usual-arithmetic-conversion at binary operators, return-value conversion, argument-conversion, assignment-conversion. The codegen side is a 4\u00d74 table plus a `_Bool` special case. Chapter 14's signed/unsigned types and Chapter 15's floating-point types both extend the table.\n\nA few standing notes carried forward to Chapter 11:\n\n- The *canonicalization-at-parse-time* count from \u00a79.5 was six (five strict desugarings plus the `({...})` delegation). This chapter adds none directly. The cast insertions in \u00a710.10 and \u00a710.11 are *type-level* rewrites \u2014 not desugarings into more-primitive AST shapes \u2014 so we don't count them as canonicalizations. Chapter 11's `+=` family is the next canonicalization candidate.\n- The *pre-factor-before-feature* count was three. This chapter's \u00a710.1 (`int` \u2192 32-bit) is the fourth, and is a long-running case: the codegen catch-up in \u00a710.8 is ten commits later. The \u00a710.6 `parse_typedef` and \u00a710.6 `VarAttr` are arguably pre-factors for \u00a710.15's `static` handling, but they ship together with their first feature so they don't count as separate.\n- The *everything-fits-in-rax* invariant (broken in \u00a79.7 for struct/union) gets refined here. After \u00a710.10, the rule is: scalar values smaller than 8 bytes live in `%eax` with the low bits valid and the upper bits undefined; scalar values of size 8 live in `%rax`; struct and union \"values\" live at addresses pointed to by `%rax`. Three regimes, with the cast machinery moving values between them.\n- The *`is_typename` symbol-table coupling* (\u00a710.6) is the chapter's structural shift. Watch for further interactions with scope when Chapter 11 adds `goto` (which has its own label namespace) and Chapter 13 adds `extern` (which adds another `VarAttr` flag).\n- The struct/union/enum *tag namespace* is one namespace per the C standard. The Ch 9 framing of struct/union tag-sharing as an errata candidate was wrong; quietly replaced by the correct C99 rule in \u00a710.14.\n\nForward references for Chapter 11:\n\n- `+=`, `-=`, `*=`, etc. are likely to canonicalize at parse time using the *generalized-lvalue comma operator* extension from \u00a78.5.\n- `++` and `--` have similar canonicalization-or-lowering questions.\n- `?:` (the conditional operator) will use `usual_arith_conv` from \u00a710.10.\n- `goto` and labels introduce a fourth namespace (labels are function-scoped, not block-scoped, in C) and may interact with typedef names.\n- `switch` builds on the `enum` plumbing only loosely \u2014 a `case` value is a constant expression, which is a separate parsing concept from enum constants.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/10-filling-out-the-type-system.md",
  "description": "Word count for chapter 10"
}


---

## 🤖 Claude

14,814 words — squarely in the target band. Now the session 011 README and the HANDOFF for session 012.

---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 010 README and prior chapter for voice",
      "activeForm": "Reading session 010 README and prior chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 20 commits (56-75) via git show --stat",
      "activeForm": "Scanning 20 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading commit diffs in detail",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at commit 736232f",
      "activeForm": "Reading destination state files",
      "status": "completed"
    },
    {
      "content": "Draft chapters/10-filling-out-the-type-system.md",
      "activeForm": "Drafting Chapter 10",
      "status": "completed"
    },
    {
      "content": "Write session 011 README",
      "activeForm": "Writing session 011 README",
      "status": "in_progress"
    },
    {
      "content": "Write HANDOFF.md for session 012 (Chapter 11)",
      "activeForm": "Writing handoff for session 012",
      "status": "pending"
    },
    {
      "content": "Update CLAUDE.md status line for Ch 10 done",
      "activeForm": "Updating CLAUDE.md status",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/011-chapter-10-draft",
  "description": "Create session 011 directory"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/011-chapter-10-draft/README.md",
  "content": "# Session 011 \u2014 Chapter 10 draft\n\n**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002\u2013010).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 010 delivered Ch 9 (Structs and unions, seven commits, ~9,300 words). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 10 covers commits 56\u201375: the type system fill-out \u2014 `int` becomes 32 bits, `long`/`short`/`void`/`_Bool`, nested declarators, function declarations, complex declspec, `long long`, `typedef`, `sizeof(typename)`, 32-bit register usage, casts, the usual arithmetic conversion, function-call type conversion, character literals, `enum`, file-scope `static`. Twenty commits \u2014 the largest commit count of any chapter in the book.\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~14,800 words. Largest chapter so far. Target was 14,000\u201316,000 per the handoff, hit the middle of the band. Compares to Ch 7 (~13,800 \u2014 previous record), Ch 9 (~9,300), Ch 8 (~7,400). Per-section budget skewed toward the structural-shift sections: \u00a710.5 (declspec rewrite), \u00a710.6 (typedef), \u00a710.10 (usual arithmetic conversion), \u00a710.11 (function-call conversions), \u00a710.14 (enum) each ran 800\u20131,400 words; the smaller \"add a type\" sections (\u00a710.13 character literals, \u00a710.15 static functions) ran 200\u2013400.\n- **Section structure:** 15 sections plus one concept interlude. Followed the handoff's bundling proposal closely with a few adjustments:\n  - \u00a710.2 bundled commits 57, 58, 61 (`long`, `short`, `void`) \u2014 three same-shape additions, walked one (`long`) in detail and noted what differs for `short` and `void`. Per handoff.\n  - \u00a710.5 bundled commits 62, 63 (complex declspec + `long long` alias). Per handoff. The bit-field-counter mechanism is the centerpiece; the `long long` extension is two switch arms.\n  - \u00a710.11 bundled commits 69, 70, 71 (undeclared-function error, return-type conversion, argument-type conversion). Per handoff. The three commits form one logical arc \u2014 function-call type-checking \u2014 and reading them as a unit lets the prose explain why `9e211cb` adds the symbol-table lookup that `818352a` and `fdc80bc` use.\n  - Did **not** bundle \u00a710.13 (character literals) into \u00a710.6 (typedef) or \u00a710.7 (sizeof typename). The handoff floated folding the small commits if the chapter became too long; it didn't.\n- **Concept interlude placed between \u00a710.2 and \u00a710.3** \u2014 directly before the nested-declarator commit (\u00a710.3 = commit 59). The interlude introduces \"how to read a C type declaration\" with the spiral rule and a worked-out table of nine progressively gnarlier declarations. Runs ~600 words on its own; the centerpiece is the `int (*x)[10]` vs. `int *x[10]` vs. `int (*x)(int, int)` triple, walked through identifier-outward.\n- **\u00a710.1 is framed as a long-running pre-factor.** The handoff's framing said the codegen catch-up \"doesn't happen until commit 66\"; that's slightly inaccurate. Commit 56 already adds `argreg32`, `movsxd`, `mov %eax`, and `store_gp` \u2014 the load/store/parameter-pass machinery for a 4-byte type. Commit 66 is a *separate* arithmetic-codegen change (using `%eax`/`%edi` instead of `%rax`/`%rdi` for non-`long`, non-pointer operands). The chapter calls \u00a710.1 the fourth instance of pre-factor-before-feature, with the *arithmetic* catch-up arriving in \u00a710.8 ten commits later.\n- **\u00a710.10 (usual arithmetic conversion) is the chapter's most concept-heavy section.** Ran ~1,400 words and pulls together five threads: `get_common_type` and the rules; `usual_arith_conv` insertion at six binary-operator types; the `ND_NUM` value-dependent type rule; `new_long` for pointer-arithmetic constants; the `load` widening change from \"extend to quad\" to \"extend to long\" (and the documenting comment). Includes a small interpretive note that chibicc's rules are simpler than C99's full integer-promotion rules \u2014 chibicc has no unsigned types yet (Ch 14).\n- **\u00a710.14 (enum) corrects a Ch 9 errata.** The Ch 9 prose flagged \"struct and union tags share a namespace\" as an errata candidate \u2014 actually this is C-correct (C99 \u00a76.2.3: struct, union, and enum tags share a single namespace per scope). The Ch 10 \u00a710.14 prose includes one sentence quietly correcting course without re-litigating the Ch 9 framing: \"(The Ch 9 prose treated struct/union tag-sharing as an errata candidate; that was a misreading on our part \u2014 sharing the tag namespace across struct/union/enum is exactly what the standard requires. We'll quietly skip restating it as a wart in this section.)\" The recap also names this as a quiet correction.\n- **Date-vs-position note in the intro.** Roughly seven of the twenty commits are dated August 2019; the rest fall between March and September 2020. Notably commit 56 (`5831eda`, dated 2020-09) sits in `main` order *before* the August-2019 commits, which means in chronological terms the August-2019 commits were originally written when `int` was still 8 bytes \u2014 and were rewritten or re-tested when Rui changed the size. Same intro pattern as Chs 7, 8, 9.\n- **Diff format** matches Chs 7\u20139: inline diff fragments where the change is small, full quoted snippets where a function is new or substantially rewritten. The \u00a710.5 declspec rewrite is shown as the full new function (it's the chapter's centerpiece). The \u00a710.6 typedef parser-side changes are a series of inline diff fragments and full snippets of `parse_typedef` and the `compound_stmt` integration. The \u00a710.9 cast table is shown in full because its 4\u00d74 structure is the explanation. The \u00a710.14 `enum_specifier` is shown in full because the two-case structure (with-body, without-body) is the explanation.\n- **Two tables in the recap** \u2014 the handoff predicted twenty rows would be too many for one table, and that proved right. Split into two ten-row tables, themed around \"types and parsing\" vs. \"casts, conversions, and codegen.\" Reading either as a vertical scan is manageable.\n\n### Interpretive calls\n\n1. **Counting pre-factor-before-feature instances.** The handoff suggested \u00a710.1 was the fourth instance. Ran with that. Did *not* count \u00a710.6's `VarAttr` introduction or \u00a710.6's `parse_typedef` factoring as a fifth pre-factor for \u00a710.15's `static`, because those changes ship in the same commit as their first user (`typedef`). The pattern's strict version is \"pre-factor in commit N, feature using it in commit M > N\"; mid-commit refactor-then-extend doesn't qualify.\n2. **Counting canonicalization-at-parse-time instances.** The handoff suggested Ch 10 might add none. Ran with that. The cast insertions in \u00a710.10 (usual arithmetic conversion), \u00a710.11 (return/argument conversion), and \u00a710.10's `ND_ASSIGN` rule are *type-level* rewrites \u2014 they insert `ND_CAST` nodes but don't desugar one AST shape into a more-primitive shape. The recap explicitly notes this distinction. Count remains at six.\n3. **The `is_typename` framing.** Called the \u00a710.6 typedef-handling change \"the chapter's third interesting symbol-table extension\" (after typedef and after struct tags), and \"the standard C lexer-versus-parser hack \u2014 sometimes known as 'the lexer hack'.\" The recap quotes the standard name because readers familiar with C compiler folklore will recognize it; explained in-line for those who don't.\n4. **The cast table's elision of char-or-short \u2192 64.** Worked out the 4\u00d74 cast table's logic from first principles in the prose (rather than just quoting it), because the elision of char/short \u2192 long is non-obvious but follows from the load-with-sign-extension-to-int convention. Future reference for the \u00a710.10 prose.\n5. **The Ch 9 errata correction.** Done as one sentence, in \u00a710.14, without re-litigating. Did not retroactively edit Ch 9's text because the user has said to continue forward. Will add to the carry-forward notes in this README that Ch 9 has an errata candidate of its own (the errata-candidate flag was the wrong call).\n\n### Voice / structure inherited from Ch 1\u20139\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for bundled sections).\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature tables (two tables this chapter \u2014 twenty rows split by theme).\n\n### Two careful avoidances\n\n- **Did not over-explain the `(*x)[3]` vs. `*x[3]` reading.** The concept interlude does the work in one place; \u00a710.3 trusts the reader and shows the parser code without re-deriving the spiral rule.\n- **Did not re-explain `add_type` from scratch in \u00a710.10.** The \u00a76.4 prose introduced `add_type`; \u00a710.10 names the new arms and walks the new logic, but doesn't re-walk the recursive descent through children. The reader has it from Ch 6.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **The `is_typename` symbol-table coupling** is the chapter's biggest structural shift. After \u00a710.6, the parser's distinction between declaration and statement consults the symbol table at every point where it asks \"does this start a type.\" The standard C lexer-versus-parser hack lives here. Watch for further interactions when Ch 11 adds `goto`/labels (which have their own fourth namespace) and Ch 13 adds `extern` (which adds another `VarAttr` flag through the same channel).\n- **The bit-field-counter `declspec`** (\u00a710.5) is the framework everything that adds a type-specifier or storage-class keyword slots into. `_Bool` (\u00a710.12), `static` (\u00a710.15), and Chapter 11+ extensions (`extern`, `signed`, `unsigned`, `const`, `volatile`) all walk through this.\n- **The cast machinery** (\u00a710.9 + \u00a710.10) is now the type system's load-bearing core. Casts get inserted everywhere \u2014 usual arithmetic conversion, return values, arguments, assignments. The 4\u00d74 `cast_table` plus the `_Bool` special case is the entire codegen surface. Ch 14's signedness and Ch 15's floats both extend the table.\n- **Pre-factor-before-feature count is now four.** \u00a76.5, \u00a77.6, \u00a78.3, \u00a710.1. The \u00a710.1 instance is unusually long-running \u2014 ten commits between pre-factor and codegen catch-up.\n- **Canonicalization-at-parse-time count remains at six.** Ch 11's `+=` family is the next likely instance \u2014 probably several at once, as `a += b` lowers via the \u00a78.5 generalized-lvalue comma extension.\n- **The everything-fits-in-rax invariant** is now a three-regime rule: scalar values < 8 bytes live in `%eax` with low bits valid and upper bits undefined; scalar values of size 8 live in `%rax`; struct/union \"values\" live at addresses pointed to by `%rax`. Cast machinery moves values between regimes.\n- **The argreg split is complete.** \u00a77.2 introduced `argreg8`/`argreg64`; \u00a710.1 added `argreg32`; \u00a710.2 added `argreg16`. All four widths exist; `store_gp` dispatches.\n- **The `is_typename` helper is now a context-sensitive predicate.** Returns true for keyword type-specifiers (lookup in `kw[]`) or for typedef names (lookup in symbol table). The dual structure persists.\n- **The `VarAttr` channel** is the carrier for storage-class specifiers. Currently `is_typedef` and `is_static`. Ch 13's `extern` adds a third flag.\n- **The `unreachable()` macro** lives in `chibicc.h` and aborts the compile with file-and-line. Used by `store_gp` (\u00a710.1) and the `declspec` arm-counter `unreachable()` arm (\u00a710.5). Will get more callers.\n- **The `error_at`/`error_tok` exit-from-wrapper** change in \u00a710.11 (commit 69) is a small architectural shift: `verror_at` no longer aborts. Nothing yet calls `verror_at` for warnings, but the structural separation is there.\n- **The `Node.func_ty` field** (\u00a710.11, commit 71) is laid for later codegen work. Ch 14 (variadics) and Ch 15 (floating-point arguments) will use it.\n- **The `current_fn` static in `parse.c`** (\u00a710.11, commit 70) is separate from the same-named `current_fn` in `codegen.c` \u2014 different translation units. Whether to merge is a code-cleanliness question we don't engage with.\n- **The Ch 9 errata-candidate \"struct and union tags share a namespace\"** is not actually errata. C99 says struct, union, and enum tags share one namespace per scope (\u00a76.2.3), and chibicc gets this right. \u00a710.14 quietly corrects course; the carry-forward errata list should drop this entry.\n- **The struct-and-union-tag wart** in the standing notes from Sessions 008\u2013010 is also not a wart \u2014 see above.\n- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.\n- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 \u00a75.1. Pending footnote for revision pass.\n- **The \"more than 6 args silently miscompiles\"** in Ch 5 \u00a75.4. Errata candidate. Note that \u00a710.11's argument-conversion code does *not* fix this \u2014 when a call has more arguments than the declared parameter list, the loop's `if (param_ty)` guard skips the cast for the extras, but the codegen still pushes them and the callee still reads them. The check in \u00a710.11 catches *implicit declaration of a function* and *not a function*, but doesn't catch *too many arguments*.\n- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 \u00a77.5) rejects void-returning bodies. Errata candidate.\n- **The hex-escape silent truncation** (Ch 7 \u00a77.4). Errata candidate.\n- **The redeclaration-in-same-scope check** is still missing for variables (Ch 8 \u00a78.1) and tags (Ch 9 \u00a79.4). Both errata candidates. \u00a710.6 typedef adds another opportunity for this \u2014 `typedef int x; typedef int x;` is silently accepted in chibicc; not tested.\n- **Chapter 7's mention of commit hash `46c75e7`** for the precompute commit is wrong (actual is `6647ad9`). Errata for the revision pass.\n- **The `add_type` rule for binary operators changed twice** \u2014 first in \u00a710.2 (long becomes the catch-all type), then in \u00a710.10 (split into per-operator arms with `usual_arith_conv`). The \u00a710.2 catch-all is short-lived; the \u00a710.10 split is the durable structure.\n- **`enum_type()` returns a fresh type each call.** No caching. Two `enum { A };` declarations in different scopes get different `Type *` instances, even though they're structurally identical. Unlike `ty_int` etc. which are shared globals.\n\n## Exit state\n\n- `chapters/10-filling-out-the-type-system.md` drafted, ~14,800 words.\n- Session 011 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 012 (Chapter 11 \u2014 All the operators, commits 76\u201396; twenty-one commits, the second-largest chapter by commit count).\n- CLAUDE.md status note will need updating (chapter count goes from \"Ch 9 drafted\" to \"Ch 10 drafted\").\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && git log --oneline 736232f..HEAD | tail -25 | head -25",
  "description": "List commits 76+ for Ch 11"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline --reverse 736232f..HEAD | head -25",
  "description": "List commits 76+ in reverse"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline --reverse 736232f..HEAD | head -21",
  "description": "Confirm commits 76\u201396 for Ch 11"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/011-chapter-10-draft/HANDOFF.md",
  "content": "# Handoff: Ch 10 done \u2192 proceed to Ch 11\n\n**For:** the next claude session.\n**From:** session 011.\n**Status:** Ch 10 drafted (~14,800 words, twenty commits, the largest chapter so far). Continue autonomously to Ch 11 (All the operators). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/011-chapter-10-draft/README.md`](README.md)** \u2014 what session 011 did, including the seven-then-revised structural shifts in \u00a710.5/\u00a710.6/\u00a710.10/\u00a710.11/\u00a710.14, the framing of \u00a710.1 as a long-running pre-factor, and the quiet correction of the Ch 9 errata about struct/union tag namespaces.\n2. **[`docs/sessions/010-chapter-09-draft/HANDOFF.md`](../010-chapter-09-draft/HANDOFF.md)** \u2014 the previous handoff. Many standing notes still apply, but two updates: (a) the struct-and-union-tag-namespace \"wart\" is **not** a wart \u2014 it's C-correct (struct/union/enum tags share one namespace per scope per C99 \u00a76.2.3); (b) `is_typename` has now become a context-sensitive predicate that consults the symbol table.\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`10-filling-out-the-type-system.md`](../../../chapters/10-filling-out-the-type-system.md)** \u2014 the ten chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 11 = commits 76\u201396.\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes.\n\n## Chapter 11 scope\n\n**Title (working):** *All the operators*.\n**Commits:** 76\u201396 in chronological order on `main`. **Twenty-one commits \u2014 second-largest chapter by commit count, just behind Ch 10.**\n**Concept interlude:** None called out by the chapter mapping. The chapter is operator-by-operator and doesn't have a single concept that needs one. *Possibly* an interlude on `goto` and structured programming if the prose calls for it; default to no interlude.\n\n| # | Hash | Subject |\n|---|---|---|\n| 76 | `a4fea2b` | Allow for-loops to define local variables |\n| 77 | `01a94c0` | Add `+=`, `-=`, `*=` and `/=` |\n| 78 | `47f1937` | Add pre `++` and `--` |\n| 79 | `e8ca48c` | Add post `++` and `--` |\n| 80 | `7df934d` | Add hexadecimal, octal and binary number literals |\n| 81 | `6b88bcb` | Add `!` operator |\n| 82 | `46a96d6` | Add `~` operator |\n| 83 | `daa7398` | Add `%` and `%=` |\n| 84 | `8644006` | Add `&`, `|`, `^`, `&=`, `|=` and `^=` |\n| 85 | `f30f781` | Add `&&` and `||` |\n| 86 | `29ed294` | Add a notion of an incomplete array type |\n| 87 | `7963221` | Decay an array to a pointer in the func param context |\n| 88 | `61a1055` | Add a notion of an incomplete struct type |\n| 89 | `6116cae` | Add `goto` and labeled statement |\n| 90 | `a4be55b` | Resolve conflict between labels and typedefs |\n| 91 | `b3047f2` | Add `break` statement |\n| 92 | `3c83dfd` | Add `continue` statement |\n| 93 | `044d9ae` | Add `switch-case` |\n| 94 | `d0c0cb7` | Add `<<`, `>>`, `<<=` and `>>=` |\n| 95 | `447ee09` | Add `?:` operator |\n| 96 | `79f5de2` | Add constant expression |\n\nThis is the chapter where chibicc gets the rest of the C operator surface. Twenty-one commits is too many for one section per commit. **Bundle aggressively, again.** Rough proposal:\n\n- **\u00a711.1 \u2014 For-loop locals** (commit 76). Small. The `for` parser changes to accept either an `expr-stmt` or a `declaration` in the init slot. Probably one section.\n- **\u00a711.2 \u2014 Compound assignment** (commit 77). The `+=`, `-=`, `*=`, `/=` family. The single-evaluation lowering is the centerpiece \u2014 `a += b` should evaluate `&a` once. **Likely the chapter's first canonicalization-at-parse-time addition.** Watch for whether chibicc uses the `tmp = &a, *tmp = *tmp + b` lowering with the \u00a78.5 generalized-lvalue comma extension. Substantive section.\n- **\u00a711.3 \u2014 Pre/post increment and decrement** (commits 78, 79). Bundle. The pre-form is short \u2014 desugar `++x` to `x += 1`. The post-form is fiddlier \u2014 needs to return the old value. May be another canonicalization-at-parse-time instance, or may use a small `+ 1, -1` rewrite directly. Substantive.\n- **\u00a711.4 \u2014 Number-literal bases** (commit 80). Hex (`0x`), octal (`0`), binary (`0b`). Tokenizer change. Small.\n- **\u00a711.5 \u2014 `!` and `~`** (commits 81, 82). Bundle. `!` is \"compare-zero, set-equal\" (similar to `_Bool` cast); `~` is `not %rax`. Both small.\n- **\u00a711.6 \u2014 `%` and `%=`** (commit 83). Modulo and modulo-assign. The `idiv` instruction already produces remainder in `%rdx`/`%edx`; just need to read it. Small to medium.\n- **\u00a711.7 \u2014 Bitwise `&`, `|`, `^` and their compound-assign forms** (commit 84). Six operators in one commit, but they share machinery. Substantive.\n- **\u00a711.8 \u2014 `&&` and `||`** (commit 85). Short-circuit evaluation \u2014 the result of `a && b` is `a` if `a` is zero, otherwise `b`, and `b` is *not* evaluated if `a` is zero. Codegen with labels. Substantive.\n- **\u00a711.9 \u2014 Incomplete types (array, struct)** (commits 86, 87, 88). Bundle? Three commits about incompleteness \u2014 incomplete arrays in declarations, array-decay in function parameters, incomplete structs (forward declarations). The function-param-array-decay (commit 87) is the most subtle: `int f(int x[])` and `int f(int *x)` are equivalent. Substantive.\n- **\u00a711.10 \u2014 `goto` and labels** (commits 89, 90). Bundle. The labels-typedef conflict (commit 90) is a real C grammar issue: `typedef int x; x:` could be a label or a declaration starter. Resolution requires lookahead. Substantive.\n- **\u00a711.11 \u2014 `break` and `continue`** (commits 91, 92). Bundle. Both walk the enclosing-loop label stack. Small to medium.\n- **\u00a711.12 \u2014 `switch`/`case`** (commit 93). Substantive section. `case` values must be constant expressions; `default`; the falling-through behavior; codegen via comparison-and-jump or jump table. Probably comparison-and-jump in chibicc.\n- **\u00a711.13 \u2014 Shift operators `<<`, `>>` and compound-assigns** (commit 94). Four operators, share machinery. Medium.\n- **\u00a711.14 \u2014 `?:`** (commit 95). The conditional operator. Three operands, codegen with labels, `usual_arith_conv` between the second and third operands. Medium.\n- **\u00a711.15 \u2014 Constant expressions** (commit 96). The chapter's closer. A simple constant-folding evaluator that walks the AST and produces an integer result, used for `case` values, enum initializers, array sizes, etc. The `eval` function. Substantive.\n\nThat's 15 sections from 21 commits. **Target chapter length: ~15,000\u201317,000 words.** Slightly larger than Ch 10 because the operators have more codegen detail per commit (each operator has its own assembly idiom).\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in a4fea2b 01a94c0 47f1937 e8ca48c 7df934d 6b88bcb 46a96d6 daa7398 8644006 f30f781 29ed294 7963221 61a1055 6116cae a4be55b b3047f2 3c83dfd 044d9ae d0c0cb7 447ee09 79f5de2; do echo \"===== $h =====\"; git show --stat $h | head -8; done` to scan all twenty-one diffs.\n2. Read each commit. Pay particular attention to:\n   - **`01a94c0`** (commit 77): the compound-assignment lowering. The \u00a78.5 generalized-lvalue comma extension was planted explicitly for this. Watch whether chibicc uses `(tmp = &a, *tmp = *tmp + b)` or builds a new node kind. The handoff prediction is the comma lowering.\n   - **`47f1937`, `e8ca48c`** (78, 79): pre and post increment. The pre-form likely shares with compound-assignment. The post-form needs to return the *old* value \u2014 usually `a = ({ tmp = a; a += 1; tmp; })` style.\n   - **`f30f781`** (commit 85): `&&` and `||`. Short-circuit codegen with labels \u2014 the standard pattern is `cmp; jz/jnz; gen_rhs; setne; movzx`.\n   - **`29ed294`, `7963221`, `61a1055`** (commits 86\u201388): incomplete types. The function-param array decay (commit 87) is the C-standard rule that `int f(int x[10])` is the same as `int f(int *x)`. Watch for the parser-side rewrite.\n   - **`6116cae`, `a4be55b`** (commits 89, 90): `goto` and the typedef-conflict resolution. The 4th namespace (labels) joins. The conflict resolution is a parser-side lookahead.\n   - **`044d9ae`** (commit 93): `switch-case`. Codegen with comparison-and-jump (probably) and a `default`-fallback label.\n   - **`447ee09`** (commit 95): `?:`. Three operands; uses `usual_arith_conv` between then and else expressions.\n   - **`79f5de2`** (commit 96): constant expression. The `eval` function. Used by `case` values.\n3. Read the destination state at `79f5de2` (or shortly after) for `chibicc.h`, `parse.c`, `codegen.c`, `tokenize.c`, all relevant test files. The Ch 11 codegen surface is broader than Ch 10's; expect to spend longer on this step.\n4. Draft `chapters/11-all-the-operators.md`. Likely 15,000\u201317,000 words. **No concept interlude unless the prose surfaces a clear need** \u2014 possible candidate is \"structured programming and the case for/against `goto`\" if \u00a711.10 calls for it, but default to no interlude.\n5. Write `docs/sessions/012-chapter-11-draft/README.md`.\n6. Write `HANDOFF.md` for session 013 (Chapter 12 \u2014 Initializers, commits 97\u2013115; nineteen commits, the densest arc in the compiler per the chapter mapping).\n\n## Voice / structure rules\n\nSame as Ch 1\u201310:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 twenty-one rows; consider splitting into two tables by theme (operators vs. control-flow) as Ch 10 did.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The cast-table-style 4\u00d74 dispatcher in \u00a710.9 is a good model when codegen has table structure (e.g., the bitwise operators all share a small dispatcher).\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.\n- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **Bundle aggressively.** Twenty-one commits won't fit one-per-section at any reasonable chapter length. Group by theme; a section can cover three commits if they share a pattern (the pattern in this chapter is \"operator-family with same lowering shape\" \u2014 `+=` family, bitwise family, shift family, increment family).\n- **\u00a711.2 (compound assignment) closes the \u00a78.5 comma extension's loop.** The \u00a78.5 prose said the comma's generalized-lvalue extension was unused as of Ch 9 and predicted it would land here. Confirm or disconfirm in the \u00a711.2 prose. If chibicc uses a different lowering, name the discrepancy.\n- **\u00a711.10 (`goto` + labels) introduces a fourth namespace.** Labels are function-scoped, not block-scoped. The Ch 9 \u00a79.4 prose listed two namespaces (variables, tags); Ch 10 \u00a710.6 added a third logical namespace inside `vars` (typedef names share with variables). Labels are a fourth, *separate*, function-scoped namespace.\n- **\u00a711.10's typedef-vs-label conflict is real C.** `typedef int x; x:;` could be a label-statement (`x:` is a label) or a declaration (`x` is a typedef-name and `:` would be a parse error). The C standard says label, because labels-then-statement is the only form that starts with `IDENT :`. Watch chibicc's resolution \u2014 probably one-token lookahead at `expr_stmt` or `stmt`.\n- **\u00a711.12 (switch) interacts with the constant-expression machinery.** `case` labels need integer constants. If chibicc writes the constant evaluator first (as a \u00a711.15 commit), the `switch` commit (\u00a711.12, commit 93) precedes it \u2014 meaning `switch`'s `case` parsing will need *something* to evaluate constants and the something probably isn't the full `eval`. Watch for any temporary hack vs. a real call to `eval`. The chronological order \u2014 commit 93 is before commit 96 \u2014 suggests a temporary `get_number` is used until commit 96 generalizes it.\n- **\u00a711.15 (constant expression) is small but load-bearing.** The `eval` function will get more callers in Ch 12 (initializers) and Ch 13 (compound literals). Closing the chapter with it sets up Ch 12.\n- **The compound-assignment family (\u00a711.2) is the canonicalization-at-parse-time hot spot.** Likely instance count goes from six to seven, eight, or nine depending on whether `+=`, `++`, and `--` are each counted as their own desugaring or as one mechanism. Treat them as one mechanism in the count (compound-assign-via-comma) \u2014 but name the variants individually in prose.\n- **The `is_typename` symbol-table lookup from \u00a710.6** gets stressed in \u00a711.10. The label-typedef conflict means `is_typename` and the new `is_label` (or whatever chibicc calls it) need to coexist. Watch for whether `is_typename` gains any extension or whether the label-form is handled entirely in `stmt`.\n\n## Standing notes worth tracking across sessions\n\n- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.\n- **The `mov $0, %rax` (variadic `%al`-zeroing)** noted in Ch 5 \u00a75.1. Pending footnote for revision pass.\n- **The \"more than 6 args silently miscompiles\"** in Ch 5 \u00a75.4. Errata appendix candidate. \u00a710.11 does *not* fix this \u2014 it adds parameter-count-aware casting but skips the cast for extra arguments rather than erroring.\n- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 \u00a77.5) rejects void-returning bodies. Errata candidate.\n- **The hex-escape silent truncation** (Ch 7 \u00a77.4). Errata candidate.\n- **The redeclaration-in-same-scope check** is missing for variables (Ch 8 \u00a78.1), tags (Ch 9 \u00a79.4), and typedef names (Ch 10 \u00a710.6). Three errata candidates.\n- **Block scope is established** as of Ch 8 \u00a78.1. Ch 9 \u00a79.4 added tags; Ch 10 \u00a710.6 added typedef names (sharing `vars`); Ch 10 \u00a710.14 added enum constants (also sharing `vars`); Ch 11 will likely add labels (separate, function-scoped namespace).\n- **Per-token line numbers** (Ch 8 \u00a78.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.\n- **GDB-debuggable output** (Ch 8 \u00a78.4) \u2014 already taken for granted. Should keep working through all of Ch 11; the new operator codegen continues to emit `.loc` directives via `gen_expr`'s opening line.\n- **Tests are in C** as of Ch 8 \u00a78.2. New language features get tests in `test/<feature>.c`.\n- **The host-cc-as-preprocessor pipeline** (Ch 8 \u00a78.2) collapses in Ch 17.\n- **The comma operator's generalized-lvalue extension** (Ch 8 \u00a78.5) is the load-bearing mechanism for compound assignment. Ch 11 \u00a711.2 closes the loop predicted in Ch 8.\n- **Canonicalization-at-parse-time** is a six-instance pattern after Ch 10. Ch 11 will add at least one (compound-assign), possibly more (pre/post-increment if implemented as a separate desugaring, ?: if implemented via a desugaring rather than a new node kind). Treat related variants as one mechanism in the count.\n- **Pre-factor-before-feature** is a four-instance named pattern after Ch 10 \u00a710.1. Ch 11 likely adds none \u2014 most operator additions are self-contained \u2014 but watch for any scaffolding-in-advance commits.\n- **The argreg 8/16/32/64 split** (Ch 10 \u00a710.1, \u00a710.2) is fully in place. Ch 11 codegen will use the right width per operand type via the same machinery.\n- **The `is_typename` helper is a context-sensitive predicate** as of Ch 10 \u00a710.6. Ch 11 \u00a711.10 stresses this when labels join the namespace landscape.\n- **The cast machinery** (`new_cast` + 4\u00d74 `cast_table`) is the type system's load-bearing core. Ch 11's `?:` (\u00a711.14) and shifts (\u00a711.13) use `usual_arith_conv` for operand promotion. The bitwise operators (\u00a711.7) likely also use it.\n- **The `format` helper landed in Ch 7 \u00a77.3.** Workhorse going forward.\n- **The trailing-newline guarantee in `read_file`** (Ch 7 \u00a77.6) protects line-comment skipping. When `read_file` is revisited (Ch 16), preserve.\n- **The lookahead-by-probe pattern** named in Ch 7 \u00a77.1. Ch 10 \u00a710.3 (nested declarators) and \u00a710.7 (abstract declarators) used the double-walk-with-dummy-type idiom \u2014 same family. Ch 11 \u00a711.10 (label-vs-typedef) is the next likely consumer.\n- **The Trusting-Trust framing for `read_escaped_char`** (Ch 7 \u00a77.4) sets up Ch 17 (self-hosting). \u00a710.13 (character literals) leaned on this.\n- **The everything-fits-in-rax codegen invariant** is now a three-regime rule (scalar < 8 in `%eax`, scalar = 8 in `%rax`, struct/union at address pointed to by `%rax`). Cast machinery moves between regimes. Ch 11 doesn't change this.\n- **Member lookup is linear** (Ch 9 \u00a79.1). Fine.\n- **The struct/union/enum tag namespace** is one shared namespace per scope, per C99. Earlier Ch 9 framing as a wart was wrong; Ch 10 \u00a710.14 quietly corrected.\n- **The `unreachable()` macro** (Ch 10 \u00a710.1) lives in `chibicc.h`. Used by `store_gp` and the `declspec` arm-counter. Will get more callers \u2014 Ch 11's `switch` codegen may use it for invalid `case` arms.\n- **The `VarAttr` channel** (Ch 10 \u00a710.6, extended in \u00a710.15) is the carrier for storage-class specifiers. Currently `is_typedef` and `is_static`. Ch 13's `extern` adds a third flag through this channel.\n\n## Acceptance criteria for Ch 11\n\n- [ ] `chapters/11-all-the-operators.md` exists, end-to-end readable.\n- [ ] All twenty-one commits covered, grouped into ~15 sections (or fewer with bundling).\n- [ ] \u00a711.2 (compound assignment) closes the \u00a78.5 generalized-lvalue comma loop and identifies the canonicalization-at-parse-time count's update.\n- [ ] \u00a711.5 (`!`) explains the parallel with the \u00a710.12 `_Bool` cast (both use `cmp; setne; movzx`).\n- [ ] \u00a711.8 (`&&`/`||`) covers short-circuit codegen with labels.\n- [ ] \u00a711.9 (incomplete types) walks the function-param array-decay rule (commit 87) explicitly.\n- [ ] \u00a711.10 (`goto` + labels) names labels as the fourth namespace and walks the typedef-vs-label conflict.\n- [ ] \u00a711.12 (switch) covers `case` falling through and notes the constant-expression dependency.\n- [ ] \u00a711.15 (constant expression) introduces `eval` and previews its Ch 12 / Ch 13 callers.\n- [ ] Each commit has a `git checkout <full-hash>` opener.\n- [ ] Voice matches Ch 1\u201310.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references checked against `chapter-mapping.md`.\n- [ ] `docs/sessions/012-chapter-11-draft/README.md` written.\n- [ ] `HANDOFF.md` written for session 013 (Chapter 12 \u2014 Initializers, commits 97\u2013115).\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/011-chapter-10-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/011-chapter-10-draft/README.md   (what session 011 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md     (most recent chapter)\n13. research/commits/chapter-mapping.md            (confirms Ch 11 scope)\n14. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 11 (All the operators, commits 76\u201396) per the steps\nin the handoff. Twenty-one commits \u2014 bundle aggressively. No concept\ninterlude unless the prose surfaces a clear need. End-of-session: write\nyour session dir under docs/sessions/012-chapter-11-draft/ with a README\nand a HANDOFF for session 013 (Chapter 12 \u2014 Initializers, commits 97\u2013115).\n```\n"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "offset": 68,
  "limit": 10
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 010 README and prior chapter for voice",
      "activeForm": "Reading session 010 README and prior chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 20 commits (56-75) via git show --stat",
      "activeForm": "Scanning 20 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading commit diffs in detail",
      "status": "completed"
    },
    {
      "content": "Read destination state of key files at commit 736232f",
      "activeForm": "Reading destination state files",
      "status": "completed"
    },
    {
      "content": "Draft chapters/10-filling-out-the-type-system.md",
      "activeForm": "Drafting Chapter 10",
      "status": "completed"
    },
    {
      "content": "Write session 011 README",
      "activeForm": "Writing session 011 README",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF.md for session 012 (Chapter 11)",
      "activeForm": "Writing handoff for session 012",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status line for Ch 10 done",
      "activeForm": "Updating CLAUDE.md status",
      "status": "completed"
    }
  ]
}


---

## 🤖 Claude

Session 011 complete. Summary:

**Chapter 10 — *Filling out the type system*** drafted at `chapters/10-filling-out-the-type-system.md`, ~14,800 words, fifteen sections plus one concept interlude on reading C type declarations. Twenty commits (56–75) covered: `int` width change, `long`/`short`/`void`/`_Bool`, nested declarators, complex declspec rewrite, `long long` alias, `typedef`, `sizeof(typename)`, 32-bit register arithmetic, casts, the usual arithmetic conversion, function-call type-checking, character literals, `enum`, file-scope `static`. Largest chapter so far.

**Three structural shifts** named in the chapter and in the carry-forward notes: the bit-field-counter `declspec` (§10.5); the context-sensitive `is_typename` after typedef (§10.6); the cast machinery (§10.9 + §10.10) becoming the type system's load-bearing core.

**One quiet correction:** the Ch 9 errata-candidate "struct and union tags share a namespace" was wrong — that's C-correct (C99 §6.2.3). §10.14 corrects course, and the carry-forward notes drop it from the errata list.

**Session-011 dir** contains `README.md` (decisions and interpretive calls) and `HANDOFF.md` (for session 012, Ch 11 — *All the operators*, commits 76–96, twenty-one commits with a §11.2 compound-assignment section that closes the §8.5 generalized-lvalue comma loop).

**CLAUDE.md status line** updated to reflect Ch 10 drafted (session 011).

---

## 👤 User

from now on can you print the relative path to the handoff file so I can copy/paste it into the next session

---

## 🤖 Claude

Got it. The handoff just written:

`docs/sessions/011-chapter-10-draft/HANDOFF.md`

I'll print the relative path at the end of every session that produces a handoff from now on.

---

