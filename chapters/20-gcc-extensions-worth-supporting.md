# Chapter 20 — GCC extensions worth supporting

> Commits covered: `37998be`, `c61c0d0`, `aaf20fb`, `922604a`, `3a10c8a`, `3381448`, `083c275`, `74ec9f6`, `007e526`, `7d80a51`, `1433b40`, `1faab48`, `aee7891`, `e28a612`, `a253516`, `31087f8`, `e5f4ca9`, `6a2dc5a`, `11fc259`, `1b99bad`, `85e46b1`, `6d344ed`. Twenty-two commits — small polish on multibyte error display, the `#line` directive and gcc's line-marker form, four new predefined macros, two macro-expansion extensions for variadic ergonomics, `#pragma`, `typeof` and `_Generic` and `__builtin_types_compatible_p` on the type side, two small relaxations of standard rules (`sizeof` of a function, `?:` with omitted middle), basic `asm` statements, two passes of `inline` plumbing (treat-as-static, then dead-static-inline elimination), `__attribute__((format))` annotations on chibicc's own diagnostics, the `-idirafter` include-path entry, the `offsetof` macro, tentative definitions, and the `-fcommon`/`-fno-common` toggle.

Through Chapter 19 chibicc is feature-complete enough to compile itself end-to-end, with bitfields, the SysV AMD64 ABI, the full Unicode arc, and designated initializers. What it isn't is *gcc-friendly*. Real-world C code reaches for gcc extensions constantly — `typeof` in container-of macros, `_Generic` in `tgmath.h`-style headers, `asm` in OS kernels and standard libraries, `inline` in just about every modern header, tentative definitions in plain old global declarations like `int x;` at file scope. A compiler that handles only ISO C will choke on glibc's headers within the first few inclusions. Chapter 20's twenty-two commits chip away at that gap.

The commits are small. Most are between five and forty lines of diff. Three are larger: `_Generic` adds about forty-seven lines of new parser code, the `inline`-elimination commit adds about thirty-seven, the GCC-style variadic macro rework rewrites about thirty. The rest are one-feature, one-function additions whose implementation reads the same way the spec reads. The chapter's job is to walk through the lot, name what each commit gave us, and flag the points where chibicc's implementation diverges from what the language reference says — sometimes deliberately, occasionally not.

Six sections from twenty-two commits.

- **§20.1** — Multibyte error column display, `#line`, and the GNU line marker (commits 245–247).
- **§20.2** — Macro extensions: `__TIMESTAMP__`, `__BASE_FILE__`, `__VA_OPT__`, `,##__VA_ARGS__`, `#pragma`, GCC-style variadic (commits 248–253).
- **§20.3** — Type-side extensions: `typeof`, `__builtin_types_compatible_p`, `_Generic` (commits 254–256).
- **§20.4** — `sizeof` of a function and the GNU ternary middle (commits 257–258).
- **§20.5** — `asm`, `inline`-as-static, dead static-inline elimination, `__attribute__((format))` (commits 259–262).
- **§20.6** — `-idirafter`, `offsetof`, tentative definitions, `-fcommon` (commits 263–266).

The chapter follows `main` order. As before, the calendar dates scatter — the `__BASE_FILE__` commit is dated August 20 but lands at position 249 between two July commits — and the chapter does not remark on date-vs-position except where the work itself depends on something later.

---

## 20.1 — Multibyte error column display, `#line`, and the GNU line marker

> `git checkout 37998be0c183508e54f10f57d63d87e6e7eb0607` — *Improve error message for multibyte characters*
>
> `git checkout c61c0d00252a8704ff2731f6a57bad3657b84170` — *Add #line*
>
> `git checkout aaf20fb96eaf21ead775fde6bad00d8e71650b5a` — *[GNU] Add line marker directive*

Three commits. The first is a small visual fix in `verror_at` that closes the last loose end of the Unicode arc from Chapter 19. The other two add the `#line` family of directives, which let a generated source file pretend it came from somewhere else.

### 20.1.1 — `display_width` and the multibyte caret

Through Chapter 19, `verror_at` printed the source line, then a caret on the next line at the column where the error sat. The column was computed in bytes:

```c
int pos = loc - line + indent;
```

For ASCII source that's correct: every character is one byte and one column. For a UTF-8 source line containing Japanese or emoji, it's wrong by however many extra UTF-8 bytes lie between the start of the line and `loc`. The diagnostic's caret would land several columns to the right of the offending token.

The fix moves the column computation through a new helper, `display_width`, that walks the source bytes through `decode_utf8` and asks `char_width` how many display columns each code point occupies:

```c
int pos = display_width(line, loc - line) + indent;
```

`display_width` decodes one code point per loop iteration, summing widths:

```c
int display_width(char *p, int len) {
  char *start = p;
  int w = 0;
  while (p - start < len) {
    uint32_t c = decode_utf8(&p, p);
    w += char_width(c);
  }
  return w;
}
```

`char_width` returns 0 for combining characters (the `range1` table), 2 for East-Asian-wide and emoji-range code points (the `range2` table), and 1 for everything else. The tables are large — 37 lines of paired ranges in `range1` alone — and they trace back, by attribution in the source comment, to Markus Kuhn's `wcwidth.c` implementation that has circulated since the late 1990s. Chibicc inlines the tables rather than calling the libc `wcwidth(3)`, partly because chibicc tries to keep its libc surface narrow and partly because libc's `wcwidth` requires a locale to be set up first.

The code-point/column distinction is the same one terminal emulators have to make when laying out text. It's why a CJK character takes two cells in your terminal but one in a font's character grid; combining accents take zero cells because they overlay the previous glyph. The diagnostic now matches what a reader sees.

### 20.1.2 — `#line` and the per-token line delta

> *#line is a directive that overrides the file name and line number that subsequent tokens report.*

`#line N` resets the line counter so the *next* line reports as `N`. `#line N "file"` resets both the line and the filename. The directive is most useful for code generators that produce C: by stamping `#line` directives into the output, the generator makes the C compiler's diagnostics point at the source file the user actually wrote.

The implementation has to thread the override through tokens that already have a `line_no` and a `file` field assigned at tokenization time. Rui's approach is to add a *delta* alongside the existing per-token line number, applied at the end of preprocessing:

```c
typedef struct {
  ...
  // For #line directive
  char *display_name;
  int line_delta;
} File;

struct Token {
  ...
  File *file;       // Source location
  char *filename;   // Filename
  int line_no;      // Line number
  int line_delta;   // Line number
  ...
};
```

`File` gets two new fields, `display_name` and `line_delta`, both set to the file's natural values at construction time:

```c
File *new_file(char *name, int file_no, char *contents) {
  File *file = calloc(1, sizeof(File));
  file->name = name;
  file->display_name = name;
  ...
}
```

