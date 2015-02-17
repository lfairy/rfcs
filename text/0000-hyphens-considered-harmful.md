- Feature Name: `hyphens_considered_harmful`
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Establish a convention that all crate names must be valid identifiers. Add a clear transition path by first warning maintainers in Cargo, then rejecting invalid names on crates.io.

# Motivation

Allowing hyphens in crate names -- `foo-bar` -- is an ergonomic nightmare. (Names with leading digits, like `0mq`, are also problematic but rare in practice.) Since hyphens are not allowed in identifiers, anyone who uses such a crate must rename it on import:

```rust
extern crate "rustc-serialize" as rustc_serialize;
```

This boilerplate confers no additional meaning, and is a common source of confusion for beginners. It is without question a Bad Thing.

However, as of January 2015 there are 589 packages with hyphens on crates.io. Any proposal will need to account for these packages, and it is not clear how a total ban on hyphenated names will play out. That we recently established a `-sys` convention for C bindings muddies the situation further.

Fortunately, Cargo presents us with a solution. We can already make the *package name* seen by Cargo differ from the *crate name* seen by `extern crate`. In other words, we can do this:

```toml
[package]
name = "rustc-serialize"  # Package name
version = "0.0.1"
authors = ["..."]

[lib]
name = "rustc_serialize"  # Crate name
```

and use the crate with

```rust
extern crate rustc_serialize;  // Crate name
```

all *without* mass-renaming packages on crates.io.

This fix is possible with Cargo today, but is not well known among the community. By making it a convention, we can draw attention to this solution and encourage more package authors to use it.

# Detailed design

The proposed solution is split into sections. The first section establishes the convention itself, while the remaining sections enforce this convention through Cargo and crates.io.

## The rule

This RFC proposes the following convention:

> If a package exposes a library crate, the name of that crate *must* be a valid Rust identifier. This rule ensures the crate can be imported using `extern crate foo` without renaming.
>
> In most cases, the crate name should be derived from the package name as follows:
>
> * Any hyphens `-` are replaced by underscores `_`;
> * If the name has a leading digit, prefix an extra underscore or spell out the number.
>
> For example, if I publish a package named `0mq-extras`, its library crate would be named `_0mq_extras` or `zeromq_extras`. An example `Cargo.toml` is shown below.
>
> ```toml
> [package]
> name = "0mq-extras"
> # ...
>
> [lib]
> name = "zeromq_extras"
> ```
>
> Alternatively, you can choose a package name that is a valid identifier already.

## Changes to `cargo new`

## Changes to `cargo publish`

## Changes to crates.io

# Drawbacks

Why should we *not* do this?

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Unresolved questions

What parts of the design are still TBD?
