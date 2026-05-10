# Chapter 19 — Unicode and designated initializers

> Commits covered: `e27417f`, `0e77f3d`, `74bcec5`, `c31886a`, `a57c661`, `454618c`, `2dac3af`, `57b21fe`, `9cabe1f`, `c467ee6`, `cae061a`, `36230e0`, `6adba75`, `e4491b8`, `0e5d250`, `adb8b98`, `2382777`, `2b2fa25`, `c618c3b`, `835cd24`, `691c4fa`, `67f5834`, `31dc1df`, `95eb5b0`. Twenty-four commits — the full Unicode arc (universal character names, multibyte and wide character literals, UTF-{8,16,32} string literals and their initializers, UTF-8 in identifiers, UTF-8 BOM stripping, cross-prefix string concatenation), the GNU `$`-in-identifier extension, the trailing date/time/counter macros that Chapter 17 deferred, and the designated-initializer arc (array, struct, union, anonymous-struct, GNU `=`-omission).

Through Chapter 18, chibicc has the SysV AMD64 calling convention, bitfields, and a self-hosting build that's `-Wall`-clean. What it doesn't have is Unicode. The tokenizer accepts ASCII identifiers and ASCII string literals; a Japanese variable name is a parse error, a `α` escape sequence is two tokens (`\u` followed by `03B1`), and a `u"αβ"` literal is the identifier `u` followed by an ASCII string.

What chibicc also doesn't have is *designated initializers* — the C99 form `struct foo x = {.a = 1, .b = 2}` and the array form `int x[10] = {[3] = 1, [7] = 2}`. The `Initializer` tree from Chapter 12 fills children left-to-right; jumping the cursor by name or index requires extending the parser.

The twenty-four commits in this chapter address both topics. They arrive on `main` in chronological order, and on calendar time they scatter from May 2020 (the `__DATE__` macro) to October 2020 (the cross-prefix concatenation, the BOM skip), with several reordered relative to when Rui actually wrote them — the wide-character commit `a57c661` is dated May 6, 2020 but lands at position 225 on `main`, after several later-dated commits.

Seven sections from twenty-four commits.

- **§19.1** — Date, time, counter macros (commits 221–222).
- **§19.2** — Newline canonicalization (commit 223).
- **§19.3** — Universal character names (commit 224).
- **§19.4** — Multibyte and wide character literals (commits 225–227).
- **§19.5** — UTF-8, UTF-16, UTF-32, and wide string literals (commits 228–234).
- **§19.6** — Identifier-side Unicode and BOM (commits 235–238).
- **§19.7** — Designated initializers (commits 239–244).

The Unicode arc (§19.3 through §19.6) is the chapter's deepest single stretch — twelve commits, two new files (`unicode.c` and `test/unicode.c`), and a small but principled split between the *tokenizer* (which now does UTF-8 decoding) and the *preprocessor* (which now normalizes the prefixes of adjacent string literals). The designated-initializer arc (§19.7) is the chapter's most invasive single stretch — six commits, all in `parse.c`, all reshaping the same handful of helpers (`array_initializer1`, `array_initializer2`, `struct_initializer1`, `struct_initializer2`, `union_initializer`, `count_array_init_elements`, plus the new `designation` and `array_designator` and `struct_designator`).

The chapter follows `main` order.

---

## 19.1 — Date, time, counter macros

> `git checkout e27417fcde500f6c01ce0dbee57a1af137510a09` — *Add __DATE__ and __TIME__ macros*
>
> `git checkout 0e77f3dff8b44547da4639c9609c216c9c896fa5` — *[GNU] Add __COUNTER__ macro*

Two commits. Both extend `init_macros` in `preprocess.c` with new predefined macros, all three of which use the `Macro->handler` field that Chapter 17 §17.5.3 added for `__FILE__` and `__LINE__`.

`__DATE__` and `__TIME__` are stateless from the program's view: they expand to the same string every time, fixed at the moment cc1 starts. The implementation is two `format_*` helpers and a single `time(NULL)` call at the bottom of `init_macros`:

```c
// __DATE__ is expanded to the current date, e.g. "May 17 2020".
static char *format_date(struct tm *tm) {
  static char mon[][4] = {
    "Jan", "Feb", "Mar", "Apr", "May", "Jun",
    "Jul", "Aug", "Sep", "Oct", "Nov", "Dec",
  };

  return format("\"%s %2d %d\"", mon[tm->tm_mon], tm->tm_mday, tm->tm_year + 1900);
}

// __TIME__ is expanded to the current time, e.g. "13:34:03".
static char *format_time(struct tm *tm) {
  return format("\"%02d:%02d:%02d\"", tm->tm_hour, tm->tm_min, tm->tm_sec);
}
```

And, at the end of `init_macros`:

```c
time_t now = time(NULL);
struct tm *tm = localtime(&now);
define_macro("__DATE__", format_date(tm));
define_macro("__TIME__", format_time(tm));
```

Two small choices worth flagging. The first is that the date and time are local-zone, not UTC — `localtime` rather than `gmtime`. Two different machines compiling the same source on the same wall-clock instant in different time zones produce different `__DATE__` strings. The C standard (§6.10.8) says only that `__DATE__` is "the date of translation"; the choice of zone is implementation-defined. Chibicc picks local; gcc and clang also pick local.

The second is that `format_date` and `format_time` produce strings with embedded quotation marks (`"May 17 2020"`, including the `"`). The macros are defined via `define_macro`, which calls the tokenizer on the value string, so the quotes are *part of the macro body* — they make the body a string literal, which is what `__DATE__` and `__TIME__` are supposed to expand to.

These two macros close a small gap that Chapter 17 left open — Rui's preprocessor commits implemented `__FILE__`, `__LINE__`, `__VA_ARGS__`, and the rest of the predefined-macro vocabulary, but `__DATE__` and `__TIME__` are unique in needing real-time-clock state at startup, and Rui broke them out into their own commit.

`__COUNTER__` is a GCC extension. Each expansion produces the next integer in the sequence 0, 1, 2, … . The implementation is six lines:

```c
// __COUNTER__ is expanded to serial values starting from 0.
static Token *counter_macro(Token *tmpl) {
  static int i = 0;
  return new_num_token(i++, tmpl);
}
```

And one line in `init_macros`:

```c
add_builtin("__COUNTER__", counter_macro);
```

The `static int i` inside the handler is the entire state. The `tmpl` parameter is the call-site token, which `new_num_token` uses for the source position. Each invocation hands back a fresh `TK_NUM` token whose value is the current counter and whose position is the call site.

`__COUNTER__` is what makes macros like `static_assert` work without source-code-level boilerplate — the typical idiom is to paste `__COUNTER__` into a generated identifier so each expansion gets a unique name. Chibicc doesn't ship a `static_assert` macro, but the underlying primitive is in place.

The three macros sit in two categories: `__DATE__` and `__TIME__` are *fixed-at-startup* (the `time()` call is once, the strings are constant after that), and `__COUNTER__` is *fixed-at-each-expansion* (the value updates on every read). Both categories use the same `Macro->handler` field, just with different state semantics.

**Where we are.** The predefined-macro vocabulary is feature-complete for Chapter 17's table plus `__DATE__`, `__TIME__`, `__COUNTER__`. The stateful macro hook (Chapter 17 §17.5.3) gains a third user.

---

## 19.2 — Newline canonicalization

> `git checkout 74bcec5b22a601451fac9d0878003d04205abca6` — *Canonicalize newline character*

One commit. Twenty lines of new code in `tokenize.c`, plus one line in `tokenize_file` to call the new helper before `remove_backslash_newline`.

The helper:

```c
// Replaces \r or \r\n with \n.
static void canonicalize_newline(char *p) {
  int i = 0, j = 0;

  while (p[i]) {
    if (p[i] == '\r' && p[i + 1] == '\n') {
      i += 2;
      p[j++] = '\n';
    } else if (p[i] == '\r') {
      i++;
      p[j++] = '\n';
    } else {
      p[j++] = p[i++];
    }
  }

  p[j] = '\0';
}
```

In-place, two cursors: `i` reads, `j` writes. A `\r\n` pair collapses to `\n`. A bare `\r` (old-Mac line endings) becomes `\n`. Anything else passes through.

The call site in `tokenize_file`:

```c
canonicalize_newline(p);
remove_backslash_newline(p);
```

Order matters. `remove_backslash_newline` checks for the two-byte sequence `\\` followed by `\n`; if `\r\n` came through unchanged, a Windows-style file with `\\\r\n` line continuations would not be recognized. By canonicalizing first, line continuations work regardless of source-file line ending.

Two file rewrites in tokenize_file are now in place: newline canonicalization, then backslash-newline removal. The next commit (§19.3) will add a third — universal-character-name conversion. By the end of this chapter the count will be four (the §19.6 BOM strip happens before all three of these). The pattern is consistent: each is an in-place rewrite of the source buffer that runs before tokenization proper.

The `canonicalize_newline` helper is a tokenizer-pre-pass, parallel in shape to `remove_backslash_newline`, and both are *destructive* — they mutate the buffer that `read_file` malloced. Source positions reported by `error_at` after canonicalization point at the canonicalized buffer, which means errors in files with mixed line endings might report a column that's slightly different from the column an editor would show. Acceptable in practice.

**Where we are.** Source files with `\r\n` and `\r` line endings parse without diagnostics. Line-continuation backslashes work in any file regardless of how the file was saved. The pre-tokenize pass count is at two.

---

## 19.3 — Universal character names

> `git checkout c31886aa7a52fd8639e09bbdf8ac8ea854c313f6` — *Add \u and \U escape sequences*

One commit. Three changes: a new file `unicode.c` with the `encode_utf8` helper, a new `convert_universal_chars` pass in `tokenize.c` that runs as a third pre-tokenize rewrite, and a new test file `test/unicode.c`.

The C standard's *universal character name* (UCN) is the source-level escape `\uXXXX` (four hex digits) or `\UXXXXXXXX` (eight hex digits) that names a Unicode code point. UCNs can appear in string literals, character literals, and identifiers, though chibicc's implementation handles them only in string and character literals — UCNs in identifiers would require running the conversion on identifier text, not just on string-literal contents, and chibicc takes the simpler path of not supporting that.

The new `convert_universal_chars` pass walks the source buffer and replaces every UCN with the corresponding UTF-8 byte sequence, in place:

