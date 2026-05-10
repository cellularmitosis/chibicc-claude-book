# Claude Conversation Log

Session ID: 74affd53-6d69-45dc-b066-5bf05f0d3141
Date: 2026-05-10 08:10:38

---

## 👤 User

please read docs/sessions/019-chapter-18-draft/HANDOFF.md and proceed

---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/019-chapter-18-draft/HANDOFF.md"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "ls /Users/cell/claude/chibicc-book/docs/sessions/019-chapter-18-draft/ /Users/cell/claude/chibicc-book/chapters/ /Users/cell/claude/chibicc-book/research/commits/",
  "description": "List session files, chapters, and research commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "mkdir -p /Users/cell/claude/chibicc-book/docs/sessions/020-chapter-19-draft",
  "description": "Create session 020 directory"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "cd /Users/cell/claude/chibicc-book/research/sources/chibicc && for h in e27417f 0e77f3d 74bcec5 c31886a a57c661 454618c 2dac3af 57b21fe 9cabe1f c467ee6 cae061a 36230e0 6adba75 e4491b8 0e5d250 adb8b98 2382777 2b2fa25 c618c3b 835cd24 691c4fa 67f5834 31dc1df 95eb5b0; do echo \"===== $h =====\"; git show --stat $h | head -10; done",
  "description": "Show stats for all 24 Ch 19 commits"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e27417f",
  "description": "Show __DATE__/__TIME__ commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 0e77f3d",
  "description": "Show __COUNTER__ commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 74bcec5",
  "description": "Show newline canonicalization commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "pwd && git rev-parse HEAD",
  "description": "Verify chibicc clone location"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show c31886a",
  "description": "Show \\u and \\U escape sequences commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show a57c661",
  "description": "Show wide char multibyte commit"
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
      "content": "Read all 24 Ch 19 commits in detail",
      "activeForm": "Reading 24 Ch 19 commits",
      "status": "in_progress"
    },
    {
      "content": "Read prior chapter 18 for voice/tone matching",
      "activeForm": "Reading chapter 18",
      "status": "pending"
    },
    {
      "content": "Read session 019 README",
      "activeForm": "Reading session 019 README",
      "status": "pending"
    },
    {
      "content": "Draft chapters/19-unicode-and-designated-initializers.md",
      "activeForm": "Drafting Ch 19",
      "status": "pending"
    },
    {
      "content": "Write session 020 README and HANDOFF for session 021",
      "activeForm": "Writing session 020 docs",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 454618c",
  "description": "Show UTF-16 char literal commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 2dac3af",
  "description": "Show UTF-32 char literal commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 57b21fe",
  "description": "Show UTF-8 string literal commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 9cabe1f",
  "description": "Show UTF-16 string literal commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show c467ee6",
  "description": "Show UTF-32 string literal commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show cae061a",
  "description": "Show wide string literal commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 36230e0",
  "description": "Show UTF-16 string literal initializer commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 6adba75",
  "description": "Show UTF-32 string literal initializer commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show e4491b8",
  "description": "Show STDC_UTF macros commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 0e5d250",
  "description": "Show UTF-8 identifier commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show adb8b98",
  "description": "Show $ in identifier commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 2382777",
  "description": "Show cross-prefix concat commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 2b2fa25",
  "description": "Show UTF-8 BOM commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show c618c3b",
  "description": "Show array designated initializer commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 835cd24",
  "description": "Show incomplete array designator commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 691c4fa",
  "description": "Show GNU =-omission commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 67f5834",
  "description": "Show struct designated initializer commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 31dc1df",
  "description": "Show union designated initializer commit"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git show 95eb5b0",
  "description": "Show anon-struct designator commit"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 24 Ch 19 commits in detail",
      "activeForm": "Reading 24 Ch 19 commits",
      "status": "completed"
    },
    {
      "content": "Read prior chapter 18 for voice/tone matching",
      "activeForm": "Reading chapter 18",
      "status": "in_progress"
    },
    {
      "content": "Read session 019 README",
      "activeForm": "Reading session 019 README",
      "status": "pending"
    },
    {
      "content": "Draft chapters/19-unicode-and-designated-initializers.md",
      "activeForm": "Drafting Ch 19",
      "status": "pending"
    },
    {
      "content": "Write session 020 README and HANDOFF for session 021",
      "activeForm": "Writing session 020 docs",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -l /Users/cell/claude/chibicc-book/chapters/18-the-full-abi.md /Users/cell/claude/chibicc-book/chapters/17-a-preprocessor-from-scratch.md /Users/cell/claude/chibicc-book/chapters/12-initializers.md",
  "description": "Check chapter line counts"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/18-the-full-abi.md",
  "offset": 1,
  "limit": 200
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/18-the-full-abi.md",
  "offset": 1640,
  "limit": 100
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/019-chapter-18-draft/README.md"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n \"Initializer\\|string_initializer\\|new_initializer\\|is_flexible\\|count_array\" /Users/cell/claude/chibicc-book/chapters/12-initializers.md | head -40",
  "description": "Check Ch 12 initializer references"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -n \"encode_utf8\\|decode_utf8\\|tokenize_string_literal\\|read_universal_char\\|convert_universal_chars\\|join_adjacent\\|StringKind\" /Users/cell/claude/chibicc-book/research/sources/chibicc/tokenize.c /Users/cell/claude/chibicc-book/research/sources/chibicc/preprocess.c /Users/cell/claude/chibicc-book/research/sources/chibicc/unicode.c 2>/dev/null | head -30",
  "description": "Verify final state references"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout 95eb5b0 -- parse.c 2>&1 && grep -n \"designation\\|array_initializer\\|struct_initializer\\|union_initializer\\|new_initializer\" /Users/cell/claude/chibicc-book/research/sources/chibicc/parse.c | head -50",
  "description": "Check final parse.c state at commit 95eb5b0"
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/research/sources/chibicc/parse.c",
  "offset": 980,
  "limit": 35
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout main -- parse.c && git status -s parse.c",
  "description": "Reset parse.c to main"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout 835cd24 -- parse.c 2>&1 && sed -n '897,930p' /Users/cell/claude/chibicc-book/research/sources/chibicc/parse.c",
  "description": "Confirm dead-code at 835cd24"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "git checkout main -- parse.c 2>&1 && git status -s",
  "description": "Reset parse.c"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 24 Ch 19 commits in detail",
      "activeForm": "Reading 24 Ch 19 commits",
      "status": "completed"
    },
    {
      "content": "Read prior chapter 18 for voice/tone matching",
      "activeForm": "Reading chapter 18",
      "status": "completed"
    },
    {
      "content": "Read session 019 README",
      "activeForm": "Reading session 019 README",
      "status": "completed"
    },
    {
      "content": "Draft chapters/19-unicode-and-designated-initializers.md",
      "activeForm": "Drafting Ch 19",
      "status": "in_progress"
    },
    {
      "content": "Write session 020 README and HANDOFF for session 021",
      "activeForm": "Writing session 020 docs",
      "status": "pending"
    }
  ]
}


---

## 🤖 Claude

Now drafting Chapter 19.

