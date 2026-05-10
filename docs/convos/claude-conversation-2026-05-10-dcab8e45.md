# Claude Conversation Log

Session ID: dcab8e45-a8a1-4364-ad38-e74734e9f2f8
Date: 2026-05-10 04:41:10

---

## 👤 User

please read docs/sessions/017-chapter-16-draft/HANDOFF.md and proceed

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/017-chapter-16-draft/HANDOFF.md"
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
      "content": "Read session 017 README and Ch 16",
      "status": "in_progress",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "pending",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "pending",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "pending",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "pending",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "pending",
      "activeForm": "Writing handoff for session 019"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/017-chapter-16-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/16-the-compiler-driver.md"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/16-the-compiler-driver.md",
  "limit": 300
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/chapters && wc -w 16-the-compiler-driver.md",
  "description": "Word count of Ch 16"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/16-the-compiler-driver.md",
  "offset": 1300
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/16-the-compiler-driver.md",
  "offset": 1000,
  "limit": 110
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in 1e1ea39 146c7b3 d367510 ec149f6 d138864 bf6ff92 aa570f3 c6e81d2 e7a1857 97d33ad 9ad60e4 2651448 acce002 1f80f58 dec3b3f b9ad3e4 dd4306c c7d7ce0 1313fc6 8f6f792 8f561ae 769b5a0 5cb2f89 a8d76ad 8075582 b33fe0e d85fc4f a1dd621 a939a7a e7fdc2e 5f5a850 6f17071 dc01f94 ba6b4b6 82ba010 ab4f1e1 7746e4e 7cbfd11 5322ea8 12a9e75; do echo \"===== $h =====\"; git show --stat $h | head -8; done",
  "description": "Scan stat for all 40 Ch17 commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 1e1ea39",
  "description": "Show first commit (do-nothing preprocessor)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 146c7b3 d367510",
  "description": "Show null directive and #include \"...\""
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show ec149f6 d138864",
  "description": "Show skip-extra and -E option"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "in_progress",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "pending",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "pending",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "pending",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "pending",
      "activeForm": "Writing handoff for session 019"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show bf6ff92 aa570f3 c6e81d2 e7a1857",
  "description": "Show #if/#endif/skip-nested/#else/#elif"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 97d33ad 9ad60e4 2651448",
  "description": "Show #define, #undef, expand-in-#if"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show acce002",
  "description": "Show no-rescan-objlike commit (hideset)"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 1f80f58 dec3b3f b9ad3e4",
  "description": "Show ifdef/ifndef, zero-arity funclike, multi-arity funclike"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show dd4306c c7d7ce0 1313fc6",
  "description": "Show empty args, parenthesized args, no-rescan funclike"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 8f6f792 8f561ae",
  "description": "Show stringizing # and pasting ##"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 769b5a0 5cb2f89 a8d76ad",
  "description": "Show test-pipeline switch, defined(), identifier-to-zero"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 8075582 b33fe0e d85fc4f a1dd621",
  "description": "Show preserve newline/space, line continuation, #include &lt;...&gt;, -I"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a939a7a e7fdc2e 5f5a850",
  "description": "Show default include paths, #error, predefined macros"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6f17071 dc01f94 ba6b4b6 82ba010",
  "description": "Show __FILE__/__LINE__/__VA_ARGS__/__func__/__FUNCTION__"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show ab4f1e1 7746e4e 7cbfd11 5322ea8 12a9e75",
  "description": "Show string concat, wide char, headers, va_arg, self-host"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "completed",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "in_progress",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "pending",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "pending",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "pending",
      "activeForm": "Writing handoff for session 019"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout 12a9e75 -- preprocess.c main.c 2>&1; wc -l preprocess.c main.c; git stash 2>&1 | head -5",
  "description": "Inspect dest preprocess.c size"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git stash pop 2>&1 | head -5; git status --short | head -10",
  "description": "Restore working dir"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git status; git stash list",
  "description": "Check repo state"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout HEAD -- preprocess.c main.c 2>&1; git status",
  "description": "Restore main HEAD versions"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 12a9e75:preprocess.c | sed -n '1,30p'",
  "description": "Top of dest preprocess.c"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book && cat research/notes/quotes-rui.md | head -100",
  "description": "Check quotes-rui"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -A 30 \"Chapter 18\\|chapter 18\\|Ch 18\" research/commits/chapter-mapping.md | head -60",
  "description": "Find Ch 18 mapping"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "completed",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "completed",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "in_progress",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "pending",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "pending",
      "activeForm": "Writing handoff for session 019"
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

Mid-task. I've finished all reading/research for Chapter 17 and have started drafting:

**Done**
- Read session 017 README, Ch 16, and the handoff for Ch 17
- Scanned all 40 commits (158–197) in detail with full diffs
- Confirmed destination state (preprocess.c at 944 lines, main.c at 397) and restored working tree

**Now**
- Drafting `chapters/17-a-preprocessor-from-scratch.md`. Six-section plan from the handoff is intact:
  - §17.1 skeleton (5 commits)
  - §17.2 conditionals (4)
  - §17.3 object-like macros + hideset (5)
  - §17.4 function-like macros + `#`/`##` (7)
  - §17.5 polish (18, sub-grouped)
  - §17.6 self-host (1)
- Target ~15,000–20,000 words.

**Next**
- Write `docs/sessions/018-chapter-17-draft/README.md`
- Write `HANDOFF.md` for session 019 (Ch 18)

Resuming the draft now.

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "ls /Users/cell/claude/chibicc-book/chapters/ | tail -5; ls /Users/cell/claude/chibicc-book/docs/sessions/ | tail -5",
  "description": "Confirm chapter and session paths"
}


---

## 👤 User

Continue from where you left off.

---

## 🤖 Claude

No response requested.

---

## 👤 User

status

---

## 🤖 Claude

No response requested.

---

## 👤 User

status

---

## 🤖 Claude

