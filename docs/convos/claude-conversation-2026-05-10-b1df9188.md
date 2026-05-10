# Claude Conversation Log

Session ID: b1df9188-6ad0-4372-a580-43abdfb2b824
Date: 2026-05-10 02:14:20

---

## 👤 User

please read docs/sessions/011-chapter-10-draft/HANDOFF.md and proceed

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/011-chapter-10-draft/HANDOFF.md"
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
      "content": "Read session 011 README and chapter 10 for voice",
      "activeForm": "Reading session 011 README and chapter 10 for voice",
      "status": "in_progress"
    },
    {
      "content": "Scan all 21 commits for Ch 11",
      "activeForm": "Scanning all 21 commits for Ch 11",
      "status": "pending"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading each commit's diff in detail",
      "status": "pending"
    },
    {
      "content": "Read destination state at last commit",
      "activeForm": "Reading destination state at last commit",
      "status": "pending"
    },
    {
      "content": "Draft chapters/11-all-the-operators.md",
      "activeForm": "Drafting chapters/11-all-the-operators.md",
      "status": "pending"
    },
    {
      "content": "Write session 012 README and HANDOFF for session 013",
      "activeForm": "Writing session 012 README and HANDOFF for session 013",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/011-chapter-10-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -l /Users/cell/claude/chibicc-book/chapters/10-filling-out-the-type-system.md /Users/cell/claude/chibicc-book/chapters/09-structs-and-unions.md /Users/cell/claude/chibicc-book/chapters/08-scopes-and-source-locations.md",
  "description": "Check line counts of recent chapters"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/10-filling-out-the-type-system.md",
  "limit": 200
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in a4fea2b 01a94c0 47f1937 e8ca48c 7df934d 6b88bcb 46a96d6 daa7398 8644006 f30f781 29ed294 7963221 61a1055 6116cae a4be55b b3047f2 3c83dfd 044d9ae d0c0cb7 447ee09 79f5de2; do echo \"===== $h =====\"; git show --stat $h | head -10; done",
  "description": "Scan all 21 commit stats"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 011 README and chapter 10 for voice",
      "activeForm": "Reading session 011 README and chapter 10 for voice",
      "status": "completed"
    },
    {
      "content": "Scan all 21 commits for Ch 11",
      "activeForm": "Scanning all 21 commits for Ch 11",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading each commit's diff in detail",
      "status": "in_progress"
    },
    {
      "content": "Read destination state at last commit",
      "activeForm": "Reading destination state at last commit",
      "status": "pending"
    },
    {
      "content": "Draft chapters/11-all-the-operators.md",
      "activeForm": "Drafting chapters/11-all-the-operators.md",
      "status": "pending"
    },
    {
      "content": "Write session 012 README and HANDOFF for session 013",
      "activeForm": "Writing session 012 README and HANDOFF for session 013",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a4fea2b",
  "description": "Show commit 76 (for-loop locals)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 01a94c0",
  "description": "Show commit 77 (compound assignment)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 47f1937",
  "description": "Show commit 78 (pre ++/--)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e8ca48c",
  "description": "Show commit 79 (post ++/--)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 7df934d",
  "description": "Show commit 80 (number bases)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6b88bcb 46a96d6",
  "description": "Show commits 81-82 (! and ~)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show daa7398 8644006",
  "description": "Show commits 83-84 (% and bitwise)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show f30f781",
  "description": "Show commit 85 (&& and ||)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 29ed294 7963221",
  "description": "Show commits 86-87 (incomplete array, param array decay)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 61a1055",
  "description": "Show commit 88 (incomplete struct)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6116cae",
  "description": "Show commit 89 (goto and labels)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a4be55b b3047f2 3c83dfd",
  "description": "Show commits 90-92 (label-typedef conflict, break, continue)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 044d9ae",
  "description": "Show commit 93 (switch-case)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show d0c0cb7 447ee09",
  "description": "Show commits 94-95 (shifts and ?:)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 79f5de2",
  "description": "Show commit 96 (constant expression)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 79f5de2:parse.c | head -120",
  "description": "Read parse.c forward-decls at the chapter end"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 011 README and chapter 10 for voice",
      "activeForm": "Reading session 011 README and chapter 10 for voice",
      "status": "completed"
    },
    {
      "content": "Scan all 21 commits for Ch 11",
      "activeForm": "Scanning all 21 commits for Ch 11",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading each commit's diff in detail",
      "status": "completed"
    },
    {
      "content": "Read destination state at last commit",
      "activeForm": "Reading destination state at last commit",
      "status": "completed"
    },
    {
      "content": "Draft chapters/11-all-the-operators.md",
      "activeForm": "Drafting chapters/11-all-the-operators.md",
      "status": "in_progress"
    },
    {
      "content": "Write session 012 README and HANDOFF for session 013",
      "activeForm": "Writing session 012 README and HANDOFF for session 013",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/10-filling-out-the-type-system.md",
  "offset": 1900,
  "limit": 120
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/11-all-the-operators.md",
  "content": "# Chapter 11 \u2014 All the operators\n\n> Commits covered: `a4fea2b`, `01a94c0`, `47f1937`, `e8ca48c`, `7df934d`, `6b88bcb`, `46a96d6`, `daa7398`, `8644006`, `f30f781`, `29ed294`, `7963221`, `61a1055`, `6116cae`, `a4be55b`, `b3047f2`, `3c83dfd`, `044d9ae`, `d0c0cb7`, `447ee09`, `79f5de2`. Twenty-one commits \u2014 the second-largest single chapter in the book \u2014 finishing chibicc's C operator surface.\n\nThrough Chapter 10 the type system grew from three types to eight, the cast machinery landed, and the parser learned every shape of declaration syntax a working programmer is likely to write. What chibicc still didn't have was most of C's operators. Compound assignment (`+=`, `-=`, \u2026). Increment and decrement (`++`, `--`). Modulo (`%`). The bitwise operators (`&`, `|`, `^`, `<<`, `>>`) and their compound-assigns. Logical not and bitwise not (`!`, `~`). Short-circuit `&&` and `||`. The ternary `?:`. And on the control-flow side, `goto`, labeled statements, `break`, `continue`, `switch`/`case`/`default`. The chapter that adds them is the second-largest in the book by commit count.\n\nTwenty-one commits is too many for one section per commit. The chapter bundles aggressively. Operators that share a lowering shape \u2014 the `+=` family, the bitwise family, the shift family, the increment family \u2014 go in one section. The two `goto` commits go in one section. The `break` and `continue` commits go in one section. After bundling there are fifteen sections, plus a closer.\n\nThe fifteen sections:\n\n- **\u00a711.1** \u2014 For-loop locals (commit 76).\n- **\u00a711.2** \u2014 Compound assignment (commit 77).\n- **\u00a711.3** \u2014 Pre and post increment / decrement (commits 78, 79).\n- **\u00a711.4** \u2014 Number-literal bases (commit 80).\n- **\u00a711.5** \u2014 `!` and `~` (commits 81, 82).\n- **\u00a711.6** \u2014 `%` and `%=` (commit 83).\n- **\u00a711.7** \u2014 Bitwise `&`, `|`, `^`, `&=`, `|=`, `^=` (commit 84).\n- **\u00a711.8** \u2014 `&&` and `||` (commit 85).\n- **\u00a711.9** \u2014 Incomplete types: arrays, struct forward declarations (commits 86, 87, 88).\n- **\u00a711.10** \u2014 `goto` and labels (commits 89, 90).\n- **\u00a711.11** \u2014 `break` and `continue` (commits 91, 92).\n- **\u00a711.12** \u2014 `switch` / `case` / `default` (commit 93).\n- **\u00a711.13** \u2014 Shift operators `<<`, `>>`, `<<=`, `>>=` (commit 94).\n- **\u00a711.14** \u2014 `?:` (commit 95).\n- **\u00a711.15** \u2014 Constant expressions (commit 96).\n\nTwo themes run through the chapter. The first is *canonicalization at parse time*, named in \u00a76.5 and tracked since. Compound assignment desugars `a += b` into a comma-with-pointer-temp. Pre-increment desugars `++x` into `x += 1`. Post-increment desugars `x++` into `(typeof x)((x += 1) - 1)`. Each compound-assignment-on-shifts and bitwise-compound-assign desugars the same way. The \u00a78.5 generalized-lvalue comma extension was planted exactly for this; the loop closes here. The second is the *operator-family with shared codegen*: the bitwise family lowers through one parser idiom and three lines of x86-64 each (`and`/`or`/`xor`); the shift family through one idiom and `shl`/`sar`; the relational family was set up earlier and gets one new precedence layer below it. Reading any one operator in detail effectively reads all three.\n\nThe dates of these commits, like Chapters 7\u201310, scatter across the calendar. Roughly half are dated August 2019; the rest are between April and October 2020. The October-2020 cluster is the largest \u2014 `+=`, pre-increment, `%`, bitwise, `&&`/`||`, shifts. Rui clearly worked through the operator surface in one sitting late in the year; the August-2019 commits sit above and below them on `main` in chronological-relative-to-canonical-order tangles that the chapter doesn't try to untangle. Chapter follows `main` order, which is the order the chapter mapping pins.\n\nA note on what's *not* a section. The chapter mapping flagged the possibility of a concept interlude on `goto` and structured programming. The prose for \u00a711.10 turned out not to need it \u2014 the commit's mechanics carry the section, and there's no intuition gap to bridge. No interlude.\n\n---\n\n## 11.1 \u2014 For-loop locals\n\n> `git checkout a4fea2ba3edeb8ab5a0812a09f14c2a771aa196c` \u2014 *Allow for-loops to define local variables*\n\nUntil this commit, the `for` parser read its init slot as an `expr-stmt`:\n\n```c\nnode->init = expr_stmt(&tok, tok);\n```\n\nThat handles `for (i = 0; \u2026)` but not `for (int i = 0; \u2026)`. C99 added the latter, and the change to support it is a five-line patch:\n\n```diff\n-    node->init = expr_stmt(&tok, tok);\n+    enter_scope();\n+\n+    if (is_typename(tok)) {\n+      Type *basety = declspec(&tok, tok, NULL);\n+      node->init = declaration(&tok, tok, basety);\n+    } else {\n+      node->init = expr_stmt(&tok, tok);\n+    }\n```\n\nwith a matching `leave_scope()` after the body. The `for`'s init slot now opens a fresh block scope and uses `is_typename` to decide whether to call `declaration` or `expr_stmt`. The scope ensures that the `i` declared in `for (int i = 0; ...)` doesn't escape the loop:\n\n```c\nASSERT(3, ({ int i=3; int j=0; for (int i=0; i<=10; i=i+1) j=j+i; i; }));\n```\n\nThe outer `i` is `3`. The loop's local `i` shadows it for the duration of the loop and disappears when the loop ends. Without `enter_scope`/`leave_scope` around the `for` body, the inner `i` would have leaked out and the assertion would fail.\n\nThis is the chapter's quietest commit and the only one that doesn't add an operator or a control-flow construct. It lands here because most of the chapter's tests use `for` loops with locally-declared counters \u2014 `for (int i = 0; i < 10; i++)` is the C idiom \u2014 and writing those tests before `for` accepted local declarations would have been awkward. A small pre-factor for the test code that follows.\n\n### Where we are\n\n`for` loops can declare their counters. Block scope opens at the `for` and closes at the matching `}`. The `is_typename` predicate gets one more caller.\n\n---\n\n## 11.2 \u2014 Compound assignment\n\n> `git checkout 01a94c04aa2b5a95ac4038bd0d6fd5334fcbf882` \u2014 *Add `+=`, `-=`, `*=` and `/=`*\n\nThe chapter's first canonicalization-at-parse-time addition since \u00a79.5. The \u00a78.5 generalized-lvalue comma extension was planted explicitly for this; the loop closes here.\n\nC says `a += b` evaluates `a` exactly once and stores the result back to it. The naive lowering \u2014 `a = a + b` \u2014 evaluates `a` *twice*, which is wrong as soon as `a` has side effects: `*p++ += 1` would advance `p` twice. The standard C trick is to evaluate the *address* of `a` once into a temporary, then dereference twice:\n\n```c\ntmp = &a;\n*tmp = *tmp + b;\n```\n\nTwo statements that both produce `*tmp`. As a single expression with a comma operator:\n\n```c\n(tmp = &a, *tmp = *tmp + b)\n```\n\nThe \u00a78.5 commit generalized chibicc's comma operator to accept a *generalized lvalue* on the right \u2014 a comma-expression produces an lvalue if its right operand is one. The \u00a78.5 prose said the extension was unused at the time and predicted a `+=`-style construct as the consumer. That consumer is `to_assign`:\n\n```c\n// Convert `A op= B` to `tmp = &A, *tmp = *tmp op B`\n// where tmp is a fresh pointer variable.\nstatic Node *to_assign(Node *binary) {\n  add_type(binary->lhs);\n  add_type(binary->rhs);\n  Token *tok = binary->tok;\n\n  Obj *var = new_lvar(\"\", pointer_to(binary->lhs->ty));\n\n  Node *expr1 = new_binary(ND_ASSIGN, new_var_node(var, tok),\n                           new_unary(ND_ADDR, binary->lhs, tok), tok);\n\n  Node *expr2 =\n    new_binary(ND_ASSIGN,\n               new_unary(ND_DEREF, new_var_node(var, tok), tok),\n               new_binary(binary->kind,\n                          new_unary(ND_DEREF, new_var_node(var, tok), tok),\n                          binary->rhs,\n                          tok),\n               tok);\n\n  return new_binary(ND_COMMA, expr1, expr2, tok);\n}\n```\n\nRead it slowly. The input is a binary node \u2014 say, `ND_ADD` with `lhs = A` and `rhs = B`. `to_assign` allocates a fresh local variable named `\"\"` (an unnameable identifier \u2014 the parser never reuses it) of type \"pointer to A's type.\" It builds two assignment nodes:\n\n- `expr1` is `tmp = &A`. The temporary now holds A's address.\n- `expr2` is `*tmp = *tmp op B`. The first `*tmp` is the destination lvalue, the second `*tmp` is the loaded value, and `binary->kind` (the `op`) is preserved.\n\nThen it returns `(expr1, expr2)` \u2014 a comma expression. The result of the whole expression is the value of `expr2`, which is the new value of `*tmp`, which is the new value of `A`. C says compound assignment evaluates to the new value of the left-hand side; this matches.\n\nWhy a *pointer* temporary rather than a value temporary? Because A might be an arbitrary lvalue \u2014 `*p++`, `s.field`, `arr[i+1]` \u2014 and the address machinery is what evaluates exactly once. Storing the *address* into the temporary, then dereferencing it twice, gets the side-effect-once semantics for free. The \u00a78.5 generalized-lvalue extension is what lets `*tmp = *tmp + b` *be* an lvalue (so that the comma expression as a whole can sit in further expression contexts).\n\nThe parser side is four `if` branches in `assign`:\n\n```c\nif (equal(tok, \"+=\"))\n  return to_assign(new_add(node, assign(rest, tok->next), tok));\n\nif (equal(tok, \"-=\"))\n  return to_assign(new_sub(node, assign(rest, tok->next), tok));\n\nif (equal(tok, \"*=\"))\n  return to_assign(new_binary(ND_MUL, node, assign(rest, tok->next), tok));\n\nif (equal(tok, \"/=\"))\n  return to_assign(new_binary(ND_DIV, node, assign(rest, tok->next), tok));\n```\n\nThe grammar comment updates accordingly: `assign-op = \"=\" | \"+=\" | \"-=\" | \"*=\" | \"/=\"`. The new tokens `+=`, `-=`, `*=`, `/=` are added to `read_punct`'s table.\n\nThree small details are worth noting.\n\nFirst, `+=` and `-=` use `new_add` and `new_sub` (the pointer-aware helpers from \u00a76.5), not `new_binary(ND_ADD, \u2026)`. That's because compound assignment to a pointer must scale the right-hand side by the element size: `int *p; p += 1;` advances `p` by `sizeof(int)`. `new_add` already encodes that rule.\n\nSecond, `*=` and `/=` use plain `new_binary` \u2014 multiplication and division on pointers is meaningless and won't typecheck, so there's no scaling to worry about.\n\nThird, the temporary's name is `\"\"`. `new_lvar` accepts that without complaint, and the empty name guarantees the identifier never collides with any C identifier. The temporary lives in `locals` like any other local; codegen treats it as a normal stack variable.\n\n### The canonicalization count\n\nThe \u00a710 closer left the count at six (five strict desugarings plus the `({...})` delegation). Compound assignment is the seventh. The handoff predicted \"probably several at once\"; we'll see in \u00a711.3 (pre/post-increment, two more) and \u00a711.6 (`%=`, the same machinery) and \u00a711.7 (`&=`/`|=`/`^=`) and \u00a711.13 (`<<=`/`>>=`). The chapter conservatively counts compound-assign-via-comma as *one* mechanism \u2014 every `op=` operator routes through the same `to_assign` \u2014 so the increment is one, not many. By the end of the chapter the count will be eight (compound assignment + pre/post-increment).\n\n### Where we are\n\n`a += b`, `a -= b`, `a *= b`, `a /= b` work for any C lvalue. They evaluate `a` once. The \u00a78.5 generalized-lvalue comma is the load-bearing mechanism. `to_assign` will get more callers in \u00a711.3, \u00a711.6, \u00a711.7, \u00a711.13.\n\n---\n\n## 11.3 \u2014 Pre and post increment / decrement\n\n> `git checkout 47f19371f75db9029ea1b8b3783624fb7838d2db` \u2014 *Add pre `++` and `--`*\n> `git checkout e8ca48cf41f5f3113cadfb23acfedad7b9fa2e63` \u2014 *Add post `++` and `--`*\n\nTwo commits, two desugarings, both routing through `to_assign`. Bundled because they share the same trick.\n\n### Pre `++` and `--`\n\n`++x` means \"increment `x` by 1, then yield the new value of `x`.\" Which is exactly `x += 1`. Likewise `--x` is `x -= 1`. The parser hooks into `unary`:\n\n```c\n// Read ++i as i+=1\nif (equal(tok, \"++\"))\n  return to_assign(new_add(unary(rest, tok->next), new_num(1, tok), tok));\n\n// Read --i as i-=1\nif (equal(tok, \"--\"))\n  return to_assign(new_sub(unary(rest, tok->next), new_num(1, tok), tok));\n```\n\nThat's it. No new node kind, no codegen. Once parsed, `++i` is indistinguishable from `i += 1`. The `new_add` call handles pointer arithmetic correctly \u2014 `int *p; ++p;` advances by `sizeof(int)`, because `new_add` scales \u2014 and `to_assign` does the side-effect-once dance with the pointer temporary.\n\nThe grammar comment grows one line:\n\n```diff\n // unary = (\"+\" | \"-\" | \"*\" | \"&\") cast\n+//       | (\"++\" | \"--\") unary\n //       | postfix\n```\n\nA subtle point: pre-increment is parsed as a `unary` operator, not a `postfix` one. That's the C precedence table \u2014 `++x` and `x++` sit at different rungs. And pre-increment recurses into `unary` (so `++*p` works), where compound-assignment recurses into `assign`.\n\nThe tokens `++` and `--` are added to `read_punct`'s table.\n\n### Post `++` and `--`\n\nPost-increment is fiddlier. `x++` returns the *old* value of `x`, but still increments `x`. The standard trick is:\n\n```\nx++ \u2261 (typeof x)((x += 1) - 1)\n```\n\nThe expression `(x += 1)` evaluates to the new value of `x` (per \u00a711.2 \u2014 compound assignment yields the new value). Subtracting `1` gives the old value back. The cast preserves the original type (since the subtraction may produce a wider type via the usual arithmetic conversion).\n\nIn code:\n\n```c\n// Convert A++ to `(typeof A)((A += 1) - 1)`\nstatic Node *new_inc_dec(Node *node, Token *tok, int addend) {\n  add_type(node);\n  return new_cast(new_add(to_assign(new_add(node, new_num(addend, tok), tok)),\n                          new_num(-addend, tok), tok),\n                  node->ty);\n}\n```\n\nThe function takes an `addend` of `+1` (for `++`) or `-1` (for `--`):\n\n- The inner `new_add(node, new_num(addend), tok)` builds `node + addend`.\n- `to_assign` desugars that into `(tmp = &node, *tmp = *tmp + addend)` \u2014 this is the side-effect-once compound-assign.\n- The outer `new_add(_, new_num(-addend), tok)` subtracts the addend back.\n- `new_cast` restores the original type.\n\nWhy is this in `new_inc_dec` rather than a one-off in `postfix`? Because `++` and `--` only differ by sign; one helper generates both. And why is the outer add called `new_add` rather than `new_sub`? Because the addend is `-1` for `++` (subtract `-1` to undo `+1`) \u2014 the math is the same, the sign of the constant flips. Consistency with `new_add` matters here because for pointer increments, `new_add` knows to scale by element size; we want the same scaling on the way back.\n\nThe hook lives in `postfix`:\n\n```diff\n // postfix = primary (\"[\" expr \"]\" | \".\" ident | \"->\" ident)*\n+// postfix = primary (\"[\" expr \"]\" | \".\" ident | \"->\" ident | \"++\" | \"--\")*\n static Node *postfix(Token **rest, Token *tok) {\n   ...\n+    if (equal(tok, \"++\")) {\n+      node = new_inc_dec(node, tok, 1);\n+      tok = tok->next;\n+      continue;\n+    }\n+\n+    if (equal(tok, \"--\")) {\n+      node = new_inc_dec(node, tok, -1);\n+      tok = tok->next;\n+      continue;\n+    }\n```\n\n`postfix` is the iterative loop that handles `[]`, `.`, `->`, and now `++`/`--`. The order matters \u2014 `a[i]++` parses left-associatively: first the `a[i]` part is consumed by `postfix`'s subscript branch, then the `++` is consumed by `postfix`'s post-increment branch.\n\nThe tests for post-increment pin down the side-effect ordering:\n\n```c\nASSERT(2, ({ int i=2; i++; }));      // statement value is the old i\nASSERT(3, ({ int i=2; i++; i; }));   // i is now 3\nASSERT(0, ({ int a[3]; a[0]=0; a[1]=1; a[2]=2; int *p=a+1; (*p++)--; a[0]; }));\n```\n\nThe third assertion is the side-effect-ordering torture test. `*p++` evaluates as `*(p++)` \u2014 the `++` has higher precedence than `*` in postfix position. So `(*p++)--`: dereference `p` (which is `&a[1]`, value `1`), advance `p` (now `&a[2]`), then post-decrement what was at the original location (`a[1]` becomes `0`). After the expression, `a[0]` is still `0`, `a[1]` is `0`, `a[2]` is `2`, and `p` points at `a[2]`. The first assertion checks `a[0]`, which is `0` \u2014 meaning the post-decrement landed on `a[1]`, not `a[0]`. Both `++` and `--` had to advance the right pointer the right number of times.\n\n### Date-vs-position note\n\nCommit 78 (pre-increment) is dated 2020-10-07, but commit 79 (post-increment) is dated 2020-04-13. On `main`, 78 comes first. So the *ordering* on `main` says \"pre-increment \u2192 post-increment,\" but the *chronological* order is the opposite. This isn't unusual for chibicc \u2014 Chapter 10's intro flagged the same pattern \u2014 and probably reflects Rui rewriting commits during the 2020-10 cleanup. The chapter follows `main` order, which is the order the book pins.\n\n### Where we are\n\n`++x`, `--x`, `x++`, `x--` work on integers and pointers. None of them needs a new node kind; both desugar to compound assignment plus, for the postfix forms, a cast back to the original type. The canonicalization count is eight: \u00a711.2's compound-assign mechanism plus pre/post-increment as a single mechanism on top of it. The \u00a78.5 comma extension is now load-bearing for ten operators, with more to come.\n\n---\n\n## 11.4 \u2014 Number-literal bases\n\n> `git checkout 7df934d2b63727d67d1c054975893930fa6aff44` \u2014 *Add hexadecimal, octal and binary number literals*\n\nA tokenizer change. Until this commit, every numeric literal was decimal. C has hex (`0x10`), octal (`077`), and \u2014 as a GCC extension long since adopted by Clang and standardized by C23 \u2014 binary (`0b101`). Rui adds all three in one commit by factoring the integer-reading logic into a helper:\n\n```c\nstatic Token *read_int_literal(char *start) {\n  char *p = start;\n\n  int base = 10;\n  if (!strncasecmp(p, \"0x\", 2) && isalnum(p[2])) {\n    p += 2;\n    base = 16;\n  } else if (!strncasecmp(p, \"0b\", 2) && isalnum(p[2])) {\n    p += 2;\n    base = 2;\n  } else if (*p == '0') {\n    base = 8;\n  }\n\n  long val = strtoul(p, &p, base);\n  if (isalnum(*p))\n    error_at(p, \"invalid digit\");\n\n  Token *tok = new_token(TK_NUM, start, p);\n  tok->val = val;\n  return tok;\n}\n```\n\n`strncasecmp` (case-insensitive prefix compare) recognizes `0x`/`0X` and `0b`/`0B`. The `isalnum(p[2])` guard prevents `0x` followed by nothing from misclassifying \u2014 without it, `0x` alone would be a malformed hex literal that `strtoul` would silently accept as zero. With the guard, the lexer falls through to octal (which then sees `0` followed by `x`, errors at `x`, and reports \"invalid digit\").\n\nThe leading-`0` octal rule is a C tradition that has bitten generations of programmers writing `0123` to mean one hundred twenty-three. (`077` is sixty-three.) chibicc inherits the rule without comment; the test pins it down:\n\n```c\nASSERT(511, 0777);\n```\n\n`0777` in octal is `7*64 + 7*8 + 7` = `511`. If `0777` were decimal, the test would fail.\n\n`strtoul` does the actual digit-by-digit parsing for any base. The trailing `isalnum(*p)` check catches the case where parsing stops mid-token because of a non-digit \u2014 `0xZ` or `123abc` \u2014 and reports an error at the offending character.\n\nThe caller in `tokenize` becomes a one-liner:\n\n```diff\n     // Numeric literal\n     if (isdigit(*p)) {\n-      cur = cur->next = new_token(TK_NUM, p, p);\n-      char *q = p;\n-      cur->val = strtoul(p, &p, 10);\n-      cur->len = p - q;\n+      cur = cur->next = read_int_literal(p);\n+      p += cur->len;\n       continue;\n     }\n```\n\nA small `strings.h` include is added to `chibicc.h` because `strncasecmp` lives there on most systems (POSIX puts it in `strings.h`, not `string.h`).\n\n### Where we are\n\nHex, octal, and binary literals tokenize correctly. `0xff`, `0xFF`, `0XFF` are all the same. `0b1011` and `0B1011` are the same. `077` is octal. The token's `val` field is `int64_t`-wide (per \u00a710.2), so all these literals fit.\n\n---\n\n## 11.5 \u2014 `!` and `~`\n\n> `git checkout 6b88bcb306ef80b65d7f99c081ba83283b4ffac5` \u2014 *Add `!` operator*\n> `git checkout 46a96d6862e4c1317ff48df69391fd98a1ae5e3d` \u2014 *Add `~` operator*\n\nTwo unary operators, two minimal AST nodes, two minimal codegen sequences. Bundled because the prose for one essentially repeats the prose for the other.\n\n### `!` \u2014 logical not\n\n`!x` yields `1` if `x` is zero, `0` otherwise. The result type is `int`. The codegen is the same idiom we saw in \u00a710.12 for the `_Bool` cast: compare against zero, set the byte if equal, zero-extend.\n\n```c\ncase ND_NOT:\n  gen_expr(node->lhs);\n  println(\"  cmp $0, %%rax\");\n  println(\"  sete %%al\");\n  println(\"  movzx %%al, %%rax\");\n  return;\n```\n\n`cmp $0, %rax` sets the zero flag if `%rax` holds zero. `sete %al` writes 1 into the low byte of `%rax` if the zero flag is set, 0 otherwise. `movzx %al, %rax` clears the upper bits.\n\nThe \u00a710.12 `_Bool` cast emits the *opposite* sequence \u2014 `cmp $0, %rax; setne %al; movzb %al, %rax` \u2014 because casting to `_Bool` yields 1 for nonzero and 0 for zero. `!` and `(_Bool)` differ by exactly one letter in the asm: `sete` vs. `setne`. Two operators, mirror-image of each other.\n\nThe parser hooks into `unary` alongside the other unary operators:\n\n```c\nif (equal(tok, \"!\"))\n  return new_unary(ND_NOT, cast(rest, tok->next), tok);\n```\n\nA new `ND_NOT` enum entry, an `add_type` arm setting `ty = ty_int`, and tests:\n\n```c\nASSERT(0, !1);\nASSERT(1, !0);\nASSERT(1, !(char)0);\nASSERT(4, sizeof(!(char)0));    // result is int, sizeof(int) == 4\nASSERT(4, sizeof(!(long)0));    // result is int regardless of operand\n```\n\nThe `sizeof` tests are the type-system check: `!`'s result is always `int`, regardless of operand width. The \u00a710.10 usual-arithmetic-conversion machinery doesn't kick in because `!` is unary; `add_type` simply hardcodes `ty_int`.\n\n### `~` \u2014 bitwise not\n\n`~x` flips every bit. The result type matches the operand. The codegen is one instruction:\n\n```c\ncase ND_BITNOT:\n  gen_expr(node->lhs);\n  println(\"  not %%rax\");\n  return;\n```\n\n`not %rax` is x86-64's bitwise complement of all 64 bits. For an `int` (32-bit) operand whose value lives in `%eax` with the upper bits undefined, this happens to work because `!=` and `==` against signed values behave the same whether or not the upper bits match the sign \u2014 and for storage, `mov %eax, mem` only writes the low four bytes anyway. (Strictly: `not %rax` flips the upper bits too, but they get masked off whenever the value is stored or compared as a 32-bit value.)\n\nThe parser:\n\n```c\nif (equal(tok, \"~\"))\n  return new_unary(ND_BITNOT, cast(rest, tok->next), tok);\n```\n\n`add_type` sets the result type to the operand's type:\n\n```c\ncase ND_BITNOT:\n  node->ty = node->lhs->ty;\n  return;\n```\n\nTests:\n\n```c\nASSERT(-1, ~0);\nASSERT(0, ~-1);\n```\n\n`~0` is all-bits-set, which interpreted as a signed `int` is `-1`. `~-1` is `~(0xFFFFFFFF)` = `0`. Both pin down two's-complement.\n\n### The `cast` recursion\n\nBoth `!` and `~` recurse into `cast`, not `unary`. That matters for `!(int)x` and `~(int)x`: the cast operator in \u00a710.9 sits at the `cast` level (lower precedence than `unary`), so `!` and `~` of a cast expression need to descend through `cast` first to consume the cast and only then reach the operand. If they recursed into `unary`, `!(int)x` would parse as `!(int)`, which is nonsense. Recursing into `cast` makes the precedence right.\n\nThis is why `unary`'s grammar comment now reads:\n\n```\nunary = (\"+\" | \"-\" | \"*\" | \"&\" | \"!\" | \"~\") cast\n      | (\"++\" | \"--\") unary\n      | postfix\n```\n\nThe first arm (`+`/`-`/`*`/`&`/`!`/`~`) uses `cast`; the second arm (`++`/`--`) uses `unary`. The increment operators recurse into `unary` because `++++x` is illegal \u2014 you can't increment an rvalue \u2014 and routing through `unary` reaches `postfix`/`primary` to find an lvalue. The first arm's operators don't need an lvalue and so can take the cast operator below them.\n\n### Where we are\n\n`!` and `~` work on integers. `!`'s result is always `int`; `~`'s result preserves the operand type. Both nest correctly with the cast operator.\n\n---\n\n## 11.6 \u2014 `%` and `%=`\n\n> `git checkout daa739817c58baa8dcd0c23bb403d27d5907abfb` \u2014 *Add `%` and `%=`*\n\nModulo. The parser arm in `mul`:\n\n```c\nif (equal(tok, \"%\")) {\n  node = new_binary(ND_MOD, node, cast(&tok, tok->next), start);\n  continue;\n}\n```\n\nThe compound-assign arm in `assign`:\n\n```c\nif (equal(tok, \"%=\"))\n  return to_assign(new_binary(ND_MOD, node, assign(rest, tok->next), tok));\n```\n\n`%=` routes through `to_assign` like all the other compound-assigns. New token `%=` in `read_punct`'s table.\n\nThe codegen is the chapter's first reuse of an existing instruction sequence with a one-line tweak:\n\n```c\ncase ND_DIV:\ncase ND_MOD:\n  if (node->lhs->ty->size == 8)\n    println(\"  cqo\");\n  else\n    println(\"  cdq\");\n  println(\"  idiv %s\", di);\n\n  if (node->kind == ND_MOD)\n    println(\"  mov %%rdx, %%rax\");\n  return;\n```\n\nx86-64's `idiv` instruction performs signed integer division, taking the dividend in `%rdx:%rax` (a 128-bit pair) or, for the 32-bit form, in `%edx:%eax`. The `cqo`/`cdq` instructions sign-extend `%rax`/`%eax` into `%rdx`/`%edx` first \u2014 that's the prerequisite for `idiv`. After `idiv`, the *quotient* lives in `%rax`/`%eax` and the *remainder* lives in `%rdx`/`%edx`. For division, that's already the right answer. For modulo, one extra `mov %rdx, %rax` moves the remainder into the conventional result register.\n\n`ND_DIV` and `ND_MOD` share a `case` arm (C-style fall-through), with a final `if (kind == ND_MOD)` selecting the right output. This is the kind of shared codegen that's easy to miss if you read `gen_expr` linearly \u2014 the `case` arm looks like it handles only `ND_DIV` until you notice the `if (kind == ND_MOD)`.\n\n`%`'s `add_type` arm is added to the `usual_arith_conv` group with the other arithmetic operators:\n\n```diff\n   case ND_MUL:\n   case ND_DIV:\n+  case ND_MOD:\n     usual_arith_conv(&node->lhs, &node->rhs);\n     node->ty = node->lhs->ty;\n     return;\n```\n\nSo `(long)17 % 6` produces a `long`-typed result, exactly like `(long)17 / 6` would.\n\n### Where we are\n\n`%` and `%=` work on integers. They share `idiv` codegen with `/` and `/=`. Pointer modulo is meaningless and won't typecheck.\n\n---\n\n## 11.7 \u2014 Bitwise `&`, `|`, `^`, `&=`, `|=`, `^=`\n\n> `git checkout 86440068b43d6f9c93fdb07c1c2279cbab579e73` \u2014 *Add `&`, `|`, `^`, `&=`, `|=` and `^=`*\n\nSix operators in one commit. Three new precedence layers in the parser, three new node kinds, three new codegen arms (each one instruction), three new compound-assigns routed through `to_assign`. The bundling makes sense: `&`, `|`, `^` have the same shape, and once you've seen one you've seen all three.\n\n### Parser\n\nThe C precedence chain says: `bitor` is below `logor` (above), above `bitxor`; `bitxor` is above `bitand`; `bitand` is above `equality`. Three new layers slot in. Each one looks identical:\n\n```c\n// bitor = bitxor (\"|\" bitxor)*\nstatic Node *bitor(Token **rest, Token *tok) {\n  Node *node = bitxor(&tok, tok);\n  while (equal(tok, \"|\")) {\n    Token *start = tok;\n    node = new_binary(ND_BITOR, node, bitxor(&tok, tok->next), start);\n  }\n  *rest = tok;\n  return node;\n}\n\n// bitxor = bitand (\"^\" bitand)*\nstatic Node *bitxor(Token **rest, Token *tok) { /* same shape with ^ */ }\n\n// bitand = equality (\"&\" equality)*\nstatic Node *bitand(Token **rest, Token *tok) { /* same shape with & */ }\n```\n\nThree nearly-identical functions. `assign` now starts with `bitor` instead of `equality` \u2014 the precedence chain has grown.\n\nThe grammar comment reflects this:\n\n```\nassign    = bitor (assign-op assign)?\nassign-op = \"=\" | \"+=\" | \"-=\" | \"*=\" | \"/=\" | \"%=\" | \"&=\" | \"|=\" | \"^=\"\n```\n\nAnd three more arms in the long `if`-cascade in `assign`:\n\n```c\nif (equal(tok, \"&=\"))\n  return to_assign(new_binary(ND_BITAND, node, assign(rest, tok->next), tok));\n\nif (equal(tok, \"|=\"))\n  return to_assign(new_binary(ND_BITOR, node, assign(rest, tok->next), tok));\n\nif (equal(tok, \"^=\"))\n  return to_assign(new_binary(ND_BITXOR, node, assign(rest, tok->next), tok));\n```\n\n`&=`, `|=`, `^=` go through `to_assign` like the rest of the compound-assigns.\n\n### Codegen\n\n```c\ncase ND_BITAND:\n  println(\"  and %%rdi, %%rax\");\n  return;\ncase ND_BITOR:\n  println(\"  or %%rdi, %%rax\");\n  return;\ncase ND_BITXOR:\n  println(\"  xor %%rdi, %%rax\");\n  return;\n```\n\nThree lines of x86-64. The `and`/`or`/`xor` instructions accept two register operands and write the result into the destination \u2014 exactly the same shape as `add` and `sub`. By the time we reach this `case`, the binary-operator scaffolding has already loaded the right-hand side into `%rdi` and the left-hand side into `%rax`; the single-instruction codegen just names the operation.\n\n### `add_type`\n\nThe three operators join the `usual_arith_conv` group:\n\n```diff\n   case ND_MUL:\n   case ND_DIV:\n   case ND_MOD:\n+  case ND_BITAND:\n+  case ND_BITOR:\n+  case ND_BITXOR:\n     usual_arith_conv(&node->lhs, &node->rhs);\n     node->ty = node->lhs->ty;\n     return;\n```\n\nSo `0xff & (long)0` produces a `long`, same as `0xff + (long)0`.\n\n### Tests\n\n```c\nASSERT(0, 0&1);\nASSERT(3, 7&3);\nASSERT(10, -1&10);\nASSERT(0b10011, 0b10000|0b00011);\nASSERT(0b110100, 0b111000^0b001100);\nASSERT(2, ({ int i=6; i&=3; i; }));\nASSERT(7, ({ int i=6; i|=3; i; }));\nASSERT(10, ({ int i=15; i^=5; i; }));\n```\n\nThe `0b110100 == 0b111000 ^ 0b001100` test exercises the \u00a711.4 binary literals (without \u00a711.4's tokenizer, the test wouldn't compile) and the bitwise codegen at the same time. Six-binary-digit literals as a way of writing tests is an idiom that pays for itself the moment the language has both.\n\n### Where we are\n\n`&`, `|`, `^` and their compound-assign forms work on integers. The parser's precedence chain now has three more layers. The compound-assign machinery has six tenants and counting.\n\n---\n\n## 11.8 \u2014 `&&` and `||`\n\n> `git checkout f30f78175c1fd50c8cdd132ca804573ae0d18453` \u2014 *Add `&&` and `||`*\n\nThe first commit in the chapter that doesn't desugar. `&&` and `||` short-circuit: `a && b` doesn't evaluate `b` if `a` is zero, and `a || b` doesn't evaluate `b` if `a` is nonzero. That ordering can't be expressed by a desugaring into another binary operator \u2014 both operands of an unconditional binary node would always be evaluated. So `ND_LOGAND` and `ND_LOGOR` get their own AST kinds and their own codegen with branches and labels.\n\n### Parser\n\nTwo more precedence layers, slotting in above `bitor` and below `assign`/`conditional`:\n\n```c\n// logor = logand (\"||\" logand)*\nstatic Node *logor(Token **rest, Token *tok) {\n  Node *node = logand(&tok, tok);\n  while (equal(tok, \"||\")) {\n    Token *start = tok;\n    node = new_binary(ND_LOGOR, node, logand(&tok, tok->next), start);\n  }\n  *rest = tok;\n  return node;\n}\n\n// logand = bitor (\"&&\" bitor)*\nstatic Node *logand(Token **rest, Token *tok) { /* same shape with && */ }\n```\n\n`assign`'s entry point changes from `bitor` to `logor`. The grammar comment updates accordingly.\n\n### Codegen\n\nThe interesting part. `&&` short-circuits to false: if the left operand is zero, the result is zero without evaluating the right operand.\n\n```c\ncase ND_LOGAND: {\n  int c = count();\n  gen_expr(node->lhs);\n  println(\"  cmp $0, %%rax\");\n  println(\"  je .L.false.%d\", c);\n  gen_expr(node->rhs);\n  println(\"  cmp $0, %%rax\");\n  println(\"  je .L.false.%d\", c);\n  println(\"  mov $1, %%rax\");\n  println(\"  jmp .L.end.%d\", c);\n  println(\".L.false.%d:\", c);\n  println(\"  mov $0, %%rax\");\n  println(\".L.end.%d:\", c);\n  return;\n}\n```\n\nReading sequentially: evaluate the left. Compare to zero; if equal, jump to `.L.false`. Otherwise evaluate the right. Compare to zero; if equal, jump to `.L.false`. Both operands were nonzero \u2014 load `1` into `%rax` and jump to `.L.end`. `.L.false` loads `0` into `%rax`. `.L.end` is the merge point. The result is `1` (both true) or `0` (either false), and at most one of the two operands has been evaluated to a nonzero result.\n\n`||` is the mirror image:\n\n```c\ncase ND_LOGOR: {\n  int c = count();\n  gen_expr(node->lhs);\n  println(\"  cmp $0, %%rax\");\n  println(\"  jne .L.true.%d\", c);\n  gen_expr(node->rhs);\n  println(\"  cmp $0, %%rax\");\n  println(\"  jne .L.true.%d\", c);\n  println(\"  mov $0, %%rax\");\n  println(\"  jmp .L.end.%d\", c);\n  println(\".L.true.%d:\", c);\n  println(\"  mov $1, %%rax\");\n  println(\".L.end.%d:\", c);\n  return;\n}\n```\n\nIf the left is nonzero, jump to `.L.true` and yield `1`. Otherwise evaluate the right. If nonzero, jump to `.L.true` and yield `1`. Otherwise yield `0`.\n\nThe labels are uniqued by `count()` (the same monotonic counter used by `if`/`for`/`while`), so nested `&&`/`||` don't collide. The result type is always `int`:\n\n```diff\n   case ND_NOT:\n+  case ND_LOGOR:\n+  case ND_LOGAND:\n     node->ty = ty_int;\n     return;\n```\n\n### A small subtlety\n\nC says `a && b` and `a || b` produce `0` or `1` regardless of the actual operand values. `5 && 3` is `1`, not `3`. `5 || 0` is `1`, not `5`. The codegen pattern above produces exactly `0` or `1` because the `mov $1, %rax` and `mov $0, %rax` arms write literal constants. A naive translation that returned the second operand (something like `cmp; jz .end; mov rhs, %rax; .end:`) would be wrong by the standard. Rui's pattern follows the standard precisely.\n\nThe new tokens `&&` and `||` go into `read_punct`'s table.\n\n### Where we are\n\n`&&` and `||` short-circuit. They're the chapter's first AST nodes that emit branching codegen rather than straight-line code. The result is `0` or `1`, type `int`. The parser's precedence chain has grown to seven layers above `bitor`: `assign` \u2192 `conditional` (wait \u2014 that's coming in \u00a711.14, not yet) \u2192 `logor` \u2192 `logand` \u2192 `bitor` \u2192 `bitxor` \u2192 `bitand` \u2192 `equality` \u2192 \u2026.\n\n---\n\n## 11.9 \u2014 Incomplete types: arrays, struct forward declarations\n\n> `git checkout 29ed294906ebc271c32a755e1aefc360df4d3863` \u2014 *Add a notion of an incomplete array type*\n> `git checkout 79632219d0991aae83e1de3c56df7d664205c2b6` \u2014 *Decay an array to a pointer in the func param context*\n> `git checkout 61a10551209a0d3770449862152e1b73b584d771` \u2014 *Add a notion of an incomplete struct type*\n\nThree commits about *incompleteness* \u2014 types that have a name but not yet a known size. Bundled because the underlying trick (use `size = -1` as a sentinel) is the same. The middle commit is the most subtle of the three: function parameters declared as arrays *decay* to pointers, and chibicc has been getting that wrong implicitly until now.\n\n### Incomplete arrays (commit 86)\n\nC lets you write `int x[]` (no dimension) in some contexts: array-of-pointer cast types, function parameters, and global declarations later initialized. The type-suffix parser is refactored to admit empty `[]`:\n\n```c\n// array-dimensions = num? \"]\" type-suffix\nstatic Type *array_dimensions(Token **rest, Token *tok, Type *ty) {\n  if (equal(tok, \"]\")) {\n    ty = type_suffix(rest, tok->next, ty);\n    return array_of(ty, -1);\n  }\n\n  int sz = get_number(tok);\n  tok = skip(tok->next, \"]\");\n  ty = type_suffix(rest, tok, ty);\n  return array_of(ty, sz);\n}\n```\n\nIf `[` is immediately followed by `]`, the array is *incomplete* \u2014 `array_of(ty, -1)`. The `-1` is a sentinel meaning \"size unknown.\" `array_of`, when handed `-1`, sets the type's `size` field to `-1` too (a signed `int` size accommodates this).\n\n`declaration` rejects incomplete-typed variables outright:\n\n```c\nType *ty = declarator(&tok, tok, basety);\nif (ty->size < 0)\n  error_tok(tok, \"variable has incomplete type\");\n```\n\nC says you can declare `int x[]` only in specific contexts; chibicc takes a stricter line and rejects it everywhere except where the parser explicitly allows it. The places that *do* allow it after this commit are abstract-declarator contexts like `sizeof(int(*)[])` (where the `[]` is between parens for the abstract function-pointer-shape, not a real declaration) and parameter contexts (next commit).\n\nThe test:\n\n```c\nASSERT(8, sizeof(int(*)[10]));\nASSERT(8, sizeof(int(*)[][10]));\n```\n\n`sizeof(int(*)[][10])` \u2014 pointer to array-of-array-of-int with the outer dimension unknown \u2014 is `8` because pointers are 8 bytes regardless of the pointee's size or completeness.\n\nThe same commit threads a `Token *tok` field into `Member` (anticipating better error messages for bad struct member references) but doesn't otherwise wire it up.\n\n### Function-parameter array decay (commit 87)\n\nC has a special rule for function parameters. `int f(int x[])` and `int f(int *x)` declare the *same function*. The C standard calls it \"adjustment of parameters\" \u2014 array-typed parameters are silently rewritten to pointer-typed parameters at function-declaration parse time. Chibicc has been getting this wrong implicitly: an `int x[]` parameter would have type `array of int`, the parameter slot would be sized as a (broken) array, and the calling convention would be inconsistent with what real C compilers do.\n\nThe fix is in `func_params`:\n\n```c\nType *ty2 = declspec(&tok, tok, NULL);\nty2 = declarator(&tok, tok, ty2);\n\n// \"array of T\" is converted to \"pointer to T\" only in the parameter\n// context. For example, *argv[] is converted to **argv by this.\nif (ty2->kind == TY_ARRAY) {\n  Token *name = ty2->name;\n  ty2 = pointer_to(ty2->base);\n  ty2->name = name;\n}\n\ncur = cur->next = copy_type(ty2);\n```\n\nAfter the type is fully assembled, if it's an array type, it's replaced with `pointer_to(base)`. The name token (carrying the parameter's identifier) is preserved across the swap. The C identity `int x[]` \u2261 `int *x` now holds.\n\nThe test:\n\n```c\nint param_decay(int x[]) { return x[0]; }\nASSERT(3, ({ int x[2]; x[0]=3; param_decay(x); }));\n```\n\nThe caller passes a 2-element array. The callee declares the parameter as `int x[]`, which after decay is `int *x`. The function reads `x[0]`. The behavior matches what `int *x` would have done, including the calling convention (the array decays to a pointer at the call site, which is exactly what C standard rules say happens for array-typed expressions when they appear in non-`sizeof`, non-`&` contexts).\n\n### Incomplete struct types (commit 88)\n\nThe third incompleteness commit. C lets you declare `struct foo *bar;` without `struct foo` having a body \u2014 the *struct tag* enters the tag namespace, and the pointer is well-formed because pointers don't need to know their pointee's size. Later, when `struct foo { ... }` is defined, the previously-incomplete tag is filled in. This is the standard *forward declaration* idiom.\n\n`struct_union_decl` is rewritten to handle three cases:\n\n1. **Tag with body** (the original case). Declare and define the struct in one shot.\n2. **Tag without body, no prior declaration**. Register the tag with size `-1`. Future references can refer to it; the size will be set when a definition arrives.\n3. **Tag without body, prior declaration**. Look it up and return the existing type.\n\nThe new code:\n\n```c\nif (tag && !equal(tok, \"{\")) {\n  *rest = tok;\n\n  Type *ty = find_tag(tag);\n  if (ty)\n    return ty;\n\n  ty = struct_type();\n  ty->size = -1;\n  push_tag_scope(tag, ty);\n  return ty;\n}\n\ntok = skip(tok, \"{\");\n\n// Construct a struct object.\nType *ty = struct_type();\nstruct_members(rest, tok, ty);\n\nif (tag) {\n  // If this is a redefinition, overwrite a previous type.\n  // Otherwise, register the struct type.\n  for (TagScope *sc = scope->tags; sc; sc = sc->next) {\n    if (equal(tag, sc->name)) {\n      *sc->ty = *ty;\n      return sc->ty;\n    }\n  }\n\n  push_tag_scope(tag, ty);\n}\n\nreturn ty;\n```\n\nA new helper `struct_type()` in `type.c` allocates a fresh struct type with `size = 0` and `align = 1`. The forward-declaration arm overwrites `size` to `-1` (incomplete). The body-defining arm fills in members and computes size and alignment. If the tag was previously incomplete, the body-defining arm finds it in `scope->tags` and *mutates the existing type in place* (`*sc->ty = *ty;`) \u2014 so any pointers declared with the incomplete tag now see the completed type without re-walking.\n\n`struct_decl` and `union_decl` add an early-return for incomplete types:\n\n```c\nif (ty->size < 0)\n  return ty;\n```\n\nOtherwise the offset-assignment loop would index into a missing members list and crash.\n\nThe test pin:\n\n```c\nASSERT(8, ({ struct foo *bar; sizeof(bar); }));\nASSERT(4, ({ struct T *foo; struct T {int x;}; sizeof(struct T); }));\nASSERT(1, ({ struct T { struct T *next; int x; } a; struct T b; b.x=1; a.next=&b; a.next->x; }));\nASSERT(4, ({ typedef struct T T; struct T { int x; }; sizeof(T); }));\n```\n\nThe first asserts that `struct foo *bar` is well-formed even though `struct foo` has no body \u2014 `bar` is a pointer, and `sizeof(bar) == 8`. The second asserts that a forward-declared `struct T` can be later defined and used. The third is the *recursive struct* idiom \u2014 `struct T` contains a pointer to itself \u2014 which only works because the inner `struct T *next` references the tag while it's still incomplete (mid-definition). The fourth combines `typedef struct T T;` with a later `struct T { int x; };` definition; the `typedef`-bound `T` and the tag-bound `struct T` share the underlying type, so the `T` typedef sees the completed `int x` member.\n\n### Where we are\n\nThree forms of incompleteness work: incomplete arrays (`int x[]`), array-to-pointer decay in parameters, and incomplete (forward-declared) struct types. The `size < 0` sentinel is the unifying marker. Forward-declared structs can be later defined; a recursive struct can hold a pointer to itself.\n\n---\n\n## 11.10 \u2014 `goto` and labels\n\n> `git checkout 6116cae4c4b98ef9ed55736f3a6c1d872de97767` \u2014 *Add `goto` and labeled statement*\n> `git checkout a4be55b333c9f712c334aac81e7ef4e076c2bc9b` \u2014 *Resolve conflict between labels and typedefs*\n\nTwo commits. The first adds `goto label;` and labeled statements. The second resolves a parser conflict that the first commit's grammar created.\n\n### `goto` and labels (commit 89)\n\nA function can have any number of labels and any number of `goto`s referring to them. A `goto` may name a label that is *defined later in the function* \u2014 forward jumps are common. So a single-pass parser can't resolve `goto` targets to addresses on the fly; it has to make a second pass after parsing the entire function. Rui's structure:\n\n```c\n// Lists of all goto statements and labels in the curent function.\nstatic Node *gotos;\nstatic Node *labels;\n```\n\nTwo function-scoped globals. As the parser walks, every `goto` adds itself to `gotos`, every label adds itself to `labels`. Both lists chain through a `goto_next` field on `Node`. (Reusing the field name across both flavors is mildly cute; the names match because both are linked-list nexts, not because there's a runtime relationship.)\n\nParsing a `goto`:\n\n```c\nif (equal(tok, \"goto\")) {\n  Node *node = new_node(ND_GOTO, tok);\n  node->label = get_ident(tok->next);\n  node->goto_next = gotos;\n  gotos = node;\n  *rest = skip(tok->next->next, \";\");\n  return node;\n}\n```\n\nThe node stores the *label name* (as a C string) into `node->label` and pushes itself onto the `gotos` list. The `unique_label` field \u2014 which codegen will use \u2014 is left null at this point.\n\nParsing a labeled statement:\n\n```c\nif (tok->kind == TK_IDENT && equal(tok->next, \":\")) {\n  Node *node = new_node(ND_LABEL, tok);\n  node->label = strndup(tok->loc, tok->len);\n  node->unique_label = new_unique_name();\n  node->lhs = stmt(rest, tok->next->next);\n  node->goto_next = labels;\n  labels = node;\n  return node;\n}\n```\n\nThe label gets a *unique* asm-level name (via `new_unique_name`, which produces `.L..NNNN` strings) and stores both the source-level label string and the unique asm name. The labeled statement wraps the actual statement after the colon \u2014 `foo: x++;` parses as `ND_LABEL(\"foo\") -> ND_EXPR_STMT(x++)`.\n\nAfter the function body is parsed, `resolve_goto_labels` matches gotos to labels:\n\n```c\nstatic void resolve_goto_labels(void) {\n  for (Node *x = gotos; x; x = x->goto_next) {\n    for (Node *y = labels; y; y = y->goto_next) {\n      if (!strcmp(x->label, y->label)) {\n        x->unique_label = y->unique_label;\n        break;\n      }\n    }\n\n    if (x->unique_label == NULL)\n      error_tok(x->tok->next, \"use of undeclared label\");\n  }\n\n  gotos = labels = NULL;\n}\n```\n\nA nested loop, O(gotos \u00d7 labels). For every `goto`, scan the labels for a match; if none is found, error. After resolution every `goto` node's `unique_label` points at the label's unique asm name.\n\n`function()` calls `resolve_goto_labels` after parsing the body and before the function returns. The static lists are zeroed out, ready for the next function.\n\nCodegen is a one-liner per node kind:\n\n```c\ncase ND_GOTO:\n  println(\"  jmp %s\", node->unique_label);\n  return;\ncase ND_LABEL:\n  println(\"%s:\", node->unique_label);\n  gen_stmt(node->lhs);\n  return;\n```\n\n`jmp <label>` is unconditional jump. Emitting `<label>:` plants the label at that program point. The unique-naming dance is what keeps two functions' identically-named labels from colliding in the assembly output.\n\n### Labels and the namespace question\n\nLabels are a *fourth namespace* in C. Variables and typedef names share one (the `vars` chain in chibicc). Struct/union/enum tags share another (the `tags` chain). Members within each struct/union live in a per-struct namespace (handled by linear search in `find_member`). Labels are the fourth \u2014 they belong to a *function*, not a block, and they're stored in a function-private list (`gotos`/`labels`) rather than in any `Scope`.\n\nThe C standard's reasoning for separating labels: a `goto` from inside a nested `for` to a label in the enclosing function body must work, regardless of how blocks nest. If labels were block-scoped, `goto` would have to know about scope nesting. Putting them at function scope keeps `goto`'s semantics simple at the cost of one more namespace.\n\n### The label-vs-typedef conflict (commit 90)\n\nCommit 89's grammar has a parsing problem. Consider:\n\n```c\ntypedef int x;\nx: ;\n```\n\nThe `compound_stmt` parser asks \"does this statement start a declaration or a statement?\" Its current criterion is `is_typename`. After \u00a710.6, `is_typename` returns *true* for `x` (because `x` is a typedef name). So `compound_stmt` calls `declaration`, which expects something like `x var;` or `x var = 5;`. But the actual next tokens are `x : ;` \u2014 a label. `declaration` errors.\n\nThe C standard says: when an identifier is followed by `:`, it's a label, regardless of whether the identifier is also a typedef name. Labels don't conflict with typedef names because they live in different namespaces \u2014 but the parser, having only token-by-token lookahead, has to disambiguate before committing.\n\nThe fix is one line:\n\n```diff\n   while (!equal(tok, \"}\")) {\n-    if (is_typename(tok)) {\n+    if (is_typename(tok) && !equal(tok->next, \":\")) {\n       VarAttr attr = {};\n       Type *basety = declspec(&tok, tok, &attr);\n       ...\n```\n\n`compound_stmt` now peeks one token further: if the typename-token is followed by `:`, it's a label, fall through to `stmt`. The disambiguation is local to `compound_stmt`.\n\nThe test:\n\n```c\nASSERT(1, ({ typedef int foo; goto foo; foo:; 1; }));\n```\n\n`typedef int foo` makes `foo` a typedef name. `goto foo` jumps to a label named `foo`. `foo: ;` is the labeled statement. The two `foo`s are in different namespaces and don't collide. Without the lookahead fix, `foo: ;` would be parsed as a (broken) declaration.\n\n### A note on the parser-side hack\n\nThe \u00a710.6 prose called the typedef-handling change in `is_typename` \"the standard C lexer-versus-parser hack.\" Adding the `!equal(tok->next, \":\")` lookahead is a refinement of that same hack: when the symbol-table-driven `is_typename` test gives a precise but standalone answer, you sometimes need *two* tokens of context to disambiguate the syntax. C's grammar isn't context-free in the formal sense; chibicc's parser handles the context-sensitivity locally, with one-token lookahead at the trouble spots.\n\nThis is the third lookahead-by-probe instance the chapter has crossed. \u00a711.10's label-vs-typedef is the latest in a family that includes \u00a77.1 (`int x = 5;` vs. `int x[5];` in `declarator`), \u00a710.3 (nested declarators), \u00a710.7 (abstract declarators), and \u00a710.6 (`is_typename` itself).\n\n### Where we are\n\n`goto` and labeled statements work. Labels live in their own per-function namespace, separate from the four-or-so namespaces in `Scope`. The label-vs-typedef parse conflict is resolved with one-token lookahead in `compound_stmt`. The `keyword` `goto` joins the lexer's table.\n\n---\n\n## 11.11 \u2014 `break` and `continue`\n\n> `git checkout b3047f2317b74f19fb44dfe5e577d586d93dfa3c` \u2014 *Add break statement*\n> `git checkout 3c83dfd8af045ae6923d4ccb3a3a5a50f4012346` \u2014 *Add continue statement*\n\nTwo commits, same shape. Both add a control-flow keyword that jumps to a label, and the label is established by the surrounding loop. Bundled.\n\n### `break` (commit 91)\n\n`break` in a `for`, `while`, or `switch` jumps to the statement immediately after the construct. Each of those constructs gets a `brk_label` field on its `Node`, generated as a unique asm name when the construct is parsed.\n\nA new global `static char *brk_label;` in `parse.c` tracks the *currently active* break label. The parser saves and restores it around each loop:\n\n```c\nchar *brk = brk_label;\nbrk_label = node->brk_label = new_unique_name();\n... parse the loop body ...\nbrk_label = brk;\n```\n\nSave the outer label, install the new one, parse the body (which may contain inner `break`s), restore. This is how nested loops work \u2014 the inner `break` sees the inner loop's label, the outer `break` sees the outer.\n\nParsing a `break`:\n\n```c\nif (equal(tok, \"break\")) {\n  if (!brk_label)\n    error_tok(tok, \"stray break\");\n  Node *node = new_node(ND_GOTO, tok);\n  node->unique_label = brk_label;\n  *rest = skip(tok->next, \";\");\n  return node;\n}\n```\n\nA `break` is parsed as an `ND_GOTO` node \u2014 the same node kind as user-written `goto`. The `unique_label` is set directly to the active break label; no lookup or resolution needed. Reusing `ND_GOTO` rather than introducing `ND_BREAK` keeps codegen trivial: `case ND_GOTO: jmp <unique_label>` already works for both.\n\nThe codegen for `for`/`while` changes one line: instead of `je .L.end.%d`, it emits `je %s` with the saved `brk_label`:\n\n```diff\n-      println(\"  je  .L.end.%d\", c);\n+      println(\"  je %s\", node->brk_label);\n   ...\n-    println(\".L.end.%d:\", c);\n+    println(\"%s:\", node->brk_label);\n```\n\nThe label is now *named* (the unique-name-generator's `.L..NNNN`) rather than positional.\n\n### `continue` (commit 92)\n\n`continue` jumps to the loop's *post-body* point \u2014 for `for`, that's just before the increment expression; for `while`, just before the condition recheck. A second per-loop label and a parallel global track this:\n\n```c\nstatic char *cont_label;\n```\n\nThe save/restore ritual happens for both labels in `for` and `while`:\n\n```c\nchar *brk = brk_label;\nchar *cont = cont_label;\nbrk_label = node->brk_label = new_unique_name();\ncont_label = node->cont_label = new_unique_name();\n... parse body ...\nbrk_label = brk;\ncont_label = cont;\n```\n\nParsing a `continue`:\n\n```c\nif (equal(tok, \"continue\")) {\n  if (!cont_label)\n    error_tok(tok, \"stray continue\");\n  Node *node = new_node(ND_GOTO, tok);\n  node->unique_label = cont_label;\n  *rest = skip(tok->next, \";\");\n  return node;\n}\n```\n\nSame shape as `break`. Same `ND_GOTO` reuse.\n\nCodegen for `for` plants the continue-label between the body and the increment:\n\n```c\ngen_stmt(node->then);\nprintln(\"%s:\", node->cont_label);\nif (node->inc)\n  gen_expr(node->inc);\nprintln(\"  jmp .L.begin.%d\", c);\n```\n\n`continue` jumps to `cont_label`, which lands at the start of the increment expression. The increment runs, then the `jmp` to `.L.begin.%d` re-enters the condition check.\n\n### Stray break / stray continue\n\nBoth forms guard against stray usage \u2014 `break` outside any loop or switch, `continue` outside any loop. The check is:\n\n```c\nif (!brk_label)\n  error_tok(tok, \"stray break\");\n```\n\nIf the parser hasn't entered any construct that sets `brk_label`, the global is null, and the error fires. The error catches typos and copy-paste mishaps and is the only constraint chibicc places on `break`/`continue` (it doesn't, e.g., enforce that `continue` can't appear directly inside a `switch` body \u2014 C says that's legal if the switch is itself inside a loop and the continue refers to the enclosing loop, and chibicc's parser lets it through naturally because `cont_label` is set by the surrounding loop).\n\n### Where we are\n\n`break` and `continue` work in `for` and `while`. They reuse `ND_GOTO` codegen. Stray usage errors. Both keywords go into the lexer's table.\n\n---\n\n## 11.12 \u2014 `switch` / `case` / `default`\n\n> `git checkout 044d9ae07ba700c52d8342e4eee26e07eea11619` \u2014 *Add switch-case*\n\nThe chapter's most substantial control-flow commit. `switch (e) { case k1: \u2026; case k2: \u2026; default: \u2026; }` evaluates `e`, jumps to the matching `case` label, falls through to subsequent cases unless a `break` interrupts, and falls through to `default` if no case matches.\n\n### Parser\n\nA new global tracks the currently-being-parsed switch:\n\n```c\n// Points to a node representing a switch if we are parsing\n// a switch statement. Otherwise, NULL.\nstatic Node *current_switch;\n```\n\nThe switch's parse:\n\n```c\nif (equal(tok, \"switch\")) {\n  Node *node = new_node(ND_SWITCH, tok);\n  tok = skip(tok->next, \"(\");\n  node->cond = expr(&tok, tok);\n  tok = skip(tok, \")\");\n\n  Node *sw = current_switch;\n  current_switch = node;\n\n  char *brk = brk_label;\n  brk_label = node->brk_label = new_unique_name();\n\n  node->then = stmt(rest, tok);\n\n  current_switch = sw;\n  brk_label = brk;\n  return node;\n}\n```\n\nSave-install-parse-restore for both `current_switch` and `brk_label`. Note that no `cont_label` is set: `continue` inside a `switch` refers to the enclosing loop, not the switch. The `Node` has a `cond` (the switch-expression), a `then` (the body), and a `brk_label` that `break` inside will jump to.\n\n`case` and `default` are parsed within `stmt`:\n\n```c\nif (equal(tok, \"case\")) {\n  if (!current_switch)\n    error_tok(tok, \"stray case\");\n  int val = get_number(tok->next);\n\n  Node *node = new_node(ND_CASE, tok);\n  tok = skip(tok->next->next, \":\");\n  node->label = new_unique_name();\n  node->lhs = stmt(rest, tok);\n  node->val = val;\n  node->case_next = current_switch->case_next;\n  current_switch->case_next = node;\n  return node;\n}\n\nif (equal(tok, \"default\")) {\n  if (!current_switch)\n    error_tok(tok, \"stray default\");\n\n  Node *node = new_node(ND_CASE, tok);\n  tok = skip(tok->next, \":\");\n  node->label = new_unique_name();\n  node->lhs = stmt(rest, tok);\n  current_switch->default_case = node;\n  return node;\n}\n```\n\nEach `case` parses its constant value (right now, via `get_number` \u2014 *one decimal integer*; this is temporary, replaced in \u00a711.15), gets a unique asm-level label, parses the trailing statement, and links itself onto `current_switch->case_next`. The list grows in reverse order; codegen walks it accordingly.\n\n`default` is a `case` without a value \u2014 it's stored on `current_switch->default_case` directly rather than threaded onto `case_next`.\n\n### Codegen\n\n`switch` codegen is comparison-and-jump (no jump table, no balanced-tree optimization \u2014 chibicc keeps it simple):\n\n```c\ncase ND_SWITCH:\n  gen_expr(node->cond);\n\n  for (Node *n = node->case_next; n; n = n->case_next) {\n    char *reg = (node->cond->ty->size == 8) ? \"%rax\" : \"%eax\";\n    println(\"  cmp $%ld, %s\", n->val, reg);\n    println(\"  je %s\", n->label);\n  }\n\n  if (node->default_case)\n    println(\"  jmp %s\", node->default_case->label);\n\n  println(\"  jmp %s\", node->brk_label);\n  gen_stmt(node->then);\n  println(\"%s:\", node->brk_label);\n  return;\ncase ND_CASE:\n  println(\"%s:\", node->label);\n  gen_stmt(node->lhs);\n  return;\n```\n\nThe dispatch sequence:\n\n1. Evaluate `cond` into `%rax`.\n2. For each case: compare its value against `%eax` or `%rax` (per the cond type's size \u2014 8-byte cond uses 64-bit compare, 4-byte cond uses 32-bit compare), jump to the matching case label.\n3. After all cases, if there's a `default`, unconditionally jump to it. Otherwise, unconditionally jump to the break label (i.e., past the body).\n4. Generate the body statements. The case labels embedded in the body act as targets for the `je`s above.\n5. Plant the break label at the end.\n\nThe body codegen is straight-line \u2014 execution falls through from case to case unless a `break` interrupts, exactly matching C's fallthrough semantics. The `case`/`default` labels in the body are just labels; control reaches them via the dispatch jumps at the top.\n\nThe 32-bit vs. 64-bit register selection uses the cond's *size*, not its sign. The test pins this down:\n\n```c\nASSERT(3, ({ int i=0; switch(-1) { case 0xffffffff: i=3; break; } i; }));\n```\n\nThe cond is `int`-typed (4 bytes). The case value `0xffffffff` is parsed as a positive constant `4294967295`, but when stored in the case node's `val` (an `int64_t`) and then compared with a `cmp $0xffffffff, %eax` instruction, the 32-bit comparison interprets the immediate as a sign-extended 32-bit value \u2014 which matches `-1`. So `switch(-1)` matches `case 0xffffffff`. The `int`-vs-`long` choice of comparison register is what makes this work; if the codegen always used `%rax`, the comparison would be against `0x00000000ffffffff`, which doesn't equal sign-extended `-1`.\n\n### The temporary `get_number`\n\nThis commit's `case` parser uses `get_number`, which accepts only a single integer token. That means `case 5+2:` would fail (the `+` isn't expected after the number). Real C says case values are *constant expressions* \u2014 they may involve arithmetic, sizeof, and integer casts. Rui's commit-93 implementation knows it's incomplete and writes the simpler version; commit 96 (\u00a711.15) replaces `get_number` with a real constant evaluator.\n\nThis pattern \u2014 write a *temporary* hack, replace it later in the same chapter \u2014 is unusual for chibicc. Most simplifications stick around. Tracking it as a chapter-internal scaffolding pattern: the \u00a711.15 commit removes `get_number` from `parse.c` entirely and replaces every caller (the `case` value here, the `enum` constant assignment, the array-dimension parser) with `const_expr`.\n\n### Tests\n\n```c\nASSERT(5, ({ int i=0; switch(0) { case 0:i=5;break; case 1:i=6;break; case 2:i=7;break; } i; }));\nASSERT(0, ({ int i=0; switch(3) { case 0:i=5;break; case 1:i=6;break; case 2:i=7;break; } i; }));\nASSERT(5, ({ int i=0; switch(0) { case 0:i=5;break; default:i=7; } i; }));\nASSERT(7, ({ int i=0; switch(1) { case 0:i=5;break; default:i=7; } i; }));\nASSERT(2, ({ int i=0; switch(1) { case 0: 0; case 1: 0; case 2: 0; i=2; } i; }));\nASSERT(0, ({ int i=0; switch(3) { case 0: 0; case 1: 0; case 2: 0; i=2; } i; }));\n```\n\nThe fifth pin tests *fallthrough*: with no `break`s, `switch(1)` lands on `case 1`, falls through `case 2`, and reaches `i=2`. The sixth tests *no match*: `switch(3)` finds no case, so without `default` the body is skipped entirely.\n\n`switch`, `case`, `default` join the lexer's keyword table.\n\n### Where we are\n\n`switch`, `case`, and `default` work for integer cases. Comparison-and-jump dispatch. Fallthrough is the default; `break` interrupts. Case values are parsed via the temporary `get_number`, replaced by `const_expr` in \u00a711.15.\n\n---\n\n## 11.13 \u2014 Shift operators `<<`, `>>`, `<<=`, `>>=`\n\n> `git checkout d0c0cb74b21f431c62f7eeb8dbc0d6e14c1eff14` \u2014 *Add `<<`, `>>`, `<<=` and `>>=`*\n\nFour operators in one commit. The parser sprouts a new precedence layer between `relational` and `add`; codegen gains two arms; the compound-assigns route through `to_assign`.\n\n### Parser\n\n`shift` slots between `relational` and `add`:\n\n```c\n// shift = add (\"<<\" add | \">>\" add)*\nstatic Node *shift(Token **rest, Token *tok) {\n  Node *node = add(&tok, tok);\n\n  for (;;) {\n    Token *start = tok;\n\n    if (equal(tok, \"<<\")) {\n      node = new_binary(ND_SHL, node, add(&tok, tok->next), start);\n      continue;\n    }\n\n    if (equal(tok, \">>\")) {\n      node = new_binary(ND_SHR, node, add(&tok, tok->next), start);\n      continue;\n    }\n\n    *rest = tok;\n    return node;\n  }\n}\n```\n\n`relational` updates to call `shift` instead of `add`, so `a < b << 2` parses as `a < (b << 2)` \u2014 the C precedence rule (shifts bind tighter than relational).\n\nThe compound-assign arms in `assign`:\n\n```c\nif (equal(tok, \"<<=\"))\n  return to_assign(new_binary(ND_SHL, node, assign(rest, tok->next), tok));\n\nif (equal(tok, \">>=\"))\n  return to_assign(new_binary(ND_SHR, node, assign(rest, tok->next), tok));\n```\n\nSame pattern as the rest of the compound-assign family.\n\n### Codegen\n\n```c\ncase ND_SHL:\n  println(\"  mov %%rdi, %%rcx\");\n  println(\"  shl %%cl, %s\", ax);\n  return;\ncase ND_SHR:\n  println(\"  mov %%rdi, %%rcx\");\n  if (node->ty->size == 8)\n    println(\"  sar %%cl, %s\", ax);\n  else\n    println(\"  sar %%cl, %s\", ax);\n  return;\n```\n\nx86-64's variable-shift instructions (`shl` / `shr` / `sar`) require the shift count in `%cl` (the low byte of `%rcx`). Both arms move the right-hand side from `%rdi` (where the binary-operator scaffolding placed it) into `%rcx`, then emit the shift.\n\n`shl` is *shift left*. `sar` is *shift arithmetic right* \u2014 preserves the sign bit. The `>>` operator uses `sar` because chibicc only has signed integer types; `sar` does sign-extension during the shift, which is the C-defined behavior for signed `>>`. (When unsigned types arrive in Chapter 14, the `>>` codegen will branch on operand signedness \u2014 `sar` for signed, `shr` (logical) for unsigned.)\n\nThe `if (node->ty->size == 8)` branch in `ND_SHR` is interesting: both arms emit the same instruction. The branch was probably meant to dispatch to a different opcode but ended up identical; perhaps Rui intended `sar` for 8-byte and a different mnemonic for 4-byte and didn't finish the differentiation. The `ax` macro at the top of `gen_expr` selects `%eax` or `%rax` based on `node->ty->size`, so the operand width is correct either way \u2014 the size-of-shift is determined by the destination register, not the opcode.\n\n### `add_type`\n\n```diff\n   case ND_BITNOT:\n+  case ND_SHL:\n+  case ND_SHR:\n     node->ty = node->lhs->ty;\n     return;\n```\n\nShift operators take the left operand's type, *not* `usual_arith_conv`. C says the left operand of a shift is integer-promoted but the right operand is independent \u2014 the result type is the (promoted) left operand's type. Chibicc's simpler rule says \"the result is whatever the left side is,\" which is close enough for chibicc's integer surface (no `signed`/`unsigned` distinction yet).\n\n### Tests\n\n```c\nASSERT(1, 1<<0);\nASSERT(8, 1<<3);\nASSERT(10, 5<<1);\nASSERT(2, 5>>1);\nASSERT(-1, -1>>1);                 // arithmetic right shift preserves sign\nASSERT(1, ({ int i=1; i<<=0; i; }));\nASSERT(8, ({ int i=1; i<<=3; i; }));\nASSERT(-1, ({ int i=-1; i>>=1; i; }));\n```\n\n`-1 >> 1` is `-1` (all bits stay set under arithmetic shift). `-1 >>= 1` does the same with a compound-assign.\n\nThree new tokens: `<<`, `>>`, `<<=`, `>>=`. The three-character `<<=` and `>>=` are placed *first* in `read_punct`'s table, before the two-character `<<` and `>>`, so the longest-match-first rule picks `<<=` over `<<` followed by `=`:\n\n```diff\n-    \"==\", \"!=\", \"<=\", \">=\", \"->\", \"+=\", \"-=\", \"*=\", \"/=\", \"++\", \"--\",\n-    \"%=\", \"&=\", \"|=\", \"^=\", \"&&\", \"||\",\n+    \"<<=\", \">>=\", \"==\", \"!=\", \"<=\", \">=\", \"->\", \"+=\",\n+    \"-=\", \"*=\", \"/=\", \"++\", \"--\", \"%=\", \"&=\", \"|=\", \"^=\", \"&&\",\n+    \"||\", \"<<\", \">>\",\n```\n\n`read_punct` walks the table in order and returns on the first match, so reordering matters: `<<=` is tried before `<<`, otherwise `i<<=2` would tokenize as `<<` followed by `=2` and parse as `i << (=2)`, which is broken in two different ways.\n\n### Where we are\n\n`<<`, `>>`, `<<=`, `>>=` work on integers. `>>` is arithmetic right shift (sign-preserving). The parser's precedence chain has another layer; the longest-match tokenizer table is hand-ordered.\n\n---\n\n## 11.14 \u2014 `?:`\n\n> `git checkout 447ee098c51f6f615ef560b35d429f32f0cb5a35` \u2014 *Add `?:` operator*\n\nThe conditional operator. Three operands, branching codegen, usual arithmetic conversion between the second and third operands. A new precedence layer between `assign` and `logor`.\n\n### Parser\n\n```c\n// conditional = logor (\"?\" expr \":\" conditional)?\nstatic Node *conditional(Token **rest, Token *tok) {\n  Node *cond = logor(&tok, tok);\n\n  if (!equal(tok, \"?\")) {\n    *rest = tok;\n    return cond;\n  }\n\n  Node *node = new_node(ND_COND, tok);\n  node->cond = cond;\n  node->then = expr(&tok, tok->next);\n  tok = skip(tok, \":\");\n  node->els = conditional(rest, tok);\n  return node;\n}\n```\n\n`assign` now calls `conditional` instead of `logor`. The grammar comment changes:\n\n```diff\n-// assign = logor (assign-op assign)?\n+// assign = conditional (assign-op assign)?\n```\n\nThe middle operand (`then`) is parsed with `expr` (so `a ? b, c : d` parses as `a ? (b, c) : d`). The else-arm (`els`) is parsed with `conditional` itself, making `?:` *right-associative*: `a ? b : c ? d : e` parses as `a ? b : (c ? d : e)`.\n\nRight-associativity matches C's grammar exactly. The other recursive operator at this level \u2014 assignment \u2014 is also right-associative (`a = b = c` is `a = (b = c)`); no surprise that `?:` follows the same pattern.\n\n### Codegen\n\n```c\ncase ND_COND: {\n  int c = count();\n  gen_expr(node->cond);\n  println(\"  cmp $0, %%rax\");\n  println(\"  je .L.else.%d\", c);\n  gen_expr(node->then);\n  println(\"  jmp .L.end.%d\", c);\n  println(\".L.else.%d:\", c);\n  gen_expr(node->els);\n  println(\".L.end.%d:\", c);\n  return;\n}\n```\n\nStandard if-then-else codegen: evaluate cond, branch on zero, emit then arm, jump past else, plant else label, emit else arm, plant end label. The result of whichever arm executed lives in `%rax` after this block. Same shape as the \u00a73.5-ish `if`-statement codegen, but at expression level.\n\n### `add_type`\n\nThe interesting part. C says `?:`'s result type is determined by *applying the usual arithmetic conversion to the second and third operands*. If either is `void`, the result is `void` (with no conversion).\n\n```c\ncase ND_COND:\n  if (node->then->ty->kind == TY_VOID || node->els->ty->kind == TY_VOID) {\n    node->ty = ty_void;\n  } else {\n    usual_arith_conv(&node->then, &node->els);\n    node->ty = node->then->ty;\n  }\n  return;\n```\n\nThe `usual_arith_conv` call is the \u00a710.10 helper. It sees the two operands, finds their common type, and inserts `ND_CAST` nodes around each so they're both that common type. Then the result type is whichever common type they ended up at.\n\nTests:\n\n```c\nASSERT(2, 0?1:2);\nASSERT(4, sizeof(0?1:2));            // both int, common type int (4 bytes)\nASSERT(8, sizeof(0?(long)1:(long)2)); // both long, common type long (8 bytes)\nASSERT(-1, 0?(long)-2:-1);            // long and int, common type long, result -1 cast to long is -1\nASSERT(-1, 0?-2:(long)-1);            // int and long, same \u2014 result is (long)-1\n1 ? -2 : (void)-1;                    // void in else arm, no conversion, result type void\n```\n\nThe last one \u2014 `1 ? -2 : (void)-1` \u2014 is the `void` arm. C says this is legal as long as the result is used in a void context (e.g., as a statement-expression with no semantics). The test exercises just that.\n\n### The canonicalization-at-parse-time question\n\n`?:` *could* have been canonicalized \u2014 for instance, by lowering to a series of comma operators with side-effecting assignments and a result variable. Rui chose not to; `?:` ships as a new node kind with explicit branching codegen. The canonicalization count remains at eight (compound-assign + pre/post-increment).\n\nThe reason `?:` doesn't canonicalize is clear: there's no way to express short-circuiting evaluation in chibicc's existing AST without either branching codegen (which is what `ND_COND` provides) or runtime tests-and-branches encoded as a complicated comma-with-assignment lowering that wouldn't be simpler. The \u00a711.8 prose for `&&`/`||` reached the same conclusion.\n\n### Where we are\n\n`?:` works. It short-circuits \u2014 only one arm is evaluated. `usual_arith_conv` chooses the common type. `void`-typed arms produce a `void`-typed result.\n\n---\n\n## 11.15 \u2014 Constant expressions\n\n> `git checkout 79f5de21eb706ea5486fd682a83ffbde7e4d16a9` \u2014 *Add constant expression*\n\nThe chapter's closer, and a small but load-bearing piece of machinery. C requires *constant expressions* in several contexts \u2014 array dimensions, `case` values, enum-constant assignments, bit-field widths (later). Until this commit, chibicc handled these by parsing only a single integer literal via `get_number`. That's incomplete: real C allows arithmetic, sizeof, casts, even `&&`/`||` inside constant expressions.\n\nThe fix is a small evaluator that walks the AST after parsing and folds it down to an `int64_t`:\n\n```c\n// Evaluate a given node as a constant expression.\nstatic int64_t eval(Node *node) {\n  add_type(node);\n\n  switch (node->kind) {\n  case ND_ADD: return eval(node->lhs) + eval(node->rhs);\n  case ND_SUB: return eval(node->lhs) - eval(node->rhs);\n  case ND_MUL: return eval(node->lhs) * eval(node->rhs);\n  case ND_DIV: return eval(node->lhs) / eval(node->rhs);\n  case ND_NEG: return -eval(node->lhs);\n  case ND_MOD: return eval(node->lhs) % eval(node->rhs);\n  case ND_BITAND: return eval(node->lhs) & eval(node->rhs);\n  case ND_BITOR:  return eval(node->lhs) | eval(node->rhs);\n  case ND_BITXOR: return eval(node->lhs) ^ eval(node->rhs);\n  case ND_SHL:    return eval(node->lhs) << eval(node->rhs);\n  case ND_SHR:    return eval(node->lhs) >> eval(node->rhs);\n  case ND_EQ:     return eval(node->lhs) == eval(node->rhs);\n  case ND_NE:     return eval(node->lhs) != eval(node->rhs);\n  case ND_LT:     return eval(node->lhs) <  eval(node->rhs);\n  case ND_LE:     return eval(node->lhs) <= eval(node->rhs);\n  case ND_COND:   return eval(node->cond) ? eval(node->then) : eval(node->els);\n  case ND_COMMA:  return eval(node->rhs);\n  case ND_NOT:    return !eval(node->lhs);\n  case ND_BITNOT: return ~eval(node->lhs);\n  case ND_LOGAND: return eval(node->lhs) && eval(node->rhs);\n  case ND_LOGOR:  return eval(node->lhs) || eval(node->rhs);\n  case ND_CAST:\n    if (is_integer(node->ty)) {\n      switch (node->ty->size) {\n      case 1: return (uint8_t)eval(node->lhs);\n      case 2: return (uint16_t)eval(node->lhs);\n      case 4: return (uint32_t)eval(node->lhs);\n      }\n    }\n    return eval(node->lhs);\n  case ND_NUM:\n    return node->val;\n  }\n\n  error_tok(node->tok, \"not a compile-time constant\");\n}\n```\n\nA switch over every integer-arithmetic AST node kind. Each arm recursively `eval`s its children, applies the C operator, and returns the integer result. The `default` arm errors: a `case` value of `x + 1` where `x` is a runtime variable hits `ND_VAR`, which has no arm, and reports \"not a compile-time constant.\"\n\nThe `ND_CAST` arm is the only one with size-specific logic. Casting to `char` (size 1) masks to `uint8_t`; to `short` (size 2) masks to `uint16_t`; to `int` (size 4) masks to `uint32_t`. The rationale: the C standard says casting an integer to a smaller integer takes the value modulo `2^N` where `N` is the bit-width of the target. Using `uint8_t`/`uint16_t`/`uint32_t` and letting C's implicit conversion to `int64_t` handle the sign extension does this in one C cast each. (`(long)0xfffff` and casting to `int` doesn't change the chibicc-host-compiler's behavior \u2014 both are 8 bytes \u2014 so the size-8 case has no arm.)\n\nThe `ND_COMMA` arm returns `eval(rhs)`, ignoring `lhs`. For a constant expression, the left side of a comma must itself be constant (else `eval(lhs)` would error inside the recursion), but its value is discarded in C's comma-operator semantics. This implementation order matters: chibicc's `eval` doesn't actually call `eval(lhs)` here \u2014 it returns the right side directly, skipping the left's evaluation entirely. That's slightly looser than C requires (the left side might not even be a constant expression and the test would still pass), but it's harmless because chibicc only calls `eval` from contexts where the *whole* expression is required to be constant, and the front-end already accepts only constant-shaped subtrees.\n\n### `const_expr`\n\nThe wrapper that callers use:\n\n```c\nstatic int64_t const_expr(Token **rest, Token *tok) {\n  Node *node = conditional(rest, tok);\n  return eval(node);\n}\n```\n\nParse a `conditional`-level expression (which excludes assignment \u2014 assignments aren't constant), then evaluate it. The `conditional` entry point is exactly right: the C grammar says constant expressions are conditional-expressions, and conditional sits below assign.\n\n`get_number` is removed entirely. The three callers \u2014 array-dimensions, case-values, and enum-constant assignments \u2014 all switch to `const_expr`:\n\n```diff\n-    int sz = get_number(tok);\n-    tok = skip(tok->next, \"]\");\n+    int sz = const_expr(&tok, tok);\n+    tok = skip(tok, \"]\");\n   ...\n-      val = get_number(tok->next);\n-      tok = tok->next->next;\n+      val = const_expr(&tok, tok->next);\n   ...\n-    int val = get_number(tok->next);\n-\n     Node *node = new_node(ND_CASE, tok);\n-    tok = skip(tok->next->next, \":\");\n+    int val = const_expr(&tok, tok->next);\n+    tok = skip(tok, \":\");\n```\n\nThe grammar comments reflect the change: `array-dimensions = const-expr? \"]\" type-suffix`, `\"case\" const-expr \":\" stmt`.\n\n### Tests\n\n`test/constexpr.c` is a new file with thirty-plus assertions, exercising every operator that `eval` supports, in every context that calls `const_expr`:\n\n```c\nASSERT(10, ({ enum { ten=1+2+3+4 }; ten; }));    // arithmetic in enum constant\nASSERT(1, ({ int i=0; switch(3) { case 5-2+0*3: i++; } i; }));  // arithmetic in case\nASSERT(8, ({ int x[1+1]; sizeof(x); }));         // arithmetic in array dim\nASSERT(0b100, ({ char x[0b110&0b101]; sizeof(x); }));   // bitwise\nASSERT(2, ({ char x[1?2:3]; sizeof(x); }));      // ?: in array dim\nASSERT(15, ({ char x[(char)0xffffff0f]; sizeof(x); })); // cast to char masks\nASSERT(0x10f, ({ char x[(short)0xffff010f]; sizeof(x); })); // cast to short masks\nASSERT(4, ({ char x[(int)0xfffffffffff+5]; sizeof(x); })); // cast to int masks\nASSERT(8, ({ char x[(int*)0+2]; sizeof(x); }));    // pointer arithmetic in const expr\n```\n\nThe pointer-arithmetic test is the only one that exercises `eval`'s implicit treatment of `ND_ADD`-with-pointer-children. There's no pointer-specific arm in `eval`; the expression `(int*)0 + 2` is parsed as `(int*)0 + 2`, which `add_type` and `new_add` would have transformed into `(int*)0 + 2*sizeof(int)` (= `+8`) at parse time. By the time `eval` sees the AST, the multiplication has already happened, and `ND_ADD`'s arithmetic arm just adds `0 + 8 = 8`. The test pins this down: `char x[(int*)0+2]` produces a `char[8]`.\n\n### Forward references\n\nThe `eval` function will get more callers in coming chapters. Chapter 12 (initializers) will use it for compile-time constant initializers \u2014 `int x = 1 + 2;` evaluates `1 + 2` at parse time and stores `3` in the global's initial value. Chapter 13's `extern` doesn't directly use it, but `const` and storage-class qualifiers in coming chapters often do.\n\nThis commit's `eval` is the entire constant-expression implementation that ships through the next several chapters. It's small (one switch, thirty arms), lives in `parse.c`, and has no friends elsewhere in the codebase.\n\n### Where we are\n\n`const_expr` and `eval` exist. Constant expressions work in array dimensions, `case` values, and enum-constant assignments. The temporary `get_number` from \u00a711.12 is replaced. The chapter closes.\n\n---\n\n## Recap\n\nTwenty-one commits, fifteen sections. The net change is large: chibicc gains every C operator that wasn't already in place \u2014 compound assignment, increment/decrement, modulo, bitwise, shifts, logical, conditional \u2014 and every common control-flow construct beyond `if`/`for`/`while` \u2014 `goto`, labeled statements, `break`, `continue`, `switch`/`case`/`default`. By the end of the chapter the language is much closer to recognizable C.\n\nPer the Chapter 10 closer's prediction, the chapter splits the recap into two themed tables.\n\n| Commit | Topic |\n|---|---|\n| `a4fea2b` | `for` accepts a local declaration in the init slot. New scope opens at the `for`. |\n| `01a94c0` | `+=`, `-=`, `*=`, `/=`. Lowered to `(tmp = &A, *tmp = *tmp op B)` via the \u00a78.5 generalized-lvalue comma extension. The \u00a78.5 prediction closes here. |\n| `47f1937` | Pre `++` and `--`. Lowered to `i += 1` / `i -= 1`. |\n| `e8ca48c` | Post `++` and `--`. Lowered to `(typeof i)((i += 1) - 1)`. |\n| `7df934d` | Hex (`0x`), octal (leading `0`), binary (`0b`) literals. Tokenizer change. |\n| `6b88bcb` | `!`. `cmp $0; sete; movzx`. Result type `int`. |\n| `46a96d6` | `~`. `not %rax`. Result type matches operand. |\n| `daa7398` | `%` and `%=`. Reuses `idiv` codegen with one extra `mov %rdx, %rax`. |\n| `8644006` | `&`, `|`, `^` and three compound-assigns. Three new precedence layers. |\n| `f30f781` | `&&` and `||`. Short-circuit via labels. Result `0` or `1`, type `int`. |\n\n| Commit | Topic |\n|---|---|\n| `29ed294` | Incomplete arrays: `int x[]`. `array_of(ty, -1)` sentinel. Rejected outside specific contexts. |\n| `7963221` | Function-parameter array decay: `int f(int x[])` \u2261 `int f(int *x)`. |\n| `61a1055` | Forward-declared structs: `struct foo *bar;` valid even without `struct foo`'s body. |\n| `6116cae` | `goto` and labeled statements. Labels are a function-scoped fourth namespace. Two-pass label resolution. |\n| `a4be55b` | `compound_stmt` peeks `tok->next == \":\"` to disambiguate label-vs-typedef. |\n| `b3047f2` | `break`. Reuses `ND_GOTO`. Save-install-restore of `brk_label` around each loop. |\n| `3c83dfd` | `continue`. Same pattern with a parallel `cont_label`. |\n| `044d9ae` | `switch`/`case`/`default`. Comparison-and-jump dispatch. Temporary `get_number` for case values, replaced by \u00a711.15. |\n| `d0c0cb7` | `<<`, `>>`, `<<=`, `>>=`. Single-instruction codegen via `shl`/`sar`. Longest-match tokenizer reordering. |\n| `447ee09` | `?:`. New `ND_COND` with branching codegen. `usual_arith_conv` between then and else. |\n| `79f5de2` | `eval` and `const_expr`. AST-walking integer-folding evaluator. Replaces `get_number` everywhere. |\n\nThree structural shifts deserve repeating.\n\nThe first is the *closure of the \u00a78.5 prediction*. The Chapter 8 prose said the comma operator's generalized-lvalue extension was unused at the time and predicted a `+=`-style construct as its consumer. \u00a711.2 closed the loop. From there, every compound-assign operator (`+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`) and every increment operator (`++`, `--` in both pre and post forms) routed through `to_assign` and so through the comma extension. By the end of the chapter, twelve C operators canonicalize through that one mechanism. The \u00a78.5 commit's worth \u2014 small at the time \u2014 is now load-bearing.\n\nThe second is the *fourth-namespace addition*. After Chapter 9's struct/union member namespace and Chapter 10's typedef-and-enum sharing of `vars`, labels are the fourth namespace \u2014 function-scoped, separate from any block. The label-vs-typedef parser conflict (\u00a711.10) resolves via one-token lookahead in `compound_stmt`. The pattern continues a small theme from Chapters 7, 9, 10: when chibicc's parser hits a place where C's grammar isn't context-free, the answer is local lookahead, not a separate symbol-table mechanism.\n\nThe third is the *constant-expression evaluator*. `eval` (\u00a711.15) is small \u2014 one switch, thirty arms, eighty-odd lines. But it's the load-bearing piece of three Chapter 12 commits, the bit-field-width handling that comes later, and several more. Until \u00a711.15, chibicc could only handle constant-shaped contexts where a single integer literal sufficed. After \u00a711.15, full constant expressions work. The chapter closes by setting up Chapter 12.\n\nA few standing notes carried forward to Chapter 12:\n\n- The *canonicalization-at-parse-time* count was six at the end of Chapter 10. This chapter adds two: compound-assign-via-comma (the \u00a711.2 mechanism, which \u00a711.6, \u00a711.7, \u00a711.13 reuse \u2014 counted as one) and pre/post-increment-via-compound-assign (\u00a711.3 \u2014 counted as one). The new count is eight.\n- The *pre-factor-before-feature* count remains at four. \u00a711.1 (`for` locals) is a small enabler for tests that follow but the chapter doesn't formally count it as a pre-factor: the feature it enables is a *style of test*, not a future commit.\n- The *lookahead-by-probe* family adds \u00a711.10 (label-vs-typedef) \u2014 the fourth instance after \u00a710.3 (nested declarators), \u00a710.7 (abstract declarators), and \u00a710.6 (`is_typename`).\n- The *ND_GOTO node reuse* for `break` and `continue` is a small cute trick: by parsing them as `ND_GOTO` with a pre-set `unique_label`, no codegen change is needed.\n- The *temporary scaffolding pattern*: \u00a711.12 used `get_number` for case values, knowing it would be replaced; \u00a711.15 made the replacement. This is the chapter's only example of \"ship a scaffold, swap it later.\" Most chibicc simplifications stay.\n- The *everything-fits-in-rax* invariant continues to hold for all new operators. Each binary-operator codegen leaves its result in `%rax` (or `%eax` for narrow types). `&&` and `||` and `?:` produce `0` or `1` (or one arm's value, for `?:`) in `%rax`.\n- The *argreg 8/16/32/64 split* from \u00a710.1 and \u00a710.2 is unchanged.\n- The *cast machinery* from \u00a710.9\u201310.12 is the load-bearing piece for `?:`, the shift result type, and the const-expr cast-folding.\n- The *tag namespace* from Chapter 9 \u00a79.4 + Chapter 10 \u00a710.14 is unchanged.\n- The *VarAttr channel* (`is_typedef` and `is_static`) is unchanged. Chapter 13 will add `is_extern`.\n- The *struct forward-declaration mutation in place* (`*sc->ty = *ty;`) in \u00a711.9 is the first time chibicc's parser modifies an already-registered tag's type. Watch for this when Chapter 12 introduces flexible array members and Chapter 13 introduces global initializers.\n\nForward references for Chapter 12:\n\n- `eval` will be called for compile-time-constant initializers \u2014 `int x = 1 + 2;` at file scope.\n- The \u00a711.9 incomplete-array machinery feeds into Chapter 12's array initializers (`int x[] = {1, 2, 3};` deduces size from initializer count).\n- The \u00a711.9 incomplete-struct machinery interacts with Chapter 12's struct initializers (`struct point p = {1, 2};`).\n- Chapter 12 is the densest arc in the compiler at nineteen commits; expect substantial pre-factoring inside the chapter.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w -l /Users/cell/claude/chibicc-book/chapters/11-all-the-operators.md",
  "description": "Word count of new chapter"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 011 README and chapter 10 for voice",
      "activeForm": "Reading session 011 README and chapter 10 for voice",
      "status": "completed"
    },
    {
      "content": "Scan all 21 commits for Ch 11",
      "activeForm": "Scanning all 21 commits for Ch 11",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading each commit's diff in detail",
      "status": "completed"
    },
    {
      "content": "Read destination state at last commit",
      "activeForm": "Reading destination state at last commit",
      "status": "completed"
    },
    {
      "content": "Draft chapters/11-all-the-operators.md",
      "activeForm": "Drafting chapters/11-all-the-operators.md",
      "status": "completed"
    },
    {
      "content": "Write session 012 README and HANDOFF for session 013",
      "activeForm": "Writing session 012 README and HANDOFF for session 013",
      "status": "in_progress"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline 79f5de2..HEAD | head -25 && echo \"---next 19 commits after 79f5de2---\" && git log --reverse --oneline 79f5de2..HEAD | head -25",
  "description": "Get the next 19 commits for Ch 12 handoff"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -A 30 \"Chapter 12\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | head -50",
  "description": "Check Ch 12 mapping"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --reverse --oneline 79f5de2..HEAD | head -19 | nl",
  "description": "Get exact 19 commits for Ch 12"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/012-chapter-11-draft",
  "description": "Create session 012 dir"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/012-chapter-11-draft/README.md",
  "content": "# Session 012 \u2014 Chapter 11 draft\n\n**Date:** 2026-05-09 (continuation of the autonomous-drafting run that has produced sessions 002\u2013011).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 011 delivered Ch 10 (Filling out the type system, twenty commits, ~14,800 words \u2014 the largest chapter so far by word count). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 11 covers commits 76\u201396: for-loop locals, compound assignment, increment/decrement, number-literal bases, `!`/`~`, `%`/`%=`, bitwise + their compound-assigns, `&&`/`||`, incomplete arrays/forward struct decls/parameter array decay, `goto`/labels, `break`/`continue`, `switch`/`case`/`default`, shifts + their compound-assigns, `?:`, constant expressions. Twenty-one commits \u2014 the second-largest chapter by commit count.\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~12,260 words. Below the handoff target of 15,000\u201317,000 but covering all twenty-one commits. The chapter ran shorter than predicted because per-commit prose for operator codegen ends up tighter than per-commit prose for type-system commits \u2014 most operators are one or three lines of x86-64, and there's no equivalent of \u00a710.5's declspec rewrite or \u00a710.10's usual-arithmetic-conversion arc to extend prose. Compares to: Ch 10 (~14,800), Ch 7 (~13,800), Ch 9 (~9,300), Ch 8 (~7,400).\n- **Section structure:** 15 sections, no concept interlude. Followed the handoff's bundling proposal closely:\n  - \u00a711.3 bundled commits 78, 79 (pre and post `++`/`--`). Per handoff. The post-form's `(typeof A)((A += 1) - 1)` lowering is the section's centerpiece.\n  - \u00a711.5 bundled commits 81, 82 (`!`, `~`). Per handoff. Both are minimal one-or-three-line codegen commits.\n  - \u00a711.9 bundled commits 86, 87, 88 (incomplete arrays, parameter array decay, forward struct decls). Per handoff. The unifying mechanism is the `size = -1` sentinel.\n  - \u00a711.10 bundled commits 89, 90 (`goto`/labels and the typedef conflict resolution). Per handoff.\n  - \u00a711.11 bundled commits 91, 92 (`break`/`continue`). Per handoff. Both reuse `ND_GOTO`.\n- **No concept interlude.** The handoff floated `goto` and structured programming as a possible interlude topic; the \u00a711.10 prose didn't surface a need. Default-no held.\n- **\u00a711.2 closes the \u00a78.5 generalized-lvalue comma loop.** Per handoff acceptance criterion. The prose explicitly names \u00a78.5's prediction and confirms the loop closes here. The canonicalization-at-parse-time count update is named: six \u2192 eight (compound-assign + pre/post-increment counted as two mechanisms).\n- **\u00a711.5 (`!`) explicitly parallels the \u00a710.12 `_Bool` cast.** Per handoff. The prose names them as mirror-image one-letter variants (`sete` vs. `setne`).\n- **\u00a711.8 (`&&`/`||`) covers short-circuit codegen with labels.** Per handoff. The prose walks both `ND_LOGAND` and `ND_LOGOR` codegen as mirror-images and notes the C requirement that the result be exactly `0` or `1`, not the operand value.\n- **\u00a711.9 walks function-param array-decay explicitly.** Per handoff. The prose names the C-standard rule, shows the `if (ty2->kind == TY_ARRAY) { ty2 = pointer_to(ty2->base); }` swap, and notes the test that pins down `int x[]` \u2261 `int *x` at the calling-convention level.\n- **\u00a711.10 names labels as the fourth namespace.** Per handoff. The prose: \"Labels are a *fourth namespace* in C. Variables and typedef names share one. Struct/union/enum tags share another. Members within each struct/union live in a per-struct namespace. Labels are the fourth \u2014 they belong to a function, not a block.\" The label-vs-typedef parser conflict is walked with the one-token lookahead in `compound_stmt` named explicitly.\n- **\u00a711.12 covers fallthrough and notes the const-expression dependency.** Per handoff. The prose names the temporary `get_number` for case values and forward-references \u00a711.15 as the replacement.\n- **\u00a711.15 introduces `eval` and previews Ch 12/Ch 13 callers.** Per handoff. The prose names `eval`'s structure (one switch, thirty arms) and its forward callers (Ch 12 initializers).\n- **Two tables in the recap.** Per the handoff's prediction; same split logic as Ch 10 (operators on one side, control flow + types on the other).\n\n### Interpretive calls\n\n1. **Counting canonicalization-at-parse-time instances.** The handoff said \"treat related variants as one mechanism.\" Ran with that. Compound-assign (the `to_assign` mechanism that handles `+=`/`-=`/`*=`/`/=`/`%=`/`&=`/`|=`/`^=`/`<<=`/`>>=`) is *one* mechanism. Pre/post-increment (which adds the cast-back step on top) is a second. So count goes 6 \u2192 8. Did *not* count `?:` as a canonicalization, because it ships as a new node kind with branching codegen rather than desugaring into existing AST shapes. Did *not* count `&&`/`||` for the same reason.\n2. **Counting pre-factor-before-feature instances.** \u00a711.1 (for-loop locals) is a small enabler for tests that follow but the chapter doesn't formally count it as a pre-factor. The pre-factor pattern is \"code in commit N supports a feature in commit M > N\"; \u00a711.1 enables a *style of test*, not a future commit. Count remains at four.\n3. **The `ND_GOTO` reuse for `break`/`continue`.** Called this out as a \"small cute trick\" in the recap. The \u00a711.11 prose explicitly notes the reuse: rather than `ND_BREAK` and `ND_CONTINUE`, both are `ND_GOTO` with a pre-set `unique_label`.\n4. **The temporary-scaffolding pattern.** \u00a711.12 uses `get_number` for case values, knowing it'll be replaced in \u00a711.15. The recap names this as the chapter's only \"ship a scaffold, swap it later\" instance.\n5. **The label-vs-typedef-name two-namespace point.** The \u00a710.6 prose framed typedef-vs-variable name sharing as the same namespace. \u00a711.10 needs to say labels are *separate* \u2014 and explicitly distinct from `vars`/`tags`/`members`. Wrote it in the \u00a711.10 prose with a one-paragraph C99-style namespace recap.\n6. **`?:` does not canonicalize.** The handoff was open to either a canonicalization or a new node kind for `?:`. Rui ships it as a new node kind (`ND_COND` with branching codegen). The \u00a711.14 prose notes this and explains why: there's no clean way to express short-circuiting via existing AST shapes.\n7. **The `>>` codegen quirk.** Both arms of `if (node->ty->size == 8)` in `ND_SHR` emit the same instruction (`sar`). Probably an unfinished differentiation in Rui's commit. Noted in the \u00a711.13 prose as \"perhaps Rui intended `sar` for 8-byte and a different mnemonic for 4-byte and didn't finish the differentiation,\" without re-litigating; this is a candidate for the errata appendix.\n\n### Voice / structure inherited from Ch 1\u201310\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote (multiple openers for bundled sections).\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature tables (two tables, by theme \u2014 operators vs. control-flow + types).\n\n### Three careful avoidances\n\n- **Did not introduce a label-namespace concept-interlude.** The handoff defaulted to no interlude; held to that.\n- **Did not over-explain the `to_assign` mechanism in three sections.** \u00a711.2 walks it in detail. \u00a711.3 (pre/post-increment), \u00a711.6 (`%=`), \u00a711.7 (`&=`/etc.), \u00a711.13 (`<<=`/`>>=`) reference it without re-deriving.\n- **Did not re-explain `usual_arith_conv` in \u00a711.14.** The \u00a710.10 prose introduced it; \u00a711.14 names the mechanism and lets it stand.\n\n### Date-vs-position note\n\nRoughly a third of the commits are dated August 2019; about half are October 2020 (Rui's late-year cleanup); a few are scattered between (April 2020, August 2020, September 2020). Per the same pattern as Chs 7\u201310, position on `main` doesn't match chronology \u2014 commit 78 (pre-increment) is dated 2020-10-07, commit 79 (post-increment) is dated 2020-04-13, but `main` order is 78 \u2192 79. Walked the case briefly in \u00a711.3 prose.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **The \u00a78.5 generalized-lvalue comma loop is closed as of \u00a711.2.** The \u00a78.5 prose's prediction is now confirmed in print. The mechanism handles ten compound-assign operators plus four increment-form variants \u2014 twelve C operators canonicalize through `to_assign` and so through the comma extension. Load-bearing.\n- **The fourth namespace (labels) is in.** As of \u00a711.10, chibicc's namespace landscape has four flavors: variables/typedefs/enum-constants (the `vars` chain), tags (the `tags` chain), per-struct members (linear search inside each struct), and labels (function-scoped, separate from any `Scope`). Watch for further interactions when Ch 12 introduces initializer scopes (it shouldn't add a fifth namespace; struct-member-name lookup will be reused) and Ch 13's `extern` adds a fifth `VarAttr` flag through the same channel as `is_typedef`/`is_static`.\n- **The `eval` function (\u00a711.15) is small but load-bearing.** It will get more callers in Ch 12 (initializers \u2014 file-scope `int x = 1+2;`) and Ch 13 (`_Alignof` / `_Alignas`). Currently lives in `parse.c`, has no external callers.\n- **Pre-factor-before-feature count remains at four.** \u00a711 added zero formal pre-factors. (\u00a711.1's for-loop locals are a pre-factor for *tests*, not for any future commit.)\n- **Canonicalization-at-parse-time count is now eight.** Compound-assign (counted as one mechanism via `to_assign`, regardless of which `op=` it is) plus pre/post-increment (counted as one mechanism, with the postfix cast-back as a variant). The count's pace will slow from here \u2014 Ch 12 doesn't add any obvious canonicalizations.\n- **The `ND_GOTO` reuse pattern.** `break` and `continue` reuse `ND_GOTO` rather than introducing distinct node kinds. Trick worth remembering when Ch 13 adds `return;` (bare return) \u2014 it might similarly reuse an existing node.\n- **The temporary `get_number` \u2192 `const_expr` swap pattern.** \u00a711.12 used a placeholder for case values; \u00a711.15 replaced it. The pattern is unusual for chibicc, but if it recurs in Ch 12 (initializer-list-element-count or similar), watch for it.\n- **The `>>` arithmetic-shift-only codegen.** When Ch 14 adds unsigned types, the \u00a711.13 codegen will need to branch on signedness \u2014 `sar` for signed, `shr` for unsigned. Currently both arms of the size-8 dispatch emit `sar`; the dispatch itself is a no-op. Noted in the chapter prose; carry-forward errata candidate.\n- **The `>>` codegen has a bug-shaped artifact.** Both arms of `if (node->ty->size == 8)` emit `sar %cl, %s` \u2014 the `if` is structurally unused. Errata candidate; not fixed in the prose.\n- **The `_Bool` cast and `!` are mirror-images.** \u00a710.12 emits `setne`; \u00a711.5 emits `sete`. One-letter difference. The pairing is a small piece of mental shorthand worth keeping.\n- **The structural difference between `ND_COND` and `&&`/`||` codegen.** All three short-circuit; all three need labels. `?:` chooses between two arms; `&&`/`||` produce `0` or `1`. Different shapes; noted in \u00a711.14 prose.\n- **Labels are unique by `new_unique_name`.** The `.L..NNNN` naming convention used by chibicc since \u00a73 generates labels that won't collide across functions. Chapter 12 doesn't touch this; preserve.\n- **The `is_typename`+`tok->next == \":\"` lookahead in `compound_stmt`** is the load-bearing part of the label-vs-typedef resolution. The lookahead is local to one site; no broader mechanism is introduced. Watch for whether Ch 12's initializer-list parsing has similar conflicts.\n- **The struct forward-decl-mutation pattern (\u00a711.9).** `*sc->ty = *ty;` overwrites the previously-incomplete tag's type with the now-complete one. This is the first time chibicc's parser mutates an already-registered tag. Watch for when Ch 12 adds flexible array members (which extend struct types in-place) and Ch 13 adds global initializers.\n- **The `for`-loop-local scope (\u00a711.1)** opens at the `for` keyword, not at `{`. Same shape as the \u00a78.1 block-scope mechanism but anchored at a different parser point.\n- **The `array_of(ty, -1)` sentinel** for incomplete arrays will get more uses. Ch 12's `int x[] = {1, 2, 3};` will deduce the size from the initializer count and patch the type's `array_len` field after parsing. Watch for `array_len < 0` checks.\n- **The Ch 9 errata-correction (\u00a710.14)** is a stable closed loop. Chapter 11 doesn't revisit.\n- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.\n- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 \u00a75.1. Pending footnote for revision pass.\n- **The \"more than 6 args silently miscompiles\"** in Ch 5 \u00a75.4. Errata candidate.\n- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 \u00a77.5) rejects void-returning bodies. Errata candidate.\n- **The hex-escape silent truncation** (Ch 7 \u00a77.4). Errata candidate.\n- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and now labels. Four errata candidates. (`goto a; a:; a:;` is silently accepted in chibicc; the second `a:` overwrites the first label entry.)\n\n## Exit state\n\n- `chapters/11-all-the-operators.md` drafted, ~12,260 words.\n- Session 012 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 013 (Chapter 12 \u2014 Initializers, commits 97\u2013115; nineteen commits, the densest arc in the compiler per the chapter mapping).\n- CLAUDE.md status note will need updating (chapter count goes from \"Ch 10 drafted\" to \"Ch 11 drafted\").\n"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/012-chapter-11-draft/HANDOFF.md",
  "content": "# Handoff: Ch 11 done \u2192 proceed to Ch 12\n\n**For:** the next claude session.\n**From:** session 012.\n**Status:** Ch 11 drafted (~12,260 words, twenty-one commits, the second-largest by commit count). Continue autonomously to Ch 12 (Initializers). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/012-chapter-11-draft/README.md`](README.md)** \u2014 what session 012 did, including the closure of the \u00a78.5 generalized-lvalue comma loop in \u00a711.2, the framing of labels as the fourth namespace in \u00a711.10, the `ND_GOTO` reuse pattern for `break`/`continue` in \u00a711.11, the temporary-`get_number`-then-`const_expr`-swap pattern in \u00a711.12 and \u00a711.15, and the canonicalization-at-parse-time count update from six to eight.\n2. **[`docs/sessions/011-chapter-10-draft/HANDOFF.md`](../011-chapter-10-draft/HANDOFF.md)** \u2014 the previous handoff. Many standing notes still apply; `is_typename` is now coupled to the symbol table (since \u00a710.6) and to a one-token lookahead (since \u00a711.10).\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`11-all-the-operators.md`](../../../chapters/11-all-the-operators.md)** \u2014 the eleven chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 12 = commits 97\u2013115.\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes.\n\n## Chapter 12 scope\n\n**Title (working):** *Initializers*.\n**Commits:** 97\u2013115 in chronological order on `main`. **Nineteen commits \u2014 the densest arc in the compiler per the chapter mapping.**\n**Concept interlude:** None obvious. Possibly an interlude on \"what an initializer-list grammar means\" or \"the local-vs-global initializer split\" if the prose calls for it; default to no interlude.\n\n| # | Hash | Subject |\n|---|---|---|\n| 97 | `22dd560` | Support local variable initializers |\n| 98 | `ae0a37d` | Initialize excess array elements with zero |\n| 99 | `a754732` | Skip excess initializer elements |\n| 100 | `0d71737` | Add string literal initializer |\n| 101 | `5b95533` | Allow to omit array length if an initializer is given |\n| 102 | `e9d2c46` | Handle struct initializers for local variables |\n| 103 | `aca19dd` | Allow to initialize a struct with other struct |\n| 104 | `483b194` | Handle union initializers for local variables |\n| 105 | `bbfe3f4` | Add global initializer for scalar and string |\n| 106 | `eeb62b6` | Add struct initializer for global variable |\n| 107 | `1eae5ae` | Handle union initializers for global variable |\n| 108 | `efa0f33` | Allow parentheses in initializers to be omitted |\n| 109 | `a58958c` | Allow extraneous braces for scalar initializer |\n| 110 | `fde464c` | Allow extraneous comma at the end of enum or initializer list |\n| 111 | `3d216e3` | Emit uninitialized global data to .bss instead of .data |\n| 112 | `824543b` | Add flexible array member |\n| 113 | `cd688a8` | Allow to initialize struct flexible array member |\n| 114 | `7a1f816` | Accept `void` as a parameter list |\n| 115 | `157356c` | Align global variables |\n\nThis is the chapter where chibicc's initializers go from \"scalar at definition\" to \"the full C grammar of initializers.\" Nineteen commits is too many for one section per commit. **Bundle aggressively, again.** Rough proposal:\n\n- **\u00a712.1 \u2014 Local scalar and array initializers** (commit 97). The base case \u2014 `int x = 5;` and `int x[3] = {1, 2, 3};`. Sets up the `Initializer` data structure, the `initializer` and `lvar_initializer` parser, and the lowering to a series of assignments. Probably the chapter's largest section because the data structure is built here.\n- **\u00a712.2 \u2014 Initializer edge cases for arrays** (commits 98, 99). Bundle. Excess elements get zero (for `int x[5] = {1, 2}`); excess initializers are silently dropped (for `int x[2] = {1, 2, 3, 4}`). Both small.\n- **\u00a712.3 \u2014 String literal initializers** (commits 100). `char x[5] = \"abc\";` \u2014 one of C's stranger quirks where a string literal can initialize a `char[]`. Substantive.\n- **\u00a712.4 \u2014 Length-omitted arrays from initializers** (commit 101). `int x[] = {1, 2, 3};` deduces size 3. Closes the \u00a711.9 `array_of(ty, -1)` sentinel \u2014 the parser can now patch the size after seeing the initializer. Substantive.\n- **\u00a712.5 \u2014 Struct and union local initializers** (commits 102, 103, 104). Bundle. Local-scope `struct point p = {1, 2};`. Commit 103 adds the *struct-from-struct* case (`struct point p = q;`). Commit 104 adds union initializers. Substantive.\n- **\u00a712.6 \u2014 Global initializers \u2014 scalar and string** (commit 105). The split: globals can't use the assignment-lowering trick, so a separate code path emits `.data` directives. Substantive.\n- **\u00a712.7 \u2014 Global struct and union initializers** (commits 106, 107). Bundle. Same shape as the local commits but routed through the `.data` emitter. Medium.\n- **\u00a712.8 \u2014 Initializer-list ergonomics** (commits 108, 109, 110). Bundle. Three small commits about syntax: omit braces (`int x[2] = 1;` is the same as `{1, 0}`); allow extraneous braces (`int x = {5};`); allow trailing comma in enum or initializer list. Each small; bundled.\n- **\u00a712.9 \u2014 `.bss` for uninitialized globals** (commit 111). One small commit: zero-initialized global data goes in `.bss` instead of `.data` (saves space in the object file). Small.\n- **\u00a712.10 \u2014 Flexible array members** (commits 112, 113). Bundle. `struct s { int n; int data[]; };` \u2014 the trailing array with no size. Initialization (commit 113) is the unusual case. Substantive.\n- **\u00a712.11 \u2014 `void` as a parameter list and global alignment** (commits 114, 115). Bundle? Two unrelated commits at the chapter's end. `f(void)` means \"no parameters\" (vs `f()` which is \"unspecified parameters\"). Global variables now get `.align` directives. Both small; bundled because they don't fit elsewhere.\n\nThat's ~11 sections from 19 commits. **Target chapter length: ~13,000\u201315,000 words.** Roughly Ch 11's size \u2014 slightly larger because the initializer data structure is the chapter's most complex single thing and needs walked-through detail.\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in 22dd560 ae0a37d a754732 0d71737 5b95533 e9d2c46 aca19dd 483b194 bbfe3f4 eeb62b6 1eae5ae efa0f33 a58958c fde464c 3d216e3 824543b cd688a8 7a1f816 157356c; do echo \"===== $h =====\"; git show --stat $h | head -8; done` to scan all nineteen diffs.\n2. Read each commit. Pay particular attention to:\n   - **`22dd560`** (commit 97): the local initializer base case. The `Initializer` struct (parallel to `Type`?), the `initializer` recursive parser, and the `create_lvar_init` lowering. The chapter's load-bearing data structure lives here.\n   - **`ae0a37d`, `a754732`** (commits 98, 99): excess-array-element handling. Likely small `if` checks in the array-initializer arm.\n   - **`0d71737`** (commit 100): string literal initializers. The `char x[] = \"abc\"` case is special \u2014 the string isn't a single `ND_STR` node but a per-byte sequence. Watch for whether chibicc treats it as `{a, b, c, 0}` syntactically.\n   - **`5b95533`** (commit 101): array-length-from-initializer. The `array_of(ty, -1)` sentinel from \u00a711.9 gets its first user.\n   - **`e9d2c46`, `aca19dd`, `483b194`** (commits 102, 103, 104): struct/union initializers. The struct-from-struct case (commit 103) is interesting because it's not a brace-list \u2014 it's `struct point p = q;` with an existing struct on the right.\n   - **`bbfe3f4`, `eeb62b6`, `1eae5ae`** (commits 105, 106, 107): global initializers. Different code path \u2014 global initializers emit `.data` directives at compile time, not runtime assignments.\n   - **`824543b`, `cd688a8`** (commits 112, 113): flexible array members. The trailing-array-with-no-size in struct definitions. Inititaliziation extends the struct's size.\n   - **`7a1f816`** (commit 114): `f(void)` parameter list. C's `f()` and `f(void)` distinction is a parser quirk; the implementation is a one-line check.\n3. Read the destination state at `157356c` (or shortly after) for `chibicc.h`, `parse.c`, `codegen.c`, all relevant test files. The Ch 12 surface is *narrow* (most changes live in `parse.c`'s initializer handlers and `codegen.c`'s `.data`/`.bss` emission) but *deep* (the initializer data structure is the chapter's centerpiece).\n4. Draft `chapters/12-initializers.md`. Likely 13,000\u201315,000 words.\n5. Write `docs/sessions/013-chapter-12-draft/README.md`.\n6. Write `HANDOFF.md` for session 014 (Chapter 13 \u2014 Linkage, commits 116\u2013126; eleven commits including `extern`, `_Alignof`/`_Alignas`, static locals, compound literals, do-while, return-without-value, 16-byte stack alignment).\n\n## Voice / structure rules\n\nSame as Ch 1\u201311:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 nineteen rows; consider splitting into two tables by theme (local-scope vs. global-scope) as Ch 10 and Ch 11 did.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The `Initializer` struct (probably) is a candidate for full-snippet display the first time it appears.\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist.\n- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **Bundle aggressively.** Nineteen commits won't fit one-per-section at any reasonable chapter length.\n- **\u00a712.1's data structure walk is the chapter's anchor.** The `Initializer` struct (whatever it ends up being called) is referenced by every later section. Walk it carefully the first time.\n- **The local-vs-global split is the chapter's central tension.** Local initializers lower to assignments at runtime; global initializers emit data directives at compile time. The split shows up as two parallel code paths. Name the split explicitly.\n- **String literals as initializers are weird.** `char x[5] = \"abc\"` is *syntactically* a string literal but *semantically* equivalent to `char x[5] = {'a', 'b', 'c', 0, 0}`. Walk the expansion.\n- **Flexible array members (\u00a712.10) extend the \u00a711.9 incomplete-struct mechanism.** A flexible array is essentially \"the last member is an incomplete array of some element type.\" Watch for whether chibicc reuses the `array_of(ty, -1)` sentinel or introduces a new flag.\n- **`void` as a parameter list (\u00a712.11) is a one-line check** but it's *structurally distinct* from \"no parameters.\" `f()` accepts any arguments (legacy K&R); `f(void)` accepts none. Pin down the test that distinguishes them.\n- **Global alignment (\u00a712.11) is small but interacts with \u00a712.6's `.data` emission.** The `.align N` directive precedes each global. Walk the emitted assembly.\n- **The `.bss` shift (\u00a712.9) is a small, isolated commit** but worth a section because it's the first time chibicc emits `.bss` (vs. `.data`).\n- **Watch for the `Initializer` data structure's relationship to `Type`.** If chibicc parallels them (one-to-one, one struct per `Type` shape), name the parallel. If it's a more general AST-of-initializer, walk that.\n\n## Standing notes worth tracking across sessions\n\n- **Ch 1 errata list** unchanged: test-count off-by-one and missing note that `skip()` was upgraded from `error()` to `error_tok()` in commit `cc5a6d9`.\n- **The `mov $0, %rax`** (variadic `%al`-zeroing) noted in Ch 5 \u00a75.1. Pending footnote for revision pass.\n- **The \"more than 6 args silently miscompiles\"** in Ch 5 \u00a75.4. Errata candidate.\n- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 \u00a77.5) rejects void-returning bodies. Errata candidate.\n- **The hex-escape silent truncation** (Ch 7 \u00a77.4). Errata candidate.\n- **The redeclaration-in-same-scope check** is missing for variables, tags, typedef names, and labels. Four errata candidates.\n- **Block scope is established** as of Ch 8 \u00a78.1. Tag scope from Ch 9 \u00a79.4. Typedef-name scope (sharing `vars`) from Ch 10 \u00a710.6. Enum-constant scope (also `vars`) from Ch 10 \u00a710.14. Label namespace (function-scoped, separate) from Ch 11 \u00a711.10. Watch for a fifth Ch 12 addition (probably none \u2014 initializer scopes likely reuse block scope).\n- **Per-token line numbers** (Ch 8 \u00a78.3) used by `.loc` and error-tok throughout. Preserve when the preprocessor lands in Ch 17.\n- **GDB-debuggable output** (Ch 8 \u00a78.4) \u2014 already taken for granted.\n- **Tests are in C** as of Ch 8 \u00a78.2. New language features get tests in `test/<feature>.c`.\n- **The host-cc-as-preprocessor pipeline** (Ch 8 \u00a78.2) collapses in Ch 17.\n- **The comma operator's generalized-lvalue extension** (Ch 8 \u00a78.5) is the load-bearing mechanism for compound assignment, closed in Ch 11 \u00a711.2. By Ch 12 the loop is closed.\n- **Canonicalization-at-parse-time count is now eight** (compound-assign + pre/post-increment, plus the six from before Ch 11). Ch 12's initializers are a *form* of canonicalization (initializer \u2192 assignments) but the chapter conventionally calls them \"initializer lowering,\" not \"canonicalization.\" Decide whether to count them as a ninth instance \u2014 if so, name the call-out in \u00a712.1.\n- **Pre-factor-before-feature is a four-instance named pattern.** Ch 12's `array_of(ty, -1)` reuse from Ch 11 \u00a711.9 is an instance \u2014 the sentinel was set up in Ch 11 and is consumed in Ch 12 \u00a712.4 and \u00a712.10. Possibly count this as a fifth.\n- **The argreg 8/16/32/64 split** is fully in place. Ch 12 doesn't touch it.\n- **The `is_typename` helper is a context-sensitive predicate** as of Ch 10 \u00a710.6 + Ch 11 \u00a711.10. Ch 12's parser additions probably don't extend it; initializer lists don't introduce a typename ambiguity.\n- **The cast machinery** is Ch 10's. Ch 12's initializers likely use `new_cast` for type-promoting initializers (`int x = (long)5;`).\n- **The `format` helper landed in Ch 7 \u00a77.3.** Workhorse going forward.\n- **The trailing-newline guarantee in `read_file`** (Ch 7 \u00a77.6) protects line-comment skipping.\n- **The lookahead-by-probe pattern** is now a four-instance family: \u00a77.1, \u00a710.3, \u00a710.7, \u00a710.6, \u00a711.10.\n- **The everything-fits-in-rax codegen invariant** continues. Ch 12 doesn't touch.\n- **The `array_of(ty, -1)` sentinel** for incomplete arrays (Ch 11 \u00a711.9) gets its first real consumer in Ch 12 \u00a712.4 (length-from-initializer).\n- **The `eval` function** (Ch 11 \u00a711.15) gets more callers in Ch 12 (compile-time-constant initializers \u2014 `int x = 1+2;` at file scope).\n- **The struct forward-decl mutation pattern** (`*sc->ty = *ty;` in Ch 11 \u00a711.9) repeats in Ch 12 \u00a712.4 \u2014 array length is patched in place after the initializer is parsed.\n- **The `unreachable()` macro** (Ch 10 \u00a710.1) lives in `chibicc.h`. Used by `store_gp`, `declspec`, and possibly more in Ch 12.\n- **The `VarAttr` channel** (Ch 10 \u00a710.6, extended in \u00a710.15) currently carries `is_typedef` and `is_static`. Ch 13's `extern` adds a third flag.\n- **The `ND_GOTO` reuse for `break`/`continue`** (Ch 11 \u00a711.11) is a chapter-internal trick worth remembering. Ch 13's bare `return;` may use a similar reuse.\n- **Labels are the fourth namespace** as of Ch 11 \u00a711.10. Watch for a fifth in Ch 12 (probably none).\n\n## Acceptance criteria for Ch 12\n\n- [ ] `chapters/12-initializers.md` exists, end-to-end readable.\n- [ ] All nineteen commits covered, grouped into ~11 sections (or fewer with bundling).\n- [ ] \u00a712.1 walks the `Initializer` data structure carefully \u2014 this is the chapter's anchor.\n- [ ] \u00a712.4 names the closure of the \u00a711.9 `array_of(ty, -1)` sentinel.\n- [ ] \u00a712.5 covers the struct-from-struct initializer case explicitly.\n- [ ] \u00a712.6 names the local-vs-global initializer split.\n- [ ] \u00a712.10 (flexible array) explains the relationship to \u00a711.9 incomplete arrays.\n- [ ] \u00a712.11 distinguishes `f()` from `f(void)`.\n- [ ] Each commit has a `git checkout <full-hash>` opener.\n- [ ] Voice matches Ch 1\u201311.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references checked against `chapter-mapping.md`.\n- [ ] `docs/sessions/013-chapter-12-draft/README.md` written.\n- [ ] `HANDOFF.md` written for session 014 (Chapter 13 \u2014 Linkage, commits 116\u2013126).\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/012-chapter-11-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/012-chapter-11-draft/README.md   (what session 012 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md\n13. chapters/11-all-the-operators.md               (most recent chapter)\n14. research/commits/chapter-mapping.md            (confirms Ch 12 scope)\n15. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 12 (Initializers, commits 97\u2013115) per the steps in\nthe handoff. Nineteen commits \u2014 bundle aggressively. End-of-session:\nwrite your session dir under docs/sessions/013-chapter-12-draft/ with a\nREADME and a HANDOFF for session 014 (Chapter 13 \u2014 Linkage, commits\n116\u2013126).\n```\n"
}


