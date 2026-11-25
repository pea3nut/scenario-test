# Case Design Quality

Use this guide when generating cases from requirements or substantially revising them. Apply the techniques relevant to the risk of the behavior; a simple case does not need every technique. Preserve the user's business rules and the project's existing executor.

## Describe the business relationship

Write scenarios around the user's action and observable outcome. Prefer relationships such as "the exported report preserves the selected date range" or "repeating the same operation does not grant additional benefits".

Use concrete business examples when they distinguish behavior. Put request formatting, selectors, database paths, and timing machinery in executor configuration or project execution notes. Do not turn an undefined policy into an expected result just to make a case executable.

## Choose data that exposes plausible mistakes

A single default input may accidentally agree with a hard-coded implementation. For data-sensitive behavior, select a small set of fixed examples that would expose likely mistakes: a normal input, a meaningful non-default input, and relevant boundaries.

For example, checking a report with only a 30-day range would not reveal an implementation that always exports 30 days. Separate scenarios for a short range and a range longer than 30 days make that defect observable:

```gherkin
Feature: Reports preserve the selected date range

  @P1
  Scenario: Export a short date range
    Given the user has selected a 7-day date range
    When the user exports the report
    Then the report's start and end dates exactly match the selected dates

  @P1
  Scenario: Export a range longer than the default
    Given the user has selected a 61-day date range
    When the user exports the report
    Then the report's start and end dates exactly match the selected dates
```

The exact durations are fixtures, not universal requirements. Choose representative values for the actual business contract. If one scenario instead declares several required variants, make all variants explicit in its execution notes; selecting one variant is not evidence that all were exercised. This guidance does not add automatic parameter expansion or change the executor's reporting format.

Generated data can explore additional valid inputs. Keep fixed counterexamples, and record the actual generated inputs, seed, and replay information supported by the chosen tool. Do not require a new property-testing framework merely to add scenarios.

## Keep expected results independent

Derive expected values from the business contract, a captured input, or another independent source. In the report example, preserve the user's selected dates before execution and compare the exported dates with those values.

Do not call the implementation's date-range helper to calculate the expectation, or copy its calculation into the test. Both can reproduce the same defect. Use assertions precise enough to reject the suspected error; checking that the report merely contains some dates would not establish that they are the selected dates.

## Make negative cases fail for the intended reason

Change the condition being tested while keeping unrelated conditions valid. For a response-ownership check, return a successful, well-formed response with valid fields but a different resource ID. Also specify a matching-ID control in an independent fixture so rejection cannot be explained by a broken request or unavailable environment.

For identity-sensitive behavior, consider similar identifiers such as `record-120` versus `record-121`, and `record-12` versus `record-120`. These can reveal accidental prefix matching or truncation that unrelated identifiers would miss. Use the actual contract's identity rules rather than inventing normalization behavior.

## Specify observable asynchronous ordering

Treat response ownership and stale-response races as separate behaviors. Receiving another resource's data tests identity; receiving an older result after a newer action tests ordering.

```gherkin
Feature: Search results follow the user's latest query

  @P1 @web
  Scenario: A late response cannot replace the latest results
    Given an earlier search is still waiting for its response
    And a newer search has already completed and its results are displayed
    When the earlier search response arrives
    Then the displayed results still correspond to the newer search
```

In execution notes, specify an observable sequence: confirm the earlier request has entered, hold its response, complete the newer action, then release the earlier response and assert the outcome. Use a controllable stub, request interception, or another capability already available to the executor. Fixed or random sleeps alone do not establish that this ordering happened.

Give waits a bound and specify cleanup for held requests. If the necessary controls are unavailable, record that prerequisite rather than pretending a sequential run covered the race. Do not require the application to perform unrelated requests concurrently if sequential processing satisfies its contract.

## Review the failure the case should detect

For important cases, identify a plausible incorrect behavior the assertions should reject: always using a default range, comparing only an ID prefix, or displaying an older response after a newer action. Use that explanation to review the chosen fixtures and assertions.

This is a design review, not an instruction to mutate production code, run tests, or install extra tools while generating cases. A written scenario remains a specification until its execution requirements are implemented and the scenario is actually run.