Same status: research complete, drafting chapter 17 now. I'll write it in passes — starting with the chapter opening and §17.1.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md",
  "content": "# Chapter 17 \u2014 A preprocessor from scratch\n\n> Commits covered: `1e1ea39`, `146c7b3`, `d367510`, `ec149f6`, `d138864`, `bf6ff92`, `aa570f3`, `c6e81d2`, `e7a1857`, `97d33ad`, `9ad60e4`, `2651448`, `acce002`, `1f80f58`, `dec3b3f`, `b9ad3e4`, `dd4306c`, `c7d7ce0`, `1313fc6`, `8f6f792`, `8f561ae`, `769b5a0`, `5cb2f89`, `a8d76ad`, `8075582`, `b33fe0e`, `d85fc4f`, `a1dd621`, `a939a7a`, `e7fdc2e`, `5f5a850`, `6f17071`, `dc01f94`, `ba6b4b6`, `82ba010`, `ab4f1e1`, `7746e4e`, `7cbfd11`, `5322ea8`, `12a9e75`. Forty commits \u2014 the longest single arc in the book \u2014 building chibicc's own C preprocessor from a do-nothing pass through `#include`, `#if`, `#define`, function-like macros with stringizing and pasting, conditional-inclusion, predefined macros, and the standard headers, ending in self-hosting at the fortieth commit.\n\nThrough Chapter 16, the C source `chibicc input.c` flows through is *preprocessed C* \u2014 the result of running the host `cc -E` over the input file. The Makefile pipes test files through `cc -o- -E -P -C` before handing them to chibicc, and the stage-2 build runs `self.py`, a Python regex script, over chibicc's own source. The reason is the same in both cases: chibicc has no preprocessor of its own. It can't see `#include`. It can't see `#define`. The hash sign at column zero is, to chibicc, an unknown punctuation token, and any source file that starts with `#include <stdio.h>` is \u2014 to chibicc, raw \u2014 a parse error.\n\nChapter 17 closes that gap. By the end, chibicc handles the full preprocessor surface that ordinary C programs lean on: file inclusion (with both `\"...\"` and `<...>` forms, search paths, and recursive includes), conditional compilation (`#if`/`#ifdef`/`#ifndef`/`#elif`/`#else`/`#endif` with token-level constexpr evaluation), macros (object-like and function-like, with empty arguments, parenthesized arguments, the no-rescan rule for both kinds, stringizing `#`, pasting `##`, and variadics through `__VA_ARGS__`), the predefined macros (`__STDC__`, `__FILE__`, `__LINE__`, the `__x86_64__`/`__linux__`/`__amd64__` family), `#error`, line continuations, adjacent-string-literal concatenation, the standard headers (`stdarg.h`, `stdbool.h`, `stddef.h`, `stdalign.h`, `float.h`, `stdnoreturn.h`), and the `va_arg()` macro that finally turns Chapter 14's magic `__va_area__` into ordinary user-side code. The fortieth commit deletes `self.py`, retires the `cc -E` pipeline, and points the stage-2 Makefile target at chibicc's own source. Chibicc compiles itself.\n\nThis is a long chapter, and it has to be. Forty commits \u2014 in the chapter mapping's grouping, the largest arc by a wide margin. Section-per-commit treatment would inflate the chapter past thirty thousand words. Instead, the forty commits group naturally into six sections, each carrying a topic that the C preprocessor itself splits along.\n\n- **\u00a717.1 \u2014 The skeleton** (commits 158\u2013162). Five commits. A do-nothing `preprocess()` pass between `tokenize_file` and `parse`. The null directive (`#`-only line). `#include \"...\"`. Skipping extra tokens after `#include`. The `-E` flag.\n- **\u00a717.2 \u2014 Conditionals** (commits 163\u2013166). Four commits. `#if`/`#endif` with a token-level constexpr evaluator separate from the AST-level `eval` quartet. Skipping nested `#if` in a skipped clause. `#else`. `#elif`.\n- **\u00a717.3 \u2014 Object-like macros, the hideset, and `#ifdef`** (commits 167\u2013171). Five commits. `#define` with an object-like body. `#undef`. Macro expansion in `#if`/`#elif` arguments. The no-rescan rule, implemented per-token through a *hideset*. `#ifdef`/`#ifndef`.\n- **\u00a717.4 \u2014 Function-like macros, stringizing, and pasting** (commits 172\u2013178). Seven commits. Zero-arity, multi-arity, empty arguments, parenthesized arguments, the no-rescan rule's function-like-macro variant (using hideset *intersection* at the closing paren), stringizing `#`, pasting `##`.\n- **\u00a717.5 \u2014 Polish: the rest of the preprocessor** (commits 179\u2013196). Eighteen commits \u2014 the longest stretch. The test pipeline switches off `cc -E`. `defined()`. Identifier-to-zero in constexpr. Whitespace and newline preservation through expansion. Line continuation. `<...>` includes. `-I`. Default include paths. `#error`. Predefined macros (the `__STDC__`/`__x86_64__` block). `__FILE__`/`__LINE__` (with the `Token->origin` chain). `__VA_ARGS__`. `__func__` and `__FUNCTION__` (which land in *parse.c*, not *preprocess.c*). Adjacent-string concatenation. Wide character literal. The five standard headers. `va_arg()` as a real macro.\n- **\u00a717.6 \u2014 Self-host** (commit 197). One commit. `self.py` deleted. Stage-2 points at chibicc's own source. Chibicc compiles itself.\n\nThe `main` order across these forty commits is even more scattered by calendar date than Chapter 16. The dates jump between March 2020 and September 2020 with no apparent rhythm \u2014 Rui clearly worked on the preprocessor in bursts, sometimes laying skeleton then immediately filling it in, sometimes branching off to a different feature for weeks before returning. The chapter follows `main` order without comment and without trying to reorder.\n\nThe macro-expansion algorithm chibicc implements is *Prosser's algorithm*, sometimes called the \"blue paint\" algorithm in pedagogical writing. The C standard's wording for how macros expand was directly drawn from a 1986 technical document by Dave Prosser, and Rui's source code links to a copy of that document in the chibicc wiki. Rui's commit 170 (the no-rescan rule, \u00a717.3) is the first commit to mention it by name, and his `preprocess.c` opens with a comment block summarizing the algorithm. We'll walk that algorithm in \u00a717.3 and \u00a717.4 in detail; for now, the headline is that the algorithm is designed to *terminate*, even on `#define X X`, even on `#define f() f()`, by tracking \u2014 for each token \u2014 the set of macro names that have already been expanded into that token's neighborhood. The set is called the *hideset*. Once a macro's name is in a token's hideset, that token is invisible to that macro. Rescans walk through it as ordinary text.\n\nThe split between *what the preprocessor does* and *what cc1 does* lands cleanly: `preprocess()` is called inside `cc1()` between `tokenize_file()` and `parse()`. There's no separate `cpp` binary, no second fork-exec round-trip beyond Chapter 16's driver-cc1 split. The cc1 process tokenizes its input file, runs `preprocess()` over the resulting token list, then parses the (now-expanded) tokens. The `-E` flag short-circuits the parse step and prints the post-`preprocess` token list as text. This is the same shape modern GCC uses internally \u2014 preprocessing is a *phase* of the compiler, not a separate process \u2014 although GCC has historically supported the option of running `cpp0` as a separate stage too.\n\nForty commits, six sections. The chapter's punchline is two-line: in the fortieth commit, `chibicc compiles chibicc`, and `self.py` \u2014 the Python regex preprocessor that has stood in for chibicc's missing preprocessor since Chapter 8 \u2014 gets deleted. Chibicc is, at that point, a self-hosting compiler. The book's stated destination, since the preface, has been self-hosting; this chapter is where chibicc arrives.\n\n---\n\n## 17.1 \u2014 The skeleton\n\nFive commits. The first does literally nothing \u2014 `preprocess()` returns its input list unchanged, after running `convert_keywords` once. The next four fill in the smallest things a preprocessor can recognize: a `#`-only line, a quoted-include directive, error recovery for stray tokens after that directive, and the `-E` driver flag that lets a user see what the preprocessor produced. By the end of the section, the preprocessor's *shape* is in place: a directive recognizer that treats a hash-at-column-zero as a token marker, a token-list-rewriting model where directives splice new tokens into the stream, and a do-nothing default that passes other tokens through.\n\n### 17.1.1 \u2014 A do-nothing preprocessor\n\n> `git checkout 1e1ea39dadd0035443f1d15c651deaf979341879` \u2014 *Add a do-nothing preprocessor*\n\nNothing is preprocessed in this commit. What changes is the *shape* of the cc1 pipeline. A new file `preprocess.c` is added with a single function, `preprocess()`, that takes a token list and returns one. The body runs `convert_keywords` over the input and returns it unchanged:\n\n```c\n#include \"chibicc.h\"\n\n// Entry point function of the preprocessor.\nToken *preprocess(Token *tok) {\n  convert_keywords(tok);\n  return tok;\n}\n```\n\n`convert_keywords` was a `static` helper inside `tokenize.c` (it walked the token list and rewrote `TK_IDENT` to `TK_KEYWORD` for the language's keywords). It loses its `static` qualifier and gains a forward declaration in `chibicc.h`:\n\n```diff\n+void convert_keywords(Token *tok);\n```\n\nThe `tokenize` function stops calling it. `preprocess()` calls it instead. The reason is *ordering*: keyword conversion happens after macro expansion, not before, because a macro can expand to text containing a keyword (`#define X int` is supposed to make `X foo;` declare an `int`). With the preprocessor in place, the keyword-conversion step has to run after all macro expansion is done. In this commit the preprocessor doesn't expand anything, but the ordering is set up correctly so the next commits don't have to re-shuffle.\n\nThe `cc1()` driver function gets one new line:\n\n```diff\n static void cc1(void) {\n   // Tokenize and parse.\n   Token *tok = tokenize_file(base_file);\n+  tok = preprocess(tok);\n   Obj *prog = parse(tok);\n```\n\nThe pipeline is now `tokenize \u2192 preprocess \u2192 parse`. Both arrows are fully typed `Token *` \u2014 chibicc's preprocessor takes a token list and returns a token list, never reading from raw text and never producing raw text. The preprocessor manipulates tokens that the tokenizer already produced. This is the *single-pass tokenize-then-preprocess* model, and it's a deliberate simplification over what the C standard literally describes (the standard frames preprocessing as operating on *preprocessing tokens*, then converting them to ordinary tokens). Chibicc collapses the two: every token is an ordinary token from the moment it leaves the tokenizer.\n\nThere's a quiet downstream consequence. Pre-Chapter-17, an idiom like `0xa` for-an-`a`-suffix-on-a-number didn't matter because chibicc's tokenizer handled it. Post-Chapter-17, when *macros* can synthesize numbers like `2##0xff`, the tokenizer has to handle the same shapes consistently. Rui's commits don't explicitly call this out, but the regression net (`make test`) catches it.\n\nThe chapter's *real* preprocessor work doesn't start until the next commit. This one commits the *seam*: a function called `preprocess` that lives in `preprocess.c` and runs at the right point in the pipeline. Once the seam exists, every subsequent commit fills in a feature, and the diff stays local to `preprocess.c` (with rare exceptions when a feature genuinely needs the tokenizer's help \u2014 line continuations, the `##` punctuation, wide character literals).\n\nThis first commit is the chapter's *pre-factor entry*. Chibicc has been growing pre-factors before features for nearly the whole book \u2014 the canonicalization-at-parse-time pattern, the type-system passes, the cc1-vs-driver split. The do-nothing preprocessor is the same move, applied to the largest single feature still missing: lay the seam first, fill it in over forty commits. With this commit the *pre-factor-before-feature* count goes from seven to eight.\n\n#### Where we are\n\n`preprocess.c` exists and is wired into the cc1 pipeline. The function does nothing \u2014 its only effect is to call `convert_keywords` (which it took over from `tokenize`). The very next commit gives it work to do.\n\n---\n\n### 17.1.2 \u2014 The null directive\n\n> `git checkout 146c7b3dd47bb65da2da86cce7f4d75d8efa157d` \u2014 *Add the null directive*\n\nThe smallest piece of preprocessor syntax that exists is the *null directive* \u2014 a line containing only `#` and nothing else. It's legal C and means nothing; it's the kind of thing that shows up in machine-generated code. This commit makes the preprocessor recognize and skip it, and along the way installs the directive-recognizer that every subsequent commit reuses.\n\nThe new field on `Token` is a single `bool`:\n\n```diff\n   int line_no;    // Line number\n+  bool at_bol;    // True if this token is at beginning of line\n };\n```\n\n`at_bol` (at beginning of line) is set by the tokenizer when it sees a newline:\n\n```c\nstatic bool at_bol;\n// ...\n// Skip newline.\nif (*p == '\\n') {\n  p++;\n  at_bol = true;\n  continue;\n}\n```\n\nEvery newly-emitted token reads the flag and clears it:\n\n```c\ntok->at_bol = at_bol;\nat_bol = false;\n```\n\nSo exactly the *first* token after a newline carries `at_bol = true`. Whitespace and comments don't reset it; only a non-trivia token does.\n\n`is_hash` is the directive recognizer:\n\n```c\nstatic bool is_hash(Token *tok) {\n  return tok->at_bol && equal(tok, \"#\");\n}\n```\n\nA hash sign in the middle of a line is *not* a directive (in chibicc's lexer, `#` is just a punctuation token; it doesn't have an operator meaning yet \u2014 `##` and stringizing arrive later in the chapter). Only a hash at the beginning of a line counts.\n\nThe preprocessor's main loop is `preprocess2`, separated from the entry-point `preprocess` so it can be called recursively later:\n\n```c\nstatic Token *preprocess2(Token *tok) {\n  Token head = {};\n  Token *cur = &head;\n\n  while (tok->kind != TK_EOF) {\n    // Pass through if it is not a \"#\".\n    if (!is_hash(tok)) {\n      cur = cur->next = tok;\n      tok = tok->next;\n      continue;\n    }\n\n    tok = tok->next;\n\n    // `#`-only line is legal. It's called a null directive.\n    if (tok->at_bol)\n      continue;\n\n    error_tok(tok, \"invalid preprocessor directive\");\n  }\n\n  cur->next = tok;\n  return head.next;\n}\n```\n\nThe structure is the *chained-list rewriter* pattern: walk the input, splice non-directive tokens onto the output (`cur = cur->next = tok`), and consume directive tokens specially. Tokens flow through the loop one at a time. When a directive is recognized, it's consumed without producing output (the null directive case here); when it's not a directive, it's spliced through unchanged.\n\nThe two key reads are `is_hash` (do we have a directive?) and `tok->at_bol` (have we hit the next line yet?). The loop after `tok = tok->next` reads the *first token after the hash*. If that token is itself at beginning of line \u2014 meaning the hash had nothing after it \u2014 we have a null directive, and we skip it by `continue`. Otherwise (in this commit) we error out, because no actual directive exists yet.\n\nThe test file `test/macro.c` is created in this commit (rather than reusing an existing test file), and lands with two null-directive lines:\n\n```c\n#\n\n/* */ #\n```\n\nThe second case verifies that a comment between newline-and-hash doesn't break recognition. The tokenizer eats the comment (`has_space` doesn't exist yet; comments simply don't produce a token), and `at_bol` carries through to the `#`.\n\n#### Where we are\n\nThe directive recognizer is in place. The preprocessor consumes one well-formed directive (the null directive) and rejects everything else. The next commit gives it a real directive to handle.\n\n---\n\n### 17.1.3 \u2014 `#include \"...\"`\n\n> `git checkout d367510fcc1396fa252c4b87439c2f9fcd0abbe7` \u2014 *Add `#include \"...\"`*\n\n`#include` is the preprocessor's most user-visible feature \u2014 for many C programmers, *the* preprocessor feature. It's also where the token-list-rewriting model earns its keep: the preprocessor reads a filename token, calls `tokenize_file` on the named file to produce a fresh token list, and *splices that list* into the position where the directive was. The body of the included file flows through `preprocess2` in the next pass over the spliced stream, recursively, with no special handling.\n\nThe shape of the splice is a helper called `append`:\n\n```c\n// Append tok2 to the end of tok1.\nstatic Token *append(Token *tok1, Token *tok2) {\n  if (!tok1 || tok1->kind == TK_EOF)\n    return tok2;\n\n  Token head = {};\n  Token *cur = &head;\n\n  for (; tok1 && tok1->kind != TK_EOF; tok1 = tok1->next)\n    cur = cur->next = copy_token(tok1);\n  cur->next = tok2;\n  return head.next;\n}\n```\n\n`append(tok1, tok2)` produces a new list that's the concatenation of `tok1` (truncated at its EOF) and `tok2`. The truncation matters because each `tokenize_file` call returns a list ending in its own EOF token; `append` strips that EOF and stitches the next list onto its tail. `copy_token` is a shallow-copy helper:\n\n```c\nstatic Token *copy_token(Token *tok) {\n  Token *t = calloc(1, sizeof(Token));\n  *t = *tok;\n  t->next = NULL;\n  return t;\n}\n```\n\nThe directive case in `preprocess2` is short:\n\n```c\nif (equal(tok, \"include\")) {\n  tok = tok->next;\n\n  if (tok->kind != TK_STR)\n    error_tok(tok, \"expected a filename\");\n\n  char *path = format(\"%s/%s\", dirname(strdup(tok->file->name)), tok->str);\n  Token *tok2 = tokenize_file(path);\n  if (!tok2)\n    error_tok(tok, \"%s\", strerror(errno));\n  tok = append(tok2, tok->next);\n  continue;\n}\n```\n\nThe argument has to be a string literal (the `\"...\"` form; the `<...>` form arrives in \u00a717.5). The path is computed relative to *the directory of the file the directive appears in* \u2014 `dirname(tok->file->name)` \u2014 joined with the filename. That's the C standard's specified search behavior for the double-quoted form. (The standard also says compilers may search additional directories; chibicc's `-I` and default-paths work arrives in \u00a717.5.)\n\nThe included file is tokenized into `tok2`. `append(tok2, tok->next)` produces a list with `tok2` at the front and the *rest of the current input* (the tokens after `#include \"foo.h\"`) at the back. The directive is consumed; the included file's tokens take its place in the stream. The `continue` re-enters the `while` loop, which now reads from the spliced list \u2014 meaning the included file is preprocessed by the same `preprocess2` machinery, recursively, including any `#include` directives *it* contains (which is what makes `include1.h` \u2192 `include2.h` work, the test exercises both directions).\n\nA subtlety: the path is computed using `tok->file->name`. That requires `Token` to know which file it was tokenized from. The `File` struct is new in this commit:\n\n```c\ntypedef struct {\n  char *name;\n  int file_no;\n  char *contents;\n} File;\n```\n\n\u2014 and `Token` gains a `File *file` field that the tokenizer populates. The `file_no` field is an integer the assembler's `.file` directive uses (the codegen change in this commit emits `  .file 1 \"foo.c\"`/`.file 2 \"foo.h\"` headers based on it; before this commit the codegen hardcoded `.file 1`).\n\nThe `tokenize_file` function gains a small bit of bookkeeping that preserves all input files' metadata across translation-unit boundaries:\n\n```c\nstatic File **input_files;\n```\n\nEvery file that's tokenized is appended to this list. `get_input_files()` returns it for the codegen's `.file` emission step. The numbering matters for GDB: when the debugger sees a `.loc N L` instruction, `N` is the file number, and `.file N \"path\"` at the top of the assembly file teaches the debugger how to map. After this commit, errors inside an included file produce the right filename, and stepping through stops at the right file.\n\nThe error machinery also adapts: pre-this-commit, `error_at` and `error_tok` both reached for a static `current_input` pointer in `tokenize.c`. Post-this-commit, those helpers take a `filename` and `input` argument explicitly, and the variants pull from `tok->file->name` and `tok->file->contents`. Errors inside `include1.h` \u2014 the test exercises this \u2014 print `test/include1.h:N: ...`, not `test/macro.c:N: ...`.\n\n`read_file` also softens its error handling: where it once errored out on `fopen` failure, it now returns `NULL`, and `tokenize_file` (and its callers) decide what to do:\n\n```diff\n-      error(\"cannot open %s: %s\", path, strerror(errno));\n+      return NULL;\n```\n\n\u2014 so `#include` can produce a *located* error (using `error_tok` to point at the directive) rather than a no-location bare `error`.\n\nThe included-file tests are minimal:\n\n```c\n// test/include1.h\n#include \"include2.h\"\nint include1 = 5;\n\n// test/include2.h\nint include2 = 7;\n\n// test/macro.c\n#include \"include1.h\"\n// ...\nassert(5, include1, \"include1\");\nassert(7, include2, \"include2\");\n```\n\n`include1.h` includes `include2.h`. The recursion is implicit: `include1.h`'s tokens flow through `preprocess2`, which sees its `#include` directive, splices `include2.h`'s tokens in, continues the loop. Two levels deep. The Chapter 17 test pipeline never hits any deep stress test of recursion in a single file (chibicc doesn't have re-include guards yet \u2014 those are `#pragma once` and the include-guard optimization, which arrive in Chapter 22), but `include1.h`/`include2.h` is the basic shape.\n\n#### Where we are\n\n`#include \"...\"` works for relative paths (computed from the including file's directory) and absolute paths (the `tok->str[0] == '/'` branch arrives in \u00a717.1.5 with the `-E` commit, but the directive itself works). Each input file gets a `File *` with a unique `file_no`. The codegen emits `.file` directives for all input files, and `.loc` directives reference the right file number, so debug-info-aware tools (gdb, addr2line) can map lines back to the right file. Errors carry the correct filename through include levels.\n\n---\n\n### 17.1.4 \u2014 Skip extra tokens after `#include`\n\n> `git checkout ec149f64d2f5c41a2080c0b4e42e4ef64444b382` \u2014 *Skip extra tokens after `#include \"...\"`*\n\nA short cleanup commit. The `#include \"foo.h\"` directive accepts the filename token and then *should* see end-of-line. Real-world C sometimes puts trailing tokens on directive lines \u2014 most often by accident, occasionally by `#include \"foo.h\" /* comment */` style; the standard says it's a constraint violation, but compilers in practice issue a warning and skip them. Chibicc opts for the skip-with-warning behavior:\n\n```c\n// Some preprocessor directives such as #include allow extraneous\n// tokens before newline. This function skips such tokens.\nstatic Token *skip_line(Token *tok) {\n  if (tok->at_bol)\n    return tok;\n  warn_tok(tok, \"extra token\");\n  while (tok->at_bol)\n    tok = tok->next;\n  return tok;\n}\n```\n\nThe first line returns immediately if we're already at the start of the next line. The second issues a warning. The loop walks forward until the next at-beginning-of-line token. Note the loop's condition: `while (tok->at_bol)` *would* spin forever on a token at-beginning-of-line \u2014 but the function only enters the loop having already passed the early-return, so `tok` is not at-bol when the loop starts. The first iteration moves to `tok->next`, and the loop re-checks `tok->at_bol`. Reading carefully: this is `while (!tok->at_bol)`, except spelled inverted because the loop *starts* on a known-not-at-bol token. The body increments the pointer and re-checks. It's a small piece of code that took the author at least a moment to read correctly; the comment helps.\n\n`warn_tok` is also new in this commit, a sibling of `error_tok` that prints the same `filename:line: message` format to stderr but doesn't `exit`:\n\n```c\nvoid warn_tok(Token *tok, char *fmt, ...) {\n  va_list ap;\n  va_start(ap, fmt);\n  verror_at(tok->file->name, tok->file->contents, tok->line_no, tok->loc, fmt, ap);\n  va_end(ap);\n}\n```\n\nThe `#include` directive is updated to call `skip_line` after consuming the filename:\n\n```diff\n-      tok = append(tok2, tok->next);\n+      tok = skip_line(tok->next);\n+      tok = append(tok2, tok);\n```\n\n\u2014 the `tok->next` after the filename is the start of the *trailing* tokens (or the newline), `skip_line` advances past them with a warning if any, and the result is what gets stitched after the included file's body.\n\nThis commit also adds the `warn_tok` declaration to `chibicc.h` and is otherwise tiny. It's useful as a preview of the chapter's larger pattern: `skip_line` is going to be reused by every directive. Object-like `#define`, function-like `#define`, `#undef`, `#if`, `#elif`, `#else`, `#endif`, `#ifdef`, `#ifndef`, `#error`, `#include` (both forms) all end with `skip_line` to consume trailing junk.\n\n#### Where we are\n\n`#include \"foo.h\"` extra tokens emit a warning and are skipped. The `skip_line` helper is the basic discipline that every later directive uses to clean up its line.\n\n---\n\n### 17.1.5 \u2014 `-E`\n\n> `git checkout d138864a2a99849e43d81ca071b7a799edc0e65a` \u2014 *Add `-E` option*\n\n`-E` is the GCC-style flag that runs only the preprocessor stage and prints the result to stdout (or the `-o` path). With `-E`, the tokens aren't parsed; they're rendered back to text and the process exits. This commit adds `-E` to chibicc's argument parser, adds a `print_tokens` function that renders the post-`preprocess` token list, and adjusts the driver to dispatch `-E` to a cc1 invocation that produces preprocessed output instead of assembly.\n\nThe driver-level change is a new `opt_E` flag and a per-input dispatch:\n\n```c\n// Just preprocess\nif (opt_E) {\n  run_cc1(argc, argv, input, NULL);\n  continue;\n}\n```\n\nThe cc1 callee handles `-E` by branching after `preprocess()`:\n\n```c\nstatic void cc1(void) {\n  // Tokenize and parse.\n  Token *tok = tokenize_file(base_file);\n  if (!tok)\n    error(\"%s: %s\", base_file, strerror(errno));\n\n  tok = preprocess(tok);\n\n  // If -E is given, print out preprocessed C code as a result.\n  if (opt_E) {\n    print_tokens(tok);\n    return;\n  }\n\n  Obj *prog = parse(tok);\n  // ...\n}\n```\n\n`print_tokens` walks the token list and emits each token's text, separating them with spaces and newlines based on `at_bol`:\n\n```c\nstatic void print_tokens(Token *tok) {\n  FILE *out = open_file(opt_o ? opt_o : \"-\");\n\n  int line = 1;\n  for (; tok->kind != TK_EOF; tok = tok->next) {\n    if (line > 1 && tok->at_bol)\n      fprintf(out, \"\\n\");\n    fprintf(out, \" %.*s\", tok->len, tok->loc);\n    line++;\n  }\n  fprintf(out, \"\\n\");\n}\n```\n\nEach token's *text* is its slice of the original file's contents \u2014 `tok->loc` and `tok->len` point into the source. For tokens that came from `#include`d files, `tok->loc` points into *that* file's contents, which is what we want: the printed form of an included file's contents reproduces the included file's text.\n\nThe `at_bol` flag drives the newlines: every time we hit a token marked at-beginning-of-line (after the first token), we print a `\\n` first, so the output preserves the source's vertical structure. Adjacent tokens in the same line get separated by a single leading space (the `\" %.*s\"` format string). Whitespace *within* a line isn't preserved \u2014 the next commit (in \u00a717.4) will add a `has_space` flag to fix that \u2014 but the line structure is preserved.\n\nThere's a subtle bug-or-feature in this output format. The `\" %.*s\"` always prints a leading space. For the first token of a line, that's an indent. For the very first token of the whole file, it's an indent that probably shouldn't be there. The `-E` output is intended to be human-inspectable rather than re-feedable to the compiler, so the leading space doesn't break anything \u2014 and the stage-2 build doesn't use `-E` (it'll feed `chibicc` directly without `-E` interposition once the preprocessor handles `#include` itself). It's fine.\n\nThe driver-side validation grows: `-o` with multiple inputs is now an error if any of `-c`, `-S`, `-E` is set:\n\n```diff\n-  if (input_paths.len > 1 && opt_o && (opt_c || opt_S))\n-    error(\"cannot specify '-o' with '-c' or '-S' with multiple files\");\n+  if (input_paths.len > 1 && opt_o && (opt_c || opt_S | opt_E))\n+    error(\"cannot specify '-o' with '-c,' '-S' or '-E' with multiple files\");\n```\n\n(The bitwise `|` between `opt_S` and `opt_E` is a typo \u2014 should be `||`. It compiles, and the values are 0/1 so `|` happens to do the same thing as `||`, but it's the kind of thing a reviewer would flag. Errata candidate. We'll catalog with the rest of Chapter 17's small slips at the chapter's close.)\n\nA second small adjustment in this commit handles absolute include paths:\n\n```c\nchar *path;\nif (tok->str[0] == '/')\n  path = tok->str;\nelse\n  path = format(\"%s/%s\", dirname(strdup(tok->file->name)), tok->str);\n```\n\n\u2014 the `#include \"/path/to/file\"` form bypasses the directory-relative resolution. (This is non-standard for the double-quoted form; the standard says implementation-defined search applies. Chibicc accepts the absolute path verbatim.)\n\nThe driver tests for `-E`:\n\n```bash\necho foo > $tmp/out\necho \"#include \\\"$tmp/out\\\"\" | $chibicc -E - | grep -q foo\ncheck -E\n\necho foo > $tmp/out1\necho \"#include \\\"$tmp/out1\\\"\" | $chibicc -E -o $tmp/out2 -\ncat $tmp/out2 | grep -q foo\ncheck '-E and -o'\n```\n\n\u2014 a `#include`-via-stdin smoke test, with both stdout-output and `-o` redirection.\n\n#### Where we are\n\nThe preprocessor's user-visible surface starts to take shape. `-E` lets users (and the test pipeline, eventually) see what the preprocessor produced. The `print_tokens` formatter is intentionally lossy \u2014 it doesn't preserve whitespace, doesn't preserve all separators perfectly \u2014 but it produces output that's recognizable as the input minus the `#include` lines and with their bodies inlined.\n\n#### Where the section leaves us\n\nFive commits, and chibicc has the *shape* of a preprocessor:\n\n- `preprocess()` is wired between `tokenize_file()` and `parse()`. Tokens flow through, get rewritten or passed through, and feed the parser.\n- The `at_bol` flag on `Token` distinguishes column-zero hashes from in-line hashes.\n- The directive recognizer (`is_hash`) and the line-skipper (`skip_line`) are the building blocks every later directive uses.\n- `#include \"...\"` works recursively, with proper file numbering for `.loc` debug info.\n- `-E` runs the preprocessor and prints the result.\n\nThe preprocessor knows two directives so far: the null directive and `#include`. The next section adds four more: `#if`, `#endif`, `#else`, `#elif`. With those, conditional compilation works.\n\n---\n\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md",
  "description": "Check word count after \u00a717.1"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cat >> /Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md << 'CHAPTERMARKER'\n## 17.2 \u2014 Conditionals\n\nFour commits. Conditional compilation is the C preprocessor's other core feature, alongside macros \u2014 `#if expression`/`#endif`, with `#else` and `#elif` for branches. Unlike `#include`, `#if` doesn't *splice* tokens into the stream; it *deletes* whole regions of source. The preprocessor reads `#if`'s argument as a constant expression, evaluates it, and either preserves or skips the body until the matching `#endif`.\n\nThe interesting piece is the constexpr evaluator. `#if` runs at preprocess time, before the AST exists, so it can't use the parser's `eval`/`eval2`/`eval_double` quartet \u2014 those operate on AST nodes. Instead, the preprocessor builds a *temporary* token list from the directive's argument, hands it to `const_expr` (which is now exposed from `parse.c`), gets back a `long`, and uses that as the truth value. The token-level evaluator is, in effect, a small parser-and-evaluator running in front of the main parser.\n\n### 17.2.1 \u2014 `#if` and `#endif`\n\n> `git checkout bf6ff928ad17d98d07f68f619e6cbe29829d0a20` \u2014 *Add `#if` and `#endif`*\n\nThe constexpr evaluator first. `parse.c`'s `const_expr` is promoted from `static` to externally visible, with a new declaration in `chibicc.h`:\n\n```c\nint64_t const_expr(Token **rest, Token *tok);\n```\n\nThe function itself is unchanged: it parses a conditional expression and runs `eval` on the resulting AST node. What's new is that it can be called from `preprocess.c`. The preprocessor builds a synthetic token list that contains just the directive's argument, hands it to `const_expr`, and uses the returned `int64_t`.\n\nThe synthetic-list machinery is `copy_line` and `new_eof`:\n\n```c\nstatic Token *new_eof(Token *tok) {\n  Token *t = copy_token(tok);\n  t->kind = TK_EOF;\n  t->len = 0;\n  return t;\n}\n\nstatic Token *copy_line(Token **rest, Token *tok) {\n  Token head = {};\n  Token *cur = &head;\n\n  for (; !tok->at_bol; tok = tok->next)\n    cur = cur->next = copy_token(tok);\n\n  cur->next = new_eof(tok);\n  *rest = tok;\n  return head.next;\n}\n```\n\n`copy_line` reads tokens until the next at-bol (the next line's first token), copies each one into a fresh list, and terminates that list with a synthetic EOF. The `*rest` output is set to the at-bol token \u2014 i.e., the *next* line's first token, which is where the preprocessor will resume after the directive. `new_eof` makes a copy of an existing token and changes its kind to `TK_EOF`; the copy is used so the synthetic EOF carries a real source location for error reporting.\n\n`eval_const_expr` wraps `const_expr` in this synthetic-list discipline:\n\n```c\nstatic long eval_const_expr(Token **rest, Token *tok) {\n  Token *start = tok;\n  Token *expr = copy_line(rest, tok->next);\n\n  if (expr->kind == TK_EOF)\n    error_tok(start, \"no expression\");\n\n  Token *rest2;\n  long val = const_expr(&rest2, expr);\n  if (rest2->kind != TK_EOF)\n    error_tok(rest2, \"extra token\");\n  return val;\n}\n```\n\nThe argument-line tokens are copied (with `tok->next` because `tok` itself is the `if` keyword), passed to `const_expr`, and the result is checked for \"no leftover tokens.\" If the expression doesn't consume the whole synthetic line, that's an error: `#if 1 +` (with no second operand) lands here.\n\nNote the duplication. Chibicc has, post this commit, *two* constant-expression evaluation paths:\n\n- `parse.c`'s `eval`/`eval2`/`eval_rval`/`eval_double` quartet, which runs over AST nodes and is used by initializers, array sizes, case labels, and bitfield widths.\n- `preprocess.c`'s `eval_const_expr`, which calls `const_expr` (which calls `eval`) but operates on a synthesized token list and runs *before* the main parse.\n\nThe two share `eval` itself \u2014 the evaluator at the AST-node level \u2014 but go through different parsers (the preprocessor builds the AST through `const_expr` \u2192 `conditional`, which is the same recursive-descent stack as the main parser's). The duplication is mostly *temporary* token-list machinery; the *evaluation* is unified through `eval`.\n\nThe conditional state is held on a stack:\n\n```c\ntypedef struct CondIncl CondIncl;\nstruct CondIncl {\n  CondIncl *next;\n  Token *tok;\n};\n\nstatic CondIncl *cond_incl;\n```\n\n`cond_incl` is a singly-linked list (used as a stack) that grows on `#if`/`#ifdef`/`#ifndef` and shrinks on `#endif`. The `tok` field stores the directive's `#` token, which is what `error_tok` reports if the conditional is unterminated.\n\nThe `if` and `endif` directive cases:\n\n```c\nif (equal(tok, \"if\")) {\n  long val = eval_const_expr(&tok, tok);\n  push_cond_incl(start, val);\n  if (!val)\n    tok = skip_cond_incl(tok);\n  continue;\n}\n\nif (equal(tok, \"endif\")) {\n  if (!cond_incl)\n    error_tok(start, \"stray #endif\");\n  cond_incl = cond_incl->next;\n  tok = skip_line(tok->next);\n  continue;\n}\n```\n\n`#if` evaluates the constant, pushes a new entry on the stack (the `included` field arrives in \u00a717.2.3), and \u2014 if the value is zero \u2014 calls `skip_cond_incl` to fast-forward past the conditional body to the matching `#endif`. `#endif` pops the stack.\n\n`skip_cond_incl` is the simplest version of itself:\n\n```c\n// Skip until next `#endif`.\nstatic Token *skip_cond_incl(Token *tok) {\n  while (tok->kind != TK_EOF) {\n    if (is_hash(tok) && equal(tok->next, \"endif\"))\n      return tok;\n    tok = tok->next;\n  }\n  return tok;\n}\n```\n\nIt scans forward looking for `#endif`. The next commit adds nesting; this one doesn't handle nested `#if` correctly. The test deliberately constructs an `#if 0`-skipping case where the body contains `#include \"/no/such/file\"` \u2014 the test passes because the include directive is *inside* the skipped region and never gets executed.\n\nA small fix in this commit: a one-character correction to `tokenize.c`'s error-line printer that addresses an EOF-without-newline case introduced by `read_file`'s changed handling:\n\n```diff\n-  while (*end != '\\n')\n+  while (*end && *end != '\\n')\n```\n\n\u2014 the loop now stops at the file's terminator, not just at a newline. This is what makes `#if 1\\n#endif\\n` (without trailing data) work cleanly through `error_at`'s line-printer when an error happens to fall on the last line.\n\nThe terminating check at the end of `preprocess()` is added:\n\n```c\nToken *preprocess(Token *tok) {\n  tok = preprocess2(tok);\n  if (cond_incl)\n    error_tok(cond_incl->tok, \"unterminated conditional directive\");\n  convert_keywords(tok);\n  return tok;\n}\n```\n\n\u2014 a missing `#endif` at end-of-file gets caught here.\n\n### 17.2.2 \u2014 Skip nested `#if` in a skipped clause\n\n> `git checkout aa570f3086ce3e2c5ac8bf6107c051fed5aabf89` \u2014 *Skip nested `#if` in a skipped `#if`-clause*\n\nA six-line correction. The previous commit's `skip_cond_incl` walks forward to the *first* `#endif` it finds, regardless of nesting. That breaks on:\n\n```c\n#if 0\n#if nested\n#endif\n#endif\n```\n\n\u2014 the `#endif` matching `#if nested` is found first, leaving the outer `#if 0` open. The fix:\n\n```c\nstatic Token *skip_cond_incl(Token *tok) {\n  while (tok->kind != TK_EOF) {\n    if (is_hash(tok) && equal(tok->next, \"if\")) {\n      tok = skip_cond_incl(tok->next->next);\n      tok = tok->next;\n      continue;\n    }\n    if (is_hash(tok) && equal(tok->next, \"endif\"))\n      break;\n    tok = tok->next;\n  }\n  return tok;\n}\n```\n\n\u2014 recursive: when `#if` is found inside the skipped region, recurse to find *its* matching `#endif`, then continue the outer scan. The recursion's depth bounds the deepest `#if` nesting; chibicc doesn't enforce a limit, but the C standard's minimum-implementation requirement is fifteen levels of conditional inclusion, which a recursive scan handles trivially.\n\n### 17.2.3 \u2014 `#else`\n\n> `git checkout c6e81d22f8189cd7bfcfcc33e4ac462529418192` \u2014 *Add `#else`*\n\n`#else` is a substantial commit \u2014 forty lines in `preprocess.c`. The structure decision: when `#if 1`'s body is *included*, the matching `#else`'s body must be *excluded*, and vice versa. The conditional state needs to carry which case applies.\n\n`CondIncl` grows two fields:\n\n```c\nstruct CondIncl {\n  CondIncl *next;\n  enum { IN_THEN, IN_ELSE } ctx;\n  Token *tok;\n  bool included;\n};\n```\n\n`included` records whether the `#if`/`#elif` body was kept (i.e., the constant was true). `ctx` tracks which clause we're currently in (`IN_THEN` for the body of `#if`, `IN_ELSE` for the body of `#else`). The `ctx` field guards against duplicate `#else` (`#else ... #else` is a stray-else error).\n\n`push_cond_incl` updates to take the included-flag:\n\n```c\nstatic CondIncl *push_cond_incl(Token *tok, bool included) {\n  CondIncl *ci = calloc(1, sizeof(CondIncl));\n  ci->next = cond_incl;\n  ci->ctx = IN_THEN;\n  ci->tok = tok;\n  ci->included = included;\n  cond_incl = ci;\n  return ci;\n}\n```\n\n\u2014 and `#if` calls `push_cond_incl(start, val)`, recording whether the value was non-zero.\n\nThe skipper splits in two. `skip_cond_incl2` is the inner (nested-only) scanner used during nested skip; it runs to `#endif` only and ignores `#else`/`#elif`. `skip_cond_incl` (renamed to mean *outer* skip) runs to `#else`, `#elif`, or `#endif` \u2014 whichever comes first at the current nesting level:\n\n```c\nstatic Token *skip_cond_incl2(Token *tok) {\n  while (tok->kind != TK_EOF) {\n    if (is_hash(tok) && equal(tok->next, \"if\")) {\n      tok = skip_cond_incl2(tok->next->next);\n      continue;\n    }\n    if (is_hash(tok) && equal(tok->next, \"endif\"))\n      return tok->next->next;\n    tok = tok->next;\n  }\n  return tok;\n}\n\nstatic Token *skip_cond_incl(Token *tok) {\n  while (tok->kind != TK_EOF) {\n    if (is_hash(tok) && equal(tok->next, \"if\")) {\n      tok = skip_cond_incl2(tok->next->next);\n      continue;\n    }\n    if (is_hash(tok) &&\n        (equal(tok->next, \"else\") || equal(tok->next, \"endif\")))\n      break;\n    tok = tok->next;\n  }\n  return tok;\n}\n```\n\nThe two-function split is necessary because at the *outermost* level we want to *stop* at `#else`/`#elif` to give the conditional machinery a chance to evaluate them, but at *inner* nested levels we want to *skip past* `#else`/`#elif` because they belong to the inner conditional.\n\nThe `else` directive case:\n\n```c\nif (equal(tok, \"else\")) {\n  if (!cond_incl || cond_incl->ctx == IN_ELSE)\n    error_tok(start, \"stray #else\");\n  cond_incl->ctx = IN_ELSE;\n  tok = skip_line(tok->next);\n\n  if (cond_incl->included)\n    tok = skip_cond_incl(tok);\n  continue;\n}\n```\n\n\u2014 if the `#if` clause was included, we're now in the `#else` clause and need to *skip* it (and the skip will land at the matching `#endif`). If the `#if` clause was excluded, we'd already arrived at the `#else` via the previous `skip_cond_incl`, and now we just continue as if `#else`'s body is the current included region.\n\nThe state machine: `included = false, ctx = IN_THEN` means we're currently in a `#else` body (we got skipped here because `#if 0`); `included = true, ctx = IN_THEN` means we're currently in a `#if` body, and any `#else` we encounter should be skipped; etc. The `included` flag plus the `ctx` flag together specify \"what to do at the next `#else`/`#elif`.\"\n\nThe test exercises the four combinations and a deeply-nested case:\n\n```c\n#if 1\n# if 0\n#  if 1\n    foo bar\n#  endif\n# endif\n      m = 3;\n#endif\n    assert(3, m, \"m\");\n```\n\n\u2014 the `foo bar` line is in a doubly-skipped region; if the nested-skipper miscounted, the assignment `m = 3` would be skipped too.\n\n### 17.2.4 \u2014 `#elif`\n\n> `git checkout e7a1857a31fc0c0012773c021639a6297f5b208f` \u2014 *Add `#elif`*\n\n`#elif` is `#else if` collapsed into one directive. The semantics: when `#if`'s body was excluded (the constant was false), the *first* `#elif` whose constant evaluates true takes over; subsequent `#elif`s and the final `#else` are skipped. The `included` flag tracks whether *any* clause has fired yet.\n\n`CondIncl::ctx` gains a third state, `IN_ELIF`:\n\n```c\nenum { IN_THEN, IN_ELIF, IN_ELSE } ctx;\n```\n\nThe skipper functions update to recognize `#elif`:\n\n```c\nif (is_hash(tok) &&\n    (equal(tok->next, \"elif\") || equal(tok->next, \"else\") ||\n     equal(tok->next, \"endif\")))\n  break;\n```\n\nThe `elif` directive case:\n\n```c\nif (equal(tok, \"elif\")) {\n  if (!cond_incl || cond_incl->ctx == IN_ELSE)\n    error_tok(start, \"stray #elif\");\n  cond_incl->ctx = IN_ELIF;\n\n  if (!cond_incl->included && eval_const_expr(&tok, tok))\n    cond_incl->included = true;\n  else\n    tok = skip_cond_incl(tok);\n  continue;\n}\n```\n\n\u2014 if no clause has fired yet (`!cond_incl->included`) and *this* clause's expression is true, fire this clause (set `included = true`, fall through to process the body normally). Otherwise skip to the next `#elif`/`#else`/`#endif`.\n\nReading the test:\n\n```c\n#if 0\n  m = 1;\n#elif 0\n  m = 2;\n#elif 3+5\n  m = 3;\n#elif 1*5\n  m = 4;\n#endif\n  assert(3, m, \"m\");\n```\n\n\u2014 `#if 0` is excluded, `#elif 0` is excluded, `#elif 3+5` (= 8, true) fires and sets `m = 3`. The fourth `#elif 1*5` (also true) is skipped because `included` is now true.\n\n`#elif` and `#else` interact correctly because `#else`'s test (`if (!cond_incl->included)` is implicit in the order \u2014 `#else` only fires if no prior `#if`/`#elif` set `included = true`), and the `IN_ELSE` ctx prevents a `#elif` from following an `#else`.\n\n#### Where the section leaves us\n\nFour conditional directives (`#if`/`#elif`/`#else`/`#endif`) plus their `#ifdef`/`#ifndef` cousins (which arrive in \u00a717.3). The constexpr evaluator runs on synthetic token lists, calling into `parse.c`'s `const_expr`, with no AST. Stack-based state lets conditionals nest.\n\nThe chapter's *eval-quartet duplication* note: chibicc has, after \u00a717.2, two constant-expression entry points \u2014 the AST-level one (used by initializers, array sizes, case labels, bitfield widths) and the token-level one (used by `#if`/`#elif`). They share `eval` itself but go through different parsers. This is the kind of duplication Rui's design notes (Chapter 16's `quotes-rui.md`: \"It's better to allow small duplications instead\") explicitly endorses; the alternative would be a unified token-or-AST evaluator that's harder to read.\n\n---\n\n## 17.3 \u2014 Object-like macros, the hideset, and `#ifdef`\n\nFive commits. `#define X 5` and `X` becomes `5` everywhere, with the wrinkle that `#define X X + 1` should expand `X` to `X + 1` once and stop, rather than infinitely. The infinite-recursion guard is the hideset machinery \u2014 a *set of macro names* attached to each token, recording which macros have already had a shot at expanding into this token's neighborhood. Once a macro's name is in a token's hideset, that token is invisible to that macro.\n\n### 17.3.1 \u2014 Object-like `#define`\n\n> `git checkout 97d33ad3bdc21c26356253046902d4b166bd115b` \u2014 *Add objlike `#define`*\n\nMacros are stored as a singly-linked list:\n\n```c\ntypedef struct Macro Macro;\nstruct Macro {\n  Macro *next;\n  char *name;\n  Token *body;\n};\n\nstatic Macro *macros;\n```\n\nThe body is a token list, terminated by EOF. `add_macro` cons-prepends a new entry:\n\n```c\nstatic Macro *add_macro(char *name, Token *body) {\n  Macro *m = calloc(1, sizeof(Macro));\n  m->next = macros;\n  m->name = name;\n  m->body = body;\n  macros = m;\n  return m;\n}\n```\n\n\u2014 and because lookups walk the list in order, a redefinition (a second `#define` of the same name) shadows the first. `find_macro` is the linear-scan lookup keyed on `Token`'s text:\n\n```c\nstatic Macro *find_macro(Token *tok) {\n  if (tok->kind != TK_IDENT)\n    return NULL;\n\n  for (Macro *m = macros; m; m = m->next)\n    if (strlen(m->name) == tok->len && !strncmp(m->name, tok->loc, tok->len))\n      return m;\n  return NULL;\n}\n```\n\n`expand_macro` is the simplest possible expander:\n\n```c\nstatic bool expand_macro(Token **rest, Token *tok) {\n  Macro *m = find_macro(tok);\n  if (!m)\n    return false;\n  *rest = append(m->body, tok->next);\n  return true;\n}\n```\n\n\u2014 append the macro's body to the rest of the input, return true (signaling the caller to re-run the loop on the spliced result). The body's tokens become part of the input stream. The next iteration through `preprocess2`'s loop reads the body's first token and processes it normally (passing it through, recognizing it as a directive, *or* expanding it as another macro if it matches).\n\nThe directive case in `preprocess2`:\n\n```c\nif (equal(tok, \"define\")) {\n  tok = tok->next;\n  if (tok->kind != TK_IDENT)\n    error_tok(tok, \"macro name must be an identifier\");\n  char *name = strndup(tok->loc, tok->len);\n  add_macro(name, copy_line(&tok, tok->next));\n  continue;\n}\n```\n\nThe body is `copy_line(&tok, tok->next)` \u2014 the rest of the line, copied (so the macro body has its own token storage, decoupled from the source's), terminated with the synthetic EOF.\n\nThe expand check is added to the top of `preprocess2`:\n\n```c\nwhile (tok->kind != TK_EOF) {\n  // If it is a macro, expand it.\n  if (expand_macro(&tok, tok))\n    continue;\n  // ... rest of the loop ...\n}\n```\n\nNow ordinary identifiers get checked against the macro table before they're treated as identifiers. If they match, the body splices in and the loop re-enters.\n\nThe infinite-recursion problem doesn't surface yet because the test cases this commit ships with don't construct a self-referential macro. Tests like `#define M1 3` followed by `M1` work fine: the body is `3`, which isn't a macro, so the next loop iteration passes it through.\n\nBut consider `#define M2 M2 + 3`. The body is `M2 + 3`; the first token of the body is `M2` itself; the next loop iteration sees `M2`, looks it up in the macro table, finds itself, and splices `M2 + 3` again. Infinite expansion. Three commits below, the no-rescan rule fixes this.\n\n### 17.3.2 \u2014 `#undef`\n\n> `git checkout 9ad60e41d512158d942d1bf3808682ede6ef5118` \u2014 *Add `#undef`*\n\n`#undef` removes a macro definition. Because chibicc's macro table is a stack-of-redefinitions where the most recent wins, `#undef` works by adding a *deleted* entry that shadows the (possibly multiple) earlier entries:\n\n```c\nstruct Macro {\n  Macro *next;\n  char *name;\n  Token *body;\n  bool deleted;\n};\n```\n\n`find_macro` skips deleted entries:\n\n```c\nfor (Macro *m = macros; m; m = m->next)\n  if (strlen(m->name) == tok->len && !strncmp(m->name, tok->loc, tok->len))\n    return m->deleted ? NULL : m;\n```\n\n\u2014 the lookup *finds* the deleted entry (it's the most-recently-pushed, so it's seen first), and returns NULL because of the `deleted` flag. The earlier (non-deleted) entries are never reached.\n\nThe directive case:\n\n```c\nif (equal(tok, \"undef\")) {\n  tok = tok->next;\n  if (tok->kind != TK_IDENT)\n    error_tok(tok, \"macro name must be an identifier\");\n  char *name = strndup(tok->loc, tok->len);\n  tok = skip_line(tok->next);\n\n  Macro *m = add_macro(name, NULL);\n  m->deleted = true;\n  continue;\n}\n```\n\n`#undef`-of-an-undefined-macro is a no-op (the deleted entry is added, but no earlier definition exists; subsequent lookups find it, see deleted, and return NULL \u2014 same as if it wasn't there).\n\nThe test exercises a redefinition case where the macro replaces a *keyword*:\n\n```c\n#define if 5\n// ...\nASSERT_ 5, if, five END;\n\n#undef if\n// ...\nif (0);\n```\n\n\u2014 while `if` is `#define`d to `5`, the source `if (0);` would expand to `5 (0);` and miscompile. After `#undef if`, the keyword reasserts itself. (Recall: `convert_keywords` runs *after* `preprocess`, so a macro with a keyword name actually wins during preprocessing \u2014 the macro lookup happens before keyword classification.)\n\n### 17.3.3 \u2014 Expand macros in `#if` and `#elif` arguments\n\n> `git checkout 2651448084a56dd0b960989798772e71e12e6c30` \u2014 *Expand macros in the `#if` and `#elif` argument context*\n\nThree lines and a forward declaration. The `eval_const_expr` function gets a recursive call to `preprocess2` over its synthetic line:\n\n```c\nstatic Token *preprocess2(Token *tok);\n```\n\n```diff\n static long eval_const_expr(Token **rest, Token *tok) {\n   Token *start = tok;\n   Token *expr = copy_line(rest, tok->next);\n+  expr = preprocess2(expr);\n\n   if (expr->kind == TK_EOF)\n     error_tok(start, \"no expression\");\n```\n\nThe synthetic line is now run through the preprocessor before being handed to `const_expr`. That means `#if M` expands `M` to its body, then evaluates the body as a constant expression.\n\nA subtle effect: `preprocess2` runs the *full* directive recognizer, which would treat a `#if` inside `#if`'s argument as nested. In practice, directive arguments don't contain hashes (the C standard prohibits it), so the `is_hash` check inside `preprocess2`'s recursive call doesn't fire. The recursion is, in practice, *only* macro expansion plus `#if`-style passes-through.\n\nTest:\n\n```c\n#define M 5\n#if M\n  m = 5;\n#else\n  m = 6;\n#endif\n  assert(5, m, \"m\");\n```\n\n\u2014 `#if M` becomes `#if 5`, which is true.\n\n### 17.3.4 \u2014 The no-rescan rule and the hideset\n\n> `git checkout acce00228b842af35df5af8c97398765a386ab1e` \u2014 *Do not expand a token more than once for the same objlike macro*\n\nThis is the most subtle commit in the chapter, and possibly in the book. The `#define X X + 1` problem \u2014 what stops `X` from expanding to `X + 1` infinitely? The C standard's answer is: *each token, in the result of macro expansion, carries a record of which macros have already been expanded at it.* The set of those macro names is called a *hideset*. When a token's hideset contains macro `M`, the preprocessor will not expand that token via `M`. Self-reference terminates: the expansion `X` \u2192 `X + 1` paints the result token's hideset with `{X}`, so the rescan sees `X` (with hideset `{X}`) and doesn't try to expand it.\n\nThe data structure is a singly-linked list of macro names per token:\n\n```c\ntypedef struct Hideset Hideset;\nstruct Hideset {\n  Hideset *next;\n  char *name;\n};\n```\n\n\u2014 and `Token` gains a `Hideset *hideset` field. (No relation to `at_bol` / `has_space` \u2014 the hideset is per-token state used purely by the preprocessor.)\n\nThe hideset operations are union, intersection (added in \u00a717.4 for function-like macros), and contains:\n\n```c\nstatic Hideset *new_hideset(char *name) { /* ... */ }\n\nstatic Hideset *hideset_union(Hideset *hs1, Hideset *hs2) {\n  Hideset head = {};\n  Hideset *cur = &head;\n\n  for (; hs1; hs1 = hs1->next)\n    cur = cur->next = new_hideset(hs1->name);\n  cur->next = hs2;\n  return head.next;\n}\n\nstatic bool hideset_contains(Hideset *hs, char *s, int len) {\n  for (; hs; hs = hs->next)\n    if (strlen(hs->name) == len && !strncmp(hs->name, s, len))\n      return true;\n  return false;\n}\n```\n\n`hideset_union` is destructive on hs2 (it appends hs1's elements as fresh links and stitches hs2 onto the end); for the in-pass rewriter pattern this is fine because the set's tail is never rewound.\n\n`add_hideset` paints the union onto every token in a list:\n\n```c\nstatic Token *add_hideset(Token *tok, Hideset *hs) {\n  Token head = {};\n  Token *cur = &head;\n\n  for (; tok; tok = tok->next) {\n    Token *t = copy_token(tok);\n    t->hideset = hideset_union(t->hideset, hs);\n    cur = cur->next = t;\n  }\n  return head.next;\n}\n```\n\n\u2014 each result token is a fresh copy so the original `Macro->body` is never mutated (the same body might be expanded many times, in many contexts, with many different hidesets).\n\nThe expansion hook:\n\n```c\nstatic bool expand_macro(Token **rest, Token *tok) {\n  if (hideset_contains(tok->hideset, tok->loc, tok->len))\n    return false;\n\n  Macro *m = find_macro(tok);\n  if (!m)\n    return false;\n\n  Hideset *hs = hideset_union(tok->hideset, new_hideset(m->name));\n  Token *body = add_hideset(m->body, hs);\n  *rest = append(body, tok->next);\n  return true;\n}\n```\n\nRead top-down. *If this token's hideset contains a macro of its own name, do not expand.* That's the gate \u2014 the test that makes recursion terminate. Otherwise, look up the macro. The replacement's hideset is the **union** of the source token's hideset (whatever painted *it*) and a singleton `{m->name}` (this expansion's own contribution). The replacement tokens all get that union painted on them via `add_hideset`. Then splice and return.\n\nThe walked example for `#define M2 M2 + 3` followed by `M2`:\n\n1. The source `M2` token has empty hideset. `expand_macro` is called with that token.\n2. The hideset doesn't contain `M2`, so we don't bail.\n3. `find_macro` returns the `M2` macro.\n4. New hideset: union of `{}` and `{M2}` is `{M2}`.\n5. The body `M2 + 3` has each of its three tokens copied, with hideset `{M2}` painted on each.\n6. `*rest` now starts with the painted `M2`, then `+`, then `3`, then the rest of the input.\n7. The outer `preprocess2` loop iterates. It calls `expand_macro` on the painted `M2`.\n8. The hideset contains `M2`. Bail.\n9. The painted `M2` is then passed through as an ordinary identifier (it's not a directive, not another expandable macro).\n10. After parsing, `M2` is an undefined identifier \u2014 a compile error if the source actually references it as a value. The test `#define M2 M2 + 3` followed by `int M2 = 6;` followed by `assert(9, M2, ...)` works because the *first* `M2` (in the assertion) has empty hideset, expands to `M2 + 3` (with `{M2}` hideset on each), then the painted `M2` reaches the parser as the variable reference (which has value 6), so the result is `6 + 3 = 9`.\n\nThe walked test for two-macro mutual recursion:\n\n```c\nint M4 = 3;\n#define M4 M5 * 5\n#define M5 M4 + 2\nASSERT(13, M4);\n```\n\nExpansion of the source `M4` (empty hideset):\n\n1. Painted body of `M4`: `M5 * 5`, each with hideset `{M4}`.\n2. Splice. Loop reads `M5` (hideset `{M4}`).\n3. `M5`'s hideset doesn't contain `M5`. Look up; find `M5`. New hideset: `{M4} \u222a {M5}` = `{M4, M5}`.\n4. Painted body of `M5`: `M4 + 2`, each with hideset `{M4, M5}`.\n5. Splice. Loop reads `M4` (hideset `{M4, M5}`).\n6. `M4`'s hideset contains `M4`. Bail. `M4` passes through.\n7. Loop reads `+`, `2`, `*` (from the original `M4` body after the painted `M5`), `5`. All non-macro. All pass through.\n8. Final token stream: `M4 + 2 * 5`, where the bare `M4` is the variable reference (value 3). Result: `3 + 2*5 = 13`.\n\nThe bookend asserts the value: `assert(13, M4, \"M4\")`.\n\nThe hideset's elegance is twofold. First, it's *correct*: the rule terminates regardless of how complicated the macro graph is, because no token can be expanded by the same macro twice. Second, it's *local*: the rule operates on the token, not on global counters or recursion-depth limits. A token painted with `{M4, M5}` carries that paint with it through subsequent expansions, even into completely unrelated regions of the program.\n\nThe algorithm's name is *Prosser's algorithm*. Rui's `preprocess.c` opens with a 23-line comment block explaining the rule, and links to a copy of Prosser's 1986 document in the chibicc wiki. The C standard's wording on macro expansion is, in the standards committee's own footnote, \"based on\" Prosser's algorithm. (Some informal C-implementation literature calls it the \"blue paint\" algorithm \u2014 the same metaphor: a macro's name \"paints\" the result tokens in its color, and a painted token can never be re-expanded by the painter.)\n\n#### A concept interlude \u2014 why the paint must terminate\n\nIt's worth pausing on the *why* of the rule, separately from the implementation.\n\nThe C preprocessor is a token-rewriting system. A naive token-rewriting system can diverge: if rule R rewrites X to a phrase containing X, and we re-apply R to the result, we get an infinite expansion. Real-world rewrite systems handle this through various means \u2014 confluence proofs, termination measures, depth limits. The C preprocessor uses a finer-grained device: it records, *per token*, which rewrite rules have already applied at that position, and forbids the same rule from applying twice.\n\nWhy per-token? Because the alternative \u2014 a global \"we already expanded X once in this file\" rule \u2014 is too coarse. The user might write:\n\n```c\n#define X 1\nint a = X;\n#undef X\n#define X 2\nint b = X;\n```\n\n\u2014 and expect `a = 1, b = 2`. A global rule that disabled `X` after its first expansion would give `a = 1, b = X`. The per-token rule lets `X` be expanded once at the first reference and once at the second; the *result tokens* carry the paint, not the *macro definition*.\n\nWhy a *set* per token, rather than a single name? Because macros nest. If `X` expands to `Y X`, the result `Y X` should have `{X}` painted on its tokens (so the inner `X` doesn't re-expand). If `Y` then expands to `Z Y`, the result of *that* should have `{X, Y}` painted (so neither `X` nor `Y` re-expands). The set grows as expansions nest. Each expansion's contribution is its own name; the set is the cumulative paint.\n\nThe alternative implementation \u2014 a recursion-depth limit, \"stop after N nested expansions\" \u2014 is what some non-standard preprocessors use. It's *almost* correct for normal code (no real macro nests deeper than 100), but it fails the same `#define X X` test for principled reasons: depth N+1 *does* equal depth N, which is finite, so the recursion would stop, but for the wrong reason. The depth limit is a safety net, not a semantic rule. The hideset is a semantic rule.\n\nThe chibicc implementation lifts the algorithm directly from Prosser. The function-like-macro variant in \u00a717.4 introduces hideset *intersection* (rather than union) at the closing-paren of an argument list, because the tokens spanning a function-like-macro invocation may have *different* hidesets, and the standard requires the hideset of the result to be only those macros *all* of the spanning tokens agreed on. We'll work through that in \u00a717.4.\n\n### 17.3.5 \u2014 `#ifdef` and `#ifndef`\n\n> `git checkout 1f80f581e517ae4a5df6ab38af48a0d2a1089c73` \u2014 *Add `#ifdef` and `#ifndef`*\n\nTwo more conditional directives. `#ifdef X` is shorthand for `#if defined(X)` (which chibicc doesn't have yet \u2014 `defined()` is in \u00a717.5). The implementation reads the next token as an identifier and tests `find_macro`:\n\n```c\nif (equal(tok, \"ifdef\")) {\n  bool defined = find_macro(tok->next);\n  push_cond_incl(tok, defined);\n  tok = skip_line(tok->next->next);\n  if (!defined)\n    tok = skip_cond_incl(tok);\n  continue;\n}\n\nif (equal(tok, \"ifndef\")) {\n  bool defined = find_macro(tok->next);\n  push_cond_incl(tok, !defined);\n  tok = skip_line(tok->next->next);\n  if (defined)\n    tok = skip_cond_incl(tok);\n  continue;\n}\n```\n\nThe two skip-functions update to recognize `#ifdef` and `#ifndef` as starting-tokens for nested skipping:\n\n```c\nif (is_hash(tok) &&\n    (equal(tok->next, \"if\") || equal(tok->next, \"ifdef\") ||\n     equal(tok->next, \"ifndef\"))) {\n```\n\n\u2014 so `#if 0\\n#ifdef X\\n#endif\\n#endif` is handled correctly.\n\n`#define M6` (with no body) followed by `#ifdef M6` works because `#define M6` adds a macro with empty body; `find_macro(M6)` returns non-NULL; `defined = true`; the body is included.\n\n#### Where the section leaves us\n\nThe full object-like-macro surface: definitions, redefinitions, undefinitions, expansion-in-conditional, expansion-elsewhere, the no-rescan rule, and `#ifdef`/`#ifndef`. The hideset is per-token state, painted on every result token by `add_hideset`, gated on by `expand_macro`'s `hideset_contains` check. The data structure is a small linked list; the operations are union and contains for the object-like case (intersection arrives in \u00a717.4).\n\nThe duplication-and-redundancy note: chibicc has, after \u00a717.3, *six* directives that all end with `skip_line(tok)` to consume trailing junk. They're each five-to-seven lines of code, repeating the same pattern: read, validate, dispatch, skip_line. Rui doesn't factor out a `read_directive` helper because the dispatch differs across them (`#define` reads an identifier and a body; `#if` reads a constexpr; `#ifdef` reads an identifier and tests it). The repetition is what `quotes-rui.md` calls \"small duplications\" \u2014 tolerated rather than abstracted.\n\n---\n\nCHAPTERMARKER\nwc -w /Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md",
  "description": "Append \u00a717.2 and \u00a717.3"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cat >> /Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md << 'CHAPTERMARKER'\n## 17.4 \u2014 Function-like macros, stringizing, and pasting\n\nSeven commits. Function-like macros \u2014 `#define MAX(a, b) ((a) < (b) ? (b) : (a))` \u2014 are the C preprocessor's most code-heavy feature. They split into a definition stage (read parameter names), an invocation stage (read argument tokens, one per parameter, balancing parentheses), a substitution stage (walk the body, replace parameter names with their arguments), and an expansion stage (the substituted body becomes the replacement, with the no-rescan rule applied via the closing-paren hideset). Stringizing (`#x`) and pasting (`##`) are operators that act inside the substitution stage on parameter references.\n\n### 17.4.1 \u2014 Zero-arity function-like `#define`\n\n> `git checkout dec3b3fa02ffb343c37f82d36ae02be6bb30eb03` \u2014 *Add zero-arity funclike `#define`*\n\nThe first function-like-macro commit handles the simplest case: `#define X() 1` \u2014 a zero-parameter function-like macro, invoked as `X()`. The interesting wrinkle is *distinguishing* a function-like definition from an object-like one with a parenthesized body: `#define X (1)` is object-like (X expands to `(1)`), while `#define X() 1` is function-like (X expands to `1` only when invoked as `X()`). The disambiguator is *whitespace*: a `(` *immediately after* the macro name (no space between) introduces parameters; a `(` *with* whitespace before it is the start of the body.\n\nThe tokenizer gains a `has_space` flag on `Token`, mirroring `at_bol`:\n\n```c\nstatic bool has_space;\n// ...\ntok->has_space = has_space;\nat_bol = has_space = false;\n```\n\n\u2014 set to true when the tokenizer skips whitespace, comments, or block comments, cleared when a token is emitted.\n\n`Macro` gains `is_objlike`:\n\n```c\nstruct Macro {\n  Macro *next;\n  char *name;\n  bool is_objlike;\n  Token *body;\n  bool deleted;\n};\n```\n\n`read_macro_definition` is the new shared definition-reader. It branches on `!tok->has_space && equal(tok, \"(\")`:\n\n```c\nstatic void read_macro_definition(Token **rest, Token *tok) {\n  if (tok->kind != TK_IDENT)\n    error_tok(tok, \"macro name must be an identifier\");\n  char *name = strndup(tok->loc, tok->len);\n  tok = tok->next;\n\n  if (!tok->has_space && equal(tok, \"(\")) {\n    // Function-like macro\n    tok = skip(tok->next, \")\");\n    add_macro(name, false, copy_line(rest, tok));\n  } else {\n    // Object-like macro\n    add_macro(name, true, copy_line(rest, tok));\n  }\n}\n```\n\n`expand_macro` branches on `is_objlike`:\n\n```c\n// Object-like macro application\nif (m->is_objlike) {\n  Hideset *hs = hideset_union(tok->hideset, new_hideset(m->name));\n  Token *body = add_hideset(m->body, hs);\n  *rest = append(body, tok->next);\n  return true;\n}\n\n// If a funclike macro token is not followed by an argument list,\n// treat it as a normal identifier.\nif (!equal(tok->next, \"(\"))\n  return false;\n\n// Function-like macro application\ntok = skip(tok->next->next, \")\");\n*rest = append(m->body, tok);\nreturn true;\n```\n\nThe function-like branch returns `false` if the next token isn't `(` \u2014 that's the case where `M7` is defined as `#define M7() 1` but used as just `M7` (no parens). Such a use is a *non-invocation*; the identifier passes through as itself. The test:\n\n```c\n#define M7() 1\nint M7 = 5;\nASSERT(1, M7());\nASSERT(5, M7);\n```\n\n\u2014 an identifier `M7` is accepted both as a function-like macro invocation (with parens) and as the variable reference (without). The same name lives in both namespaces. (This is also why pure macros redefining keywords work: `#define if 5` followed by `if (0);` after `#undef if` works because the macro is gone by the time the code is parsed.)\n\nThe zero-arity case skips the body's argument processing entirely (`m->body` is appended directly to `tok` after the `)`, with no substitution). That's why \u00a717.4.2 (multi-arity) is the substantial commit, not this one.\n\nThe print-tokens function in `main.c` updates to honor `has_space`:\n\n```diff\n-    fprintf(out, \" %.*s\", tok->len, tok->loc);\n+    if (tok->has_space && !tok->at_bol)\n+      fprintf(out, \" \");\n+    fprintf(out, \"%.*s\", tok->len, tok->loc);\n```\n\n\u2014 the `-E` output now reproduces *only* the spaces that were in the source (or that were preserved through expansion).\n\n### 17.4.2 \u2014 Multi-arity function-like `#define`\n\n> `git checkout b9ad3e43cf7479712972514aa3f2c55a0f650f76` \u2014 *Add multi-arity funclike `#define`*\n\nThe biggest single commit in \u00a717.4. It introduces parameter parsing, argument parsing, and substitution.\n\n`MacroParam` is a parameter list:\n\n```c\ntypedef struct MacroParam MacroParam;\nstruct MacroParam {\n  MacroParam *next;\n  char *name;\n};\n```\n\n`MacroArg` is a parsed argument:\n\n```c\ntypedef struct MacroArg MacroArg;\nstruct MacroArg {\n  MacroArg *next;\n  char *name;\n  Token *tok;\n};\n```\n\nNote the *parallel structure*: arguments carry parameter names (the `name` field), letting `find_arg(args, tok)` look up an argument by parameter name during substitution.\n\n`Macro` gains `MacroParam *params`. `read_macro_params` parses the comma-separated parameter list:\n\n```c\nstatic MacroParam *read_macro_params(Token **rest, Token *tok) {\n  MacroParam head = {};\n  MacroParam *cur = &head;\n\n  while (!equal(tok, \")\")) {\n    if (cur != &head)\n      tok = skip(tok, \",\");\n\n    if (tok->kind != TK_IDENT)\n      error_tok(tok, \"expected an identifier\");\n    MacroParam *m = calloc(1, sizeof(MacroParam));\n    m->name = strndup(tok->loc, tok->len);\n    cur = cur->next = m;\n    tok = tok->next;\n  }\n  *rest = tok->next;\n  return head.next;\n}\n```\n\n`read_macro_arg_one` reads a single argument's tokens up to a `,` or `)`:\n\n```c\nstatic MacroArg *read_macro_arg_one(Token **rest, Token *tok) {\n  Token head = {};\n  Token *cur = &head;\n\n  while (!equal(tok, \",\") && !equal(tok, \")\")) {\n    if (tok->kind == TK_EOF)\n      error_tok(tok, \"premature end of input\");\n    cur = cur->next = copy_token(tok);\n    tok = tok->next;\n  }\n\n  cur->next = new_eof(tok);\n\n  MacroArg *arg = calloc(1, sizeof(MacroArg));\n  arg->tok = head.next;\n  *rest = tok;\n  return arg;\n}\n```\n\n\u2014 the argument's tokens are copied into a fresh list terminated by EOF, so `subst` can iterate over it as a small token stream. (Argument tokens are *not yet* expanded; expansion happens lazily inside `subst`.)\n\n`read_macro_args` zips parameters and arguments:\n\n```c\nstatic MacroArg *read_macro_args(Token **rest, Token *tok, MacroParam *params) {\n  Token *start = tok;\n  tok = tok->next->next;  // skip past macro name and '('\n\n  MacroArg head = {};\n  MacroArg *cur = &head;\n\n  MacroParam *pp = params;\n  for (; pp; pp = pp->next) {\n    if (cur != &head)\n      tok = skip(tok, \",\");\n    cur = cur->next = read_macro_arg_one(&tok, tok);\n    cur->name = pp->name;\n  }\n\n  if (pp)\n    error_tok(start, \"too many arguments\");\n  *rest = skip(tok, \")\");\n  return head.next;\n}\n```\n\nThe loop walks `params` in lockstep with the comma-separated argument tokens. Each argument inherits its parameter's name. Too few or too many arguments is an error.\n\n`subst` is the substitution pass:\n\n```c\nstatic Token *subst(Token *tok, MacroArg *args) {\n  Token head = {};\n  Token *cur = &head;\n\n  while (tok->kind != TK_EOF) {\n    MacroArg *arg = find_arg(args, tok);\n\n    // Macro arguments are completely macro-expanded\n    // before they are substituted into a macro body.\n    if (arg) {\n      Token *t = preprocess2(arg->tok);\n      for (; t->kind != TK_EOF; t = t->next)\n        cur = cur->next = copy_token(t);\n      tok = tok->next;\n      continue;\n    }\n\n    // Handle a non-macro token.\n    cur = cur->next = copy_token(tok);\n    tok = tok->next;\n    continue;\n  }\n\n  cur->next = tok;\n  return head.next;\n}\n```\n\nRead carefully. Each token of the body is either a parameter reference (in which case its argument's tokens are pre-expanded via `preprocess2` and spliced in) or a literal token (in which case it's copied through). The pre-expansion of arguments is a key piece of the C standard's specified order: `f(g(x))` \u2014 where `f` and `g` are both macros \u2014 has `g(x)` fully expanded before being substituted into `f`'s body. This matters for the no-rescan rule: arguments are expanded *with their own hidesets*, separately from the function-like macro's expansion paint.\n\n`expand_macro`'s function-like branch becomes:\n\n```c\n// Function-like macro application\nMacroArg *args = read_macro_args(&tok, tok, m->params);\n*rest = append(subst(m->body, args), tok);\nreturn true;\n```\n\nThe hideset machinery isn't fully integrated yet \u2014 that's \u00a717.4.5. This commit's expansion *can* infinite-loop on a self-referential function-like macro. It happens to not, in the tests, because the tests don't construct one.\n\nThe two-line tests cover redefinition and basic substitution:\n\n```c\n#define M8(x,y) x+y\nASSERT(7, M8(3, 4));\n\n#define M8(x,y) x*y\nASSERT(24, M8(3+4, 4+5));     // (3+4) * (4+5) but parsed as 3+4*4+5 = 24\n\n#define M8(x,y) (x)*(y)\nASSERT(63, M8(3+4, 4+5));     // (3+4)*(4+5) = 63\n```\n\n\u2014 the second case demonstrates the classic argument-grouping pitfall: `M8(3+4, 4+5)` with `x*y` expands to `3+4*4+5 = 3 + 16 + 5 = 24`, not `63`. The `(x)*(y)` definition fixes it.\n\n### 17.4.3 \u2014 Allow empty macro arguments\n\n> `git checkout dd4306cdd8158f76f094fc699530311228536adb` \u2014 *Allow empty macro arguments*\n\nThree test lines, no preprocess.c change. The previous commit's `read_macro_arg_one` already handles empty arguments (its `while` loop terminates at `,` or `)`, which produces an empty token list if `,` is the very first token). The test:\n\n```c\n#define M8(x,y) x y\nASSERT(9, M8(, 4+5));       // empty x, y = 4+5; result is \" 4+5\" \u2192 4+5 = 9\n```\n\n\u2014 `M8(, 4+5)`'s first argument is empty; the substitution `x y` becomes `(empty) (4+5)` = ` 4+5` = `4+5`. Three lines, but worth its own commit because it's a behavior assertion.\n\n### 17.4.4 \u2014 Allow parenthesized expressions as macro arguments\n\n> `git checkout c7d7ce0f0cbd5869259a3365211ab92126a27ff6` \u2014 *Allow parenthesized expressions as macro arguments*\n\n`read_macro_arg_one` learns to balance parentheses. Pre-this-commit, `M8((2+3), 4)` would tokenize the first argument as `(2+3` (stopping at the comma inside, even though it's nested). Post-this-commit, it tracks a `level` counter:\n\n```c\nstatic MacroArg *read_macro_arg_one(Token **rest, Token *tok) {\n  Token head = {};\n  Token *cur = &head;\n  int level = 0;\n\n  while (level > 0 || (!equal(tok, \",\") && !equal(tok, \")\"))) {\n    if (tok->kind == TK_EOF)\n      error_tok(tok, \"premature end of input\");\n\n    if (equal(tok, \"(\"))\n      level++;\n    else if (equal(tok, \")\"))\n      level--;\n\n    cur = cur->next = copy_token(tok);\n    tok = tok->next;\n  }\n  // ...\n}\n```\n\n\u2014 `level > 0` keeps the loop going past commas and right-parens that are nested inside an opened paren. `M8((2,3), 4)` now lexes as `(2,3)` and `4`. Two-token nested expressions like `M8((2+3), 4)` work too.\n\n### 17.4.5 \u2014 The function-like no-rescan rule and hideset intersection\n\n> `git checkout 1313fc6d3a77cedbca18fa0ffee1a86d0903ad7f` \u2014 *Do not expand a token more than once for the same funclike macro*\n\nThe function-like-macro's no-rescan rule is subtler than the object-like one. Object-like: take the source token's hideset, union with `{macroname}`, paint the result. Function-like: the *invocation* spans multiple tokens (the macro name, the open paren, the arguments, the close paren). Those tokens may have *different* hidesets \u2014 they came from different expansions. The standard says: the result's hideset is the **intersection** of the hidesets at the *first* token (the macro name) and at the *last* token (the closing paren), unioned with `{macroname}`.\n\nThe intersection helper:\n\n```c\nstatic Hideset *hideset_intersection(Hideset *hs1, Hideset *hs2) {\n  Hideset head = {};\n  Hideset *cur = &head;\n\n  for (; hs1; hs1 = hs1->next)\n    if (hideset_contains(hs2, hs1->name, strlen(hs1->name)))\n      cur = cur->next = new_hideset(hs1->name);\n  return head.next;\n}\n```\n\nThe expansion's function-like branch updates:\n\n```c\n// Function-like macro application\nToken *macro_token = tok;\nMacroArg *args = read_macro_args(&tok, tok, m->params);\nToken *rparen = tok;\n\n// Tokens that consist a func-like macro invocation may have different\n// hidesets, and if that's the case, it's not clear what the hideset\n// for the new tokens should be. We take the interesection of the\n// macro token and the closing parenthesis and use it as a new hideset\n// as explained in the Dave Prossor's algorithm.\nHideset *hs = hideset_intersection(macro_token->hideset, rparen->hideset);\nhs = hideset_union(hs, new_hideset(m->name));\n\nToken *body = subst(m->body, args);\nbody = add_hideset(body, hs);\n*rest = append(body, tok->next);\nreturn true;\n```\n\n`read_macro_args` also has its `*rest = skip(tok, \")\")` changed to leave `tok` at the close-paren rather than advancing past it (so that `rparen = tok` captures the close-paren's hideset before we consume it):\n\n```diff\n-  *rest = skip(tok, \")\");\n+  skip(tok, \")\");\n+  *rest = tok;\n```\n\n\u2014 `skip` here is purely an assertion (it errors out if the token isn't `)`); the *advance* happens at the bottom of `expand_macro` via `tok->next`.\n\nThe walked example for the test case:\n\n```c\n#define dbl(x) M10(x) * x\n#define M10(x) dbl(x) + 3\nASSERT(10, dbl(2));\n```\n\nSource: `dbl(2)` \u2014 both tokens have empty hideset.\n\n1. `expand_macro` on `dbl`. Hideset of `dbl` is `{}`. Hideset of `)` is `{}`. Intersection: `{}`. Union with `{dbl}` = `{dbl}`.\n2. Substitute `x = 2` in `M10(x) * x`: result is `M10(2) * 2`. Paint `{dbl}` on every token: `M10\u208ddbl\u208e(\u208ddbl\u208e2\u208ddbl\u208e)\u208ddbl\u208e *\u208ddbl\u208e 2\u208ddbl\u208e`.\n3. Splice. Loop reads `M10` (hideset `{dbl}`).\n4. `M10`'s hideset doesn't contain `M10`. Look up. `(` after has hideset `{dbl}`. `)` after `2` has hideset `{dbl}`. Intersection of `{dbl}` and `{dbl}` is `{dbl}`. Union with `{M10}` = `{dbl, M10}`.\n5. Substitute `x = 2` in `dbl(x) + 3`: result is `dbl(2) + 3`. Paint `{dbl, M10}`: `dbl\u208ddbl,M10\u208e(2) + 3`.\n6. Splice. Loop reads `dbl` (hideset `{dbl, M10}`).\n7. `dbl`'s hideset *contains* `dbl`. Bail. `dbl` passes through as an ordinary identifier.\n8. Loop reads `(`, `2`, `)`, `+`, `3`, `*`, `2`. All non-macro. All pass through.\n9. Final: `dbl(2) + 3 * 2`.\n10. The leftover `dbl` matches the source-level *function* `int dbl(int x) { return x*x; }` defined earlier in the test. So `dbl(2) = 4`. Result: `4 + 3*2 = 10`. Asserted.\n\nTwo macros chase each other recursively but the hideset terminates expansion after one round of each. The *function* `dbl` (lower-case identifier shadowing the macro at the call site after the macro is hidden) catches the residual call.\n\nThe intersection rule's *why*: when an invocation's macro name has hideset `H1` and its close-paren has hideset `H2`, the only macros that have *definitely* already painted across this whole invocation are those in `H1 \u2229 H2`. A macro in `H1 \\ H2` painted only the start; a macro in `H2 \\ H1` painted only the end. The result's tokens haven't been protected against either \u2014 a future rescan should be able to expand them.\n\n### 17.4.6 \u2014 Stringizing (`#`)\n\n> `git checkout 8f6f7925a04ca070167a38b8952a1a0bb7b63d23` \u2014 *Add macro stringizing operator (`#`)*\n\n`#x` (where `x` is a macro parameter) substitutes a *string literal* containing the textual form of the argument. `#define M11(x) #x; M11(a + b)` produces `\"a + b\"`.\n\nThe implementation has three pieces. `quote_string` produces the C-source-level form of a string with `\\` and `\"` escaped:\n\n```c\nstatic char *quote_string(char *str) { /* doubles every \\ and \" */ }\n```\n\n`new_str_token` runs the result through the tokenizer to produce a string-literal token:\n\n```c\nstatic Token *new_str_token(char *str, Token *tmpl) {\n  char *buf = quote_string(str);\n  return tokenize(new_file(tmpl->file->name, tmpl->file->file_no, buf));\n}\n```\n\nThis is the kind of move that's pragmatic but worth pausing on: rather than constructing a `TK_STR` token by hand (with all its `Type *ty`, `char *str`, etc.), Rui *runs the tokenizer on the synthesized text*. The text contains a quoted string; the tokenizer produces the right `TK_STR` token. The `tmpl` parameter is the source-position template for error reporting.\n\n`tokenize` and `new_file` get exposed in `chibicc.h`:\n\n```c\nFile *new_file(char *name, int file_no, char *contents);\nToken *tokenize(File *file);\n```\n\n`join_tokens` concatenates the textual forms of a token list:\n\n```c\nstatic char *join_tokens(Token *tok) {\n  // Compute the length, copy in token texts, separate by ' ' on has_space.\n}\n```\n\n`stringize` ties the two together:\n\n```c\nstatic Token *stringize(Token *hash, Token *arg) {\n  char *s = join_tokens(arg);\n  return new_str_token(s, hash);\n}\n```\n\nAnd the substitution pass gains a `#`-handling branch:\n\n```c\n// \"#\" followed by a parameter is replaced with stringized actuals.\nif (equal(tok, \"#\")) {\n  MacroArg *arg = find_arg(args, tok->next);\n  if (!arg)\n    error_tok(tok->next, \"'#' is not followed by a macro parameter\");\n  cur = cur->next = stringize(tok, arg->tok);\n  tok = tok->next->next;\n  continue;\n}\n```\n\n\u2014 note the `find_arg` is called on `tok->next` (the parameter name after the `#`); the result is the argument's token list, which `stringize` joins and quotes.\n\nThe test exercises preserved spaces:\n\n```c\n#define M11(x) #x\nASSERT('a', M11( a!b  `\"\"c)[0]);  // string is \"a!b  `\\\"\\\"c\"\nASSERT('!', M11( a!b  `\"\"c)[1]);\nASSERT(' ', M11( a!b  `\"\"c)[3]);\nASSERT('`', M11( a!b  `\"\"c)[4]);\nASSERT('\"', M11( a!b  `\"\"c)[5]);  // escaped \\\" round-trips\n```\n\nThe argument `a!b  \\`\"\"c` (eight tokens, lots of whitespace) stringizes to `\"a!b  \\`\\\"\\\"c\"` (the spaces are preserved because `has_space` was set on the inner tokens; the `\\\"` are escaped because `quote_string` doubles them). The C-source-level result is a single string literal that, when read by the C string-literal tokenizer, produces the expected character sequence.\n\n### 17.4.7 \u2014 Pasting (`##`)\n\n> `git checkout 8f561aed9b7a47c38afd8c1cc75bc9a700ae97b5` \u2014 *Add macro token-pasting operator (`##`)*\n\nThe token-pasting operator: `a ## b` produces the *single* token `ab`. Unlike stringizing, it doesn't quote \u2014 it concatenates the source text and re-tokenizes:\n\n```c\nstatic Token *paste(Token *lhs, Token *rhs) {\n  char *buf = format(\"%.*s%.*s\", lhs->len, lhs->loc, rhs->len, rhs->loc);\n\n  Token *tok = tokenize(new_file(lhs->file->name, lhs->file->file_no, buf));\n  if (tok->next->kind != TK_EOF)\n    error_tok(lhs, \"pasting forms '%s', an invalid token\", buf);\n  return tok;\n}\n```\n\nThe *invalid token* check: pasting must produce a *single* token. `1 ## 5` \u2192 `15` (one token). `1 ## +` \u2192 `1+` would tokenize as two tokens (`1` and `+`); error.\n\n`##` becomes a punctuator in `tokenize.c`:\n\n```diff\n-    \"||\", \"<<\", \">>\",\n+    \"||\", \"<<\", \">>\", \"##\",\n```\n\n\u2014 so the tokenizer emits `##` as one token rather than two `#`s.\n\nThe substitution pass gains two branches. First, `##` directly:\n\n```c\nif (equal(tok, \"##\")) {\n  if (cur == &head)\n    error_tok(tok, \"'##' cannot appear at start of macro expansion\");\n  if (tok->next->kind == TK_EOF)\n    error_tok(tok, \"'##' cannot appear at end of macro expansion\");\n\n  MacroArg *arg = find_arg(args, tok->next);\n  if (arg) {\n    if (arg->tok->kind != TK_EOF) {\n      *cur = *paste(cur, arg->tok);\n      for (Token *t = arg->tok->next; t->kind != TK_EOF; t = t->next)\n        cur = cur->next = copy_token(t);\n    }\n    tok = tok->next->next;\n    continue;\n  }\n\n  *cur = *paste(cur, tok->next);\n  tok = tok->next->next;\n  continue;\n}\n```\n\n\u2014 after `##`, the next token is either a parameter (in which case the argument's first token is pasted to the previous output token, and the rest of the argument's tokens copy through) or a literal (in which case it's pasted directly). The empty-argument case (`arg->tok->kind == TK_EOF`) is handled by skipping.\n\nSecond, `arg ## ...` \u2014 a parameter followed by `##`:\n\n```c\nMacroArg *arg = find_arg(args, tok);\n\nif (arg && equal(tok->next, \"##\")) {\n  Token *rhs = tok->next->next;\n\n  if (arg->tok->kind == TK_EOF) {\n    MacroArg *arg2 = find_arg(args, rhs);\n    if (arg2) {\n      for (Token *t = arg2->tok; t->kind != TK_EOF; t = t->next)\n        cur = cur->next = copy_token(t);\n    } else {\n      cur = cur->next = copy_token(rhs);\n    }\n    tok = rhs->next;\n    continue;\n  }\n\n  for (Token *t = arg->tok; t->kind != TK_EOF; t = t->next)\n    cur = cur->next = copy_token(t);\n  tok = tok->next;\n  continue;\n}\n```\n\n\u2014 if the parameter is followed by `##`, *don't* expand it (`paste` is non-expanding by design; the C standard says macros aren't expanded across `##`'s operands), copy the argument's tokens through, and let the next loop iteration handle the `##`. The empty-argument case (`paste5,` with the empty side) defers to the rhs handling.\n\nThe cleanest tests:\n\n```c\n#define paste(x,y) x##y\nASSERT(15, paste(1,5));       // 1##5 \u2192 15\nASSERT(255, paste(0,xff));    // 0##xff \u2192 0xff (a hex literal)\nASSERT(3, ({ int foobar=3; paste(foo,bar); }));  // foo##bar \u2192 foobar\nASSERT(5, paste(5,));         // 5## \u2192 5 (empty rhs)\nASSERT(5, paste(,5));         // ##5 \u2192 5 (empty lhs)\n```\n\n\u2014 and the most twisted:\n\n```c\n#define i 5\nASSERT(101, ({ int i3=100; paste(1+i,3); }));\n#undef i\n```\n\n\u2014 `paste(1+i, 3)` should *not* expand `i` while pasting (the second argument's first token is `3`, but the first argument's last token is `i`, and we paste `i` to `3` to form `i3`, which is a valid identifier and references the local variable `i3 = 100`). The result is `1 + i3 = 1 + 100 = 101`. Pasting suppresses argument expansion on the side adjacent to `##`.\n\n#### Where the section leaves us\n\nFunction-like macros work end-to-end: definitions with parameters, invocations with comma-separated arguments, balanced-paren argument scanning, empty arguments, the no-rescan rule via hideset intersection, stringizing `#x`, pasting `a##b`. The substitution pass in `subst` is the chapter's most code-heavy single function \u2014 about 80 lines after this section, with branches for parameter substitution, stringizing, and pasting. The expansion driver in `expand_macro` is shorter but more subtle, doing the hideset bookkeeping that makes Prosser's algorithm terminate.\n\nThe non-trivial file-touch for the section: the tokenizer gains `##` as a punctuator, `has_space` tracking, and (in \u00a717.4.6) is exposed via `chibicc.h`. The header gets a `Hideset` forward declaration. Otherwise, all the code is in `preprocess.c`.\n\n---\n\n## 17.5 \u2014 Polish: the rest of the preprocessor\n\nEighteen commits. The longest stretch of the chapter, but the *easiest* to bundle, because each commit is small and topic-coherent. The eighteen group into four sub-bundles by topic:\n\n- **Tests, `defined()`, identifier-to-zero, whitespace** (commits 179\u2013182): switching the test pipeline off `cc -E`, adding `defined()`, the constexpr-identifier-to-zero rule, and whitespace preservation through expansion.\n- **Line continuation, `<...>` and search paths** (commits 183\u2013186): the trailing-`\\`-newline handling, `#include <...>`, `-I<dir>`, default include paths.\n- **`#error` and predefined macros** (commits 187\u2013192): `#error`, the `__STDC__`/`__x86_64__`/`__linux__` block, `__FILE__` and `__LINE__` (with the `Token->origin` chain), `__VA_ARGS__`, `__func__`, `__FUNCTION__`.\n- **Adjacent-string concat, wide chars, headers, `va_arg`** (commits 193\u2013196): adjacent-string-literal concatenation, wide character literal `L'\\0'`, the standard headers, `va_arg()` as a real macro.\n\nEach sub-bundle is one or two paragraphs.\n\n### 17.5.1 \u2014 Tests switch off `cc -E`; `defined()`; identifier-to-zero; whitespace\n\n> Commits `769b5a0` (use chibicc's preprocessor for all tests), `5cb2f89` (`defined()`), `a8d76ad` (identifier-to-zero), `8075582` (preserve newline and space).\n\nCommit 179 (`769b5a0`) is the chapter's first user-visible behavioral milestone: the test Makefile stops piping through `cc -E`. Pre-this-commit, the test pipeline was:\n\n```makefile\ntest/%.exe: chibicc test/%.c\n\t$(CC) -o- -E -P -C test/$*.c | ./chibicc -c -o test/$*.o -\n```\n\nPost-this-commit:\n\n```makefile\ntest/%.exe: chibicc test/%.c\n\t./chibicc -c -o test/$*.o test/$*.c\n```\n\n`test/macro.c` updates from `assert(...)` calls (which require its own forward declarations) to `ASSERT(...)` calls (a macro from `test/test.h`, included via the now-working `#include \"test.h\"`). This is the moment the chapter's progress *cuts over*. From here, every test runs through chibicc's own preprocessor. The host `cc` is still invoked by the test Makefile for the final link step (with `test/common`), but it's no longer the preprocessor.\n\nCommit 180 (`5cb2f89`, `defined()`) gives `#if` access to \"is this macro defined?\" The implementation is a substitution pass over the constexpr's synthetic line, inserting `0` or `1` for each `defined(X)` or `defined X`:\n\n```c\nstatic Token *read_const_expr(Token **rest, Token *tok) {\n  // walk tokens; replace defined(X) with new_num_token(m ? 1 : 0, start)\n}\n```\n\n\u2014 and the constexpr evaluator runs `read_const_expr` first, *before* `preprocess2`'s recursive macro-expansion pass. The order matters: `defined(X)` must be evaluated *without* X being expanded, even if X is defined as something else. Substituting first produces a clean `0` or `1` that survives the subsequent macro pass.\n\n`new_num_token(int val, Token *tmpl)` is the integer counterpart of `new_str_token` \u2014 synthesize a number's textual form, run it through the tokenizer:\n\n```c\nstatic Token *new_num_token(int val, Token *tmpl) {\n  char *buf = format(\"%d\\n\", val);\n  return tokenize(new_file(tmpl->file->name, tmpl->file->file_no, buf));\n}\n```\n\nCommit 181 (`a8d76ad`, identifier-to-zero) addresses: what should `#if foo` mean when `foo` is undefined? The C standard answers (6.10.1p4): replace remaining identifiers with `0`. So `#if foo` becomes `#if 0` (false). Chibicc's evaluator walks the post-expansion synthetic line and rewrites lingering `TK_IDENT` tokens to `0`-numbers:\n\n```c\nfor (Token *t = expr; t->kind != TK_EOF; t = t->next) {\n  if (t->kind == TK_IDENT) {\n    Token *next = t->next;\n    *t = *new_num_token(0, t);\n    t->next = next;\n  }\n}\n```\n\n\u2014 the in-place mutation preserves the list's chain.\n\nCommit 182 (`8075582`, preserve newline and space) is twelve lines in `subst` and `expand_macro`. After substitution, the *first* token of the expanded body inherits the `at_bol` and `has_space` flags of the source token at the call site. Without this, the `-E` output would lose the blank line or space that preceded the macro reference, breaking visual alignment. The `Token->at_bol` and `has_space` propagate through expansion as if the call site's whitespace had been carried.\n\n### 17.5.2 \u2014 Line continuation; `<...>`; `-I`; default include paths\n\n> Commits `b33fe0e` (line continuation), `d85fc4f` (`#include <...>`), `a1dd621` (`-I<dir>`), `a939a7a` (default include paths).\n\nCommit 183 (`b33fe0e`) handles `\\<newline>` by deleting the pair before tokenization. The function `remove_backslash_newline` walks the file's contents and removes each occurrence, taking care to *preserve the line count* (the deleted newlines are appended at the end so `__LINE__` stays accurate):\n\n```c\nstatic void remove_backslash_newline(char *p) {\n  int i = 0, j = 0, n = 0;\n\n  while (p[i]) {\n    if (p[i] == '\\\\' && p[i + 1] == '\\n') { i += 2; n++; }\n    else if (p[i] == '\\n') {\n      p[j++] = p[i++];\n      for (; n > 0; n--) p[j++] = '\\n';\n    }\n    else { p[j++] = p[i++]; }\n  }\n  for (; n > 0; n--) p[j++] = '\\n';\n  p[j] = '\\0';\n}\n```\n\n\u2014 the `n` counter tracks deleted newlines and re-inserts them at the next physical newline (or end-of-file), so the column offsets are wrong after a continuation but the line counts match. This is important for `#define` bodies that span continuation lines and for `__LINE__` accuracy in continuation regions.\n\nCommit 184 (`d85fc4f`, `#include <...>`) adds the angle-bracket form. The argument lexes as a sequence of tokens between `<` and `>` because the tokenizer never special-cased `<...>`. The `read_include_filename` function reconstructs the filename text from the token sequence:\n\n```c\nif (equal(tok, \"<\")) {\n  Token *start = tok;\n  for (; !equal(tok, \">\"); tok = tok->next)\n    if (tok->at_bol || tok->kind == TK_EOF)\n      error_tok(tok, \"expected '>'\");\n  *is_dquote = false;\n  *rest = skip_line(tok->next);\n  return join_tokens(start->next, tok);\n}\n```\n\n\u2014 `join_tokens` (the same helper used for stringizing) concatenates the tokens between the brackets, preserving spaces. The test cases `#include <foo.h>` and `#include <some/path/foo.h>` both work because `<` and `/` and `.` all tokenize as their own tokens that get joined back.\n\nThe `is_dquote` flag distinguishes the two forms: the `\"...\"` form first searches the *including file's directory* (Chapter 17.1.3); the `<...>` form does not. Both forms then fall through to the `-I`-path search and the default-include-path search.\n\nCommit 185 (`a1dd621`, `-I<dir>`) adds the `StringArray include_paths`. Each `-I<dir>` argument pushes a path. `search_include_paths` iterates:\n\n```c\nstatic char *search_include_paths(char *filename) {\n  if (filename[0] == '/') return filename;\n  for (int i = 0; i < include_paths.len; i++) {\n    char *path = format(\"%s/%s\", include_paths.data[i], filename);\n    if (file_exists(path)) return path;\n  }\n  return NULL;\n}\n```\n\n\u2014 first match wins. The `StringArray` type from Chapter 16's driver work gets its first \u00a717 reuse here.\n\nCommit 186 (`a939a7a`, default include paths) adds three system paths and one chibicc-specific path:\n\n```c\nstatic void add_default_include_paths(char *argv0) {\n  strarray_push(&include_paths, format(\"%s/include\", dirname(strdup(argv0))));\n  strarray_push(&include_paths, \"/usr/local/include\");\n  strarray_push(&include_paths, \"/usr/include/x86_64-linux-gnu\");\n  strarray_push(&include_paths, \"/usr/include\");\n}\n```\n\n\u2014 called from `cc1()` at startup. The `${argv0}/include` path is for chibicc's own header bundle (added in \u00a717.5.4 \u2014 `stdarg.h`, `stdbool.h`, etc.). The three system paths are Linux/glibc-specific (the same kind of brittleness Chapter 16's `find_libpath` showed). On a non-Linux system, the paths are wrong, and `#include <stdio.h>` would only succeed if the user supplied a working `-I`. Errata candidate, lower priority \u2014 same category as Chapter 16's link-path probing.\n\n### 17.5.3 \u2014 `#error`, predefined macros, `__FILE__`/`__LINE__`, `__VA_ARGS__`, `__func__`\n\n> Commits `e7fdc2e` (`#error`), `5f5a850` (predefined macros), `6f17071` (`__FILE__`/`__LINE__`), `dc01f94` (`__VA_ARGS__`), `ba6b4b6` (`__func__`), `82ba010` (`__FUNCTION__`).\n\nCommit 187 (`e7fdc2e`, `#error`) is *three lines*:\n\n```c\nif (equal(tok, \"error\"))\n  error_tok(tok, \"error\");\n```\n\n\u2014 `#error` aborts the preprocessor with an error message at the directive's location. The directive line's *content* (the diagnostic message the user wrote) isn't reported; chibicc just says \"error\". Errata candidate (real `#error` should print the rest of the directive line as the message).\n\nCommit 188 (`5f5a850`, predefined macros) adds 41 macros via `define_macro`:\n\n```c\nstatic void define_macro(char *name, char *buf) {\n  Token *tok = tokenize(new_file(\"<built-in>\", 1, buf));\n  add_macro(name, true, tok);\n}\n\nstatic void init_macros(void) {\n  define_macro(\"_LP64\", \"1\");\n  define_macro(\"__C99_MACRO_WITH_VA_ARGS\", \"1\");\n  define_macro(\"__ELF__\", \"1\");\n  // ... 38 more ...\n  define_macro(\"__x86_64__\", \"1\");\n  define_macro(\"linux\", \"1\");\n  define_macro(\"unix\", \"1\");\n}\n```\n\n\u2014 a Linux/x86-64-specific block that lets standard library headers detect chibicc's target. `__chibicc__` is also defined (`1`), letting consumer code detect the compiler. `__alignof__`, `__inline__`, etc., are alias macros \u2014 they expand to the proper C11 keyword (`_Alignof`, `inline`, etc.). `__STDC_VERSION__` is `201112L` (claiming C11 support, which chibicc imperfectly approximates).\n\nCommit 189 (`6f17071`, `__FILE__` and `__LINE__`) is the chapter's most *invasive* change to the existing `Token` struct after the hideset:\n\n```diff\n struct Token {\n   // ...\n   Hideset *hideset;\n+  Token *origin;    // If this is expanded from a macro, the original token\n };\n```\n\nThe `Token->origin` field forms a chain: an expanded token's `origin` points at its source token (the one in the macro invocation, before substitution); that source token's `origin` points at *its* origin if it was itself an expansion result; and so on. The chain ends at a token whose `origin` is `NULL` \u2014 the \"real\" source token, the one in the program's original `.c` file.\n\n`__FILE__` and `__LINE__` walk this chain to find the *real* source location:\n\n```c\nstatic Token *file_macro(Token *tmpl) {\n  while (tmpl->origin) tmpl = tmpl->origin;\n  return new_str_token(tmpl->file->name, tmpl);\n}\n\nstatic Token *line_macro(Token *tmpl) {\n  while (tmpl->origin) tmpl = tmpl->origin;\n  return new_num_token(tmpl->line_no, tmpl);\n}\n```\n\n\u2014 so `#define LINE() __LINE__` followed by `int x = LINE();` produces the line number where `LINE()` was *invoked*, not the line where `__LINE__` appears in the macro's body.\n\nThe `Macro` struct gains a `handler` field that lets a macro be evaluated by C code rather than by token substitution:\n\n```c\ntypedef Token *macro_handler_fn(Token *);\nstruct Macro {\n  // ...\n  macro_handler_fn *handler;\n};\n```\n\n`add_builtin(\"__FILE__\", file_macro)` registers a builtin. `expand_macro` checks:\n\n```c\n// Built-in dynamic macro application such as __LINE__\nif (m->handler) {\n  *rest = m->handler(tok);\n  (*rest)->next = tok->next;\n  return true;\n}\n```\n\nThe expansion paths for object-like and function-like macros also gain `Token->origin` painting:\n\n```c\nfor (Token *t = body; t->kind != TK_EOF; t = t->next)\n  t->origin = tok;\n```\n\n\u2014 so every token in an expanded body knows its source token, and `__FILE__`/`__LINE__` can walk back through nested expansions to the original source.\n\nCommit 190 (`dc01f94`, `__VA_ARGS__`) adds variadic function-like macros \u2014 `#define LOG(...) printf(__VA_ARGS__)`. The implementation:\n\n- `Macro` gains `bool is_variadic`.\n- `read_macro_params` recognizes `...` as the trailing parameter and sets `*is_variadic`.\n- `read_macro_arg_one` gains a `read_rest` flag that disables comma-stops for the variadic argument.\n- `read_macro_args` collects extra arguments (after fixed parameters) into a single `__VA_ARGS__` argument.\n\nThe variadic argument's `name` is hardcoded to `\"__VA_ARGS__\"`, which `find_arg` matches against in `subst`. So the body's `__VA_ARGS__` references look up the synthetic argument and substitute its (comma-separated) contents.\n\n```c\n#define M14(...) __VA_ARGS__\nASSERT(2, M14() 2);\nASSERT(5, M14(5));\n\n#define M14(x, ...) add6(1,2,x,__VA_ARGS__,6)\nASSERT(21, M14(3,4,5));   // add6(1,2,3,4,5,6) = 21\n```\n\nCommits 191 and 192 (`ba6b4b6` and `82ba010`, `__func__` and `__FUNCTION__`) land in `parse.c`, not `preprocess.c`:\n\n```c\npush_scope(\"__func__\")->var =\n  new_string_literal(fn->name, array_of(ty_char, strlen(fn->name) + 1));\n\n// [GNU] __FUNCTION__ is yet another name of __func__.\npush_scope(\"__FUNCTION__\")->var =\n  new_string_literal(fn->name, array_of(ty_char, strlen(fn->name) + 1));\n```\n\n\u2014 at function-entry, a variable scope named `__func__` (and the GNU-extension alias `__FUNCTION__`) is pushed, with a string-literal value containing the function's name. References to `__func__` in the function body resolve to that string literal through the ordinary variable-name lookup. Note this is *not* a preprocessor macro; the C standard specifies `__func__` as a *predefined identifier*, not a macro, and chibicc implements it that way. (If it were a macro, it would expand at preprocessing time, before the parser knows what function it's in.)\n\n### 17.5.4 \u2014 Adjacent-string concat; wide char; standard headers; `va_arg`\n\n> Commits `ab4f1e1` (concatenate strings), `7746e4e` (wide char), `7cbfd11` (standard headers), `5322ea8` (`va_arg`).\n\nCommit 193 (`ab4f1e1`, adjacent-string concatenation) handles `\"foo\" \"bar\"` \u2192 `\"foobar\"`. The post-expansion pass (called from the entry point of `preprocess`) walks the token list looking for runs of adjacent `TK_STR` tokens and merges them:\n\n```c\nstatic void join_adjacent_string_literals(Token *tok1) {\n  while (tok1->kind != TK_EOF) {\n    if (tok1->kind != TK_STR || tok1->next->kind != TK_STR) {\n      tok1 = tok1->next; continue;\n    }\n    // ... compute total length, allocate buffer, copy contents,\n    // mutate tok1 to be the joined token, splice past the consumed ones ...\n  }\n}\n```\n\n\u2014 the merged token's `ty` becomes a longer `array_of(ty->base, len)`, so `sizeof(\"foo\" \"bar\") = 7` (six characters plus terminator). Called at the very end of `preprocess`:\n\n```c\nToken *preprocess(Token *tok) {\n  init_macros();\n  tok = preprocess2(tok);\n  if (cond_incl) error_tok(cond_incl->tok, \"unterminated conditional directive\");\n  convert_keywords(tok);\n  join_adjacent_string_literals(tok);\n  return tok;\n}\n```\n\n\u2014 the order is: macro expansion, keyword conversion, string concatenation. The standard puts string concatenation in translation phase 6 (after preprocessing in phase 4 and lexical conversion in phase 5). Chibicc collapses phases but keeps the pipeline order.\n\nCommit 194 (`7746e4e`, wide character literal) recognizes `L'a'` as a character literal, treats it identically to `'a'` for now. The commit message: \"For now, L'' is equivalent to ''.\" A real wide-char implementation would produce a `wchar_t` (32-bit on Linux) value; chibicc punts. The test:\n\n```c\nASSERT(4, sizeof(L'\\0'));\nASSERT(97, L'a');\n```\n\n\u2014 `sizeof(L'\\0')` is `4` only because `'a'` itself has type `int` in C, and `int` is 4 bytes. The `L` prefix doesn't change the type yet. Errata candidate; the proper fix arrives with full wide-character work in Chapter 19.\n\nCommit 195 (`7cbfd11`) adds the `include/` directory with five small headers: `stdarg.h` (with `va_start`, `va_end`, the `__va_elem` and `va_list` typedefs, but *not* `va_arg` yet \u2014 that's the next commit), `stdbool.h` (defining `bool`/`true`/`false` to `_Bool`/`1`/`0`), `stddef.h` (`NULL`, `size_t`, `ptrdiff_t`), `stdalign.h` (`alignas`/`alignof` aliases for `_Alignas`/`_Alignof`), `float.h` (the `FLT_*`/`DBL_*` constants), and `stdnoreturn.h` (`noreturn` alias for `_Noreturn`). The headers are short \u2014 they're not full implementations, just enough to make `#include <stdarg.h>` work the way real C programs expect.\n\nThe Makefile gains `-Iinclude` for the test build. The chibicc-specific include path (the one rooted at `${argv0}/include`) finds these headers at runtime when the binary is invoked from its source directory.\n\nCommit 196 (`5322ea8`, `va_arg()`) \u2014 the long-awaited finishing move on Chapter 14's variadic-function magic. Pre-this-commit, accessing variadic arguments required the magic-name `__va_area__` plus hand-rolled arithmetic (or `self.py`'s regex substitution). Post-this-commit, `<stdarg.h>` provides:\n\n```c\nstatic void *__va_arg_gp(__va_elem *ap) {\n  void *r = (char *)ap->reg_save_area + ap->gp_offset;\n  ap->gp_offset += 8;\n  return r;\n}\n\nstatic void *__va_arg_fp(__va_elem *ap) {\n  void *r = (char *)ap->reg_save_area + ap->fp_offset;\n  ap->fp_offset += 8;\n  return r;\n}\n\n#define va_arg(ap, type)                        \\\n  ({                                            \\\n    int klass = __builtin_reg_class(type);      \\\n    *(type *)(klass == 0 ? __va_arg_gp(ap) :    \\\n              klass == 1 ? __va_arg_fp(ap) :    \\\n              __va_arg_mem(ap));                \\\n  })\n```\n\n\u2014 `__builtin_reg_class(type)` is a chibicc-specific intrinsic added in `parse.c`'s `primary` function:\n\n```c\nif (equal(tok, \"__builtin_reg_class\")) {\n  tok = skip(tok->next, \"(\");\n  Type *ty = typename(&tok, tok);\n  *rest = skip(tok, \")\");\n  if (is_integer(ty) || ty->kind == TY_PTR) return new_num(0, start);\n  if (is_flonum(ty)) return new_num(1, start);\n  return new_num(2, start);\n}\n```\n\n\u2014 it returns `0` for integer/pointer types (they go through `gp_offset`), `1` for floats/doubles (`fp_offset`), `2` for \"memory\" types (the `__va_arg_mem` path, which `1/0`s \u2014 divides-by-zero \u2014 because chibicc doesn't yet support struct-passing). The `va_arg(ap, type)` macro picks one of three offset-advancers at compile time based on the type-classifier intrinsic, advances the appropriate offset, and dereferences.\n\nThe result is that *user code* that uses `va_arg(ap, int)` no longer goes through `self.py`'s regex substitution \u2014 chibicc's preprocessor substitutes the macro's body, and the body's `__builtin_reg_class` is parsed by chibicc itself. The whole variadic surface is now real C, no special-cased Python in sight.\n\n#### Where the section leaves us\n\nEighteen commits, four sub-bundles. The preprocessor's surface is now feature-complete for ordinary C code: include paths and search rules, line continuation, `#error`, predefined macros, `__FILE__`/`__LINE__`/`__VA_ARGS__`/`__func__`, adjacent-string concatenation, wide char. Standard headers exist as small bundles. `va_arg()` retires the last hand-coded magic-name access pattern.\n\nThe chapter's standing notes:\n\n- **The `Token->origin` chain** is the chapter's quiet structural addition. Every expanded token knows its source. `__FILE__`/`__LINE__` walk the chain. After Chapter 17, `.loc` directives keep tracking the original source line, not the expanded line \u2014 which is what GDB needs to step through macro-using code.\n- **The host-cc-as-preprocessor pipeline collapses at commit 179.** The test Makefile no longer pipes through `cc -E`; chibicc's preprocessor handles `#include \"test.h\"` directly.\n- **Tests are still in C** (since Chapter 8 \u00a78.2). After commit 179, they're tests in C-with-real-#include.\n- **The `StringArray` type** has its first \u00a717 user (`include_paths`).\n- **The `unreachable()` macro** still has no readers in `preprocess.c` \u2014 the preprocessor's error paths use `error_tok` and `error` directly, never `unreachable()`.\n- **Errata candidates added in \u00a717.5:** `#error` doesn't print the directive's message text; `L''` is equivalent to `''`; `__va_arg_mem` divides by zero rather than failing cleanly; the `opt_S | opt_E` bitwise-`|` typo from \u00a717.1.5; the default include paths are Linux/glibc-specific.\n\nThe \u00a717.5 sub-bundles are *long* in commit count but *short* in interesting structural change \u2014 most of the work is feature-by-feature, with the `Token->origin` chain (commit 189) being the chapter's only deeply structural \u00a717.5 addition. The chapter mapping called this section's commits \"polish\" with good reason.\n\n---\n\n## 17.6 \u2014 Self-host\n\n> `git checkout 12a9e7506c092fcbab8852db85c3aebefc8a8c81` \u2014 *Self-host: including preprocessor, chibicc can compile itself*\n\nOne commit. The chapter's, and arguably the book's, punchline.\n\nThe Makefile change is small:\n\n```diff\n-stage2/%.o: chibicc self.py %.c\n+stage2/%.o: chibicc %.c\n \tmkdir -p stage2/test\n-\t./self.py chibicc.h $*.c > stage2/$*.c\n-\t./chibicc -c -o stage2/$*.o stage2/$*.c\n+\t./chibicc -c -o $(@D)/$*.o $*.c\n```\n\n\u2014 the stage-2 build no longer runs `self.py` to preprocess chibicc's source. It compiles chibicc's source directly, with chibicc, end-to-end. The Python regex preprocessor that has stood in for chibicc's missing C preprocessor since Chapter 16 \u00a716.1 is now obsolete, because chibicc's *own* preprocessor handles every directive `self.py` was working around.\n\nTwo consequences fall out immediately. First, `self.py` itself is *deleted* \u2014 127 lines of Python, gone. The script that prepended hand-typed `<stdio.h>`-equivalent declarations, stripped `\\<newline>` continuations, deleted `#include` and `#define` lines, substituted `bool`/`true`/`false`/`NULL`/`MIN`/`va_start`/`unreachable`, and otherwise massaged chibicc's source into something the pre-Chapter-17 chibicc could parse \u2014 that whole apparatus is no longer needed because the post-Chapter-17 chibicc handles real C.\n\nSecond, the test Makefile target gains `-Iinclude`:\n\n```diff\n-\t./chibicc -Itest -c -o test/$*.o test/$*.c\n+\t./chibicc -Iinclude -Itest -c -o test/$*.o test/$*.c\n```\n\n\u2014 pointing at the chibicc-shipped `include/` directory so test files that `#include <stdarg.h>` resolve to the bundled header. This is symmetric with what stage-2 already did; the regular test pipeline catches up.\n\nThe `make test-stage2` target now does what the Chapter 16 \u00a716.1 prose forecast it would: chibicc compiles chibicc, the resulting `stage2/chibicc` runs the test suite, every test passes. *Stage 2 is bug-equivalent to stage 1.* Which means stage 2 *is* a correct C compiler \u2014 because it was produced by a process that, at every step, used a tool we trust (the host `cc` for stage 1, then stage 1 chibicc for stage 2). And because stage 2 produces the same outputs as stage 1 on the test suite, we can trust that future stage-3 builds (compiling chibicc's source with stage-2 chibicc) would also work.\n\nThat's the self-hosting milestone: the compiler is its own host. The chibicc README's stated destination, the JP-book pedagogy's stated destination, the book's preface \u2014 all converge here. Commit 197 is the moment.\n\nA note on what self-hosting *isn't*. Chibicc compiling chibicc means chibicc is a *complete-enough* C compiler to handle one specific C program (its own source). It's not yet a *general* C compiler \u2014 there's plenty of standard C still missing (most prominently: variadic argument passing on the stack with more than 8 register-eligible args, struct passing by value, full bitfields, designated initializers, generic selection, etc.). Chapters 18 through 23 fill those in. But the *capability gap* between \"a C compiler good enough for chibicc's source\" and \"a C compiler good enough for typical real-world C\" is now an incremental matter: each remaining feature is a self-contained addition, not a structural rebuild. The chapter's progression has been: *fill in the gaps, not redesign anything.* That continues in Chapter 18 and beyond.\n\n#### Where we are\n\nChibicc compiles itself. `self.py` is gone. The chapter has done its work.\n\n---\n\n## Where the chapter leaves us\n\nForty commits, six sections, the longest single arc in chibicc's history. The chapter built a C preprocessor from a do-nothing seam to a full preprocessor with all the directives, macros, builtins, and supporting infrastructure that real-world C code uses, and ended with chibicc compiling itself.\n\n| Commit | Topic |\n|---|---|\n| `1e1ea39` | Do-nothing preprocessor (the seam). `convert_keywords` moves to post-`preprocess`. |\n| `146c7b3` | Null directive. `at_bol` flag on `Token`. `is_hash`, `preprocess2`. |\n| `d367510` | `#include \"...\"`. `File` struct. Per-file `.file`/`.loc` numbering. |\n| `ec149f6` | Skip extra tokens after `#include`. `warn_tok`, `skip_line`. |\n| `d138864` | `-E` flag. `print_tokens`. Absolute-path `#include`. |\n| `bf6ff92` | `#if`/`#endif`. `const_expr` exposed. `CondIncl` stack. `eval_const_expr`. |\n| `aa570f3` | Skip nested `#if` in skipped clause. Recursive `skip_cond_incl`. |\n| `c6e81d2` | `#else`. `IN_THEN`/`IN_ELSE` ctx. Two-tier skipper. |\n| `e7a1857` | `#elif`. `IN_ELIF` ctx. `included` flag governs first-match-wins. |\n| `97d33ad` | Object-like `#define`. `Macro` struct. `expand_macro`. |\n| `9ad60e4` | `#undef`. `deleted` flag on `Macro`. |\n| `2651448` | Expand macros in `#if`/`#elif` arguments. Recursive `preprocess2` call. |\n| `acce002` | No-rescan via hideset. `Hideset` struct. Prosser's algorithm cited. |\n| `1f80f58` | `#ifdef` and `#ifndef`. Skipper recognizes them as nesting-starters. |\n| `dec3b3f` | Zero-arity function-like `#define`. `is_objlike` flag. `has_space` on `Token`. |\n| `b9ad3e4` | Multi-arity function-like `#define`. `MacroParam`/`MacroArg`. `subst`. |\n| `dd4306c` | Empty arguments. (Test-only.) |\n| `c7d7ce0` | Parenthesized arguments. `level` counter in `read_macro_arg_one`. |\n| `1313fc6` | Function-like no-rescan. `hideset_intersection` at the closing paren. |\n| `8f6f792` | Stringizing `#x`. `quote_string`, `new_str_token`, `stringize`. |\n| `8f561ae` | Pasting `a##b`. `paste`. `##` becomes a punctuator. |\n| `769b5a0` | Tests use chibicc's preprocessor. `cc -E` retires from the test pipeline. |\n| `5cb2f89` | `defined()`. `read_const_expr`. |\n| `a8d76ad` | Identifier-to-zero in constexpr. C standard 6.10.1p4. |\n| `8075582` | Whitespace and newline preservation through expansion. |\n| `b33fe0e` | Line continuation. `remove_backslash_newline` in tokenizer. |\n| `d85fc4f` | `#include <...>`. `is_dquote` flag. `join_tokens` reuses stringizer. |\n| `a1dd621` | `-I<dir>`. `include_paths` `StringArray`. |\n| `a939a7a` | Default include paths. `add_default_include_paths`. |\n| `e7fdc2e` | `#error`. Three lines. (Errata: doesn't print the message.) |\n| `5f5a850` | Predefined macros. 41 entries: `__STDC__`, `__x86_64__`, `__linux__`, etc. |\n| `6f17071` | `__FILE__`/`__LINE__`. `Token->origin` chain. `macro_handler_fn`. |\n| `dc01f94` | `__VA_ARGS__`. `is_variadic`. Variadic argument `name = \"__VA_ARGS__\"`. |\n| `ba6b4b6` | `__func__`. Implemented in `parse.c`, not `preprocess.c`. |\n| `82ba010` | `__FUNCTION__` (GNU alias for `__func__`). |\n| `ab4f1e1` | Adjacent-string concatenation. `join_adjacent_string_literals`. |\n| `7746e4e` | Wide character literal. `L''` \u2261 `''`. (Errata candidate.) |\n| `7cbfd11` | `stdarg.h`/`stdbool.h`/`stddef.h`/`stdalign.h`/`float.h`/`stdnoreturn.h`. |\n| `5322ea8` | `va_arg()` as a real macro. `__builtin_reg_class`. Magic name retired user-side. |\n| `12a9e75` | **Self-host.** `self.py` deleted. `chibicc compiles chibicc`. |\n\nThe structural moves carried forward.\n\nThe first is the *hideset-on-Token* mechanism. Chibicc's preprocessor implements Prosser's algorithm directly: a per-token set of macro names that have already been expanded \"into\" this token, gated on at expansion time, painted on at expansion time, intersected at function-like-macro close-parens to handle invocations whose start and end tokens have different paint. The algorithm terminates on `#define X X`, `#define f() f()`, mutually-recursive macros, and any other shape the C language permits, because no token is ever expanded by the same macro twice. The data structure is a small linked list; the operations are union, intersection, contains. Forty lines of `preprocess.c` implement the whole machinery.\n\nThe second is the *token-list-rewriting model*. The preprocessor takes a `Token *` and returns a `Token *`. It never reads from text after the initial `tokenize_file`, never writes to text except for `-E`. Directives splice (`#include`), delete regions (`#if 0`), or define rewrite rules (`#define`). Macros are rewrite rules whose application is gated by hidesets. The model is *uniform* \u2014 there's no special-case code path that bypasses the rewriter. It's the same shape modern preprocessors use; chibicc's implementation is small enough to fit in one `preprocess.c` of about 900 lines.\n\nThe third is the *Token->origin chain*. Every expanded token knows its source. The chain is what makes `__FILE__` and `__LINE__` work *inside* macros (they walk the chain to find the original source location). It's also what makes error reporting accurate when an error happens in expanded code: `error_tok(t, ...)` reports `t->file->name`, but if `t` is an expansion result, the chain can be walked to find where the *user* wrote the offending construct, not where the macro body wrote it. Chibicc's error messages don't fully exploit this yet \u2014 they generally point at expansion results \u2014 but the infrastructure is there for future use.\n\nThe fourth is *chibicc compiling chibicc*. The chapter's destination. By commit 197, the stage-2 Makefile target produces a chibicc binary that's been compiled by the stage-1 chibicc, with no `self.py` interposition, with chibicc's own preprocessor handling all `#include` and `#define` work in chibicc's source. The binary passes the test suite. Self-hosting is real.\n\nThe chapter's other notes worth tracking forward.\n\n- **The cc1-vs-driver split** is unchanged. The preprocessor lives inside cc1, not as a third process. The `-E` flag short-circuits `cc1()` after `preprocess()`.\n- **The eval-quartet duplication** is real but principled: AST-level constexpr (`eval`/`eval2`/`eval_rval`/`eval_double`) handles initializers, array sizes, case labels, bitfield widths; token-level constexpr (`eval_const_expr` \u2192 `const_expr` \u2192 `eval`) handles `#if`/`#elif`. They share `eval` itself but go through different parsers.\n- **The `Initializer` tree** is unchanged.\n- **The `Relocation` mechanism** is unchanged.\n- **The `is_static` default in `new_gvar`** is unchanged.\n- **The `is_definition` flag on `Obj`** is unchanged.\n- **The `is_unsigned` flag on `Type`** is unchanged.\n- **The `__va_area__` magic name** survives in chibicc's source, but the user-side hook (`va_start`, `va_arg`, `va_end`) is now expressed as ordinary `<stdarg.h>` macros. The magic-name access pattern is still emitted by `va_start`, but `va_arg(ap, type)` no longer needs hand-rolling.\n- **The register-save-area layout** is unchanged.\n- **The argreg integer/FP split** is unchanged.\n- **Canonicalization-at-parse-time count is unchanged at eight.** The preprocessor runs *before* parse, so its work is upstream of canonicalization.\n- **Pre-factor-before-feature count ticks to eight.** \u00a717.1's do-nothing preprocessor is the seam-first move; the next 39 commits fill it in. (Chapter 16 was at seven; this chapter's first commit makes eight.)\n- **The fourth namespace (labels)** is unchanged.\n- **The `is_typename` predicate** is unchanged.\n- **The VarAttr channel** is unchanged.\n- **The `unreachable()` macro** has no readers in `preprocess.c` after Chapter 17 \u2014 the preprocessor's error paths use `error_tok` and `error` directly.\n- **Per-token line numbers** (Chapter 8 \u00a78.3) survive macro expansion via the `Token->origin` chain. `.loc` directives reference the original source line. GDB-debuggable output (Chapter 8 \u00a78.4) keeps working through macro-using code.\n- **Tests in C** (Chapter 8 \u00a78.2) are now C with real `#include` (commit 179). The driver tests are still shell.\n- **The `Obj->tok` field** still has no readers.\n- **The `Type->name_pos` field** is unchanged.\n- **The `>>` codegen quirk** is unchanged.\n- **psABI conformance thread is unchanged at nine.** The preprocessor doesn't touch ABI.\n- **Errata catalog grows by five small items in Chapter 17:** `#error` doesn't print the directive's message text (commit 187); `L''` is equivalent to `''` (commit 194); `__va_arg_mem` divides by zero rather than aborting cleanly (commit 196); the `opt_S | opt_E` bitwise-`|` typo in `main.c` (commit 162); the default include paths are Linux/glibc-specific (commit 186).\n\nForward references for Chapter 18 (the full ABI, commits 198\u2013220):\n\n- **More than 6 integer args, more than 8 FP args**: Chapter 5 \u00a75.4 and Chapter 15 \u00a715.6 introduced silent miscompilation when a function has more args than registers can pass. Chapter 18 fills in stack-passed args and parameters, closing the silent-miscompile errata items.\n- **Struct-by-value**: Chapter 12 introduced struct types and member access, but passing a struct as a function argument or returning one as a value still doesn't work. Chapter 18 lands the SysV ABI's struct-passing rules \u2014 the eightbyte classification, the `MEMORY`/`INTEGER`/`SSE` class assignments, the register-and-stack hybrid passing.\n- **Variadic register-spilling**: Chapter 14 \u00a714.1 set up the register-save area for variadic functions, but variadic functions whose fixed parameters spill onto the stack don't work yet. Chapter 18 fixes this with the `va_copy` work and full register/stack mixed handling.\n- **Bitfields**: Chapter 12 introduced struct members but not bitfields. Chapter 18 lands `int x:3;` syntax, the storage-allocation rules, the access codegen, and the related restrictions.\n- **Pp-numbers**: Chapter 18 retokenizes preprocessor numbers (the C-spec category that lets `0x1.5p-3` lex as a single token before number parsing) \u2014 affecting how `#define X 0xff` and `#define X 1e10` work.\n- **`-D`/`-U`**: Command-line macro definition, finally.\n\nChapter 18 is somewhere between Chapter 16's eight commits and Chapter 17's forty \u2014 twenty-three commits, the chapter mapping forecasts. Chapter 17's sweeping preprocessor work is behind us; Chapter 18 returns to the ABI grind, filling in what Chapters 5, 12, 14, and 15 left silently broken at the edges. The book's path from here is steady ABI-conformance until Chapter 19's Unicode arc and Chapters 20\u201323's GCC-extensions and standard-library rounding-out.\n\nFor now, Chapter 17 closes with chibicc compiling chibicc. The book's stated destination has been reached.\nCHAPTERMARKER\nwc -w /Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md",
  "description": "Append \u00a717.4\u201317.6 and closing recap"
}


