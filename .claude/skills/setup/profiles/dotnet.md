# Profile: .NET

Questions to ask and defaults to offer when setting up a .NET (C#) project. Offer the default, record the user's choice.

## Architecture

Ask:

1. **Deployment style.** Single deployable (monolith), modular monolith, or microservices? If microservices, how do services communicate (HTTP, messaging) and where does each one's data live?
2. **Domain-driven design.** Is DDD used? If so, what are the bounded contexts, and are aggregates and domain events in use?
3. **Layering.** How is each deployable structured, and which direction may dependencies point?

Defaults:

- Modular monolith until a service boundary is justified.
- Layers `Domain`, `Application`, `Infrastructure` and a host project (`Api` or `Web`). Dependencies point inward: `Domain` references nothing, `Application` references `Domain`, `Infrastructure` references `Application`.
- One solution, one project per layer, a `.Tests` project per layer under test.

## Testing

Ask:

1. **Test framework.** xUnit, NUnit or MSTest?
2. **Test levels.** Which exist: unit, integration, end-to-end?
3. **Integration tests.** Real dependencies (such as a database in a container) or fakes?
4. **Naming and layout.** Where do test projects live and how are tests named?
5. **Assertions and mocking.** Which libraries, if any?

Defaults:

- xUnit. Single cases use `[Fact]`, and tables of cases use `[Theory]` with `[InlineData]`. If another framework is chosen, record its equivalents (`[Test]` and `[TestCase]` for NUnit, `[TestMethod]` and `[DataRow]` for MSTest).
- Test projects named `<Project>.Tests`, mirroring the namespace of the code under test.
- Test names read `Method_Scenario_ExpectedResult`.
- Arrange, act, assert, with one behaviour per test.
- Domain and application logic covered by fast unit tests with no I/O. Infrastructure covered by integration tests against real dependencies.
- Run all tests with `dotnet test`. Run one with `dotnet test --filter "FullyQualifiedName~<TestName>"`.

## Conventions

Ask:

1. **Target framework and language version.**
2. **Formatting and analysis.** `.editorconfig`, analyzers, warnings as errors?
3. **Branching and CI.** Branch naming, PR rules, what CI runs.

Defaults:

- Latest LTS .NET, nullable reference types enabled.
- `dotnet format` for formatting, with the rules in `.editorconfig`.
- CI runs build, format check and tests on every PR.

## Enforcement ideas

Not applied by `/setup` in v1. Note them in the conventions so they can be added later:

- Architecture tests (for example with NetArchTest) that fail the build when a layer references one it should not.
- A hook that runs `dotnet format` after edits.
- A hook or CI check that blocks changes to test files during a fix task.