---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/chapters/19-unicode-and-designated-initializers.md",
  "content": "# Chapter 19 \u2014 Unicode and designated initializers\n\n> Commits covered: `e27417f`, `0e77f3d`, `74bcec5`, `c31886a`, `a57c661`, `454618c`, `2dac3af`, `57b21fe`, `9cabe1f`, `c467ee6`, `cae061a`, `36230e0`, `6adba75`, `e4491b8`, `0e5d250`, `adb8b98`, `2382777`, `2b2fa25`, `c618c3b`, `835cd24`, `691c4fa`, `67f5834`, `31dc1df`, `95eb5b0`. Twenty-four commits \u2014 the full Unicode arc (universal character names, multibyte and wide character literals, UTF-{8,16,32} string literals and their initializers, UTF-8 in identifiers, UTF-8 BOM stripping, cross-prefix string concatenation), the GNU `$`-in-identifier extension, the trailing date/time/counter macros that Chapter 17 deferred, and the designated-initializer arc (array, struct, union, anonymous-struct, GNU `=`-omission).\n\nThrough Chapter 18, chibicc has the SysV AMD64 calling convention, bitfields, and a self-hosting build that's `-Wall`-clean. What it doesn't have is Unicode. The tokenizer accepts ASCII identifiers and ASCII string literals; a Japanese variable name is a parse error, a `\u03b1` escape sequence is two tokens (`\\u` followed by `03B1`), and a `u\"\u03b1\u03b2\"` literal is the identifier `u` followed by an ASCII string.\n\nWhat chibicc also doesn't have is *designated initializers* \u2014 the C99 form `struct foo x = {.a = 1, .b = 2}` and the array form `int x[10] = {[3] = 1, [7] = 2}`. The `Initializer` tree from Chapter 12 fills children left-to-right; jumping the cursor by name or index requires extending the parser.\n\nThe twenty-four commits in this chapter address both topics. They arrive on `main` in chronological order, and on calendar time they scatter from May 2020 (the `__DATE__` macro) to October 2020 (the cross-prefix concatenation, the BOM skip), with several reordered relative to when Rui actually wrote them \u2014 the wide-character commit `a57c661` is dated May 6, 2020 but lands at position 225 on `main`, after several later-dated commits.\n\nSeven sections from twenty-four commits.\n\n- **\u00a719.1** \u2014 Date, time, counter macros (commits 221\u2013222).\n- **\u00a719.2** \u2014 Newline canonicalization (commit 223).\n- **\u00a719.3** \u2014 Universal character names (commit 224).\n- **\u00a719.4** \u2014 Multibyte and wide character literals (commits 225\u2013227).\n- **\u00a719.5** \u2014 UTF-8, UTF-16, UTF-32, and wide string literals (commits 228\u2013234).\n- **\u00a719.6** \u2014 Identifier-side Unicode and BOM (commits 235\u2013238).\n- **\u00a719.7** \u2014 Designated initializers (commits 239\u2013244).\n\nThe Unicode arc (\u00a719.3 through \u00a719.6) is the chapter's deepest single stretch \u2014 twelve commits, two new files (`unicode.c` and `test/unicode.c`), and a small but principled split between the *tokenizer* (which now does UTF-8 decoding) and the *preprocessor* (which now normalizes the prefixes of adjacent string literals). The designated-initializer arc (\u00a719.7) is the chapter's most invasive single stretch \u2014 six commits, all in `parse.c`, all reshaping the same handful of helpers (`array_initializer1`, `array_initializer2`, `struct_initializer1`, `struct_initializer2`, `union_initializer`, `count_array_init_elements`, plus the new `designation` and `array_designator` and `struct_designator`).\n\nThe chapter follows `main` order.\n\n---\n\n## 19.1 \u2014 Date, time, counter macros\n\n> `git checkout e27417fcde500f6c01ce0dbee57a1af137510a09` \u2014 *Add __DATE__ and __TIME__ macros*\n>\n> `git checkout 0e77f3dff8b44547da4639c9609c216c9c896fa5` \u2014 *[GNU] Add __COUNTER__ macro*\n\nTwo commits. Both extend `init_macros` in `preprocess.c` with new predefined macros, all three of which use the `Macro->handler` field that Chapter 17 \u00a717.5.3 added for `__FILE__` and `__LINE__`.\n\n`__DATE__` and `__TIME__` are stateless from the program's view: they expand to the same string every time, fixed at the moment cc1 starts. The implementation is two `format_*` helpers and a single `time(NULL)` call at the bottom of `init_macros`:\n\n```c\n// __DATE__ is expanded to the current date, e.g. \"May 17 2020\".\nstatic char *format_date(struct tm *tm) {\n  static char mon[][4] = {\n    \"Jan\", \"Feb\", \"Mar\", \"Apr\", \"May\", \"Jun\",\n    \"Jul\", \"Aug\", \"Sep\", \"Oct\", \"Nov\", \"Dec\",\n  };\n\n  return format(\"\\\"%s %2d %d\\\"\", mon[tm->tm_mon], tm->tm_mday, tm->tm_year + 1900);\n}\n\n// __TIME__ is expanded to the current time, e.g. \"13:34:03\".\nstatic char *format_time(struct tm *tm) {\n  return format(\"\\\"%02d:%02d:%02d\\\"\", tm->tm_hour, tm->tm_min, tm->tm_sec);\n}\n```\n\nAnd, at the end of `init_macros`:\n\n```c\ntime_t now = time(NULL);\nstruct tm *tm = localtime(&now);\ndefine_macro(\"__DATE__\", format_date(tm));\ndefine_macro(\"__TIME__\", format_time(tm));\n```\n\nTwo small choices worth flagging. The first is that the date and time are local-zone, not UTC \u2014 `localtime` rather than `gmtime`. Two different machines compiling the same source on the same wall-clock instant in different time zones produce different `__DATE__` strings. The C standard (\u00a76.10.8) says only that `__DATE__` is \"the date of translation\"; the choice of zone is implementation-defined. Chibicc picks local; gcc and clang also pick local.\n\nThe second is that `format_date` and `format_time` produce strings with embedded quotation marks (`\"May 17 2020\"`, including the `\"`). The macros are defined via `define_macro`, which calls the tokenizer on the value string, so the quotes are *part of the macro body* \u2014 they make the body a string literal, which is what `__DATE__` and `__TIME__` are supposed to expand to.\n\nThese two macros close a small gap that Chapter 17 left open \u2014 Rui's preprocessor commits implemented `__FILE__`, `__LINE__`, `__VA_ARGS__`, and the rest of the predefined-macro vocabulary, but `__DATE__` and `__TIME__` are unique in needing real-time-clock state at startup, and Rui broke them out into their own commit.\n\n`__COUNTER__` is a GCC extension. Each expansion produces the next integer in the sequence 0, 1, 2, \u2026 . The implementation is six lines:\n\n```c\n// __COUNTER__ is expanded to serial values starting from 0.\nstatic Token *counter_macro(Token *tmpl) {\n  static int i = 0;\n  return new_num_token(i++, tmpl);\n}\n```\n\nAnd one line in `init_macros`:\n\n```c\nadd_builtin(\"__COUNTER__\", counter_macro);\n```\n\nThe `static int i` inside the handler is the entire state. The `tmpl` parameter is the call-site token, which `new_num_token` uses for the source position. Each invocation hands back a fresh `TK_NUM` token whose value is the current counter and whose position is the call site.\n\n`__COUNTER__` is what makes macros like `static_assert` work without source-code-level boilerplate \u2014 the typical idiom is to paste `__COUNTER__` into a generated identifier so each expansion gets a unique name. Chibicc doesn't ship a `static_assert` macro, but the underlying primitive is in place.\n\nThe three macros sit in two categories: `__DATE__` and `__TIME__` are *fixed-at-startup* (the `time()` call is once, the strings are constant after that), and `__COUNTER__` is *fixed-at-each-expansion* (the value updates on every read). Both categories use the same `Macro->handler` field, just with different state semantics.\n\n**Where we are.** The predefined-macro vocabulary is feature-complete for Chapter 17's table plus `__DATE__`, `__TIME__`, `__COUNTER__`. The stateful macro hook (Chapter 17 \u00a717.5.3) gains a third user.\n\n---\n\n## 19.2 \u2014 Newline canonicalization\n\n> `git checkout 74bcec5b22a601451fac9d0878003d04205abca6` \u2014 *Canonicalize newline character*\n\nOne commit. Twenty lines of new code in `tokenize.c`, plus one line in `tokenize_file` to call the new helper before `remove_backslash_newline`.\n\nThe helper:\n\n```c\n// Replaces \\r or \\r\\n with \\n.\nstatic void canonicalize_newline(char *p) {\n  int i = 0, j = 0;\n\n  while (p[i]) {\n    if (p[i] == '\\r' && p[i + 1] == '\\n') {\n      i += 2;\n      p[j++] = '\\n';\n    } else if (p[i] == '\\r') {\n      i++;\n      p[j++] = '\\n';\n    } else {\n      p[j++] = p[i++];\n    }\n  }\n\n  p[j] = '\\0';\n}\n```\n\nIn-place, two cursors: `i` reads, `j` writes. A `\\r\\n` pair collapses to `\\n`. A bare `\\r` (old-Mac line endings) becomes `\\n`. Anything else passes through.\n\nThe call site in `tokenize_file`:\n\n```c\ncanonicalize_newline(p);\nremove_backslash_newline(p);\n```\n\nOrder matters. `remove_backslash_newline` checks for the two-byte sequence `\\\\` followed by `\\n`; if `\\r\\n` came through unchanged, a Windows-style file with `\\\\\\r\\n` line continuations would not be recognized. By canonicalizing first, line continuations work regardless of source-file line ending.\n\nTwo file rewrites in tokenize_file are now in place: newline canonicalization, then backslash-newline removal. The next commit (\u00a719.3) will add a third \u2014 universal-character-name conversion. By the end of this chapter the count will be four (the \u00a719.6 BOM strip happens before all three of these). The pattern is consistent: each is an in-place rewrite of the source buffer that runs before tokenization proper.\n\nThe `canonicalize_newline` helper is a tokenizer-pre-pass, parallel in shape to `remove_backslash_newline`, and both are *destructive* \u2014 they mutate the buffer that `read_file` malloced. Source positions reported by `error_at` after canonicalization point at the canonicalized buffer, which means errors in files with mixed line endings might report a column that's slightly different from the column an editor would show. Acceptable in practice.\n\n**Where we are.** Source files with `\\r\\n` and `\\r` line endings parse without diagnostics. Line-continuation backslashes work in any file regardless of how the file was saved. The pre-tokenize pass count is at two.\n\n---\n\n## 19.3 \u2014 Universal character names\n\n> `git checkout c31886aa7a52fd8639e09bbdf8ac8ea854c313f6` \u2014 *Add \\u and \\U escape sequences*\n\nOne commit. Three changes: a new file `unicode.c` with the `encode_utf8` helper, a new `convert_universal_chars` pass in `tokenize.c` that runs as a third pre-tokenize rewrite, and a new test file `test/unicode.c`.\n\nThe C standard's *universal character name* (UCN) is the source-level escape `\\uXXXX` (four hex digits) or `\\UXXXXXXXX` (eight hex digits) that names a Unicode code point. UCNs can appear in string literals, character literals, and identifiers, though chibicc's implementation handles them only in string and character literals \u2014 UCNs in identifiers would require running the conversion on identifier text, not just on string-literal contents, and chibicc takes the simpler path of not supporting that.\n\nThe new `convert_universal_chars` pass walks the source buffer and replaces every UCN with the corresponding UTF-8 byte sequence, in place:\n\n```c\nstatic uint32_t read_universal_char(char *p, int len) {\n  uint32_t c = 0;\n  for (int i = 0; i < len; i++) {\n    if (!isxdigit(p[i]))\n      return 0;\n    c = (c << 4) | from_hex(p[i]);\n  }\n  return c;\n}\n\n// Replace \\u or \\U escape sequences with corresponding UTF-8 bytes.\nstatic void convert_universal_chars(char *p) {\n  char *q = p;\n\n  while (*p) {\n    if (startswith(p, \"\\\\u\")) {\n      uint32_t c = read_universal_char(p + 2, 4);\n      if (c) {\n        p += 6;\n        q += encode_utf8(q, c);\n      } else {\n        *q++ = *p++;\n      }\n    } else if (startswith(p, \"\\\\U\")) {\n      uint32_t c = read_universal_char(p + 2, 8);\n      if (c) {\n        p += 10;\n        q += encode_utf8(q, c);\n      } else {\n        *q++ = *p++;\n      }\n    } else if (p[0] == '\\\\') {\n      *q++ = *p++;\n      *q++ = *p++;\n    } else {\n      *q++ = *p++;\n    }\n  }\n\n  *q = '\\0';\n}\n```\n\nTwo-cursor in-place rewrite again, like `canonicalize_newline`. The output is always shorter than or equal to the input (six or ten input bytes become up to four UTF-8 bytes). The middle `else if (p[0] == '\\\\')` arm passes ordinary backslash escapes (`\\n`, `\\t`, `\\xff`, etc.) through unchanged so they're seen later by `read_escaped_char` during string-literal tokenization.\n\n`encode_utf8`, the new helper, lives in `unicode.c`:\n\n```c\n// Encode a given character in UTF-8.\nint encode_utf8(char *buf, uint32_t c) {\n  if (c <= 0x7F) {\n    buf[0] = c;\n    return 1;\n  }\n\n  if (c <= 0x7FF) {\n    buf[0] = 0b11000000 | (c >> 6);\n    buf[1] = 0b10000000 | (c & 0b00111111);\n    return 2;\n  }\n\n  if (c <= 0xFFFF) {\n    buf[0] = 0b11100000 | (c >> 12);\n    buf[1] = 0b10000000 | ((c >> 6) & 0b00111111);\n    buf[2] = 0b10000000 | (c & 0b00111111);\n    return 3;\n  }\n\n  buf[0] = 0b11110000 | (c >> 18);\n  buf[1] = 0b10000000 | ((c >> 12) & 0b00111111);\n  buf[2] = 0b10000000 | ((c >> 6) & 0b00111111);\n  buf[3] = 0b10000000 | (c & 0b00111111);\n  return 4;\n}\n```\n\nA direct transcription of the UTF-8 encoding rule. Code points up to U+007F are one byte (the ASCII range, where UTF-8 and ASCII coincide). U+0080 through U+07FF are two bytes (`110xxxxx 10xxxxxx`). U+0800 through U+FFFF are three bytes (`1110xxxx 10xxxxxx 10xxxxxx`). U+10000 through U+10FFFF are four bytes (`11110xxx 10xxxxxx 10xxxxxx 10xxxxxx`). The function returns the byte count.\n\nChibicc adopts the convention used by gcc and clang: source files are UTF-8, and a UCN in the source becomes UTF-8 bytes in the parsed string-literal contents. A literal like `\"\u03b1\"` and a literal like `\"\u03b1\"` are byte-for-byte equivalent after this pre-pass \u2014 both produce a two-byte UTF-8 sequence in the token's `str` field.\n\nThe pre-tokenize sequence in `tokenize_file` is now three steps:\n\n```c\ncanonicalize_newline(p);\nremove_backslash_newline(p);\nconvert_universal_chars(p);\n```\n\nAfter this, the tokenizer sees a buffer that's free of `\\r`, free of line continuations, and free of UCNs. It can stay ASCII-byte-based for ordinary identifier and number recognition; the multibyte handling that \u00a719.6 will introduce for identifiers, and the \u00a719.4\u2013\u00a719.5 handling for character and string literals, layer on top of this clean baseline.\n\nThe new test file `test/unicode.c` exercises the round-trip:\n\n```c\nASSERT(0, strcmp(\"\u03b1\u03b2\u03b3\", \"\u03b1\u03b2\u03b3\"));\nASSERT(0, strcmp(\"\u65e5\u672c\u8a9e\", \"\u65e5\u672c\u8a9e\"));\nASSERT(0, strcmp(\"\u65e5\u672c\u8a9e\", \"\\U000065E5\\U0000672C\\U00008A9E\"));\nASSERT(0, strcmp(\"\ud83c\udf2e\", \"\\U0001F32E\"));\n```\n\nThe first two test that `\\u` escapes encode the same UTF-8 bytes as a literal multibyte source character. The third tests that `\\U` is the same when the code point fits in `\\u`'s four hex digits. The fourth tests four-byte UTF-8 (the taco emoji is U+1F32E, outside the BMP).\n\n`unicode.c` is added to the Makefile's `OBJS` list. It will pick up two more functions through the chapter \u2014 `decode_utf8` in \u00a719.4, `is_ident1` and `is_ident2` in \u00a719.6.\n\n**Where we are.** Universal character names work in any string or character literal context. UTF-8 is the canonical source-byte representation. The pre-tokenize pass count is at three.\n\n---\n\n## 19.4 \u2014 Multibyte and wide character literals\n\n> `git checkout a57c661d46d9523bed01ad1b074f7a78d9e94ca3` \u2014 *Accept multibyte character as wide character literal*\n>\n> `git checkout 454618cd15c2c87d9f5a6a6727e1b09a8e22a799` \u2014 *Add UTF-16 character literal*\n>\n> `git checkout 2dac3afece31c27bf773efbc1f30c6a67088d3b6` \u2014 *Add UTF-32 character literal*\n\nThree commits. The first introduces `decode_utf8` and rewires `read_char_literal` to call it. The next two add `u'X'` and `U'X'` token recognition, both routed through the same `read_char_literal` with a different result type.\n\n### `a57c661` \u2014 multibyte in wide character literal\n\nThe pre-commit `read_char_literal` was a simple ASCII-byte read:\n\n```c\nchar c;\nif (*p == '\\\\')\n  c = read_escaped_char(&p, p + 1);\nelse\n  c = *p++;\n```\n\nRead an ASCII byte (or a parsed escape), assign to a `char`. A `L'\u3042'` source \u2014 three UTF-8 bytes for the hiragana A \u2014 would either fail or read just the first byte, depending on luck.\n\nThe post-commit version:\n\n```c\nint c;\nif (*p == '\\\\')\n  c = read_escaped_char(&p, p + 1);\nelse\n  c = decode_utf8(&p, p);\n```\n\nType changes from `char` to `int` (so a 32-bit code point fits), and the non-escape path goes through the new `decode_utf8` helper. The new helper, in `unicode.c`:\n\n```c\nuint32_t decode_utf8(char **new_pos, char *p) {\n  if ((unsigned char)*p < 128) {\n    *new_pos = p + 1;\n    return *p;\n  }\n\n  char *start = p;\n  int len;\n  uint32_t c;\n\n  if ((unsigned char)*p >= 0b11110000) {\n    len = 4;\n    c = *p & 0b111;\n  } else if ((unsigned char)*p >= 0b11100000) {\n    len = 3;\n    c = *p & 0b1111;\n  } else if ((unsigned char)*p >= 0b11000000) {\n    len = 2;\n    c = *p & 0b11111;\n  } else {\n    error_at(start, \"invalid UTF-8 sequence\");\n  }\n\n  for (int i = 1; i < len; i++) {\n    if ((unsigned char)p[i] >> 6 != 0b10)\n      error_at(start, \"invalid UTF-8 sequence\");\n    c = (c << 6) | (p[i] & 0b111111);\n  }\n\n  *new_pos = p + len;\n  return c;\n}\n```\n\nThe mirror of `encode_utf8`. Read the leading byte, classify it by which high bits are set, read the continuation bytes (each starting with `10`), and assemble the code point. The `error_at` calls catch leading bytes that aren't in `[0xxxxxxx, 110xxxxx, 1110xxxx, 11110xxx]` and continuation bytes that don't start with `10`. ASCII bytes (`< 128`) are returned directly without entering the multibyte machinery.\n\nThere's also a quiet line in `tokenize` that this commit adds:\n\n```c\n// Character literal\nif (*p == '\\'') {\n  cur = cur->next = read_char_literal(p, p);\n  cur->val = (char)cur->val;\n  p += cur->len;\n  continue;\n}\n```\n\nThe `cur->val = (char)cur->val` truncates the value to a `char` *for the ordinary `'X'` form only* \u2014 narrow character literals must fit in a byte. The wide character literal `L'X'` doesn't go through this branch. The narrowing is what makes `(char)0xff == -1` for ordinary `'\\xff'` and `0x000000ff` for `L'\\xff'`.\n\nThe commit message has Rui's design note:\n\n> On most Unix-like systems, wide character literal is 32-bit long and encodes a Unicode code point. On Windows, that is 16-bit long and encodes a UTF-16 code unit. Clearly, there's a portability issue here. Personally I've never used wide characters in my code as I didn't find it useful.\n>\n> Being said that, some header files contain wide character literal, so we need to support that so that chibicc can include such files.\n>\n> We assume that source files are always encoded in UTF-8.\n\nThree things to extract. First, chibicc's `wchar_t` is 32-bit (the type `ty_int` in `read_char_literal`'s first call from the wide-char branch). That's the Linux / SysV convention. Windows would want 16-bit, but chibicc isn't a Windows compiler. Second, the motivation for adding wide character literals at all is *include compatibility* \u2014 system headers contain `L'\\0'` and similar, and chibicc can't include them otherwise. Third, the source-files-are-UTF-8 assumption is now declared in writing; the entire UTF-8 decoder (`decode_utf8`) and the \u00a719.3 UCN-to-UTF-8 conversion both rely on this.\n\nThis commit closes the Chapter 17 \u00a717.5.3 errata candidate \"`L''` \u2261 `''`\" \u2014 the wide-character-literal punt is gone. `L'\u3042'` is now `12354` (decimal), not `0` or whatever the first UTF-8 byte happens to be. `L'\\xffffffff'` is `-1` after the implicit narrowing-to-`int`-then-shift.\n\n### `454618c` \u2014 UTF-16 character literal\n\nTwo-line addition to the dispatch in `tokenize`, plus a four-line refactor of `read_char_literal`'s signature to take the result type as a parameter:\n\n```c\nstatic Token *read_char_literal(char *start, char *quote, Type *ty) {\n  // ... unchanged body, except the final assignment is to `ty` instead of `ty_int`\n  tok->ty = ty;\n  return tok;\n}\n```\n\nThe dispatch grows a UTF-16 case:\n\n```c\n// UTF-16 character literal\nif (startswith(p, \"u'\")) {\n  cur = cur->next = read_char_literal(p, p + 1, ty_ushort);\n  cur->val &= 0xffff;\n  p += cur->len;\n  continue;\n}\n```\n\nThe result type is `unsigned short` (the chibicc convention for `char16_t`). The `cur->val &= 0xffff` truncates to 16 bits. The `p + 1` argument tells `read_char_literal` to start reading after the `u`, so the function sees the leading `'` as if it were the first character.\n\nThe wide-character-literal branch is also touched \u2014 the prior `p = cur->loc + cur->len` line, which advanced relative to the start-of-token, is replaced with `p += cur->len`, which advances relative to the source pointer. The `cur->len` is set by `read_char_literal` to span the full literal including the prefix; the older form was a confused leftover that happened to work because `cur->loc == p` for the L'X' case. The fix is unrelated to UTF-16 but lands in the same commit.\n\nA code point above U+FFFF in a `u'X'` literal silently truncates. The C standard requires this to be a constraint violation, but chibicc's implementation is permissive. The test file exercises the in-range case:\n\n```c\nASSERT(2, sizeof(u'\\0'));\nASSERT(1, u'\\xffff'>>15);\nASSERT(97, u'a');\nASSERT(946, u'\u03b2');\nASSERT(12354, u'\u3042');\nASSERT(62307, u'\ud83c\udf63');     // U+1F363, truncated to 0xF363 = 62307\n```\n\nThe last test pins the silent truncation: the sushi emoji's code point is 0x1F363 in full, 0xF363 after the low-16 mask.\n\n### `2dac3af` \u2014 UTF-32 character literal\n\nSeven-line addition to the dispatch:\n\n```c\n// UTF-32 character literal\nif (startswith(p, \"U'\")) {\n  cur = cur->next = read_char_literal(p, p + 1, ty_uint);\n  p += cur->len;\n  continue;\n}\n```\n\nThe result type is `unsigned int` (the chibicc convention for `char32_t`). No mask \u2014 32 bits is the full Unicode range. The test cases include a U+1F363 code point that arrives unmangled, in contrast to the UTF-16 case:\n\n```c\nASSERT(127843, U'\ud83c\udf63');    // 0x1F363, the full code point\n```\n\nAfter this commit, the four-way dispatch on character-literal prefix is complete: `'X'` is `int` (then narrowed to `char`), `L'X'` is `int`, `u'X'` is `unsigned short` (truncated to 16 bits), `U'X'` is `unsigned int`. Each goes through the same `read_char_literal` with a different `Type *`.\n\n**Where we are.** The four character-literal forms are all functional. UTF-8 source bytes decode to Unicode code points via `decode_utf8`. `wchar_t` is 32-bit Linux convention. The Chapter 17 `L''` \u2261 `''` errata is closed. The first ASCII-truncation step in `tokenize` sets the narrow-form behavior; the prefix branches set the wide-form behavior.\n\n---\n\n## 19.5 \u2014 UTF-8, UTF-16, UTF-32, and wide string literals\n\n> `git checkout 57b21fe90296c867888d7c8c60d243bc254a39d7` \u2014 *Add UTF-8 string literal*\n>\n> `git checkout 9cabe1f204a8a6139e8b072dfd6f0a15275ad25f` \u2014 *Add UTF-16 string literal*\n>\n> `git checkout c467ee665de0c385170850ecc895add04b52b8a3` \u2014 *Add UTF-32 string literal*\n>\n> `git checkout cae061af2b65ad0962fb4b6fe3b55abe2f3a5bf8` \u2014 *Add wide string literal*\n>\n> `git checkout 36230e0827ca33a9b09ea5aa7b06e170fd188ca1` \u2014 *Add UTF-16 string literal initializer*\n>\n> `git checkout 6adba75af879d8ac2bc43a7337b02e64d10e60f1` \u2014 *Add UTF-32 string literal initializer*\n>\n> `git checkout e4491b811510d08f880d0f9c7553ecfd18635469` \u2014 *Define __STDC_UTF_{16,32}__ macros*\n\nSeven commits. Four for the four prefix forms (`u8`, `u`, `U`, `L`), two for the initializer side (so `char16_t s[] = u\"...\"` and the UTF-32 / wide equivalents work), one for the `__STDC_UTF_*` predefined macros that announce the encoding choice.\n\n### `57b21fe` \u2014 UTF-8 string literal\n\nThe smallest of the four prefix commits. `u8\"abc\"` is *byte-equivalent* to `\"abc\"` for ASCII input, and equivalent to a string literal for any non-ASCII input as well (since chibicc's source-file convention is already UTF-8).\n\nThe implementation: refactor `read_string_literal` to take the position-after-prefix as a parameter, then add a one-branch dispatch for `u8\"`:\n\n```c\nstatic Token *read_string_literal(char *start, char *quote) {\n  char *end = string_literal_end(quote + 1);\n  char *buf = calloc(1, end - quote);\n  int len = 0;\n\n  for (char *p = quote + 1; p < end;) {\n    if (*p == '\\\\')\n      buf[len++] = read_escaped_char(&p, p + 1);\n    else\n      buf[len++] = *p++;\n  }\n  // ...\n}\n```\n\nThe `start` is the start of the token (including the `u8` prefix); `quote` points at the opening `\"`. The two are the same for an ordinary `\"foo\"` and differ by two bytes for `u8\"foo\"`.\n\nThe dispatch:\n\n```c\n// String literal\nif (*p == '\"') {\n  cur = cur->next = read_string_literal(p, p);\n  p += cur->len;\n  continue;\n}\n\n// UTF-8 string literal\nif (startswith(p, \"u8\\\"\")) {\n  cur = cur->next = read_string_literal(p, p + 2);\n  p += cur->len;\n  continue;\n}\n```\n\n`u8` literals get the same `char[]` type as ordinary string literals and the same source-byte-pass-through reading. The result is identical to the unprefixed form for any input chibicc accepts. The `u8` prefix's value is mostly social \u2014 it announces \"this is meant as UTF-8\" \u2014 and matters more once cross-prefix concatenation enters the picture in \u00a719.6.\n\n### `9cabe1f` \u2014 UTF-16 string literal\n\nThe largest of the four prefix commits. UTF-16 is a *transcoding* \u2014 UTF-8 source bytes are decoded into code points and then re-encoded into 16-bit units, with surrogate pairs for code points above U+FFFF.\n\nA new helper, `read_utf16_string_literal`:\n\n```c\n// Read a UTF-8-encoded string literal and transcode it in UTF-16.\n//\n// UTF-16 is yet another variable-width encoding for Unicode. Code\n// points smaller than U+10000 are encoded in 2 bytes. Code points\n// equal to or larger than that are encoded in 4 bytes. Each 2 bytes\n// in the 4 byte sequence is called \"surrogate\", and a 4 byte sequence\n// is called a \"surrogate pair\".\nstatic Token *read_utf16_string_literal(char *start, char *quote) {\n  char *end = string_literal_end(quote + 1);\n  uint16_t *buf = calloc(2, end - start);\n  int len = 0;\n\n  for (char *p = quote + 1; p < end;) {\n    if (*p == '\\\\') {\n      buf[len++] = read_escaped_char(&p, p + 1);\n      continue;\n    }\n\n    uint32_t c = decode_utf8(&p, p);\n    if (c < 0x10000) {\n      // Encode a code point in 2 bytes.\n      buf[len++] = c;\n    } else {\n      // Encode a code point in 4 bytes.\n      c -= 0x10000;\n      buf[len++] = 0xd800 + ((c >> 10) & 0x3ff);\n      buf[len++] = 0xdc00 + (c & 0x3ff);\n    }\n  }\n\n  Token *tok = new_token(TK_STR, start, end + 1);\n  tok->ty = array_of(ty_ushort, len + 1);\n  tok->str = (char *)buf;\n  return tok;\n}\n```\n\nThe buffer is a `uint16_t *` cast to `char *` for token storage. The element type on the token is `ty_ushort` (chibicc's `char16_t`), and the array length is `len + 1` for the trailing null. The surrogate-pair encoding is the standard one: subtract 0x10000, split the remaining 20 bits into two 10-bit halves, and prefix them with 0xD800 and 0xDC00 respectively.\n\nThe dispatch:\n\n```c\n// UTF-16 string literal\nif (startswith(p, \"u\\\"\")) {\n  cur = cur->next = read_utf16_string_literal(p, p + 1);\n  p += cur->len;\n  continue;\n}\n```\n\nThe test cases pin the byte layout:\n\n```c\nASSERT(0, memcmp(u\"\u65e5\u672c\u8a9e\", \"\\345e,g\\236\\212\\0\\0\", 8));\nASSERT(0, memcmp(u\"\ud83c\udf63\", \"<\\330c\\337\\0\\0\", 6));\n```\n\nThe `\ud83c\udf63` test pins the surrogate pair: 0x1F363 - 0x10000 = 0xF363, split into 0x3D8 (high 10 bits) and 0x363 (low 10 bits), encoded as 0xD83C, 0xDF63. The bytes `<\\330c\\337` are 0x3C, 0xD8, 0x63, 0xDF \u2014 little-endian 0xD83C, 0xDF63. (The byte-order convention is that of the host machine; chibicc emits the data as raw bytes via the assembler's `.byte` directives.)\n\nEscaped characters skip `decode_utf8` and go through `read_escaped_char` directly. The result is *not* re-encoded \u2014 `u\"\\xff\"` produces a single 16-bit unit `0x00FF`, not the two-byte UTF-16 encoding of U+00FF. The C standard treats `\\x` and `\\u` differently in string-prefix contexts; chibicc's permissive reading matches what gcc and clang do.\n\n### `c467ee6` \u2014 UTF-32 string literal\n\nThe simplest transcoding. Each UTF-8 source code point becomes one 32-bit unit:\n\n```c\n// Read a UTF-8-encoded string literal and transcode it in UTF-32.\n//\n// UTF-32 is a fixed-width encoding for Unicode. Each code point is\n// encoded in 4 bytes.\nstatic Token *read_utf32_string_literal(char *start, char *quote, Type *ty) {\n  char *end = string_literal_end(quote + 1);\n  uint32_t *buf = calloc(4, end - quote);\n  int len = 0;\n\n  for (char *p = quote + 1; p < end;) {\n    if (*p == '\\\\')\n      buf[len++] = read_escaped_char(&p, p + 1);\n    else\n      buf[len++] = decode_utf8(&p, p);\n  }\n\n  Token *tok = new_token(TK_STR, start, end + 1);\n  tok->ty = array_of(ty, len + 1);\n  tok->str = (char *)buf;\n  return tok;\n}\n```\n\nThe element type is passed as a parameter (`ty`). For `U\"...\"` the type is `ty_uint` (chibicc's `char32_t`). For `L\"...\"`, the next commit, the type is `ty_int` (chibicc's `wchar_t`). Same function, two callers.\n\nThe dispatch:\n\n```c\n// UTF-32 string literal\nif (startswith(p, \"U\\\"\")) {\n  cur = cur->next = read_utf32_string_literal(p, p + 1, ty_uint);\n  p += cur->len;\n  continue;\n}\n```\n\n### `cae061a` \u2014 wide string literal\n\nSeven-line addition. The wide string literal `L\"...\"` reuses `read_utf32_string_literal` with a different element type:\n\n```c\n// Wide string literal\nif (startswith(p, \"L\\\"\")) {\n  cur = cur->next = read_utf32_string_literal(p, p + 1, ty_int);\n  p += cur->len;\n  continue;\n}\n```\n\n`ty_int` is signed; `ty_uint` is unsigned. The byte layout is identical for any code point with the high bit clear (which is everything in valid Unicode, since the maximum code point is U+10FFFF). The signedness shows up in the test cases:\n\n```c\nASSERT(1, U\"\\xffffffff\"[0] >> 31);   // unsigned: high bit set, shifted right by 31 gives 1\nASSERT(-1, L\"\\xffffffff\"[0] >> 31);  // signed: arithmetic shift gives -1\n```\n\nSame source byte sequence, different signedness, different shift result. Chibicc is consistent here \u2014 `L\"X\"` is signed because `wchar_t` on Linux is signed `int`, and `U\"X\"` is unsigned because `char32_t` is unsigned by C-spec.\n\nAfter `cae061a` the dispatch in `tokenize` has, in order: ordinary string, UTF-8 string, UTF-16 string, UTF-32 string, wide string. Five branches, four helpers (UTF-8 and ordinary share `read_string_literal`, UTF-32 and wide share `read_utf32_string_literal`).\n\n### `36230e0` \u2014 UTF-16 string literal initializer\n\nThe string literal exists as a token with the right type and bytes; the question is how it lands in the `Initializer` tree.\n\nPre-commit, `string_initializer` assumed bytes-to-bytes:\n\n```c\nint len = MIN(init->ty->array_len, tok->ty->array_len);\nfor (int i = 0; i < len; i++)\n  init->children[i]->expr = new_num(tok->str[i], tok);\n```\n\n`tok->str[i]` reads one byte. For `wchar_t s[] = L\"abc\"`, where the destination is `int[4]` and the token's str holds three 32-bit values, this would read four bytes from the start of the buffer and produce the wrong children.\n\nThe post-commit version dispatches on the destination's element size:\n\n```c\nswitch (init->ty->base->size) {\ncase 1: {\n  char *str = tok->str;\n  for (int i = 0; i < len; i++)\n    init->children[i]->expr = new_num(str[i], tok);\n  break;\n}\ncase 2: {\n  uint16_t *str = (uint16_t *)tok->str;\n  for (int i = 0; i < len; i++)\n    init->children[i]->expr = new_num(str[i], tok);\n  break;\n}\ndefault:\n  unreachable();\n}\n```\n\n`init->ty->base->size` is 1 for `char[]`, 2 for `unsigned short[]` / `char16_t[]`. Each branch reinterprets `tok->str` as a pointer of the right element width. The default case is `unreachable()` because no other element size makes sense \u2014 yet (the next commit will add the size-4 case).\n\nThere's also a one-character fix for the `L\"...\"` advance-pointer line, the same kind of `cur->loc + cur->len` \u2192 `cur->len` correction the \u00a719.4 wide-character-literal commit had:\n\n```c\n// Wide character literal\nif (startswith(p, \"L'\")) {\n  cur = cur->next = read_char_literal(p, p + 1, ty_int);\n- p = cur->loc + cur->len;\n+ p += cur->len;\n  continue;\n}\n```\n\nSame kind of leftover, same fix, different tokenizer arm. Rui packaged both into the same string-initializer commit, perhaps because `L'X'` and `L\"X\"` are adjacent in tokenize.c.\n\nThe test file gains:\n\n```c\ntypedef unsigned short char16_t;\n\nASSERT(u'\u03b1', ({ char16_t x[] = u\"\u03b1\u03b2\"; x[0]; }));\nASSERT(u'\u03b2', ({ char16_t x[] = u\"\u03b1\u03b2\"; x[1]; }));\nASSERT(6, ({ char16_t x[] = u\"\u03b1\u03b2\"; sizeof(x); }));\n```\n\nThe `typedef` gives `char16_t` a definition (chibicc has no `<uchar.h>`); the rest tests that the initializer wires up correctly.\n\n### `6adba75` \u2014 UTF-32 string literal initializer\n\nThe size-4 case. Six new lines in the switch:\n\n```c\ncase 4: {\n  uint32_t *str = (uint32_t *)tok->str;\n  for (int i = 0; i < len; i++)\n    init->children[i]->expr = new_num(str[i], tok);\n  break;\n}\n```\n\nAfter this, the switch covers element widths 1, 2, and 4 \u2014 every chibicc-supported character or string element width. The `unreachable()` default catches programmer error.\n\n### `e4491b8` \u2014 `__STDC_UTF_{16,32}__`\n\nTwo predefined macros, two `define_macro` lines. The C11 standard's \u00a76.10.8 says that a conforming implementation that uses UTF-16 for `char16_t` and UTF-32 for `char32_t` defines `__STDC_UTF_16__` and `__STDC_UTF_32__` to 1.\n\nChibicc uses both encodings (\u00a719.5's tokenizer commits made them so), so the macros are now appropriate to define:\n\n```c\ndefine_macro(\"__STDC_UTF_16__\", \"1\");\ndefine_macro(\"__STDC_UTF_32__\", \"1\");\n```\n\nTwo-line commit. The position in `init_macros` is alphabetical-ish, in among the `__STDC_NO_*` family from Chapter 17.\n\n**Where we are.** The four string-literal prefixes are functional, with type assignments matching gcc on Linux: `char[]` for ordinary and `u8`, `unsigned short[]` for `u`, `unsigned int[]` for `U`, `int[]` for `L`. Initializer wiring covers element widths 1, 2, and 4. The two `__STDC_UTF_*` macros announce the encoding choice. The `Initializer` tree's `string_initializer` path now dispatches on element size; the `unreachable()` macro picks up two new sites by the end of the section.\n\n---\n\n## 19.6 \u2014 Identifier-side Unicode and BOM\n\n> `git checkout 0e5d250ebfd29845c8c26b0ad63379994a2b8560` \u2014 *Allow multibyte UTF-8 character in identifier*\n>\n> `git checkout adb8b988897758d0d4f74dcd9129bff0831634ae` \u2014 *[GNU] Accept $ as an identifier character*\n>\n> `git checkout 238277714ddc407f966f3c503e13a114d6a91630` \u2014 *Allow to concatenate regular string literals with L/u/U string literals*\n>\n> `git checkout 2b2fa25507cdc491d2b5dafb2c4b5e33158b996a` \u2014 *Skip UTF-8 BOM markers*\n\nFour commits. The first two extend the identifier-character predicate. The third is the cross-prefix string concatenation, which lives mostly in the preprocessor. The fourth is the UTF-8 byte-order-mark skip, which is a tokenizer-pre-pass parallel to \u00a719.2's newline canonicalization.\n\n### `0e5d250` \u2014 UTF-8 in identifiers\n\nThe pre-commit `is_ident1` and `is_ident2` were ASCII byte predicates:\n\n```c\nstatic bool is_ident1(char c) {\n  return ('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z') || c == '_';\n}\n\nstatic bool is_ident2(char c) {\n  return is_ident1(c) || ('0' <= c && c <= '9');\n}\n```\n\nThe post-commit signatures take a `uint32_t` (a Unicode code point, not a byte) and live in `unicode.c`, so they can be called from the tokenizer's identifier-recognizer:\n\n```c\nbool is_ident1(uint32_t c);\nbool is_ident2(uint32_t c);\n```\n\nThe tokenizer's pre-commit identifier recognition was a do-while on bytes:\n\n```c\nif (is_ident1(*p)) {\n  char *start = p;\n  do {\n    p++;\n  } while (is_ident2(*p));\n  cur = cur->next = new_token(TK_IDENT, start, p);\n  continue;\n}\n```\n\nThe post-commit version threads through `decode_utf8`:\n\n```c\n// Read an identifier and returns the length of it.\n// If p does not point to a valid identifier, 0 is returned.\nstatic int read_ident(char *start) {\n  char *p = start;\n  uint32_t c = decode_utf8(&p, p);\n  if (!is_ident1(c))\n    return 0;\n\n  for (;;) {\n    char *q;\n    c = decode_utf8(&q, p);\n    if (!is_ident2(c))\n      return p - start;\n    p = q;\n  }\n}\n```\n\nAnd the dispatch becomes:\n\n```c\n// Identifier or keyword\nint ident_len = read_ident(p);\nif (ident_len) {\n  cur = cur->next = new_token(TK_IDENT, p, p + ident_len);\n  p += cur->len;\n  continue;\n}\n```\n\nThe do-while loop is gone. The new function decodes one code point at a time, checks the predicate, and stops at the first non-ident-character byte boundary. The `q`-variable trick in the loop is so that a non-ident-character lookahead can be backed out without consuming any bytes: `decode_utf8(&q, p)` advances `q`, and only `p = q` commits the advance.\n\nThe new `is_ident1` is a long range-table:\n\n```c\nbool is_ident1(uint32_t c) {\n  static uint32_t range[] = {\n    '_', '_', 'a', 'z', 'A', 'Z',\n    0x00A8, 0x00A8, 0x00AA, 0x00AA, 0x00AD, 0x00AD, 0x00AF, 0x00AF,\n    0x00B2, 0x00B5, 0x00B7, 0x00BA, 0x00BC, 0x00BE, 0x00C0, 0x00D6,\n    0x00D8, 0x00F6, 0x00F8, 0x00FF, 0x0100, 0x02FF, 0x0370, 0x167F,\n    /* ... about 30 more pairs ... */\n    -1,\n  };\n\n  return in_range(range, c);\n}\n```\n\nThe pairs encode `[start, end]` inclusive ranges. The terminator is `-1` (cast from `uint32_t` to a sentinel; the `in_range` helper checks `range[i] != -1`). The list of ranges is from C11 Annex D \u2014 the \"characters in identifiers\" Unicode-range table that the standard spells out.\n\nThe comment above `is_ident1` cites the source:\n\n> [https://www.sigbus.info/n1570#D] C11 allows not only ASCII but some multibyte characters in certan Unicode ranges to be used in an identifier.\n>\n> This function returns true if a given character is acceptable as the first character of an identifier.\n>\n> For example, \u00be (U+00BE) is a valid identifier because characters in 0x00BE-0x00C0 are allowed, while neither \u27d8 (U+27D8) nor '\u3000' (U+3000, full-width space) are allowed because they are out of range.\n\nThe C11 Annex D ranges are deliberately conservative \u2014 they exclude punctuation, whitespace, and control characters. The choice has its quirks (\u00be is a digit-shaped fraction, but it's in the \"letter-like\" range; \u27d8 is a mathematical operator and is excluded). Chibicc inherits whatever the standard says.\n\n`is_ident2` adds a few more ranges:\n\n```c\nbool is_ident2(uint32_t c) {\n  static uint32_t range[] = {\n    '0', '9', 0x0300, 0x036F, 0x1DC0, 0x1DFF, 0x20D0, 0x20FF,\n    0xFE20, 0xFE2F, -1,\n  };\n\n  return is_ident1(c) || in_range(range, c);\n}\n```\n\nThese are the *combining marks* \u2014 characters that attach to a preceding character (accents, diacritics, etc.) and are valid in identifiers but not as the first character.\n\nTest cases:\n\n```c\nint \u03c0 = 3;\nASSERT(3, \u03c0);\nASSERT(3, ({ int \u3042\u03b20\u00be=3; \u3042\u03b20\u00be; }));\n```\n\nThe first declares an identifier using the Greek letter pi (U+03C0). The second uses Japanese hiragana, Greek beta, an ASCII digit, and the fraction `\u00be` (U+00BE) in a single identifier \u2014 all four are in the C11 Annex D ranges.\n\n### `adb8b98` \u2014 `$` in identifiers\n\nA four-line edit. Add `'$', '$'` to both `is_ident1` and `is_ident2`'s range tables:\n\n```c\n   static uint32_t range[] = {\n-    '_', '_', 'a', 'z', 'A', 'Z',\n+    '_', '_', 'a', 'z', 'A', 'Z', '$', '$',\n     0x00A8, 0x00A8, 0x00AA, 0x00AA, 0x00AD, 0x00AD, 0x00AF, 0x00AF,\n```\n\n```c\n   static uint32_t range[] = {\n-    '0', '9', 0x0300, 0x036F, 0x1DC0, 0x1DFF, 0x20D0, 0x20FF,\n+    '0', '9', '$', '$', 0x0300, 0x036F, 0x1DC0, 0x1DFF, 0x20D0, 0x20FF,\n     0xFE20, 0xFE2F, -1,\n   };\n```\n\nThe C standard reserves `$` for implementation use; gcc and clang both accept it as an identifier character (the GCC `-fdollars-in-identifiers` flag, which is on by default on most targets). Chibicc follows. The test:\n\n```c\nASSERT(5, ({ int $$$=5; $$$; }));\n```\n\nThree dollar signs as an identifier. Legal under chibicc and gcc, illegal under strict C.\n\n### `2382777` \u2014 cross-prefix string concatenation\n\nThe biggest of the four. Two literals like `\"foo\" L\"bar\"` are *adjacent string literals*; the C standard says they concatenate into one literal. When the prefixes are mixed \u2014 one ordinary, one wide \u2014 the ordinary one is treated as if it had the wide prefix, so the concatenation is `L\"foobar\"`.\n\nPre-commit, `join_adjacent_string_literals` (in `preprocess.c`) handled the same-prefix case only: two adjacent ordinary strings concatenated, two adjacent wide strings concatenated, but a mix produced an error or wrong codegen.\n\nPost-commit, the function does two passes. The first pass detects adjacent-string runs that include both an ordinary and a wide prefix, and *re-tokenizes* the ordinary literals using the wide prefix's encoding. The second pass does the actual concatenation, now uniform.\n\nThe detection helper:\n\n```c\ntypedef enum {\n  STR_NONE, STR_UTF8, STR_UTF16, STR_UTF32, STR_WIDE,\n} StringKind;\n\nstatic StringKind getStringKind(Token *tok) {\n  if (!strcmp(tok->loc, \"u8\"))\n    return STR_UTF8;\n\n  switch (tok->loc[0]) {\n  case '\"': return STR_NONE;\n  case 'u': return STR_UTF16;\n  case 'U': return STR_UTF32;\n  case 'L': return STR_WIDE;\n  }\n  unreachable();\n}\n```\n\nThe first-byte switch reads the prefix off the token's source position. The `strcmp(tok->loc, \"u8\")` is a special case because `u` and `u8` start with the same byte. Note that `tok->loc` is not null-terminated in the usual sense \u2014 it points into the source buffer \u2014 but `strcmp` will stop at the first non-matching byte (the `\"` after the prefix), so this works as a startswith check.\n\nThe first pass:\n\n```c\nfor (Token *tok1 = tok; tok1->kind != TK_EOF;) {\n  if (tok1->kind != TK_STR || tok1->next->kind != TK_STR) {\n    tok1 = tok1->next;\n    continue;\n  }\n\n  StringKind kind = getStringKind(tok1);\n  Type *basety = tok1->ty->base;\n\n  for (Token *t = tok1->next; t->kind == TK_STR; t = t->next) {\n    StringKind k = getStringKind(t);\n    if (kind == STR_NONE) {\n      kind = k;\n      basety = t->ty->base;\n    } else if (k != STR_NONE && kind != k) {\n      error_tok(t, \"unsupported non-standard concatenation of string literals\");\n    }\n  }\n\n  if (basety->size > 1)\n    for (Token *t = tok1; t->kind == TK_STR; t = t->next)\n      if (t->ty->base->size == 1)\n        *t = *tokenize_string_literal(t, basety);\n\n  while (tok1->kind == TK_STR)\n    tok1 = tok1->next;\n}\n```\n\nFor each maximal run of adjacent `TK_STR` tokens, classify each, find the run's \"kind\" (which ought to be either all-`STR_NONE` or all-of-one-wide-kind, possibly with `STR_NONE` mixed in). Two different non-`STR_NONE` kinds in the same run are an error (\"unsupported non-standard concatenation\"). If the run has any wide kind, re-tokenize the `STR_NONE` members using `tokenize_string_literal`, exporting the right encoding.\n\n`tokenize_string_literal` is the new public entry point in `tokenize.c`:\n\n```c\nToken *tokenize_string_literal(Token *tok, Type *basety) {\n  Token *t;\n  if (basety->size == 2)\n    t = read_utf16_string_literal(tok->loc, tok->loc);\n  else\n    t = read_utf32_string_literal(tok->loc, tok->loc, basety);\n  t->next = tok->next;\n  return t;\n}\n```\n\nIt dispatches on the target element width (2 \u2192 UTF-16, otherwise \u2192 UTF-32, with the `basety` carrying the signedness for the wide case). The token's `loc` is re-read, with the first `tok->loc` argument standing in for both `start` and `quote` because the ordinary string has no prefix. `t->next` is patched to splice the new token into the existing chain.\n\nThe second pass \u2014 the actual concatenation \u2014 is structurally identical to pre-commit, just now operating on a uniform run.\n\nThe test cases pin the rewriting:\n\n```c\nASSERT(L'a', (L\"abc\" \"def\")[0]);\nASSERT(L'd', (L\"abc\" \"def\")[3]);\n\nASSERT(u'a', (u\"abc\" \"def\")[0]);\nASSERT(u'd', (u\"abc\" \"def\")[3]);\n\nASSERT(L'\u3042', (\"\u3042\" L\"\")[0]);\nASSERT(0343, (\"\\343\\201\\202\" L\"\")[0]);  // first byte of '\u3042' UTF-8\n```\n\nThe last test case is informative: `(\"\u3042\" L\"\")[0]` is `L'\u3042'` (the wide character), but `(\"\\343\\201\\202\" L\"\")[0]` is `0343` (the first byte, treated as a wide character). The reason is that `\\343\\201\\202` is three octal escapes inside the ordinary string, so the literal contains three bytes (not the character `\u3042`); when the run gets re-tokenized as wide, those three bytes become three `int[]` elements, and the first one is `0343`. The two literals look equivalent because UTF-8 makes them so at the byte level, but they aren't equivalent under cross-prefix promotion.\n\n### `2b2fa25` \u2014 UTF-8 BOM skip\n\nA seven-line addition to `tokenize_file`, before the canonicalization step:\n\n```c\n// UTF-8 texts may start with a 3-byte \"BOM\" marker sequence.\n// If exists, just skip them because they are useless bytes.\n// (It is actually not recommended to add BOM markers to UTF-8\n// texts, but it's not uncommon particularly on Windows.)\nif (!memcmp(p, \"\\xef\\xbb\\xbf\", 3))\n  p += 3;\n```\n\nThe byte order mark for UTF-8 is the three-byte sequence `EF BB BF`. UTF-16 needs a BOM because byte order matters; UTF-8 doesn't, but Windows tools sometimes write one anyway. Chibicc strips it.\n\nThe pre-tokenize pass count is now four: BOM skip, newline canonicalization, backslash-newline removal, universal-character-name conversion. All four are destructive in-place rewrites of the read-from-file buffer; together they make up the \"normalize the source before tokenizing\" stage.\n\nA driver test exercises the BOM strip end-to-end:\n\n```bash\n# BOM marker\nprintf '\\xef\\xbb\\xbfxyz\\n' | $chibicc -E -o- - | grep -q '^xyz'\ncheck 'BOM marker'\n```\n\n`-E` runs the preprocessor and dumps the result; the test checks that the BOM is gone and `xyz` made it through.\n\n**Where we are.** Identifiers can contain UTF-8-encoded multibyte characters in the C11 Annex D ranges, plus `$`. Adjacent string literals across prefixes concatenate by promoting ordinary literals to the wide encoding. UTF-8 BOM markers are silently stripped. The `read_ident` helper is now the canonical identifier recognizer, replacing the inline ASCII loop.\n\n---\n\n## 19.7 \u2014 Designated initializers\n\n> `git checkout c618c3b582de1d0b10b334a4f2ba6b85d5128940` \u2014 *Add array designated initializer*\n>\n> `git checkout 835cd24b2c4598ee784d8bfd1c0427bfa948b947` \u2014 *Allow array designators to initialize incomplete arrays*\n>\n> `git checkout 691c4fac1529eaf1d825ca6093800912a4df3c91` \u2014 *[GNU] Allow to omit \"=\" in designated initializers*\n>\n> `git checkout 67f5834378660abf271722a16294a634106d047e` \u2014 *Add struct designated initializer*\n>\n> `git checkout 31dc1dfa211ee27e74907ce3aa3986401dcedb82` \u2014 *Add union designated initializer*\n>\n> `git checkout 95eb5b01b30b24d68cbeb3991f65c617fc2a35cb` \u2014 *Handle struct designator for anonymous struct member*\n\nSix commits. All in `parse.c`. The arc is small in line count (~150 lines added across the six commits) but invasive in structure \u2014 the same handful of helpers (`array_initializer1`, `array_initializer2`, `struct_initializer1`, `struct_initializer2`, `union_initializer`, `count_array_init_elements`) get reshaped multiple times, and a new helper `designation` sits at the heart of the new path.\n\nDesignated initializers are a C99 addition that lets a programmer pick which element or field a particular initializer goes to:\n\n```c\nint x[10] = {[3] = 1, [7] = 2};\nstruct point p = {.y = 3, .x = 5};\n```\n\nThe cursor that the un-designated form advances left-to-right can now jump. Rui's commits introduce the designator parsing, then extend it to incomplete arrays, then add the GNU `=`-omission shortcut, then add struct support, then union, then anonymous-struct.\n\n### `c618c3b` \u2014 array designated initializer\n\nThe first commit. Three new functions and one signature change.\n\n`array_designator` reads a `[index]` designator and returns the index:\n\n```c\n// array-designator = \"[\" const-expr \"]\"\n//\n// C99 added the designated initializer to the language, which allows\n// programmers to move the \"cursor\" of an initializer to any element.\n// The syntax looks like this:\n//\n//   int x[10] = { 1, 2, [5]=3, 4, 5, 6, 7 };\n//\n// `[5]` moves the cursor to the 5th element, so the 5th element of x\n// is set to 3. Initialization then continues forward in order, so\n// 6th, 7th, 8th and 9th elements are initialized with 4, 5, 6 and 7,\n// respectively. Unspecified elements (in this case, 3rd and 4th\n// elements) are initialized with zero.\n//\n// Nesting is allowed, so the following initializer is valid:\n//\n//   int x[5][10] = { [5][8]=1, 2, 3 };\n//\n// It sets x[5][8], x[5][9] and x[6][0] to 1, 2 and 3, respectively.\nstatic int array_designator(Token **rest, Token *tok, Type *ty) {\n  Token *start = tok;\n  int i = const_expr(&tok, tok->next);\n  if (i >= ty->array_len)\n    error_tok(start, \"array designator index exceeds array bounds\");\n  *rest = skip(tok, \"]\");\n  return i;\n}\n```\n\nThe doc comment is unusually long \u2014 Rui wrote it as a mini-tutorial because the semantics aren't trivial. The cursor-jump rule is what makes the second example `{[5][8]=1, 2, 3}` set `x[5][8]`, `x[5][9]`, `x[6][0]` (the un-designated `2` and `3` follow the designated `1` in storage order, not the index that the designator named).\n\n`designation` is the recursive entry point that handles a single designation (which may be a chain of `[...]` designators) and the value it designates:\n\n```c\n// designation = (\"[\" const-expr \"]\")* \"=\" initializer\nstatic void designation(Token **rest, Token *tok, Initializer *init) {\n  if (equal(tok, \"[\")) {\n    if (init->ty->kind != TY_ARRAY)\n      error_tok(tok, \"array index in non-array initializer\");\n    int i = array_designator(&tok, tok, init->ty);\n    designation(&tok, tok, init->children[i]);\n    array_initializer2(rest, tok, init, i + 1);\n    return;\n  }\n\n  tok = skip(tok, \"=\");\n  initializer2(rest, tok, init);\n}\n```\n\nWhen the current token is `[`, parse one array designator, recurse into the corresponding child (which handles any nested designator like `[5][8]`), and *then* call `array_initializer2` with the new cursor position. That last step is the key: after the designation lands its initializer in `children[i]`, any un-designated initializers that follow (like the `2, 3` in `[5][8]=1, 2, 3`) need to fill in `children[i+1]`, `children[i+2]`, and so on. `array_initializer2` is the un-designated array-init helper, which now takes a starting cursor position so the post-designation continuation can resume from the right index.\n\n`array_initializer2` gains an `int i` parameter:\n\n```c\n- static void array_initializer2(Token **rest, Token *tok, Initializer *init) {\n+ static void array_initializer2(Token **rest, Token *tok, Initializer *init, int i) {\n    if (init->is_flexible) {\n      int len = count_array_init_elements(tok, init->ty);\n      *init = *new_initializer(array_of(init->ty->base, len), false);\n    }\n\n-   for (int i = 0; i < init->ty->array_len && !is_end(tok); i++) {\n+   for (; i < init->ty->array_len && !is_end(tok); i++) {\n+     Token *start = tok;\n      if (i > 0)\n        tok = skip(tok, \",\");\n+\n+     if (equal(tok, \"[\")) {\n+       *rest = start;\n+       return;\n+     }\n+\n      initializer2(&tok, tok, init->children[i]);\n    }\n    *rest = tok;\n  }\n```\n\nTwo changes. First, the loop variable starts from the passed-in `i` rather than 0. Second, when the loop sees a `[` designator, it bails \u2014 restoring `*rest` to the comma-or-bracket position and returning without consuming the designator. The control flow then unwinds back to whichever caller is the outermost `array_initializer1`, which will see the `[` and route through its own designator handling.\n\n`array_initializer1` (the brace form) gains its own designator handling:\n\n```c\nstatic void array_initializer1(Token **rest, Token *tok, Initializer *init) {\n  tok = skip(tok, \"{\");\n  bool first = true;\n\n  if (init->is_flexible) {\n    int len = count_array_init_elements(tok, init->ty);\n    *init = *new_initializer(array_of(init->ty->base, len), false);\n  }\n\n  for (int i = 0; !consume_end(rest, tok); i++) {\n    if (!first)\n      tok = skip(tok, \",\");\n    first = false;\n\n    if (equal(tok, \"[\")) {\n      i = array_designator(&tok, tok, init->ty);\n      designation(&tok, tok, init->children[i]);\n      continue;\n    }\n\n    if (i < init->ty->array_len)\n      initializer2(&tok, tok, init->children[i]);\n    else\n      tok = skip_excess_element(tok);\n  }\n}\n```\n\nThe new `if (equal(tok, \"[\")) { ... }` arm reads the designator, dispatches into `designation` (which recurses into the child and handles any continuation un-designated initializers), and then `continue`s the loop. The `i` variable's reassignment makes the *next* iteration's auto-increment pick up from the designated index, so a sequence like `{1, 2, [0]=4, 5}` re-overwrites `children[0]` with 4 and then `children[1]` with 5 \u2014 see the test:\n\n```c\nASSERT(4, ({ int x[3]={1, 2, 3, [0]=4, 5}; x[0]; }));\nASSERT(5, ({ int x[3]={1, 2, 3, [0]=4, 5}; x[1]; }));\nASSERT(3, ({ int x[3]={1, 2, 3, [0]=4, 5}; x[2]; }));\n```\n\nThe first three initializers set `x[0]=1, x[1]=2, x[2]=3`. The designator `[0]=4` then jumps the cursor to index 0 and overwrites `x[0]` to 4. The next un-designated `5` advances the cursor to index 1 and overwrites `x[1]` to 5. The result is `{4, 5, 3}`. Designators *replace*, they don't merge.\n\nThere's also a `bool first` flag added to track \"is this the loop's first iteration\" \u2014 replacing the older `if (i > 0) tok = skip(tok, \",\")` form. With designators, `i` doesn't reliably increment by one each iteration, so the flag is needed for correct comma handling.\n\n### `835cd24` \u2014 incomplete-array sizing with designators\n\nWhen an array's length is omitted (`int x[] = {1, 2, 3}`), the parser counts initializer elements to decide the length. With designators, the length should be `max(designator_index, sequential_count) + 1`:\n\n```c\nint x[] = {[10-3] = 1, 2, 3};   // length is 10 \u2014 index 7 is the largest, plus two more\n```\n\nThe `count_array_init_elements` helper is rewritten to handle both forms:\n\n```c\nstatic int count_array_init_elements(Token *tok, Type *ty) {\n  bool first = true;\n  Initializer *dummy = new_initializer(ty->base, true);\n\n  int i = 0, max = 0;\n\n  while (!consume_end(&tok, tok)) {\n    if (!first)\n      tok = skip(tok, \",\");\n    first = false;\n\n    if (equal(tok, \"[\")) {\n      i = const_expr(&tok, tok->next);\n      if (equal(tok, \"...\"))\n        i = const_expr(&tok, tok->next);\n      tok = skip(tok, \"]\");\n      designation(&tok, tok, dummy);\n    } else {\n      initializer2(&tok, tok, dummy);\n    }\n\n    i++;\n    max = MAX(max, i);\n  }\n  return max;\n}\n```\n\nThe walk maintains `i` (the running cursor) and `max` (the largest cursor seen). For a designator, `i` jumps to the designated index. For a non-designator, `i` continues from wherever it was. Each iteration ends with `max = MAX(max, i)` after the `i++`. Returned: `max`, which is the array's length.\n\nThe `if (equal(tok, \"...\"))` line is forward-compatible plumbing for GCC range designators (`[3 ... 7] = 1`, which initializes indices 3 through 7 to 1). Chibicc's `count_array_init_elements` skips both endpoints' const-exprs, treating the range as a single jump to the *upper* endpoint. The actual range expansion isn't supported elsewhere, so the feature is half-implemented \u2014 chibicc accepts the syntax in counting but doesn't honor it in the rest of the parser. Errata candidate: range designators in initializers are silently mishandled.\n\nThe `array_initializer1` body is also modified \u2014 the `if (init->is_flexible)` block at the top is added a *second time*, before the original block. Looking at the post-commit code:\n\n```c\nstatic void array_initializer1(Token **rest, Token *tok, Initializer *init) {\n  tok = skip(tok, \"{\");\n\n  if (init->is_flexible) {\n    int len = count_array_init_elements(tok, init->ty);\n    *init = *new_initializer(array_of(init->ty->base, len), false);\n  }\n\n  bool first = true;\n\n  if (init->is_flexible) {\n    int len = count_array_init_elements(tok, init->ty);\n    *init = *new_initializer(array_of(init->ty->base, len), false);\n  }\n  // ...\n}\n```\n\nThe first `if (init->is_flexible)` block does the work; the second is dead code, because after the first `*init = *new_initializer(..., false)` call the `is_flexible` flag is `false`. The duplication is a small bug \u2014 the new block was meant to *replace* the old one, but the old one wasn't deleted. Errata candidate: dead-code duplicate `count_array_init_elements` call in `array_initializer1`. The duplication doesn't change behavior; it's just an extra CPU cycle that the compiler optimizer probably elides.\n\nThe test cases:\n\n```c\nASSERT(10, ({ char x[]={[10-3]=1,2,3}; sizeof(x); }));\nASSERT(20, ({ char x[][2]={[8][1]=1,2}; sizeof(x); }));\n```\n\nThe first: index 7 designates `x[7]`, then `2` and `3` go to `x[8]` and `x[9]`. Length is 10. The second: a 2D array where `[8][1]=1` designates `x[8][1]`, then `2` goes to `x[9][0]`. Outer length is 10, inner is 2 (declared), total bytes 20.\n\n### `691c4fa` \u2014 GNU `=`-omission\n\nA two-line edit. The `=` in `[3] = 1` becomes optional:\n\n```c\n- // designation = (\"[\" const-expr \"]\")* \"=\" initializer\n+ // designation = (\"[\" const-expr \"]\")* \"=\"? initializer\n  static void designation(Token **rest, Token *tok, Initializer *init) {\n    if (equal(tok, \"[\")) {\n      // ...\n      return;\n    }\n\n-   tok = skip(tok, \"=\");\n+   if (equal(tok, \"=\"))\n+     tok = tok->next;\n    initializer2(rest, tok, init);\n  }\n```\n\nThe GNU extension. `{[3] 1}` is now equivalent to `{[3] = 1}`. The test:\n\n```c\nASSERT(7, ((int[10]){ [3] 7 })[3]);\n```\n\nThe space between `[3]` and `7` is parsed as the elided `=`. The pre-commit form would have errored on the missing `=`.\n\n### `67f5834` \u2014 struct designated initializer\n\nThe biggest of the six commits. Three changes: `struct_designator` is the new helper for `.field` parsing, `designation` learns about it, and `struct_initializer2` gains a `Member *mem` parameter.\n\n`struct_designator`:\n\n```c\n// struct-designator = \".\" ident\nstatic Member *struct_designator(Token **rest, Token *tok, Type *ty) {\n  tok = skip(tok, \".\");\n  if (tok->kind != TK_IDENT)\n    error_tok(tok, \"expected a field designator\");\n\n  for (Member *mem = ty->members; mem; mem = mem->next) {\n    if (mem->name->len == tok->len && !strncmp(mem->name->loc, tok->loc, tok->len)) {\n      *rest = tok->next;\n      return mem;\n    }\n  }\n\n  error_tok(tok, \"struct has no such member\");\n}\n```\n\nRead `.`, then an identifier, then look up the member by name in the struct's `members` list. Returns the matched `Member *`, or errors if the name isn't found.\n\n`designation` gains two new arms:\n\n```c\nif (equal(tok, \".\") && init->ty->kind == TY_STRUCT) {\n  Member *mem = struct_designator(&tok, tok, init->ty);\n  designation(&tok, tok, init->children[mem->idx]);\n  init->expr = NULL;\n  struct_initializer2(rest, tok, init, mem->next);\n  return;\n}\n\nif (equal(tok, \".\"))\n  error_tok(tok, \"field name not in struct or union initializer\");\n```\n\nMirror of the array case: parse `.field`, recurse into the child for any nested designation, then call `struct_initializer2` with `mem->next` as the cursor so post-designation un-designated initializers continue from the right field.\n\nThe `init->expr = NULL` line is subtle. It clears any prior scalar-form initializer that might have been set on the struct (e.g. from `{x}` syntax that initializes a struct via a single value of compatible type). When a designation arrives, the struct-form initializer takes over and the prior scalar binding has to be discarded. The line catches a case the test cases exercise:\n\n```c\nASSERT(0, ({ typedef struct { int a,b; } T; T x={1,2}; T y[]={x, [0].b=3}; y[0].a; }));\nASSERT(3, ({ typedef struct { int a,b; } T; T x={1,2}; T y[]={x, [0].b=3}; y[0].b; }));\n```\n\nThe first initializer of `y[0]` is `x` \u2014 a struct value, which sets `y[0]`'s `expr` to copy from `x`. The second initializer is the designation `[0].b=3`, which jumps back to `y[0]` and sets `b=3`. The `init->expr = NULL` line on `y[0]` ensures the struct copy is gone after the designation; `y[0].a` is now `0` (zeroed), and `y[0].b` is `3`. Without the line, both `expr` and `b`'s child would be set, and codegen would emit both stores, with the order determining the final value.\n\n`struct_initializer2` gains the `Member *mem` parameter, like `array_initializer2`'s `int i`:\n\n```c\nstatic void struct_initializer2(Token **rest, Token *tok, Initializer *init, Member *mem) {\n  bool first = true;\n\n  for (; mem && !is_end(tok); mem = mem->next) {\n    Token *start = tok;\n\n    if (!first)\n      tok = skip(tok, \",\");\n    first = false;\n\n    if (equal(tok, \"[\") || equal(tok, \".\")) {\n      *rest = start;\n      return;\n    }\n\n    initializer2(&tok, tok, init->children[mem->idx]);\n  }\n  *rest = tok;\n}\n```\n\nThe bail-on-designator pattern, parallel to `array_initializer2`. Note the `equal(tok, \"[\")` check too: a struct-of-arrays might have a `[index]` designator that names an array-typed member's element rather than the struct's field. The bail propagates the designator back up to `array_initializer1` or `struct_initializer1`'s outermost handler.\n\n`struct_initializer1` (the brace form) similarly grows a `.` arm:\n\n```c\nstatic void struct_initializer1(Token **rest, Token *tok, Initializer *init) {\n  tok = skip(tok, \"{\");\n\n  Member *mem = init->ty->members;\n  bool first = true;\n\n  while (!consume_end(rest, tok)) {\n    if (!first)\n      tok = skip(tok, \",\");\n    first = false;\n\n    if (equal(tok, \".\")) {\n      mem = struct_designator(&tok, tok, init->ty);\n      designation(&tok, tok, init->children[mem->idx]);\n      mem = mem->next;\n      continue;\n    }\n\n    if (mem) {\n      initializer2(&tok, tok, init->children[mem->idx]);\n      mem = mem->next;\n    } else {\n      tok = skip_excess_element(tok);\n    }\n  }\n}\n```\n\nThe cursor variable here is `mem` (a `Member *`) rather than `i` (an `int`). After a designator, `mem = mem->next` continues the un-designated walk from one past the designated field.\n\n### `31dc1df` \u2014 union designated initializer\n\nA union initializer is unusual: there's only one initializer for the whole union, and pre-commit it always set the *first* member. With designators, any member can be initialized.\n\nThe `Initializer` struct gains a `mem` field:\n\n```c\nstruct Initializer {\n  // ...\n\n  // Only one member can be initialized for a union.\n  // `mem` is used to clarify which member is initialized.\n  Member *mem;\n};\n```\n\n`designation` grows a union arm:\n\n```c\nif (equal(tok, \".\") && init->ty->kind == TY_UNION) {\n  Member *mem = struct_designator(&tok, tok, init->ty);\n  init->mem = mem;\n  designation(rest, tok, init->children[mem->idx]);\n  return;\n}\n```\n\nWhere the struct case calls `struct_initializer2` to continue with un-designated fields after, the union case doesn't \u2014 a union has one initialized member, and that member is named by `init->mem`. After the designation, the function returns directly.\n\n`union_initializer` itself gains the designator path and explicitly sets `init->mem`:\n\n```c\nstatic void union_initializer(Token **rest, Token *tok, Initializer *init) {\n  // Unlike structs, union initializers take only one initializer,\n  // and that initializes the first union member by default.\n  // You can initialize other member using a designated initializer.\n  if (equal(tok, \"{\") && equal(tok->next, \".\")) {\n    Member *mem = struct_designator(&tok, tok->next, init->ty);\n    init->mem = mem;\n    designation(&tok, tok, init->children[mem->idx]);\n    *rest = skip(tok, \"}\");\n    return;\n  }\n\n  init->mem = init->ty->members;\n\n  if (equal(tok, \"{\")) {\n    initializer2(&tok, tok->next, init->children[0]);\n    consume(&tok, tok, \",\");\n    *rest = skip(tok, \"}\");\n  } else {\n    initializer2(rest, tok, init->children[0]);\n  }\n}\n```\n\nThree branches: brace-and-dot (the designated form), brace-only (the default form initializing the first member), no-brace (a single-expression initializer like `union x = 5`, also initializing the first member). All three set `init->mem`.\n\nThe lowering paths \u2014 `create_lvar_init` and `write_gvar_data` \u2014 both have to read `init->mem` rather than blindly using `ty->members`:\n\n```c\n// create_lvar_init's union case:\nif (ty->kind == TY_UNION) {\n  Member *mem = init->mem ? init->mem : ty->members;\n  InitDesg desg2 = {desg, 0, mem};\n  return create_lvar_init(init->children[mem->idx], mem->ty, &desg2, tok);\n}\n\n// write_gvar_data's union case:\nif (ty->kind == TY_UNION) {\n  if (!init->mem)\n    return cur;\n  return write_gvar_data(cur, init->children[init->mem->idx],\n                         init->mem->ty, buf, offset);\n}\n```\n\nThe `init->mem ? init->mem : ty->members` fallback in `create_lvar_init` is for cases where the union initializer was implicit \u2014 `union { int a; char b; } u = {};`, where `init->mem` is null because no designator nor explicit member-initializer ran. In that case the first member is the default. The `write_gvar_data` version is stricter: a global union with no `init->mem` is treated as fully zero (the relocation walk returns without writing).\n\nThe test fixtures include `union { int a; char b[4]; } g50 = {.b[2]=0x12};` at file scope, and the assertion `ASSERT(0x00120000, g50.a);` pins the byte layout \u2014 the `0x12` lands at offset 2 of the four-byte storage, so reading the same bytes as an `int` (little-endian) gives `0x00120000`.\n\n### `95eb5b0` \u2014 anonymous-struct designator\n\nThe smallest of the six. Twelve lines, in `struct_designator`. Anonymous struct members (Chapter 18 \u00a718.6) are members with `mem->name == NULL` whose type is itself a `TY_STRUCT`; their fields flatten into the outer struct's namespace, so `outer.a` accesses an `a` that lives in the anonymous inner struct.\n\nThe designator path needs to follow that flattening:\n\n```c\nstatic Member *struct_designator(Token **rest, Token *tok, Type *ty) {\n  Token *start = tok;\n  tok = skip(tok, \".\");\n  if (tok->kind != TK_IDENT)\n    error_tok(tok, \"expected a field designator\");\n\n  for (Member *mem = ty->members; mem; mem = mem->next) {\n    // Anonymous struct member\n    if (mem->ty->kind == TY_STRUCT && !mem->name) {\n      if (get_struct_member(mem->ty, tok)) {\n        *rest = start;\n        return mem;\n      }\n      continue;\n    }\n\n    // Regular struct member\n    if (mem->name->len == tok->len && !strncmp(mem->name->loc, tok->loc, tok->len)) {\n      *rest = tok->next;\n      return mem;\n    }\n  }\n\n  error_tok(tok, \"struct has no such member\");\n}\n```\n\nFor each member, if it's a regular named member, match by name as before. If it's an anonymous struct member, recurse into its members (via `get_struct_member`, the \u00a718.6 helper that walks the anonymous-flattening chain). On a match, return *the anonymous member*, not the named member inside it \u2014 the cursor then does the right thing because the recursive `designation` call will descend through the anonymous member's children.\n\nThe `*rest = start` line in the anonymous-match arm is what makes the recursion work. The `start` is the position *before* the leading `.`, so when the recursive call to `designation` (from the caller) sees the same `.field` it will parse it again, this time against the anonymous inner struct's type. Each layer of anonymous nesting peels off one designator-parse-and-recurse iteration.\n\nThe test:\n\n```c\nASSERT(1, ({ struct { struct { int a; struct { int b; }; }; int c; } x={1,2,3,.b=4,5}; x.a; }));\nASSERT(4, ({ struct { struct { int a; struct { int b; }; }; int c; } x={1,2,3,.b=4,5}; x.b; }));\nASSERT(5, ({ struct { struct { int a; struct { int b; }; }; int c; } x={1,2,3,.b=4,5}; x.c; }));\n```\n\nThe struct has an outer struct with: an anonymous inner struct (containing `int a` and an inner-inner anonymous struct containing `int b`), and `int c`. The initializer is `{1,2,3,.b=4,5}`. The first three un-designated initializers fill `a`, `b`, `c` in storage order: `a=1`, `b=2`, `c=3`. The designator `.b=4` jumps to the anonymous-of-anonymous `b` and overwrites it to `4`. The next `5` continues from one-past-`b`'s position, which is the next outer member \u2014 `c`. So `c=5`.\n\nThe walking is *layered* \u2014 `struct_designator` returns the outermost anonymous member (the inner struct containing `b`), `designation` recurses into that member's `Initializer` child, which itself recurses into the inner-inner struct's child, until eventually the leaf `b`'s Initializer is set to `4`. The `*rest = start` trick lets the same `.b` token be re-parsed at each anonymous layer.\n\n**Where we are.** Designated initializers work for arrays, structs, and unions, with nested designators (`[5][8]`, `.a.b`), the GNU `=`-omission shortcut, and anonymous-struct member designators. The `Initializer` tree gains the `mem` field (for unions) but no other shape changes. The `count_array_init_elements` helper is now designator-aware, so `int x[] = {[7] = 1}` produces an 8-element array. The `array_initializer1` helper carries a small dead-code duplication from `835cd24` that doesn't affect behavior. Range designators (`[3 ... 7]`) are partially recognized in counting but not honored in elaboration \u2014 errata candidate.\n\n---\n\n## Where we end the chapter\n\nTwenty-four commits, seven sections.\n\n| Hash | What landed |\n|---|---|\n| `e27417f` | `__DATE__` and `__TIME__`. Two `format_*` helpers + one `time(NULL)` call. Local-zone date and time, fixed at cc1 startup. |\n| `0e77f3d` | `__COUNTER__` (GCC). Stateful builtin macro: a `static int i` returns 0, 1, 2, ... per expansion. Uses Chapter 17 \u00a717.5.3's `Macro->handler`. |\n| `74bcec5` | Newline canonicalization. New tokenizer pre-pass: `\\r\\n` \u2192 `\\n`, bare `\\r` \u2192 `\\n`. Runs before backslash-newline removal. |\n| `c31886a` | Universal character names. New `unicode.c` with `encode_utf8`. New tokenizer pre-pass `convert_universal_chars` rewrites `\\u`/`\\U` to UTF-8 bytes in place. |\n| `a57c661` | Wide character literal accepts multibyte. New `decode_utf8` in `unicode.c`. `read_char_literal` returns an `int`. The narrow form post-narrows to `(char)`. Closes the Chapter 17 `L''` \u2261 `''` errata. |\n| `454618c` | UTF-16 character literal. New `u'X'` recognized; type `unsigned short`; truncated to 16 bits. `read_char_literal` gains a `Type *ty` parameter. |\n| `2dac3af` | UTF-32 character literal. New `U'X'` recognized; type `unsigned int`. |\n| `57b21fe` | UTF-8 string literal. New `u8\"X\"` recognized; type `char[]`; byte-equivalent to ordinary string for valid UTF-8 input. `read_string_literal` gains a `quote` parameter. |\n| `9cabe1f` | UTF-16 string literal. New `read_utf16_string_literal`. UTF-8 source is decoded and re-encoded, with surrogate pairs for code points above U+FFFF. |\n| `c467ee6` | UTF-32 string literal. New `read_utf32_string_literal`, parameterized by element type. |\n| `cae061a` | Wide string literal. `L\"X\"` reuses `read_utf32_string_literal` with `ty_int`. |\n| `36230e0` | UTF-16 string literal initializer. `string_initializer` dispatches on `init->ty->base->size`, with cases for size 1 and size 2. The `default: unreachable()` arm is in place. |\n| `6adba75` | UTF-32 string literal initializer. Adds the size-4 case to `string_initializer`'s switch. |\n| `e4491b8` | `__STDC_UTF_16__` and `__STDC_UTF_32__`. Two `define_macro` lines in `init_macros`. |\n| `0e5d250` | UTF-8 in identifiers. `is_ident1` / `is_ident2` move to `unicode.c` and take `uint32_t`. New `read_ident` in `tokenize.c` decodes UTF-8 to recognize multibyte identifiers. C11 Annex D ranges. |\n| `adb8b98` | `$` in identifiers (GCC). Two range-table entries added to `is_ident1` and `is_ident2`. |\n| `2382777` | Cross-prefix string concatenation. New `StringKind` enum and `getStringKind` helper. New public `tokenize_string_literal` in `tokenize.c`. `join_adjacent_string_literals` runs two passes: re-tokenize ordinary literals as wide if mixed, then concatenate. |\n| `2b2fa25` | UTF-8 BOM skip. Three-byte `EF BB BF` at file start is silently stripped before the existing pre-tokenize passes. |\n| `c618c3b` | Array designated initializer. New `array_designator` and `designation` helpers. `array_initializer2` gains `int i`. `array_initializer1` learns the `[index]` arm. |\n| `835cd24` | Incomplete array sized via designators. `count_array_init_elements` rewritten: tracks running `i` and `max(i)`. `array_initializer1` picks up a duplicate `is_flexible` block \u2014 dead code, errata. Range designator `[3 ... 7]` syntax is recognized in counting but not honored in elaboration \u2014 errata. |\n| `691c4fa` | GNU `=`-omission. `designation`'s trailing `skip(tok, \"=\")` becomes optional. |\n| `67f5834` | Struct designated initializer. New `struct_designator`. `designation` learns the `.` arm. `struct_initializer2` gains `Member *mem`. `struct_initializer1` learns `.field` parsing. The `init->expr = NULL` clears prior scalar bindings on re-designation. |\n| `31dc1df` | Union designated initializer. `Initializer` gains `Member *mem`. `union_initializer` and `designation` set it. `create_lvar_init` and `write_gvar_data` read it. |\n| `95eb5b0` | Anonymous-struct designator. `struct_designator` checks for anonymous members via `get_struct_member` and returns the anonymous outer member when an inner field name matches; the recursive `designation` peels one anonymous layer per call. |\n\nSix structural moves carry forward.\n\nThe first is *the tokenizer-pre-pass family*. Through Chapter 18 the tokenizer ran one pre-pass on the source buffer (`remove_backslash_newline`). After Chapter 19 the count is four: BOM strip, newline canonicalization, backslash-newline removal, universal-character-name conversion. All four are destructive in-place rewrites. The order matters \u2014 BOM first (it has to come before any byte-position-sensitive pass), newline second (so backslash-newline can recognize Windows endings), backslash-newline third, UCN conversion fourth.\n\nThe second is *the UTF-8 baseline*. Chibicc's source-file convention is now formally UTF-8: the BOM strip handles Windows quirks, `decode_utf8` is the canonical multi-byte reader, and the four `\\u`/`\\U`/`u8`/`u`/`U`/`L` entry points all route source bytes through it. The two helpers `encode_utf8` and `decode_utf8` form a small two-function module in the new `unicode.c` file. `unicode.c` also holds the C11 Annex D range tables for `is_ident1` and `is_ident2`.\n\nThe third is *the four-prefix string-literal family*. Ordinary, `u8`, `u`, `U`, `L`. Each prefix has its own `read_*_string_literal` helper (with `u8` reusing the ordinary one and `L` reusing UTF-32's). The `Initializer` tree's `string_initializer` dispatches on element size. The cross-prefix concatenation handler in the preprocessor re-tokenizes ordinary literals to match a wide neighbor, so a mixed run becomes uniform before the second-pass concatenation runs. The pre-Chapter-19 `string_initializer` was a single byte-by-byte loop; the post-Chapter-19 version is a switch on `init->ty->base->size`.\n\nThe fourth is *designated initializers*. The `Initializer` tree itself doesn't change shape \u2014 children still live in a fixed-size array indexed by position \u2014 but the parser now has a recursive `designation` helper that drives cursor jumps, and three callers (`array_initializer1`, `struct_initializer1`, `union_initializer`) and three companion helpers (`array_initializer2`, `struct_initializer2`, `count_array_init_elements`) cooperate via the cursor parameter. The `Initializer` struct picks up one new field, `Member *mem`, which is read by `create_lvar_init` and `write_gvar_data` for the union case.\n\nThe fifth is *the StringKind enum and `getStringKind` helper*. These let the preprocessor's adjacent-string concatenation classify literals by prefix without re-parsing. Five kinds: NONE (ordinary), UTF8, UTF16, UTF32, WIDE. The classification is purely syntactic (looks at the token's `loc` byte), and the public `tokenize_string_literal` provides the re-tokenization path.\n\nThe sixth is *the anonymous-struct designator integration*. Chapter 18 \u00a718.6's `get_struct_member` recursion is reused unchanged in `struct_designator`'s anonymous-match arm. The trick of returning the *outer* anonymous member while marking `*rest = start` lets the same `.field` token be re-parsed at each anonymous layer.\n\nOne errata item is *closed* by Chapter 19.\n\n- The Chapter 17 \u00a717.5.3 `L''` \u2261 `''` punt \u2014 closed by `a57c661`. `L'X'` is now a 32-bit Unicode code point, distinct from the narrow `'X'` form.\n\nTwo new errata candidates surface in \u00a719.7.\n\n- The dead-code duplicate `if (init->is_flexible)` block in `array_initializer1` (introduced by `835cd24`). Behavior is unaffected; the second `count_array_init_elements` call is dead because the first sets `is_flexible` to `false`.\n- Range designators `[3 ... 7]` are syntactically accepted in `count_array_init_elements` (so an incomplete-array's length is computed using the upper endpoint) but not honored in `array_initializer1` or `array_initializer2` (which would need to repeat the designation across the range). The half-implementation means `int x[] = {[3 ... 7] = 1}` declares an 8-element array but only initializes `x[3]`.\n\nThe pre-factor-before-feature count is unchanged at nine \u2014 Chapter 19 adds features by direct change rather than by laying groundwork that a later commit fills in. (The closest candidate was `read_char_literal`'s `Type *ty` parameter in `454618c`, but that change and the use of the new parameter are in the same commit; `read_string_literal`'s `quote` parameter in `57b21fe` is similarly same-commit.)\n\nThe canonicalization-at-parse-time count is unchanged at nine. None of the \u00a719.7 commits desugars one syntactic shape into another at parse time \u2014 the designator handling threads through `Initializer` tree population without reshaping the tree. The closest candidate was the cross-prefix re-tokenization in `2382777`, which *does* change a token's contents at preprocessing time; the prose treats it as a token-level rewrite within the preprocessor rather than parse-time canonicalization, since it's in `preprocess.c` rather than `parse.c`.\n\nThe psABI conformance count is unchanged at sixteen. Chapter 19's commits don't touch the SysV AMD64 ABI surface.\n\nStanding notes worth carrying forward.\n\n- The hideset on Token, the Token->origin chain, the eval-quartet duplication \u2014 all unchanged.\n- The cc1-vs-driver split \u2014 unchanged.\n- The `Initializer` tree gains the `Member *mem` field for the union case (set by `union_initializer` or `designation`'s union arm; read by `create_lvar_init` and `write_gvar_data`'s union branches).\n- The `Relocation` mechanism \u2014 unchanged. UTF-16 and UTF-32 string literals don't need relocations because their contents are constant byte arrays; `gen_data` emits them as raw `.byte` sequences via the same path as ordinary string literals.\n- The local-vs-global split \u2014 stable.\n- The `is_static` default in `new_gvar` \u2014 unchanged.\n- The `is_definition` flag on `Obj` \u2014 unchanged.\n- The `is_unsigned` flag on `Type` \u2014 used for UTF-16 (`ty_ushort`) and UTF-32 (`ty_uint`) string-literal element types, which are unsigned by C-spec; wide (`ty_int`) is signed.\n- The `__va_area__` magic name \u2014 unchanged.\n- The register-save-area layout \u2014 unchanged.\n- The argreg integer/FP split \u2014 unchanged.\n- The `Member->idx` field \u2014 unchanged. The bitfield siblings (`is_bitfield`/`bit_offset`/`bit_width`) \u2014 unchanged.\n- The `is_flexible` flag \u2014 used in `count_array_init_elements`'s dummy initializer (set to `true` so the dummy can absorb nested initializers without size-checking), and in `array_initializer1`'s flexible-resize branch. The dead-code duplicate is in `array_initializer1`'s second `if (init->is_flexible)` block.\n- `copy_struct_type` \u2014 unchanged.\n- `MIN`/`MAX` macros \u2014 `MAX` picks up a new use in `count_array_init_elements` for the running-max-of-cursor calculation. `MIN` picks up a new use in `string_initializer` (already present from Chapter 12 \u00a712.5; no new use this chapter).\n- `is_numeric` predicate \u2014 unchanged.\n- `unreachable()` macro \u2014 gains new sites in `string_initializer`'s switch (the impossible-element-size default), and in `getStringKind` (the impossible-prefix default). Two new callers in this chapter.\n- Per-token line numbers \u2014 unchanged.\n- GDB-debuggable output \u2014 unchanged.\n- Tests are in C. `test/unicode.c` is the new test file (added in `c31886a`). Designated-initializer tests go into existing `test/initializer.c`. Cross-prefix concatenation tests go into existing `test/string.c`. Driver test for BOM marker added to `test/driver.sh`.\n- The `Obj->tok` field \u2014 still no readers.\n- The `Type->name_pos` field \u2014 no new uses.\n- The `>>` codegen quirk \u2014 unchanged.\n- The `add_type` rule for `ND_STMT_EXPR` \u2014 errata candidate, unchanged.\n- The hex-escape silent truncation \u2014 errata candidate, unchanged. (UTF-16 character literal's silent truncation of code points above U+FFFF is a related but distinct case, also errata-worthy.)\n- The redeclaration-in-same-scope check missing for variables/tags/typedef-names/labels/struct-members \u2014 five errata candidates, unchanged.\n- `f()` and `f(void)` are accepted as identical \u2014 errata candidate, unchanged.\n- Empty brace initializer \u2014 chibicc-specific extension, still in use.\n- `.bss` is the third assembly section \u2014 unchanged.\n- `.align` \u2014 unchanged.\n- The `mov $0, %rax` for variadic FP-count \u2014 errata candidate, unchanged.\n- The `fp_offset = fp * 8 + 48` non-conforming stride \u2014 errata candidate, unchanged.\n- `long double` is `double` \u2014 errata candidate, unchanged.\n- The default-argument-promotion gap \u2014 errata candidate, unchanged.\n- Float literals inlined as integer-immediate-bit-cast \u2014 unchanged.\n- The cast/compound-literal disambiguator \u2014 unchanged.\n- The cast table is 10\u00d710 \u2014 unchanged. UTF-16 (`unsigned short`) and UTF-32 (`unsigned int`) are existing cells; the four prefix forms don't add cast cells.\n- Driver brittleness in `find_libpath`/`find_gcc_libpath` \u2014 unchanged.\n- The link command's hardcoded distro list \u2014 unchanged.\n- `Node->funcname` is gone \u2014 still gone.\n- `mov %rax, %r10; call *%r10` is uniform across all calls \u2014 unchanged.\n- The `StringArray` type \u2014 unchanged.\n- `atexit(cleanup)` \u2014 unchanged.\n- The `run_subprocess` helper \u2014 unchanged.\n- Errata candidates added in Chapter 17 \u2014 `#error` doesn't print message text, `L''` \u2261 `''` (closed by `a57c661`), `__va_arg_mem` divides by zero (closed by Chapter 18), `opt_S | opt_E` typo, default include paths Linux/glibc-specific. Two closed; three remain.\n- Errata candidates added in Chapter 18 \u2014 None high-priority. The bitfield zero-width test exposed the missing struct-member-name redeclaration check.\n- Errata candidates added in Chapter 19 \u2014 UTF-16 character-literal silent truncation of code points above U+FFFF; the dead-code duplicate `is_flexible` block in `array_initializer1`; range designators `[3 ... 7]` syntactically accepted but not honored in elaboration. Three new low-priority candidates.\n- `self.py` is gone \u2014 still gone.\n- Stage-2 build is end-to-end chibicc, `-Wall`-clean \u2014 still is.\n- Chibicc compiles itself \u2014 still does.\n- The `has_flonum` family \u2014 unchanged.\n- Bitfield support is feature-complete \u2014 unchanged.\n- Anonymous struct/union members flatten via recursive `get_struct_member` \u2014 now reused by `struct_designator`'s anonymous-match arm.\n\nForward references for Chapter 20 (commits 245\u2013266 \u2014 the GCC-extension and small-completions arc).\n\n- `_Generic`, `_Static_assert`, `_Alignof`, `_Alignas` are deferred to Chapter 20.\n- The GCC ternary-without-middle (`a ?: b`) is deferred to Chapter 20.\n- The GCC `__attribute__` parsing surface is deferred to Chapter 20.\n- The `<stdarg.h>`, `<stdbool.h>`, `<stddef.h>`, `<stdalign.h>`, `<stdnoreturn.h>`, and `<float.h>` headers are unchanged in Chapter 19. Some may grow in Chapter 20.\n\nTwenty-four commits. The chapter brings chibicc from \"ASCII-only source, ASCII-only identifiers, no designated initializers\" to \"UTF-8 source with C11 Annex D identifiers, four string-literal prefix forms, four character-literal prefix forms, and full designated-initializer support including the GNU `=`-omission and anonymous-struct cases.\" The next chapter takes on the GCC-extension corner of the C language \u2014 `_Generic`, `_Static_assert`, the alignment specifiers, and the small `__attribute__` parsing surface that real C codebases assume.\n"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "wc -w /Users/cell/claude/chibicc-book/chapters/19-unicode-and-designated-initializers.md",
  "description": "Word count for Ch 19"
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 24 Ch 19 commits in detail",
      "activeForm": "Reading 24 Ch 19 commits",
      "status": "completed"
    },
    {
      "content": "Read prior chapter 18 for voice/tone matching",
      "activeForm": "Reading chapter 18",
      "status": "completed"
    },
    {
      "content": "Read session 019 README",
      "activeForm": "Reading session 019 README",
      "status": "completed"
    },
    {
      "content": "Draft chapters/19-unicode-and-designated-initializers.md",
      "activeForm": "Drafting Ch 19",
      "status": "completed"
    },
    {
      "content": "Write session 020 README and HANDOFF for session 021",
      "activeForm": "Writing session 020 docs",
      "status": "in_progress"
    }
  ]
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "grep -A 50 \"Chapter 20\\|chapter 20\" /Users/cell/claude/chibicc-book/research/commits/chapter-mapping.md | head -80",
  "description": "Find Ch 20 mapping"
}