---

## 🤖 Claude

Chapter 17 complete at 17,134 words — within the 15,000–20,000 target. Now writing the session 018 README and Ch 18 handoff.

---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "completed",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "completed",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "completed",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "in_progress",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "pending",
      "activeForm": "Writing handoff for session 019"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/018-chapter-17-draft && ls /Users/cell/claude/chibicc-book/docs/sessions/018-chapter-17-draft",
  "description": "Make session 018 dir"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/018-chapter-17-draft/README.md",
  "content": "# Session 018 \u2014 Chapter 17 draft\n\n**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002\u2013017).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 017 delivered Ch 16 (The compiler driver, eight commits, ~9,688 words). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 17 covers commits 158\u2013197: forty commits, the longest single arc in chibicc's history, building the C preprocessor from a do-nothing seam to a full-feature preprocessor and ending in self-hosting at commit 197.\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~17,134 words. Within the 15,000\u201320,000-word handoff forecast, near the middle. Forty commits can't be compressed below ~15K without dropping substance; the chapter ran to its natural length without padding. The \u00a717.4 function-like-macro arc and the \u00a717.3 hideset interlude are the two single biggest stretches; both wanted full treatment.\n- **Section structure:** 6 sections from 40 commits, exactly as the handoff proposed. \u00a717.1 (5 commits, the skeleton) gets per-commit subsections (17.1.1 through 17.1.5) since each of those five commits introduces something distinct enough to warrant its own header. \u00a717.2 (4 commits, conditionals) has per-commit subsections likewise. \u00a717.3 (5 commits, object-like macros) has per-commit subsections plus a *concept interlude* on the no-rescan rule between \u00a717.3.4 and \u00a717.3.5. \u00a717.4 (7 commits, function-like macros) has per-commit subsections. \u00a717.5 (18 commits, polish) groups into four sub-bundles by topic (17.5.1 tests/defined/whitespace, 17.5.2 line-continuation/include-paths, 17.5.3 #error/predefined/__FILE__/__VA_ARGS__/__func__, 17.5.4 string-concat/wide-char/headers/va_arg) with each sub-bundle covering 4\u20136 commits. \u00a717.6 (1 commit, self-host) gets its own focused section as the chapter's punchline.\n- **Concept interlude on the no-rescan rule** lands inside \u00a717.3.4. The handoff defaulted to yes, conditional on whether \u00a717.4's prose surfaces the need; reading the \u00a717.3.4 implementation, the *why* of the per-token paint mechanism \u2014 versus a recursion-depth limit, or a global exclusion list \u2014 wanted explicit framing. The interlude is short (~400 words) and titled \"A concept interlude \u2014 why the paint must terminate.\" It walks the per-token-vs-global question, the set-vs-single-name question, and names *Prosser's algorithm* / *blue paint algorithm* explicitly. The handoff defaulted to \"no\" on naming Prosser by name, but the \u00a717.3.4 code itself updates its comment block to credit \"Dave Prossor\" [sic \u2014 Rui's typo] in commit 1313fc6, which makes the citation natural.\n- **No second concept interlude** on what self-hosting means. \u00a717.6 prose handles it inline. Pulling self-hosting out into its own interlude would have duplicated content with \u00a717.6's ~700 words.\n- **\u00a717.1 names the do-nothing-skeleton-first approach as deliberate** \u2014 handoff acceptance criterion. \u00a717.1.1's prose calls it \"the chapter's pre-factor entry\" and explicitly increments the count from seven to eight.\n- **\u00a717.2 names the token-level evaluator as separate from the AST-level eval quartet** \u2014 handoff acceptance criterion. \u00a717.2.1's prose explicitly calls out the duplication: \"chibicc has, post this commit, *two* constant-expression evaluation paths.\" The note is repeated in the section closer.\n- **\u00a717.3 walks the no-rescan rule and the hideset** \u2014 handoff acceptance criterion. \u00a717.3.4 walks the rule with two worked examples (`#define M2 M2 + 3` and the mutually-recursive `M4`/`M5` case). The concept interlude reinforces the *why*.\n- **\u00a717.4 walks function-like macros, the second no-rescan rule, stringizing, pasting** \u2014 handoff acceptance criterion. \u00a717.4.5 walks the hideset *intersection* rule with a worked example (`dbl`/`M10` mutual recursion). \u00a717.4.6 walks stringizing's `quote_string` + tokenize-the-result mechanism. \u00a717.4.7 walks pasting's `paste` helper and the standard's \"macros not expanded across `##` operands\" rule.\n- **\u00a717.5 covers all eighteen polish commits** \u2014 handoff acceptance criterion. Sub-grouped exactly as the handoff proposed (four sub-bundles, 4\u20136 commits each).\n- **\u00a717.6 names self-hosting as the chapter's punchline** \u2014 handoff acceptance criterion. The \u00a717.6 prose calls it \"the chapter's, and arguably the book's, punchline\" and walks the Makefile change. References \u00a716.1's stage-2 forecast explicitly.\n- **Commit 179 (`use chibicc's preprocessor for all tests`) is named as the moment the host `cc -E` pipeline retires** \u2014 handoff acceptance criterion. Lands in \u00a717.5.1.\n- **Commit 197 (self-host) is named as the moment chibicc compiles itself end-to-end without `self.py`** \u2014 handoff acceptance criterion. \u00a717.6 carries this.\n- **One-table recap** at the chapter close, forty rows. Resisted multi-table-by-section, since the table is intended as a single index of features for readers to find a commit.\n\n### Interpretive calls\n\n1. **The do-nothing-preprocessor counts as a pre-factor.** The handoff said \"Ch 17 likely to add: the `preprocess()` skeleton (\u00a717.1) is the pre-factor for everything that follows.\" Decision: yes. Pre-factor count goes from seven to eight.\n2. **The eval-quartet duplication is named twice** \u2014 once in \u00a717.2.1's prose (where the duplication first appears) and once in the \u00a717.2 closer (where it's flagged as \"principled\" with the small-duplications-are-tolerated framing from `quotes-rui.md`).\n3. **Prosser's algorithm gets named.** The handoff defaulted to no, but commit 1313fc6's diff itself updates the source comment to credit \"Dave Prossor\" [sic]. Naming it in the chapter is faithful to Rui's own source comments. The naming is restrained (the algorithm is named once in the chapter overview, once in \u00a717.3.4, and once in the closer; the metaphor \"blue paint\" is mentioned twice).\n4. **The hideset section is the longest single subsection in the chapter.** \u00a717.3.4 plus the concept interlude run to ~1,800 words. This is consistent with the handoff's prediction that the no-rescan rule \"is the most subtle thing in the chapter and probably wants the concept interlude.\"\n5. **The \u00a717.4 worked examples are explicit.** Both the object-like recursion (`M2 = M2 + 3`) and the function-like recursion (`dbl`/`M10`) get step-by-step walks with hideset states tracked at each step. This is unusual depth for chibicc-book; the chapter mapping flagged the no-rescan rule as the chapter's most subtle, and the prose follows.\n6. **The `__func__` / `__FUNCTION__` commits land in `parse.c`, not `preprocess.c`** \u2014 and the prose calls this out explicitly in \u00a717.5.3, naming `__func__` as a *predefined identifier* not a macro. This matters because a reader expecting `__func__` to be a preprocessor feature would be looking in the wrong file; the chapter's prose corrects the expectation.\n7. **The `va_arg(ap, type)` macro from commit 196 gets the \"magic name retired user-side\" framing** \u2014 exactly as the handoff predicted. \u00a717.5.4 names it as \"the long-awaited finishing move on Chapter 14's variadic-function magic.\" The `__builtin_reg_class` intrinsic gets walked because it's the bridge between the macro body and the codegen.\n8. **The cpp-as-separate-binary forecast gets the explicit \"wrong-prediction\" treatment.** \u00a717.0 (the chapter overview) names it: \"There's no separate `cpp` binary, no second fork-exec round-trip beyond Chapter 16's driver-cc1 split.\" This is the handoff's forecast about the \u00a716.1 prose being wrong; the chapter-17 prose handles it.\n9. **Errata candidates added in \u00a717.5 are catalogued in the chapter closer.** Five small items: `#error` doesn't print the message text, `L''` \u2261 `''`, `__va_arg_mem` divides by zero, the `opt_S | opt_E` bitwise-`|` typo, and the Linux/glibc-specific default include paths.\n10. **The chapter doesn't add a fifth namespace, doesn't grow `VarAttr`, doesn't extend the cast table, doesn't add a canonicalization-at-parse-time arm, doesn't add psABI conformance corrections.** All of those forecasts from the handoff held.\n\n### Voice / structure inherited from Ch 1\u201316\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For sub-section openers (within \u00a717.1, \u00a717.2, \u00a717.3, \u00a717.4), each commit's subsection has its own opener.\n- \u00a717.5's sub-bundle openers list the commits in the bundle with their subjects compact-style (one line).\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers (and per-subsection \"Where we are\" closers in \u00a717.1).\n- One-table recap at the chapter close.\n- One concept interlude (in \u00a717.3.4) titled and offset.\n\n### Three careful avoidances\n\n- **Did not invent a \"history of macro expansion algorithms\" interlude.** The hideset machinery has a real history (Prosser's 1986 document, the C committee's adoption of it, alternative approaches in pre-standard preprocessors), but that history is a detour. The chapter cites Prosser's name and the wiki link Rui himself includes; it doesn't try to compete with the standards-history literature.\n- **Did not invent a Linkers-and-Loaders-style chapter on dynamic linking.** \u00a717.5.2's `-I`/default-paths discussion stays in chibicc's lane: what chibicc does, why it's brittle, what the alternative would cost.\n- **Did not over-explain the difference between `__func__` and `__FUNCTION__`.** The two-line GNU-extension commit gets ~50 words. The C standard's intent (predefined identifier rather than macro, named-after-the-current-function rather than computed-by-substitution) is summarized once and not belabored.\n\n### Date-vs-position note\n\nThe forty commits scatter across calendar time: March 2020 (commits 163, 164, 165, 166, 167, 168, 170, 171, 172, 173, 174, 175, 187, 191, 192, 193, 195, 197) interleaved with August 2020 (158, 161, 162, 169, 178, 188, 189) interleaved with September 2020 (160, 184, 185) interleaved with April 2020 (177, 186, 190, 194, 196). The chapter follows `main` order without remark, the same as Chapters 7\u201316. The \u00a717.0 overview prose names the date scatter as \"even more scattered by calendar date than Chapter 16\" \u2014 Rui clearly worked on the preprocessor in non-linear bursts, and `main` order isn't `chronological` order.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **The hideset on Token** is the chapter's signature data structure. Per-token, paint-on-expansion, gate-on-expansion, intersect at function-like-macro close-paren. Chapter 18 doesn't touch the preprocessor; the hideset machinery is stable post-Ch 17.\n- **The Token->origin chain** is the chapter's other major Token-struct addition. Every expanded token knows its source. `__FILE__`/`__LINE__` walk it. Error reporting *could* walk it (chibicc doesn't yet) \u2014 future-error-message work is a candidate for using `origin`.\n- **The eval-quartet duplication** stays. AST-level constexpr (`eval`/`eval2`/`eval_rval`/`eval_double`) for parser uses; token-level (`eval_const_expr` \u2192 `const_expr`) for `#if`/`#elif`. Both share `eval` itself.\n- **The host-cc-as-preprocessor pipeline retired at commit 179.** The test Makefile no longer pipes through `cc -E`. The host `cc` is still invoked for the test harness's final link with `test/common`, but the preprocessor side is fully chibicc.\n- **`self.py` is deleted.** 127 lines of Python, gone with commit 197. The compiler's input pipeline is now entirely chibicc.\n- **`stage2` is bug-equivalent to `stage1`.** The test suite passes against stage 2, which means stage 2 *is* a correct chibicc.\n- **Default include paths** \u2014 Linux/glibc-specific, hardcoded to three system paths plus `${argv0}/include`. Errata candidate, lower priority.\n- **`#error` doesn't print the message text** \u2014 three-line implementation in commit 187. Errata candidate.\n- **`L''` is `''`** \u2014 wide-char punted in commit 194. Errata candidate; full wide-char arrives in Chapter 19.\n- **The standard headers** in `include/` are minimal-but-functional: `stdarg.h` with `va_start`/`va_end`/`va_arg` plus the `__va_elem`/`va_list` typedefs; `stdbool.h` with `bool`/`true`/`false`/`__bool_true_false_are_defined`; `stddef.h` with `NULL`/`size_t`/`ptrdiff_t`/`wchar_t`/`max_align_t`; `stdalign.h` with `alignas`/`alignof` aliases; `float.h` with the `FLT_*`/`DBL_*`/`LDBL_*` constants; `stdnoreturn.h` with `noreturn`. Together: ~95 lines.\n- **`__builtin_reg_class`** is the new chibicc-specific intrinsic from commit 196. Used only by `<stdarg.h>`'s `va_arg(ap, type)`. Not documented.\n- **The `Macro->handler` field** is the dynamic-builtin-macro hook. Used by `__FILE__` and `__LINE__`. Could be reused by future builtins (`__DATE__`, `__TIME__`, `__COUNTER__` arrive in Chapter 19).\n- **Pre-factor-before-feature count is now eight.** \u00a717.1.1's do-nothing preprocessor is the latest entry.\n- **Canonicalization-at-parse-time count is unchanged at eight.** The preprocessor runs before parse.\n- **psABI conformance thread is unchanged at nine.** Preprocessor doesn't touch ABI.\n- **The cc1-vs-driver split** is unchanged. The preprocessor lives inside cc1.\n- **The `Initializer` tree, `Relocation` mechanism, `is_static` default, `is_definition` flag, `is_unsigned` flag, register-save-area layout, argreg integer/FP split, fourth namespace (labels), `is_typename` predicate, VarAttr channel, cast table, anonymous-global pattern (no new uses), `Member->idx` field (no new uses), `is_flexible` flag, `copy_struct_type` (no new uses), `MIN`/`MAX` macros (no new uses), `is_numeric` predicate, `Obj->tok` field (still no readers), `Type->name_pos` field (no new uses), `>>` codegen quirk, the `add_type` rule for `ND_STMT_EXPR` (errata candidate), the hex-escape silent truncation (errata candidate), the redeclaration-in-same-scope check missing for variables/tags/typedef-names/labels (four errata), `f()` and `f(void)` accepted as identical (errata), empty brace initializer (chibicc-specific extension), `.bss` as third assembly section, `.align` for every global, \"more than 6 integer args silently miscompiles\" / \"more than 8 FP args silently miscompiles\" (errata, both), the variadic-FP-call wrongness (errata), the `fp_offset = fp * 8 + 48` non-conforming stride (errata), `long double` \u2261 `double` (errata), default-argument-promotion gap for chars/shorts (errata), float literals inlined as integer-immediate-bit-cast, the cast/compound-literal disambiguator, the cast table at 10\u00d710, driver brittleness in `find_libpath`/`find_gcc_libpath` (errata), the link command's hardcoded distro list (errata), `Node->funcname` being gone, `call *%rax` being uniform across all calls, the `StringArray` type, `atexit(cleanup)` for tempfile disposal, the `run_subprocess` helper** \u2014 all unchanged from Ch 16.\n\n## Exit state\n\n- `chapters/17-a-preprocessor-from-scratch.md` drafted, ~17,134 words.\n- Session 018 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 019 (Chapter 18 \u2014 The full ABI, commits 198\u2013220, ~23 commits).\n- CLAUDE.md status note will be updated to \"Ch 17 drafted\".\n"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "completed",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "completed",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "completed",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "completed",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "in_progress",
      "activeForm": "Writing handoff for session 019"
    }
  ]
}


