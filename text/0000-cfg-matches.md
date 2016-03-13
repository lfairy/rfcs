- Feature Name: `cfg_matches`
- Start Date: 2015-09-18
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Allow matching `#[cfg]` expressions in Cargo dependencies.

Add a `--cfg-matches` option to rustc, which evaluates a `#[cfg]` expression against the current environment.

# Motivation

Why are we doing this? What use cases does it support? What is the expected outcome?

# Detailed design

## Changes to rustc

Add a `--cfg-matches` option to rustc.

`rustc --cfg-matches EXPR` will return 0 if `#[cfg(EXPR)]` would match the current environment, or 1 otherwise.

For example, this script:

```sh
if rustc --cfg-matches 'target_os = "linux"'; then
    echo 'Linux!'
else
    echo 'Not Linux!'
fi
```

will print "Linux!" only when targeting the Linux platform.

If the expression cannot be parsed, rustc will return 2 instead.

## Changes to Cargo

Change the target dependency syntax to allow `#[cfg]` expressions.

# Drawbacks

Why should we *not* do this?

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Unresolved questions

What parts of the design are still TBD?
