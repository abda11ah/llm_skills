---
name: perl-reduce-lines
description: Transform readable Perl code into the most compact idiomatic form possible while strictly preserving behavior, correctness, and performance characteristics. Prioritize same behavior and performance over line count reduction. Keep variable names meaningful, but remove all unused code (variables, subroutines, imports, etc.).
---

# perl-reduce-lines

Transform readable Perl code into its most compact idiomatic form without altering semantics, side effects, error handling, or performance characteristics.

## When to use

Use this skill when refactoring Perl code to be more concise and idiomatic, when reviewing Perl for "code golf" style cleanups, or when preparing a script where line count and idiomaticity matter more than verbosity. Always profile before micro-optimizing, and never trade correctness or performance for brevity.

## Instructions

1. **Preserve behavior**: never change semantics, side effects, or error handling — correctness is non-negotiable.
2. **Eliminate dead code**: remove unused subroutines, variables, statements, imports, and unreachable code.
3. **Inline helpers**: inline subroutines called only once and convert single-expression helpers to one-liners; use `do { ... }` or `({ ... })` blocks when a value is needed from multi-statement logic or where a do block is enough.
4. **Create helpers sparingly**: only when identical logic appears ≥2 times (≥5 repeated lines).
5. **Merge control flow**: combine consecutive `if` blocks testing the same condition, and prefer ternaries (`$v = $cond ? $a : $b`), defined-or (`$v //= $default`), short-circuit chains (`next unless $a && $b && $c;`), and guard clauses.
6. **Use postfix forms**: `do_something() if $condition;`, `warn "bad" if $err;`, `return unless $user && $user->{active};`.
7. **Convert loops to functional idioms**: replace `for` loops that only `push` with `map`/`grep`, and return expressions directly (e.g. `return map { chomp; $_ } @items;`).
8. **Avoid "for grep"**: prefer `EXPR for LIST` (with the condition folded into `EXPR`) over `EXPR for grep { COND } LIST` to skip the intermediate list while respecting operator precedence.
9. **Use list idioms**: list assignment and slices (`my ($a,$b,$c) = split /:/, $line;`), `@lines = <IN>;`, and `chomp(my @lines = <IN>);`.
10. **Optimize large structures**: pass large arrays/hashes by reference (`\@array`, `\%hash`) unless it breaks behavior.
11. **Leverage regex & text idioms**: `tr///` for character translation, `qr/.../` for precompiled regexes, and aggressive use of `s///` and `m//`.
12. **Slurp files safely**: `local $/; my $content = <FILE>;`, and use `-n`/`-p` for one-liners when appropriate.
13. **Reduce miscellaneous noise**: `@array` in numeric context (instead of `scalar(@array)`), autovivification (`push @{ $hash{$key} }, $value;`), short code refs (`sub { $_[0]*2 }` or `\&func`), and `//`/`||`/`&&` for defaults and guards.
14. **Avoid smartmatch**: skip `~~` for portability.
15. **Format cleanly**: keep STRICTLY one statement per line.
16. **Keep meaningful names**: preserve long, descriptive variable names; only drop names when the code is truly dead.
17. **Remove intermediate variables** : if a variable is only used once, remove it and use the expression directly. **ONLY IN THE CASE PERFORMANCE IS NOT AFFECTED AND THE CODE IS STILL READABLE. ELSE DO NOT DO IT.**


## Example

**Before:**
```perl
my @clean;
for my $line (@raw) {
    chomp $line;
    push @clean, $line if $line =~ /^\w+$/;
}
```

**After:**
```perl
chomp(my @clean = grep /^\w+$/, @raw);
```

## Constraint (non-negotiable)

All transformations must strictly preserve observable behavior.

Never use `awk`, `sed`, or `grep` from the shell — prefer pure Perl for any text filtering, substitution, or matching. Shelling out to external tools breaks portability, hides semantics, and prevents Perl-native optimization (e.g. `-n`/`-p` one-liners, `s///`, `m//`, `tr///`).

## Pitfall: Dereferencing the result of an indexer

Any expression that may yield `undef` (empty `grep`/`map`, out-of-range slice, sub returning `()`, optional chain) must not be immediately followed by `->{...}`, `->[...]`, or `->(...)` — `undef->X` dies. The original readable form handled the empty case and must be preserved.

```perl
# Unsafe (do NOT write)
my $v = (grep { /foo/ } @list)[0]{key};
my $v = (func())[0][0];

# Safe
my ($m) = EXPR;                  # bind — empty → $m=undef, $m->X still dies, so:
my $v = ($m //= default) && $m->{key};
# or:  my $v = (EXPR)[0] ? (EXPR)[0]{key} : default;   # explicit guard
```

