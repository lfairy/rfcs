- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Add a fast path to `x.to_string()` for the common case that `x` is of type `&str`.

# Motivation

Currently, the idiomatic way to convert `&str` to `String` is using `.to_string()`.

However, `.to_string()` is very slow. For strings `s`, `s.to_string()` is about 2 to 5 times slower than `String::from_str(s)`. This is because `.to_string()` uses the string formatting machinery, which introduces significant overhead compared to the specialized `from_str()`.

We can close the gap by specializing the `ToString` trait for string types. But proper instance specialization would require negative bounds, which are unlikely to land before 1.0. It would be nice to have a fast `.to_string()` in the mean time.

# Detailed design

This RFC proposes the following addition to the `Display` trait:

```rust
pub trait Display {
    // ...

    /// View the value efficiently as a string slice, if possible. Most types
    /// should not need to implement this method.
    ///
    /// This method is used in the implementation of `ToString::to_string()`. If
    /// `Some` is returned, then `.to_string()` will copy the string directly
    /// instead of going through `.fmt()`.
    #[unstable(feature = "core",
               reason = "implementation detail; may be removed when negative
                         bounds land")]
    #[inline]
    fn fmt_as_str(&self) -> Option<&str> {
        None
    }
}
```

This method has custom implementations for `str`, `String`, and the smart pointer types:

```rust
impl Display for str {
    fn fmt(...) { ... }

    #[inline]
    fn fmt_as_str(&self) -> Option<&str> {
        Some(self)
    }
}

impl Display for String {
    fn fmt(...) { ... }

    #[inline]
    fn fmt_as_str(&self) -> Option<&str> {
        Some(&**self)
    }
}

impl<T> Display for Box<T> {
    fn fmt(...) { ... }

    #[inline]
    fn fmt_as_str(&self) -> Option<&str> {
        Display::fmt_as_str(&**self)
    }
}

// Similarly for Arc, Rc, Cow
```

The magic lies in the blanket `impl` for `ToString`. It branches on the result of `.fmt_as_str()`, calling the optimized `String::from_str` if possible:

```rust
impl<T: Display + ?Sized> ToString for T {
    #[inline]
    fn to_string(&self) -> String {
        match self.fmt_as_str() {
            Some(s) => String::from_str(s),
            None => format!("{}", self)
        }
    }
}
```

Since `.fmt_as_str()` is marked `#[inline]`, LLVM should optimize out the branch, leaving us with efficient code.

That's all we need to do. As `.fmt_as_str()` has a sensible default implementation, its addition should be backward compatible.

These changes are implemented on the author's [personal branch][codez]. Initial results are promising, with no trace of `std::fmt` remaining in the string case, and very minor (< 1 ns) regressions on other types.

[codez]: https://github.com/lfairy/rust/tree/faster-to-string

# Drawbacks

Adding a method to a commonly-used trait has the usual drawbacks in terms of complexity.

Note that since `.fmt_as_str()` is marked `#[unstable]`, we will be free to remove it if a better solution presents itself after 1.0.

# Alternatives

## Do nothing

As with any problem, we can choose to ignore it.

But by having the idiomatic method also remain the slowest one, we risk causing a rift in the community. Some users will stick to `.to_string()` because that's what the tutorial says, while others will keep using `String::from_str()` or `.to_owned()` for performance.

This is not a hypothetical concern: Servo recently switched to `.to_owned()`, while Cargo continues to use `.to_string()`.

# Unresolved questions

Should we allow users to implement `.fmt_as_str()`? The author leans toward no, as it's likely to be removed post-1.0.

What is the timeline for features such as negative bounds? This proposal depends on a better solution not landing within the next few months.