---

## 🤖 Claude


🔧 Using tool: Bash
Input: {
  "command": "awk 'NR>=244 && NR<=270' /Users/cell/claude/chibicc-book/research/commits/main-commits.txt",
  "description": "Show commits 245-266"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/020-chapter-19-draft/README.md",
  "content": "# Session 020 \u2014 Chapter 19 draft\n\n**Date:** 2026-05-10 (continuation of the autonomous-drafting run that has produced sessions 002\u2013019).\n**Phase:** 2 (bulk drafting).\n\n## Arrival state\n\nSession 019 delivered Ch 18 (The full ABI, twenty-three commits, ~14,357 words). User direction is still autonomous \u2014 no chapter-by-chapter review. Ch 19 covers commits 221\u2013244: twenty-four commits, the Unicode arc plus designated initializers plus the trailing date/time/counter macros that Ch 17 deferred.\n\n## What was done\n\n### Drafting decisions\n\n- **Length:** ~12,128 words. At the lower end of the 12,000\u201315,000-word handoff forecast. The Unicode arc compresses well \u2014 most commits are small enough that a per-commit subsection of 100\u2013200 words covers them adequately, and the four string-literal-prefix commits in \u00a719.5 share enough structure that they can be walked in parallel without redundancy. The designated-initializer arc in \u00a719.7 is the chapter's longest single stretch (~2,800 words across the six commits).\n- **Section structure:** 7 sections from 24 commits, exactly as the handoff proposed. \u00a719.1 (2 commits, two integrated subsections in prose). \u00a719.2 (1 commit, no subsections). \u00a719.3 (1 commit, no subsections). \u00a719.4 (3 commits, three named subsections). \u00a719.5 (7 commits, seven named subsections \u2014 kept each commit explicit even though the four prefix-form commits are structurally similar). \u00a719.6 (4 commits, four named subsections). \u00a719.7 (6 commits, six named subsections, no skipping).\n- **No concept interlude.** The handoff defaulted to \"possibly one\" with the conditional escape on \u00a719.7 if the prose felt overstuffed. Reading the \u00a719.7 prose, the cursor-jump rule and the bail-and-restart pattern in `array_initializer2` / `struct_initializer2` fit inline without straining. The short summary in the \u00a719.7 opening (explaining what the cursor-jump rule does) is enough scaffolding.\n- **\u00a719.4 closes the Ch 17 `L''` \u2261 `''` errata** \u2014 explicitly named in the \u00a719.4 `a57c661` subsection and in the chapter recap.\n- **\u00a719.7 surfaces two new errata candidates** \u2014 the dead-code duplicate `is_flexible` block in `array_initializer1` (introduced by `835cd24` and never deleted), and the half-implemented range designator `[3 ... 7]` syntax (recognized in `count_array_init_elements` for length computation, not honored in elaboration). Both are named in the \u00a719.7 prose and in the chapter recap.\n- **\u00a719.5 walks each of the four prefixes (L, u, U, u8) and their type assignments** \u2014 and the two initializer commits (232\u2013233) get explicit treatment as separate subsections. The `__STDC_UTF_*` macros commit (234) gets its own short subsection.\n- **\u00a719.6 names UTF-8 BOM stripping as a tokenizer-pre-pass concern** \u2014 the chapter recap also names the pre-tokenize pass count moving from 1 (just `remove_backslash_newline`) to 4 (BOM, newline, backslash-newline, UCN).\n- **One-table recap** at the chapter close, twenty-four rows. Resisted multi-table-by-section.\n\n### Interpretive calls\n\n1. **The cross-prefix re-tokenization in `2382777` is named as a token-level rewrite within the preprocessor**, not a parse-time canonicalization. The canonicalization-at-parse-time count stays at nine. The mechanism is similar in spirit (rewriting tokens before the parser sees them) but lives in `preprocess.c` rather than `parse.c`, and the prose treats it as a different category.\n2. **The pre-factor count stays at nine.** The closest candidate was `read_char_literal`'s `Type *ty` parameter in `454618c`, but that change and its first use are in the same commit. `read_string_literal`'s `quote` parameter in `57b21fe` is similarly same-commit. Neither qualifies as pre-factor-before-feature.\n3. **The psABI conformance count stays at sixteen.** Ch 19's commits don't touch the ABI surface.\n4. **The Initializer tree is named as gaining one new field (`Member *mem` for the union case).** No other shape changes \u2014 the children array is still position-indexed, designators just jump the cursor without restructuring the tree.\n5. **The `init->expr = NULL` line in `designation`'s struct arm (commit `67f5834`) is given its own short walk.** It clears prior scalar bindings on re-designation. The test case `T y[]={x, [0].b=3}` exercises the line \u2014 without it, both the struct-copy and the b-field designation would emit stores, and the order would determine the final value.\n6. **The anonymous-struct designator's `*rest = start` trick (commit `95eb5b0`) gets one paragraph.** The trick lets the same `.field` token be re-parsed at each anonymous layer; the prose walks the test case `{1,2,3,.b=4,5}` against a struct-of-anonymous-struct-of-anonymous-struct to show how `b=4` lands and `5` continues to `c`.\n7. **Ch 17 errata closures: two now closed, three remaining.** `L''` \u2261 `''` closed by `a57c661`. `__va_arg_mem` divides by zero closed by Ch 18. The remaining three: `#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific.\n8. **The UTF-16 character literal silent truncation is named as errata.** Commit `454618c`'s `cur->val &= 0xffff` truncates code points above U+FFFF to their low 16 bits. The C standard says this should be a constraint violation; chibicc is permissive. Test case in \u00a719.4 exercises it (`U+1F363` becomes `0xF363`).\n9. **The \u00a719.5 prose handles `read_escaped_char` consistently across the prefixes.** The note that `u\"\\xff\"` produces a single 16-bit unit (`0x00FF`) rather than the UTF-16 encoding of U+00FF is named in the \u00a719.5 UTF-16 subsection. gcc and clang behave the same way, so it's not a chibicc-specific quirk.\n10. **The \u00a719.6 cross-prefix concatenation walks the `(\"\\343\\201\\202\" L\"\")` test case.** The test case shows that an octal-escape ordinary string and a UTF-8 source string are byte-identical at parse but *differ* under cross-prefix promotion (escape sequences are kept as bytes; UTF-8 source is decoded to code points). This is the most subtle distinction in the chapter; the prose names it explicitly.\n\n### Voice / structure inherited from Ch 1\u201318\n\n- \"we\" for reader-journey, \"Rui\" for design intent.\n- Past tense for what the commit did, present for current behavior.\n- Each section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, all hashes listed at the top.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- One-table recap at the chapter close.\n- No concept interludes.\n\n### Three careful avoidances\n\n- **Did not invent a \"history of UTF-8\" interlude.** UTF-8 has a famous design history (Pike/Thompson, 1992); walking it would have been a detour. The chapter cites the encoding rule and walks chibicc's encode/decode helpers; it doesn't try to reconstruct the design history.\n- **Did not over-explain the surrogate-pair encoding.** UTF-16's surrogate pairs are a curious historical artifact; the prose walks the encoding step-by-step for one test case (`\ud83c\udf63`, U+1F363 \u2192 0xD83C, 0xDF63) and leaves the abstract rule to the encoded comment. Walking the abstract rule plus several test cases would have inflated \u00a719.5 without adding clarity.\n- **Did not invent a \"C99 designated initializer history\" detour.** Designated initializers have a long history in C compilers (gcc had them as an extension for a decade before C99); walking that history would be a detour. The chapter sticks to chibicc's specific implementation choices.\n\n### Date-vs-position note\n\nThe twenty-four commits scatter wildly across calendar time: April 2020 (no commits in Ch 19), May 2020 (`a57c661` wide-char, `0e5d250` UTF-8 in identifier, `e27417f` `__DATE__`), June 2020 (`0e77f3d` `__COUNTER__`, `74bcec5` newline canonicalization), July 2020 (`adb8b98` `$`, `cae061a` wide string, `36230e0` UTF-16 init, `6adba75` UTF-32 init, `e4491b8` `__STDC_UTF_*`, `691c4fa` `=`-omission, `95eb5b0` anonymous-struct designator), August 2020 (`c31886a` UCN, `31dc1df` union designator), September 2020 (`454618c` UTF-16 char, `2dac3af` UTF-32 char, `57b21fe` UTF-8 string, `9cabe1f` UTF-16 string, `c467ee6` UTF-32 string, `c618c3b` array designator, `835cd24` incomplete-array sizing), October 2020 (`67f5834` struct designator, `2382777` cross-prefix concat, `2b2fa25` BOM). The chapter follows `main` order without remark \u2014 `a57c661` (May 6, 2020) lands at position 225 even though several later-dated commits appear before it on `main`. This is consistent with prior chapters' policy.\n\n## Open questions surfaced for user\n\nNone \u2014 autonomous mode.\n\n## Notes worth carrying forward\n\n- **The Unicode baseline:** chibicc's source-file convention is now formally UTF-8. `decode_utf8` and `encode_utf8` are the only two byte-level Unicode helpers needed. `unicode.c` is the new file; `tokenize.c` calls into it; `chibicc.h` declares its public surface.\n- **The pre-tokenize pass count** is now 4: BOM, newline, backslash-newline, UCN. All four are destructive in-place rewrites of the source buffer. Order matters: BOM first, newline second, backslash-newline third, UCN fourth.\n- **The four character-literal prefixes** are functional with type assignments matching gcc on Linux: `'X'` is `int` then narrowed to `char`, `L'X'` is `int`, `u'X'` is `unsigned short` truncated to 16 bits, `U'X'` is `unsigned int`.\n- **The four string-literal prefixes** are functional with these element types: ordinary and `u8` are `char`, `u` is `unsigned short`, `U` is `unsigned int`, `L` is `int`. The `Initializer` tree's `string_initializer` dispatches on `init->ty->base->size`, with cases for 1, 2, and 4.\n- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined to 1.\n- **The cross-prefix concatenation handler** does two passes: detect-and-promote, then concatenate. The promotion rewrites ordinary literals to match a wide neighbor's encoding using `tokenize_string_literal` (the new public entry point in `tokenize.c`). The `StringKind` enum and `getStringKind` helper classify literals by prefix without re-parsing.\n- **UTF-8 in identifiers** uses the C11 Annex D ranges. The range tables are in `unicode.c` as static `uint32_t[]` arrays terminated by `-1`. `is_ident1` and `is_ident2` move from `tokenize.c` (where they were `char`-based) to `unicode.c` (where they are `uint32_t`-based). The new `read_ident` in `tokenize.c` is the canonical identifier recognizer; the do-while-on-bytes loop is gone.\n- **The GNU `$` extension** adds two range-table entries to `is_ident1` and `is_ident2`. Tests live in `test/unicode.c`.\n- **The UTF-8 BOM skip** is a three-byte `EF BB BF` test at the start of `tokenize_file`, before all other pre-tokenize passes.\n- **The designator parsing surface** is three new helpers (`array_designator`, `struct_designator`, `designation`) plus the cursor-jump-aware variants of three existing helpers (`array_initializer2`, `struct_initializer2`, `count_array_init_elements`).\n- **The `Initializer` struct gains `Member *mem`** for the union case. Set by `union_initializer` or `designation`'s union arm. Read by `create_lvar_init` and `write_gvar_data`'s union branches. The fallback `init->mem ? init->mem : ty->members` in `create_lvar_init` handles implicit-first-member cases (`union x = {}`).\n- **The cursor-bail pattern** in `array_initializer2` and `struct_initializer2`: when the loop sees a designator (`[` or `.`), it restores `*rest` to the bail position and returns. The bail propagates up to whichever outer initializer handler (`array_initializer1`, `struct_initializer1`, or `designation`'s nested call) parses the designator.\n- **The `init->expr = NULL` line in `designation`'s struct arm** clears prior scalar bindings on re-designation. Necessary for cases where a scalar struct value is followed by a designation that overrides it.\n- **The anonymous-struct designator integration** reuses `get_struct_member` from Chapter 18 \u00a718.6. The `*rest = start` trick lets the same `.field` token be re-parsed at each anonymous layer.\n- **Two new errata candidates from \u00a719.7:**\n  - The dead-code duplicate `if (init->is_flexible)` block in `array_initializer1` (introduced by `835cd24`).\n  - Range designators `[3 ... 7]` syntactically accepted in `count_array_init_elements` but not honored in elaboration.\n- **One new errata candidate from \u00a719.4:** UTF-16 character-literal silent truncation of code points above U+FFFF.\n- **One Ch 17 errata closed:** `L''` \u2261 `''` (closed by `a57c661`).\n- **Pre-factor-before-feature count** stays at nine.\n- **Canonicalization-at-parse-time count** stays at nine.\n- **psABI conformance count** stays at sixteen.\n- **`unreachable()` callers** gain two new sites: `string_initializer`'s switch default, `getStringKind`'s switch default.\n- **`MAX` macro** picks up a new use in `count_array_init_elements`.\n- **The `is_unsigned` flag on `Type`** has new readers via the UTF-16 (`ty_ushort`) and UTF-32 (`ty_uint`) string-literal element types.\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean \u2014 unchanged.\n- **Chibicc compiles itself** \u2014 unchanged.\n\n## Exit state\n\n- `chapters/19-unicode-and-designated-initializers.md` drafted, ~12,128 words.\n- Session 020 dir populated with this README and HANDOFF.md.\n- HANDOFF.md primes session 021 (Chapter 20 \u2014 GCC extensions worth supporting, commits 245\u2013266, ~22 commits).\n- CLAUDE.md status note will be updated to \"Ch 19 drafted\".\n"
}