```c
static uint32_t read_universal_char(char *p, int len) {
  uint32_t c = 0;
  for (int i = 0; i < len; i++) {
    if (!isxdigit(p[i]))
      return 0;
    c = (c << 4) | from_hex(p[i]);
  }
  return c;
}

// Replace \u or \U escape sequences with corresponding UTF-8 bytes.
static void convert_universal_chars(char *p) {
  char *q = p;

  while (*p) {
    if (startswith(p, "\\u")) {
      uint32_t c = read_universal_char(p + 2, 4);
      if (c) {
        p += 6;
        q += encode_utf8(q, c);
      } else {
        *q++ = *p++;
      }
    } else if (startswith(p, "\\U")) {
      uint32_t c = read_universal_char(p + 2, 8);
      if (c) {
        p += 10;
        q += encode_utf8(q, c);
      } else {
        *q++ = *p++;
      }
    } else if (p[0] == '\\') {
      *q++ = *p++;
      *q++ = *p++;
    } else {
      *q++ = *p++;
    }
  }

  *q = '\0';
}
```

Two-cursor in-place rewrite again, like `canonicalize_newline`. The output is always shorter than or equal to the input (six or ten input bytes become up to four UTF-8 bytes). The middle `else if (p[0] == '\\')` arm passes ordinary backslash escapes (`\n`, `\t`, `\xff`, etc.) through unchanged so they're seen later by `read_escaped_char` during string-literal tokenization.

`encode_utf8`, the new helper, lives in `unicode.c`:

```c
// Encode a given character in UTF-8.
int encode_utf8(char *buf, uint32_t c) {
  if (c <= 0x7F) {
    buf[0] = c;
    return 1;
  }

  if (c <= 0x7FF) {
    buf[0] = 0b11000000 | (c >> 6);
    buf[1] = 0b10000000 | (c & 0b00111111);
    return 2;
  }

  if (c <= 0xFFFF) {
    buf[0] = 0b11100000 | (c >> 12);
    buf[1] = 0b10000000 | ((c >> 6) & 0b00111111);
    buf[2] = 0b10000000 | (c & 0b00111111);
    return 3;
  }

  buf[0] = 0b11110000 | (c >> 18);
  buf[1] = 0b10000000 | ((c >> 12) & 0b00111111);
  buf[2] = 0b10000000 | ((c >> 6) & 0b00111111);
  buf[3] = 0b10000000 | (c & 0b00111111);
  return 4;
}
```

A direct transcription of the UTF-8 encoding rule. Code points up to U+007F are one byte (the ASCII range, where UTF-8 and ASCII coincide). U+0080 through U+07FF are two bytes (`110xxxxx 10xxxxxx`). U+0800 through U+FFFF are three bytes (`1110xxxx 10xxxxxx 10xxxxxx`). U+10000 through U+10FFFF are four bytes (`11110xxx 10xxxxxx 10xxxxxx 10xxxxxx`). The function returns the byte count.

Chibicc adopts the convention used by gcc and clang: source files are UTF-8, and a UCN in the source becomes UTF-8 bytes in the parsed string-literal contents. A literal like `"α"` and a literal like `"α"` are byte-for-byte equivalent after this pre-pass — both produce a two-byte UTF-8 sequence in the token's `str` field.

The pre-tokenize sequence in `tokenize_file` is now three steps:

```c
canonicalize_newline(p);
remove_backslash_newline(p);
convert_universal_chars(p);
```

After this, the tokenizer sees a buffer that's free of `\r`, free of line continuations, and free of UCNs. It can stay ASCII-byte-based for ordinary identifier and number recognition; the multibyte handling that §19.6 will introduce for identifiers, and the §19.4–§19.5 handling for character and string literals, layer on top of this clean baseline.

The new test file `test/unicode.c` exercises the round-trip:

```c
ASSERT(0, strcmp("αβγ", "αβγ"));
ASSERT(0, strcmp("日本語", "日本語"));
ASSERT(0, strcmp("日本語", "\U000065E5\U0000672C\U00008A9E"));
ASSERT(0, strcmp("🌮", "\U0001F32E"));
```

The first two test that `\u` escapes encode the same UTF-8 bytes as a literal multibyte source character. The third tests that `\U` is the same when the code point fits in `\u`'s four hex digits. The fourth tests four-byte UTF-8 (the taco emoji is U+1F32E, outside the BMP).

`unicode.c` is added to the Makefile's `OBJS` list. It will pick up two more functions through the chapter — `decode_utf8` in §19.4, `is_ident1` and `is_ident2` in §19.6.

**Where we are.** Universal character names work in any string or character literal context. UTF-8 is the canonical source-byte representation. The pre-tokenize pass count is at three.

---

## 19.4 — Multibyte and wide character literals

> `git checkout a57c661d46d9523bed01ad1b074f7a78d9e94ca3` — *Accept multibyte character as wide character literal*
>
> `git checkout 454618cd15c2c87d9f5a6a6727e1b09a8e22a799` — *Add UTF-16 character literal*
>
> `git checkout 2dac3afece31c27bf773efbc1f30c6a67088d3b6` — *Add UTF-32 character literal*

Three commits. The first introduces `decode_utf8` and rewires `read_char_literal` to call it. The next two add `u'X'` and `U'X'` token recognition, both routed through the same `read_char_literal` with a different result type.

### `a57c661` — multibyte in wide character literal

The pre-commit `read_char_literal` was a simple ASCII-byte read:

```c
char c;
if (*p == '\\')
  c = read_escaped_char(&p, p + 1);
else
  c = *p++;
```

Read an ASCII byte (or a parsed escape), assign to a `char`. A `L'あ'` source — three UTF-8 bytes for the hiragana A — would either fail or read just the first byte, depending on luck.

The post-commit version:

```c
int c;
if (*p == '\\')
  c = read_escaped_char(&p, p + 1);
else
  c = decode_utf8(&p, p);
```

Type changes from `char` to `int` (so a 32-bit code point fits), and the non-escape path goes through the new `decode_utf8` helper. The new helper, in `unicode.c`:

```c
uint32_t decode_utf8(char **new_pos, char *p) {
  if ((unsigned char)*p < 128) {
    *new_pos = p + 1;
    return *p;
  }

  char *start = p;
  int len;
  uint32_t c;

  if ((unsigned char)*p >= 0b11110000) {
    len = 4;
    c = *p & 0b111;
  } else if ((unsigned char)*p >= 0b11100000) {
    len = 3;
    c = *p & 0b1111;
  } else if ((unsigned char)*p >= 0b11000000) {
    len = 2;
    c = *p & 0b11111;
  } else {
    error_at(start, "invalid UTF-8 sequence");
  }

  for (int i = 1; i < len; i++) {
    if ((unsigned char)p[i] >> 6 != 0b10)
      error_at(start, "invalid UTF-8 sequence");
    c = (c << 6) | (p[i] & 0b111111);
  }

  *new_pos = p + len;
  return c;
}
```

The mirror of `encode_utf8`. Read the leading byte, classify it by which high bits are set, read the continuation bytes (each starting with `10`), and assemble the code point. The `error_at` calls catch leading bytes that aren't in `[0xxxxxxx, 110xxxxx, 1110xxxx, 11110xxx]` and continuation bytes that don't start with `10`. ASCII bytes (`< 128`) are returned directly without entering the multibyte machinery.

There's also a quiet line in `tokenize` that this commit adds:

```c
// Character literal
if (*p == '\'') {
  cur = cur->next = read_char_literal(p, p);
  cur->val = (char)cur->val;
  p += cur->len;
  continue;
}
```

The `cur->val = (char)cur->val` truncates the value to a `char` *for the ordinary `'X'` form only* — narrow character literals must fit in a byte. The wide character literal `L'X'` doesn't go through this branch. The narrowing is what makes `(char)0xff == -1` for ordinary `'\xff'` and `0x000000ff` for `L'\xff'`.

The commit message has Rui's design note:

> On most Unix-like systems, wide character literal is 32-bit long and encodes a Unicode code point. On Windows, that is 16-bit long and encodes a UTF-16 code unit. Clearly, there's a portability issue here. Personally I've never used wide characters in my code as I didn't find it useful.
>
> Being said that, some header files contain wide character literal, so we need to support that so that chibicc can include such files.
>
> We assume that source files are always encoded in UTF-8.

Three things to extract. First, chibicc's `wchar_t` is 32-bit (the type `ty_int` in `read_char_literal`'s first call from the wide-char branch). That's the Linux / SysV convention. Windows would want 16-bit, but chibicc isn't a Windows compiler. Second, the motivation for adding wide character literals at all is *include compatibility* — system headers contain `L'\0'` and similar, and chibicc can't include them otherwise. Third, the source-files-are-UTF-8 assumption is now declared in writing; the entire UTF-8 decoder (`decode_utf8`) and the §19.3 UCN-to-UTF-8 conversion both rely on this.

This commit closes the Chapter 17 §17.5.3 errata candidate "`L''` ≡ `''`" — the wide-character-literal punt is gone. `L'あ'` is now `12354` (decimal), not `0` or whatever the first UTF-8 byte happens to be. `L'\xffffffff'` is `-1` after the implicit narrowing-to-`int`-then-shift.

### `454618c` — UTF-16 character literal

Two-line addition to the dispatch in `tokenize`, plus a four-line refactor of `read_char_literal`'s signature to take the result type as a parameter:

```c
static Token *read_char_literal(char *start, char *quote, Type *ty) {
  // ... unchanged body, except the final assignment is to `ty` instead of `ty_int`
  tok->ty = ty;
  return tok;
}
```

The dispatch grows a UTF-16 case:

```c
// UTF-16 character literal
if (startswith(p, "u'")) {
  cur = cur->next = read_char_literal(p, p + 1, ty_ushort);
  cur->val &= 0xffff;
  p += cur->len;
  continue;
}
```

The result type is `unsigned short` (the chibicc convention for `char16_t`). The `cur->val &= 0xffff` truncates to 16 bits. The `p + 1` argument tells `read_char_literal` to start reading after the `u`, so the function sees the leading `'` as if it were the first character.

The wide-character-literal branch is also touched — the prior `p = cur->loc + cur->len` line, which advanced relative to the start-of-token, is replaced with `p += cur->len`, which advances relative to the source pointer. The `cur->len` is set by `read_char_literal` to span the full literal including the prefix; the older form was a confused leftover that happened to work because `cur->loc == p` for the L'X' case. The fix is unrelated to UTF-16 but lands in the same commit.

A code point above U+FFFF in a `u'X'` literal silently truncates. The C standard requires this to be a constraint violation, but chibicc's implementation is permissive. The test file exercises the in-range case:

```c
ASSERT(2, sizeof(u'\0'));
ASSERT(1, u'\xffff'>>15);
ASSERT(97, u'a');
ASSERT(946, u'β');
ASSERT(12354, u'あ');
ASSERT(62307, u'🍣');     // U+1F363, truncated to 0xF363 = 62307
```

The last test pins the silent truncation: the sushi emoji's code point is 0x1F363 in full, 0xF363 after the low-16 mask.

### `2dac3af` — UTF-32 character literal

