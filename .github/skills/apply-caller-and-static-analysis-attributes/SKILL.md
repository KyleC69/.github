# Apply caller info and static analysis attributes

Use this skill when writing or refactoring C# code that should preserve caller context in logs/exceptions or improve nullability/data-flow analysis for the compiler and analyzers.

## When to use this skill

- A method should capture caller details without forcing each caller to pass them manually.
- API contracts are not fully expressed to nullable/static analysis tools.
- You need to reduce false-positive nullability warnings while keeping runtime behavior unchanged.

## Caller information attributes

Use optional parameters decorated with caller info attributes from `System.Runtime.CompilerServices`:

- `[CallerMemberName]`
- `[CallerFilePath]`
- `[CallerLineNumber]`
- `[CallerArgumentExpression]` (C# 10+ and .NET 6+ unless the attribute is polyfilled)

Guidelines:

1. Add caller info only where it has clear diagnostic value (logging, guard helpers, telemetry, exception helpers).
2. Use the expected parameter shapes: `string` for member name, file path, and argument expression; `int` for line number.
3. Keep caller info parameters optional and defaulted so existing call sites do not change (`= ""` for `string`, `= 0` for `int`).
4. Avoid exposing full file paths in user-facing error messages unless explicitly intended.
5. Prefer storing caller info for structured diagnostics rather than building brittle string formats.

## Static analysis attributes

Use attributes from `System.Diagnostics.CodeAnalysis` to describe postconditions and flow:

- Null-state contracts: `[NotNull]`, `[MaybeNull]`, `[AllowNull]`, `[DisallowNull]`
- Conditional null-state: `[NotNullWhen]`, `[MaybeNullWhen]`, `[NotNullIfNotNull]`
- Member state: `[MemberNotNull]`, `[MemberNotNullWhen]`
- Member initialization contracts: `[SetsRequiredMembers]` on constructors that fully initialize all `required` members
- Control-flow intent: `[DoesNotReturn]`, `[DoesNotReturnIf]`

Guidelines:

1. Choose the narrowest attribute that matches real runtime behavior.
2. Apply attributes to public/internal APIs where they clarify intent for callers.
3. Do not use attributes to silence warnings unless behavior actually guarantees the contract.
4. Prefer contract attributes over suppression pragmas when possible.

## Combined pattern

For validation helpers, combine caller expression capture with static analysis contracts:

- Capture the failing expression with `[CallerArgumentExpression]`.
- Use `[NotNull]`/`[NotNullWhen]` or `[DoesNotReturnIf]` to express analyzer-visible guarantees.

## Validation checklist

- Caller info parameters are optional and non-breaking.
- Attributes reflect true behavior in all branches.
- No sensitive caller file path leakage.
- Nullable warnings are reduced by improved contracts, not suppressed diagnostics.
