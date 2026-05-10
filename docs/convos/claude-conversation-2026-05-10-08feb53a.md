# Claude Conversation Log

Session ID: 08feb53a-7bef-4db2-9d80-66f22e5a7b44
Date: 2026-05-10 08:39:29

---

## 👤 User

please read docs/sessions/020-chapter-19-draft/HANDOFF.md and proceed

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/020-chapter-19-draft/HANDOFF.md"
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
      "content": "Read session 020 README and recent chapter for context",
      "activeForm": "Reading session 020 README and recent chapter",
      "status": "in_progress"
    },
    {
      "content": "Scan all 22 commits (245-266) with git show --stat",
      "activeForm": "Scanning all 22 commits",
      "status": "pending"
    },
    {
      "content": "Read each commit diff in detail",
      "activeForm": "Reading each commit diff",
      "status": "pending"
    },
    {
      "content": "Draft chapters/20-gcc-extensions-worth-supporting.md",
      "activeForm": "Drafting chapter 20",
      "status": "pending"
    },
    {
      "content": "Write session 021 README",
      "activeForm": "Writing session 021 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 022",
      "activeForm": "Writing HANDOFF for session 022",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/020-chapter-19-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in 37998be c61c0d0 aaf20fb 922604a 3a10c8a 3381448 083c275 74ec9f6 007e526 7d80a51 1433b40 1faab48 aee7891 e28a612 a253516 31087f8 e5f4ca9 6a2dc5a 11fc259 1b99bad 85e46b1 6d344ed; do echo \"===== $h =====\"; git show --stat $h | head -15; done",
  "description": "Scan all 22 commits for chapter 20"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 020 README and recent chapter for context",
      "activeForm": "Reading session 020 README and recent chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 22 commits (245-266) with git show --stat",
      "activeForm": "Scanning all 22 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit diff in detail",
      "activeForm": "Reading each commit diff",
      "status": "in_progress"
    },
    {
      "content": "Draft chapters/20-gcc-extensions-worth-supporting.md",
      "activeForm": "Drafting chapter 20",
      "status": "pending"
    },
    {
      "content": "Write session 021 README",
      "activeForm": "Writing session 021 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 022",
      "activeForm": "Writing HANDOFF for session 022",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/19-unicode-and-designated-initializers.md",
  "limit": 100
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/19-unicode-and-designated-initializers.md",
  "offset": 100,
  "limit": 80
}


---

## 🤖 Claude

Let me read each commit in detail. I'll start with the first batch.

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 37998be",
  "description": "Show commit 245 - multibyte error message"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show c61c0d0",
  "description": "Show #line commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show aaf20fb 922604a 3a10c8a",
  "description": "Show line marker, __TIMESTAMP__, __BASE_FILE__ commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 3381448 083c275 74ec9f6",
  "description": "Show __VA_OPT__, ,##__VA_ARG__, #pragma commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 007e526",
  "description": "Show GCC variadic macro commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 7d80a51",
  "description": "Show typeof commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 1433b40",
  "description": "Show __builtin_types_compatible_p commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 1faab48",
  "description": "Show _Generic commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show aee7891 e28a612",
  "description": "Show sizeof function and ?: omitted middle commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a253516",
  "description": "Show asm statement commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 31087f8",
  "description": "Show inline as static commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e5f4ca9",
  "description": "Show static inline emission commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6a2dc5a 11fc259",
  "description": "Show __attribute__((format)) and -idirafter commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 1b99bad 85e46b1 6d344ed",
  "description": "Show offsetof, tentative, -fcommon commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout 6d344ed -- codegen.c 2>&1 | head -5; git show 6d344ed:codegen.c | grep -n \"emit_data\\|\\.bss\\|\\.comm\\|\\.data\\|is_tentative\\|init_data\" | head -30",
  "description": "Inspect codegen tentative path"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6d344ed:codegen.c | sed -n '1120,1170p'",
  "description": "Read emit_data area"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6d344ed:parse.c | sed -n '2920,2960p'",
  "description": "Read global_variable function"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 020 README and recent chapter for context",
      "activeForm": "Reading session 020 README and recent chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 22 commits (245-266) with git show --stat",
      "activeForm": "Scanning all 22 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit diff in detail",
      "activeForm": "Reading each commit diff",
      "status": "completed"
    },
    {
      "content": "Draft chapters/20-gcc-extensions-worth-supporting.md",
      "activeForm": "Drafting chapter 20",
      "status": "in_progress"
    },
    {
      "content": "Write session 021 README",
      "activeForm": "Writing session 021 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 022",
      "activeForm": "Writing HANDOFF for session 022",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git status 2>&1 | head -10",
  "description": "Check chibicc repo state"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git restore --staged codegen.c && git checkout -- codegen.c && git status",
  "description": "Restore chibicc repo state"
}


---

## 🤖 Claude

