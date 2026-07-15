# NL Specification — Coherence Tracker (Resolved)

Archived items from [review/coherence.md](../coherence.md), resolved as of **2026-07-14** (versions 0.8.37–0.8.43).

---

## II. Language specification omissions (6 resolved)

### II-1. No `instanceof` keyword or expression

- [x] **vm.md** defines an `INSTANCEOF` opcode. **milestones.md** lists `instanceof` as an expression. But **specs.md** did not define an `instanceof` keyword or expression syntax. *(fixed 0.8.37 — specs.md § Keywords + § Other operators: `expr instanceof ClassName` → `bool`; compiler.md § Instanceof expression)*

### II-2. No complete operator precedence table

- [x] **specs.md § Operator precedence and usage** gave only partial relative precedence for `??`, `?:`, and `? :`. *(fixed 0.8.41 — specs.md § Operator precedence: full 12-level table (primary/postfix → unary → multiplicative → additive → `<=>` → relational/`instanceof` → equality → `&&` → `||` → ternary → `??`/`?:` → assignment) with associativity per level, consistent with the previously stated relative ordering of the conditional operators)*

### II-8. Interface extending interface

- [x] Can interfaces extend other interfaces? *(fixed 0.8.41 — specs.md § Interface inheritance: `interface A extends B, C` with any number of parent interfaces; implementing classes must implement all inherited methods; extended interfaces are supertypes for upcasts and `instanceof`; diamond declarations merge when signature and return type are identical, otherwise E041. vm.md: for INTERFACE modules, the `interfaces` list holds the extended interfaces and `super_class` is 0)*

### II-9. Constructor chaining (`this(…)`)

