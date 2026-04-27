---
name: professional-code-design
description: Applies professional software design — clear abstractions, separation of concerns, justified design patterns, readability, and SOLID-oriented structure. Use this skill whenever writing new code, refactoring existing code, designing modules or APIs, reviewing diffs, or when the user mentions maintainability, clean architecture, engineering quality, code smells, technical debt, or SOLID principles — even when they don't say "design" explicitly. Trigger on phrases like "make this cleaner", "is this well-designed?", "refactor this", "review my code", "how should I structure...", or any non-trivial code-writing task where quality matters.
---

# Professional Code Design

The goal of this skill is to produce code that the next person to read it — possibly you in six months — can understand, change, and trust. Everything below is a means to that end. When a rule and the goal conflict, the goal wins.

## How to use this skill

When this skill triggers, hold two passes in mind:

1. **First pass — make it work, simply.** Write the most direct code that solves the actual problem in front of you. Don't reach for patterns yet. Don't add layers for hypothetical futures.
2. **Second pass — read it as a stranger.** Ask: where would a new reader get confused, misled, or stuck? Fix *those* spots. Most of the value of "good design" lives in this pass.

Do the second pass before declaring a code task complete. For reviews, the second pass *is* the work.

## The principles, and why each matters

These are not rules to mechanically apply — they're lenses to evaluate the code you just wrote.

### 1. Clear abstractions

A good abstraction names a concept that exists in the problem domain and hides detail that the caller genuinely doesn't need. A bad abstraction names a piece of *code structure* (`Manager`, `Helper`, `Util`, `Processor`) and forces the caller to understand the implementation anyway.

**Test it:** Can you describe what the abstraction does in one sentence, without using the words "and" or "or"? If not, it's probably doing two things.

**Anti-pattern:** wrapping a one-line standard library call in a function named after the call. That's not abstraction, it's renaming.

### 2. Separation of concerns

Each module/function/class should have one reason to change. Mixing concerns (e.g., HTTP parsing + business logic + database writes in one function) means a change to any one of them risks breaking the others, and tests have to set up all three to verify any one.

**Practical heuristic:** if you find yourself writing "and then" repeatedly while explaining what a function does, it's doing too much. Split at the "and then" boundaries.

**But:** don't split prematurely. Three lines of related code aren't a "concern." Splitting tiny things creates indirection without benefit. Wait until you see the seam — usually when a second caller wants part of the behavior, or a test wants to stub part of it.

### 3. Justified design patterns

Patterns (Strategy, Factory, Observer, Adapter, etc.) are answers to specific problems. Using one without that problem is cargo-culting and adds cost — more files, more indirection, more vocabulary the reader has to know.

**Before introducing a pattern, name the concrete problem it solves in *this* codebase.** "We might need to swap implementations later" is not a concrete problem; "we already have two implementations and a third is coming next sprint" is.

When a pattern *is* justified, use the canonical name in the code so readers can pattern-match. A Strategy implementation should be obviously a Strategy.

### 4. Readability

Code is read far more often than written. Optimize for the reader.

- **Names carry meaning.** A well-named variable removes the need for a comment. `elapsed_ms` beats `t` plus a comment about units. `is_eligible` beats `flag`.
- **Shape reveals intent.** Early returns for guard clauses, the happy path unindented, related lines grouped, unrelated lines separated by a blank line.
- **Comments explain *why*, not *what*.** The code already says what. Reserve comments for hidden constraints, non-obvious tradeoffs, links to bug reports or specs, or warnings about subtle invariants. If the comment restates the code, delete one of them.
- **Consistency with the surrounding code beats personal preference.** A reader holding the file in their head shouldn't have to switch styles mid-scroll.

### 5. SOLID, demystified

SOLID is useful when treated as five questions, not five commandments:

- **S — Single responsibility:** "If this changes, what changes with it?" If the answer is "several unrelated things," split it.
- **O — Open/closed:** "Can I add a new variant without editing existing, tested code?" If every new case forces you to crack open a switch in five files, the shape is wrong.
- **L — Liskov substitution:** "Can a caller use the subtype without knowing it's a subtype?" If subclasses throw on methods the base class promises, the inheritance is lying.
- **I — Interface segregation:** "Does this caller depend on methods it doesn't use?" Fat interfaces force fake implementations in tests and create false coupling.
- **D — Dependency inversion:** "Does the policy depend on the mechanism, or the other way around?" Business rules shouldn't import the database driver directly.

**Beware the SOLID trap:** mechanically applying all five to small code produces over-engineered messes — interfaces with one implementation, factories that just call constructors, abstract base classes that exist for tests. Apply SOLID where the code is actually under stress (multiple variants, multiple consumers, frequent change). Leave simple code simple.

## Tradeoffs to keep visible

Good design is full of tensions. Make them explicit rather than picking a side blindly:

- **DRY vs. coupling.** Two pieces of code that look the same may represent different concepts that happened to converge. De-duplicating them couples them forever. Wait for the third occurrence before extracting.
- **Flexibility vs. simplicity.** Every extension point is a place where the code has to keep working under variation. Pay that cost only when you have or imminently expect the variation.
- **Encapsulation vs. transparency.** Hiding too much makes debugging hard; hiding too little leaks implementation. Hide what changes, expose what's stable.
- **Layers vs. directness.** Layers buy isolation; they cost a stack trace that's harder to read and a mental model that's harder to load. Add a layer when something concrete pierces it; don't add it speculatively.

## When refactoring existing code

1. **Understand before changing.** Read the code, the tests, and at least one or two callers. Many "obvious" simplifications break invariants the original author knew about.
2. **Preserve behavior; change shape.** Refactors and behavior changes should be separate commits. Mixing them makes review impossible and bisecting useless.
3. **Lean on tests.** If the code isn't tested, get a characterization test in place *first* — even an ugly one — so you have a safety net while you reshape.
4. **Refactor toward the change you're about to make,** not toward an abstract ideal. Kent Beck's framing: "make the change easy, then make the easy change." If you're not about to make a change, the refactor probably isn't urgent.

## When reviewing code

Look in this order — the higher items dominate:

1. **Correctness and security.** Does it do what it claims? Are inputs from outside the trust boundary validated?
2. **Right shape.** Are the boundaries between modules in the right places? Will this be painful to change next quarter?
3. **Readability.** Will someone unfamiliar with the change understand it in a single pass?
4. **Local style.** Naming, ordering, idioms.

Don't lead a review with style nits when the shape is wrong — you'll spend the author's attention budget on the wrong things.

## Smells worth naming out loud

When you spot one of these, call it by name in your suggestion — it gives the author a handle to think with:

- **Primitive obsession** — passing raw strings/ints where a small named type would prevent whole classes of bugs (e.g., `UserId` vs. `int`).
- **Feature envy** — a method that mostly pokes at another object's data; usually wants to live on that object.
- **Shotgun surgery** — one logical change forces edits in many files; the concept is scattered.
- **Speculative generality** — abstractions, parameters, or hooks with no current caller. Delete them; YAGNI.
- **Long parameter list** — usually a hidden object asking to be born.
- **Boolean parameter** — `do_thing(x, True)` is unreadable at the call site; split into two functions or use a named enum.
- **Train wreck** — `a.b().c().d().e()` couples to the entire chain's shape.

## What "done" looks like

For a code-writing task, the second pass (read-as-a-stranger) has been done and the obvious cleanups applied. For a review, the comments are ordered by what matters most, and each comment either explains *why* or proposes a concrete alternative. For a design discussion, the chosen approach is paired with the alternatives that were considered and the reason this one won — so the next person can revisit the decision when conditions change.

## What this skill explicitly does *not* ask for

- Layers, interfaces, or patterns that have no concrete justification today.
- Comments that restate the code.
- Refactors bundled into feature commits.
- Abstractions whose only consumer is a test.
- Renaming for taste alone in code you don't otherwise own.

If you find yourself doing one of these in the name of "good design," stop — the goal is code that is easy to read and change, and these specifically work against that goal.