---

## 🤖 Claude


🔧 Using tool: Write
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/docs/sessions/020-chapter-19-draft/HANDOFF.md",
  "content": "# Handoff: Ch 19 done \u2192 proceed to Ch 20\n\n**For:** the next claude session.\n**From:** session 020.\n**Status:** Ch 19 drafted (~12,128 words, twenty-four commits, the full Unicode arc plus designated initializers plus the trailing date/time/counter macros). Continue autonomously to Ch 20 (GCC extensions worth supporting, commits 245\u2013266 \u2014 twenty-two commits covering improved multibyte error messages, `#line` and the GNU line marker directive, `__TIMESTAMP__` and `__BASE_FILE__`, `__VA_OPT__` and the GNU `,##__VA_ARGS__` swallow, `#pragma`, GCC-style variadic macros, `typeof`, `__builtin_types_compatible_p`, `_Generic`, `sizeof` of a function type, the GNU `?:` with omitted middle operand, basic `asm`, `inline` as static, `__attribute__((format))`, `-idirafter`, `offsetof`, tentative definitions, and `-fcommon`/`-fno-common`). Don't pause for review.\n\n## Read these first, in order\n\n1. **[`docs/sessions/020-chapter-19-draft/README.md`](README.md)** \u2014 what session 020 did, including the seven-section structure (per-commit subsections in \u00a719.4, \u00a719.5, \u00a719.6, \u00a719.7; integrated subsections in \u00a719.1; no subsections in \u00a719.2 and \u00a719.3), the closure of the Ch 17 `L''` \u2261 `''` errata in \u00a719.4, three new errata candidates surfaced (UTF-16 char silent truncation, dead-code duplicate `is_flexible` block, half-implemented range designator), the unchanged counts (canonicalization-at-parse-time at nine, pre-factor at nine, psABI conformance at sixteen).\n2. **[`docs/sessions/019-chapter-18-draft/HANDOFF.md`](../019-chapter-18-draft/HANDOFF.md)** \u2014 the previous handoff. Standing notes still apply with Ch 19 updates folded in (see \u00a720 README for the running list).\n3. **[`chapters/01-a-calculator.md`](../../../chapters/01-a-calculator.md)** through **[`19-unicode-and-designated-initializers.md`](../../../chapters/19-unicode-and-designated-initializers.md)** \u2014 the nineteen chapters drafted. Match the register.\n4. **[`research/commits/chapter-mapping.md`](../../../research/commits/chapter-mapping.md)** \u2014 confirms Ch 20 = commits 245\u2013266 (22 commits, scoped to \"GCC extensions worth supporting\").\n5. **[`research/notes/quotes-rui.md`](../../../research/notes/quotes-rui.md)** \u2014 quotable Rui quotes. Ch 20's commits are mostly feature additions; less commit-message material than the early chapters but worth scanning. The `_Generic` and `typeof` and `__VA_OPT__` commits may have design notes.\n6. **[`research/notes/japanese-book-toc.md`](../../../research/notes/japanese-book-toc.md)** \u2014 JP TOC; `_Generic` and `typeof` could be candidates for a concept interlude.\n\n## Chapter 20 scope\n\n**Title (working):** *GCC extensions worth supporting*.\n**Commits:** 245\u2013266 in chronological order on `main`. **Twenty-two commits** \u2014 back below the size of Chapter 19 (24 commits) and Chapter 18 (23 commits).\n**Concept interlude:** Possibly one. `_Generic` is type-driven dispatch at compile time and the implementation walks each association arm; `typeof` is a parser-side feature that has interesting interactions with the type-name vs expression decision. A short interlude on *how chibicc routes type-vs-expression contextual decisions* (the `is_typename` predicate, the cast-vs-compound-literal disambiguator from Ch 13, and now `typeof`/`_Generic`) could land in \u00a720.5 or \u00a720.6. Default conditional \u2014 judge while reading the commits.\n\n| # | Hash | Subject |\n|---|---|---|\n| 245 | `37998be` | Improve error message for multibyte characters |\n| 246 | `c61c0d0` | Add #line |\n| 247 | `aaf20fb` | [GNU] Add line marker directive |\n| 248 | `922604a` | [GNU] Add __TIMESTAMP__ macro |\n| 249 | `3a10c8a` | [GNU] Add __BASE_FILE__ macro |\n| 250 | `3381448` | Add __VA_OPT__ |\n| 251 | `083c275` | [GNU] Handle ,##__VA_ARG__ |\n| 252 | `74ec9f6` | Ignore #pragma |\n| 253 | `007e526` | [GNU] Support GCC-style variadic macro |\n| 254 | `7d80a51` | Add typeof |\n| 255 | `1433b40` | [GNU] Add __builtin_types_compatible_p |\n| 256 | `1faab48` | Add _Generic |\n| 257 | `aee7891` | [GNU] Allow sizeof(<function type>) |\n| 258 | `e28a612` | [GNU] Add ?: operator with omitted operand |\n| 259 | `a253516` | Add basic \"asm\" statement |\n| 260 | `31087f8` | Handle inline functions as static functions |\n| 261 | `e5f4ca9` | Do not emit static inline functions if referenced by no one |\n| 262 | `6a2dc5a` | Use __attribute__((format(print, ...))) to find programming errors |\n| 263 | `11fc259` | Add -idirafter option |\n| 264 | `1b99bad` | Add offsetof |\n| 265 | `85e46b1` | Add tentative definition |\n| 266 | `6d344ed` | Add -fcommon and -fno-common flags |\n\nTwenty-two commits. The natural section grouping:\n\n- **\u00a720.1 \u2014 Multibyte error message + `#line` and line markers** (commits 245\u2013247). Three commits. The error-message tweak for multibyte characters is the smallest possible polish for \u00a719. `#line` is a directive that overrides the line/file the preprocessor reports in subsequent tokens; the GNU line-marker directive (`# 123 \"file\"`) is gcc's preprocessor-output format that cc1 also has to read for the `-E` round-trip.\n- **\u00a720.2 \u2014 Macro extensions: `__TIMESTAMP__`, `__BASE_FILE__`, `__VA_OPT__`, `,##__VA_ARGS__`, `#pragma`, GCC-style variadic** (commits 248\u2013253). Six commits. `__TIMESTAMP__` is \"MM DD HH:MM:SS YYYY\" of the source file's mtime. `__BASE_FILE__` is the top-level source filename. `__VA_OPT__` is C2X's conditional-expansion feature. The `,##__VA_ARGS__` is GCC's swallow-the-comma trick for empty variadics. `#pragma` is silently ignored. GCC-style variadic macros are `#define foo(args...)` (vs C's `#define foo(...)`).\n- **\u00a720.3 \u2014 Type-side extensions: `typeof`, `__builtin_types_compatible_p`, `_Generic`** (commits 254\u2013256). Three commits. These are the chapter's most parser-invasive changes. `typeof(expr)` and `typeof(type)` produce a type from an expression or a typename. `__builtin_types_compatible_p` is a compile-time predicate over two type-names. `_Generic` is the C11 type-driven dispatch.\n- **\u00a720.4 \u2014 Sizeof-of-function and the GNU ternary middle** (commits 257\u2013258). Two commits. `sizeof(<function-type>)` returns 1 (an extension; standard C says it's a constraint violation). `a ?: b` is a GNU extension where the middle operand is omitted and `a` serves as both the condition and (when truthy) the value.\n- **\u00a720.5 \u2014 `asm`, `inline` (\u00d7 2), `__attribute__((format))`** (commits 259\u2013262). Four commits. Basic `asm` statement support. `inline` is treated as `static` (chibicc doesn't do real inlining). The \"do not emit static inline functions if referenced by no one\" commit is a small linker-friendliness optimization. The `__attribute__((format(print, ...)))` annotation is added to chibicc's own `error*` and `warn` functions to catch printf-format mismatches at compile time.\n- **\u00a720.6 \u2014 `-idirafter`, `offsetof`, tentative definitions, `-fcommon`** (commits 263\u2013266). Four commits. `-idirafter` is the include-path family's \"after the standard paths\" entry. `offsetof` is the standard library macro that chibicc can now define using `__builtin_offsetof` (added in this commit). Tentative definitions are the C feature that `int x;` at file scope is a tentative definition that can be overridden by a later real definition. `-fcommon` and `-fno-common` toggle the tentative-definition placement (`.bss` vs `.comm`).\n\nThat's six sections from twenty-two commits. **Target chapter length: ~11,000\u201314,000 words.** Likely closer to 12K \u2014 the GCC-extension commits are mostly small (`#pragma` is one line; `sizeof(<function>)` is two lines), and the larger ones (`_Generic`, `typeof`, `asm`, line markers, tentative definitions) compress well.\n\n## Steps\n\n1. `cd research/sources/chibicc && for h in 37998be c61c0d0 aaf20fb 922604a 3a10c8a 3381448 083c275 74ec9f6 007e526 7d80a51 1433b40 1faab48 aee7891 e28a612 a253516 31087f8 e5f4ca9 6a2dc5a 11fc259 1b99bad 85e46b1 6d344ed; do echo \"===== $h =====\"; git show --stat $h | head -10; done` to scan all 22 diffs.\n2. Read each commit. Pay particular attention to:\n   - **\u00a720.1's `#line` and line markers** \u2014 both have to coexist with chibicc's per-token line-number tracking from Ch 8 \u00a78.3. Read carefully. The GNU line-marker form is `# 123 \"file\"` or `# 123 \"file\" 1` etc.; the trailing flag is a pushed/popped/system-header marker.\n   - **\u00a720.2's `__VA_OPT__`** \u2014 C2X feature. Walk the implementation; this is the most interesting macro-side change in the chapter.\n   - **\u00a720.2's `,##__VA_ARGS__`** \u2014 the swallow-the-comma trick. Walk how the preprocessor recognizes the pattern. Probably a special case in the substitution code.\n   - **\u00a720.2's GCC variadic** \u2014 `args...` instead of `...`. Probably a one-line addition to `read_macro_params`.\n   - **\u00a720.3's `typeof`** \u2014 parser-side change. Read the new type-parser arm. Probably reuses `is_typename` but extends it for `typeof` token.\n   - **\u00a720.3's `_Generic`** \u2014 the most invasive single change. Parse the association list, compare types, return the matching expression. Walk the implementation step by step. The \"default\" arm and the \"no match found\" error case are both interesting.\n   - **\u00a720.5's `asm`** \u2014 basic only (no clobbers, no inputs/outputs). Read the implementation. Probably a one-statement parser arm that emits raw `.s` output.\n   - **\u00a720.5's inline-as-static** \u2014 chibicc doesn't actually inline; `inline` is just a synonym for `static`. The \"don't emit if no references\" commit follows up with a deferred-emit pass. Walk both.\n   - **\u00a720.6's tentative definitions** \u2014 Ch 13's linkage section had this on the to-do list. Now it's done. Walk how `new_gvar` and `parse` cooperate. The `-fcommon` flag toggles placement.\n3. Read the destination state at `6d344ed` for `parse.c`, `tokenize.c`, `preprocess.c`, `codegen.c`, `chibicc.h`, `main.c`. The `_Generic` and `typeof` parser changes are likely the most invasive; tentative definitions touch `parse.c` and `codegen.c`.\n4. Draft `chapters/20-gcc-extensions-worth-supporting.md`. Likely 11,000\u201314,000 words. Six sections.\n5. Write `docs/sessions/021-chapter-20-draft/README.md`.\n6. Write `HANDOFF.md` for session 022 (Chapter 21 \u2014 Thread-local, alloca, VLAs, commits 267\u2013283).\n\n## Voice / structure rules\n\nSame as Ch 1\u201319:\n- Section opens with `git checkout <full-hash>` and the commit's subject as a blockquote. For multi-commit sections, list the checkouts at the section opener.\n- \"we\" for reader, \"Rui\" for design intent.\n- Past tense for what the commit did, present tense for current behavior.\n- No emoji, no callouts, no admonitions.\n- Per-section \"Where we are\" closers.\n- Closing recap with feature table \u2014 twenty-two rows, single table is fine.\n- Diff format: lean toward inline diff fragments and quoted file snippets. The \u00a720.3 `_Generic` and `typeof` will want larger code blocks; the \u00a720.6 tentative-definition codegen too.\n\n## Pitfalls to avoid\n\n(Carried forward and updated.)\n\n- Don't switch voice mid-chapter.\n- Don't fix Rui's code in the prose. The errata appendix still doesn't exist; just collect.\n- Don't invent features chibicc doesn't have. Forward-references must point at actual upcoming commits.\n- Don't ventriloquize Rui \u2014 quote `quotes-rui.md` only when there's a genuinely apt passage.\n- **\u00a720.1's `#line` is a preprocessor directive**, not a tokenizer feature \u2014 it lands in `preprocess.c`. The GNU line-marker is parsed when the preprocessor *reads* a file produced by `-E` (cc1 must round-trip its own preprocessor output).\n- **\u00a720.2's `__TIMESTAMP__` is the file's mtime**, not the compilation time. Different from `__DATE__`/`__TIME__` (which are compilation-time). Walk how the implementation gets the mtime \u2014 probably `stat` or `fstat`.\n- **\u00a720.2's `__VA_OPT__` is non-trivial to implement.** It's a token-list operation: in `__VA_OPT__(X)`, the tokens between the parens are emitted iff `__VA_ARGS__` is non-empty. Walk carefully.\n- **\u00a720.3's `typeof`** must be careful around side effects \u2014 `typeof` doesn't evaluate its argument. The chibicc implementation probably suspends evaluation while parsing.\n- **\u00a720.3's `_Generic`** is type-equality-based dispatch. The C standard's compatibility rule for \"same type\" is subtle (e.g., `int` and `int` are compatible; `int` and `int *` are not; `int *` and `void *` are not under `_Generic`'s equality). Walk what chibicc does.\n- **\u00a720.4's `sizeof(<function-type>)`** returning 1 is a GNU extension; standard C makes this a constraint violation. The motivation is that gcc uses it for various sneaky tricks (`offsetof`-style member-pointer arithmetic, etc.). Note this in the prose.\n- **\u00a720.5's `asm`** is very minimal in chibicc \u2014 just emits the string as raw assembly. No clobbers, no inputs, no outputs, no goto. Real codebases that use `asm` will hit this limit.\n- **\u00a720.5's inline-as-static** \u2014 chibicc treats `inline` as `static`. This is *almost* what the C standard says, but not quite (the standard's `inline` rules are more complex around external linkage). Note the simplification.\n- **\u00a720.6's tentative definition** is a real C feature, not a GNU extension. Two file-scope `int x;` declarations were a parse error pre-this-commit; post-commit, they collapse into one definition. The `-fcommon` flag (default off in modern gcc, default on historically) controls whether tentative definitions go to `.bss` or `.comm`.\n- **\u00a720.6's `-fcommon` is default-off** \u2014 gcc 10+ defaults to `-fno-common` (which puts tentative defs in `.bss`). chibicc's default is unclear; check the diff.\n- **The `__attribute__((format))` commit** is just adding annotations to chibicc's own source; don't read it as adding parser support for `__attribute__` (that's a different, smaller change probably also in the chapter \u2014 verify while drafting).\n\n## Standing notes worth tracking across sessions\n\n- **The hideset on Token** \u2014 unchanged through Ch 19. Ch 20's `__VA_OPT__` and `,##__VA_ARGS__` will likely interact with the existing hideset/expansion machinery. Verify while drafting.\n- **The Token->origin chain** \u2014 likely picks up new uses for the line-marker round-trip.\n- **The eval-quartet duplication** \u2014 unchanged through Ch 19.\n- **The cc1-vs-driver split** \u2014 unchanged.\n- **The `Initializer` tree** \u2014 Ch 19 added `Member *mem` for unions. No expected changes in Ch 20.\n- **The local-vs-global split** \u2014 stable. Tentative definitions affect the global path.\n- **The `Relocation` mechanism** \u2014 likely unchanged in Ch 20.\n- **The anonymous-global pattern** \u2014 likely unchanged in Ch 20.\n- **The `is_static` default in `new_gvar`** \u2014 likely changes for `inline`-as-static (Ch 20 \u00a720.5).\n- **The `is_definition` flag on `Obj`** \u2014 likely changes for tentative definitions (Ch 20 \u00a720.6).\n- **The `is_unsigned` flag on `Type`** \u2014 unchanged.\n- **The `__va_area__` magic name** \u2014 unchanged.\n- **The register-save-area layout** \u2014 unchanged.\n- **The argreg integer/FP split** \u2014 unchanged.\n- **The `Member->idx` field and bitfield siblings** \u2014 unchanged.\n- **The `is_flexible` flag** \u2014 unchanged. The dead-code duplicate from \u00a719.7's `835cd24` is still in the source; if `array_initializer1` is touched in Ch 20 the prose should note whether the duplicate is finally removed.\n- **`copy_struct_type`** \u2014 unchanged.\n- **`MIN`/`MAX` macros** \u2014 unchanged.\n- **`is_numeric` predicate** \u2014 unchanged.\n- **Canonicalization-at-parse-time count is at nine.** Ch 20 might add one in \u00a720.4 (the `?:`-with-omitted-middle commit may rewrite to a temporary-binding form) \u2014 verify while drafting.\n- **Pre-factor-before-feature count is at nine.** Ch 20 unlikely to add new entries.\n- **psABI conformance count is at sixteen.** Ch 20 unlikely to touch it.\n- **The fourth namespace (labels)** is unchanged.\n- **The `is_typename` predicate** likely changes in \u00a720.3 (`typeof` is a typename-introducer).\n- **The VarAttr channel** has four fields. Ch 20 might grow it \u2014 `inline` and `is_tentative` are candidate fields.\n- **The `ND_NULL_EXPR` seed-pattern** \u2014 unchanged.\n- **The `rep stosb` pattern** \u2014 unchanged.\n- **The `unreachable()` macro** \u2014 likely picks up new callers in `_Generic`'s default-not-found case and elsewhere.\n- **Per-token line numbers** \u2014 preserved through preprocessing as of Ch 17. Will be touched by `#line` in \u00a720.1.\n- **GDB-debuggable output** \u2014 unchanged.\n- **Tests are in C.** New test file likely for `_Generic`. Driver tests for `-idirafter`, `-fcommon`. `asm` tests in shell driver.\n- **The `Obj->tok` field** added in Ch 14 \u00a714.11 still has no readers.\n- **The `Type->name_pos` field** \u2014 no new uses since Ch 14.\n- **The `>>` codegen quirk** \u2014 partially repaired.\n- **The `add_type` rule for `ND_STMT_EXPR`** \u2014 errata candidate.\n- **The hex-escape silent truncation** \u2014 errata candidate.\n- **The redeclaration-in-same-scope check** missing for variables, tags, typedef names, labels, struct-members \u2014 five errata candidates.\n- **`f()` and `f(void)` are accepted as identical** \u2014 errata candidate.\n- **Empty brace initializer (`int x[3] = {};`)** \u2014 chibicc-specific extension.\n- **`.bss` is the third assembly section.** `-fcommon` may add `.comm` as a fourth path.\n- **`.align`** \u2014 unchanged.\n- **The `mov $0, %rax`** for variadic FP-count \u2014 errata candidate.\n- **The `fp_offset = fp * 8 + 48` non-conforming stride** \u2014 errata candidate.\n- **`long double` is `double`** \u2014 errata candidate.\n- **The default-argument-promotion gap for chars and shorts** \u2014 errata candidate.\n- **Float literals are inlined as integer-immediate-bit-cast.**\n- **Ch 1 errata list** unchanged.\n- **The cast/compound-literal disambiguator** \u2014 same lookahead pattern.\n- **The cast table is 10\u00d710.** Possibly grows in \u00a720.3 if `_Generic` introduces new cast cells; verify while drafting.\n- **Driver brittleness** \u2014 unchanged.\n- **The link command's hardcoded distro list** \u2014 errata candidate.\n- **`Node->funcname` is gone.**\n- **`mov %rax, %r10; call *%r10` is uniform across all calls.**\n- **The `StringArray` type** \u2014 used by `include_paths`. `-idirafter` will likely add to it.\n- **`atexit(cleanup)` for tempfile disposal** \u2014 unchanged.\n- **The `run_subprocess` helper** \u2014 unchanged.\n- **Errata candidates added in Ch 17:** `#error` doesn't print message text; `L''` \u2261 `''` (closed by Ch 19's `a57c661`); `__va_arg_mem` divides by zero (closed by Ch 18); `opt_S | opt_E` typo; default include paths Linux/glibc-specific. Two closed; three remaining.\n- **Errata candidates added in Ch 18:** None high-priority. The bitfield zero-width test exposed the missing struct-member-name redeclaration check.\n- **Errata candidates added in Ch 19:**\n  - UTF-16 character-literal silent truncation of code points above U+FFFF (in \u00a719.4, commit `454618c`).\n  - Dead-code duplicate `is_flexible` block in `array_initializer1` (in \u00a719.7, commit `835cd24`).\n  - Range designators `[3 ... 7]` syntactically accepted but not honored in elaboration (in \u00a719.7, commit `835cd24`).\n- **`self.py` is gone.**\n- **Stage-2 build** is end-to-end chibicc, `-Wall`-clean.\n- **Chibicc compiles itself** as of commit 197 (Ch 17 \u00a717.6).\n- **The `has_flonum` family** unchanged.\n- **Bitfield support is feature-complete.**\n- **Anonymous struct/union members** flatten via recursive `get_struct_member`.\n- **The pre-tokenize pass count is four** (Ch 19 \u00a719.6): BOM, newline, backslash-newline, UCN. Order matters.\n- **The four char-literal prefixes** are functional. Types: `'X'` \u2192 `int` then `(char)`; `L'X'` \u2192 `int`; `u'X'` \u2192 `unsigned short` masked to 16; `U'X'` \u2192 `unsigned int`.\n- **The four string-literal prefixes** are functional. Element types: ordinary and `u8` \u2192 `char`; `u` \u2192 `unsigned short`; `U` \u2192 `unsigned int`; `L` \u2192 `int`. The cross-prefix concat re-tokenizes ordinary literals to match wide neighbors.\n- **`__STDC_UTF_16__` and `__STDC_UTF_32__`** are defined.\n- **UTF-8 in identifiers** uses C11 Annex D ranges. `is_ident1`/`is_ident2` live in `unicode.c`.\n- **The GNU `$` extension** in identifiers.\n- **`__DATE__`, `__TIME__`, `__COUNTER__`** are predefined.\n- **Designated initializers** work for arrays, structs, unions, anonymous-struct, plus the GNU `=`-omission. The `Initializer` tree's `Member *mem` field handles unions.\n\n## Acceptance criteria for Ch 20\n\n- [ ] `chapters/20-gcc-extensions-worth-supporting.md` exists, end-to-end readable.\n- [ ] All twenty-two commits covered, grouped into ~6 sections.\n- [ ] \u00a720.1 explains `#line` and line markers and how they coexist with the existing per-token line-number tracking from Ch 8 \u00a78.3.\n- [ ] \u00a720.2 walks `__VA_OPT__` and `,##__VA_ARGS__` step by step, with at least one test case for each.\n- [ ] \u00a720.3 walks `typeof` as a parser-side typename-introducer, walks `_Generic` as a type-equality-based dispatcher, and walks `__builtin_types_compatible_p` as a compile-time predicate.\n- [ ] \u00a720.5 names that chibicc treats `inline` as `static` (a simplification of the standard's actual `inline` rules).\n- [ ] \u00a720.6 walks tentative definitions and the `-fcommon`/`-fno-common` toggle.\n- [ ] Voice matches Ch 1\u201319.\n- [ ] No emoji, no callouts, no admonitions.\n- [ ] Forward-references checked against `chapter-mapping.md`.\n- [ ] psABI conformance thread count noted as still at sixteen unless Ch 20 commits add to it (they probably don't).\n- [ ] `docs/sessions/021-chapter-20-draft/README.md` written.\n- [ ] `HANDOFF.md` written for session 022 (Chapter 21 \u2014 Thread-local, alloca, VLAs, commits 267\u2013283).\n\n## Prompt block to paste into a fresh session\n\n```\nContinue the chibicc book project. The user has asked for autonomous\nprogress \u2014 do not stop between chapters for review.\n\nRead in order:\n1. docs/sessions/020-chapter-19-draft/HANDOFF.md  (this handoff)\n2. docs/sessions/020-chapter-19-draft/README.md   (what session 020 did)\n3. chapters/01-a-calculator.md                     (template, voice)\n4. chapters/02-from-program-to-programs.md\n5. chapters/03-statements-and-local-variables.md\n6. chapters/04-pointers.md\n7. chapters/05-functions.md\n8. chapters/06-arrays.md\n9. chapters/07-globals-characters-strings.md\n10. chapters/08-scopes-and-source-locations.md\n11. chapters/09-structs-and-unions.md\n12. chapters/10-filling-out-the-type-system.md\n13. chapters/11-all-the-operators.md\n14. chapters/12-initializers.md\n15. chapters/13-linkage.md\n16. chapters/14-variadics-signedness-qualifiers.md\n17. chapters/15-floating-point.md\n18. chapters/16-the-compiler-driver.md\n19. chapters/17-a-preprocessor-from-scratch.md\n20. chapters/18-the-full-abi.md\n21. chapters/19-unicode-and-designated-initializers.md (most recent chapter)\n22. research/commits/chapter-mapping.md            (confirms Ch 20 scope)\n23. CLAUDE.md and book-plan.md                     (conventions)\n\nThen draft Chapter 20 (GCC extensions worth supporting, commits\n245\u2013266) per the steps in the handoff. Twenty-two commits, six\nsections proposed in the handoff. The type-side extensions arc (\u00a720.3,\nthree commits \u2014 typeof, __builtin_types_compatible_p, _Generic) is the\nchapter's most parser-invasive stretch and is where a possible concept\ninterlude on type-vs-expression context lands. End-of-session: write\nyour session dir under docs/sessions/021-chapter-20-draft/ with a\nREADME and a HANDOFF for session 022 (Chapter 21 \u2014 Thread-local,\nalloca, VLAs, commits 267\u2013283).\n```\n"
}