- [x] The NL spec only defined `super(args)`. *(fixed 0.8.41 — specs.md § Constructor chaining (`this(...)`): a constructor may delegate to a same-class constructor; at most one delegation call (`this` or `super`, never both) and it must be the first statement (E045); chains must be acyclic (E046); the delegating constructor is credited with the target's property initializations. compiler.md § Constructor delegation; vm.md § Constructors: `INVOKE_SPECIAL` on the sibling constructor)*

### II-11. `match` expression — `default` clause mandatory?

- [x] If no arm matches and there is no `default`, what happens? *(fixed 0.8.41 — specs.md § Match exhaustiveness: `match` must be exhaustive at compile time (E047). Enum: all cases or `default` (redundant `default` allowed as defensive arm). `bool`: both values or `default`. int/string/others: `default` required. A `match` therefore never fails at runtime. See VI-6)*

### II-16. `Self` in interface context

- [x] `Self` in interfaces was used (`Cloneable`, `ValueEquatable`) but never defined. *(fixed 0.8.41 — specs.md § `Self` in interfaces: inside an interface body, `Self` denotes the implementing class; each implementing class instantiates the signature with `Self = C` (covariant return / parameter types); resolution is compile-time, per class, like a built-in one-parameter template; inherited implementations keep the defining class's resolution; `new Self(...)` is invalid inside an interface body)*

---

## IV. VM and compiler specification gaps (5 resolved)

### IV-1. Class and method flags missing `ABSTRACT` and `FINAL`

- [x] vm.md class flags had no bits for abstract/final classes; method flags lacked abstract/final. *(fixed 0.8.42 — vm.md: class flag bits 3 `ABSTRACT`, 4 `FINAL`; method flag bits 8 `ABSTRACT`, 9 `FINAL`. VM enforcement: `NEW` on an ABSTRACT class rejected; extending a FINAL class and overriding a FINAL method rejected at link time; both bits set is a load error)*

### IV-2. Module format — no debug / line-number table

- [x] Stack traces referenced a line-number table that had no field in the module format. *(fixed 0.8.42 — vm.md § Method descriptor: `line_table_count` (u16) + `line_table` entries `{start_pc: u16, line: u32}` sorted by start_pc, each covering offsets up to the next entry; `line_table_count = 0` for stripped builds → stack trace lines reported as 0. Module format version bumped to 2)*

### IV-3. Abstract method representation in module format

- [x] Abstract methods have no body but the format always emitted code. *(fixed 0.8.42 — vm.md § Method descriptor: ABSTRACT methods have `max_locals = 0`, `max_stack = 0`, `code_length = 0`, empty code/exception/line tables; loader rejects ABSTRACT methods with code and concrete non-interface methods without code; interface method declarations encoded the same way)*

### IV-4. Conditional branch width limitation

- [x] `IF_TRUE`/`IF_FALSE` have i16 offsets with no wide variants. *(fixed 0.8.42 — vm.md § Control flow: deliberately no `IF_TRUE_W`/`IF_FALSE_W`; the compiler must emit the inverted condition jumping over a `GOTO_W` when the target is beyond ±32 KiB; pattern documented)*

### IV-7. Exception `readonly` vs VM `stackTrace` assignment

- [x] `stackTrace` was populated by the VM after construction, requiring a readonly bypass. *(fixed 0.8.42 — the VM captures the call stack **during** the base `Exception` constructor and assigns `stackTrace` before the constructor returns (frames of the exception's own constructor chain excluded), which is an ordinary readonly assignment inside `construct`; no bypass mechanism exists. specs.md § Exception class hierarchy + vm.md § Stack trace construction. Also resolves security finding SEC-17)*

---

## VI. Under-specified semantics (8 resolved)

### VI-1. Ternary operator — result type rules

- [x] "Compatible types" was undefined. *(fixed 0.8.41 — specs.md § Common result type: 5-rule algorithm — identical type; implicit conversion (numeric widening, upcast, `T`→`T|null`); `null` literal → `T|null`; nearest common ancestor; otherwise the **union** `T1|T2`. Shared by `? :`, `??`, `?:`, and lambda return deduction. Mixing unrelated types is not an error thanks to union types)*

### VI-2. Elvis and nullish coalescing — result type rules

- [x] Result type of `a ?? b` / `a ?: b` was undefined. *(fixed 0.8.41 — specs.md: `??` requires a nullable left operand; result = common result type of (left minus `null`) and right. `?:`: result = common result type of (left minus `null`) and right — a returned left value is never null since null is falsy)*

### VI-3. Union type narrowing (smart casts)

- [x] Narrowing rules were not defined. *(fixed 0.8.41 — compiler.md § Type narrowing (smart casts): flow-sensitive narrowing for **locals and parameters** on `!= null`/`== null`, `instanceof`, early exits (`return`/`throw`/`break`/`continue`/`Process.exit`), `&&`/`||` chains, ternary conditions; invalidation on reassignment; no narrowing for closure-mutated captures nor for properties (copy to a local first); insufficient narrowing → E004)*

### VI-4. Anonymous function return type deduction

- [x] Deduction rules were not specified. *(fixed 0.8.41 — specs.md § Return type deduction rules: single-expression body → expression type; block body → common result type of all `return` expressions (unions when unrelated); no return / bare `return;` → void; mixing bare and valued returns is an error; target typing takes precedence when the lambda is assigned to an explicit function type)*

### VI-5. Operator overloading — which operators, exact rules

- [x] The exhaustive list and signatures were missing. *(fixed 0.8.41 — specs.md § Overloadable operators: binary `+ - * / %` (`type operator+(const T) const`), compound `+= -= *= /= %=` (`Self`), comparisons `< > <= >=` (`bool … const`, independent) and `<=>` (`int … const`, no implicit derivation), unary `-` and `!` (const), `++`/`--` (`Self`, same method for prefix/postfix — postfix evaluates to the mutated reference, documented divergence from primitives). Instance methods only. Not overloadable: `==`/`!=` (identity; use ValueEquatable), `&&`/`||`, `=`, conditionals, `.`/`new`/cast/`instanceof`, and `[]` (moved to Planned))*

### VI-6. `match` expression — exhaustiveness

- [x] Exhaustiveness for enums/int/string was unspecified. *(fixed 0.8.41 — compile-time exhaustiveness required, E047; duplicate arms unreachable → E047; `default` must be last. See II-11. compiler.md § Match exhaustiveness; vm.md § Match expressions notes no runtime unmatched path is emitted)*

### VI-7. Multiple `catch` blocks for same exception type

- [x] Behavior of duplicate/shadowed catch clauses was unspecified. *(fixed 0.8.41 — specs.md § Exception handling: catch clauses tested in declaration order; a clause is unreachable when an earlier clause catches the same type or a supertype → E048. compiler.md § Unreachable catch clauses)*

### VI-8. `readonly` class interaction with `abstract` and `final`

- [x] Interaction rules and modifier ordering were not stated. *(fixed 0.8.41 — specs.md § Readonly: `readonly` is orthogonal; `abstract class readonly` and `final class readonly` are valid; subclass constructors assign inherited readonly properties via the `construct` chain; `abstract` + `final` mutually exclusive on classes (E049, same as methods); canonical order `[abstract | final] class [readonly] Name`)*

---

## VIII. Security-related specification gaps (10 resolved)

*See [security-audit.md](../security-audit.md) for the full audit and per-finding resolution notes.*

### VIII-1. `system.ps.Process.run(string)` — no command injection warning

- [x] *(fixed 0.8.38 — stdlib.md § system.ps.Process: command injection warning; `run(string[] args)` recommended)* *[SEC-01]*

### VIII-2. File system APIs — no path traversal warning

- [x] *(fixed 0.8.43 — stdlib.md § system.io.File: security note; no sanitization performed, validation is the caller's responsibility; `Path.normalize` + base-directory check recommended for untrusted input)* *[SEC-02]*

### VIII-3. No `tryParseInt` / `tryParseFloat` safe parsing methods

- [x] *(fixed 0.8.43 — `tryParse(string) : T|null` added to `system.Int`, `system.Float`, and `system.Bool` (naming aligned with `enum.tryFrom`); safety note on `parse`; specs.md entry point example rewritten with `tryParse` + null check)* *[SEC-05]*

### VIII-4. Integer overflow behavior undocumented in specs.md

- [x] *(fixed 0.8.43 — specs.md § Integer overflow: 64-bit two's complement, silent wrap on `+ - *`, `ArithmeticException` on division by zero, clamped `float`→`int`; comparator anti-pattern documented and all `a - b` examples replaced with `a <=> b`)* *[SEC-10]*

### VIII-5. Byte array I/O bounds checking unspecified

- [x] *(fixed 0.8.43 — stdlib.md: `read`/`write` on FileHandle and TcpStream must throw `IndexOutOfBoundsException` when `offset < 0 || length < 0 || offset + length > buffer.length()`, checked before any I/O and immune to integer overflow; `UdpSocket.receive` truncates oversized datagrams; Exceptions table updated)* *[SEC-16]*

### VIII-6. Network APIs — no TLS certificate validation requirement

- [x] *(fixed 0.8.43 — stdlib.md § system.net.Http: mandatory certificate validation for `https://` (chain, expiration, hostname); failure throws IOException; no skip-verify option specified)* *[SEC-08]*

### VIII-7. No cryptographically secure random number generator

- [x] *(fixed 0.8.43 — stdlib.md: new `system.SecureRandom` (static `nextBytes`, `nextInt`, bias-free bounded `nextInt`, OS CSPRNG, not seedable); `system.Uuid.random()` specified as UUID v4 from the CSPRNG; `system.Random` documented as unsuitable for security)* *[SEC-09]*

### VIII-9. `Regex.match` — full vs partial match unspecified

- [x] *(fixed 0.8.43 — stdlib.md § system.text.Regex: `match`/`matchFirst` are **partial match** (consistent with Grep; anchor `^…$` for full match); `escape(string)` added for literal use of user input in patterns)* *[SEC-23]*

### VIII-10. Module format — no integrity verification

- [x] *(fixed 0.8.43 — vm.md § Module integrity: integrity trailer (`hash_algo` u8 + hash), SHA-256 recommended by default, mandatory verification at load time when present, trust model documented; module format version 2. Digital signatures remain future work)* *[SEC-03]*
