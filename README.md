# Proposal: HostEnsureCanCompileStrings Passthru

## Status

Champion(s): mikesamuel <br>
Author(s): mikesamuel <br>
Stage: 0

## Motivation

> ### `eval` is Evil
>
> The `eval` function is the most misused feature of JavaScript. Avoid it.
>
> -- <cite>Douglas Crockford, "JavaScript: The Good Parts"</cite>

`eval` and its friend `new Function` are problematic because, too
often, an attacker can turn it against the application.

Most code avoids `eval`, but JS programs are no longer small, and
self-contained as they were when Crock wrote that.

If one module uses `eval`, even if it's as simple as
[`Function('return this')()`][core-js-example] to get a handle to the
global object then `eval` has to work.

This prevents the use of security measures like:

*  [Content-Security-Policy](https://csp.withgoogle.com/docs/index.html)
*  `node --disallow_code_generation_from_strings`

which turn off `eval` globally.

As JavaScript programs get larger, the chance that no part of it needs `eval` or `Function()`
to operate gets smaller.

----

It is difficult in JavaScript for a code reviewer to determine that
code never uses these operators.  For example, the below can when `x` is constructor.

```js
({})[x][x](y)()

// ({})[x]       === Object
// ({})[x][x]    === Function
// ({})[x][x](y)  ~  Function(y)
```

So code review and developer training are unlikely to prevent abuse of these
operators.

----

This aims to solve the problem by providing more context to host environments so that they
can make finer-grained trust decisions.


## Use cases

The [Trusted Types proposal][TT] aims to allow JavaScript development to scale securely.

It makes it easy to separate:

*  the decisions to trust a chunk of code to load.
*  the check that an input to a sensitive operator like `eval` is trustworthy.

This allows trust decisions tto be made where the maximum context is
available and allows these decisions to be concentrated in small
amounts of thoroughly reviewed code

The checks can also be moved into host code so that they reliably happen before
irrevocable actions like code loading complete.

Specifically, Trusted Types would like to require that the code portion of the
inputs to %Function% and %eval% are [*TrustedScript*][TrustedScript].


## Possible spec language

You can browse the [ecmarkup output][] or browse the [source][].


[core-js-example]: https://github.com/zloirock/core-js/blob/2a005abe68520248d4431cab70d86e40b55d6e98/packages/core-js/internals/global.js#L5
[TT]: https://wicg.github.io/trusted-types/dist/spec/
[TrustedScript]: https://wicg.github.io/trusted-types/dist/spec/#typedef-trustedscript
[ecmarkup output]: https://mikesamuel.github.io/proposal-hostensurecancompilestrings-passthru/
[source]: https://github.com/mikesamuel/proposal-hostensurecancompilestrings-passthru/blob/master/spec.emu