---

## 🤖 Claude

Now updating CLAUDE.md status note.

---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019). Continuing autonomously per user direction.",
  "new_string": "Phase 2 (bulk drafting): in progress. Ch 2 drafted (session 003), Ch 3 drafted (session 004), Ch 4 drafted (session 005), Ch 5 drafted (session 006), Ch 6 drafted (session 007), Ch 7 drafted (session 008), Ch 8 drafted (session 009), Ch 9 drafted (session 010), Ch 10 drafted (session 011), Ch 11 drafted (session 012), Ch 12 drafted (session 013), Ch 13 drafted (session 014), Ch 14 drafted (session 015), Ch 15 drafted (session 016), Ch 16 drafted (session 017), Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: Read
Input: {
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "offset": 68,
  "limit": 8
}


---

## 🤖 Claude


🔧 Using tool: Edit
Input: {
  "replace_all": false,
  "file_path": "/Users/cell/claude/chibicc-book/CLAUDE.md",
  "old_string": "Ch 17 drafted (session 018), Ch 18 drafted (session 019). Continuing autonomously per user direction.",
  "new_string": "Ch 17 drafted (session 018), Ch 18 drafted (session 019), Ch 19 drafted (session 020). Continuing autonomously per user direction."
}


---

## 🤖 Claude