Seven-line addition to the dispatch:

```c
// UTF-32 character literal
if (startswith(p, "U'")) {
  cur = cur->next = read_char_literal(p, p + 1, ty_uint);
  p += cur->len;
  continue;
}
```

The result type is `unsigned int` (the chibicc convention for `char32_t`). No mask — 32 bits is the full Unicode range. The test cases include a U+1F363 code point that arrives unmangled, in contrast to the UTF-16 case:

```c
ASSERT(127843, U'🍣');    // 0x1F363, the full code point
```

After this commit, the four-way dispatch on character-literal prefix is complete: `'X'` is `int` (then narrowed to `char`), `L'X'` is `int`, `u'X'` is `unsigned short` (truncated to 16 bits), `U'X'` is `unsigned int`. Each goes through the same `read_char_literal` with a different `Type *`.

**Where we are.** The four character-literal forms are all functional. UTF-8 source bytes decode to Unicode code points via `decode_utf8`. `wchar_t` is 32-bit Linux convention. The Chapter 17 `L''` ≡ `''` errata is closed. The first ASCII-truncation step in `tokenize` sets the narrow-form behavior; the prefix branches set the wide-form behavior.

---

## 19.5 — UTF-8, UTF-16, UTF-32, and wide string literals

> `git checkout 57b21fe90296c867888d7c8c60d243bc254a39d7` — *Add UTF-8 string literal*
>
> `git checkout 9cabe1f204a8a6139e8b072dfd6f0a15275ad25f` — *Add UTF-16 string literal*
>
> `git checkout c467ee665de0c385170850ecc895add04b52b8a3` — *Add UTF-32 string literal*
>
> `git checkout cae061af2b65ad0962fb4b6fe3b55abe2f3a5bf8` — *Add wide string literal*
>
> `git checkout 36230e0827ca33a9b09ea5aa7b06e170fd188ca1` — *Add UTF-16 string literal initializer*
>
> `git checkout 6adba75af879d8ac2bc43a7337b02e64d10e60f1` — *Add UTF-32 string literal initializer*
>
> `git checkout e4491b811510d08f880d0f9c7553ecfd18635469` — *Define __STDC_UTF_{16,32}__ macros*

Seven commits. Four for the four prefix forms (`u8`, `u`, `U`, `L`), two for the initializer side (so `char16_t s[] = u"..."` and the UTF-32 / wide equivalents work), one for the `__STDC_UTF_*` predefined macros that announce the encoding choice.

### `57b21fe` — UTF-8 string literal

