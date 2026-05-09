# TOC of Rui's existing Japanese book (sigbus.info/compilerbook)

Source: https://www.sigbus.info/compilerbook

This is the **structural template** for our early chapters — Rui already designed this curriculum once, and the new chibicc is a refinement of it. Note especially how he interleaves *concept* chapters between *step* (implementation) chapters.

| # | Japanese | English | Type |
|---|---|---|---|
| 1 | はじめに | Introduction | concept |
| 2 | 機械語とアセンブラ | Machine Language and Assembly | concept |
| 3 | 電卓レベルの言語の作成 | Creating a Calculator-Level Language | mixed |
| 3.1 | ステップ1: 整数1個をコンパイルする言語 | Step 1: Compile a single integer | step |
| 3.2 | ステップ2: 加減算のできるコンパイラ | Step 2: Addition and subtraction | step |
| 3.3 | ステップ3: トークナイザを導入 | Step 3: Introduce a tokenizer | step |
| 3.4 | ステップ4: エラーメッセージを改良 | Step 4: Improve error messages | step |
| 3.5 | 文法の記述方法と再帰下降構文解析 | Grammar notation & recursive-descent parsing | concept |
| (3.6+ — implied; calculator continues with `*` `/` `()` and unary, comparison ops) | | | |
| 4 | 分割コンパイルとリンク | Separate compilation and linking | concept |
| 4.x | ステップ8: ファイル分割とMakefileの変更 | Step 8: File splitting and Makefile changes | step |
| 5 | 関数とローカル変数 | Functions and local variables | mixed |
| 5.1 | ステップ9: 1文字のローカル変数 | Step 9: Single-character local variables | step |
| 5.2 | ステップ10: 複数文字のローカル変数 | Step 10: Multi-char local variables | step |
| 5.3 | ステップ11: return文 | Step 11: Return statement | step |
| 5.4 | 1973年のCコンパイラ | The 1973 C compiler (historical interlude) | concept |
| 5.5 | ステップ12: 制御構文を足す | Step 12: Add control structures | step |
| 5.6 | ステップ13: ブロック | Step 13: Blocks | step |
| 5.7 | ステップ14: 関数の呼び出しに対応する | Step 14: Function calls | step |
| 5.8 | ステップ15: 関数の定義に対応する | Step 15: Function definitions | step |
| 5.9 | バイナリレベルのインターフェイス | Binary-level interface (calling conventions) | concept |
| 6 | コンピュータにおける整数の表現 | Integer representation in computers | concept |
| 7 | ポインタと文字列リテラル | Pointers and string literals | mixed |
| 7.1–7.12 | ステップ16–28 | Steps 16–28: ptrs, int kw, ptr type, ptr arith, sizeof, arrays, [], globals, char, string lit, file input, comments, tests in C | step |
| 8 | プログラムの実行イメージと初期化式 | Program execution image & initializer expressions | concept |
| 9 | ステップ29以降 | Step 29 and beyond | **`[要加筆]` — TO BE WRITTEN** |
| 10 | スタティックリンクとダイナミックリンク | Static and dynamic linking | concept |
| 11 | Cの型の構文 | C type syntax (declaration reading) | concept |
| 12 | おわりに | Conclusion | — |
| A1 | x86-64命令セット チートシート | x86-64 cheatsheet | appendix |
| A2 | Gitによるバージョン管理 | Version control with Git | appendix |
| A3 | Dockerを使った開発環境の作成 | Docker dev env setup | appendix |

## Observations

1. **Step granularity matches commits.** Rui's "Step N" labels are basically commit titles. The early Japanese steps map cleanly to the early `main` commits of the new chibicc. (See `commits/chapter-mapping.md`.)
2. **Concept chapters punctuate the steps.** Roughly every 5–10 steps, Rui pauses for a non-implementation chapter that explains the next batch of background.
3. **Concept chapters are "just-in-time."** Integer representation comes right *before* readers need it (for sign extension during pointer/array work). Linking comes right *before* the linking chapter would matter. We should preserve this just-in-time ordering.
4. **The book is incomplete past step ~28.** Everything in chibicc beyond that point — typedef, enum, switch, initializers, extern/static, variadic, floats, full preprocessor, _Generic, atomics, VLA, alloca, bitfields, self-host — is **uncharted territory** for the curriculum. We are designing the second half of the book from scratch (with the commits as our guide).
5. **The new chibicc rewrite added a real preprocessor and full self-hosting.** The old chibicc the Japanese book documents did *not* self-host with its own preprocessor. So Chapter 9-onwards ("Step 29 and beyond") will be considerably more ambitious than the Japanese book ever was.
