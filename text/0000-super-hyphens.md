- Feature Name: `super_hyphens`
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Treat hyphens `-` in crate names as path separators `::` in Rust.

For example, the crate `rustc-serialize` would be imported as

```rust
extern crate rustc::serialize;
```

which is desugared to:

```rust
mod rustc {
    pub mod serialize {
        extern crate "rustc-serialize" as krate;
        pub use krate::*;
    }
}
```

The old syntax (`extern crate "foo" as foo`) will remain as-is.

# Motivation

This RFC aims to solve two issues, which at first glance seem unrelated.

## Hyphens are awkward to use

Currently, hyphens in crate names – `foo-bar` – are an ergonomic nightmare. Since hyphens are not allowed in identifiers, anyone who uses such a crate must rename it on import:

```rust
extern crate "rustc-serialize" as rustc_serialize;
```

This boilerplate confers no additional meaning, and is a common source of confusion for beginners. It is without question a Bad Thing.

However, as of January 2015 there are 589 packages with hyphens on crates.io. Any proposal will need to account for these packages, and it is not clear how a total ban on hyphenated names will play out. That we recently established a `-sys` convention for C bindings muddies the situation further.

## Crate namespaces are useful

Package namespaces are an oft-requested feature for Cargo.

One use case is the Iron web framework, which consists of multiple small packages extending a common core. Since most of these packages don't make sense outside of Iron, it would be beneficial to put them under an `iron` namespace. Other package "families", such as retep998's Windows API bindings, could make use of this feature as well.

On platforms which don't implement namespacing, users tend to overload the hyphen `-` for this purpose. For example in Python, a [quick search][1] yields countless packages prefixed with `django-`. A solution for Rust should take this existing convention into account.

[1]: https://pypi.python.org/pypi?%3Aaction=search&term=django&submit=search

# Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the language to understand, and for somebody familiar with the compiler to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used.

# Drawbacks

Why should we *not* do this?

# Alternatives

## Do nothing

## Ban hyphens altogether

## Rename the crate in `Cargo.toml`

## Use `/` or `.` for namespacing

Both of these characters have their own drawbacks:

* `/` is also a path separator. This will cause problems in filenames and URLs:

    + Currently, a package named `foo-bar` compiles to a library named `libfoo-bar.rlib` by default. Under this scheme, it is unclear what `foo/bar` would map to.

    + On `crates.io`, you can look up the versions of a package via the path `/crates/<name>/versions`. If we allow slashes in package names, then `/crates/foo/versions` will be ambiguous: does it show the versions of `foo`, or the package named `foo/versions`? We can solve this problem using an extra sigil (e.g. `/crates/foo/:versions`), but other solutions don't need such hacks at all.

* `.` is a separator in domain names.

# Unresolved questions

What parts of the design are still TBD?