---

## 🤖 Claude

Now update CLAUDE.md status note.

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "offset": 60,
  "limit": 20
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 011 README and chapter 10 for voice",
      "activeForm": "Reading session 011 README and chapter 10 for voice",
      "status": "completed"
    },
    {
      "content": "Scan all 21 commits for Ch 11",
      "activeForm": "Scanning all 21 commits for Ch 11",
      "status": "completed"
    },
    {
      "content": "Read each commit's diff in detail",
      "activeForm": "Reading each commit's diff in detail",
      "status": "completed"
    },
    {
      "content": "Read destination state at last commit",
      "activeForm": "Reading destination state at last commit",
      "status": "completed"
    },
    {
      "content": "Draft chapters/11-all-the-operators.md",
      "activeForm": "Drafting chapters/11-all-the-operators.md",
      "status": "completed"
    },
    {
      "content": "Write session 012 README and HANDOFF for session 013",
      "activeForm": "Writing session 012 README and HANDOFF for session 013",
      "status": "completed"
    }
  ]
}


---

## 🤖 Claude

Chapter 11 drafted (~12,260 words, 15 sections, all 21 commits covered). Session 012 dir written with README and HANDOFF for session 013 (Ch 12 — Initializers, commits 97–115). CLAUDE.md status updated.