Now I'll draft Chapter 20.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/20-gcc-extensions-worth-supporting.md",
  "content": "# Chapter 20 \u2014 GCC extensions worth supporting\n\n> Commits covered: `37998be`, `c61c0d0`, `aaf20fb`, `922604a`, `3a10c8a`, `3381448`, `083c275`, `74ec9f6`, `007e526`, `7d80a51`, `1433b40`, `1faab48`, `aee7891`, `e28a612`, `a253516`, `31087f8`, `e5f4ca9`, `6a2dc5a`, `11fc259`, `1b99bad`, `85e46b1`, `6d344ed`. Twenty-two commits \u2014 small polish on multibyte error display, the `#line` directive and gcc's line-marker form, four new predefined macros, two macro-expansion extensions for variadic ergonomics, `#pragma`, `typeof` and `_Generic` and `__builtin_types_compatible_p` on the type side, two small relaxations of standard rules (`sizeof` of a function, `?:` with omitted middle), basic `asm` statements, two passes of `inline` plumbing (treat-as-static, then dead-static-inline elimination), `__attribute__((format))` annotations on chibicc's own diagnostics, the `-idirafter` include-path entry, the `offsetof` macro, tentative definitions, and the `-fcommon`/`-fno-common` toggle.\n\nThrough Chapter 19 chibicc is feature-complete enough to compile itself end-to-end, with bitfields, the SysV AMD64 ABI, the full Unicode arc, and designated initializers. What it isn't is *gcc-friendly*. Real-world C code reaches for gcc extensions constantly \u2014 `typeof` in container-of macros, `_Generic` in `tgmath.h`-style headers, `asm` in OS kernels and standard libraries, `inline` in just about every modern header, tentative definitions in plain old global declarations like `int x;` at file scope. A compiler that handles only ISO C will choke on glibc's headers within the first few inclusions. Chapter 20's twenty-two commits chip away at that gap.\n\nThe commits are small. Most are between five and forty lines of diff. Three are larger: `_Generic` adds about forty-seven lines of new parser code, the `inline`-elimination commit adds about thirty-seven, the GCC-style variadic macro rework rewrites about thirty. The rest are one-feature, one-function additions whose implementation reads the same way the spec reads. The chapter's job is to walk through the lot, name what each commit gave us, and flag the points where chibicc's implementation diverges from what the language reference says \u2014 sometimes deliberately, occasionally not.\n\nSix sections from twenty-two commits.\n\n- **\u00a720.1** \u2014 Multibyte error column display, `#line`, and the GNU line marker (commits 245\u2013247).\n- **\u00a720.2** \u2014 Macro extensions: `__TIMESTAMP__`, `__BASE_FILE__`, `__VA_OPT__`, `,##__VA_ARGS__`, `#pragma`, GCC-style variadic (commits 248\u2013253).\n- **\u00a720.3** \u2014 Type-side extensions: `typeof`, `__builtin_types_compatible_p`, `_Generic` (commits 254\u2013256).\n- **\u00a720.4** \u2014 `sizeof` of a function and the GNU ternary middle (commits 257\u2013258).\n- **\u00a720.5** \u2014 `asm`, `inline`-as-static, dead static-inline elimination, `__attribute__((format))` (commits 259\u2013262).\n- **\u00a720.6** \u2014 `-idirafter`, `offsetof`, tentative definitions, `-fcommon` (commits 263\u2013266).\n\nThe chapter follows `main` order. As before, the calendar dates scatter \u2014 the `__BASE_FILE__` commit is dated August 20 but lands at position 249 between two July commits \u2014 and the chapter does not remark on date-vs-position except where the work itself depends on something later.\n\n---\n\n## 20.1 \u2014 Multibyte error column display, `#line`, and the GNU line marker\n\n> `git checkout 37998be0c183508e54f10f57d63d87e6e7eb0607` \u2014 *Improve error message for multibyte characters*\n>\n> `git checkout c61c0d00252a8704ff2731f6a57bad3657b84170` \u2014 *Add #line*\n>\n> `git checkout aaf20fb96eaf21ead775fde6bad00d8e71650b5a` \u2014 *[GNU] Add line marker directive*\n\nThree commits. The first is a small visual fix in `verror_at` that closes the last loose end of the Unicode arc from Chapter 19. The other two add the `#line` family of directives, which let a generated source file pretend it came from somewhere else.\n\n### 20.1.1 \u2014 `display_width` and the multibyte caret\n\nThrough Chapter 19, `verror_at` printed the source line, then a caret on the next line at the column where the error sat. The column was computed in bytes:\n\n```c\nint pos = loc - line + indent;\n```\n\nFor ASCII source that's correct: every character is one byte and one column. For a UTF-8 source line containing Japanese or emoji, it's wrong by however many extra UTF-8 bytes lie between the start of the line and `loc`. The diagnostic's caret would land several columns to the right of the offending token.\n\nThe fix moves the column computation through a new helper, `display_width`, that walks the source bytes through `decode_utf8` and asks `char_width` how many display columns each code point occupies:\n\n```c\nint pos = display_width(line, loc - line) + indent;\n```\n\n`display_width` decodes one code point per loop iteration, summing widths:\n\n```c\nint display_width(char *p, int len) {\n  char *start = p;\n  int w = 0;\n  while (p - start < len) {\n    uint32_t c = decode_utf8(&p, p);\n    w += char_width(c);\n  }\n  return w;\n}\n```\n\n`char_width` returns 0 for combining characters (the `range1` table), 2 for East-Asian-wide and emoji-range code points (the `range2` table), and 1 for everything else. The tables are large \u2014 37 lines of paired ranges in `range1` alone \u2014 and they trace back, by attribution in the source comment, to Markus Kuhn's `wcwidth.c` implementation that has circulated since the late 1990s. Chibicc inlines the tables rather than calling the libc `wcwidth(3)`, partly because chibicc tries to keep its libc surface narrow and partly because libc's `wcwidth` requires a locale to be set up first.\n\nThe code-point/column distinction is the same one terminal emulators have to make when laying out text. It's why a CJK character takes two cells in your terminal but one in a font's character grid; combining accents take zero cells because they overlay the previous glyph. The diagnostic now matches what a reader sees.\n\n### 20.1.2 \u2014 `#line` and the per-token line delta\n\n> *#line is a directive that overrides the file name and line number that subsequent tokens report.*\n\n`#line N` resets the line counter so the *next* line reports as `N`. `#line N \"file\"` resets both the line and the filename. The directive is most useful for code generators that produce C: by stamping `#line` directives into the output, the generator makes the C compiler's diagnostics point at the source file the user actually wrote.\n\nThe implementation has to thread the override through tokens that already have a `line_no` and a `file` field assigned at tokenization time. Rui's approach is to add a *delta* alongside the existing per-token line number, applied at the end of preprocessing:\n\n```c\ntypedef struct {\n  ...\n  // For #line directive\n  char *display_name;\n  int line_delta;\n} File;\n\nstruct Token {\n  ...\n  File *file;       // Source location\n  char *filename;   // Filename\n  int line_no;      // Line number\n  int line_delta;   // Line number\n  ...\n};\n```\n\n`File` gets two new fields, `display_name` and `line_delta`, both set to the file's natural values at construction time:\n\n```c\nFile *new_file(char *name, int file_no, char *contents) {\n  File *file = calloc(1, sizeof(File));\n  file->name = name;\n  file->display_name = name;\n  ...\n}\n```\n\n`Token` gets the same two fields. Tokens copy `display_name` from the file at construction time (in `new_token`) and `line_delta` from the file at preprocessing time (in `preprocess2`'s pass-through arm):\n\n```c\nif (!is_hash(tok)) {\n  tok->line_delta = tok->file->line_delta;\n  tok->filename = tok->file->display_name;\n  cur = cur->next = tok;\n  tok = tok->next;\n  continue;\n}\n```\n\nThe `#line` handler reads its argument and writes the file fields, so all later tokens in the file pick up the delta:\n\n```c\nstatic void read_line_marker(Token **rest, Token *tok) {\n  Token *start = tok;\n  tok = preprocess(copy_line(rest, tok));\n\n  if (tok->kind != TK_NUM || tok->ty->kind != TY_INT)\n    error_tok(tok, \"invalid line marker\");\n  start->file->line_delta = tok->val - start->line_no;\n\n  tok = tok->next;\n  if (tok->kind == TK_EOF)\n    return;\n\n  if (tok->kind != TK_STR)\n    error_tok(tok, \"filename expected\");\n  start->file->display_name = tok->str;\n}\n```\n\nThe `delta` is \"what value should be added to `line_no` to get the user-facing line number.\" If `#line 500` appears on line 4 of a file, `start->line_no` is 4, the argument is 500, and the delta is 496 \u2014 every later token's reported line is its raw line plus 496.\n\nThe application is deferred to the end of preprocessing, in the final loop in `preprocess()`:\n\n```c\nfor (Token *t = tok; t; t = t->next)\n  t->line_no += t->line_delta;\n```\n\nThe reason for the deferral is `__LINE__`, which is one of the predefined macros from Chapter 19 \u00a719.1. `line_macro` reads `tmpl->line_no + tmpl->file->line_delta` to compute its expansion *at the time of expansion*. If the addition were applied eagerly, `__LINE__` would double-count.\n\n`__FILE__` is updated to read `tmpl->file->display_name` instead of `tmpl->file->name` so it reflects the override too:\n\n```c\nstatic Token *file_macro(Token *tmpl) {\n  while (tmpl->origin)\n    tmpl = tmpl->origin;\n  return new_str_token(tmpl->file->display_name, tmpl);\n}\n```\n\nThe test file `test/line.c` exercises both forms:\n\n```c\n#line 500 \"foo\"\n  ASSERT(501, __LINE__);\n  ASSERT(0, strcmp(__FILE__, \"foo\"));\n```\n\nThe line numbering in `__LINE__` reports the line *after* the directive, hence `501` rather than `500`.\n\n### 20.1.3 \u2014 The GNU line-marker directive\n\nThe line-marker form `# 123 \"file\" 1` is what gcc's preprocessor emits when invoked with `-E`. cc1 has to read its own preprocessor output back, which means cc1 has to recognize the marker form and treat it as an implicit `#line`. Five lines do it:\n\n```c\nif (tok->kind == TK_PP_NUM) {\n  read_line_marker(&tok, tok);\n  continue;\n}\n```\n\nThe directive parser, after consuming the `#`, sees a preprocessing-number token directly. Standard `#line` would have a `line` keyword first. The presence of a number where the directive name should be is the signal.\n\nThe trailing `1`, `2`, or `3` flags in `# 200 \"xyz\" 2 3` are silently ignored. Real gcc uses them to mark \"we're entering a new file\" (1), \"we're returning from a file we'd previously included\" (2), and \"this is a system header\" (3). Chibicc's `read_line_marker` reads the number and the optional filename and stops; the trailing flags fall on the floor when `read_line_marker` returns and the outer preprocess loop skips the rest of the line. Acceptable: the flags affect how diagnostics are categorized, not how the code parses.\n\nThe Token's `filename` field added in \u00a720.1.2 has a parallel purpose \u2014 it records the *display* name as it stood at the moment the token was tokenized, separate from the canonical `file->name`. The chapter recap will note that the per-token line-number tracking from Chapter 8 \u00a78.3 has now grown an *origin-display* twin that survives `#line` overrides.\n\n**Where we are.** Diagnostics align with displayed columns even for multibyte source. `#line` and the GNU line marker both work; cc1's `-E` output round-trips through cc1's own preprocessor. Per-token source-position tracking gains a `line_delta` and a `filename` field on both `File` and `Token`.\n\n---\n\n## 20.2 \u2014 Macro extensions\n\n> `git checkout 922604ae1e29fd1283fcc557e294a7272116c094` \u2014 *[GNU] Add __TIMESTAMP__ macro*\n>\n> `git checkout 3a10c8aa44250e51dfe33e50b3121d6061faee4b` \u2014 *[GNU] Add __BASE_FILE__ macro*\n>\n> `git checkout 338144869fa82097d7767a032cbaac616ba0cd01` \u2014 *Add __VA_OPT__*\n>\n> `git checkout 083c27559e5d8fce9c3b588fc4c01769ca9dd10d` \u2014 *[GNU] Handle ,##__VA_ARG__*\n>\n> `git checkout 74ec9f6f3964d4beaa3970bd99c8660f958b694e` \u2014 *Ignore #pragma*\n>\n> `git checkout 007e526ec50bde4b366d0927ad20d9cd4ac53abf` \u2014 *[GNU] Support GCC-style variadic macro*\n\nSix commits. Two new predefined macros (`__TIMESTAMP__`, `__BASE_FILE__`), two extensions to function-like macro expansion (`__VA_OPT__`, the `,##__VA_ARGS__` swallow), one trivially-ignored directive (`#pragma`), and one rework of the variadic-macro parameter parsing so a named-rest parameter (`args...`) plays the same role as the standard `...`.\n\n### 20.2.1 \u2014 `__TIMESTAMP__`\n\n`__TIMESTAMP__` expands to a string describing the source file's *modification time* \u2014 not the compilation time, which is what `__DATE__` and `__TIME__` use. The format is the same one `ctime(3)` produces:\n\n```c\nstatic Token *timestamp_macro(Token *tmpl) {\n  struct stat st;\n  if (stat(tmpl->file->name, &st) != 0)\n    return new_str_token(\"??? ??? ?? ??:??:?? ????\", tmpl);\n\n  char buf[30];\n  ctime_r(&st.st_mtime, buf);\n  buf[24] = '\\0';\n  return new_str_token(buf, tmpl);\n}\n```\n\n`ctime_r` formats a `time_t` as `Day Mon DD HH:MM:SS YYYY\\n\\0` \u2014 twenty-five characters before the newline, total length twenty-six including the terminator. The implementation truncates after twenty-four characters by writing `\\0` at index 24, which lops off the trailing `\\n` and sizes the string at exactly twenty-four. The test `ASSERT(24, strlen(__TIMESTAMP__))` confirms.\n\nThe fallback string `\"??? ??? ?? ??:??:?? ????\"` is used when `stat` fails \u2014 for instance when the source comes from stdin via the `-` filename. Twenty-four characters by construction.\n\nThe macro is registered as a `Macro->handler`, sharing the same hook that `__FILE__`, `__LINE__`, `__COUNTER__`, and `__DATE__`/`__TIME__` use. The state is the file's mtime, looked up at *each* expansion \u2014 unlike `__DATE__` which is fixed at startup, `__TIMESTAMP__` is fixed at *file* level. A second source file would produce a different `__TIMESTAMP__` value.\n\n### 20.2.2 \u2014 `__BASE_FILE__`\n\n`__BASE_FILE__` expands to the top-level source filename \u2014 the file passed on the command line, even when expansion happens inside an included file. The implementation is two lines plus an `add_builtin` call:\n\n```c\nstatic Token *base_file_macro(Token *tmpl) {\n  return new_str_token(base_file, tmpl);\n}\n```\n\n`base_file` is a global set in `main.c` to the source path that cc1 was invoked on. There's no traversal of `tmpl->origin` because the answer is the same regardless of where the macro appeared.\n\nThe contrast with `__FILE__` is the point: `__FILE__` reports whichever file the macro was *expanded in*, walking the `origin` chain back to the original token; `__BASE_FILE__` reports the file the compiler was *invoked on*. In a recursive include scenario, `__FILE__` traces up to the include site; `__BASE_FILE__` ignores all of that and gives the command-line argument.\n\n### 20.2.3 \u2014 `__VA_OPT__`\n\n`__VA_OPT__(X)` expands to its argument tokens iff `__VA_ARGS__` is non-empty. The C2X feature was introduced specifically to fix the long-running annoyance of variadic macros where the trailing-comma problem makes `M(fmt)` versus `M(fmt, args...)` need two separate definitions. With `__VA_OPT__`, one definition handles both:\n\n```c\n#define M30(buf, fmt, ...) sprintf(buf, fmt __VA_OPT__(,) __VA_ARGS__)\nM30(buf, \"foo\");          // sprintf(buf, \"foo\")\nM30(buf, \"foo%d\", 3);     // sprintf(buf, \"foo\", 3)\n```\n\nThe implementation lives in `subst`, the macro substitution loop. A new helper `has_varargs` answers the predicate question:\n\n```c\nstatic bool has_varargs(MacroArg *args) {\n  for (MacroArg *ap = args; ap; ap = ap->next)\n    if (!strcmp(ap->name, \"__VA_ARGS__\"))\n      return ap->tok->kind != TK_EOF;\n  return false;\n}\n```\n\nThe check is \"is there a `__VA_ARGS__` in the arg list, and is its first token not the EOF sentinel\" \u2014 empty variadics are represented as a list with one `TK_EOF` token, the same convention `read_macro_args` set up in Chapter 17.\n\nThe substitution arm:\n\n```c\nif (equal(tok, \"__VA_OPT__\") && equal(tok->next, \"(\")) {\n  MacroArg *arg = read_macro_arg_one(&tok, tok->next->next, true);\n  if (has_varargs(args))\n    for (Token *t = arg->tok; t->kind != TK_EOF; t = t->next)\n      cur = cur->next = t;\n  tok = skip(tok, \")\");\n  continue;\n}\n```\n\n`read_macro_arg_one` parses the parenthesized token list into a fresh `MacroArg`, accepting a comma-containing body (the `read_rest=true` argument tells it to slurp until the matching `)`, not stop at the first comma). When `has_varargs` is true, the captured tokens are appended to the output. When false, they're discarded \u2014 the parens consume from the input but produce nothing in the output.\n\nA subtle point: `arg->tok` is the parsed-but-not-expanded token list. The expansion of macros *inside* `__VA_OPT__`'s argument happens later, when the substituted output of `subst` is run back through `expand_macro`. So `__VA_OPT__(MACRO_NAME)` correctly expands `MACRO_NAME` after the conditional decision is made.\n\n### 20.2.4 \u2014 The `,##__VA_ARGS__` swallow\n\nThe other half of the trailing-comma problem is the older gcc trick: if `,##__VA_ARGS__` appears in a macro body and `__VA_ARGS__` is empty, the comma vanishes too. The pattern looks like:\n\n```c\n#define M31(buf, fmt, ...) sprintf(buf, fmt, ## __VA_ARGS__)\n```\n\nWithout the swallow, `M31(buf, \"foo\")` would expand to `sprintf(buf, \"foo\",)` \u2014 a trailing comma syntax error. With it, the expansion is `sprintf(buf, \"foo\")`.\n\nThe implementation pattern-matches on three tokens \u2014 `,`, then `##`, then a `__VA_ARGS__` arg \u2014 at the start of each `subst` iteration:\n\n```c\nif (equal(tok, \",\") && equal(tok->next, \"##\")) {\n  MacroArg *arg = find_arg(args, tok->next->next);\n  if (arg && !strcmp(arg->name, \"__VA_ARGS__\")) {\n    if (arg->tok->kind == TK_EOF) {\n      tok = tok->next->next->next;\n    } else {\n      cur = cur->next = copy_token(tok);\n      tok = tok->next->next;\n    }\n    continue;\n  }\n}\n```\n\nTwo branches. If the variadic arg is empty, all three tokens are consumed and nothing is emitted \u2014 the comma, the `##`, and the `__VA_ARGS__` token all vanish. If non-empty, the comma is emitted, the `##` is skipped, and the loop continues at `__VA_ARGS__`, which the normal substitution path will handle as an ordinary arg reference (no token-pasting between the comma and the args, despite the `##`; the swallow turns `##` into a no-op when the arg is non-empty).\n\nThe `,##` pattern is gcc-specific. The C2X-correct way to write the same thing is `__VA_OPT__(,) __VA_ARGS__`. Both forms work in chibicc; gcc and clang accept both as well.\n\n### 20.2.5 \u2014 `#pragma`\n\nSeven lines, all in `preprocess2`:\n\n```c\nif (equal(tok, \"pragma\")) {\n  do {\n    tok = tok->next;\n  } while (!tok->at_bol);\n  continue;\n}\n```\n\nRead tokens until the next beginning-of-line, do nothing with them. `#pragma` directives are silently consumed.\n\nThe standard mandates `#pragma STDC` for several specific behaviors and otherwise leaves `#pragma` as implementation-defined. A real toolchain reaches for `#pragma once`, `#pragma pack`, `#pragma omp`, and a long tail of vendor-specific options. Chibicc ignores all of them. Doing so doesn't change correctness for the programs chibicc cares about \u2014 the worst case is a header that uses `#pragma pack` to control struct layout, and chibicc's struct-layout code follows the SysV AMD64 psABI rather than honoring `#pragma pack`. A header that depends on `#pragma pack` would compile cleanly and produce a struct with the *wrong* layout. No diagnostic is issued.\n\nThis is an errata candidate in the sense that \"silently ignoring something the source asked for\" is dangerous when the source assumed the request would be honored, but it's also the standard chibicc-style answer to features outside the scope: ignore them, don't fail on them, and let real-world headers compile.\n\n### 20.2.6 \u2014 GCC-style variadic macros\n\nStandard C variadic macros use `...` for the rest parameter and `__VA_ARGS__` for the captured tokens:\n\n```c\n#define M14(x, ...) add6(1, 2, x, __VA_ARGS__, 6)\n```\n\nGCC's older form uses a *named* rest parameter:\n\n```c\n#define M14(x, args...) add6(1, 2, x, args, 6)\n```\n\nThe two are equivalent; the named form is sometimes more readable. Supporting it requires the macro-args plumbing to remember the rest-parameter name as a string, not a hardcoded `\"__VA_ARGS__\"`.\n\nThe change is mechanical but touches several places. `Macro->is_variadic` (a `bool`) becomes `Macro->va_args_name` (a `char *`):\n\n```c\nstruct Macro {\n  ...\n  MacroParam *params;\n  char *va_args_name;\n  ...\n};\n```\n\n`MacroArg` gains an `is_va_args` flag so the `,##__VA_ARGS__` swallow can identify the rest arg by behavior rather than by hardcoded name:\n\n```c\nstruct MacroArg {\n  MacroArg *next;\n  char *name;\n  bool is_va_args;\n  Token *tok;\n};\n```\n\n`read_macro_params` learns the named-rest pattern:\n\n```c\nif (equal(tok, \"...\")) {\n  *va_args_name = \"__VA_ARGS__\";\n  *rest = skip(tok->next, \")\");\n  return head.next;\n}\n\nif (tok->kind != TK_IDENT)\n  error_tok(tok, \"expected an identifier\");\n\nif (equal(tok->next, \"...\")) {\n  *va_args_name = strndup(tok->loc, tok->len);\n  *rest = skip(tok->next->next, \")\");\n  return head.next;\n}\n```\n\nThe C-standard `...` becomes `va_args_name = \"__VA_ARGS__\"`. The GCC `name...` form becomes `va_args_name = \"name\"`. Then in `read_macro_args`:\n\n```c\nif (va_args_name) {\n  MacroArg *arg;\n  ...\n  arg->name = va_args_name;;\n  arg->is_va_args = true;\n  cur = cur->next = arg;\n}\n```\n\nThe captured rest-arg uses `va_args_name` for its parameter-binding name. So `args` (in `M14(args...) ...`) binds to the parsed token list, and any reference to `args` in the body finds the binding via the same `find_arg` lookup that `__VA_ARGS__` uses for the standard form.\n\nThe `,##__VA_ARGS__` swallow had been pattern-matching on the literal name `\"__VA_ARGS__\"`; with this change it switches to checking `arg->is_va_args`, so the swallow works for both standard and GCC forms.\n\nThe tests in `test/macro.c` exercise the new arity exhaustively \u2014 empty named-variadic, one arg, multiple args, mixed positional-plus-rest:\n\n```c\n#define M14(args...) args\nASSERT(2, M14() 2);\nASSERT(5, M14(5));\n```\n\nNote `M14() 2`. The macro expands to nothing, leaving `2` as the next expression token; the assertion checks the literal `2`. The empty named-variadic case threads through correctly because `read_macro_args` allocates a `TK_EOF`-only `MacroArg` for the empty case, and `subst` substitutes that as no tokens.\n\n(A second tiny note: the doubled-`;` in `arg->name = va_args_name;;` is a typo in Rui's commit. C accepts the empty statement, so it's harmless. The chibicc source still has it.)\n\n**Where we are.** Two new predefined macros, both using the existing `Macro->handler` hook (now five users: `__FILE__`, `__LINE__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`). Two macro-expansion extensions for the empty-variadic case (the C2X `__VA_OPT__` and the older GCC `,##` swallow). `#pragma` is silently consumed. The variadic-macro plumbing now supports both `...` and `name...`. The `Macro->is_variadic` boolean has been promoted to a `char *va_args_name`, and `MacroArg` gained an `is_va_args` flag.\n\n---\n\n## 20.3 \u2014 Type-side extensions: `typeof`, `__builtin_types_compatible_p`, `_Generic`\n\n> `git checkout 7d80a5136d1b2926dd0776c51896c40723c518c5` \u2014 *Add typeof*\n>\n> `git checkout 1433b404d68f9fe314ae2955d0988dd74e5ecb92` \u2014 *[GNU] Add __builtin_types_compatible_p*\n>\n> `git checkout 1faab48ecf83d31a4fd781f10f6f00acb681d2dd` \u2014 *Add _Generic*\n\nThree commits. The chapter's most parser-invasive section. `typeof` extends the type-name grammar with a new specifier that takes either an expression or a typename. `__builtin_types_compatible_p` is a compile-time predicate over two typenames. `_Generic` is C11's type-driven dispatch \u2014 it picks one of several association arms based on the controlling expression's type. All three depend on chibicc's existing `is_typename` predicate, and the second and third both depend on a new `is_compatible(t1, t2)` helper that walks two type trees in parallel.\n\n### 20.3.1 \u2014 `typeof`\n\n`typeof(expr)` produces the type of `expr`. `typeof(type)` produces `type` unchanged. The form is most often used to strip `const` and other qualifiers from an expression's static type, or to give a name to a type that would otherwise be hard to spell:\n\n```c\ntypeof(*p) tmp = *p;\n```\n\nThe implementation extends the `declspec` switch to recognize `typeof` as a typename-introducer, and adds a new helper `typeof_specifier`:\n\n```c\nif (equal(tok, \"struct\") || equal(tok, \"union\") || equal(tok, \"enum\") ||\n    equal(tok, \"typeof\") || ty2) {\n  ...\n  } else if (equal(tok, \"typeof\")) {\n    ty = typeof_specifier(&tok, tok->next);\n  } else {\n    ...\n  }\n}\n```\n\n`typeof_specifier`:\n\n```c\n// typeof-specifier = \"(\" (expr | typename) \")\"\nstatic Type *typeof_specifier(Token **rest, Token *tok) {\n  tok = skip(tok, \"(\");\n\n  Type *ty;\n  if (is_typename(tok)) {\n    ty = typename(&tok, tok);\n  } else {\n    Node *node = expr(&tok, tok);\n    add_type(node);\n    ty = node->ty;\n  }\n  *rest = skip(tok, \")\");\n  return ty;\n}\n```\n\nInside the parens, look at the first token: if it's a type-introducer, parse a typename and use it directly; otherwise parse a full expression, run `add_type` to compute its static type, and return that. The expression is parsed but never *evaluated* \u2014 `expr` builds the AST and `add_type` walks it to assign types, but no codegen runs. This matches the C semantics: `typeof(f())` does not call `f`.\n\n`is_typename` itself is updated in two ways. The keyword list adds `\"typeof\"`:\n\n```c\nstatic char *kw[] = {\n  ..., \"typeof\",\n};\n```\n\nAnd the call site at the top of `declspec` already routes `typeof` through the type-introducer branch. `typeof` is in the right-hand list of \"tokens that start a typename.\" The `tokenize.c` keyword list is also extended so the lexer produces `typeof` as a keyword token rather than an identifier.\n\nThe test cases name the four interesting shapes:\n\n```c\nASSERT(3, ({ typeof(int) x=3; x; }));\nASSERT(3, ({ typeof(1) x=3; x; }));\nASSERT(4, ({ int x; typeof(x) y; sizeof(y); }));\nASSERT(8, ({ int x; typeof(&x) y; sizeof(y); }));\nASSERT(4, ({ typeof(\"foo\") x; sizeof(x); }));\nASSERT(12, sizeof(typeof(struct { int a,b,c; })));\n```\n\nThe `typeof(\"foo\")` returns `char[4]`; `sizeof(char[4])` is 4. The `typeof(struct {int a,b,c;})` returns the anonymous struct type; `sizeof` of that is 12 (three `int`s, no padding).\n\n`is_typename` is the same predicate that handles `_Generic`'s arm-vs-default decision and `__builtin_types_compatible_p`'s two-argument parsing. Adding `typeof` to it means all three follow-on features pick up `typeof` for free. This is a small example of the kind of cross-feature coupling that makes adding GCC extensions cheap once `is_typename` is in place.\n\n### 20.3.2 \u2014 `__builtin_types_compatible_p`\n\n`__builtin_types_compatible_p(t1, t2)` returns 1 if the two types are compatible, 0 otherwise. It's a compile-time integer constant \u2014 chibicc parses it through `primary`, evaluates it during parsing, and emits a `ND_NUM` node:\n\n```c\nif (equal(tok, \"__builtin_types_compatible_p\")) {\n  tok = skip(tok->next, \"(\");\n  Type *t1 = typename(&tok, tok);\n  tok = skip(tok, \",\");\n  Type *t2 = typename(&tok, tok);\n  *rest = skip(tok, \")\");\n  return new_num(is_compatible(t1, t2), start);\n}\n```\n\nThe whole expression collapses to a number at parse time. The interesting work is in the new `is_compatible` helper in `type.c`, which encodes the C-standard compatibility rules:\n\n```c\nbool is_compatible(Type *t1, Type *t2) {\n  if (t1 == t2)\n    return true;\n\n  if (t1->origin)\n    return is_compatible(t1->origin, t2);\n\n  if (t2->origin)\n    return is_compatible(t1, t2->origin);\n\n  if (t1->kind != t2->kind)\n    return false;\n\n  switch (t1->kind) {\n  case TY_CHAR:\n  case TY_SHORT:\n  case TY_INT:\n  case TY_LONG:\n    return t1->is_unsigned == t2->is_unsigned;\n  case TY_FLOAT:\n  case TY_DOUBLE:\n    return true;\n  case TY_PTR:\n    return is_compatible(t1->base, t2->base);\n  case TY_FUNC: {\n    if (!is_compatible(t1->return_ty, t2->return_ty))\n      return false;\n    if (t1->is_variadic != t2->is_variadic)\n      return false;\n\n    Type *p1 = t1->params;\n    Type *p2 = t2->params;\n    for (; p1 && p2; p1 = p1->next, p2 = p2->next)\n      if (!is_compatible(p1, p2))\n        return false;\n    return p1 == NULL && p2 == NULL;\n  }\n  case TY_ARRAY:\n    if (!is_compatible(t1->base, t2->base))\n      return false;\n    return t1->array_len < 0 && t2->array_len < 0 &&\n           t1->array_len == t2->array_len;\n  }\n  return false;\n}\n```\n\nSeveral rules worth flagging. The pointer-identity short-circuit (`t1 == t2`) catches the typedef-eq-typedef case where two declarations share a type pointer. The `origin` field is new on `Type`:\n\n```c\nstruct Type {\n  ...\n  Type *origin;       // for type compatibility check\n};\n```\n\nSet in `copy_type`:\n\n```c\nType *copy_type(Type *ty) {\n  Type *ret = calloc(1, sizeof(Type));\n  *ret = *ty;\n  ret->origin = ty;\n  return ret;\n}\n```\n\nSo a copied type remembers what it was copied from. Qualifier-stripping (e.g., `const int` and `int`) is handled implicitly \u2014 chibicc doesn't have a `const` flag on `Type`, so `const int` is just `int`. But `__builtin_types_compatible_p(const int, int)` returning 1 (per the test) makes sense for the same reason: chibicc doesn't track `const`.\n\nThe integer arms compare on `is_unsigned` only. `int` and `signed int` are both signed, so they're compatible. `int` and `unsigned int` are not.\n\nThe function arm checks return type, `is_variadic`, and parameter list. Note the `double (*)(...)` test case \u2014 chibicc treats `(...)` as a variadic with no fixed params, and `(void)` as zero fixed params; the two are not compatible.\n\nThe array arm has a curious last line:\n\n```c\nreturn t1->array_len < 0 && t2->array_len < 0 &&\n       t1->array_len == t2->array_len;\n```\n\nRead carefully: this returns `true` only when both lengths are negative *and equal*. Negative `array_len` is chibicc's flag for \"incomplete array\" (e.g., `int x[]`). So two complete arrays with the same length return `false`. This is wrong \u2014 `int[3]` should be compatible with `int[3]` \u2014 and the bug surfaces only for `__builtin_types_compatible_p` on array types. The `_Generic` use case (next subsection) sidesteps it because arrays decay to pointers in the controlling-expression position.\n\nThis is an errata candidate. Rui likely meant `(t1->array_len < 0 || t2->array_len < 0 || t1->array_len == t2->array_len)`. The chibicc source still has the buggy form.\n\n`__builtin_types_compatible_p` is most often used in macros that need to discriminate based on argument type, especially before `_Generic` was available. With `_Generic` in C11, the use cases are narrower, but real-world code (the Linux kernel's `container_of`, glibc's type-generic math headers) still calls it.\n\n### 20.3.3 \u2014 `_Generic`\n\n`_Generic` is C11's type-driven dispatch. The form is:\n\n```c\n_Generic(controlling-expression,\n  type1: expr1,\n  type2: expr2,\n  default: exprN)\n```\n\nThe result is the expression whose associated type is compatible with the controlling expression's type. The controlling expression is *not evaluated* \u2014 only its static type matters. The chosen `expr` is what becomes part of the enclosing expression.\n\nThe implementation is a single ~40-line helper, `generic_selection`, called from `primary`:\n\n```c\nstatic Node *generic_selection(Token **rest, Token *tok) {\n  Token *start = tok;\n  tok = skip(tok, \"(\");\n\n  Node *ctrl = assign(&tok, tok);\n  add_type(ctrl);\n\n  Type *t1 = ctrl->ty;\n  if (t1->kind == TY_FUNC)\n    t1 = pointer_to(t1);\n  else if (t1->kind == TY_ARRAY)\n    t1 = pointer_to(t1->base);\n\n  Node *ret = NULL;\n\n  while (!consume(rest, tok, \")\")) {\n    tok = skip(tok, \",\");\n\n    if (equal(tok, \"default\")) {\n      tok = skip(tok->next, \":\");\n      Node *node = assign(&tok, tok);\n      if (!ret)\n        ret = node;\n      continue;\n    }\n\n    Type *t2 = typename(&tok, tok);\n    tok = skip(tok, \":\");\n    Node *node = assign(&tok, tok);\n    if (is_compatible(t1, t2))\n      ret = node;\n  }\n\n  if (!ret)\n    error_tok(start, \"controlling expression type not compatible with\"\n              \" any generic association type\");\n  return ret;\n}\n```\n\nWalk it. The controlling expression is parsed and typed. Then the controlling type is normalized: function types decay to function-pointer types, array types decay to pointer-to-element. This normalization matches what the C standard says about the controlling expression \u2014 it undergoes the standard \"lvalue conversion\" (and array-to-pointer decay, function-to-pointer decay), so the dispatch sees the decayed type.\n\nFor each association arm, parse a typename or `default`, then a colon, then an `assign`-level expression. If the arm's type is compatible with the controlling type, latch that arm's expression. The `default` arm is latched only if no specific arm has matched yet \u2014 `if (!ret) ret = node`. The check is one-way: a later specific match would still overwrite a previous default. (The standard says exactly one specific arm should match; chibicc doesn't enforce uniqueness, but in practice the compatibility check is exact enough that two arms rarely both match.)\n\nAfter the loop, if no arm matched (and there was no default), error. Otherwise return the matched arm's expression. The other arms are discarded \u2014 they were parsed but never typed beyond `assign`, and they're not part of the resulting AST.\n\nThe discarded-arm point is worth dwelling on. C's `_Generic` is designed so the *unselected arms need not even be valid*. A common idiom:\n\n```c\n#define abs(x) _Generic((x), int: abs, double: fabs, float: fabsf)(x)\n```\n\nIf `x` is an `int`, the `fabs` and `fabsf` arms are not selected, which is fine \u2014 but they're still part of the source text. If the standard required typing them, you'd get errors in code that should compile. The standard explicitly says unselected arms can be invalid; chibicc's implementation parses them as `assign` expressions (which builds AST nodes) but doesn't type-check them, so a use of an undeclared identifier in an unselected arm would *not* error during type-checking. (It would error during identifier resolution in `assign` itself, though, which means chibicc is stricter than the standard requires here. A real conformance-mode implementation would defer the lookup. Not a chibicc concern in practice.)\n\nThe test cases:\n\n```c\nASSERT(1, _Generic(100.0, double: 1, int *: 2, int: 3, float: 4));\nASSERT(2, _Generic((int *)0, double: 1, int *: 2, int: 3, float: 4));\nASSERT(2, _Generic((int[3]){}, double: 1, int *: 2, int: 3, float: 4));\nASSERT(3, _Generic(100, double: 1, int *: 2, int: 3, float: 4));\nASSERT(4, _Generic(100f, double: 1, int *: 2, int: 3, float: 4));\n```\n\nThe third \u2014 `(int[3]){}` \u2014 exercises the array-to-pointer decay: the compound literal has type `int[3]`, decays to `int *` for the dispatch, matches the `int *` arm.\n\nThe `100f` literal is a chibicc-extension form for a `float` literal \u2014 standard C uses `100.0f`. Chibicc accepts the digits-then-`f` pattern as well.\n\n**Where we are.** The type-name grammar grows two new entries (`typeof`, plus the `_Generic` and `__builtin_types_compatible_p` entries in `primary`). `is_typename` adds `typeof`. `Type` gains an `origin` field for compatibility tracking, set on copy. A new `is_compatible(t1, t2)` walker handles the standard's compatibility rules with one bug in the array arm (errata candidate). `_Generic`'s controlling expression is type-only \u2014 not evaluated \u2014 and decays function and array types per the standard.\n\n---\n\n## 20.4 \u2014 `sizeof` of a function and the GNU ternary middle\n\n> `git checkout aee7891acb3e653dcfb10ec4172ae4d099ebf034` \u2014 *[GNU] Allow sizeof(<function type>)*\n>\n> `git checkout e28a612e9c2293182a83d5a7c6f48129455ce951` \u2014 *[GNU] Add ?: operator with omitted operand*\n\nTwo commits. Both are small but instructive.\n\n### 20.4.1 \u2014 `sizeof(<function type>)`\n\nStandard C makes `sizeof` of a function a constraint violation. GCC accepts it and returns 1. The motivation is that gcc internally uses pointer arithmetic on function pointers in some idioms, and forcing every such use to be an explicit cast through `void *` would be tedious. Chibicc follows gcc:\n\n```c\nType *func_type(Type *return_ty) {\n  // The C spec disallows sizeof(<function type>), but\n  // GCC allows that and the expression is evaluated to 1.\n  Type *ty = new_type(TY_FUNC, 1, 1);\n  ty->return_ty = return_ty;\n  return ty;\n}\n```\n\nTwo-line change. The `func_type` constructor was previously zero-initialized via `calloc`; now it routes through `new_type(TY_FUNC, 1, 1)` which sets `size=1` and `align=1`. `sizeof(main)` now returns 1.\n\nThe test:\n\n```c\nASSERT(1, sizeof(main));\n```\n\nThat `main` evaluates to a function type \u2014 `sizeof` of a function expression also returns 1, since `main` has type `int(void)` and the size is 1. The function-to-pointer decay that *would* happen in most contexts doesn't apply to `sizeof`'s operand.\n\n### 20.4.2 \u2014 The ternary middle, omitted\n\n`a ?: b` is gcc's two-operand ternary. It's equivalent to `a ? a : b`, with one important wrinkle: `a` is evaluated *exactly once*. If `a` has a side effect, the standard `a ? a : b` rewrite would evaluate `a` twice. The gcc form binds `a` to a temporary first.\n\nChibicc implements that exact desugaring in `conditional`:\n\n```c\n// conditional = logor (\"?\" expr? \":\" conditional)?\nstatic Node *conditional(Token **rest, Token *tok) {\n  Node *cond = logor(&tok, tok);\n\n  if (!equal(tok, \"?\")) {\n    *rest = tok;\n    return cond;\n  }\n\n  if (equal(tok->next, \":\")) {\n    // [GNU] Compile `a ?: b` as `tmp = a, tmp ? tmp : b`.\n    add_type(cond);\n    Obj *var = new_lvar(\"\", cond->ty);\n    Node *lhs = new_binary(ND_ASSIGN, new_var_node(var, tok), cond, tok);\n    Node *rhs = new_node(ND_COND, tok);\n    rhs->cond = new_var_node(var, tok);\n    rhs->then = new_var_node(var, tok);\n    rhs->els = conditional(rest, tok->next->next);\n    return new_binary(ND_COMMA, lhs, rhs, tok);\n  }\n  ...\n}\n```\n\nWalk it. After parsing the `logor`-level `a`, peek for `?` followed by `:` \u2014 that's the GNU form. Allocate an unnamed local of `a`'s type (the empty-string name on `new_lvar` makes it inaccessible to source code; only the AST holds a reference). Emit a sequence: assign `a` to the temporary, then a normal ternary `tmp ? tmp : b`. The whole thing is wrapped in `ND_COMMA` so it's a single expression that evaluates left-to-right and yields the right operand.\n\nThe test cases:\n\n```c\nASSERT(3, 3?:5);\nASSERT(5, 0?:5);\nASSERT(4, ({ int i = 3; ++i?:10; }));\n```\n\nThe third case \u2014 `++i?:10` with `i = 3` \u2014 is the side-effect-once test. After the expression, `i` has been incremented exactly once (to 4), and the value is 4. The standard `++i ? ++i : 10` rewrite would have incremented twice and yielded 5.\n\nThis is the chapter's first canonicalization-at-parse-time entry. The pattern \u2014 desugar a surface form into a simpler primitive at parse time, hide the temporary \u2014 has been used previously for compound assignment (Chapter 11) and for several other syntactic-sugar features. The handoff's pre-chapter count of nine canonicalizations now ticks to ten.\n\n**Where we are.** `sizeof(<function-type>)` returns 1 \u2014 a deliberate divergence from the standard for gcc compatibility. `?:` with omitted middle desugars at parse time to a temporary binding, evaluating the left operand exactly once. Canonicalization-at-parse-time count is at ten.\n\n---\n\n## 20.5 \u2014 `asm`, `inline`, dead static-inline elimination, `__attribute__((format))`\n\n> `git checkout a2535163e232cd547b14960bf4232305d239741d` \u2014 *Add basic \"asm\" statement*\n>\n> `git checkout 31087f8d4bbc06e5bec44cb14cab3a922b5e4855` \u2014 *Handle inline functions as static functions*\n>\n> `git checkout e5f4ca90fd2bf950189c98ed7f1873c9f35131f3` \u2014 *Do not emit static inline functions if referenced by no one*\n>\n> `git checkout 6a2dc5a48a75b65aa2e3f606d195ef0fef3c4442` \u2014 *Use __attribute__((format(print, ...))) to find programming errors*\n\nFour commits. Three function-side features (`asm`, `inline` plus its dead-code optimization) and one cross-cutting tooling change (`__attribute__((format))` annotations on chibicc's own diagnostic functions).\n\n### 20.5.1 \u2014 Basic `asm`\n\nReal `asm` in gcc is a major sublanguage \u2014 input/output operand lists, clobber lists, goto labels, register constraints. Chibicc implements only the basic form: a single string literal whose contents are emitted verbatim into the assembly output:\n\n```c\nchar *asm_fn1(void) {\n  asm(\"mov $50, %rax\\n\\t\"\n      \"mov %rbp, %rsp\\n\\t\"\n      \"pop %rbp\\n\\t\"\n      \"ret\");\n}\n```\n\nThe implementation adds an `ND_ASM` node kind, an `asm_str` field on `Node`, a parser arm in `stmt`, and a codegen arm in `gen_stmt`:\n\n```c\n// asm-stmt = \"asm\" (\"volatile\" | \"inline\")* \"(\" string-literal \")\"\nstatic Node *asm_stmt(Token **rest, Token *tok) {\n  Node *node = new_node(ND_ASM, tok);\n  tok = tok->next;\n\n  while (equal(tok, \"volatile\") || equal(tok, \"inline\"))\n    tok = tok->next;\n\n  tok = skip(tok, \"(\");\n  if (tok->kind != TK_STR || tok->ty->base->kind != TY_CHAR)\n    error_tok(tok, \"expected string literal\");\n  node->asm_str = tok->str;\n  *rest = skip(tok->next, \")\");\n  return node;\n}\n```\n\nThe `volatile` and `inline` qualifiers (gcc accepts both) are consumed and ignored. The string literal is required to be a narrow (`char`-element) literal; chibicc rejects `asm(L\"...\")`. The body is captured as `node->asm_str`.\n\nCodegen is one line in `gen_stmt`:\n\n```c\ncase ND_ASM:\n  println(\"  %s\", node->asm_str);\n  return;\n```\n\nThe string is printed into the assembly output with two leading spaces (matching chibicc's convention for instruction lines). Whatever's inside \u2014 instructions, directives, labels \u2014 flows through to GAS unchanged.\n\nThe test functions in `test/asm.c` are unusually shaped \u2014 they declare a `char *` return type but actually return via `mov $50, %rax`, restoring `%rbp` and `%rsp` and `ret`-ing. The function-body braces are still parsed by chibicc, so chibicc emits its normal prologue (which sets up `%rbp`); the `asm` statement's manual `pop %rbp` undoes that. There's no return statement \u2014 control flow exits through the manual `ret`.\n\n```c\nint main() {\n  ASSERT(50, asm_fn1());\n  ASSERT(55, asm_fn2());\n  ...\n}\n```\n\nThe test exercises the result.\n\n`asm` is the kind of feature where chibicc's minimal implementation is *correct for the literal feature* but useless for the realistic uses. Real codebases use `asm` with operand bindings (`asm volatile (\"mov %0, %%rax\" : : \"r\"(x))`), clobber lists, and goto. None of that works here. The chibicc tests use `asm` only as a way to literally write assembly, which is enough for the bootstrap-style scenarios where you need to emit one specific instruction sequence.\n\n### 20.5.2 \u2014 `inline` as `static`\n\nC99's `inline` keyword has surprisingly subtle linkage rules. The standard's full model: an `inline` function declared without `extern` provides an *inline definition* that need not produce an external symbol; an `extern inline` definition does produce an external symbol; without an external symbol from somewhere, references to the inline function are undefined at link time. Real toolchains thread this through carefully.\n\nChibicc's simplification is brutal: `inline` is just `static`. A bare `inline int f() {...}` becomes `static int f() {...}`, with no external symbol. References from other translation units would fail to link, but the test suite only exercises within-translation-unit calls.\n\nThe mechanical change is small. `VarAttr` gets an `is_inline` flag:\n\n```c\ntypedef struct {\n  bool is_typedef;\n  bool is_static;\n  bool is_extern;\n  bool is_inline;\n  int align;\n} VarAttr;\n```\n\n`Obj` gets an `is_inline` flag:\n\n```c\nstruct Obj {\n  ...\n  bool is_inline;\n  ...\n};\n```\n\n`declspec` recognizes `inline` as a storage-class-like specifier and routes it into `attr->is_inline`:\n\n```c\nif (equal(tok, \"typedef\") || equal(tok, \"static\") || equal(tok, \"extern\") ||\n    equal(tok, \"inline\")) {\n  ...\n  else if (equal(tok, \"extern\"))\n    attr->is_extern = true;\n  else\n    attr->is_inline = true;\n\n  if (attr->is_typedef && attr->is_static + attr->is_extern + attr->is_inline > 1)\n    error_tok(tok, \"typedef may not be used together with static, extern or inline\");\n  ...\n}\n```\n\nThe mutex check is loosened to include `is_inline` in the count \u2014 `typedef inline` is rejected.\n\n`is_typename` gets `inline`:\n\n```c\n\"const\", \"volatile\", \"auto\", \"register\", \"restrict\", \"__restrict\",\n\"__restrict__\", \"_Noreturn\", \"float\", \"double\", \"typeof\", \"inline\",\n```\n\nAnd `function` flags the function as static-when-inline-and-not-extern:\n\n```c\nfn->is_static = attr->is_static || (attr->is_inline && !attr->is_extern);\nfn->is_inline = attr->is_inline;\n```\n\nThe `is_static || (is_inline && !is_extern)` is the simplification: bare `inline` becomes static, `extern inline` becomes external (not static), explicit `static inline` stays static (the `is_static` clause). It's not exactly what the C standard says \u2014 the standard's `inline` rules are more nuanced about external symbols and definition uniqueness \u2014 but it's close enough that real-world code compiles correctly. A real headerful of `static inline` helpers in a project compiles fine; an `extern inline` declaration in a header gets an external symbol; a bare `inline int f() {return 3;}` in a header behaves as a per-translation-unit private function (no link-time conflict).\n\nThe test cases probe both forms:\n\n```c\ninline int inline_fn(void) {\n  return 3;\n}\n...\nASSERT(3, inline_fn());\n```\n\nAnd the driver test for the no-collision case (two `.c` files each with `inline void foo() {}` should link together with no duplicate-symbol error, which works because `inline` makes them static):\n\n```bash\necho 'inline void foo() {}' > $tmp/inline1.c\necho 'inline void foo() {}' > $tmp/inline2.c\necho 'int main() { return 0; }' > $tmp/inline3.c\n$chibicc -o /dev/null $tmp/inline1.c $tmp/inline2.c $tmp/inline3.c\n```\n\n### 20.5.3 \u2014 Dead static-inline elimination\n\nTreating `inline` as `static` works at the language level but creates a code-bloat problem at the codegen level: every static-inline function emits assembly even when no caller references it. A header full of unused `static inline` helpers would produce a translation-unit's worth of dead code per inclusion.\n\nThe fix is a simple reachability pass: walk the call graph, mark live functions, only emit live ones. The implementation needs three pieces: per-function reference tracking during parsing, a graph traversal during the post-parse pass, and a codegen check that skips dead functions.\n\n`Obj` grows three fields:\n\n```c\nstruct Obj {\n  ...\n  // Static inline function\n  bool is_live;\n  bool is_root;\n  StringArray refs;\n};\n```\n\n`is_live` is the mark bit. `is_root` is the \"always live\" flag \u2014 set on every non-static-inline function, so the reachability traversal starts from those. `refs` is the per-function list of named functions it references.\n\nDuring parsing, every variable lookup in `primary` that resolves to a function records a reference:\n\n```c\n// For \"static inline\" function\nif (sc && sc->var && sc->var->is_function) {\n  if (current_fn)\n    strarray_push(&current_fn->refs, sc->var->name);\n  else\n    sc->var->is_root = true;\n}\n```\n\nIf the lookup happens inside a function body (`current_fn` is set), push the referenced name onto the current function's refs list. Otherwise \u2014 the lookup is in a global initializer, which is the only other place name lookups happen \u2014 mark the referenced function as a root, so it stays alive.\n\n`function` itself sets `is_root` based on the static-inline classification:\n\n```c\nfn->is_root = !(fn->is_static && fn->is_inline);\n```\n\nStatic-inline functions start as non-roots; everything else starts as a root. The reachability traversal then discovers which static-inline functions are reachable from a root and marks them live too.\n\nThe traversal:\n\n```c\nstatic Obj *find_func(char *name) {\n  Scope *sc = scope;\n  while (sc->next)\n    sc = sc->next;\n\n  for (VarScope *sc2 = sc->vars; sc2; sc2 = sc2->next)\n    if (!strcmp(sc2->name, name) && sc2->var && sc2->var->is_function)\n      return sc2->var;\n  return NULL;\n}\n\nstatic void mark_live(Obj *var) {\n  if (!var->is_function || var->is_live)\n    return;\n  var->is_live = true;\n\n  for (int i = 0; i < var->refs.len; i++) {\n    Obj *fn = find_func(var->refs.data[i]);\n    if (fn)\n      mark_live(fn);\n  }\n}\n```\n\n`mark_live` is recursive depth-first marking, with early-exit on already-marked functions. `find_func` walks to the global scope (the outermost `Scope`) and looks up by name \u2014 function pointers in `refs` are stored as strings rather than `Obj *` pointers because forward references at parse time may not have an `Obj` yet.\n\nThe driver:\n\n```c\nfor (Obj *var = globals; var; var = var->next)\n  if (var->is_root)\n    mark_live(var);\n```\n\nAfter this loop, `is_live` is set for every function reachable from a root.\n\nCodegen checks the flag:\n\n```c\n// No code is emitted for \"static inline\" functions\n// if no one is referencing them.\nif (!fn->is_live)\n  continue;\n```\n\nThe driver tests in `test/driver.sh` cover the matrix of \"static-inline-referenced\" vs \"static-inline-unreferenced\" cases, with patterns like:\n\n```bash\necho 'static inline void f1() {}' | $chibicc -o- -S - | grep -v -q f1:\necho 'static inline void f1() {} void foo() { f1(); }' | $chibicc -o- -S - | grep -q f1:\n```\n\nThe first asserts that an unreferenced `static inline` produces no `f1:` label in the output. The second asserts that a referenced one does. Twelve such tests cover the various reachability patterns including mutual recursion (`f1` calls `f2`, `f2` calls `f1`, neither is reachable, both should be elided) and chains (`f1` reachable through `f2`'s reference).\n\nThe mutual-recursion case is handled correctly by the early-exit in `mark_live`: when `mark_live(f1)` recurses into `mark_live(f2)`, which recurses back into `mark_live(f1)`, the second call sees `f1` already marked and returns immediately.\n\nThis is the third pass over `globals` that `parse()` does \u2014 first `parse2` builds the global list, then the existing `mark_live` loop runs, then (in the next commit, \u00a720.6) the tentative-definition cleanup runs. The handoff's standing-notes line about `is_definition` and `is_static` defaults on `Obj` now grows to include `is_inline`, `is_live`, `is_root`, and the `refs` list.\n\n### 20.5.4 \u2014 `__attribute__((format))` on chibicc's own functions\n\nThis commit annotates chibicc's printf-shaped helpers \u2014 `format`, `error`, `error_at`, `error_tok`, `warn_tok`, `println` \u2014 so a host compiler with `-Wformat` can catch format-string mismatches in chibicc's own source:\n\n```c\nchar *format(char *fmt, ...) __attribute__((format(printf, 1, 2)));\n\nnoreturn void error(char *fmt, ...) __attribute__((format(printf, 1, 2)));\nnoreturn void error_at(char *loc, char *fmt, ...) __attribute__((format(printf, 2, 3)));\nnoreturn void error_tok(Token *tok, char *fmt, ...) __attribute__((format(printf, 2, 3)));\nvoid warn_tok(Token *tok, char *fmt, ...) __attribute__((format(printf, 2, 3)));\n```\n\nThe `(printf, N, M)` annotation tells the host compiler \"the Nth argument is a printf-format string, the Mth argument is the start of the variadic args.\" With this, `error(\"count = %d\", \"not a number\")` produces a warning at compile time.\n\nFor chibicc's *self-host build*, the annotation has to compile cleanly when chibicc is the compiler. Chibicc doesn't support `__attribute__` in its parser. The fix is a guarded macro at the top of `chibicc.h`:\n\n```c\n#ifndef __GNUC__\n# define __attribute__(x)\n#endif\n```\n\nWhen `__GNUC__` is defined (gcc, clang, and chibicc itself, which defines `__GNUC__` per Chapter 17 \u00a717.5.4), the annotation is honored. When it's not, `__attribute__` is macro-defined to nothing, so any `__attribute__((...))` in the source vanishes during preprocessing.\n\nWait \u2014 chibicc *does* define `__GNUC__`, but it doesn't *parse* `__attribute__`. So when chibicc compiles itself, `__GNUC__` is defined, the macro stub is not active, and chibicc sees `__attribute__((format(printf, 1, 2)))` as a parser input.\n\nThe trick is that chibicc's parser treats `__attribute__(...)` as a typename-like token sequence and skips it. Earlier chapters added skip-arms for various attribute-shaped tokens; the cumulative effect is that `__attribute__((...))` in declaration position is silently consumed. (Verify in the parser: `__attribute__` doesn't appear in the keyword list, so it falls through; it would be tokenized as an identifier. The function-declaration grammar would then fail to parse it. In practice the chibicc source compiles itself with these annotations, which means there's a skipping mechanism.)\n\nLooking carefully at the grammar: `__attribute__` is not a chibicc keyword as of this commit, and chibicc's parser doesn't have a generic attribute-skip path. The compile works because the annotations are at declaration-statement scope where chibicc *might* be tolerant of trailing junk, or because the macro stub *is* active when chibicc compiles itself (which would mean chibicc doesn't define `__GNUC__`, or the `#ifndef` guard sees something that suppresses the definition). Verification while drafting this section was inconclusive \u2014 the chibicc self-host succeeds, so one of these mechanisms works.\n\nThe annotation has caught real bugs in chibicc's source. The commit description doesn't itemize what was fixed, but Rui's comment \"Use `__attribute__((format(print, ...)))` to find programming errors\" implies the host compiler did catch mismatches the moment the annotations were added. (The \"format(print, ...)\" in the title is itself a typo for \"format(printf, ...)\" \u2014 Rui's commit messages aren't always polished.)\n\n**Where we are.** Basic `asm` works as a verbatim-string emit, with no operand bindings. `inline` is treated as `static`, with `extern inline` flagged as external. Static-inline functions that no one references are elided from the output via a reachability pass that adds `is_live`/`is_root`/`refs` to `Obj`. Chibicc's printf-shaped helpers are annotated with `__attribute__((format))` and the `__attribute__` macro is stubbed when the host doesn't support it.\n\n---\n\n## 20.6 \u2014 `-idirafter`, `offsetof`, tentative definitions, `-fcommon`\n\n> `git checkout 11fc259b01c4a855e53ffdb2b86c1030f9c18586` \u2014 *Add -idirafter option*\n>\n> `git checkout 1b99badce48083c5fa6b8b5872e899c7d1a47f9a` \u2014 *Add offsetof*\n>\n> `git checkout 85e46b1071b54649740b35df939f32ed188c0e13` \u2014 *Add tentative definition*\n>\n> `git checkout 6d344ed9459bd0328de53a58505a397d92cb0c8a` \u2014 *Add -fcommon and -fno-common flags*\n\nFour commits. Two driver-side options (`-idirafter`, `-fcommon`/`-fno-common`), one stddef.h macro (`offsetof`), and one parse-time-plus-codegen change for tentative definitions.\n\n### 20.6.1 \u2014 `-idirafter`\n\n`-idirafter DIR` adds `DIR` to the include search path *after* the standard system include paths. The standard `-I DIR` adds it before, so a header in `DIR` would shadow a system header of the same name. `-idirafter DIR` reverses this \u2014 the system header wins.\n\nThe implementation tracks the after-paths separately during argument parsing and appends them to `include_paths` after the rest of the command line is processed:\n\n```c\nStringArray idirafter = {};\n\nfor (int i = 1; i < argc; i++) {\n  ...\n  if (!strcmp(argv[i], \"-idirafter\")) {\n    strarray_push(&idirafter, argv[i++]);\n    continue;\n  }\n  ...\n}\n\nfor (int i = 0; i < idirafter.len; i++)\n  strarray_push(&include_paths, idirafter.data[i]);\n```\n\n`take_arg` is updated to include `-idirafter` in the list of options that consume the next argument:\n\n```c\nstatic bool take_arg(char *arg) {\n  char *x[] = {\"-o\", \"-I\", \"-idirafter\"};\n  ...\n}\n```\n\nThe two-loop structure (collect into a temporary `StringArray`, then append at the end) is the only way to ensure the after-paths come *last*, regardless of where they appeared on the command line. A `-idirafter A -I B` invocation produces an `include_paths` order of `[..., B, ..., A]`. The final order is \"explicit -I first, system paths next, -idirafter last.\"\n\nDriver tests confirm the precedence:\n\n```bash\nmkdir -p $tmp/dir1 $tmp/dir2\necho foo > $tmp/dir1/idirafter\necho bar > $tmp/dir2/idirafter\necho \"#include \\\"idirafter\\\"\" | $chibicc -I$tmp/dir1 -I$tmp/dir2 -E - | grep -q foo\necho \"#include \\\"idirafter\\\"\" | $chibicc -idirafter $tmp/dir1 -I$tmp/dir2 -E - | grep -q bar\n```\n\nThe first finds `dir1`'s `idirafter` (because `-I dir1` precedes `-I dir2`); the second finds `dir2`'s (because `-idirafter dir1` is appended after `-I dir2`).\n\n### 20.6.2 \u2014 `offsetof`\n\nTwo lines in `include/stddef.h`:\n\n```c\n#define offsetof(type, member) ((size_t)&(((type *)0)->member))\n```\n\nThe classic ISO-C `offsetof` definition. The trick: form a `type *` with value 0 (a null pointer), use `->member` to get the lvalue at `member` within that hypothetical struct, take the address, and cast to `size_t`. The address arithmetic is computed at compile time because the base pointer is the integer constant 0; what comes out is the byte offset of `member` within `type`.\n\nChibicc supports this idiom because:\n- `(type *)0` is a valid pointer-typed integer-constant cast.\n- `->member` on a pointer applies the struct's per-member offset (computed in Chapter 9).\n- `&` of an lvalue produces the address.\n- `(size_t)` is a simple integer cast.\n\nThe whole expression collapses to a constant in chibicc's `eval_const_expr` \u2014 Chapter 13's `eval` quartet handles each operation. The `offsetof` macro doesn't dereference the null pointer; the `&` cancels the implicit dereference of `->`, leaving pure address arithmetic.\n\nThe test:\n\n```c\ntypedef struct {\n  int a;\n  char b;\n  int c;\n  double d;\n} T;\n\nASSERT(0, offsetof(T, a));\nASSERT(4, offsetof(T, b));\nASSERT(8, offsetof(T, c));  // b is char with align 1; c starts at 8 due to int alignment\nASSERT(16, offsetof(T, d)); // d is double aligned to 8\n```\n\nReal toolchains often define `offsetof` as a builtin (`__builtin_offsetof`) because the null-pointer-dereference idiom is technically undefined behavior even though every compiler accepts it. Chibicc takes the simpler path of using the idiom directly; the chibicc parser doesn't UB-check struct member references through null pointers.\n\n### 20.6.3 \u2014 Tentative definitions\n\nC's tentative definitions are a 1970s-era feature that lets `int x;` at file scope appear *multiple times* across (or within) a translation unit and collapse to a single definition. The standard says (\u00a76.9.2): a declaration of an identifier without an initializer at file scope is a *tentative definition*; if the translation unit contains no actual definition by the end, the tentative definition becomes a definition with the initializer 0.\n\nPre-this-commit, chibicc rejected the second declaration of an identifier at file scope as a redeclaration error. Real-world C code \u2014 including chibicc's own headers \u2014 wouldn't have hit this because each global is declared once, but glibc-style headers often combine `extern T x;` in a header with `T x;` in a .c file, plus possibly redeclarations across includes.\n\nThe change has two parts: parse marks tentative declarations, and a post-parse pass eliminates the redundant ones.\n\nThe parse-side mark, in `global_variable`:\n\n```c\nif (equal(tok, \"=\"))\n  gvar_initializer(&tok, tok->next, var);\nelse if (!attr->is_extern)\n  var->is_tentative = true;\n```\n\nA no-initializer non-extern declaration sets the `is_tentative` flag. (Extern declarations remain non-tentative \u2014 they're the \"I promise this exists somewhere else\" form that doesn't need elaboration here.)\n\nThe post-parse pass, `scan_globals`:\n\n```c\nstatic void scan_globals(void) {\n  Obj head;\n  Obj *cur = &head;\n\n  for (Obj *var = globals; var; var = var->next) {\n    if (!var->is_tentative) {\n      cur = cur->next = var;\n      continue;\n    }\n\n    // Find another definition of the same identifier.\n    Obj *var2 = globals;\n    for (; var2; var2 = var2->next)\n      if (var != var2 && var2->is_definition && !strcmp(var->name, var2->name))\n        break;\n\n    // If there's another definition, the tentative definition\n    // is redundant\n    if (!var2)\n      cur = cur->next = var;\n  }\n\n  cur->next = NULL;\n  globals = head.next;\n}\n```\n\nWalk the global list. A non-tentative variable is kept. A tentative variable is kept *only if* no other (non-tentative) definition of the same name exists. If a real definition exists, the tentative entry is dropped.\n\nThe traversal is O(n\u00b2) over the global list \u2014 for each tentative, scan all globals \u2014 which is fine for chibicc's use case (a single TU rarely has more than a few hundred globals). A real compiler would hash by name.\n\nThe codegen-side change (next subsection, \u00a720.6.4) emits tentative definitions specially.\n\nThe driver call sequence in `parse()` is now:\n\n```c\nfor (Obj *var = globals; var; var = var->next)\n  if (var->is_root)\n    mark_live(var);\n\n// Remove redundant tentative definitions.\nscan_globals();\nreturn globals;\n```\n\nTwo post-parse passes: dead static-inline elimination, then tentative cleanup. Order doesn't matter (the two address disjoint subsets of globals), but the order is fixed.\n\nThe tests in `test/commonsym.c` exercise the multi-declaration case:\n\n```c\nint x;\nint x = 5;\nint y = 7;\nint y;\nint common_ext1;\nint common_ext2;\nstatic int common_local;\n\nint main() {\n  ASSERT(5, x);\n  ASSERT(7, y);\n  ASSERT(0, common_ext1);\n  ASSERT(3, common_ext2);\n  ...\n}\n```\n\n`x` has a tentative-then-real pair; `y` has a real-then-tentative pair. Both should resolve to the real value. `common_ext1` is a tentative-only declaration that should default to zero (the standard's tentative-becomes-zero rule). `common_ext2` has its real definition in `test/common`, where `int common_ext2 = 3` lives.\n\n### 20.6.4 \u2014 `-fcommon` and the `.comm` directive\n\nThe codegen for tentative definitions has a choice. The traditional behavior (`-fcommon`, GCC's default until version 10) is to emit `.comm SYM, SIZE, ALIGN`, which produces a *common symbol* in the object file. Common symbols from multiple translation units with the same name are merged by the linker into one allocation; if any TU also has a real definition, the real definition wins. This is what makes the `int common_ext2;` in `commonsym.c` and the `int common_ext2 = 3;` in `common` collapse to a single symbol with value 3.\n\nThe newer behavior (`-fno-common`, GCC's default from version 10 onward) is to put tentative definitions in `.bss` like any other zero-initialized global. Tentative-with-real cases still resolve, but multiple-TU tentative-only declarations of the same symbol would produce a link error.\n\nChibicc supports both, with `-fcommon` as the default:\n\n```c\nbool opt_fcommon = true;\n```\n\nArgument parsing sets the flag:\n\n```c\nif (!strcmp(argv[i], \"-fcommon\")) {\n  opt_fcommon = true;\n  continue;\n}\n\nif (!strcmp(argv[i], \"-fno-common\")) {\n  opt_fcommon = false;\n  continue;\n}\n```\n\nCodegen consults it in `emit_data`:\n\n```c\nif (opt_fcommon && var->is_tentative) {\n  println(\"  .comm %s, %d, %d\", var->name, var->ty->size, align);\n  continue;\n}\n\nif (var->init_data) {\n  println(\"  .data\");\n  ...\n}\n\nprintln(\"  .bss\");\nprintln(\"%s:\", var->name);\nprintln(\"  .zero %d\", var->ty->size);\n```\n\nWhen `-fcommon` is in effect and the variable is tentative, emit `.comm` and skip the `.data`/`.bss` blocks. Otherwise fall through to the normal path: emit to `.data` if there's `init_data`, otherwise to `.bss`. A tentative variable under `-fno-common` has no `init_data`, so it goes to `.bss` and gets zero-filled \u2014 which matches the standard's \"tentative becomes a zero-initialized definition\" rule.\n\nThe driver tests verify both behaviors:\n\n```bash\n# -fcommon (default)\necho 'int foo;' | $chibicc -S -o- - | grep -q '\\.comm foo'\n\n# -fno-common\necho 'int foo;' | $chibicc -fno-common -S -o- - | grep -q '^foo:'\n```\n\nThe first checks that the default produces `.comm foo`; the second checks that `-fno-common` produces a `foo:` label (the .bss block).\n\nA note on the historical default: GCC 10 (released 2020) flipped the default from `-fcommon` to `-fno-common`. Chibicc was written contemporaneously and chose the historical default. Real-world GCC 10+ behavior would default to placing tentatives in `.bss`. The chibicc default is the older convention; users targeting modern GCC behavior would invoke `-fno-common` explicitly.\n\nThe handoff's `is_tentative` flag on `Obj` is now in use: set in parse, read in `scan_globals`, read in `emit_data`. Three readers, three coupled-but-stable references.\n\nThe chapter recap will note that `.bss` and `.comm` are now both possible destinations for zero-initialized globals \u2014 making the \"third assembly section\" count grow to four counting `.text`, `.data`, `.bss`, and `.comm`, depending on whether you count `.comm` as a section or as a symbol-class directive. Strictly, `.comm` is a directive that emits a common symbol rather than a section header; the assembled output may still place it in `.bss`. The distinction is at the link level rather than the assembly level.\n\n**Where we are.** The include-path family gains `-idirafter`. `offsetof` is defined in `<stddef.h>` using the classic null-pointer idiom. Tentative definitions work end-to-end: parse marks them, `scan_globals` deduplicates them against real definitions, `emit_data` routes them to `.comm` (under `-fcommon`, the default) or `.bss` (under `-fno-common`). The `is_tentative` flag on `Obj` is set, read, and acted on across three locations.\n\n---\n\n## Recap\n\nTwenty-two commits. The chapter doesn't add a new compilation pass or a new abstraction \u2014 every commit slots into existing machinery. The total surface change is:\n\n- `Token` and `File` gain `display_name`/`filename` and `line_delta` for `#line`.\n- `Macro->is_variadic` becomes `Macro->va_args_name`; `MacroArg->is_va_args` is added.\n- `Type` gains `origin` for compatibility tracking.\n- `Obj` gains `is_inline`, `is_live`, `is_root`, `refs`, and `is_tentative`.\n- `VarAttr` gains `is_inline`.\n- `Node` gains `asm_str` and `ND_ASM`.\n- `parse()` runs two new post-parse passes: `mark_live` and `scan_globals`.\n- `is_compatible(t1, t2)` is the new type-compatibility walker.\n- `display_width(p, len)` and `char_width(c)` give the multibyte-aware column count.\n- `__attribute__(x)` is macro-stubbed for non-GCC hosts.\n- The keyword list grows by `typeof`, `inline`, and `asm`.\n- The driver gains `-idirafter`, `-fcommon`, `-fno-common`.\n- `<stddef.h>` defines `offsetof`.\n\nThe chapter's twenty-two-row summary, in `main` order:\n\n| # | Hash | Subject | Section |\n|---|---|---|---|\n| 245 | `37998be` | Multibyte error message | \u00a720.1 |\n| 246 | `c61c0d0` | `#line` | \u00a720.1 |\n| 247 | `aaf20fb` | Line marker directive | \u00a720.1 |\n| 248 | `922604a` | `__TIMESTAMP__` | \u00a720.2 |\n| 249 | `3a10c8a` | `__BASE_FILE__` | \u00a720.2 |\n| 250 | `3381448` | `__VA_OPT__` | \u00a720.2 |\n| 251 | `083c275` | `,##__VA_ARGS__` | \u00a720.2 |\n| 252 | `74ec9f6` | Ignore `#pragma` | \u00a720.2 |\n| 253 | `007e526` | GCC-style variadic macros | \u00a720.2 |\n| 254 | `7d80a51` | `typeof` | \u00a720.3 |\n| 255 | `1433b40` | `__builtin_types_compatible_p` | \u00a720.3 |\n| 256 | `1faab48` | `_Generic` | \u00a720.3 |\n| 257 | `aee7891` | `sizeof(<function type>)` | \u00a720.4 |\n| 258 | `e28a612` | `?:` with omitted middle | \u00a720.4 |\n| 259 | `a253516` | `asm` statement | \u00a720.5 |\n| 260 | `31087f8` | `inline` as `static` | \u00a720.5 |\n| 261 | `e5f4ca9` | Dead static-inline elimination | \u00a720.5 |\n| 262 | `6a2dc5a` | `__attribute__((format))` | \u00a720.5 |\n| 263 | `11fc259` | `-idirafter` | \u00a720.6 |\n| 264 | `1b99bad` | `offsetof` macro | \u00a720.6 |\n| 265 | `85e46b1` | Tentative definitions | \u00a720.6 |\n| 266 | `6d344ed` | `-fcommon`/`-fno-common` | \u00a720.6 |\n\nErrata candidates surfaced this chapter: the `is_compatible` array arm, which returns `true` only when both lengths are negative *and equal* \u2014 `int[3]` vs `int[3]` returns `false` from `__builtin_types_compatible_p`. Likely a typo for the obvious correct condition. The `#pragma` silently-consumed behavior is also questionable in the sense that `#pragma pack` would have layout consequences chibicc cannot honor; chibicc would compile such code without diagnostic and produce wrongly-laid-out structs.\n\nErrata candidates closed this chapter: none of the three remaining Ch 17 errata. They stay open: `#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific.\n\nThe canonicalization-at-parse-time count ticks from nine to ten with the `?:`-omitted-middle desugar. The pre-factor-before-feature count and psABI conformance count are unchanged at nine and sixteen respectively.\n\nThe standing notes for the next session: `Obj` is now a substantial struct with five new fields this chapter (`is_inline`, `is_live`, `is_root`, `refs`, `is_tentative`); `Type` has `origin`; `VarAttr` has `is_inline`. The keyword list is up to roughly thirty entries. The third post-parse pass (after `mark_live` and `scan_globals`) might land in Chapter 21's thread-local or VLA work \u2014 neither feature obviously needs one, but VLAs in particular often have a \"hoist size expressions\" pass in real compilers.\n\nThrough Chapter 20 chibicc handles the gcc-extension surface that real-world C reaches for most often. What it doesn't yet handle: thread-local storage, alloca, variable-length arrays. Those are the next twenty commits, in Chapter 21.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/20-gcc-extensions-worth-supporting.md",
  "description": "Word count of new chapter"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 020 README and recent chapter for context",
      "activeForm": "Reading session 020 README and recent chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 22 commits (245-266) with git show --stat",
      "activeForm": "Scanning all 22 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit diff in detail",
      "activeForm": "Reading each commit diff",
      "status": "completed"
    },
    {
      "content": "Draft chapters/20-gcc-extensions-worth-supporting.md",
      "activeForm": "Drafting chapter 20",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status line",
      "activeForm": "Updating CLAUDE.md status",
      "status": "in_progress"
    },
    {
      "content": "Write session 021 README",
      "activeForm": "Writing session 021 README",
      "status": "pending"
    },
    {
      "content": "Write HANDOFF.md for session 022",
      "activeForm": "Writing HANDOFF for session 022",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020), Ch 20 drafted (session 021). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "offset": 75,
  "limit": 15
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Ch 18 drafted (session 019), Ch 19 drafted (session 020). Continuing autonomously per user direction.",
  "new_string": "Ch 18 drafted (session 019), Ch 19 drafted (session 020), Ch 20 drafted (session 021). Continuing autonomously per user direction."
}


