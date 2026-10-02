# Type-test matcher and assertion failures

Use this file when a `.tst.*` assertion is failing, the matcher direction looks wrong, or a directive is unexpectedly being treated as ordinary comment text. The authoritative matcher and directive declarations are in `tstyche/types` and at https://tstyche.org/reference/expect-api.

## Matcher direction

- `toBe` tests structural equality, including inference markers like `NoInfer`. It is not a substitute for assignability.
- `toBeAssignableFrom(source)` succeeds when `source` is assignable to the `expect<S>()` input. Put the public contract as `<S>` and the candidate as the argument.
- `toBeAssignableTo(target)` succeeds when the `expect<S>()` input is assignable to `target`. Use this when the asserted value is the candidate and the contract type is the question.
- Swap the matcher before swapping the assertion. Rewrite `expect<T>().type.toBeAssignableFrom(value)` as `expect(value).type.toBeAssignableTo<T>()` only when that is the clearer direction.

## Reduced surface failures

- `rejectAnyType` (default `true`) and `rejectNeverType` (default `true`) reject inferred `any` and `never`. An expression call that infers `any` will fail even when the literal source is fine. Disable the protection intentionally rather than recasting.
- Explicit `any` or `never` targets remain valid when the test is deliberate; the inference guard is for the source side, not the target side.
- `.type` is required before any matcher. A bare `expect<T>().toBe<U>()` is invalid; the matcher chain does not interpret a bare matcher.

## Ability matchers

- `toAcceptProps` requires a `.tsx` test, the `jsx` compiler option, and that the component type actually accepts JSX attributes. Passing an object literal only works when the component is typed as a JSX component; functions with object props do not implicitly accept `key`.
- `toBeApplicable` checks a decorator applied to a class or class member. Standard decorator examples use `DecoratorContext` without `experimentalDecorators`; enable legacy decorator options only when the library under test uses legacy decorators.
- `toBeCallableWith` and `toBeConstructableWith` accept argument lists as runtime expressions. The expression is analyzed but not executed; runtime setup belongs in a unit test.
- `toBeInstantiableWith` receives a single tuple of generic arguments. `_` is `never` and fills required generics.
- `toHaveProperty` checks property key existence with `string | number | symbol`. It does not validate the property's type; pair with a `toBe` on the resulting type if the type matters.
- `toRaiseError` is deprecated. Keep it only for compatibility or migration work; the ability matcher that expresses the intended invalid operation is the preferred form.

## Directive scope

- A directive applies to the next assertion, helper, or `describe` block when placed immediately above it. Place it below its target and the gate does not fire.
- A file-level directive must be the first non-comment content of the file. A file-level directive placed after an import is treated as ordinary comment text.
- `// @tstyche if { target: ">=5.7" }` scopes a single assertion. Use separate `if` directives for distinct version-gated assertions rather than wrapping a whole `describe` block.
- `// @tstyche fixme` requires a failing child. A `fixme`-marked helper that contains only passing assertions is itself treated as failing.

## `@ts-expect-error` and `checkSuppressedErrors`

- `checkSuppressedErrors` defaults to `true`. The directive text after the directive line is matched against the suppressed diagnostic.
- `...` truncates an unstable portion of the expected message. Append `!` to the directive line to suppress a directive intentionally rather than removing it.
- A missing error reports as a diagnostic. Always run the affected assertion before declaring the directive correct.
- A directive that hides an unrelated missing symbol or import is misuse; prefer a `tstyche` import or a focused assertion rather than broadening the directive.

## Inference traps

- `expect(expr)` widens the expression to whatever the language service infers. If the contract is a literal, use `expect<Type>()` instead.
- `NoInfer<T>` is structural. A `toBe` comparison succeeds when both sides carry the inference marker; mismatched `NoInfer` placement produces asymmetric assertion results.
- `exactOptionalPropertyTypes` changes the optional-vs-undefined relationship. Tests that pass without it may fail with it; reflect that in the dedicated test TSConfig only when the package under test enforces it.
- Generic inference defaults are part of the call signature. `expect<Generic<DefaultArg>>()` is different from `expect<Generic>()` when the default is what the contract promises.
