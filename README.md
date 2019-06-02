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

This allows trust decisions to be made where the maximum context is
available and allows these decisions to be concentrated in small
amounts of thoroughly reviewed code

The checks can also be moved into host code so that they reliably happen before
irrevocable actions like code loading complete.

Specifically, Trusted Types would like to require that the code portions of the
inputs to %Function% and %eval% are [*TrustedScript*][TrustedScript].


## What are code portions

`eval(x)` treats `x` as code.

`new Function(x)` also treates `x` as code.

The parameter list portion of a function can also be code since *FormalParameterList* elements may contain arbitrary default value expressions.

For example, the following function has a blank body, but still has a side effect.

```js
(new Function('a = alert(1)', ''))()
```


## Host callout should be a two-way street

Trusted Types defines a [default policy][] which
is invoked when a string reaches a sink to make it easier to migrate applications that pass around strings.

This policy is invoked when a string value reaches a sink, so any default policy's
[`createScript` callback][createScript callback] is invoked on `eval(myString)`.

If the callback throws an error, then `eval` should be blocked.
But if the callback returns a value, *result*, then ToString(*result*) should be used as the source text to parse.

Being able to adjust the code that runs provides the maximum flexibility when dealing with a thorny legacy module
that might otherwise prevent the entire application from running with XSS protections enabled.

To enable that, this proposal adjusts `eval` and `new Function` to expect return values from the host callout and
to use those in place of the inputs.


## Possible spec language

You can browse the [ecmarkup output][] or browse the [source][].


[core-js-example]: https://github.com/zloirock/core-js/blob/2a005abe68520248d4431cab70d86e40b55d6e98/packages/core-js/internals/global.js#L5
[TT]: https://wicg.github.io/trusted-types/dist/spec/
[TrustedScript]: https://wicg.github.io/trusted-types/dist/spec/#typedef-trustedscript
[ecmarkup output]: https://mikesamuel.github.io/proposal-hostensurecancompilestrings-passthru/
[source]: https://github.com/mikesamuel/proposal-hostensurecancompilestrings-passthru/blob/master/spec.emu
[default policy]: https://wicg.github.io/trusted-types/dist/spec/#default-policy-hdr
[createScript callback]: https://wicg.github.io/trusted-types/dist/spec/#callbackdef-createscriptcallback
