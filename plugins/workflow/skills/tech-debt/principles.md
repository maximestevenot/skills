# Design principles reference

Used by the Design judge. A finding cites the principle it violates and the concrete cost — never a principle alone. When principles conflict, the project's declared architecture wins, then simplicity.

## Architecture (enforced)

If the project declares an architecture (CLAUDE.md, ADRs, boundary lint rules, or an unambiguous layout like `domain/`, `application/`, `infrastructure/`, `adapters/`, `ports/`), it is the reference and violations are **high** or **critical**.

For hexagonal / ports & adapters:
- **Dependency rule** — dependencies point inward. Domain imports nothing from application, infrastructure, frameworks, ORMs, HTTP, or env.
- **Ports owned by the core** — interfaces live in domain/application; adapters implement them. An adapter type in a port signature is a leak.
- **No business logic in adapters** — controllers, repositories, and consumers translate and delegate.
- **Mapping at the boundary** — DTOs, ORM entities, and domain models stay distinct. Similar shapes across layers are intentional, not DRY violations.
- **A port with a single adapter is legitimate** — it is the boundary, not speculative abstraction.

No declared architecture → don't impose one. Report only concrete coupling pain (a UI component calling the ORM, business rules duplicated across transports).

## Principles (judged)

- **SOLID** — SRP: a module changing for several unrelated reasons. OCP: a growing `switch`/`if` chain on a type edited for every new case. LSP: subtypes that throw "not supported" or narrow inputs. ISP: consumers forced to depend on members they never use. DIP: high-level policy instantiating low-level details directly.
- **DRY** — duplicated *knowledge* (a business rule, a validation, a constant) maintained in several places. Incidental similarity isn't duplication; apply the rule of three before recommending an extraction.
- **KISS** — indirection that doesn't pay for itself: wrappers that only forward, layers of factories for one product, generics with one instantiation, clever code where plain code reads better.
- **YAGNI** — unused options, parameters, extension points, config, or abstractions built for a future that hasn't come. Dead code belongs here.
- **Make it work, make it right, make it fast** — flag premature optimization that hurts readability (manual caching, micro-optimizations without a measured need). Never flag the absence of optimization.
- **Clean code** — names that lie or mislead, functions doing several things, flag arguments, hidden side effects, comments compensating for unclear code.
- **Consistency** — the same problem solved several ways (error handling, data fetching, validation, naming). Recommend converging on the dominant way, not a new one.
- **Least surprise** — getters with side effects, functions mutating their inputs, names contradicting behavior, async functions that sometimes return sync values, silent fallbacks.
- **Agile / boy scout** — fixes must be incremental and shippable in small slices. Mark a finding `When: boy-scout` when it is low-risk and local enough to fix the next time someone touches the file, instead of scheduling it.

## Design patterns (inspiration only)

Never report the absence of a pattern. Suggest one only as the *fix* to a concrete smell already found — e.g. Strategy for an OCP switch, Adapter for a leaking third-party API, Repository for queries scattered through services — and only when it is simpler than the alternative. Also report patterns applied where they add nothing (KISS/YAGNI).