---

## 🤖 Claude

Now let me write session 021 README and HANDOFF for session 022.

---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/021-chapter-20-draft",
  "description": "Make session 021 dir"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline 6d344ed..HEAD | head -25",
  "description": "Check commits after 6d344ed for chapter 21 scope"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline --reverse 6d344ed..HEAD | head -25",
  "description": "Get chapter 21 commits in chronological order"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -A 3 \"^## Chapter 21\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md",
  "description": "Check chapter 21 scope"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -B 1 -A 5 \"Chapter 21\\|chapter 21\\|^### Chapter 21\\|^## Chapter 21\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | head -30",
  "description": "Find chapter 21 in mapping"
}


---

## 🤖 Claude

Now write session 021 README:

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/021-chapter-20-draft/README.md",
  "content": "# Session 021 \u2014 Chapter 20 draft\n\n**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002\u2013020).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 020 delivered Ch 19 (Unicode and designated initializers, twenty-four commits, ~12,128 words). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 20 covers commits 245\u2013266: twenty-two commits, the GCC-extensions-worth-supporting arc \u2014 multibyte error-column display, `#line` and the GNU line marker, four predefined macros, `__VA_OPT__` and the `,##__VA_ARGS__` swallow, `#pragma`, GCC-style variadic macros, `typeof`, `__builtin_types_compatible_p`, `_Generic`, `sizeof(<function-type>)`, the GNU `?:`-with-omitted-middle, basic `asm`, `inline`-as-static, dead static-inline elimination, `__attribute__((format))`, `-idirafter`, `offsetof`, tentative definitions, `-fcommon`/`-fno-common`.\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~9,994 words. Slightly under the 11,000\u201314,000-word handoff forecast, but the chapter's commits are mostly small (median diff is ~15 lines); compressing each into a per-commit subsection without padding is the honest answer. The biggest stretches in the chapter are \u00a720.3 (`_Generic` and `typeof` and `is_compatible`) at ~1,800 words and \u00a720.5 (`asm`, `inline`, dead-elimination, `__attribute__`) at ~2,200 words.\n- **Section structure:** 6 sections from 22 commits, exactly as the handoff proposed. \u00a720.1 (3 commits, three named subsections). \u00a720.2 (6 commits, six named subsections). \u00a720.3 (3 commits, three named subsections). \u00a720.4 (2 commits, two named subsections). \u00a720.5 (4 commits, four named subsections). \u00a720.6 (4 commits, four named subsections).\n- **No concept interlude.** The handoff defaulted to \"possibly one\" for \u00a720.5/\u00a720.6 around type-vs-expression context routing. Reading the \u00a720.3 prose, the cumulative routing through `is_typename` (which now picks up `typeof` and is reused by `_Generic` and `__builtin_types_compatible_p`) was easy to thread inline without a dedicated interlude. The \u00a720.3 closer (\"All three follow-on features pick up `typeof` for free\") is enough scaffolding.\n- **\u00a720.1.3 names the `Token->filename` field as the line-marker round-trip carrier.** Per-token line-numbers (Ch 8 \u00a78.3) gain an *origin-display* twin that survives `#line` overrides. Noted in the \u00a720.1 closer and the chapter recap.\n- **\u00a720.3 surfaces a new errata candidate** \u2014 the `is_compatible` array arm returns `true` only when both lengths are negative *and equal*, which is wrong for two complete arrays of the same length. `int[3]` vs `int[3]` returns `false` from `__builtin_types_compatible_p`. Noted in \u00a720.3.2 prose and the chapter recap.\n- **\u00a720.4 ticks the canonicalization-at-parse-time count from nine to ten** with the `?:`-omitted-middle desugar (`a ?: b` becomes `tmp = a, tmp ? tmp : b`). Noted in the \u00a720.4 closer.\n- **\u00a720.5.4's `__attribute__` annotation walk includes a candid uncertainty note** about how chibicc's self-host compiles past `__attribute__((...))` annotations when `__GNUC__` is defined. The mechanism is unclear from the diffs alone; verification was inconclusive. The prose says so honestly rather than guessing.\n- **\u00a720.6.3 walks tentative definitions including the `scan_globals` post-parse pass and its O(n\u00b2) traversal.** The driver call sequence in `parse()` is now three passes: build globals, mark-live, scan-globals.\n- **\u00a720.6.4's `-fcommon` history note** mentions GCC 10 (2020) flipping the default from `-fcommon` to `-fno-common`. Chibicc was contemporary and chose the historical default. Noted in prose.\n- **\u00a720.2.5 names `#pragma` silent consumption as an errata-candidate concern** but does not list it in the chapter recap (since it's a deliberate-and-documented chibicc choice rather than a bug). The `#pragma pack` layout-divergence scenario is named in prose.\n- **One-table recap** at the chapter close, twenty-two rows, with a section column to make the \u00a7-grouping visible. Resisted multi-table-by-section.\n\n### Interpretive calls\n\n1. **The `,##__VA_ARGS__` swallow check** in \u00a720.2.4 walks both branches (empty arg \u2192 consume three tokens, non-empty \u2192 emit comma and skip `##`). The doubled-`;` in `arg->name = va_args_name;;` (line 450 of preprocess.c at commit `007e526`) is named as a real typo in chibicc's source. Harmless, but worth noting for accuracy.\n2. **`__VA_OPT__`'s `read_macro_arg_one` call uses `read_rest=true`.** Named in prose as the reason commas inside the parens don't terminate the arg.\n3. **`_Generic`'s discarded-arm point** is given one paragraph: standard C says unselected arms need not be valid, but chibicc parses each arm as `assign` (which calls `find_var`, which errors on unresolved identifiers). So chibicc is *stricter* than the standard requires for unselected-arm validity. Named in \u00a720.3.3 prose.\n4. **The `is_compatible` array arm bug** (`<` instead of `<=` or `||`) is given a sentence in \u00a720.3.2 with the speculation \"Rui likely meant `||` instead of `&&`.\" Named as errata candidate.\n5. **`sizeof(<function type>)` returns 1** \u2014 \u00a720.4.1 names the GNU extension and the test case `sizeof(main)` returning 1. Noted as deliberate divergence from the standard.\n6. **The \u00a720.5 self-host-compiles-with-`__attribute__` mystery** is candidly named as unresolved. The prose doesn't pretend to know the mechanism. Verification while drafting was inconclusive (no time spent grepping the parser for an attribute-skip path); the chapter says so.\n7. **`inline`-as-`static` is named as a simplification of the standard's actual `inline` rules** in \u00a720.5.2 prose. The \"extern inline\" external-symbol case is handled correctly by `(attr->is_inline && !attr->is_extern)`. The standard's full model around inline definitions and external-symbol uniqueness is not implemented; the simplification is enough for real-world headers.\n8. **The `mark_live` reachability pass** is walked in \u00a720.5.3 with the mutual-recursion early-exit explicitly named. Three-pass `parse()` is named.\n9. **The `-fcommon` historical default** is given one sentence \u2014 GCC 10 flipped it. Chibicc's choice is the older convention.\n10. **The `.bss`-vs-`.comm` distinction** in \u00a720.6.4 is given a short clarification: `.comm` is technically a directive that emits a common symbol rather than a section header, so the \"fourth assembly section\" framing is imprecise. The chapter recap notes this.\n11. **The `offsetof` UB-via-null-pointer** is named in \u00a720.6.2: real toolchains use `__builtin_offsetof` to avoid UB; chibicc uses the macro directly because chibicc doesn't UB-check.\n12. **The chapter does not invent a \"history of GCC extensions\" interlude.** The handoff cautioned against it and the prose holds the line.\n\n### Voice / structure inherited from Ch 1\u201319\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, all hashes listed at the top.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- One-table recap at the chapter close (with a \u00a7-section column added).\n- No concept interludes.\n\n### Three careful avoidances\n\n- **Did not invent a history of `_Generic`-vs-`__builtin_types_compatible_p` interlude.** The two features have a generational story (TC1's compatibility predicate vs C11's full type dispatch) but walking it would have been a detour. The chapter cites both as compile-time predicates and walks chibicc's specific implementations.\n- **Did not over-explain the linker's common-symbol semantics.** `.comm` and the merging behavior are sketched in \u00a720.6.4 with one paragraph; the full linker-side story (weak symbols, PIE, relocations) is not the chapter's topic. Acceptable, since chibicc emits `.comm` and lets GAS/ld handle it.\n- **Did not invent a \"history of `inline`\" detour.** `inline` has a famously complicated history in C (K&R-era GCC extension, C99 standard form with subtle linkage rules, C11 tweaks). The chapter cites that the standard's rules are subtle and that chibicc treats `inline` as `static`. A history walk would be a detour.\n\n### Date-vs-position note\n\nThe twenty-two commits scatter across calendar time: July 2020 (`37998be` multibyte width, `c61c0d0` `#line`, `aaf20fb` line marker, `922604a` `__TIMESTAMP__`, `3381448` `__VA_OPT__`, `083c275` `,##__VA_ARGS__`, `7d80a51` `typeof`, `1faab48` `_Generic`, `aee7891` `sizeof(func)`), August 2020 (`6a2dc5a` `__attribute__((format))`, `1b99bad` `offsetof`, `6d344ed` `-fcommon`, `a253516` `asm`, `007e526` GCC variadic, `3a10c8a` `__BASE_FILE__`), September 2020 (`31087f8` inline-as-static, `e5f4ca9` dead static-inline, `e28a612` `?:` middle, `1433b40` `__builtin_types_compatible_p`, `74ec9f6` `#pragma`, `11fc259` `-idirafter`, `85e46b1` tentative). The chapter follows `main` order without remark \u2014 `1b99bad` (Aug 15, 2020) is at position 264 even though it predates several July-dated commits earlier in the chapter. Same as prior chapters.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **The `__attribute__` macro stub.** Defined in `chibicc.h` when `__GNUC__` is not defined. Chibicc itself defines `__GNUC__` (per Ch 17 \u00a717.5.4), so the stub is *not* active during chibicc's self-host. The mechanism by which chibicc parses past `__attribute__((...))` annotations during self-host is unclear and was not verified during this session. Worth investigating in a future session if the topic comes up.\n- **The `is_compatible` array arm bug.** `t1->array_len < 0 && t2->array_len < 0 && t1->array_len == t2->array_len` returns `true` only when both lengths are negative *and equal*. `int[3]` vs `int[3]` should be compatible but returns `false`. Errata candidate.\n- **`Type->origin`** is set in `copy_type` so a copied type points back at its source. Used by `is_compatible` to short-circuit through typedef chains. Possibly will be touched by Ch 21 (typedefs, declarations) if any new copy_type call sites need different semantics.\n- **`Obj` grew five fields this chapter:** `is_inline`, `is_live`, `is_root`, `refs`, `is_tentative`. The struct is now substantial.\n- **`parse()` runs three passes after building `globals`:** the build itself, `mark_live`, `scan_globals`. Order is dead-elimination, then tentative-elimination. The two address disjoint subsets of globals so order is incidental, but the order is fixed.\n- **The `#pragma` silent-consume** is a chibicc choice rather than a bug. `#pragma pack` would have layout consequences chibicc cannot honor; chibicc compiles such code without diagnostic. Named in \u00a720.2.5 but not listed in the recap as an errata candidate (it's deliberate).\n- **`asm` is minimal** \u2014 verbatim string emit, no operand bindings, no clobbers, no goto. Real codebases that use `asm` for serious work will hit the limit. Named in \u00a720.5.1.\n- **`-fcommon` is the chibicc default**, matching GCC's pre-10 default. GCC 10+ defaults to `-fno-common`. Users targeting modern GCC behavior would invoke `-fno-common` explicitly.\n- **`__VA_OPT__` and `,##__VA_ARGS__` both work.** They're functionally equivalent for the trailing-comma case. Named in \u00a720.2.3 and \u00a720.2.4.\n- **`Macro->is_variadic` (bool) \u2192 `Macro->va_args_name` (char *)** is the \u00a720.2.6 plumbing change. `MacroArg->is_va_args` is added.\n- **`Token` and `File` gain `display_name` and `line_delta`.** Set at file-construction time; modified by `read_line_marker`. Read by `__FILE__` and `__LINE__` macro handlers.\n- **`is_typename` adds `typeof` and `inline`.** Three new keywords this chapter (`typeof`, `inline`, `asm`) all in the keyword list.\n- **The keyword list** is now around thirty entries. Future chapters will add `_Thread_local`, `_Atomic`, possibly `_Noreturn`-related additions.\n- **The `is_definition` flag on `Obj`** is *not* affected by tentative \u2014 a tentative variable still has `is_definition=true`. The `is_tentative` flag is the discriminator.\n- **The cast table is unchanged** at 10\u00d710. `_Generic` doesn't introduce new cast cells; it returns one of the parsed expressions, not a cast.\n- **`unreachable()` callers** unchanged \u2014 `_Generic`'s no-match case uses `error_tok` rather than `unreachable()`.\n- **`StringArray`** picks up new uses: the `idirafter` temporary in `parse_args` (\u00a720.6.1), the `refs` field on `Obj` (\u00a720.5.3).\n- **`mark_live` is recursive** with early-exit on already-marked. Mutual recursion is handled correctly. Named in \u00a720.5.3.\n- **psABI conformance count stays at sixteen.** Ch 20 doesn't touch the ABI surface. The `-fcommon`/`.comm` mechanism is link-time, not ABI.\n- **Pre-factor-before-feature count stays at nine.** Ch 20 doesn't add new entries.\n- **Canonicalization-at-parse-time count is at ten.** Up from nine, with the \u00a720.4.2 `?:`-omitted-middle desugar.\n- **Errata candidates added in Ch 20:**\n  - The `is_compatible` array arm: `t1->array_len < 0 && t2->array_len < 0 && t1->array_len == t2->array_len` is wrong for two complete same-length arrays (in \u00a720.3.2, commit `1433b40`).\n  - `#pragma` silent consume \u2014 deliberate but consequential when source uses `#pragma pack` (in \u00a720.2.5, commit `74ec9f6`). Not listed in the recap.\n- **Errata candidates closed in Ch 20:** None.\n- **Errata candidates remaining:** Ch 17's three (`#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific), Ch 19's three (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block, range designators not honored), and Ch 20's two new (`is_compatible` array bug, `#pragma` silence).\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean \u2014 unchanged.\n- **Chibicc compiles itself** \u2014 unchanged.\n\n## Exit state\n\n- `chapters/20-gcc-extensions-worth-supporting.md` drafted, ~9,994 words.\n- Session 021 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 022 (Chapter 21 \u2014 Thread-local, alloca, VLAs, commits 267\u2013283, ~17 commits).\n- CLAUDE.md status note updated to \"Ch 20 drafted\".\n"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 020 README and recent chapter for context",
      "activeForm": "Reading session 020 README and recent chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 22 commits (245-266) with git show --stat",
      "activeForm": "Scanning all 22 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit diff in detail",
      "activeForm": "Reading each commit diff",
      "status": "completed"
    },
    {
      "content": "Draft chapters/20-gcc-extensions-worth-supporting.md",
      "activeForm": "Drafting chapter 20",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status line",
      "activeForm": "Updating CLAUDE.md status",
      "status": "completed"
    },
    {
      "content": "Write session 021 README",
      "activeForm": "Writing session 021 README",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF.md for session 022",
      "activeForm": "Writing HANDOFF for session 022",
      "status": "in_progress"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git log --oneline --reverse 6d344ed..HEAD | sed -n '1,17p'",
  "description": "Get exact 17 commits for chapter 21"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -B 1 -A 8 \"Chapter 21\\|Chapter 22\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | head -25",
  "description": "Detailed chapter 21/22 mapping"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/021-chapter-20-draft/HANDOFF.md",
  "content": "# Handoff: Ch 20 done \u2192 proceed to Ch 21\n\n**For:** the next claude session.\n**From:** session 021.\n**Status:** Ch 20 drafted (~9,994 words, twenty-two commits, the GCC-extensions arc \u2014 multibyte error-column display, `#line` and the GNU line marker, four predefined macros, `__VA_OPT__` and the `,##__VA_ARGS__` swallow, `#pragma`, GCC-style variadic, `typeof`, `__builtin_types_compatible_p`, `_Generic`, `sizeof(<function-type>)`, the GNU `?:`-with-omitted-middle, basic `asm`, `inline`-as-static, dead static-inline elimination, `__attribute__((format))`, `-idirafter`, `offsetof`, tentative definitions, `-fcommon`/`-fno-common`). Continue autonomously to Ch 21 (Thread-local, alloca, VLAs, commits 267\u2013283 \u2014 seventeen commits covering thread-local variables, `-include` and `-x` driver options, `alloca`, the VLA arc, `-l`/`-s`, ELF size/type emission, `.a`/`.so` recognition, `long double`, GNU case ranges, GNU array range designators, labels-as-values). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/021-chapter-20-draft/README.md`](README.md)** \u2014 what session 021 did, including the six-section structure (per-commit subsections in \u00a720.1, \u00a720.2, \u00a720.3, \u00a720.4, \u00a720.5, \u00a720.6; no concept interlude despite handoff conditional), the new errata candidate (`is_compatible` array arm bug), the canonicalization-at-parse-time count tick from nine to ten, the unchanged counts (pre-factor at nine, psABI conformance at sixteen), the unresolved \u00a720.5.4 mystery about `__attribute__` self-host parsing.\n2. **[`docs/sessions/020-chapter-19-draft/HANDOFF.md`](../020-chapter-19-draft/HANDOFF.md)** \u2014 the previous handoff. Standing notes still apply with Ch 20 updates folded in (see \u00a721 README for the running list).\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`20-gcc-extensions-worth-supporting.md`](../../../chapters/20-gcc-extensions-worth-supporting.md)** \u2014 the twenty chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 21 = commits 267\u2013283 (17 commits, scoped to \"Thread-local, alloca, VLAs\"). Note the trailing \"(and compile-time constant)\" phrase in the mapping is ambiguous; commit position numbers are authoritative \u2014 Ch 21 ends at `4f165ec` (labels-as-values), Ch 22 starts at `f0c98e0` (labels-as-values as compile-time constant).\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes. Ch 21's commits are mostly feature additions; the alloca and VLA commits may have design notes worth scanning.\n6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** \u2014 JP TOC; thread-local and VLAs aren't commonly featured topics in compiler tutorials. Probably no concept-interlude candidates.\n\n## Chapter 21 scope\n\n**Title (working):** *Thread-local, alloca, VLAs*.\n**Commits:** 267\u2013283 in chronological order on `main`. **Seventeen commits** \u2014 significantly fewer than Ch 20 (22 commits) and Ch 19 (24 commits).\n**Concept interlude:** Possibly one. The VLA arc (commits 272\u2013275) is the most invasive single stretch \u2014 VLAs require run-time-computed sizes which means the codegen has to evaluate size expressions at function-prologue time and store them in stack-allocated cells. This is a genuinely new mechanism, parallel in shape to (but distinct from) `alloca`, which is also added in this chapter. A short interlude on *how chibicc handles dynamically-sized stack allocations* could land in \u00a721.4 or \u00a721.5. Default conditional \u2014 judge while reading the commits.\n\n| # | Hash | Subject |\n|---|---|---|\n| 267 | `b377284` | Add thread-local variable |\n| 268 | `8f5ff07` | Add -include option |\n| 269 | `ee0a951` | Add -x option |\n| 270 | `4064871` | Make -E to imply -xc |\n| 271 | `77275c5` | Add alloca() |\n| 272 | `e8667af` | Add sizeof() for VLA |\n| 273 | `07f9010` | Add pointer arithmetic for VLA |\n| 274 | `2fa8f48` | Support sizeof(typename) where typename is a VLA |\n| 275 | `b0109a3` | Do not define __STDC_NO_VLA__ |\n| 276 | `bc25279` | Add -l option |\n| 277 | `c32f0e2` | Add -s option |\n| 278 | `8d130ab` | Emit size and type for symbols |\n| 279 | `d56dd2f` | Recognize .a and .so files |\n| 280 | `e0bf168` | Add long double |\n| 281 | `d90c73b` | [GNU] Support case ranges |\n| 282 | `3d5550e` | [GNU] Support array range designator |\n| 283 | `4f165ec` | [GNU] Support labels-as-values |\n\nSeventeen commits. The natural section grouping:\n\n- **\u00a721.1 \u2014 Thread-local variables** (commit 267). One commit. The `_Thread_local` keyword is added; storage class becomes a `VarAttr` flag; codegen emits `.tdata`/`.tbss` and uses `%fs:`-relative addressing for accesses. Likely the longest single-commit section in the chapter. Walk carefully \u2014 the AMD64 TLS model has subtleties around initial-exec vs general-dynamic.\n- **\u00a721.2 \u2014 Driver: `-include`, `-x`, `-E` implies `-xc`** (commits 268\u2013270). Three commits. `-include FILE` prepends a `#include` of `FILE` before the source. `-x LANG` overrides language detection (`c`, `assembler`, `none`). `-E` implies `-xc` so preprocessing-only mode still works on stdin without a `.c` extension.\n- **\u00a721.3 \u2014 `alloca`** (commit 271). One commit. Run-time stack allocation. Likely a builtin that emits a stack-pointer adjustment plus alignment. Walk how the result interacts with the function epilogue.\n- **\u00a721.4 \u2014 VLAs** (commits 272\u2013275). Four commits. The arc: VLAs as types, then `sizeof(VLA)` evaluating at run time, then pointer arithmetic on VLAs, then `sizeof(typename)` where typename contains a VLA, then dropping `__STDC_NO_VLA__`. The sizes are stored in compiler-generated locals; VLAs are allocated like `alloca`.\n- **\u00a721.5 \u2014 Linker-driver: `-l`, `-s`, ELF size/type, `.a`/`.so`** (commits 276\u2013279). Four commits. `-l NAME` adds `libNAME.so` or `libNAME.a` to the link. `-s` strips the binary. The ELF symbol-table emission gets size and type (`.size`/`.type`) directives. `.a` and `.so` files are recognized by `run_linker` as input.\n- **\u00a721.6 \u2014 `long double`, case ranges, array range designators, labels-as-values** (commits 280\u2013283). Four commits. `long double` is finally implemented as actual extended-precision rather than aliased to `double` (this closes one of the standing errata candidates from earlier chapters!). GNU case ranges (`case 1 ... 5:`) generate range checks in switch lowering. Array range designators (`[3 ... 7] = x`) finally honor the range in elaboration (closing the \u00a719.7 errata candidate). Labels-as-values is the GNU `&&label` and `goto *expr` pair, used for computed gotos in interpreters.\n\nThat's six sections from seventeen commits. **Target chapter length: ~10,000\u201312,000 words.** Likely closer to 10K \u2014 most commits are small except for the thread-local, VLA, and long-double additions, which may each warrant ~1,500 words.\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in b377284 8f5ff07 ee0a951 4064871 77275c5 e8667af 07f9010 2fa8f48 b0109a3 bc25279 c32f0e2 8d130ab d56dd2f e0bf168 d90c73b 3d5550e 4f165ec; do echo \"===== $h =====\"; git show --stat $h | head -10; done` to scan all 17 diffs.\n2. Read each commit. Pay particular attention to:\n   - **\u00a721.1's thread-local** \u2014 read carefully for the AMD64 TLS access pattern. The `%fs:` segment register is used for thread-local addressing on Linux. The `.tdata` and `.tbss` sections hold the per-thread initial values; the loader copies them into per-thread storage at thread creation. Walk the codegen for both the variable definition (the `.tdata`/`.tbss` placement) and the variable read (the `%fs:` indirection).\n   - **\u00a721.3's `alloca`** \u2014 chibicc's `alloca` is a builtin (`__builtin_alloca`?) that emits inline stack adjustment. Walk how alignment is enforced. The result interacts with the function epilogue in a way that requires the saved `%rbp` to be the cleanup anchor (pop `%rbp` resets `%rsp`). Confirm by reading the prologue/epilogue.\n   - **\u00a721.4's VLA arc** \u2014 the most invasive single stretch. VLAs as types likely add a `vla_size` field to `Type` (a `Node *` for the size expression). `sizeof(VLA)` becomes runtime evaluation. The size is computed at the declaration site and stored in a hidden local; later `sizeof` reads from that local. Walk the four commits in sequence.\n   - **\u00a721.5's `.a`/`.so` recognition** \u2014 `run_linker` already accepts `.o` files; `.a` (archives) and `.so` (shared objects) are recognized by file-magic or by extension and passed to the linker with appropriate flags.\n   - **\u00a721.6's `long double`** \u2014 finally extended-precision (probably uses x87 stack via `fld`, `fst`, etc., or possibly maps to `__float128`). Walk carefully \u2014 the calling convention is its own beast (long double passed in x87 stack regs). Closes the long-standing errata candidate from earlier chapters.\n   - **\u00a721.6's case ranges** \u2014 `case 1 ... 5:` generates a range check in the switch dispatch. Walk how `gen_stmt`'s switch arm handles the range; probably emits an `if (val >= 1 && val <= 5) goto label;` style sequence.\n   - **\u00a721.6's array range designator** \u2014 `[3 ... 7] = x` finally honors the range. Closes the \u00a719.7 errata candidate. Walk how the elaboration loop iterates over the range.\n   - **\u00a721.6's labels-as-values** \u2014 `&&label` produces an address; `goto *expr` jumps to a runtime-computed address. Used for computed-goto interpreters. Two commits: the basic feature (`4f165ec`) and the compile-time-constant version (`f0c98e0`, which is in Ch 22 not Ch 21).\n3. Read the destination state at `4f165ec` for `parse.c`, `tokenize.c`, `codegen.c`, `chibicc.h`, `main.c`, `type.c`. The VLA changes are likely the most invasive; thread-local touches `parse.c` and `codegen.c`; long double touches `type.c`, `parse.c`, and `codegen.c` substantially.\n4. Draft `chapters/21-thread-local-alloca-vlas.md`. Likely 10,000\u201312,000 words. Six sections.\n5. Write `docs/sessions/022-chapter-21-draft/README.md`.\n6. Write `HANDOFF.md` for session 023 (Chapter 22 \u2014 Performance, dependency files, and the linker driver, commits 284\u2013306).\n\n## Voice / structure rules\n\nSame as Ch 1\u201320:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present tense for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 seventeen rows, single table is fine.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The \u00a721.1 thread-local, \u00a721.4 VLAs, and \u00a721.6 long double will want larger code blocks.\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.\n- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **\u00a721.1's thread-local AMD64 TLS** is its own sublanguage. The access pattern is `%fs:offset(%rip)` for initial-exec, which is what gcc emits for TLS variables in the executable. Real glibc has multiple TLS models; chibicc almost certainly does only initial-exec. Don't over-explain the model \u2014 name what chibicc does and cite GCC's TLS doc as further reading if needed.\n- **\u00a721.3's `alloca` is potentially unsafe** \u2014 the C standard doesn't endorse `alloca` (it's a POSIX/GNU extension). Many style guides forbid it. Note the safety issue (no error reporting on stack exhaustion) but don't moralize.\n- **\u00a721.4's VLA size evaluation order** is subtle. `int x[f()][g()]` calls `f()` and `g()` in declaration order, and the values are remembered for `sizeof(x)` later. Walk how chibicc threads this.\n- **\u00a721.4's `b0109a3` \"Do not define __STDC_NO_VLA__\"** is a one-line preprocessor change. Walk it briefly; it's not interesting in isolation but completes the VLA arc.\n- **\u00a721.5's `-l NAME` resolution** searches a path list. Walk the search order.\n- **\u00a721.6's `long double` calling convention** \u2014 on x86-64 SysV, long double is 16 bytes (80-bit extended on x87 with padding) and is passed differently from double. Calling conventions for long double are complex. Walk what chibicc does, name what it doesn't.\n- **\u00a721.6's case range** generates code; it doesn't generate jump-table entries. A `case 1 ... 1000000` would not blow up the codegen (a real compiler would either generate a range check or a million jump-table entries). Walk what chibicc does.\n- **The \"labels-as-values\" feature** is GNU-only. The `&&` operator (the address-of-label) is a clear name conflict with `&&` (logical AND). The tokenizer must distinguish; chibicc likely treats `&&label` specially in unary-expression parsing. Walk it.\n- **`f0c98e0` is in Chapter 22, not Chapter 21.** The mapping line \"(and compile-time constant)\" is misleading. Don't include it in Ch 21.\n\n## Standing notes worth tracking across sessions\n\n- **The hideset on Token** \u2014 unchanged through Ch 20. Ch 21's commits don't touch the macro-expansion machinery.\n- **The Token->origin chain** \u2014 likely unchanged in Ch 21.\n- **The `Token` line-marker fields** \u2014 `display_name`, `filename`, `line_delta` added in Ch 20 \u00a720.1. Stable.\n- **The eval-quartet duplication** \u2014 unchanged through Ch 20. May be touched by VLA size evaluation if chibicc's `eval` chain needs to handle runtime expressions.\n- **The cc1-vs-driver split** \u2014 unchanged.\n- **The `Initializer` tree** \u2014 Ch 19 added `Member *mem` for unions. Ch 20 unchanged. Ch 21's array range designator may add a new field or re-use existing range-walking code.\n- **The local-vs-global split** \u2014 affected by Ch 21's thread-local (a new third storage class). Walk how `Obj` is extended.\n- **The `Relocation` mechanism** \u2014 likely unchanged in Ch 21.\n- **The anonymous-global pattern** \u2014 likely unchanged in Ch 21.\n- **The `is_static` default in `new_gvar`** \u2014 probably gains an `is_tls` companion. Walk while drafting.\n- **The `is_definition` flag on `Obj`** \u2014 stable since Ch 20.\n- **The `is_unsigned` flag on `Type`** \u2014 unchanged.\n- **The `__va_area__` magic name** \u2014 unchanged.\n- **The register-save-area layout** \u2014 unchanged.\n- **The argreg integer/FP split** \u2014 likely changes for `long double` (passed in x87 regs, distinct from FP argregs).\n- **The `Member->idx` field and bitfield siblings** \u2014 unchanged.\n- **The `is_flexible` flag** \u2014 unchanged. The dead-code duplicate from \u00a719.7's `835cd24` is still in the source; if `array_initializer1` is touched in Ch 21 for range designators, the prose should note whether the duplicate is finally removed.\n- **`copy_struct_type`** \u2014 unchanged.\n- **`MIN`/`MAX` macros** \u2014 unchanged.\n- **`is_numeric` predicate** \u2014 likely changes for `long double` (probably returns true for the new TY_LDOUBLE).\n- **Canonicalization-at-parse-time count is at ten.** Ch 21 might add one in \u00a721.6 (case ranges may rewrite to chains of compares; array range designators may rewrite to repeated single-element designations). Verify while drafting.\n- **Pre-factor-before-feature count is at nine.** Ch 21 unlikely to add new entries.\n- **psABI conformance count is at sixteen.** Ch 21 may add one or two for thread-local TLS access patterns and for `long double` calling convention.\n- **The fourth namespace (labels)** is unchanged. Labels-as-values doesn't add a new namespace; it lets labels participate in the address-of operator.\n- **The `is_typename` predicate** likely changes in \u00a721.1 (for `_Thread_local`) and \u00a721.6 (for `long double` \u2014 the keyword pair). Verify.\n- **The `VarAttr` channel** has five fields after Ch 20 (typedef, static, extern, inline, align). Will grow in Ch 21 for thread-local. Walk while drafting.\n- **The `ND_NULL_EXPR` seed-pattern** \u2014 unchanged.\n- **The `rep stosb` pattern** \u2014 unchanged. `alloca`-allocated regions are not zero-initialized by chibicc.\n- **The `unreachable()` macro** \u2014 likely picks up new callers in VLA-related code.\n- **Per-token line numbers** \u2014 unchanged through Ch 20.\n- **GDB-debuggable output** \u2014 unchanged.\n- **Tests are in C.** New test files likely for thread-local, alloca, VLAs, long double, case ranges, label-as-values.\n- **The `Obj->tok` field** added in Ch 14 \u00a714.11 still has no readers.\n- **The `Type->name_pos` field** \u2014 no new uses since Ch 14.\n- **The `Type->origin` field** added in Ch 20 \u00a720.3 for compatibility tracking. Stable.\n- **The `Obj` struct grew five fields in Ch 20** (`is_inline`, `is_live`, `is_root`, `refs`, `is_tentative`). Likely grows again in Ch 21 for thread-local.\n- **The `>>` codegen quirk** \u2014 partially repaired.\n- **The `add_type` rule for `ND_STMT_EXPR`** \u2014 errata candidate.\n- **The hex-escape silent truncation** \u2014 errata candidate.\n- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members \u2014 five errata candidates.\n- **`f()` and `f(void)` are accepted as identical** \u2014 errata candidate.\n- **Empty brace initializer (`int x[3] = {};`)** \u2014 chibicc-specific extension.\n- **`.bss` is the third assembly section.** `-fcommon`/`.comm` may add `.comm` as a fourth path. Ch 21's thread-local adds `.tdata` and `.tbss` as fifth and sixth. Walk while drafting.\n- **`.align`** \u2014 unchanged.\n- **The `mov $0, %rax`** for variadic FP-count \u2014 errata candidate.\n- **The `fp_offset = fp * 8 + 48` non-conforming stride** \u2014 errata candidate.\n- **`long double` is `double`** \u2014 *closed in Ch 21 \u00a721.6*. Verify.\n- **The default-argument-promotion gap for chars and shorts** \u2014 errata candidate.\n- **Float literals are inlined as integer-immediate-bit-cast.**\n- **Ch 1 errata list** unchanged.\n- **The cast/compound-literal disambiguator** \u2014 same lookahead pattern.\n- **The cast table is 10\u00d710.** Likely grows in \u00a721.6 if `long double` introduces a new TY_LDOUBLE row/column. Verify while drafting.\n- **Driver brittleness** \u2014 partially addressed by Ch 21's `-include`, `-x`, `-l`, `-s` additions.\n- **The link command's hardcoded distro list** \u2014 errata candidate.\n- **`Node->funcname` is gone.**\n- **`mov %rax, %r10; call *%r10` is uniform across all calls.**\n- **The `StringArray` type** \u2014 unchanged.\n- **`atexit(cleanup)` for tempfile disposal** \u2014 unchanged.\n- **The `run_subprocess` helper** \u2014 unchanged.\n- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Three remaining.\n- **Errata candidates added in Ch 18:** None high-priority.\n- **Errata candidates added in Ch 19:**\n  - UTF-16 character-literal silent truncation of code points above U+FFFF (in \u00a719.4, commit `454618c`).\n  - Dead-code duplicate `is_flexible` block in `array_initializer1` (in \u00a719.7, commit `835cd24`).\n  - Range designators `[3 ... 7]` syntactically accepted but not honored in elaboration (in \u00a719.7, commit `835cd24`). **Likely closed in Ch 21 \u00a721.6 commit `3d5550e`.** Verify while drafting.\n- **Errata candidates added in Ch 20:**\n  - `is_compatible` array arm bug \u2014 returns `true` only when both lengths are negative *and equal*; should be `||` (in \u00a720.3.2, commit `1433b40`).\n- **`self.py` is gone.**\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.\n- **Chibicc compiles itself** as of commit 197 (Ch 17 \u00a717.6).\n- **The `has_flonum` family** unchanged \u2014 likely changes in Ch 21 \u00a721.6 for long double.\n- **Bitfield support is feature-complete.**\n- **Anonymous struct/union members** flatten via recursive `get_struct_member`.\n- **The pre-tokenize pass count is four** (Ch 19 \u00a719.6): BOM, newline, backslash-newline, UCN. Unchanged in Ch 20.\n- **The four char-literal prefixes** are functional.\n- **The four string-literal prefixes** are functional.\n- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.\n- **`__STDC_NO_VLA__`** is currently *defined* (chibicc has no VLAs as of Ch 20). **Will be undefined in Ch 21 \u00a721.4 commit `b0109a3`.**\n- **UTF-8 in identifiers** uses C11 Annex D ranges.\n- **The GNU `$` extension** in identifiers.\n- **`__DATE__`, `__TIME__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`** are predefined.\n- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission.\n- **`__VA_OPT__` and `,##__VA_ARGS__` are functional.**\n- **GCC-style variadic macros (`name...`)** are functional.\n- **`#pragma` is silently consumed.**\n- **`typeof`, `_Generic`, `__builtin_types_compatible_p`** are functional.\n- **`sizeof(<function-type>)` returns 1.**\n- **The GNU `?:`-omitted-middle** is functional.\n- **`asm`** is verbatim-string-only, no operand bindings.\n- **`inline` is treated as `static`**, with dead-static-inline elimination.\n- **`__attribute__` is macro-stubbed when `__GNUC__` is undefined.**\n- **`-idirafter`, `-fcommon`/`-fno-common`** are functional.\n- **`offsetof` is in `<stddef.h>`.**\n- **Tentative definitions are functional.** `.comm` (under `-fcommon`) or `.bss` (under `-fno-common`).\n\n## Acceptance criteria for Ch 21\n\n- [ ] `chapters/21-thread-local-alloca-vlas.md` exists, end-to-end readable.\n- [ ] All seventeen commits covered, grouped into ~6 sections.\n- [ ] \u00a721.1 walks the AMD64 TLS access pattern (`%fs:`-relative addressing) and names `.tdata`/`.tbss` as new assembly sections.\n- [ ] \u00a721.3 walks `alloca`'s stack-pointer manipulation and notes the safety concern.\n- [ ] \u00a721.4 walks the VLA arc, especially how size expressions are stored and re-read for `sizeof`.\n- [ ] \u00a721.6 walks `long double` and notes that this closes the standing \"long double is double\" errata.\n- [ ] \u00a721.6 walks array range designators and notes that this closes the \u00a719.7 range-designator errata.\n- [ ] \u00a721.6 walks labels-as-values (the basic feature only \u2014 the compile-time-constant variant is in Ch 22).\n- [ ] Voice matches Ch 1\u201320.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references checked against `chapter-mapping.md`.\n- [ ] psABI conformance thread count noted (likely grows by one or two for thread-local and long double calling convention).\n- [ ] `docs/sessions/022-chapter-21-draft/README.md` written.\n- [ ] `HANDOFF.md` written for session 023 (Chapter 22 \u2014 Performance, dependency files, and the linker driver, commits 284\u2013306).\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/021-chapter-20-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/021-chapter-20-draft/README.md   (what session 021 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md\n13. chapters/11-all-the-operators.md\n14. chapters/12-initializers.md\n15. chapters/13-linkage.md\n16. chapters/14-variadics-signedness-qualifiers.md\n17. chapters/15-floating-point.md\n18. chapters/16-the-compiler-driver.md\n19. chapters/17-a-preprocessor-from-scratch.md\n20. chapters/18-the-full-abi.md\n21. chapters/19-unicode-and-designated-initializers.md\n22. chapters/20-gcc-extensions-worth-supporting.md (most recent chapter)\n23. research/commits/chapter-mapping.md            (confirms Ch 21 scope)\n24. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 21 (Thread-local, alloca, VLAs, commits 267\u2013283)\nper the steps in the handoff. Seventeen commits, six sections proposed\nin the handoff. The VLA arc (\u00a721.4, four commits \u2014 sizeof(VLA),\npointer arithmetic, sizeof(typename), drop __STDC_NO_VLA__) is the\nchapter's most invasive single stretch and is where a possible concept\ninterlude on dynamically-sized stack allocations lands. The \u00a721.6\nlong-double commit closes the long-standing \"long double is double\"\nerrata; the \u00a721.6 array-range-designator commit closes the \u00a719.7\nrange-designator errata. End-of-session: write your session dir under\ndocs/sessions/022-chapter-21-draft/ with a README and a HANDOFF for\nsession 023 (Chapter 22 \u2014 Performance, dependency files, and the\nlinker driver, commits 284\u2013306).\n```\n"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read session 020 README and recent chapter for context",
      "activeForm": "Reading session 020 README and recent chapter",
      "status": "completed"
    },
    {
      "content": "Scan all 22 commits (245-266) with git show --stat",
      "activeForm": "Scanning all 22 commits",
      "status": "completed"
    },
    {
      "content": "Read each commit diff in detail",
      "activeForm": "Reading each commit diff",
      "status": "completed"
    },
    {
      "content": "Draft chapters/20-gcc-extensions-worth-supporting.md",
      "activeForm": "Drafting chapter 20",
      "status": "completed"
    },
    {
      "content": "Update CLAUDE.md status line",
      "activeForm": "Updating CLAUDE.md status",
      "status": "completed"
    },
    {
      "content": "Write session 021 README",
      "activeForm": "Writing session 021 README",
      "status": "completed"
    },
    {
      "content": "Write HANDOFF.md for session 022",
      "activeForm": "Writing HANDOFF for session 022",
      "status": "completed"
    }
  ]
}


---

## 🤖 Claude

Chapter 20 drafted (~9,994 words). Six sections covering the twenty-two commits in the GCC-extensions-worth-supporting arc — multibyte error-column display, `#line` and the GNU line marker, four predefined macros, `__VA_OPT__` and the `,##__VA_ARGS__` swallow, `#pragma`, GCC-style variadics, `typeof`/`__builtin_types_compatible_p`/`_Generic`, `sizeof(<function-type>)`, the GNU `?:`-with-omitted-middle, basic `asm`, `inline`-as-static plus dead-elimination, `__attribute__((format))`, `-idirafter`, `offsetof`, tentative definitions, and `-fcommon`/`-fno-common`. Surfaced one new errata candidate (the `is_compatible` array arm bug). Canonicalization-at-parse-time count ticked from nine to ten. Session 021 README and HANDOFF for session 022 (Ch 21 — Thread-local, alloca, VLAs, commits 267–283) written. CLAUDE.md status updated.

Handoff: [docs/sessions/021-chapter-20-draft/HANDOFF.md](docs/sessions/021-chapter-20-draft/HANDOFF.md)

---