🔧 Using tool: TodoWrite
Input: {
  "todos": [
    {
      "content": "Read all 24 Ch 19 commits in detail",
      "activeForm": "Reading 24 Ch 19 commits",
      "status": "completed"
    },
    {
      "content": "Read prior chapter 18 for voice/tone matching",
      "activeForm": "Reading chapter 18",
      "status": "completed"
    },
    {
      "content": "Read session 019 README",
      "activeForm": "Reading session 019 README",
      "status": "completed"
    },
    {
      "content": "Draft chapters/19-unicode-and-designated-initializers.md",
      "activeForm": "Drafting Ch 19",
      "status": "completed"
    },
    {
      "content": "Write session 020 README and HANDOFF for session 021",
      "activeForm": "Writing session 020 docs",
      "status": "completed"
    }
  ]
}


---

## 🤖 Claude

Session 020 complete. Chapter 19 (Unicode and designated initializers, ~12,128 words, 24 commits, 7 sections) is drafted at [chapters/19-unicode-and-designated-initializers.md](chapters/19-unicode-and-designated-initializers.md). Session 020 README and the HANDOFF for session 021 are at [docs/sessions/020-chapter-19-draft/](docs/sessions/020-chapter-19-draft/). CLAUDE.md status updated.

Highlights:
- §19.4 closes the Ch 17 `L''` ≡ `''` errata.
- §19.7 surfaces three new errata candidates (UTF-16 silent truncation, dead-code duplicate `is_flexible` block in `array_initializer1`, half-implemented `[3 ... 7]` range designators).
- Counts unchanged: canonicalization-at-parse-time at 9, pre-factor at 9, psABI conformance at 16.
- Pre-tokenize pass count moves from 1 to 4 (BOM, newline, backslash-newline, UCN).

Handoff: docs/sessions/020-chapter-19-draft/HANDOFF.md

---