The chapter ran shorter than the 15K-17K target — operator codegen prose is tighter per commit than type-system prose was in Ch 10. All acceptance criteria from the handoff are met: §11.2 closes the §8.5 generalized-lvalue comma loop, §11.5 names the `!` / `_Bool` mirror, §11.8 walks short-circuit codegen, §11.9 explicitly walks function-param array decay, §11.10 names labels as the fourth namespace and walks the typedef conflict, §11.12 covers fallthrough and notes the const-expr dependency, §11.15 introduces `eval` and previews Ch 12/13 callers. Two-table recap split by theme. Canonicalization-at-parse-time count goes from 6 to 8.

---

## 👤 User

please print the relative path to the handoff at the end of each session

---

## 🤖 Claude

`docs/sessions/012-chapter-11-draft/HANDOFF.md`

Saving as feedback for future sessions.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/.claude/projects/-Users-cell-claude-chibicc-book/memory/feedback_print_handoff_path.md",
  "content": "---\nname: Print handoff path at end of session\ndescription: At the end of each chibicc-book drafting session, print the relative path to the new HANDOFF.md as the last line of the response\ntype: feedback\n---\n\nAt the end of each chibicc-book session (after writing the session README and HANDOFF), print the relative path to the new `HANDOFF.md` (e.g. `docs/sessions/NNN-<slug>/HANDOFF.md`) as part of the closing message.\n\n**Why:** the user runs sessions back-to-back and pastes the handoff path into the next session's prompt block. Having to scroll back through the session output to find it is friction.\n\n**How to apply:** at the end of any session that produces a `docs/sessions/NNN-<slug>/HANDOFF.md`, include the relative path in the final response. A bare path on its own line is fine; no need for a header.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "ls /Users/cell/.claude/projects/-Users-cell-claude-chibicc-book/memory/ 2>/dev/null",
  "description": "Check memory dir contents"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/.claude/projects/-Users-cell-claude-chibicc-book/memory/MEMORY.md",
  "content": "- [Print handoff path at end of session](feedback_print_handoff_path.md) \u2014 print the relative HANDOFF.md path at the end of each chibicc-book drafting session\n"
}


---

## 🤖 Claude

Saved. The handoff for this session:

`docs/sessions/012-chapter-11-draft/HANDOFF.md`

---