---

## 🤖 Claude

Now let me grab the commit list for Ch 18 to write a substantive handoff.

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && git log --oneline --reverse 12a9e75..main | head -25",
  "description": "List commits 198+ for Ch 18"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline --reverse 12a9e75..main | sed -n '23p'",
  "description": "Confirm commit 220 is c3075b3"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/018-chapter-17-draft/HANDOFF.md",
  "content": "# Handoff: Ch 17 done \u2192 proceed to Ch 18\n\n**For:** the next claude session.\n**From:** session 018.\n**Status:** Ch 17 drafted (~17,134 words, forty commits, the C preprocessor from a do-nothing seam to self-hosting). Continue autonomously to Ch 18 (The full ABI, commits 198\u2013220 \u2014 twenty-three commits covering stack-passed args/parameters, struct-by-value, variadic-with-more-than-6-fixed-params, `va_copy`, function-deref no-op, pp-numbers, `-D`/`-U`, bitfields, buffered output, ignored flags, `-Wall`-clean self-build, 16-byte array alignment, implicit `return 0` in `main`, anonymous struct/union). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/018-chapter-17-draft/README.md`](README.md)** \u2014 what session 018 did, including the six-section structure (per-commit subsections in \u00a717.1 through \u00a717.4, four sub-bundles in \u00a717.5, single-section \u00a717.6), the concept interlude on the no-rescan rule (lands in \u00a717.3.4, ~400 words, named \"A concept interlude \u2014 why the paint must terminate\", names Prosser's algorithm and the blue-paint metaphor), the \"chibicc compiles chibicc\" framing for \u00a717.6, the `cpp-as-separate-binary forecast was wrong` note resolved, the five new errata candidates added to the catalog, the pre-factor count incremented to eight, the canonicalization-at-parse-time count unchanged at eight, the psABI conformance count unchanged at nine.\n2. **[`docs/sessions/017-chapter-16-draft/HANDOFF.md`](../017-chapter-16-draft/HANDOFF.md)** \u2014 the previous handoff. Standing notes still apply with Ch 17 updates folded in (see \u00a718 README for the running list).\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`17-a-preprocessor-from-scratch.md`](../../../chapters/17-a-preprocessor-from-scratch.md)** \u2014 the seventeen chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 18 = commits 198\u2013220 (23 commits, scoped to \"the full ABI\" plus a few unrelated polish commits).\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes. Ch 18's commits are mostly bugfixes-and-completions; less likely to have especially apt commit-message material.\n6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** \u2014 JP TOC; the SysV ABI's struct-passing rules might be a candidate for a concept interlude in \u00a718.2 (struct-by-value), but default-no \u2014 the algorithm is intricate but its *why* is \"the SysV ABI says so,\" which doesn't repay an interlude.\n\n## Chapter 18 scope\n\n**Title (working):** *The full ABI*.\n**Commits:** 198\u2013220 in chronological order on `main`. **Twenty-three commits** \u2014 back to a manageable size after Ch 17's forty-commit arc.\n**Concept interlude:** Likely no. The chapter's substance is \"fill in the gaps the ABI work has been hiding under.\" A concept interlude on *the SysV AMD64 ABI's eightbyte classification algorithm* is tempting \u2014 it's a clever piece of design \u2014 but the algorithm only lands in chibicc as the function-by-function classification implementation, which \u00a718.2 prose can carry inline. Default no, conditional on whether \u00a718.2's prose feels overstuffed.\n\n| # | Hash | Subject |\n|---|---|---|\n| 198 | `b29f052` | Support passed-on-stack arguments |\n| 199 | `9021f7f` | Support passed-on-stack parameters |\n| 200 | `5e0f8c4` | Allow struct parameter |\n| 201 | `d63b1f4` | Allow struct argument |\n| 202 | `c72df1c` | Allow to call a fucntion returning a struct |\n| 203 | `d7bad96` | Allow to define a function returning a struct |\n| 204 | `b6d3cd0` | Allow variadic function to take more than 6 parameters |\n| 205 | `603de50` | Add va_copy() |\n| 206 | `e0b5da3` | Dereferencing a function shouldn't do anything |\n| 207 | `3f2c2d5` | Tokenize numeric tokens as pp-numbers |\n| 208 | `fc69f5c` | Add -D option |\n| 209 | `be8b6f6` | Add -U option |\n| 210 | `cc852fe` | Add bitfield |\n| 211 | `441a89b` | Support global struct bitfield initializer |\n| 212 | `54c2b3b` | Handle op=-style assignments to bitfields |\n| 213 | `17ea802` | Handle zero-width bitfield member |\n| 214 | `c302a96` | Do not allow to obtain an address of a bitfield |\n| 215 | `2bdc6b8` | Write to an in-memory buffer before writing to an actual output file |\n| 216 | `b1fdddf` | Ignore -O, -W and -g and other flags |\n| 217 | `2c91da5` | Turn on -Wall compiler flag and fix compiler warnings |\n| 218 | `5257ee0` | Make an array of at least 16 bytes long to have alignment of at least 16 bytes |\n| 219 | `9c36dd7` | Make \"main\" to implicitly return 0 |\n| 220 | `c3075b3` | Add anonymous struct and union |\n\nTwenty-three commits. The natural section grouping:\n\n- **\u00a718.1 \u2014 Stack-passed args and parameters** (commits 198\u2013199). Two commits. Closes the \"more than 6 integer args silently miscompiles\" errata from Ch 5 \u00a75.4 and the \"more than 8 FP args silently miscompiles\" errata from Ch 15 \u00a715.6. Codegen learns to push extra args onto the stack and pop them after the call; parameter handling learns to read them from the caller's stack frame.\n- **\u00a718.2 \u2014 Struct-by-value** (commits 200\u2013203). Four commits. Struct as parameter, struct as argument, struct as return type (caller side and callee side). This is the SysV AMD64 ABI's *eightbyte classification* \u2014 a struct \u226416 bytes splits into one or two 8-byte chunks, each classified as INTEGER or SSE based on its members; chunks pass via registers (`%rdi`/`%rsi`/`%xmm0`/`%xmm1`/etc.) or memory based on classification. Larger structs pass by hidden pointer. Returns work the same way (with a hidden first parameter for memory-class returns).\n- **\u00a718.3 \u2014 Variadic with stack-resident fixed parameters; va_copy** (commits 204\u2013205). Two commits. Closes the \"variadic functions whose fixed parameters spill onto the stack don't work yet\" gap from Ch 14. `va_copy(dst, src)` lets the user duplicate a `va_list`, which is a one-line copy because `va_list` is `__va_elem[1]` (an array, so assignment is illegal but pointer-style copy via `*dst = *src` works).\n- **\u00a718.4 \u2014 Small completion commits** (commits 206\u2013209). Four commits. *Dereferencing a function* (`(*fp)(x)` should be a no-op when `fp` is `int(*)(int)` \u2014 it stays a function-pointer). *Pp-numbers* (preprocessor numbers, the C-spec lexical category that lets `0x1.5p-3` lex as one token). *`-D` option* (command-line `#define`). *`-U` option* (command-line `#undef`).\n- **\u00a718.5 \u2014 Bitfields** (commits 210\u2013214). Five commits. The bitfield arc: `int x:3;` syntax in struct declarations; storage-allocation rules (consecutive bitfields share storage; alignment forces a new unit when needed); the read/write codegen (mask-and-shift); global initializers; op-assign (`x.bf += 1` requires loading, modifying, storing back); zero-width bitfield (forces alignment-to-next-storage-unit); address-of restriction (a bitfield doesn't have an address \u2014 `&s.bf` is illegal).\n- **\u00a718.6 \u2014 Polish and tail** (commits 215\u2013220). Six commits. Buffered output (write the assembly into an `open_memstream` buffer first, then dump). Ignoring `-O`/`-W`/`-g`/`-MD`/`-pipe` and other GCC flags (with a TODO to actually honor them). `-Wall` self-build (chibicc compiles cleanly under `gcc -Wall`, which catches uninitialized variables and other host-cc-detectable bugs). 16-byte alignment for arrays >= 16 bytes (an x86-64 ABI nicety that helps SSE intrinsics). `main` implicitly returns 0 (per the C standard's special case). Anonymous struct/union (`struct { struct { int a, b; }; int c; }` lets `s.a` reach the inner anonymous member).\n\nThat's six sections from twenty-three commits. **Target chapter length: ~10,000\u201313,000 words.** Likely closer to 12K \u2014 the bitfield arc is detailed and \u00a718.2's eightbyte classification is involved.\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in b29f052 9021f7f 5e0f8c4 d63b1f4 c72df1c d7bad96 b6d3cd0 603de50 e0b5da3 3f2c2d5 fc69f5c be8b6f6 cc852fe 441a89b 54c2b3b 17ea802 c302a96 2bdc6b8 b1fdddf 2c91da5 5257ee0 9c36dd7 c3075b3; do echo \"===== $h =====\"; git show --stat $h | head -10; done` to scan all 23 diffs.\n2. Read each commit. Pay particular attention to:\n   - **\u00a718.1's stack-passed-args codegen (198\u2013199):** The push-extras-then-call sequence is x86-64's standard: fixed-args go in registers, overflow-args go on the stack in left-to-right order (so caller and callee agree which is which). The callee reads them from positive offsets relative to `%rbp`. Note the alignment: x86-64 requires `%rsp` 16-byte-aligned at the call instruction.\n   - **\u00a718.2's eightbyte classification (200\u2013203):** Read carefully. The SysV ABI's algorithm is: walk the struct's members; for each 8-byte chunk, take the *worst* class among the members in that chunk (memory > integer > SSE); if any chunk is memory, the whole struct is memory. The implementation lives in a small helper function (probably `classify_param` or similar). The 16-byte limit is fundamental \u2014 at 17 bytes, the whole thing goes to memory.\n   - **\u00a718.3's `va_copy` (205):** `va_copy(dst, src)` is `*(dst) = *(src)` \u2014 but in chibicc's `<stdarg.h>`, `va_list` is `__va_elem[1]`, an array, so `dst` and `src` are pointers. A real C compiler implements `va_copy` as a builtin; chibicc gets away with a one-line macro because of the array-as-pointer-decay equivalence.\n   - **\u00a718.4's pp-numbers (207):** The C spec defines a lexical category called *preprocessor numbers* that's broader than the ordinary integer/float syntax. A pp-number can be `0x1.5p-3`, `1e+10`, `1.5e-3.foo` \u2014 anything starting with a digit (or `.` followed by a digit) and continuing with digits, letters, periods, and `+`/`-` after `e`/`E`/`p`/`P`. The preprocessor passes pp-numbers through; the parser later attempts to interpret them as actual numbers. This matters for `#define X 1e10` and `0xff.5` and similar.\n   - **\u00a718.5's bitfields (210\u2013214):** The most code-heavy section after \u00a718.2. Read `Member`'s new fields (`is_bitfield`, `bit_offset`, `bit_width`). The codegen for read needs `shr`/`and`-mask. The codegen for write needs read-modify-write (load the unit, clear the bits, OR in the new value, store back). Op-assign (`s.bf += 1`) is read-modify-write twice. Zero-width bitfield is a parser-side rule \u2014 `int :0;` forces alignment to the next storage unit, no member is created. Address-of restriction: the parser rejects `&s.bf` for bitfield members.\n   - **\u00a718.6's `-Wall`-clean build (217):** This commit is interesting because it shows what host-cc warnings catch in chibicc's source: probably unused variables, possibly some uninitialized-on-some-paths cases, possibly some signed/unsigned compares. Worth a paragraph on what kinds of issues `-Wall` finds.\n3. Read the destination state at `c3075b3` for `parse.c`, `codegen.c`, `type.c`, `chibicc.h`, `main.c`. The struct-by-value codegen will be the most invasive change.\n4. Draft `chapters/18-the-full-abi.md`. Likely 10,000\u201313,000 words. Six sections.\n5. Write `docs/sessions/019-chapter-18-draft/README.md`.\n6. Write `HANDOFF.md` for session 020 (Chapter 19 \u2014 Unicode and designated initializers, commits 221\u2013244).\n\n## Voice / structure rules\n\nSame as Ch 1\u201317:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For bundled sub-sections, each commit gets its own opener.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present tense for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 twenty-three rows, single table is fine.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The \u00a718.2 struct-by-value codegen and the \u00a718.5 bitfield codegen will want larger code blocks.\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.\n- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **\u00a718.1's stack-passed-args closes errata items.** The \"more than 6 integer args silently miscompiles\" (Ch 5 \u00a75.4) and \"more than 8 FP args silently miscompiles\" (Ch 15 \u00a715.6) errata candidates are *closed by* this section. Name the closure explicitly.\n- **\u00a718.2's eightbyte classification is the ABI's central trick.** It's tempting to say \"the ABI says structs \u226416 bytes go in registers\" \u2014 that's almost right but oversimplifies. The classification is per-eightbyte-chunk, and a struct can have one chunk in a register and another in memory; or a chunk that's INTEGER in `%rdi` and a chunk that's SSE in `%xmm0`. Walk it carefully.\n- **\u00a718.3's `va_copy` is one line of macro.** Don't oversell. The interesting story is *why* it's one line (the array-decay equivalence), not the implementation.\n- **\u00a718.4's pp-numbers lexer change is invasive.** It moves number tokenization from the integer/float branches into a unified pp-number branch, then the parser interprets the pp-number's text. The pre-Ch 18 number lexer is a *parser*; the post-Ch 18 number lexer is a *recognizer*. The semantic change is small but it touches a lot of code paths.\n- **\u00a718.5's bitfield arc is the most subtle in Ch 18.** Read each commit; don't blur the five into one walk. The op-assign commit (212) is the hardest \u2014 the read-modify-write sequence has to be ordered correctly and shouldn't double-evaluate the lhs. The zero-width commit (213) has a small alignment-rule subtlety. The address-of commit (214) is a one-line restriction in the parser.\n- **\u00a718.6's `-Wall`-clean build is interesting but not deep.** A sentence per kind of warning fixed is plenty.\n- **The buffered-output commit (215)** changes how `codegen` writes to its output file: it goes through an `open_memstream` buffer, then dumps. The reason is to avoid partial-output situations on errors. Don't mistake it for performance work.\n- **The implicit-`return 0`-in-`main` commit (219)** lands in `parse.c`'s `function`. `main` (and only `main`) gets a synthetic `return 0;` appended if the function falls off the end. This is the C standard's special case for `main`; chibicc implements it minimally.\n- **The anonymous-struct/union commit (220)** is the \u00a718 chapter's only struct-system extension. `Member` gains a way to inline an unnamed struct's members into the parent's member-list, so member lookup walks transparent layers.\n\n## Standing notes worth tracking across sessions\n\n- **The hideset on Token** is unchanged. Ch 18 doesn't touch the preprocessor.\n- **The Token->origin chain** is unchanged. Ch 18 doesn't touch the preprocessor.\n- **The eval-quartet duplication** is unchanged.\n- **The cc1-vs-driver split** is unchanged. Ch 18 doesn't touch the driver.\n- **The `Initializer` tree** likely changes in \u00a718.5 (bitfield initializers).\n- **The local-vs-global split** is stable.\n- **The `Relocation` mechanism** likely changes in \u00a718.5 (global bitfield initializers).\n- **The anonymous-global pattern** (no new uses) \u2014 Ch 18 unlikely to use.\n- **The `is_static` default in `new_gvar`** \u2014 unchanged.\n- **The `is_definition` flag on `Obj`** \u2014 unchanged.\n- **The `is_unsigned` flag on `Type`** \u2014 unchanged.\n- **The `__va_area__` magic name** \u2014 unchanged in chibicc's source.\n- **The register-save-area layout** \u2014 likely changes in \u00a718.3 to handle stack-resident fixed parameters.\n- **The argreg integer/FP split** \u2014 unchanged.\n- **The `Member->idx` field** \u2014 likely gets new bitfield-related siblings in \u00a718.5.\n- **The `is_flexible` flag** \u2014 unchanged.\n- **`copy_struct_type`** \u2014 unchanged.\n- **`MIN`/`MAX` macros** \u2014 unchanged.\n- **`is_numeric` predicate** \u2014 unchanged.\n- **Canonicalization-at-parse-time count is at eight.** Ch 17 added zero. Ch 18 *might* add one in \u00a718.4 (pp-numbers parsed from text at parse time) \u2014 verify while drafting.\n- **Pre-factor-before-feature count is at eight.** Ch 17 added one (the do-nothing preprocessor). Ch 18 might add: \u00a718.5's bitfield work likely splits a `Member` extension before the bitfield feature lands.\n- **The fourth namespace (labels)** is unchanged.\n- **The `is_typename` predicate** is unchanged.\n- **The VarAttr channel** has four fields. Ch 18 unlikely to grow it.\n- **The `ND_NULL_EXPR` seed-pattern** \u2014 no new uses since Ch 12.\n- **The `rep stosb` pattern** \u2014 no new uses since Ch 12.\n- **The `unreachable()` macro** \u2014 Ch 18 likely to add callers in struct-by-value codegen.\n- **Per-token line numbers** (Ch 8 \u00a78.3) used by `.loc` and error-tok throughout. Preserved through preprocessing as of Ch 17.\n- **GDB-debuggable output** (Ch 8 \u00a78.4) \u2014 already taken for granted.\n- **Tests are in C** (Ch 8 \u00a78.2). Driver tests in shell (`test/driver.sh`). The host-cc-as-preprocessor pipeline is *gone* (Ch 17 \u00a717.5.1).\n- **The `Obj->tok` field** added in Ch 14 \u00a714.11 still has no readers.\n- **The `Type->name_pos` field** (Ch 14 \u00a714.11) \u2014 no new uses since.\n- **The `>>` codegen quirk** (Ch 11 \u00a711.13) \u2014 partially repaired.\n- **The `add_type` rule for `ND_STMT_EXPR`** (Ch 7 \u00a77.5) \u2014 errata candidate.\n- **The hex-escape silent truncation** (Ch 7 \u00a77.4) \u2014 errata candidate.\n- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels \u2014 four errata candidates.\n- **`f()` and `f(void)` are accepted as identical** \u2014 errata candidate.\n- **Empty brace initializer (`int x[3] = {};`)** \u2014 chibicc-specific extension.\n- **`.bss` is the third assembly section.**\n- **`.align`** is emitted for every global. \u00a718.6's commit 218 adds 16-byte alignment for arrays >=16 bytes; check if the rule extends to other types.\n- **The \"more than 6 integer args silently miscompiles\"** \u2014 *will be closed* by \u00a718.1.\n- **The \"more than 8 FP args silently miscompiles\"** (Ch 15 \u00a715.6) \u2014 *will be closed* by \u00a718.1.\n- **The `mov $0, %rax`** \u2014 closed loop; still pessimistic; plus the variadic-FP-call wrongness (Ch 15 \u00a715.6) \u2014 *will be closed* by \u00a718.3.\n- **psABI conformance thread:** stands at nine after Ch 17. Ch 18 will land *several* psABI corrections \u2014 stack-passed args/parameters (\u00a718.1, two more), struct-by-value (\u00a718.2, four more), variadic-with-stack-fixed-parameters (\u00a718.3, one more), bitfields (\u00a718.5, one more for layout). New count should reach approximately fifteen by Ch 18 close. Verify exact tally while drafting.\n- **The `fp_offset = fp * 8 + 48` non-conforming stride** (Ch 15 \u00a715.9) \u2014 errata candidate, *might be closed* by \u00a718.3 \u2014 verify.\n- **`long double` is `double`** (Ch 15 \u00a715.11) \u2014 errata candidate.\n- **The default-argument-promotion gap for chars and shorts** (Ch 15 \u00a715.8) \u2014 errata candidate.\n- **Float literals are inlined as integer-immediate-bit-cast** (Ch 15 \u00a715.1).\n- **Ch 1 errata list** unchanged.\n- **The cast/compound-literal disambiguator** (Ch 13 \u00a713.4) \u2014 same lookahead pattern.\n- **The cast table is 10\u00d710.** Possibly grows in \u00a718.2 if struct-by-value introduces a new cast cell.\n- **Driver brittleness** \u2014 unchanged.\n- **The link command's hardcoded distro list** (Ch 16 \u00a716.6) \u2014 errata candidate, lower priority.\n- **`Node->funcname` is gone** (Ch 16 \u00a716.2). Function calls identify the callee by `lhs`.\n- **`call *%rax` is uniform across all calls** (Ch 16 \u00a716.2). No fast path for direct named calls. Ch 18 \u00a718.1's stack-args work doesn't change this.\n- **The `StringArray` type** (Ch 16 \u00a716.4) \u2014 used by `include_paths` (Ch 17 \u00a717.5.2). Ch 18 might add new users for `-D`/`-U` macro lists.\n- **`atexit(cleanup)` for tempfile disposal** \u2014 unchanged.\n- **The `run_subprocess` helper** \u2014 unchanged.\n- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `L''` \u2261 `''`; `__va_arg_mem` divides by zero; `opt_S | opt_E` bitwise-`|` typo; default include paths Linux/glibc-specific.\n- **`self.py` is gone** (Ch 17 \u00a717.6).\n- **Stage-2 build** is end-to-end chibicc as of Ch 17 \u00a717.6.\n- **Chibicc compiles itself** as of commit 197.\n\n## Acceptance criteria for Ch 18\n\n- [ ] `chapters/18-the-full-abi.md` exists, end-to-end readable.\n- [ ] All twenty-three commits covered, grouped into ~6 sections.\n- [ ] \u00a718.1 names the closure of \"more than 6 integer args silently miscompiles\" and \"more than 8 FP args silently miscompiles\" errata items.\n- [ ] \u00a718.2 walks the SysV AMD64 eightbyte classification \u2014 the per-chunk classification, the worst-class merge rule, the 16-byte size cutoff to memory.\n- [ ] \u00a718.3 walks `va_copy` and the closure of the variadic-stack-fixed-parameters gap.\n- [ ] \u00a718.4 walks pp-numbers as a lexical category and the lexer-vs-parser split.\n- [ ] \u00a718.5 walks each of the five bitfield commits with no skipping. The op-assign and zero-width commits each get their own subsection.\n- [ ] \u00a718.6 walks the buffered-output commit, the ignored-flags commit, the `-Wall`-clean self-build, the 16-byte array alignment, the implicit-`return 0`-in-`main`, and the anonymous struct/union commit.\n- [ ] Voice matches Ch 1\u201317.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references checked against `chapter-mapping.md`.\n- [ ] psABI conformance thread count updated to its new value (approximately fifteen).\n- [ ] `docs/sessions/019-chapter-18-draft/README.md` written.\n- [ ] `HANDOFF.md` written for session 020 (Chapter 19 \u2014 Unicode and designated initializers, commits 221\u2013244).\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/018-chapter-17-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/018-chapter-17-draft/README.md   (what session 018 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md\n13. chapters/11-all-the-operators.md\n14. chapters/12-initializers.md\n15. chapters/13-linkage.md\n16. chapters/14-variadics-signedness-qualifiers.md\n17. chapters/15-floating-point.md\n18. chapters/16-the-compiler-driver.md\n19. chapters/17-a-preprocessor-from-scratch.md     (most recent chapter)\n20. research/commits/chapter-mapping.md            (confirms Ch 18 scope)\n21. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 18 (The full ABI, commits 198\u2013220) per the steps in\nthe handoff. Twenty-three commits, six sections proposed in the handoff.\nThe bitfield arc (\u00a718.5, five commits) is the chapter's longest single\nstretch; the struct-by-value SysV ABI work (\u00a718.2, four commits) is the\ndeepest. Likely no concept interlude. End-of-session: write your session\ndir under docs/sessions/019-chapter-18-draft/ with a README and a\nHANDOFF for session 020 (Chapter 19 \u2014 Unicode and designated\ninitializers, commits 221\u2013244).\n```\n"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "completed",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "completed",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "completed",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "completed",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "completed",
      "activeForm": "Writing handoff for session 019"
    },
    {
      "content": "Update CLAUDE.md status note",
      "status": "in_progress",
      "activeForm": "Updating CLAUDE.md status"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018). Continuing autonomously per user direction."
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
  "old_string": "Ch 16 drafted (session 017). Continuing autonomously per user direction.",
  "new_string": "Ch 16 drafted (session 017), Ch 17 drafted (session 018). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 017 README and Ch 16",
      "status": "completed",
      "activeForm": "Reading session 017 README and Ch 16"
    },
    {
      "content": "Scan all 40 commits for Ch 17",
      "status": "completed",
      "activeForm": "Scanning all 40 commits for Ch 17"
    },
    {
      "content": "Read destination state at 12a9e75 (preprocess.c, etc.)",
      "status": "completed",
      "activeForm": "Reading destination state at 12a9e75"
    },
    {
      "content": "Draft chapters/17-a-preprocessor-from-scratch.md",
      "status": "completed",
      "activeForm": "Drafting chapter 17"
    },
    {
      "content": "Write docs/sessions/018-chapter-17-draft/README.md",
      "status": "completed",
      "activeForm": "Writing session 018 README"
    },
    {
      "content": "Write HANDOFF.md for session 019 (Ch 18)",
      "status": "completed",
      "activeForm": "Writing handoff for session 019"
    },
    {
      "content": "Update CLAUDE.md status note",
      "status": "completed",
      "activeForm": "Updating CLAUDE.md status"
    }
  ]
}


---

## 🤖 Claude

Session 018 complete.

**Delivered**
- `chapters/17-a-preprocessor-from-scratch.md` — ~17,134 words covering all forty commits (158–197), six sections (skeleton / conditionals / object-like + hideset / function-like + `#`/`##` / polish in four sub-bundles / self-host), one concept interlude on Prosser's no-rescan algorithm in §17.3.4, ending with chibicc compiling itself
- `docs/sessions/018-chapter-17-draft/README.md` — drafting decisions, interpretive calls, voice notes, errata catalog updates
- `docs/sessions/018-chapter-17-draft/HANDOFF.md` — primes session 019 for Chapter 18 (The full ABI, commits 198–220, twenty-three commits)
- `CLAUDE.md` — status updated to "Ch 17 drafted (session 018)"

Handoff path: `docs/sessions/018-chapter-17-draft/HANDOFF.md`

---

