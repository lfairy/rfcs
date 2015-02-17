- Feature Name: `hyphens_considered_harmful`
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Address the issue of hyphens in crate names.

# Motivation

Allowing hyphens in crate names -- `foo-bar` -- is an ergonomic nightmare. Since hyphens are not allowed in identifiers, anyone who uses such a crate must rename it on import:

```rust
extern crate "rustc-serialize" as rustc_serialize;
```

This boilerplate confers no additional meaning, and is a common source of confusion for beginners. It is without question a Bad Thing.

However, as of January 2015 there are 589 packages with hyphens on crates.io. Any proposal will need to account for these packages, and it is not clear how a total ban on hyphenated names will play out. That we recently established a `-sys` convention for C bindings muddies the situation further.

This RFC proposes a compromise. Cargo will continue to allow hyphens in package names, but the compiler will map these to underscores underneath. This will solve the primary issue with hyphens, while minimizing breakage to code that uses them.

# Detailed design

## Changes to rustc

In an `extern crate` statement, any hyphens in the crate name will be transparently mapped to underscores.

For example, this means `extern crate "apple-fritter" as fritter;` will be read as `extern crate "apple_fritter" as fritter;`.

However other contexts, such as the `#[crate_name]` annotation and `--crate-name`/`--extern` options, will no longer accept hyphenated names.

> **Rationale:** Renaming imports are very common in user code, but the other uses are rare outside of Cargo.

## Changes to Cargo

Currently, Cargo tracks at least *two* names for each package: the *package name* (used by Cargo and crates.io) and the *crate name* (used by rustc and `extern crate`). While it is possible to set the crate name explicitly, most packages do not. In this case, Cargo simply sets it equal to the package name.

Under this proposal

# Drawbacks

A little complexity, and some breakage. But the additions to rustc and Cargo are minor, and it's acceptable for breaking changes to land before 1.0.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Unresolved questions

What parts of the design are still TBD?
