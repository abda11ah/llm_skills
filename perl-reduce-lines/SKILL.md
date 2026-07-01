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
3. **Inline helpers**: inline subroutines called only once and convert single-expression helpers to one-liners; use `do { ... }` blocks when a value is needed from multi-statement logic.
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
15. **Format cleanly**: keep one statement per line, and remove unnecessary parentheses and final semicolons only when it doesn't hurt readability.
16. **Keep meaningful names**: preserve long, descriptive variable names; only drop names when the code is truly dead.

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
