# Dependency Build Failure Report

Repository: `TAPL-in-MoonBit`

After the required `.gitignore` update and the `promote` commit, `moon check`
failed before checking this repository's own packages. The errors come from
binary dependencies under `.mooncakes`.

Toolchain:

```text
moon 0.1.20260417 (8650a31 2026-04-17)
moonc v0.9.0+69d374a17 (2026-04-20)
moonrun 0.1.20260417 (8650a31 2026-04-17)
Feature flags enabled: rr_moon_pkg
```

Current manifest:

```json
"bin-deps": {
  "moonbitlang/yacc": "0.5.0",
  "moonbitlang/lex": "0.3.5"
}
```

Commands attempted:

```bash
moon check
moon add moonbitlang/yacc
moon add moonbitlang/lex
moon add --bin moonbitlang/yacc
moon add --bin moonbitlang/lex
```

Result:

- `moon check` fails with `Error: Building binary dependency moonbitlang/yacc failed`.
- `moon add --bin moonbitlang/yacc` resolves `moonbitlang/yacc@0.7.13`, but the install/build phase then fails while building the existing `moonbitlang/lex` binary dependency.
- `moon add --bin moonbitlang/lex` resolves `moonbitlang/lex@0.3.9`, but the install/build phase still fails while building `moonbitlang/yacc`.
- The `moon add` attempts did not leave a manifest diff, so there is no dependency update to commit.

Representative dependency errors:

- Transitive dependency `Yoorkin/trie` fails to parse old loop syntax:

```text
.mooncakes/moonbitlang/yacc/.mooncakes/Yoorkin/trie/trie.mbt:7:26
Parse error, unexpected token `,`, you may expect `{`.
```

- `moonbitlang/yacc` fails with missing loaded package imports:

```text
.mooncakes/moonbitlang/yacc/src/lib/ast/ast.mbt:86:20
Package "immut/list" not found in the loaded packages.
```

- `moonbitlang/yacc` also fails on old abstract-type access syntax:

```text
.mooncakes/moonbitlang/yacc/src/lib/util/small_int_set/small_int_set.mbt
This expression has type SmallIntSet, which is a abstract type and not a struct.
```

- `moonbitlang/lex` fails similarly when the `yacc` update attempt reaches the
  older `lex` binary dependency:

```text
.mooncakes/moonbitlang/lex/.mooncakes/Yoorkin/trie/trie.mbt:7:26
Parse error, unexpected token `,`, you may expect `{`.
```

Conclusion:

The current MoonBit toolchain cannot build this repository's binary dependencies
`moonbitlang/yacc` and `moonbitlang/lex` as resolved from the registry. The
failure occurs before repository-local errors or warnings can be checked.