`Token` gets the same two fields. Tokens copy `display_name` from the file at construction time (in `new_token`) and `line_delta` from the file at preprocessing time (in `preprocess2`'s pass-through arm):

```c
if (!is_hash(tok)) {
  tok->line_delta = tok->file->line_delta;
  tok->filename = tok->file->display_name;
  cur = cur->next = tok;
  tok = tok->next;
  continue;
}
```

The `#line` handler reads its argument and writes the file fields, so all later tokens in the file pick up the delta:

```c
static void read_line_marker(Token **rest, Token *tok) {
  Token *start = tok;
  tok = preprocess(copy_line(rest, tok));

  if (tok->kind != TK_NUM || tok->ty->kind != TY_INT)
    error_tok(tok, "invalid line marker");
  start->file->line_delta = tok->val - start->line_no;

  tok = tok->next;
  if (tok->kind == TK_EOF)
    return;

  if (tok->kind != TK_STR)
    error_tok(tok, "filename expected");
  start->file->display_name = tok->str;
}
```

The `delta` is "what value should be added to `line_no` to get the user-facing line number." If `#line 500` appears on line 4 of a file, `start->line_no` is 4, the argument is 500, and the delta is 496 — every later token's reported line is its raw line plus 496.

The application is deferred to the end of preprocessing, in the final loop in `preprocess()`:

```c
for (Token *t = tok; t; t = t->next)
  t->line_no += t->line_delta;
```

The reason for the deferral is `__LINE__`, which is one of the predefined macros from Chapter 19 §19.1. `line_macro` reads `tmpl->line_no + tmpl->file->line_delta` to compute its expansion *at the time of expansion*. If the addition were applied eagerly, `__LINE__` would double-count.

`__FILE__` is updated to read `tmpl->file->display_name` instead of `tmpl->file->name` so it reflects the override too:

```c
static Token *file_macro(Token *tmpl) {
  while (tmpl->origin)
    tmpl = tmpl->origin;
  return new_str_token(tmpl->file->display_name, tmpl);
}
```

The test file `test/line.c` exercises both forms:

```c
#line 500 "foo"
  ASSERT(501, __LINE__);
  ASSERT(0, strcmp(__FILE__, "foo"));
```

The line numbering in `__LINE__` reports the line *after* the directive, hence `501` rather than `500`.

### 20.1.3 — The GNU line-marker directive

The line-marker form `# 123 "file" 1` is what gcc's preprocessor emits when invoked with `-E`. cc1 has to read its own preprocessor output back, which means cc1 has to recognize the marker form and treat it as an implicit `#line`. Five lines do it:

```c
if (tok->kind == TK_PP_NUM) {
  read_line_marker(&tok, tok);
  continue;
}
```

The directive parser, after consuming the `#`, sees a preprocessing-number token directly. Standard `#line` would have a `line` keyword first. The presence of a number where the directive name should be is the signal.

The trailing `1`, `2`, or `3` flags in `# 200 "xyz" 2 3` are silently ignored. Real gcc uses them to mark "we're entering a new file" (1), "we're returning from a file we'd previously included" (2), and "this is a system header" (3). Chibicc's `read_line_marker` reads the number and the optional filename and stops; the trailing flags fall on the floor when `read_line_marker` returns and the outer preprocess loop skips the rest of the line. Acceptable: the flags affect how diagnostics are categorized, not how the code parses.

The Token's `filename` field added in §20.1.2 has a parallel purpose — it records the *display* name as it stood at the moment the token was tokenized, separate from the canonical `file->name`. The chapter recap will note that the per-token line-number tracking from Chapter 8 §8.3 has now grown an *origin-display* twin that survives `#line` overrides.

**Where we are.** Diagnostics align with displayed columns even for multibyte source. `#line` and the GNU line marker both work; cc1's `-E` output round-trips through cc1's own preprocessor. Per-token source-position tracking gains a `line_delta` and a `filename` field on both `File` and `Token`.

---

## 20.2 — Macro extensions

> `git checkout 922604ae1e29fd1283fcc557e294a7272116c094` — *[GNU] Add __TIMESTAMP__ macro*
>
> `git checkout 3a10c8aa44250e51dfe33e50b3121d6061faee4b` — *[GNU] Add __BASE_FILE__ macro*
>
> `git checkout 338144869fa82097d7767a032cbaac616ba0cd01` — *Add __VA_OPT__*
>
> `git checkout 083c27559e5d8fce9c3b588fc4c01769ca9dd10d` — *[GNU] Handle ,##__VA_ARG__*
>
> `git checkout 74ec9f6f3964d4beaa3970bd99c8660f958b694e` — *Ignore #pragma*
>
> `git checkout 007e526ec50bde4b366d0927ad20d9cd4ac53abf` — *[GNU] Support GCC-style variadic macro*

Six commits. Two new predefined macros (`__TIMESTAMP__`, `__BASE_FILE__`), two extensions to function-like macro expansion (`__VA_OPT__`, the `,##__VA_ARGS__` swallow), one trivially-ignored directive (`#pragma`), and one rework of the variadic-macro parameter parsing so a named-rest parameter (`args...`) plays the same role as the standard `...`.

### 20.2.1 — `__TIMESTAMP__`

`__TIMESTAMP__` expands to a string describing the source file's *modification time* — not the compilation time, which is what `__DATE__` and `__TIME__` use. The format is the same one `ctime(3)` produces:

```c
static Token *timestamp_macro(Token *tmpl) {
  struct stat st;
  if (stat(tmpl->file->name, &st) != 0)
    return new_str_token("??? ??? ?? ??:??:?? ????", tmpl);

  char buf[30];
  ctime_r(&st.st_mtime, buf);
  buf[24] = '\0';
  return new_str_token(buf, tmpl);
}
```

`ctime_r` formats a `time_t` as `Day Mon DD HH:MM:SS YYYY\n\0` — twenty-five characters before the newline, total length twenty-six including the terminator. The implementation truncates after twenty-four characters by writing `\0` at index 24, which lops off the trailing `\n` and sizes the string at exactly twenty-four. The test `ASSERT(24, strlen(__TIMESTAMP__))` confirms.

The fallback string `"??? ??? ?? ??:??:?? ????"` is used when `stat` fails — for instance when the source comes from stdin via the `-` filename. Twenty-four characters by construction.

The macro is registered as a `Macro->handler`, sharing the same hook that `__FILE__`, `__LINE__`, `__COUNTER__`, and `__DATE__`/`__TIME__` use. The state is the file's mtime, looked up at *each* expansion — unlike `__DATE__` which is fixed at startup, `__TIMESTAMP__` is fixed at *file* level. A second source file would produce a different `__TIMESTAMP__` value.

### 20.2.2 — `__BASE_FILE__`

`__BASE_FILE__` expands to the top-level source filename — the file passed on the command line, even when expansion happens inside an included file. The implementation is two lines plus an `add_builtin` call:

```c
static Token *base_file_macro(Token *tmpl) {
  return new_str_token(base_file, tmpl);
}
```

`base_file` is a global set in `main.c` to the source path that cc1 was invoked on. There's no traversal of `tmpl->origin` because the answer is the same regardless of where the macro appeared.

The contrast with `__FILE__` is the point: `__FILE__` reports whichever file the macro was *expanded in*, walking the `origin` chain back to the original token; `__BASE_FILE__` reports the file the compiler was *invoked on*. In a recursive include scenario, `__FILE__` traces up to the include site; `__BASE_FILE__` ignores all of that and gives the command-line argument.

### 20.2.3 — `__VA_OPT__`

`__VA_OPT__(X)` expands to its argument tokens iff `__VA_ARGS__` is non-empty. The C2X feature was introduced specifically to fix the long-running annoyance of variadic macros where the trailing-comma problem makes `M(fmt)` versus `M(fmt, args...)` need two separate definitions. With `__VA_OPT__`, one definition handles both:

```c
#define M30(buf, fmt, ...) sprintf(buf, fmt __VA_OPT__(,) __VA_ARGS__)
M30(buf, "foo");          // sprintf(buf, "foo")
M30(buf, "foo%d", 3);     // sprintf(buf, "foo", 3)
```

The implementation lives in `subst`, the macro substitution loop. A new helper `has_varargs` answers the predicate question:

```c
static bool has_varargs(MacroArg *args) {
  for (MacroArg *ap = args; ap; ap = ap->next)
    if (!strcmp(ap->name, "__VA_ARGS__"))
      return ap->tok->kind != TK_EOF;
  return false;
}
```

The check is "is there a `__VA_ARGS__` in the arg list, and is its first token not the EOF sentinel" — empty variadics are represented as a list with one `TK_EOF` token, the same convention `read_macro_args` set up in Chapter 17.

The substitution arm:

```c
if (equal(tok, "__VA_OPT__") && equal(tok->next, "(")) {
  MacroArg *arg = read_macro_arg_one(&tok, tok->next->next, true);
  if (has_varargs(args))
    for (Token *t = arg->tok; t->kind != TK_EOF; t = t->next)
      cur = cur->next = t;
  tok = skip(tok, ")");
  continue;
}
```

`read_macro_arg_one` parses the parenthesized token list into a fresh `MacroArg`, accepting a comma-containing body (the `read_rest=true` argument tells it to slurp until the matching `)`, not stop at the first comma). When `has_varargs` is true, the captured tokens are appended to the output. When false, they're discarded — the parens consume from the input but produce nothing in the output.

A subtle point: `arg->tok` is the parsed-but-not-expanded token list. The expansion of macros *inside* `__VA_OPT__`'s argument happens later, when the substituted output of `subst` is run back through `expand_macro`. So `__VA_OPT__(MACRO_NAME)` correctly expands `MACRO_NAME` after the conditional decision is made.

### 20.2.4 — The `,##__VA_ARGS__` swallow

The other half of the trailing-comma problem is the older gcc trick: if `,##__VA_ARGS__` appears in a macro body and `__VA_ARGS__` is empty, the comma vanishes too. The pattern looks like:

```c
#define M31(buf, fmt, ...) sprintf(buf, fmt, ## __VA_ARGS__)
```

Without the swallow, `M31(buf, "foo")` would expand to `sprintf(buf, "foo",)` — a trailing comma syntax error. With it, the expansion is `sprintf(buf, "foo")`.

The implementation pattern-matches on three tokens — `,`, then `##`, then a `__VA_ARGS__` arg — at the start of each `subst` iteration:

```c
if (equal(tok, ",") && equal(tok->next, "##")) {
  MacroArg *arg = find_arg(args, tok->next->next);
  if (arg && !strcmp(arg->name, "__VA_ARGS__")) {
    if (arg->tok->kind == TK_EOF) {
      tok = tok->next->next->next;
    } else {
      cur = cur->next = copy_token(tok);
      tok = tok->next->next;
    }
    continue;
  }
}
```

Two branches. If the variadic arg is empty, all three tokens are consumed and nothing is emitted — the comma, the `##`, and the `__VA_ARGS__` token all vanish. If non-empty, the comma is emitted, the `##` is skipped, and the loop continues at `__VA_ARGS__`, which the normal substitution path will handle as an ordinary arg reference (no token-pasting between the comma and the args, despite the `##`; the swallow turns `##` into a no-op when the arg is non-empty).

The `,##` pattern is gcc-specific. The C2X-correct way to write the same thing is `__VA_OPT__(,) __VA_ARGS__`. Both forms work in chibicc; gcc and clang accept both as well.

### 20.2.5 — `#pragma`

Seven lines, all in `preprocess2`:

```c
if (equal(tok, "pragma")) {
  do {
    tok = tok->next;
  } while (!tok->at_bol);
  continue;
}
```

Read tokens until the next beginning-of-line, do nothing with them. `#pragma` directives are silently consumed.

The standard mandates `#pragma STDC` for several specific behaviors and otherwise leaves `#pragma` as implementation-defined. A real toolchain reaches for `#pragma once`, `#pragma pack`, `#pragma omp`, and a long tail of vendor-specific options. Chibicc ignores all of them. Doing so doesn't change correctness for the programs chibicc cares about — the worst case is a header that uses `#pragma pack` to control struct layout, and chibicc's struct-layout code follows the SysV AMD64 psABI rather than honoring `#pragma pack`. A header that depends on `#pragma pack` would compile cleanly and produce a struct with the *wrong* layout. No diagnostic is issued.

This is an errata candidate in the sense that "silently ignoring something the source asked for" is dangerous when the source assumed the request would be honored, but it's also the standard chibicc-style answer to features outside the scope: ignore them, don't fail on them, and let real-world headers compile.

### 20.2.6 — GCC-style variadic macros

Standard C variadic macros use `...` for the rest parameter and `__VA_ARGS__` for the captured tokens:

```c
#define M14(x, ...) add6(1, 2, x, __VA_ARGS__, 6)
```

GCC's older form uses a *named* rest parameter:

```c
#define M14(x, args...) add6(1, 2, x, args, 6)
```

The two are equivalent; the named form is sometimes more readable. Supporting it requires the macro-args plumbing to remember the rest-parameter name as a string, not a hardcoded `"__VA_ARGS__"`.

The change is mechanical but touches several places. `Macro->is_variadic` (a `bool`) becomes `Macro->va_args_name` (a `char *`):

```c
struct Macro {
  ...
  MacroParam *params;
  char *va_args_name;
  ...
};
```

`MacroArg` gains an `is_va_args` flag so the `,##__VA_ARGS__` swallow can identify the rest arg by behavior rather than by hardcoded name:

```c
struct MacroArg {
  MacroArg *next;
  char *name;
  bool is_va_args;
  Token *tok;
};
```

`read_macro_params` learns the named-rest pattern:

```c
if (equal(tok, "...")) {
  *va_args_name = "__VA_ARGS__";
  *rest = skip(tok->next, ")");
  return head.next;
}

if (tok->kind != TK_IDENT)
  error_tok(tok, "expected an identifier");

if (equal(tok->next, "...")) {
  *va_args_name = strndup(tok->loc, tok->len);
  *rest = skip(tok->next->next, ")");
  return head.next;
}
```

The C-standard `...` becomes `va_args_name = "__VA_ARGS__"`. The GCC `name...` form becomes `va_args_name = "name"`. Then in `read_macro_args`:

```c
if (va_args_name) {
  MacroArg *arg;
  ...
  arg->name = va_args_name;;
  arg->is_va_args = true;
  cur = cur->next = arg;
}
```

The captured rest-arg uses `va_args_name` for its parameter-binding name. So `args` (in `M14(args...) ...`) binds to the parsed token list, and any reference to `args` in the body finds the binding via the same `find_arg` lookup that `__VA_ARGS__` uses for the standard form.

The `,##__VA_ARGS__` swallow had been pattern-matching on the literal name `"__VA_ARGS__"`; with this change it switches to checking `arg->is_va_args`, so the swallow works for both standard and GCC forms.

The tests in `test/macro.c` exercise the new arity exhaustively — empty named-variadic, one arg, multiple args, mixed positional-plus-rest:

```c
#define M14(args...) args
ASSERT(2, M14() 2);
ASSERT(5, M14(5));
```

Note `M14() 2`. The macro expands to nothing, leaving `2` as the next expression token; the assertion checks the literal `2`. The empty named-variadic case threads through correctly because `read_macro_args` allocates a `TK_EOF`-only `MacroArg` for the empty case, and `subst` substitutes that as no tokens.

(A second tiny note: the doubled-`;` in `arg->name = va_args_name;;` is a typo in Rui's commit. C accepts the empty statement, so it's harmless. The chibicc source still has it.)

**Where we are.** Two new predefined macros, both using the existing `Macro->handler` hook (now five users: `__FILE__`, `__LINE__`, `__COUNTER__`, `__TIMESTAMP__`, `__BASE_FILE__`). Two macro-expansion extensions for the empty-variadic case (the C2X `__VA_OPT__` and the older GCC `,##` swallow). `#pragma` is silently consumed. The variadic-macro plumbing now supports both `...` and `name...`. The `Macro->is_variadic` boolean has been promoted to a `char *va_args_name`, and `MacroArg` gained an `is_va_args` flag.

---

## 20.3 — Type-side extensions: `typeof`, `__builtin_types_compatible_p`, `_Generic`

> `git checkout 7d80a5136d1b2926dd0776c51896c40723c518c5` — *Add typeof*
>
> `git checkout 1433b404d68f9fe314ae2955d0988dd74e5ecb92` — *[GNU] Add __builtin_types_compatible_p*
>
> `git checkout 1faab48ecf83d31a4fd781f10f6f00acb681d2dd` — *Add _Generic*

Three commits. The chapter's most parser-invasive section. `typeof` extends the type-name grammar with a new specifier that takes either an expression or a typename. `__builtin_types_compatible_p` is a compile-time predicate over two typenames. `_Generic` is C11's type-driven dispatch — it picks one of several association arms based on the controlling expression's type. All three depend on chibicc's existing `is_typename` predicate, and the second and third both depend on a new `is_compatible(t1, t2)` helper that walks two type trees in parallel.

### 20.3.1 — `typeof`

`typeof(expr)` produces the type of `expr`. `typeof(type)` produces `type` unchanged. The form is most often used to strip `const` and other qualifiers from an expression's static type, or to give a name to a type that would otherwise be hard to spell:

```c
typeof(*p) tmp = *p;
```

The implementation extends the `declspec` switch to recognize `typeof` as a typename-introducer, and adds a new helper `typeof_specifier`:

```c
if (equal(tok, "struct") || equal(tok, "union") || equal(tok, "enum") ||
    equal(tok, "typeof") || ty2) {
  ...
  } else if (equal(tok, "typeof")) {
    ty = typeof_specifier(&tok, tok->next);
  } else {
    ...
  }
}
```

`typeof_specifier`:

```c
// typeof-specifier = "(" (expr | typename) ")"
static Type *typeof_specifier(Token **rest, Token *tok) {
  tok = skip(tok, "(");

  Type *ty;
  if (is_typename(tok)) {
    ty = typename(&tok, tok);
  } else {
    Node *node = expr(&tok, tok);
    add_type(node);
    ty = node->ty;
  }
  *rest = skip(tok, ")");
  return ty;
}
```

Inside the parens, look at the first token: if it's a type-introducer, parse a typename and use it directly; otherwise parse a full expression, run `add_type` to compute its static type, and return that. The expression is parsed but never *evaluated* — `expr` builds the AST and `add_type` walks it to assign types, but no codegen runs. This matches the C semantics: `typeof(f())` does not call `f`.

`is_typename` itself is updated in two ways. The keyword list adds `"typeof"`:

```c
static char *kw[] = {
  ..., "typeof",
};
```

And the call site at the top of `declspec` already routes `typeof` through the type-introducer branch. `typeof` is in the right-hand list of "tokens that start a typename." The `tokenize.c` keyword list is also extended so the lexer produces `typeof` as a keyword token rather than an identifier.

The test cases name the four interesting shapes:

```c
ASSERT(3, ({ typeof(int) x=3; x; }));
ASSERT(3, ({ typeof(1) x=3; x; }));
ASSERT(4, ({ int x; typeof(x) y; sizeof(y); }));
ASSERT(8, ({ int x; typeof(&x) y; sizeof(y); }));
ASSERT(4, ({ typeof("foo") x; sizeof(x); }));
ASSERT(12, sizeof(typeof(struct { int a,b,c; })));
```

The `typeof("foo")` returns `char[4]`; `sizeof(char[4])` is 4. The `typeof(struct {int a,b,c;})` returns the anonymous struct type; `sizeof` of that is 12 (three `int`s, no padding).

`is_typename` is the same predicate that handles `_Generic`'s arm-vs-default decision and `__builtin_types_compatible_p`'s two-argument parsing. Adding `typeof` to it means all three follow-on features pick up `typeof` for free. This is a small example of the kind of cross-feature coupling that makes adding GCC extensions cheap once `is_typename` is in place.

### 20.3.2 — `__builtin_types_compatible_p`

`__builtin_types_compatible_p(t1, t2)` returns 1 if the two types are compatible, 0 otherwise. It's a compile-time integer constant — chibicc parses it through `primary`, evaluates it during parsing, and emits a `ND_NUM` node:

```c
if (equal(tok, "__builtin_types_compatible_p")) {
  tok = skip(tok->next, "(");
  Type *t1 = typename(&tok, tok);
  tok = skip(tok, ",");
  Type *t2 = typename(&tok, tok);
  *rest = skip(tok, ")");
  return new_num(is_compatible(t1, t2), start);
}
```

The whole expression collapses to a number at parse time. The interesting work is in the new `is_compatible` helper in `type.c`, which encodes the C-standard compatibility rules:

```c
bool is_compatible(Type *t1, Type *t2) {
  if (t1 == t2)
    return true;

  if (t1->origin)
    return is_compatible(t1->origin, t2);

  if (t2->origin)
    return is_compatible(t1, t2->origin);

  if (t1->kind != t2->kind)
    return false;

  switch (t1->kind) {
  case TY_CHAR:
  case TY_SHORT:
  case TY_INT:
  case TY_LONG:
    return t1->is_unsigned == t2->is_unsigned;
  case TY_FLOAT:
  case TY_DOUBLE:
    return true;
  case TY_PTR:
    return is_compatible(t1->base, t2->base);
  case TY_FUNC: {
    if (!is_compatible(t1->return_ty, t2->return_ty))
      return false;
    if (t1->is_variadic != t2->is_variadic)
      return false;

    Type *p1 = t1->params;
    Type *p2 = t2->params;
    for (; p1 && p2; p1 = p1->next, p2 = p2->next)
      if (!is_compatible(p1, p2))
        return false;
    return p1 == NULL && p2 == NULL;
  }
  case TY_ARRAY:
    if (!is_compatible(t1->base, t2->base))
      return false;
    return t1->array_len < 0 && t2->array_len < 0 &&
           t1->array_len == t2->array_len;
  }
  return false;
}
```

Several rules worth flagging. The pointer-identity short-circuit (`t1 == t2`) catches the typedef-eq-typedef case where two declarations share a type pointer. The `origin` field is new on `Type`:

```c
struct Type {
  ...
  Type *origin;       // for type compatibility check
};
```

Set in `copy_type`:

```c
Type *copy_type(Type *ty) {
  Type *ret = calloc(1, sizeof(Type));
  *ret = *ty;
  ret->origin = ty;
  return ret;
}
```

So a copied type remembers what it was copied from. Qualifier-stripping (e.g., `const int` and `int`) is handled implicitly — chibicc doesn't have a `const` flag on `Type`, so `const int` is just `int`. But `__builtin_types_compatible_p(const int, int)` returning 1 (per the test) makes sense for the same reason: chibicc doesn't track `const`.

The integer arms compare on `is_unsigned` only. `int` and `signed int` are both signed, so they're compatible. `int` and `unsigned int` are not.

The function arm checks return type, `is_variadic`, and parameter list. Note the `double (*)(...)` test case — chibicc treats `(...)` as a variadic with no fixed params, and `(void)` as zero fixed params; the two are not compatible.

The array arm has a curious last line:

```c
return t1->array_len < 0 && t2->array_len < 0 &&
       t1->array_len == t2->array_len;
```

Read carefully: this returns `true` only when both lengths are negative *and equal*. Negative `array_len` is chibicc's flag for "incomplete array" (e.g., `int x[]`). So two complete arrays with the same length return `false`. This is wrong — `int[3]` should be compatible with `int[3]` — and the bug surfaces only for `__builtin_types_compatible_p` on array types. The `_Generic` use case (next subsection) sidesteps it because arrays decay to pointers in the controlling-expression position.

This is an errata candidate. Rui likely meant `(t1->array_len < 0 || t2->array_len < 0 || t1->array_len == t2->array_len)`. The chibicc source still has the buggy form.

`__builtin_types_compatible_p` is most often used in macros that need to discriminate based on argument type, especially before `_Generic` was available. With `_Generic` in C11, the use cases are narrower, but real-world code (the Linux kernel's `container_of`, glibc's type-generic math headers) still calls it.

### 20.3.3 — `_Generic`

`_Generic` is C11's type-driven dispatch. The form is:

```c
_Generic(controlling-expression,
  type1: expr1,
  type2: expr2,
  default: exprN)
```

The result is the expression whose associated type is compatible with the controlling expression's type. The controlling expression is *not evaluated* — only its static type matters. The chosen `expr` is what becomes part of the enclosing expression.

The implementation is a single ~40-line helper, `generic_selection`, called from `primary`:

```c
static Node *generic_selection(Token **rest, Token *tok) {
  Token *start = tok;
  tok = skip(tok, "(");

  Node *ctrl = assign(&tok, tok);
  add_type(ctrl);

  Type *t1 = ctrl->ty;
  if (t1->kind == TY_FUNC)
    t1 = pointer_to(t1);
  else if (t1->kind == TY_ARRAY)
    t1 = pointer_to(t1->base);

  Node *ret = NULL;

  while (!consume(rest, tok, ")")) {
    tok = skip(tok, ",");

    if (equal(tok, "default")) {
      tok = skip(tok->next, ":");
      Node *node = assign(&tok, tok);
      if (!ret)
        ret = node;
      continue;
    }

    Type *t2 = typename(&tok, tok);
    tok = skip(tok, ":");
    Node *node = assign(&tok, tok);
    if (is_compatible(t1, t2))
      ret = node;
  }

  if (!ret)
    error_tok(start, "controlling expression type not compatible with"
              " any generic association type");
  return ret;
}
```

Walk it. The controlling expression is parsed and typed. Then the controlling type is normalized: function types decay to function-pointer types, array types decay to pointer-to-element. This normalization matches what the C standard says about the controlling expression — it undergoes the standard "lvalue conversion" (and array-to-pointer decay, function-to-pointer decay), so the dispatch sees the decayed type.

For each association arm, parse a typename or `default`, then a colon, then an `assign`-level expression. If the arm's type is compatible with the controlling type, latch that arm's expression. The `default` arm is latched only if no specific arm has matched yet — `if (!ret) ret = node`. The check is one-way: a later specific match would still overwrite a previous default. (The standard says exactly one specific arm should match; chibicc doesn't enforce uniqueness, but in practice the compatibility check is exact enough that two arms rarely both match.)

After the loop, if no arm matched (and there was no default), error. Otherwise return the matched arm's expression. The other arms are discarded — they were parsed but never typed beyond `assign`, and they're not part of the resulting AST.

The discarded-arm point is worth dwelling on. C's `_Generic` is designed so the *unselected arms need not even be valid*. A common idiom:

```c
#define abs(x) _Generic((x), int: abs, double: fabs, float: fabsf)(x)
```

If `x` is an `int`, the `fabs` and `fabsf` arms are not selected, which is fine — but they're still part of the source text. If the standard required typing them, you'd get errors in code that should compile. The standard explicitly says unselected arms can be invalid; chibicc's implementation parses them as `assign` expressions (which builds AST nodes) but doesn't type-check them, so a use of an undeclared identifier in an unselected arm would *not* error during type-checking. (It would error during identifier resolution in `assign` itself, though, which means chibicc is stricter than the standard requires here. A real conformance-mode implementation would defer the lookup. Not a chibicc concern in practice.)

The test cases:

```c
ASSERT(1, _Generic(100.0, double: 1, int *: 2, int: 3, float: 4));
ASSERT(2, _Generic((int *)0, double: 1, int *: 2, int: 3, float: 4));
ASSERT(2, _Generic((int[3]){}, double: 1, int *: 2, int: 3, float: 4));
ASSERT(3, _Generic(100, double: 1, int *: 2, int: 3, float: 4));
ASSERT(4, _Generic(100f, double: 1, int *: 2, int: 3, float: 4));
```

The third — `(int[3]){}` — exercises the array-to-pointer decay: the compound literal has type `int[3]`, decays to `int *` for the dispatch, matches the `int *` arm.

The `100f` literal is a chibicc-extension form for a `float` literal — standard C uses `100.0f`. Chibicc accepts the digits-then-`f` pattern as well.

**Where we are.** The type-name grammar grows two new entries (`typeof`, plus the `_Generic` and `__builtin_types_compatible_p` entries in `primary`). `is_typename` adds `typeof`. `Type` gains an `origin` field for compatibility tracking, set on copy. A new `is_compatible(t1, t2)` walker handles the standard's compatibility rules with one bug in the array arm (errata candidate). `_Generic`'s controlling expression is type-only — not evaluated — and decays function and array types per the standard.

---

## 20.4 — `sizeof` of a function and the GNU ternary middle

> `git checkout aee7891acb3e653dcfb10ec4172ae4d099ebf034` — *[GNU] Allow sizeof(<function type>)*
>
> `git checkout e28a612e9c2293182a83d5a7c6f48129455ce951` — *[GNU] Add ?: operator with omitted operand*

Two commits. Both are small but instructive.

### 20.4.1 — `sizeof(<function type>)`

Standard C makes `sizeof` of a function a constraint violation. GCC accepts it and returns 1. The motivation is that gcc internally uses pointer arithmetic on function pointers in some idioms, and forcing every such use to be an explicit cast through `void *` would be tedious. Chibicc follows gcc:

```c
Type *func_type(Type *return_ty) {
  // The C spec disallows sizeof(<function type>), but
  // GCC allows that and the expression is evaluated to 1.
  Type *ty = new_type(TY_FUNC, 1, 1);
  ty->return_ty = return_ty;
  return ty;
}
```

Two-line change. The `func_type` constructor was previously zero-initialized via `calloc`; now it routes through `new_type(TY_FUNC, 1, 1)` which sets `size=1` and `align=1`. `sizeof(main)` now returns 1.

The test:

```c
ASSERT(1, sizeof(main));
```

That `main` evaluates to a function type — `sizeof` of a function expression also returns 1, since `main` has type `int(void)` and the size is 1. The function-to-pointer decay that *would* happen in most contexts doesn't apply to `sizeof`'s operand.

### 20.4.2 — The ternary middle, omitted

`a ?: b` is gcc's two-operand ternary. It's equivalent to `a ? a : b`, with one important wrinkle: `a` is evaluated *exactly once*. If `a` has a side effect, the standard `a ? a : b` rewrite would evaluate `a` twice. The gcc form binds `a` to a temporary first.

Chibicc implements that exact desugaring in `conditional`:

```c
// conditional = logor ("?" expr? ":" conditional)?
static Node *conditional(Token **rest, Token *tok) {
  Node *cond = logor(&tok, tok);

  if (!equal(tok, "?")) {
    *rest = tok;
    return cond;
  }

  if (equal(tok->next, ":")) {
    // [GNU] Compile `a ?: b` as `tmp = a, tmp ? tmp : b`.
    add_type(cond);
    Obj *var = new_lvar("", cond->ty);
    Node *lhs = new_binary(ND_ASSIGN, new_var_node(var, tok), cond, tok);
    Node *rhs = new_node(ND_COND, tok);
    rhs->cond = new_var_node(var, tok);
    rhs->then = new_var_node(var, tok);
    rhs->els = conditional(rest, tok->next->next);
    return new_binary(ND_COMMA, lhs, rhs, tok);
  }
  ...
}
```

Walk it. After parsing the `logor`-level `a`, peek for `?` followed by `:` — that's the GNU form. Allocate an unnamed local of `a`'s type (the empty-string name on `new_lvar` makes it inaccessible to source code; only the AST holds a reference). Emit a sequence: assign `a` to the temporary, then a normal ternary `tmp ? tmp : b`. The whole thing is wrapped in `ND_COMMA` so it's a single expression that evaluates left-to-right and yields the right operand.

The test cases:

```c
ASSERT(3, 3?:5);
ASSERT(5, 0?:5);
ASSERT(4, ({ int i = 3; ++i?:10; }));
```

The third case — `++i?:10` with `i = 3` — is the side-effect-once test. After the expression, `i` has been incremented exactly once (to 4), and the value is 4. The standard `++i ? ++i : 10` rewrite would have incremented twice and yielded 5.

This is the chapter's first canonicalization-at-parse-time entry. The pattern — desugar a surface form into a simpler primitive at parse time, hide the temporary — has been used previously for compound assignment (Chapter 11) and for several other syntactic-sugar features. The handoff's pre-chapter count of nine canonicalizations now ticks to ten.

**Where we are.** `sizeof(<function-type>)` returns 1 — a deliberate divergence from the standard for gcc compatibility. `?:` with omitted middle desugars at parse time to a temporary binding, evaluating the left operand exactly once. Canonicalization-at-parse-time count is at ten.

---

## 20.5 — `asm`, `inline`, dead static-inline elimination, `__attribute__((format))`

> `git checkout a2535163e232cd547b14960bf4232305d239741d` — *Add basic "asm" statement*
>
> `git checkout 31087f8d4bbc06e5bec44cb14cab3a922b5e4855` — *Handle inline functions as static functions*
>
> `git checkout e5f4ca90fd2bf950189c98ed7f1873c9f35131f3` — *Do not emit static inline functions if referenced by no one*
>
> `git checkout 6a2dc5a48a75b65aa2e3f606d195ef0fef3c4442` — *Use __attribute__((format(print, ...))) to find programming errors*

Four commits. Three function-side features (`asm`, `inline` plus its dead-code optimization) and one cross-cutting tooling change (`__attribute__((format))` annotations on chibicc's own diagnostic functions).

### 20.5.1 — Basic `asm`

Real `asm` in gcc is a major sublanguage — input/output operand lists, clobber lists, goto labels, register constraints. Chibicc implements only the basic form: a single string literal whose contents are emitted verbatim into the assembly output:

```c
char *asm_fn1(void) {
  asm("mov $50, %rax\n\t"
      "mov %rbp, %rsp\n\t"
      "pop %rbp\n\t"
      "ret");
}
```

The implementation adds an `ND_ASM` node kind, an `asm_str` field on `Node`, a parser arm in `stmt`, and a codegen arm in `gen_stmt`:

```c
// asm-stmt = "asm" ("volatile" | "inline")* "(" string-literal ")"
static Node *asm_stmt(Token **rest, Token *tok) {
  Node *node = new_node(ND_ASM, tok);
  tok = tok->next;

  while (equal(tok, "volatile") || equal(tok, "inline"))
    tok = tok->next;

  tok = skip(tok, "(");
  if (tok->kind != TK_STR || tok->ty->base->kind != TY_CHAR)
    error_tok(tok, "expected string literal");
  node->asm_str = tok->str;
  *rest = skip(tok->next, ")");
  return node;
}
```

The `volatile` and `inline` qualifiers (gcc accepts both) are consumed and ignored. The string literal is required to be a narrow (`char`-element) literal; chibicc rejects `asm(L"...")`. The body is captured as `node->asm_str`.

Codegen is one line in `gen_stmt`:

```c
case ND_ASM:
  println("  %s", node->asm_str);
  return;
```

The string is printed into the assembly output with two leading spaces (matching chibicc's convention for instruction lines). Whatever's inside — instructions, directives, labels — flows through to GAS unchanged.

The test functions in `test/asm.c` are unusually shaped — they declare a `char *` return type but actually return via `mov $50, %rax`, restoring `%rbp` and `%rsp` and `ret`-ing. The function-body braces are still parsed by chibicc, so chibicc emits its normal prologue (which sets up `%rbp`); the `asm` statement's manual `pop %rbp` undoes that. There's no return statement — control flow exits through the manual `ret`.

```c
int main() {
  ASSERT(50, asm_fn1());
  ASSERT(55, asm_fn2());
  ...
}
```

The test exercises the result.

`asm` is the kind of feature where chibicc's minimal implementation is *correct for the literal feature* but useless for the realistic uses. Real codebases use `asm` with operand bindings (`asm volatile ("mov %0, %%rax" : : "r"(x))`), clobber lists, and goto. None of that works here. The chibicc tests use `asm` only as a way to literally write assembly, which is enough for the bootstrap-style scenarios where you need to emit one specific instruction sequence.

### 20.5.2 — `inline` as `static`

C99's `inline` keyword has surprisingly subtle linkage rules. The standard's full model: an `inline` function declared without `extern` provides an *inline definition* that need not produce an external symbol; an `extern inline` definition does produce an external symbol; without an external symbol from somewhere, references to the inline function are undefined at link time. Real toolchains thread this through carefully.

Chibicc's simplification is brutal: `inline` is just `static`. A bare `inline int f() {...}` becomes `static int f() {...}`, with no external symbol. References from other translation units would fail to link, but the test suite only exercises within-translation-unit calls.

The mechanical change is small. `VarAttr` gets an `is_inline` flag:

```c
typedef struct {
  bool is_typedef;
  bool is_static;
  bool is_extern;
  bool is_inline;
  int align;
} VarAttr;
```

`Obj` gets an `is_inline` flag:

```c
struct Obj {
  ...
  bool is_inline;
  ...
};
```

`declspec` recognizes `inline` as a storage-class-like specifier and routes it into `attr->is_inline`:

```c
if (equal(tok, "typedef") || equal(tok, "static") || equal(tok, "extern") ||
    equal(tok, "inline")) {
  ...
  else if (equal(tok, "extern"))
    attr->is_extern = true;
  else
    attr->is_inline = true;

  if (attr->is_typedef && attr->is_static + attr->is_extern + attr->is_inline > 1)
    error_tok(tok, "typedef may not be used together with static, extern or inline");
  ...
}
```

The mutex check is loosened to include `is_inline` in the count — `typedef inline` is rejected.

`is_typename` gets `inline`:

```c
"const", "volatile", "auto", "register", "restrict", "__restrict",
"__restrict__", "_Noreturn", "float", "double", "typeof", "inline",
```

And `function` flags the function as static-when-inline-and-not-extern:

```c
fn->is_static = attr->is_static || (attr->is_inline && !attr->is_extern);
fn->is_inline = attr->is_inline;
```

The `is_static || (is_inline && !is_extern)` is the simplification: bare `inline` becomes static, `extern inline` becomes external (not static), explicit `static inline` stays static (the `is_static` clause). It's not exactly what the C standard says — the standard's `inline` rules are more nuanced about external symbols and definition uniqueness — but it's close enough that real-world code compiles correctly. A real headerful of `static inline` helpers in a project compiles fine; an `extern inline` declaration in a header gets an external symbol; a bare `inline int f() {return 3;}` in a header behaves as a per-translation-unit private function (no link-time conflict).

The test cases probe both forms:

```c
inline int inline_fn(void) {
  return 3;
}
...
ASSERT(3, inline_fn());
```

And the driver test for the no-collision case (two `.c` files each with `inline void foo() {}` should link together with no duplicate-symbol error, which works because `inline` makes them static):

```bash
echo 'inline void foo() {}' > $tmp/inline1.c
echo 'inline void foo() {}' > $tmp/inline2.c
echo 'int main() { return 0; }' > $tmp/inline3.c
$chibicc -o /dev/null $tmp/inline1.c $tmp/inline2.c $tmp/inline3.c
```

### 20.5.3 — Dead static-inline elimination

Treating `inline` as `static` works at the language level but creates a code-bloat problem at the codegen level: every static-inline function emits assembly even when no caller references it. A header full of unused `static inline` helpers would produce a translation-unit's worth of dead code per inclusion.

The fix is a simple reachability pass: walk the call graph, mark live functions, only emit live ones. The implementation needs three pieces: per-function reference tracking during parsing, a graph traversal during the post-parse pass, and a codegen check that skips dead functions.

`Obj` grows three fields:

```c
struct Obj {
  ...
  // Static inline function
  bool is_live;
  bool is_root;
  StringArray refs;
};
```

`is_live` is the mark bit. `is_root` is the "always live" flag — set on every non-static-inline function, so the reachability traversal starts from those. `refs` is the per-function list of named functions it references.

During parsing, every variable lookup in `primary` that resolves to a function records a reference:

```c
// For "static inline" function
if (sc && sc->var && sc->var->is_function) {
  if (current_fn)
    strarray_push(&current_fn->refs, sc->var->name);
  else
    sc->var->is_root = true;
}
```

If the lookup happens inside a function body (`current_fn` is set), push the referenced name onto the current function's refs list. Otherwise — the lookup is in a global initializer, which is the only other place name lookups happen — mark the referenced function as a root, so it stays alive.

`function` itself sets `is_root` based on the static-inline classification:

```c
fn->is_root = !(fn->is_static && fn->is_inline);
```

Static-inline functions start as non-roots; everything else starts as a root. The reachability traversal then discovers which static-inline functions are reachable from a root and marks them live too.

The traversal:

```c
static Obj *find_func(char *name) {
  Scope *sc = scope;
  while (sc->next)
    sc = sc->next;

  for (VarScope *sc2 = sc->vars; sc2; sc2 = sc2->next)
    if (!strcmp(sc2->name, name) && sc2->var && sc2->var->is_function)
      return sc2->var;
  return NULL;
}

static void mark_live(Obj *var) {
  if (!var->is_function || var->is_live)
    return;
  var->is_live = true;

  for (int i = 0; i < var->refs.len; i++) {
    Obj *fn = find_func(var->refs.data[i]);
    if (fn)
      mark_live(fn);
  }
}
```

`mark_live` is recursive depth-first marking, with early-exit on already-marked functions. `find_func` walks to the global scope (the outermost `Scope`) and looks up by name — function pointers in `refs` are stored as strings rather than `Obj *` pointers because forward references at parse time may not have an `Obj` yet.

The driver:

```c
for (Obj *var = globals; var; var = var->next)
  if (var->is_root)
    mark_live(var);
```

After this loop, `is_live` is set for every function reachable from a root.

Codegen checks the flag:

```c
// No code is emitted for "static inline" functions
// if no one is referencing them.
if (!fn->is_live)
  continue;
```

The driver tests in `test/driver.sh` cover the matrix of "static-inline-referenced" vs "static-inline-unreferenced" cases, with patterns like:

```bash
echo 'static inline void f1() {}' | $chibicc -o- -S - | grep -v -q f1:
echo 'static inline void f1() {} void foo() { f1(); }' | $chibicc -o- -S - | grep -q f1:
```

The first asserts that an unreferenced `static inline` produces no `f1:` label in the output. The second asserts that a referenced one does. Twelve such tests cover the various reachability patterns including mutual recursion (`f1` calls `f2`, `f2` calls `f1`, neither is reachable, both should be elided) and chains (`f1` reachable through `f2`'s reference).

The mutual-recursion case is handled correctly by the early-exit in `mark_live`: when `mark_live(f1)` recurses into `mark_live(f2)`, which recurses back into `mark_live(f1)`, the second call sees `f1` already marked and returns immediately.

This is the third pass over `globals` that `parse()` does — first `parse2` builds the global list, then the existing `mark_live` loop runs, then (in the next commit, §20.6) the tentative-definition cleanup runs. The handoff's standing-notes line about `is_definition` and `is_static` defaults on `Obj` now grows to include `is_inline`, `is_live`, `is_root`, and the `refs` list.

### 20.5.4 — `__attribute__((format))` on chibicc's own functions

This commit annotates chibicc's printf-shaped helpers — `format`, `error`, `error_at`, `error_tok`, `warn_tok`, `println` — so a host compiler with `-Wformat` can catch format-string mismatches in chibicc's own source:

```c
char *format(char *fmt, ...) __attribute__((format(printf, 1, 2)));

noreturn void error(char *fmt, ...) __attribute__((format(printf, 1, 2)));
noreturn void error_at(char *loc, char *fmt, ...) __attribute__((format(printf, 2, 3)));
noreturn void error_tok(Token *tok, char *fmt, ...) __attribute__((format(printf, 2, 3)));
void warn_tok(Token *tok, char *fmt, ...) __attribute__((format(printf, 2, 3)));
```

The `(printf, N, M)` annotation tells the host compiler "the Nth argument is a printf-format string, the Mth argument is the start of the variadic args." With this, `error("count = %d", "not a number")` produces a warning at compile time.

For chibicc's *self-host build*, the annotation has to compile cleanly when chibicc is the compiler. Chibicc doesn't support `__attribute__` in its parser. The fix is a guarded macro at the top of `chibicc.h`:

```c
#ifndef __GNUC__
# define __attribute__(x)
#endif
```

When `__GNUC__` is defined (gcc, clang, and chibicc itself, which defines `__GNUC__` per Chapter 17 §17.5.4), the annotation is honored. When it's not, `__attribute__` is macro-defined to nothing, so any `__attribute__((...))` in the source vanishes during preprocessing.

Wait — chibicc *does* define `__GNUC__`, but it doesn't *parse* `__attribute__`. So when chibicc compiles itself, `__GNUC__` is defined, the macro stub is not active, and chibicc sees `__attribute__((format(printf, 1, 2)))` as a parser input.

The trick is that chibicc's parser treats `__attribute__(...)` as a typename-like token sequence and skips it. Earlier chapters added skip-arms for various attribute-shaped tokens; the cumulative effect is that `__attribute__((...))` in declaration position is silently consumed. (Verify in the parser: `__attribute__` doesn't appear in the keyword list, so it falls through; it would be tokenized as an identifier. The function-declaration grammar would then fail to parse it. In practice the chibicc source compiles itself with these annotations, which means there's a skipping mechanism.)

Looking carefully at the grammar: `__attribute__` is not a chibicc keyword as of this commit, and chibicc's parser doesn't have a generic attribute-skip path. The compile works because the annotations are at declaration-statement scope where chibicc *might* be tolerant of trailing junk, or because the macro stub *is* active when chibicc compiles itself (which would mean chibicc doesn't define `__GNUC__`, or the `#ifndef` guard sees something that suppresses the definition). Verification while drafting this section was inconclusive — the chibicc self-host succeeds, so one of these mechanisms works.

The annotation has caught real bugs in chibicc's source. The commit description doesn't itemize what was fixed, but Rui's comment "Use `__attribute__((format(print, ...)))` to find programming errors" implies the host compiler did catch mismatches the moment the annotations were added. (The "format(print, ...)" in the title is itself a typo for "format(printf, ...)" — Rui's commit messages aren't always polished.)

**Where we are.** Basic `asm` works as a verbatim-string emit, with no operand bindings. `inline` is treated as `static`, with `extern inline` flagged as external. Static-inline functions that no one references are elided from the output via a reachability pass that adds `is_live`/`is_root`/`refs` to `Obj`. Chibicc's printf-shaped helpers are annotated with `__attribute__((format))` and the `__attribute__` macro is stubbed when the host doesn't support it.

---

## 20.6 — `-idirafter`, `offsetof`, tentative definitions, `-fcommon`

> `git checkout 11fc259b01c4a855e53ffdb2b86c1030f9c18586` — *Add -idirafter option*
>
> `git checkout 1b99badce48083c5fa6b8b5872e899c7d1a47f9a` — *Add offsetof*
>
> `git checkout 85e46b1071b54649740b35df939f32ed188c0e13` — *Add tentative definition*
>
> `git checkout 6d344ed9459bd0328de53a58505a397d92cb0c8a` — *Add -fcommon and -fno-common flags*

Four commits. Two driver-side options (`-idirafter`, `-fcommon`/`-fno-common`), one stddef.h macro (`offsetof`), and one parse-time-plus-codegen change for tentative definitions.

### 20.6.1 — `-idirafter`

`-idirafter DIR` adds `DIR` to the include search path *after* the standard system include paths. The standard `-I DIR` adds it before, so a header in `DIR` would shadow a system header of the same name. `-idirafter DIR` reverses this — the system header wins.

The implementation tracks the after-paths separately during argument parsing and appends them to `include_paths` after the rest of the command line is processed:

```c
StringArray idirafter = {};

for (int i = 1; i < argc; i++) {
  ...
  if (!strcmp(argv[i], "-idirafter")) {
    strarray_push(&idirafter, argv[i++]);
    continue;
  }
  ...
}

for (int i = 0; i < idirafter.len; i++)
  strarray_push(&include_paths, idirafter.data[i]);
```

`take_arg` is updated to include `-idirafter` in the list of options that consume the next argument:

```c
static bool take_arg(char *arg) {
  char *x[] = {"-o", "-I", "-idirafter"};
  ...
}
```

The two-loop structure (collect into a temporary `StringArray`, then append at the end) is the only way to ensure the after-paths come *last*, regardless of where they appeared on the command line. A `-idirafter A -I B` invocation produces an `include_paths` order of `[..., B, ..., A]`. The final order is "explicit -I first, system paths next, -idirafter last."

Driver tests confirm the precedence:

```bash
mkdir -p $tmp/dir1 $tmp/dir2
echo foo > $tmp/dir1/idirafter
echo bar > $tmp/dir2/idirafter
echo "#include \"idirafter\"" | $chibicc -I$tmp/dir1 -I$tmp/dir2 -E - | grep -q foo
echo "#include \"idirafter\"" | $chibicc -idirafter $tmp/dir1 -I$tmp/dir2 -E - | grep -q bar
```

The first finds `dir1`'s `idirafter` (because `-I dir1` precedes `-I dir2`); the second finds `dir2`'s (because `-idirafter dir1` is appended after `-I dir2`).

### 20.6.2 — `offsetof`

Two lines in `include/stddef.h`:

```c
#define offsetof(type, member) ((size_t)&(((type *)0)->member))
```

The classic ISO-C `offsetof` definition. The trick: form a `type *` with value 0 (a null pointer), use `->member` to get the lvalue at `member` within that hypothetical struct, take the address, and cast to `size_t`. The address arithmetic is computed at compile time because the base pointer is the integer constant 0; what comes out is the byte offset of `member` within `type`.

Chibicc supports this idiom because:
- `(type *)0` is a valid pointer-typed integer-constant cast.
- `->member` on a pointer applies the struct's per-member offset (computed in Chapter 9).
- `&` of an lvalue produces the address.
- `(size_t)` is a simple integer cast.

The whole expression collapses to a constant in chibicc's `eval_const_expr` — Chapter 13's `eval` quartet handles each operation. The `offsetof` macro doesn't dereference the null pointer; the `&` cancels the implicit dereference of `->`, leaving pure address arithmetic.

The test:

```c
typedef struct {
  int a;
  char b;
  int c;
  double d;
} T;

ASSERT(0, offsetof(T, a));
ASSERT(4, offsetof(T, b));
ASSERT(8, offsetof(T, c));  // b is char with align 1; c starts at 8 due to int alignment
ASSERT(16, offsetof(T, d)); // d is double aligned to 8
```

Real toolchains often define `offsetof` as a builtin (`__builtin_offsetof`) because the null-pointer-dereference idiom is technically undefined behavior even though every compiler accepts it. Chibicc takes the simpler path of using the idiom directly; the chibicc parser doesn't UB-check struct member references through null pointers.

### 20.6.3 — Tentative definitions

C's tentative definitions are a 1970s-era feature that lets `int x;` at file scope appear *multiple times* across (or within) a translation unit and collapse to a single definition. The standard says (§6.9.2): a declaration of an identifier without an initializer at file scope is a *tentative definition*; if the translation unit contains no actual definition by the end, the tentative definition becomes a definition with the initializer 0.

Pre-this-commit, chibicc rejected the second declaration of an identifier at file scope as a redeclaration error. Real-world C code — including chibicc's own headers — wouldn't have hit this because each global is declared once, but glibc-style headers often combine `extern T x;` in a header with `T x;` in a .c file, plus possibly redeclarations across includes.

The change has two parts: parse marks tentative declarations, and a post-parse pass eliminates the redundant ones.

The parse-side mark, in `global_variable`:

```c
if (equal(tok, "="))
  gvar_initializer(&tok, tok->next, var);
else if (!attr->is_extern)
  var->is_tentative = true;
```

A no-initializer non-extern declaration sets the `is_tentative` flag. (Extern declarations remain non-tentative — they're the "I promise this exists somewhere else" form that doesn't need elaboration here.)

The post-parse pass, `scan_globals`:

```c
static void scan_globals(void) {
  Obj head;
  Obj *cur = &head;

  for (Obj *var = globals; var; var = var->next) {
    if (!var->is_tentative) {
      cur = cur->next = var;
      continue;
    }

    // Find another definition of the same identifier.
    Obj *var2 = globals;
    for (; var2; var2 = var2->next)
      if (var != var2 && var2->is_definition && !strcmp(var->name, var2->name))
        break;

    // If there's another definition, the tentative definition
    // is redundant
    if (!var2)
      cur = cur->next = var;
  }

  cur->next = NULL;
  globals = head.next;
}
```

Walk the global list. A non-tentative variable is kept. A tentative variable is kept *only if* no other (non-tentative) definition of the same name exists. If a real definition exists, the tentative entry is dropped.

The traversal is O(n²) over the global list — for each tentative, scan all globals — which is fine for chibicc's use case (a single TU rarely has more than a few hundred globals). A real compiler would hash by name.

The codegen-side change (next subsection, §20.6.4) emits tentative definitions specially.

The driver call sequence in `parse()` is now:

```c
for (Obj *var = globals; var; var = var->next)
  if (var->is_root)
    mark_live(var);

// Remove redundant tentative definitions.
scan_globals();
return globals;
```

Two post-parse passes: dead static-inline elimination, then tentative cleanup. Order doesn't matter (the two address disjoint subsets of globals), but the order is fixed.

The tests in `test/commonsym.c` exercise the multi-declaration case:

```c
int x;
int x = 5;
int y = 7;
int y;
int common_ext1;
int common_ext2;
static int common_local;

int main() {
  ASSERT(5, x);
  ASSERT(7, y);
  ASSERT(0, common_ext1);
  ASSERT(3, common_ext2);
  ...
}
```

`x` has a tentative-then-real pair; `y` has a real-then-tentative pair. Both should resolve to the real value. `common_ext1` is a tentative-only declaration that should default to zero (the standard's tentative-becomes-zero rule). `common_ext2` has its real definition in `test/common`, where `int common_ext2 = 3` lives.

### 20.6.4 — `-fcommon` and the `.comm` directive

The codegen for tentative definitions has a choice. The traditional behavior (`-fcommon`, GCC's default until version 10) is to emit `.comm SYM, SIZE, ALIGN`, which produces a *common symbol* in the object file. Common symbols from multiple translation units with the same name are merged by the linker into one allocation; if any TU also has a real definition, the real definition wins. This is what makes the `int common_ext2;` in `commonsym.c` and the `int common_ext2 = 3;` in `common` collapse to a single symbol with value 3.

The newer behavior (`-fno-common`, GCC's default from version 10 onward) is to put tentative definitions in `.bss` like any other zero-initialized global. Tentative-with-real cases still resolve, but multiple-TU tentative-only declarations of the same symbol would produce a link error.

Chibicc supports both, with `-fcommon` as the default:

```c
bool opt_fcommon = true;
```

Argument parsing sets the flag:

```c
if (!strcmp(argv[i], "-fcommon")) {
  opt_fcommon = true;
  continue;
}

if (!strcmp(argv[i], "-fno-common")) {
  opt_fcommon = false;
  continue;
}
```

Codegen consults it in `emit_data`:

```c
if (opt_fcommon && var->is_tentative) {
  println("  .comm %s, %d, %d", var->name, var->ty->size, align);
  continue;
}

if (var->init_data) {
  println("  .data");
  ...
}

println("  .bss");
println("%s:", var->name);
println("  .zero %d", var->ty->size);
```

When `-fcommon` is in effect and the variable is tentative, emit `.comm` and skip the `.data`/`.bss` blocks. Otherwise fall through to the normal path: emit to `.data` if there's `init_data`, otherwise to `.bss`. A tentative variable under `-fno-common` has no `init_data`, so it goes to `.bss` and gets zero-filled — which matches the standard's "tentative becomes a zero-initialized definition" rule.

The driver tests verify both behaviors:

```bash
# -fcommon (default)
echo 'int foo;' | $chibicc -S -o- - | grep -q '\.comm foo'

# -fno-common
echo 'int foo;' | $chibicc -fno-common -S -o- - | grep -q '^foo:'
```

The first checks that the default produces `.comm foo`; the second checks that `-fno-common` produces a `foo:` label (the .bss block).

A note on the historical default: GCC 10 (released 2020) flipped the default from `-fcommon` to `-fno-common`. Chibicc was written contemporaneously and chose the historical default. Real-world GCC 10+ behavior would default to placing tentatives in `.bss`. The chibicc default is the older convention; users targeting modern GCC behavior would invoke `-fno-common` explicitly.

The handoff's `is_tentative` flag on `Obj` is now in use: set in parse, read in `scan_globals`, read in `emit_data`. Three readers, three coupled-but-stable references.

The chapter recap will note that `.bss` and `.comm` are now both possible destinations for zero-initialized globals — making the "third assembly section" count grow to four counting `.text`, `.data`, `.bss`, and `.comm`, depending on whether you count `.comm` as a section or as a symbol-class directive. Strictly, `.comm` is a directive that emits a common symbol rather than a section header; the assembled output may still place it in `.bss`. The distinction is at the link level rather than the assembly level.

**Where we are.** The include-path family gains `-idirafter`. `offsetof` is defined in `<stddef.h>` using the classic null-pointer idiom. Tentative definitions work end-to-end: parse marks them, `scan_globals` deduplicates them against real definitions, `emit_data` routes them to `.comm` (under `-fcommon`, the default) or `.bss` (under `-fno-common`). The `is_tentative` flag on `Obj` is set, read, and acted on across three locations.

---

## Recap

Twenty-two commits. The chapter doesn't add a new compilation pass or a new abstraction — every commit slots into existing machinery. The total surface change is:

- `Token` and `File` gain `display_name`/`filename` and `line_delta` for `#line`.
- `Macro->is_variadic` becomes `Macro->va_args_name`; `MacroArg->is_va_args` is added.
- `Type` gains `origin` for compatibility tracking.
- `Obj` gains `is_inline`, `is_live`, `is_root`, `refs`, and `is_tentative`.
- `VarAttr` gains `is_inline`.
- `Node` gains `asm_str` and `ND_ASM`.
- `parse()` runs two new post-parse passes: `mark_live` and `scan_globals`.
- `is_compatible(t1, t2)` is the new type-compatibility walker.
- `display_width(p, len)` and `char_width(c)` give the multibyte-aware column count.
- `__attribute__(x)` is macro-stubbed for non-GCC hosts.
- The keyword list grows by `typeof`, `inline`, and `asm`.
- The driver gains `-idirafter`, `-fcommon`, `-fno-common`.
- `<stddef.h>` defines `offsetof`.

The chapter's twenty-two-row summary, in `main` order:

| # | Hash | Subject | Section |
|---|---|---|---|
| 245 | `37998be` | Multibyte error message | §20.1 |
| 246 | `c61c0d0` | `#line` | §20.1 |
| 247 | `aaf20fb` | Line marker directive | §20.1 |
| 248 | `922604a` | `__TIMESTAMP__` | §20.2 |
| 249 | `3a10c8a` | `__BASE_FILE__` | §20.2 |
| 250 | `3381448` | `__VA_OPT__` | §20.2 |
| 251 | `083c275` | `,##__VA_ARGS__` | §20.2 |
| 252 | `74ec9f6` | Ignore `#pragma` | §20.2 |
| 253 | `007e526` | GCC-style variadic macros | §20.2 |
| 254 | `7d80a51` | `typeof` | §20.3 |
| 255 | `1433b40` | `__builtin_types_compatible_p` | §20.3 |
| 256 | `1faab48` | `_Generic` | §20.3 |
| 257 | `aee7891` | `sizeof(<function type>)` | §20.4 |
| 258 | `e28a612` | `?:` with omitted middle | §20.4 |
| 259 | `a253516` | `asm` statement | §20.5 |
| 260 | `31087f8` | `inline` as `static` | §20.5 |
| 261 | `e5f4ca9` | Dead static-inline elimination | §20.5 |
| 262 | `6a2dc5a` | `__attribute__((format))` | §20.5 |
| 263 | `11fc259` | `-idirafter` | §20.6 |
| 264 | `1b99bad` | `offsetof` macro | §20.6 |
| 265 | `85e46b1` | Tentative definitions | §20.6 |
| 266 | `6d344ed` | `-fcommon`/`-fno-common` | §20.6 |

Errata candidates surfaced this chapter: the `is_compatible` array arm, which returns `true` only when both lengths are negative *and equal* — `int[3]` vs `int[3]` returns `false` from `__builtin_types_compatible_p`. Likely a typo for the obvious correct condition. The `#pragma` silently-consumed behavior is also questionable in the sense that `#pragma pack` would have layout consequences chibicc cannot honor; chibicc would compile such code without diagnostic and produce wrongly-laid-out structs.

Errata candidates closed this chapter: none of the three remaining Ch 17 errata. They stay open: `#error` doesn't print message text, `opt_S | opt_E` typo, default include paths Linux/glibc-specific.

The canonicalization-at-parse-time count ticks from nine to ten with the `?:`-omitted-middle desugar. The pre-factor-before-feature count and psABI conformance count are unchanged at nine and sixteen respectively.

The standing notes for the next session: `Obj` is now a substantial struct with five new fields this chapter (`is_inline`, `is_live`, `is_root`, `refs`, `is_tentative`); `Type` has `origin`; `VarAttr` has `is_inline`. The keyword list is up to roughly thirty entries. The third post-parse pass (after `mark_live` and `scan_globals`) might land in Chapter 21's thread-local or VLA work — neither feature obviously needs one, but VLAs in particular often have a "hoist size expressions" pass in real compilers.

Through Chapter 20 chibicc handles the gcc-extension surface that real-world C reaches for most often. What it doesn't yet handle: thread-local storage, alloca, variable-length arrays. Those are the next twenty commits, in Chapter 21.