The smallest of the four prefix commits. `u8"abc"` is *byte-equivalent* to `"abc"` for ASCII input, and equivalent to a string literal for any non-ASCII input as well (since chibicc's source-file convention is already UTF-8).

The implementation: refactor `read_string_literal` to take the position-after-prefix as a parameter, then add a one-branch dispatch for `u8"`:

```c
static Token *read_string_literal(char *start, char *quote) {
  char *end = string_literal_end(quote + 1);
  char *buf = calloc(1, end - quote);
  int len = 0;

  for (char *p = quote + 1; p < end;) {
    if (*p == '\\')
      buf[len++] = read_escaped_char(&p, p + 1);
    else
      buf[len++] = *p++;
  }
  // ...
}
```

The `start` is the start of the token (including the `u8` prefix); `quote` points at the opening `"`. The two are the same for an ordinary `"foo"` and differ by two bytes for `u8"foo"`.

The dispatch:

```c
// String literal
if (*p == '"') {
  cur = cur->next = read_string_literal(p, p);
  p += cur->len;
  continue;
}

// UTF-8 string literal
if (startswith(p, "u8\"")) {
  cur = cur->next = read_string_literal(p, p + 2);
  p += cur->len;
  continue;
}
```

`u8` literals get the same `char[]` type as ordinary string literals and the same source-byte-pass-through reading. The result is identical to the unprefixed form for any input chibicc accepts. The `u8` prefix's value is mostly social — it announces "this is meant as UTF-8" — and matters more once cross-prefix concatenation enters the picture in §19.6.

### `9cabe1f` — UTF-16 string literal

The largest of the four prefix commits. UTF-16 is a *transcoding* — UTF-8 source bytes are decoded into code points and then re-encoded into 16-bit units, with surrogate pairs for code points above U+FFFF.

A new helper, `read_utf16_string_literal`:

```c
// Read a UTF-8-encoded string literal and transcode it in UTF-16.
//
// UTF-16 is yet another variable-width encoding for Unicode. Code
// points smaller than U+10000 are encoded in 2 bytes. Code points
// equal to or larger than that are encoded in 4 bytes. Each 2 bytes
// in the 4 byte sequence is called "surrogate", and a 4 byte sequence
// is called a "surrogate pair".
static Token *read_utf16_string_literal(char *start, char *quote) {
  char *end = string_literal_end(quote + 1);
  uint16_t *buf = calloc(2, end - start);
  int len = 0;

  for (char *p = quote + 1; p < end;) {
    if (*p == '\\') {
      buf[len++] = read_escaped_char(&p, p + 1);
      continue;
    }

    uint32_t c = decode_utf8(&p, p);
    if (c < 0x10000) {
      // Encode a code point in 2 bytes.
      buf[len++] = c;
    } else {
      // Encode a code point in 4 bytes.
      c -= 0x10000;
      buf[len++] = 0xd800 + ((c >> 10) & 0x3ff);
      buf[len++] = 0xdc00 + (c & 0x3ff);
    }
  }

  Token *tok = new_token(TK_STR, start, end + 1);
  tok->ty = array_of(ty_ushort, len + 1);
  tok->str = (char *)buf;
  return tok;
}
```

The buffer is a `uint16_t *` cast to `char *` for token storage. The element type on the token is `ty_ushort` (chibicc's `char16_t`), and the array length is `len + 1` for the trailing null. The surrogate-pair encoding is the standard one: subtract 0x10000, split the remaining 20 bits into two 10-bit halves, and prefix them with 0xD800 and 0xDC00 respectively.

The dispatch:

```c
// UTF-16 string literal
if (startswith(p, "u\"")) {
  cur = cur->next = read_utf16_string_literal(p, p + 1);
  p += cur->len;
  continue;
}
```

The test cases pin the byte layout:

```c
ASSERT(0, memcmp(u"日本語", "\345e,g\236\212\0\0", 8));
ASSERT(0, memcmp(u"🍣", "<\330c\337\0\0", 6));
```

The `🍣` test pins the surrogate pair: 0x1F363 - 0x10000 = 0xF363, split into 0x3D8 (high 10 bits) and 0x363 (low 10 bits), encoded as 0xD83C, 0xDF63. The bytes `<\330c\337` are 0x3C, 0xD8, 0x63, 0xDF — little-endian 0xD83C, 0xDF63. (The byte-order convention is that of the host machine; chibicc emits the data as raw bytes via the assembler's `.byte` directives.)

Escaped characters skip `decode_utf8` and go through `read_escaped_char` directly. The result is *not* re-encoded — `u"\xff"` produces a single 16-bit unit `0x00FF`, not the two-byte UTF-16 encoding of U+00FF. The C standard treats `\x` and `\u` differently in string-prefix contexts; chibicc's permissive reading matches what gcc and clang do.

### `c467ee6` — UTF-32 string literal

The simplest transcoding. Each UTF-8 source code point becomes one 32-bit unit:

```c
// Read a UTF-8-encoded string literal and transcode it in UTF-32.
//
// UTF-32 is a fixed-width encoding for Unicode. Each code point is
// encoded in 4 bytes.
static Token *read_utf32_string_literal(char *start, char *quote, Type *ty) {
  char *end = string_literal_end(quote + 1);
  uint32_t *buf = calloc(4, end - quote);
  int len = 0;

  for (char *p = quote + 1; p < end;) {
    if (*p == '\\')
      buf[len++] = read_escaped_char(&p, p + 1);
    else
      buf[len++] = decode_utf8(&p, p);
  }

  Token *tok = new_token(TK_STR, start, end + 1);
  tok->ty = array_of(ty, len + 1);
  tok->str = (char *)buf;
  return tok;
}
```

The element type is passed as a parameter (`ty`). For `U"..."` the type is `ty_uint` (chibicc's `char32_t`). For `L"..."`, the next commit, the type is `ty_int` (chibicc's `wchar_t`). Same function, two callers.

The dispatch:

```c
// UTF-32 string literal
if (startswith(p, "U\"")) {
  cur = cur->next = read_utf32_string_literal(p, p + 1, ty_uint);
  p += cur->len;
  continue;
}
```

### `cae061a` — wide string literal

Seven-line addition. The wide string literal `L"..."` reuses `read_utf32_string_literal` with a different element type:

```c
// Wide string literal
if (startswith(p, "L\"")) {
  cur = cur->next = read_utf32_string_literal(p, p + 1, ty_int);
  p += cur->len;
  continue;
}
```

`ty_int` is signed; `ty_uint` is unsigned. The byte layout is identical for any code point with the high bit clear (which is everything in valid Unicode, since the maximum code point is U+10FFFF). The signedness shows up in the test cases:

```c
ASSERT(1, U"\xffffffff"[0] >> 31);   // unsigned: high bit set, shifted right by 31 gives 1
ASSERT(-1, L"\xffffffff"[0] >> 31);  // signed: arithmetic shift gives -1
```

Same source byte sequence, different signedness, different shift result. Chibicc is consistent here — `L"X"` is signed because `wchar_t` on Linux is signed `int`, and `U"X"` is unsigned because `char32_t` is unsigned by C-spec.

After `cae061a` the dispatch in `tokenize` has, in order: ordinary string, UTF-8 string, UTF-16 string, UTF-32 string, wide string. Five branches, four helpers (UTF-8 and ordinary share `read_string_literal`, UTF-32 and wide share `read_utf32_string_literal`).

### `36230e0` — UTF-16 string literal initializer

The string literal exists as a token with the right type and bytes; the question is how it lands in the `Initializer` tree.

Pre-commit, `string_initializer` assumed bytes-to-bytes:

```c
int len = MIN(init->ty->array_len, tok->ty->array_len);
for (int i = 0; i < len; i++)
  init->children[i]->expr = new_num(tok->str[i], tok);
```

`tok->str[i]` reads one byte. For `wchar_t s[] = L"abc"`, where the destination is `int[4]` and the token's str holds three 32-bit values, this would read four bytes from the start of the buffer and produce the wrong children.

The post-commit version dispatches on the destination's element size:

```c
switch (init->ty->base->size) {
case 1: {
  char *str = tok->str;
  for (int i = 0; i < len; i++)
    init->children[i]->expr = new_num(str[i], tok);
  break;
}
case 2: {
  uint16_t *str = (uint16_t *)tok->str;
  for (int i = 0; i < len; i++)
    init->children[i]->expr = new_num(str[i], tok);
  break;
}
default:
  unreachable();
}
```

`init->ty->base->size` is 1 for `char[]`, 2 for `unsigned short[]` / `char16_t[]`. Each branch reinterprets `tok->str` as a pointer of the right element width. The default case is `unreachable()` because no other element size makes sense — yet (the next commit will add the size-4 case).

There's also a one-character fix for the `L"..."` advance-pointer line, the same kind of `cur->loc + cur->len` → `cur->len` correction the §19.4 wide-character-literal commit had:

```c
// Wide character literal
if (startswith(p, "L'")) {
  cur = cur->next = read_char_literal(p, p + 1, ty_int);
- p = cur->loc + cur->len;
+ p += cur->len;
  continue;
}
```

Same kind of leftover, same fix, different tokenizer arm. Rui packaged both into the same string-initializer commit, perhaps because `L'X'` and `L"X"` are adjacent in tokenize.c.

The test file gains:

```c
typedef unsigned short char16_t;

ASSERT(u'α', ({ char16_t x[] = u"αβ"; x[0]; }));
ASSERT(u'β', ({ char16_t x[] = u"αβ"; x[1]; }));
ASSERT(6, ({ char16_t x[] = u"αβ"; sizeof(x); }));
```

The `typedef` gives `char16_t` a definition (chibicc has no `<uchar.h>`); the rest tests that the initializer wires up correctly.

### `6adba75` — UTF-32 string literal initializer

The size-4 case. Six new lines in the switch:

```c
case 4: {
  uint32_t *str = (uint32_t *)tok->str;
  for (int i = 0; i < len; i++)
    init->children[i]->expr = new_num(str[i], tok);
  break;
}
```

After this, the switch covers element widths 1, 2, and 4 — every chibicc-supported character or string element width. The `unreachable()` default catches programmer error.

### `e4491b8` — `__STDC_UTF_{16,32}__`

Two predefined macros, two `define_macro` lines. The C11 standard's §6.10.8 says that a conforming implementation that uses UTF-16 for `char16_t` and UTF-32 for `char32_t` defines `__STDC_UTF_16__` and `__STDC_UTF_32__` to 1.

Chibicc uses both encodings (§19.5's tokenizer commits made them so), so the macros are now appropriate to define:

```c
define_macro("__STDC_UTF_16__", "1");
define_macro("__STDC_UTF_32__", "1");
```

Two-line commit. The position in `init_macros` is alphabetical-ish, in among the `__STDC_NO_*` family from Chapter 17.

**Where we are.** The four string-literal prefixes are functional, with type assignments matching gcc on Linux: `char[]` for ordinary and `u8`, `unsigned short[]` for `u`, `unsigned int[]` for `U`, `int[]` for `L`. Initializer wiring covers element widths 1, 2, and 4. The two `__STDC_UTF_*` macros announce the encoding choice. The `Initializer` tree's `string_initializer` path now dispatches on element size; the `unreachable()` macro picks up two new sites by the end of the section.

---

## 19.6 — Identifier-side Unicode and BOM

> `git checkout 0e5d250ebfd29845c8c26b0ad63379994a2b8560` — *Allow multibyte UTF-8 character in identifier*
>
> `git checkout adb8b988897758d0d4f74dcd9129bff0831634ae` — *[GNU] Accept $ as an identifier character*
>
> `git checkout 238277714ddc407f966f3c503e13a114d6a91630` — *Allow to concatenate regular string literals with L/u/U string literals*
>
> `git checkout 2b2fa25507cdc491d2b5dafb2c4b5e33158b996a` — *Skip UTF-8 BOM markers*

Four commits. The first two extend the identifier-character predicate. The third is the cross-prefix string concatenation, which lives mostly in the preprocessor. The fourth is the UTF-8 byte-order-mark skip, which is a tokenizer-pre-pass parallel to §19.2's newline canonicalization.

### `0e5d250` — UTF-8 in identifiers

The pre-commit `is_ident1` and `is_ident2` were ASCII byte predicates:

```c
static bool is_ident1(char c) {
  return ('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z') || c == '_';
}

static bool is_ident2(char c) {
  return is_ident1(c) || ('0' <= c && c <= '9');
}
```

The post-commit signatures take a `uint32_t` (a Unicode code point, not a byte) and live in `unicode.c`, so they can be called from the tokenizer's identifier-recognizer:

```c
bool is_ident1(uint32_t c);
bool is_ident2(uint32_t c);
```

The tokenizer's pre-commit identifier recognition was a do-while on bytes:

```c
if (is_ident1(*p)) {
  char *start = p;
  do {
    p++;
  } while (is_ident2(*p));
  cur = cur->next = new_token(TK_IDENT, start, p);
  continue;
}
```

The post-commit version threads through `decode_utf8`:

```c
// Read an identifier and returns the length of it.
// If p does not point to a valid identifier, 0 is returned.
static int read_ident(char *start) {
  char *p = start;
  uint32_t c = decode_utf8(&p, p);
  if (!is_ident1(c))
    return 0;

  for (;;) {
    char *q;
    c = decode_utf8(&q, p);
    if (!is_ident2(c))
      return p - start;
    p = q;
  }
}
```

And the dispatch becomes:

```c
// Identifier or keyword
int ident_len = read_ident(p);
if (ident_len) {
  cur = cur->next = new_token(TK_IDENT, p, p + ident_len);
  p += cur->len;
  continue;
}
```

The do-while loop is gone. The new function decodes one code point at a time, checks the predicate, and stops at the first non-ident-character byte boundary. The `q`-variable trick in the loop is so that a non-ident-character lookahead can be backed out without consuming any bytes: `decode_utf8(&q, p)` advances `q`, and only `p = q` commits the advance.

The new `is_ident1` is a long range-table:

```c
bool is_ident1(uint32_t c) {
  static uint32_t range[] = {
    '_', '_', 'a', 'z', 'A', 'Z',
    0x00A8, 0x00A8, 0x00AA, 0x00AA, 0x00AD, 0x00AD, 0x00AF, 0x00AF,
    0x00B2, 0x00B5, 0x00B7, 0x00BA, 0x00BC, 0x00BE, 0x00C0, 0x00D6,
    0x00D8, 0x00F6, 0x00F8, 0x00FF, 0x0100, 0x02FF, 0x0370, 0x167F,
    /* ... about 30 more pairs ... */
    -1,
  };

  return in_range(range, c);
}
```

The pairs encode `[start, end]` inclusive ranges. The terminator is `-1` (cast from `uint32_t` to a sentinel; the `in_range` helper checks `range[i] != -1`). The list of ranges is from C11 Annex D — the "characters in identifiers" Unicode-range table that the standard spells out.

The comment above `is_ident1` cites the source:

> [https://www.sigbus.info/n1570#D] C11 allows not only ASCII but some multibyte characters in certan Unicode ranges to be used in an identifier.
>
> This function returns true if a given character is acceptable as the first character of an identifier.
>
> For example, ¾ (U+00BE) is a valid identifier because characters in 0x00BE-0x00C0 are allowed, while neither ⟘ (U+27D8) nor '　' (U+3000, full-width space) are allowed because they are out of range.

The C11 Annex D ranges are deliberately conservative — they exclude punctuation, whitespace, and control characters. The choice has its quirks (¾ is a digit-shaped fraction, but it's in the "letter-like" range; ⟘ is a mathematical operator and is excluded). Chibicc inherits whatever the standard says.

`is_ident2` adds a few more ranges:

```c
bool is_ident2(uint32_t c) {
  static uint32_t range[] = {
    '0', '9', 0x0300, 0x036F, 0x1DC0, 0x1DFF, 0x20D0, 0x20FF,
    0xFE20, 0xFE2F, -1,
  };

  return is_ident1(c) || in_range(range, c);
}
```

These are the *combining marks* — characters that attach to a preceding character (accents, diacritics, etc.) and are valid in identifiers but not as the first character.

Test cases:

```c
int π = 3;
ASSERT(3, π);
ASSERT(3, ({ int あβ0¾=3; あβ0¾; }));
```

The first declares an identifier using the Greek letter pi (U+03C0). The second uses Japanese hiragana, Greek beta, an ASCII digit, and the fraction `¾` (U+00BE) in a single identifier — all four are in the C11 Annex D ranges.

### `adb8b98` — `$` in identifiers

A four-line edit. Add `'$', '$'` to both `is_ident1` and `is_ident2`'s range tables:

```c
   static uint32_t range[] = {
-    '_', '_', 'a', 'z', 'A', 'Z',
+    '_', '_', 'a', 'z', 'A', 'Z', '$', '$',
     0x00A8, 0x00A8, 0x00AA, 0x00AA, 0x00AD, 0x00AD, 0x00AF, 0x00AF,
```

```c
   static uint32_t range[] = {
-    '0', '9', 0x0300, 0x036F, 0x1DC0, 0x1DFF, 0x20D0, 0x20FF,
+    '0', '9', '$', '$', 0x0300, 0x036F, 0x1DC0, 0x1DFF, 0x20D0, 0x20FF,
     0xFE20, 0xFE2F, -1,
   };
```

The C standard reserves `$` for implementation use; gcc and clang both accept it as an identifier character (the GCC `-fdollars-in-identifiers` flag, which is on by default on most targets). Chibicc follows. The test:

```c
ASSERT(5, ({ int $$$=5; $$$; }));
```

Three dollar signs as an identifier. Legal under chibicc and gcc, illegal under strict C.

### `2382777` — cross-prefix string concatenation

The biggest of the four. Two literals like `"foo" L"bar"` are *adjacent string literals*; the C standard says they concatenate into one literal. When the prefixes are mixed — one ordinary, one wide — the ordinary one is treated as if it had the wide prefix, so the concatenation is `L"foobar"`.

Pre-commit, `join_adjacent_string_literals` (in `preprocess.c`) handled the same-prefix case only: two adjacent ordinary strings concatenated, two adjacent wide strings concatenated, but a mix produced an error or wrong codegen.

Post-commit, the function does two passes. The first pass detects adjacent-string runs that include both an ordinary and a wide prefix, and *re-tokenizes* the ordinary literals using the wide prefix's encoding. The second pass does the actual concatenation, now uniform.

The detection helper:

```c
typedef enum {
  STR_NONE, STR_UTF8, STR_UTF16, STR_UTF32, STR_WIDE,
} StringKind;

static StringKind getStringKind(Token *tok) {
  if (!strcmp(tok->loc, "u8"))
    return STR_UTF8;

  switch (tok->loc[0]) {
  case '"': return STR_NONE;
  case 'u': return STR_UTF16;
  case 'U': return STR_UTF32;
  case 'L': return STR_WIDE;
  }
  unreachable();
}
```

The first-byte switch reads the prefix off the token's source position. The `strcmp(tok->loc, "u8")` is a special case because `u` and `u8` start with the same byte. Note that `tok->loc` is not null-terminated in the usual sense — it points into the source buffer — but `strcmp` will stop at the first non-matching byte (the `"` after the prefix), so this works as a startswith check.

The first pass:

```c
for (Token *tok1 = tok; tok1->kind != TK_EOF;) {
  if (tok1->kind != TK_STR || tok1->next->kind != TK_STR) {
    tok1 = tok1->next;
    continue;
  }

  StringKind kind = getStringKind(tok1);
  Type *basety = tok1->ty->base;

  for (Token *t = tok1->next; t->kind == TK_STR; t = t->next) {
    StringKind k = getStringKind(t);
    if (kind == STR_NONE) {
      kind = k;
      basety = t->ty->base;
    } else if (k != STR_NONE && kind != k) {
      error_tok(t, "unsupported non-standard concatenation of string literals");
    }
  }

  if (basety->size > 1)
    for (Token *t = tok1; t->kind == TK_STR; t = t->next)
      if (t->ty->base->size == 1)
        *t = *tokenize_string_literal(t, basety);

  while (tok1->kind == TK_STR)
    tok1 = tok1->next;
}
```

For each maximal run of adjacent `TK_STR` tokens, classify each, find the run's "kind" (which ought to be either all-`STR_NONE` or all-of-one-wide-kind, possibly with `STR_NONE` mixed in). Two different non-`STR_NONE` kinds in the same run are an error ("unsupported non-standard concatenation"). If the run has any wide kind, re-tokenize the `STR_NONE` members using `tokenize_string_literal`, exporting the right encoding.

`tokenize_string_literal` is the new public entry point in `tokenize.c`:

```c
Token *tokenize_string_literal(Token *tok, Type *basety) {
  Token *t;
  if (basety->size == 2)
    t = read_utf16_string_literal(tok->loc, tok->loc);
  else
    t = read_utf32_string_literal(tok->loc, tok->loc, basety);
  t->next = tok->next;
  return t;
}
```

It dispatches on the target element width (2 → UTF-16, otherwise → UTF-32, with the `basety` carrying the signedness for the wide case). The token's `loc` is re-read, with the first `tok->loc` argument standing in for both `start` and `quote` because the ordinary string has no prefix. `t->next` is patched to splice the new token into the existing chain.

The second pass — the actual concatenation — is structurally identical to pre-commit, just now operating on a uniform run.

The test cases pin the rewriting:

```c
ASSERT(L'a', (L"abc" "def")[0]);
ASSERT(L'd', (L"abc" "def")[3]);

ASSERT(u'a', (u"abc" "def")[0]);
ASSERT(u'd', (u"abc" "def")[3]);

ASSERT(L'あ', ("あ" L"")[0]);
ASSERT(0343, ("\343\201\202" L"")[0]);  // first byte of 'あ' UTF-8
```

The last test case is informative: `("あ" L"")[0]` is `L'あ'` (the wide character), but `("\343\201\202" L"")[0]` is `0343` (the first byte, treated as a wide character). The reason is that `\343\201\202` is three octal escapes inside the ordinary string, so the literal contains three bytes (not the character `あ`); when the run gets re-tokenized as wide, those three bytes become three `int[]` elements, and the first one is `0343`. The two literals look equivalent because UTF-8 makes them so at the byte level, but they aren't equivalent under cross-prefix promotion.

### `2b2fa25` — UTF-8 BOM skip

A seven-line addition to `tokenize_file`, before the canonicalization step:

```c
// UTF-8 texts may start with a 3-byte "BOM" marker sequence.
// If exists, just skip them because they are useless bytes.
// (It is actually not recommended to add BOM markers to UTF-8
// texts, but it's not uncommon particularly on Windows.)
if (!memcmp(p, "\xef\xbb\xbf", 3))
  p += 3;
```

The byte order mark for UTF-8 is the three-byte sequence `EF BB BF`. UTF-16 needs a BOM because byte order matters; UTF-8 doesn't, but Windows tools sometimes write one anyway. Chibicc strips it.

The pre-tokenize pass count is now four: BOM skip, newline canonicalization, backslash-newline removal, universal-character-name conversion. All four are destructive in-place rewrites of the read-from-file buffer; together they make up the "normalize the source before tokenizing" stage.

A driver test exercises the BOM strip end-to-end:

```bash
# BOM marker
printf '\xef\xbb\xbfxyz\n' | $chibicc -E -o- - | grep -q '^xyz'
check 'BOM marker'
```

`-E` runs the preprocessor and dumps the result; the test checks that the BOM is gone and `xyz` made it through.

**Where we are.** Identifiers can contain UTF-8-encoded multibyte characters in the C11 Annex D ranges, plus `$`. Adjacent string literals across prefixes concatenate by promoting ordinary literals to the wide encoding. UTF-8 BOM markers are silently stripped. The `read_ident` helper is now the canonical identifier recognizer, replacing the inline ASCII loop.

---

## 19.7 — Designated initializers

> `git checkout c618c3b582de1d0b10b334a4f2ba6b85d5128940` — *Add array designated initializer*
>
> `git checkout 835cd24b2c4598ee784d8bfd1c0427bfa948b947` — *Allow array designators to initialize incomplete arrays*
>
> `git checkout 691c4fac1529eaf1d825ca6093800912a4df3c91` — *[GNU] Allow to omit "=" in designated initializers*
>
> `git checkout 67f5834378660abf271722a16294a634106d047e` — *Add struct designated initializer*
>
> `git checkout 31dc1dfa211ee27e74907ce3aa3986401dcedb82` — *Add union designated initializer*
>
> `git checkout 95eb5b01b30b24d68cbeb3991f65c617fc2a35cb` — *Handle struct designator for anonymous struct member*

Six commits. All in `parse.c`. The arc is small in line count (~150 lines added across the six commits) but invasive in structure — the same handful of helpers (`array_initializer1`, `array_initializer2`, `struct_initializer1`, `struct_initializer2`, `union_initializer`, `count_array_init_elements`) get reshaped multiple times, and a new helper `designation` sits at the heart of the new path.

Designated initializers are a C99 addition that lets a programmer pick which element or field a particular initializer goes to:

```c
int x[10] = {[3] = 1, [7] = 2};
struct point p = {.y = 3, .x = 5};
```

The cursor that the un-designated form advances left-to-right can now jump. Rui's commits introduce the designator parsing, then extend it to incomplete arrays, then add the GNU `=`-omission shortcut, then add struct support, then union, then anonymous-struct.

### `c618c3b` — array designated initializer

The first commit. Three new functions and one signature change.

`array_designator` reads a `[index]` designator and returns the index:

```c
// array-designator = "[" const-expr "]"
//
// C99 added the designated initializer to the language, which allows
// programmers to move the "cursor" of an initializer to any element.
// The syntax looks like this:
//
//   int x[10] = { 1, 2, [5]=3, 4, 5, 6, 7 };
//
// `[5]` moves the cursor to the 5th element, so the 5th element of x
// is set to 3. Initialization then continues forward in order, so
// 6th, 7th, 8th and 9th elements are initialized with 4, 5, 6 and 7,
// respectively. Unspecified elements (in this case, 3rd and 4th
// elements) are initialized with zero.
//
// Nesting is allowed, so the following initializer is valid:
//
//   int x[5][10] = { [5][8]=1, 2, 3 };
//
// It sets x[5][8], x[5][9] and x[6][0] to 1, 2 and 3, respectively.
static int array_designator(Token **rest, Token *tok, Type *ty) {
  Token *start = tok;
  int i = const_expr(&tok, tok->next);
  if (i >= ty->array_len)
    error_tok(start, "array designator index exceeds array bounds");
  *rest = skip(tok, "]");
  return i;
}
```

The doc comment is unusually long — Rui wrote it as a mini-tutorial because the semantics aren't trivial. The cursor-jump rule is what makes the second example `{[5][8]=1, 2, 3}` set `x[5][8]`, `x[5][9]`, `x[6][0]` (the un-designated `2` and `3` follow the designated `1` in storage order, not the index that the designator named).

`designation` is the recursive entry point that handles a single designation (which may be a chain of `[...]` designators) and the value it designates:

```c
// designation = ("[" const-expr "]")* "=" initializer
static void designation(Token **rest, Token *tok, Initializer *init) {
  if (equal(tok, "[")) {
    if (init->ty->kind != TY_ARRAY)
      error_tok(tok, "array index in non-array initializer");
    int i = array_designator(&tok, tok, init->ty);
    designation(&tok, tok, init->children[i]);
    array_initializer2(rest, tok, init, i + 1);
    return;
  }

  tok = skip(tok, "=");
  initializer2(rest, tok, init);
}
```

When the current token is `[`, parse one array designator, recurse into the corresponding child (which handles any nested designator like `[5][8]`), and *then* call `array_initializer2` with the new cursor position. That last step is the key: after the designation lands its initializer in `children[i]`, any un-designated initializers that follow (like the `2, 3` in `[5][8]=1, 2, 3`) need to fill in `children[i+1]`, `children[i+2]`, and so on. `array_initializer2` is the un-designated array-init helper, which now takes a starting cursor position so the post-designation continuation can resume from the right index.

`array_initializer2` gains an `int i` parameter:

```c
- static void array_initializer2(Token **rest, Token *tok, Initializer *init) {
+ static void array_initializer2(Token **rest, Token *tok, Initializer *init, int i) {
    if (init->is_flexible) {
      int len = count_array_init_elements(tok, init->ty);
      *init = *new_initializer(array_of(init->ty->base, len), false);
    }

-   for (int i = 0; i < init->ty->array_len && !is_end(tok); i++) {
+   for (; i < init->ty->array_len && !is_end(tok); i++) {
+     Token *start = tok;
      if (i > 0)
        tok = skip(tok, ",");
+
+     if (equal(tok, "[")) {
+       *rest = start;
+       return;
+     }
+
      initializer2(&tok, tok, init->children[i]);
    }
    *rest = tok;
  }
```

Two changes. First, the loop variable starts from the passed-in `i` rather than 0. Second, when the loop sees a `[` designator, it bails — restoring `*rest` to the comma-or-bracket position and returning without consuming the designator. The control flow then unwinds back to whichever caller is the outermost `array_initializer1`, which will see the `[` and route through its own designator handling.

`array_initializer1` (the brace form) gains its own designator handling:

```c
static void array_initializer1(Token **rest, Token *tok, Initializer *init) {
  tok = skip(tok, "{");
  bool first = true;

  if (init->is_flexible) {
    int len = count_array_init_elements(tok, init->ty);
    *init = *new_initializer(array_of(init->ty->base, len), false);
  }

  for (int i = 0; !consume_end(rest, tok); i++) {
    if (!first)
      tok = skip(tok, ",");
    first = false;

    if (equal(tok, "[")) {
      i = array_designator(&tok, tok, init->ty);
      designation(&tok, tok, init->children[i]);
      continue;
    }

    if (i < init->ty->array_len)
      initializer2(&tok, tok, init->children[i]);
    else
      tok = skip_excess_element(tok);
  }
}
```

The new `if (equal(tok, "[")) { ... }` arm reads the designator, dispatches into `designation` (which recurses into the child and handles any continuation un-designated initializers), and then `continue`s the loop. The `i` variable's reassignment makes the *next* iteration's auto-increment pick up from the designated index, so a sequence like `{1, 2, [0]=4, 5}` re-overwrites `children[0]` with 4 and then `children[1]` with 5 — see the test:

```c
ASSERT(4, ({ int x[3]={1, 2, 3, [0]=4, 5}; x[0]; }));
ASSERT(5, ({ int x[3]={1, 2, 3, [0]=4, 5}; x[1]; }));
ASSERT(3, ({ int x[3]={1, 2, 3, [0]=4, 5}; x[2]; }));
```

The first three initializers set `x[0]=1, x[1]=2, x[2]=3`. The designator `[0]=4` then jumps the cursor to index 0 and overwrites `x[0]` to 4. The next un-designated `5` advances the cursor to index 1 and overwrites `x[1]` to 5. The result is `{4, 5, 3}`. Designators *replace*, they don't merge.

There's also a `bool first` flag added to track "is this the loop's first iteration" — replacing the older `if (i > 0) tok = skip(tok, ",")` form. With designators, `i` doesn't reliably increment by one each iteration, so the flag is needed for correct comma handling.

### `835cd24` — incomplete-array sizing with designators

When an array's length is omitted (`int x[] = {1, 2, 3}`), the parser counts initializer elements to decide the length. With designators, the length should be `max(designator_index, sequential_count) + 1`:

```c
int x[] = {[10-3] = 1, 2, 3};   // length is 10 — index 7 is the largest, plus two more
```

The `count_array_init_elements` helper is rewritten to handle both forms:

```c
static int count_array_init_elements(Token *tok, Type *ty) {
  bool first = true;
  Initializer *dummy = new_initializer(ty->base, true);

  int i = 0, max = 0;

  while (!consume_end(&tok, tok)) {
    if (!first)
      tok = skip(tok, ",");
    first = false;

    if (equal(tok, "[")) {
      i = const_expr(&tok, tok->next);
      if (equal(tok, "..."))
        i = const_expr(&tok, tok->next);
      tok = skip(tok, "]");
      designation(&tok, tok, dummy);
    } else {
      initializer2(&tok, tok, dummy);
    }

    i++;
    max = MAX(max, i);
  }
  return max;
}
```

The walk maintains `i` (the running cursor) and `max` (the largest cursor seen). For a designator, `i` jumps to the designated index. For a non-designator, `i` continues from wherever it was. Each iteration ends with `max = MAX(max, i)` after the `i++`. Returned: `max`, which is the array's length.

The `if (equal(tok, "..."))` line is forward-compatible plumbing for GCC range designators (`[3 ... 7] = 1`, which initializes indices 3 through 7 to 1). Chibicc's `count_array_init_elements` skips both endpoints' const-exprs, treating the range as a single jump to the *upper* endpoint. The actual range expansion isn't supported elsewhere, so the feature is half-implemented — chibicc accepts the syntax in counting but doesn't honor it in the rest of the parser. Errata candidate: range designators in initializers are silently mishandled.

The `array_initializer1` body is also modified — the `if (init->is_flexible)` block at the top is added a *second time*, before the original block. Looking at the post-commit code:

```c
static void array_initializer1(Token **rest, Token *tok, Initializer *init) {
  tok = skip(tok, "{");

  if (init->is_flexible) {
    int len = count_array_init_elements(tok, init->ty);
    *init = *new_initializer(array_of(init->ty->base, len), false);
  }

  bool first = true;

  if (init->is_flexible) {
    int len = count_array_init_elements(tok, init->ty);
    *init = *new_initializer(array_of(init->ty->base, len), false);
  }
  // ...
}
```

The first `if (init->is_flexible)` block does the work; the second is dead code, because after the first `*init = *new_initializer(..., false)` call the `is_flexible` flag is `false`. The duplication is a small bug — the new block was meant to *replace* the old one, but the old one wasn't deleted. Errata candidate: dead-code duplicate `count_array_init_elements` call in `array_initializer1`. The duplication doesn't change behavior; it's just an extra CPU cycle that the compiler optimizer probably elides.

The test cases:

```c
ASSERT(10, ({ char x[]={[10-3]=1,2,3}; sizeof(x); }));
ASSERT(20, ({ char x[][2]={[8][1]=1,2}; sizeof(x); }));
```

The first: index 7 designates `x[7]`, then `2` and `3` go to `x[8]` and `x[9]`. Length is 10. The second: a 2D array where `[8][1]=1` designates `x[8][1]`, then `2` goes to `x[9][0]`. Outer length is 10, inner is 2 (declared), total bytes 20.

### `691c4fa` — GNU `=`-omission

A two-line edit. The `=` in `[3] = 1` becomes optional:

```c
- // designation = ("[" const-expr "]")* "=" initializer
+ // designation = ("[" const-expr "]")* "="? initializer
  static void designation(Token **rest, Token *tok, Initializer *init) {
    if (equal(tok, "[")) {
      // ...
      return;
    }

-   tok = skip(tok, "=");
+   if (equal(tok, "="))
+     tok = tok->next;
    initializer2(rest, tok, init);
  }
```

The GNU extension. `{[3] 1}` is now equivalent to `{[3] = 1}`. The test:

```c
ASSERT(7, ((int[10]){ [3] 7 })[3]);
```

The space between `[3]` and `7` is parsed as the elided `=`. The pre-commit form would have errored on the missing `=`.

### `67f5834` — struct designated initializer

The biggest of the six commits. Three changes: `struct_designator` is the new helper for `.field` parsing, `designation` learns about it, and `struct_initializer2` gains a `Member *mem` parameter.

`struct_designator`:

```c
// struct-designator = "." ident
static Member *struct_designator(Token **rest, Token *tok, Type *ty) {
  tok = skip(tok, ".");
  if (tok->kind != TK_IDENT)
    error_tok(tok, "expected a field designator");

  for (Member *mem = ty->members; mem; mem = mem->next) {
    if (mem->name->len == tok->len && !strncmp(mem->name->loc, tok->loc, tok->len)) {
      *rest = tok->next;
      return mem;
    }
  }

  error_tok(tok, "struct has no such member");
}
```

Read `.`, then an identifier, then look up the member by name in the struct's `members` list. Returns the matched `Member *`, or errors if the name isn't found.

`designation` gains two new arms:

```c
if (equal(tok, ".") && init->ty->kind == TY_STRUCT) {
  Member *mem = struct_designator(&tok, tok, init->ty);
  designation(&tok, tok, init->children[mem->idx]);
  init->expr = NULL;
  struct_initializer2(rest, tok, init, mem->next);
  return;
}

if (equal(tok, "."))
  error_tok(tok, "field name not in struct or union initializer");
```

Mirror of the array case: parse `.field`, recurse into the child for any nested designation, then call `struct_initializer2` with `mem->next` as the cursor so post-designation un-designated initializers continue from the right field.

The `init->expr = NULL` line is subtle. It clears any prior scalar-form initializer that might have been set on the struct (e.g. from `{x}` syntax that initializes a struct via a single value of compatible type). When a designation arrives, the struct-form initializer takes over and the prior scalar binding has to be discarded. The line catches a case the test cases exercise:

```c
ASSERT(0, ({ typedef struct { int a,b; } T; T x={1,2}; T y[]={x, [0].b=3}; y[0].a; }));
ASSERT(3, ({ typedef struct { int a,b; } T; T x={1,2}; T y[]={x, [0].b=3}; y[0].b; }));
```

The first initializer of `y[0]` is `x` — a struct value, which sets `y[0]`'s `expr` to copy from `x`. The second initializer is the designation `[0].b=3`, which jumps back to `y[0]` and sets `b=3`. The `init->expr = NULL` line on `y[0]` ensures the struct copy is gone after the designation; `y[0].a` is now `0` (zeroed), and `y[0].b` is `3`. Without the line, both `expr` and `b`'s child would be set, and codegen would emit both stores, with the order determining the final value.

`struct_initializer2` gains the `Member *mem` parameter, like `array_initializer2`'s `int i`:

```c
static void struct_initializer2(Token **rest, Token *tok, Initializer *init, Member *mem) {
  bool first = true;

  for (; mem && !is_end(tok); mem = mem->next) {
    Token *start = tok;

    if (!first)
      tok = skip(tok, ",");
    first = false;

    if (equal(tok, "[") || equal(tok, ".")) {
      *rest = start;
      return;
    }

    initializer2(&tok, tok, init->children[mem->idx]);
  }
  *rest = tok;
}
```

The bail-on-designator pattern, parallel to `array_initializer2`. Note the `equal(tok, "[")` check too: a struct-of-arrays might have a `[index]` designator that names an array-typed member's element rather than the struct's field. The bail propagates the designator back up to `array_initializer1` or `struct_initializer1`'s outermost handler.

`struct_initializer1` (the brace form) similarly grows a `.` arm:

```c
static void struct_initializer1(Token **rest, Token *tok, Initializer *init) {
  tok = skip(tok, "{");

  Member *mem = init->ty->members;
  bool first = true;

  while (!consume_end(rest, tok)) {
    if (!first)
      tok = skip(tok, ",");
    first = false;

    if (equal(tok, ".")) {
      mem = struct_designator(&tok, tok, init->ty);
      designation(&tok, tok, init->children[mem->idx]);
      mem = mem->next;
      continue;
    }

    if (mem) {
      initializer2(&tok, tok, init->children[mem->idx]);
      mem = mem->next;
    } else {
      tok = skip_excess_element(tok);
    }
  }
}
```

The cursor variable here is `mem` (a `Member *`) rather than `i` (an `int`). After a designator, `mem = mem->next` continues the un-designated walk from one past the designated field.

### `31dc1df` — union designated initializer

A union initializer is unusual: there's only one initializer for the whole union, and pre-commit it always set the *first* member. With designators, any member can be initialized.

The `Initializer` struct gains a `mem` field:

```c
struct Initializer {
  // ...

  // Only one member can be initialized for a union.
  // `mem` is used to clarify which member is initialized.
  Member *mem;
};
```

`designation` grows a union arm:

```c
if (equal(tok, ".") && init->ty->kind == TY_UNION) {
  Member *mem = struct_designator(&tok, tok, init->ty);
  init->mem = mem;
  designation(rest, tok, init->children[mem->idx]);
  return;
}
```

Where the struct case calls `struct_initializer2` to continue with un-designated fields after, the union case doesn't — a union has one initialized member, and that member is named by `init->mem`. After the designation, the function returns directly.

`union_initializer` itself gains the designator path and explicitly sets `init->mem`:

```c
static void union_initializer(Token **rest, Token *tok, Initializer *init) {
  // Unlike structs, union initializers take only one initializer,
  // and that initializes the first union member by default.
  // You can initialize other member using a designated initializer.
  if (equal(tok, "{") && equal(tok->next, ".")) {
    Member *mem = struct_designator(&tok, tok->next, init->ty);
    init->mem = mem;
    designation(&tok, tok, init->children[mem->idx]);
    *rest = skip(tok, "}");
    return;
  }

  init->mem = init->ty->members;

  if (equal(tok, "{")) {
    initializer2(&tok, tok->next, init->children[0]);
    consume(&tok, tok, ",");
    *rest = skip(tok, "}");
  } else {
    initializer2(rest, tok, init->children[0]);
  }
}
```

Three branches: brace-and-dot (the designated form), brace-only (the default form initializing the first member), no-brace (a single-expression initializer like `union x = 5`, also initializing the first member). All three set `init->mem`.

The lowering paths — `create_lvar_init` and `write_gvar_data` — both have to read `init->mem` rather than blindly using `ty->members`:

```c
// create_lvar_init's union case:
if (ty->kind == TY_UNION) {
  Member *mem = init->mem ? init->mem : ty->members;
  InitDesg desg2 = {desg, 0, mem};
  return create_lvar_init(init->children[mem->idx], mem->ty, &desg2, tok);
}

// write_gvar_data's union case:
if (ty->kind == TY_UNION) {
  if (!init->mem)
    return cur;
  return write_gvar_data(cur, init->children[init->mem->idx],
                         init->mem->ty, buf, offset);
}
```

The `init->mem ? init->mem : ty->members` fallback in `create_lvar_init` is for cases where the union initializer was implicit — `union { int a; char b; } u = {};`, where `init->mem` is null because no designator nor explicit member-initializer ran. In that case the first member is the default. The `write_gvar_data` version is stricter: a global union with no `init->mem` is treated as fully zero (the relocation walk returns without writing).

The test fixtures include `union { int a; char b[4]; } g50 = {.b[2]=0x12};` at file scope, and the assertion `ASSERT(0x00120000, g50.a);` pins the byte layout — the `0x12` lands at offset 2 of the four-byte storage, so reading the same bytes as an `int` (little-endian) gives `0x00120000`.

### `95eb5b0` — anonymous-struct designator

The smallest of the six. Twelve lines, in `struct_designator`. Anonymous struct members (Chapter 18 §18.6) are members with `mem->name == NULL` whose type is itself a `TY_STRUCT`; their fields flatten into the outer struct's namespace, so `outer.a` accesses an `a` that lives in the anonymous inner struct.

The designator path needs to follow that flattening:

```c
static Member *struct_designator(Token **rest, Token *tok, Type *ty) {
  Token *start = tok;
  tok = skip(tok, ".");
  if (tok->kind != TK_IDENT)
    error_tok(tok, "expected a field designator");

  for (Member *mem = ty->members; mem; mem = mem->next) {
    // Anonymous struct member
    if (mem->ty->kind == TY_STRUCT && !mem->name) {
      if (get_struct_member(mem->ty, tok)) {
        *rest = start;
        return mem;
      }
      continue;
    }

    // Regular struct member
    if (mem->name->len == tok->len && !strncmp(mem->name->loc, tok->loc, tok->len)) {
      *rest = tok->next;
      return mem;
    }
  }

  error_tok(tok, "struct has no such member");
}
```

For each member, if it's a regular named member, match by name as before. If it's an anonymous struct member, recurse into its members (via `get_struct_member`, the §18.6 helper that walks the anonymous-flattening chain). On a match, return *the anonymous member*, not the named member inside it — the cursor then does the right thing because the recursive `designation` call will descend through the anonymous member's children.

The `*rest = start` line in the anonymous-match arm is what makes the recursion work. The `start` is the position *before* the leading `.`, so when the recursive call to `designation` (from the caller) sees the same `.field` it will parse it again, this time against the anonymous inner struct's type. Each layer of anonymous nesting peels off one designator-parse-and-recurse iteration.

The test:

```c
ASSERT(1, ({ struct { struct { int a; struct { int b; }; }; int c; } x={1,2,3,.b=4,5}; x.a; }));
ASSERT(4, ({ struct { struct { int a; struct { int b; }; }; int c; } x={1,2,3,.b=4,5}; x.b; }));
ASSERT(5, ({ struct { struct { int a; struct { int b; }; }; int c; } x={1,2,3,.b=4,5}; x.c; }));
```

The struct has an outer struct with: an anonymous inner struct (containing `int a` and an inner-inner anonymous struct containing `int b`), and `int c`. The initializer is `{1,2,3,.b=4,5}`. The first three un-designated initializers fill `a`, `b`, `c` in storage order: `a=1`, `b=2`, `c=3`. The designator `.b=4` jumps to the anonymous-of-anonymous `b` and overwrites it to `4`. The next `5` continues from one-past-`b`'s position, which is the next outer member — `c`. So `c=5`.

The walking is *layered* — `struct_designator` returns the outermost anonymous member (the inner struct containing `b`), `designation` recurses into that member's `Initializer` child, which itself recurses into the inner-inner struct's child, until eventually the leaf `b`'s Initializer is set to `4`. The `*rest = start` trick lets the same `.b` token be re-parsed at each anonymous layer.

**Where we are.** Designated initializers work for arrays, structs, and unions, with nested designators (`[5][8]`, `.a.b`), the GNU `=`-omission shortcut, and anonymous-struct member designators. The `Initializer` tree gains the `mem` field (for unions) but no other shape changes. The `count_array_init_elements` helper is now designator-aware, so `int x[] = {[7] = 1}` produces an 8-element array. The `array_initializer1` helper carries a small dead-code duplication from `835cd24` that doesn't affect behavior. Range designators (`[3 ... 7]`) are partially recognized in counting but not honored in elaboration — errata candidate.

---

## Where we end the chapter

Twenty-four commits, seven sections.

| Hash | What landed |
|---|---|
| `e27417f` | `__DATE__` and `__TIME__`. Two `format_*` helpers + one `time(NULL)` call. Local-zone date and time, fixed at cc1 startup. |
| `0e77f3d` | `__COUNTER__` (GCC). Stateful builtin macro: a `static int i` returns 0, 1, 2, ... per expansion. Uses Chapter 17 §17.5.3's `Macro->handler`. |
| `74bcec5` | Newline canonicalization. New tokenizer pre-pass: `\r\n` → `\n`, bare `\r` → `\n`. Runs before backslash-newline removal. |
| `c31886a` | Universal character names. New `unicode.c` with `encode_utf8`. New tokenizer pre-pass `convert_universal_chars` rewrites `\u`/`\U` to UTF-8 bytes in place. |
| `a57c661` | Wide character literal accepts multibyte. New `decode_utf8` in `unicode.c`. `read_char_literal` returns an `int`. The narrow form post-narrows to `(char)`. Closes the Chapter 17 `L''` ≡ `''` errata. |
| `454618c` | UTF-16 character literal. New `u'X'` recognized; type `unsigned short`; truncated to 16 bits. `read_char_literal` gains a `Type *ty` parameter. |
| `2dac3af` | UTF-32 character literal. New `U'X'` recognized; type `unsigned int`. |
| `57b21fe` | UTF-8 string literal. New `u8"X"` recognized; type `char[]`; byte-equivalent to ordinary string for valid UTF-8 input. `read_string_literal` gains a `quote` parameter. |
| `9cabe1f` | UTF-16 string literal. New `read_utf16_string_literal`. UTF-8 source is decoded and re-encoded, with surrogate pairs for code points above U+FFFF. |
| `c467ee6` | UTF-32 string literal. New `read_utf32_string_literal`, parameterized by element type. |
| `cae061a` | Wide string literal. `L"X"` reuses `read_utf32_string_literal` with `ty_int`. |
| `36230e0` | UTF-16 string literal initializer. `string_initializer` dispatches on `init->ty->base->size`, with cases for size 1 and size 2. The `default: unreachable()` arm is in place. |
| `6adba75` | UTF-32 string literal initializer. Adds the size-4 case to `string_initializer`'s switch. |
| `e4491b8` | `__STDC_UTF_16__` and `__STDC_UTF_32__`. Two `define_macro` lines in `init_macros`. |
| `0e5d250` | UTF-8 in identifiers. `is_ident1` / `is_ident2` move to `unicode.c` and take `uint32_t`. New `read_ident` in `tokenize.c` decodes UTF-8 to recognize multibyte identifiers. C11 Annex D ranges. |
| `adb8b98` | `$` in identifiers (GCC). Two range-table entries added to `is_ident1` and `is_ident2`. |
| `2382777` | Cross-prefix string concatenation. New `StringKind` enum and `getStringKind` helper. New public `tokenize_string_literal` in `tokenize.c`. `join_adjacent_string_literals` runs two passes: re-tokenize ordinary literals as wide if mixed, then concatenate. |
| `2b2fa25` | UTF-8 BOM skip. Three-byte `EF BB BF` at file start is silently stripped before the existing pre-tokenize passes. |
| `c618c3b` | Array designated initializer. New `array_designator` and `designation` helpers. `array_initializer2` gains `int i`. `array_initializer1` learns the `[index]` arm. |
| `835cd24` | Incomplete array sized via designators. `count_array_init_elements` rewritten: tracks running `i` and `max(i)`. `array_initializer1` picks up a duplicate `is_flexible` block — dead code, errata. Range designator `[3 ... 7]` syntax is recognized in counting but not honored in elaboration — errata. |
| `691c4fa` | GNU `=`-omission. `designation`'s trailing `skip(tok, "=")` becomes optional. |
| `67f5834` | Struct designated initializer. New `struct_designator`. `designation` learns the `.` arm. `struct_initializer2` gains `Member *mem`. `struct_initializer1` learns `.field` parsing. The `init->expr = NULL` clears prior scalar bindings on re-designation. |
| `31dc1df` | Union designated initializer. `Initializer` gains `Member *mem`. `union_initializer` and `designation` set it. `create_lvar_init` and `write_gvar_data` read it. |
| `95eb5b0` | Anonymous-struct designator. `struct_designator` checks for anonymous members via `get_struct_member` and returns the anonymous outer member when an inner field name matches; the recursive `designation` peels one anonymous layer per call. |

Six structural moves carry forward.

The first is *the tokenizer-pre-pass family*. Through Chapter 18 the tokenizer ran one pre-pass on the source buffer (`remove_backslash_newline`). After Chapter 19 the count is four: BOM strip, newline canonicalization, backslash-newline removal, universal-character-name conversion. All four are destructive in-place rewrites. The order matters — BOM first (it has to come before any byte-position-sensitive pass), newline second (so backslash-newline can recognize Windows endings), backslash-newline third, UCN conversion fourth.

The second is *the UTF-8 baseline*. Chibicc's source-file convention is now formally UTF-8: the BOM strip handles Windows quirks, `decode_utf8` is the canonical multi-byte reader, and the four `\u`/`\U`/`u8`/`u`/`U`/`L` entry points all route source bytes through it. The two helpers `encode_utf8` and `decode_utf8` form a small two-function module in the new `unicode.c` file. `unicode.c` also holds the C11 Annex D range tables for `is_ident1` and `is_ident2`.

The third is *the four-prefix string-literal family*. Ordinary, `u8`, `u`, `U`, `L`. Each prefix has its own `read_*_string_literal` helper (with `u8` reusing the ordinary one and `L` reusing UTF-32's). The `Initializer` tree's `string_initializer` dispatches on element size. The cross-prefix concatenation handler in the preprocessor re-tokenizes ordinary literals to match a wide neighbor, so a mixed run becomes uniform before the second-pass concatenation runs. The pre-Chapter-19 `string_initializer` was a single byte-by-byte loop; the post-Chapter-19 version is a switch on `init->ty->base->size`.

The fourth is *designated initializers*. The `Initializer` tree itself doesn't change shape — children still live in a fixed-size array indexed by position — but the parser now has a recursive `designation` helper that drives cursor jumps, and three callers (`array_initializer1`, `struct_initializer1`, `union_initializer`) and three companion helpers (`array_initializer2`, `struct_initializer2`, `count_array_init_elements`) cooperate via the cursor parameter. The `Initializer` struct picks up one new field, `Member *mem`, which is read by `create_lvar_init` and `write_gvar_data` for the union case.

The fifth is *the StringKind enum and `getStringKind` helper*. These let the preprocessor's adjacent-string concatenation classify literals by prefix without re-parsing. Five kinds: NONE (ordinary), UTF8, UTF16, UTF32, WIDE. The classification is purely syntactic (looks at the token's `loc` byte), and the public `tokenize_string_literal` provides the re-tokenization path.

The sixth is *the anonymous-struct designator integration*. Chapter 18 §18.6's `get_struct_member` recursion is reused unchanged in `struct_designator`'s anonymous-match arm. The trick of returning the *outer* anonymous member while marking `*rest = start` lets the same `.field` token be re-parsed at each anonymous layer.

One errata item is *closed* by Chapter 19.

- The Chapter 17 §17.5.3 `L''` ≡ `''` punt — closed by `a57c661`. `L'X'` is now a 32-bit Unicode code point, distinct from the narrow `'X'` form.

Two new errata candidates surface in §19.7.

- The dead-code duplicate `if (init->is_flexible)` block in `array_initializer1` (introduced by `835cd24`). Behavior is unaffected; the second `count_array_init_elements` call is dead because the first sets `is_flexible` to `false`.
- Range designators `[3 ... 7]` are syntactically accepted in `count_array_init_elements` (so an incomplete-array's length is computed using the upper endpoint) but not honored in `array_initializer1` or `array_initializer2` (which would need to repeat the designation across the range). The half-implementation means `int x[] = {[3 ... 7] = 1}` declares an 8-element array but only initializes `x[3]`.

The pre-factor-before-feature count is unchanged at nine — Chapter 19 adds features by direct change rather than by laying groundwork that a later commit fills in. (The closest candidate was `read_char_literal`'s `Type *ty` parameter in `454618c`, but that change and the use of the new parameter are in the same commit; `read_string_literal`'s `quote` parameter in `57b21fe` is similarly same-commit.)

The canonicalization-at-parse-time count is unchanged at nine. None of the §19.7 commits desugars one syntactic shape into another at parse time — the designator handling threads through `Initializer` tree population without reshaping the tree. The closest candidate was the cross-prefix re-tokenization in `2382777`, which *does* change a token's contents at preprocessing time; the prose treats it as a token-level rewrite within the preprocessor rather than parse-time canonicalization, since it's in `preprocess.c` rather than `parse.c`.

The psABI conformance count is unchanged at sixteen. Chapter 19's commits don't touch the SysV AMD64 ABI surface.

Standing notes worth carrying forward.

- The hideset on Token, the Token->origin chain, the eval-quartet duplication — all unchanged.
- The cc1-vs-driver split — unchanged.
- The `Initializer` tree gains the `Member *mem` field for the union case (set by `union_initializer` or `designation`'s union arm; read by `create_lvar_init` and `write_gvar_data`'s union branches).
- The `Relocation` mechanism — unchanged. UTF-16 and UTF-32 string literals don't need relocations because their contents are constant byte arrays; `gen_data` emits them as raw `.byte` sequences via the same path as ordinary string literals.
- The local-vs-global split — stable.
- The `is_static` default in `new_gvar` — unchanged.
- The `is_definition` flag on `Obj` — unchanged.
- The `is_unsigned` flag on `Type` — used for UTF-16 (`ty_ushort`) and UTF-32 (`ty_uint`) string-literal element types, which are unsigned by C-spec; wide (`ty_int`) is signed.
- The `__va_area__` magic name — unchanged.
- The register-save-area layout — unchanged.
- The argreg integer/FP split — unchanged.
- The `Member->idx` field — unchanged. The bitfield siblings (`is_bitfield`/`bit_offset`/`bit_width`) — unchanged.
- The `is_flexible` flag — used in `count_array_init_elements`'s dummy initializer (set to `true` so the dummy can absorb nested initializers without size-checking), and in `array_initializer1`'s flexible-resize branch. The dead-code duplicate is in `array_initializer1`'s second `if (init->is_flexible)` block.
- `copy_struct_type` — unchanged.
- `MIN`/`MAX` macros — `MAX` picks up a new use in `count_array_init_elements` for the running-max-of-cursor calculation. `MIN` picks up a new use in `string_initializer` (already present from Chapter 12 §12.5; no new use this chapter).
- `is_numeric` predicate — unchanged.
- `unreachable()` macro — gains new sites in `string_initializer`'s switch (the impossible-element-size default), and in `getStringKind` (the impossible-prefix default). Two new callers in this chapter.
- Per-token line numbers — unchanged.
- GDB-debuggable output — unchanged.
- Tests are in C. `test/unicode.c` is the new test file (added in `c31886a`). Designated-initializer tests go into existing `test/initializer.c`. Cross-prefix concatenation tests go into existing `test/string.c`. Driver test for BOM marker added to `test/driver.sh`.
- The `Obj->tok` field — still no readers.
- The `Type->name_pos` field — no new uses.
- The `>>` codegen quirk — unchanged.
- The `add_type` rule for `ND_STMT_EXPR` — errata candidate, unchanged.
- The hex-escape silent truncation — errata candidate, unchanged. (UTF-16 character literal's silent truncation of code points above U+FFFF is a related but distinct case, also errata-worthy.)
- The redeclaration-in-same-scope check missing for variables/tags/typedef-names/labels/struct-members — five errata candidates, unchanged.
- `f()` and `f(void)` are accepted as identical — errata candidate, unchanged.
- Empty brace initializer — chibicc-specific extension, still in use.
- `.bss` is the third assembly section — unchanged.
- `.align` — unchanged.
- The `mov $0, %rax` for variadic FP-count — errata candidate, unchanged.
- The `fp_offset = fp * 8 + 48` non-conforming stride — errata candidate, unchanged.
- `long double` is `double` — errata candidate, unchanged.
- The default-argument-promotion gap — errata candidate, unchanged.
- Float literals inlined as integer-immediate-bit-cast — unchanged.
- The cast/compound-literal disambiguator — unchanged.
- The cast table is 10×10 — unchanged. UTF-16 (`unsigned short`) and UTF-32 (`unsigned int`) are existing cells; the four prefix forms don't add cast cells.
- Driver brittleness in `find_libpath`/`find_gcc_libpath` — unchanged.
- The link command's hardcoded distro list — unchanged.
- `Node->funcname` is gone — still gone.
- `mov %rax, %r10; call *%r10` is uniform across all calls — unchanged.
- The `StringArray` type — unchanged.
- `atexit(cleanup)` — unchanged.
- The `run_subprocess` helper — unchanged.
- Errata candidates added in Chapter 17 — `#error` doesn't print message text, `L''` ≡ `''` (closed by `a57c661`), `__va_arg_mem` divides by zero (closed by Chapter 18), `opt_S | opt_E` typo, default include paths Linux/glibc-specific. Two closed; three remain.
- Errata candidates added in Chapter 18 — None high-priority. The bitfield zero-width test exposed the missing struct-member-name redeclaration check.
- Errata candidates added in Chapter 19 — UTF-16 character-literal silent truncation of code points above U+FFFF; the dead-code duplicate `is_flexible` block in `array_initializer1`; range designators `[3 ... 7]` syntactically accepted but not honored in elaboration. Three new low-priority candidates.
- `self.py` is gone — still gone.
- Stage-2 build is end-to-end chibicc, `-Wall`-clean — still is.
- Chibicc compiles itself — still does.
- The `has_flonum` family — unchanged.
- Bitfield support is feature-complete — unchanged.
- Anonymous struct/union members flatten via recursive `get_struct_member` — now reused by `struct_designator`'s anonymous-match arm.

Forward references for Chapter 20 (commits 245–266 — the GCC-extension and small-completions arc).

- `_Generic`, `_Static_assert`, `_Alignof`, `_Alignas` are deferred to Chapter 20.
- The GCC ternary-without-middle (`a ?: b`) is deferred to Chapter 20.
- The GCC `__attribute__` parsing surface is deferred to Chapter 20.
- The `<stdarg.h>`, `<stdbool.h>`, `<stddef.h>`, `<stdalign.h>`, `<stdnoreturn.h>`, and `<float.h>` headers are unchanged in Chapter 19. Some may grow in Chapter 20.

Twenty-four commits. The chapter brings chibicc from "ASCII-only source, ASCII-only identifiers, no designated initializers" to "UTF-8 source with C11 Annex D identifiers, four string-literal prefix forms, four character-literal prefix forms, and full designated-initializer support including the GNU `=`-omission and anonymous-struct cases." The next chapter takes on the GCC-extension corner of the C language — `_Generic`, `_Static_assert`, the alignment specifiers, and the small `__attribute__` parsing surface that real C codebases assume.
