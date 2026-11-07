# TEST PLAN — E-COMMERCE STORE

**COMP.SE.200-2026-2027-1 Software Testing | Part 1 — Test Plan**  
**Group 31 | Pham Thanh | Georgios Malezoglou**  
**Student numbers:** [Insert both student numbers before submission]  
**Version:** 1.0 | **Date:** 7 November 2026

## Definitions and abbreviations

| Term | Meaning |
|---|---|
| SUT | System under test: the selected utility-library source files |
| E2E | End-to-end; a complete user workflow across system components |
| CI | Continuous integration; automated verification on code changes |
| REST/API | Representational State Transfer / application programming interface |
| Coverage | Proportion of instrumented statements or branches exercised by tests |
| Stub | Controlled substitute for a dependency during a test |
| Unit test | A test of one small unit of code in isolation |
| Quality gate | A condition that must be satisfied before testing or a change is accepted |

# 1. Introduction

## 1.1 Purpose and deliverables

This plan explains what will be tested, why those areas matter, how tests will be designed and reported, and how results will support a decision about reusing an undocumented utility library in a food-product e-commerce store. It defines three representative E2E scenarios, ten prioritised utility source files, a reproducible test approach, reporting and defect procedures, quality gates, detailed unit-test designs, and additional system-level testing recommendations. Part 1 delivers this plan only; Part 2 will implement unit tests for the selected files, GitHub Actions and Coveralls integration [1].

## 1.2 Application context and risk

Customers search and filter products by name, category, price, contents and producer; browse paginated results; update a cart; and use an external checkout provider. Producers add products individually or through an existing bulk-import portal. Name, price and producer are mandatory; category and contents may be absent. The frontend is expected to validate prices, descriptions and order inputs; a REST backend handles catalogue operations and third-party services [1]. Wrong filters may hide products; numeric conversion errors may affect totals; poor validation may admit invalid product records. The library is available in an abandoned project with missing documentation [2], so the intended behaviour must be corroborated against its exported functions, examples and source rather than assumed from filenames alone.

# 2. Context of testing

## 2.1 Three representative E2E scenarios

### S1 — Product search, filtering and pagination

**Preconditions:** A catalogue has multiple producers, optional missing attributes and more matches than fit on one page. **Main flow:** (1) customer opens catalogue; (2) enters a search term; (3) frontend validates and sends a REST request; (4) backend returns matches; (5) customer applies category, price, contents or producer filters; (6) the system combines criteria; (7) customer changes page; (8) the UI displays the correct subset while retaining filters. **Alternatives:** empty/whitespace query, zero results, absent optional fields, last/invalid page, or invalid price boundary. **Expected outcome:** only qualifying products appear; pagination is stable; empty results are explained. **Candidate utilities:** `filter`, `every`, `toNumber`, `isEmpty`, `defaultTo`, `chunk`, `slice`. **Risk:** incorrect predicates or page boundaries conceal or misrepresent stock. Activity diagram: Appendix A.

### S2 — Cart, quantities and checkout

**Preconditions:** Products have valid prices; cart and checkout are accessible. **Main flow:** (1) customer selects products; (2) adds or removes an item; (3) changes quantity; (4) cart recalculates line totals and overall total after each action; (5) customer reviews the cart; (6) backend submits checkout data to the external provider; (7) the UI communicates the actual outcome. **Alternatives:** malformed, zero, negative or fractional quantities; empty cart; unavailable provider; duplicate submission. **Expected outcome:** correct cart state, appropriately rounded monetary presentation and no false payment-success message. **Candidate utilities:** `add`, `reduce`, `toNumber`, `isEmpty` (and `toInteger`/`clamp` as unselected alternatives). **Risk:** incorrect financial totals or duplicate purchases. Activity diagram: Appendix B. A general-purpose `add` is not assumed to guarantee decimal-currency correctness; application-level money handling needs separate tests.

### S3 — Producer creation and bulk import

**Preconditions:** Producer can access the entry form or import portal; mandatory-field rules exist. **Main flow:** (1) producer chooses single entry or bulk import; (2) supplies product name, price, producer and optional details; (3) frontend/import service validates data; (4) invalid records are reported according to the import policy; (5) valid records are saved via REST; (6) producer confirms catalogue entries. **Alternatives:** missing required fields, optional nulls, malformed prices, mixed valid/invalid rows, duplicates and large batches. **Expected outcome:** no silently accepted invalid product, clear import result and consistent catalogue. **Candidate utilities:** `toNumber`, `isEmpty`, `every`, `defaultTo`, `countBy`; `upperFirst` is an unselected candidate for description formatting. **Risk:** corrupt catalogue data propagates into search and purchasing. Activity diagram: Appendix C.

These scenarios specify **proposed** application behaviour; the actual application and interfaces are not supplied. They guide the choice of library unit tests, not a claim that E2E testing can be executed in Part 2 [1].

## 2.2 Scope and risk-based selection

Only ten top-level `src` files will receive unit tests. The hidden `.internal` directory is assumed correct and excluded from both testing and coverage, as instructed [1]. The application frontend, REST backend, integration with the payment provider and executable E2E flows are outside the available implementation scope. Their recommended future tests appear in §4.3.

The group selected the following ten files using three 1–3 factors: **scenario coverage** (number of S1–S3 workflows supported), **impact of failure** (business harm) and **estimated defect likelihood** (complexity and edge cases). **Total = coverage + impact + likelihood.** This is a planning judgement, not an empirical defect probability. Ties prioritise higher impact, then higher coverage, then the group's chosen implementation relevance. Scores and the unselected candidates are recorded in Appendix D.

| Selected file | Scenarios | Primary test focus |
|---|---|---|
| `toNumber.js` | S1, S2, S3 | Numeric coercion, invalid values and prices |
| `filter.js` | S1 | Predicate correctness, empty and mixed arrays |
| `chunk.js` | S1 | Page sizes, remainder and boundary inputs |
| `isEmpty.js` | S1, S2, S3 | Empty strings, arrays, objects and null-like data |
| `add.js` | S2 | Arithmetic, coercion and precision limitations |
| `countBy.js` | S3 | Group counts and duplicate detection support |
| `reduce.js` | S2 | Accumulation, initial value and empty collections |
| `every.js` | S1, S3 | All-condition predicates and empty collections |
| `defaultTo.js` | S1, S3 | Missing optional values and valid falsy values |
| `slice.js` | S1 | Start/end bounds and result subsets |

`clamp.js`, `toInteger.js`, `toString.js` and `upperFirst.js` were considered but excluded under the ten-file limit; `map.js` and `toFinite.js` are additional plausible alternatives but are not part of the group's final selection. The prioritisation is a **selection rationale**; before Part 2, the team must confirm the functions' actual exported contracts and module compatibility against the repository. A library helper alone does not implement search, checkout or producer validation.

## 2.3 Assumptions and limitations

The library's `.internal` helpers are trusted only because the assignment explicitly permits that assumption; other `src` functions are not. Expected results should follow confirmed function semantics, not idealised application rules. Where an exported helper deliberately coerces inputs or returns IEEE-754 floating-point values, tests should document that behaviour and separately identify application-level risks. No application code, backend contract, payment sandbox or example product data will be supplied [1]. The team will report defects, not modify the library. Student responsibilities and scheduling are outside the required scope of this assignment [1].

# 3. Tools, reporting and quality controls

## 3.1 Environment and toolchain

The proposed **Part 2** stack is Node.js/npm, Git/GitHub, **Jest** for test execution/assertions, **Jest's coverage reporting** for local statement/branch data and LCOV output, **GitHub Actions** for CI and **Coveralls** for published coverage. This is a concrete planned configuration rather than a claim of installation or successful execution. The existing repository's `package.json`, module format and scripts must be inspected before finalising Jest's ESM/CommonJS configuration; if incompatible, document the adjustment rather than silently changing the source. `c8` is a viable alternative coverage tool but should not be layered over Jest coverage without a reason. Chai/Mocha, testdouble and third-party reporters are not required for the chosen approach. GitHub Actions and Coveralls are configured in Part 2, not Part 1 [1,3–6].

**Reproducible setup outline:** clone the supplied repository; record the tested commit and Node/npm versions; inspect `package.json`, source imports and existing tests; install locked dependencies where a lockfile exists (`npm ci`, otherwise `npm install`); add/configure Jest in the test project; create a minimal smoke test for one selected export; run Jest locally; enable `collectCoverageFrom` for **only the ten selected files** and exclude `**/.internal/**`; emit LCOV for Coveralls; in Part 2, add a GitHub Actions workflow to install, run tests and upload the LCOV report. Keep the source code unmodified and record commands/configuration in the repository README. Appendix E contains a configuration sketch, **not a verified working command sequence**.

## 3.2 Execution and result reporting

For each test run record date/time, branch and commit SHA, Node version, command, total/planned/executed/passed/failed/skipped/blocked tests, failing case IDs, expected versus actual output, coverage per selected file, and issue links. A failure is triaged first: verify the test oracle and environment before labelling a library defect. During testing, maintain a concise progress log; after testing, publish a report with results by function, unresolved issues, scenario implications and limitations. Archive test code, machine-readable test results, coverage/LCOV artefacts and GitHub Issues. Never present a blocked or unexecuted test as passed.

## 3.3 Defect tracking

Use **GitHub Issues** with labels for severity, priority, function and status. Workflow: **New → Confirmed → Investigating → Documented/Resolved → Retest (if applicable) → Closed**; a non-defect can be **Rejected** with an explanation. “Resolved” does not imply the team repaired library source code. Severity is technical/business impact (critical: materially unsafe totals or unusable core operation; high: important incorrect outputs; medium: bounded edge-case failure; low: minor unusual-input discrepancy). Priority is urgency (P1 immediate report, P2 prompt, P3 routine, P4 low). Severity and priority are recorded separately.

**Issue template:** title and ID; affected file/export; test-case ID; commit/environment; exact inputs and preconditions; steps; expected result with source of oracle; actual result; logs/screenshots if relevant; severity/priority; assignee/status; scenario consequence; and resolution/retest notes. Duplicate reports are linked rather than counted twice.

## 3.4 Metrics, entry/exit criteria and quality gates

**Entry:** ten files fixed; expected contracts inspected; runnable test environment and coverage configuration; cases have explicit inputs and oracles. **Case pass:** observed output equals the expected contract, including thrown exceptions where specified. **Blocked:** unmet prerequisite prevents execution; record separately. **Exit:** all planned cases executed or justified as blocked; failures triaged and recorded; coverage collected for the selected files only; uncovered paths and unresolved risks explained; final report produced. A stage is not declared clean if material failures remain, even when coverage is high.

| Metric | Calculation / target |
|---|---|
| Execution rate | Executed / planned × 100; target 100% or documented exceptions |
| Pass rate | Passed / executed × 100; report observed rate, do not invent a target outcome |
| Confirmed defects | Count by function, severity and scenario consequence |
| Statement coverage | Covered statements / measurable statements; **aim ≥80%** |
| Branch coverage | Covered branches / measurable branches; **aim ≥80%** where available |

These 80% thresholds are **group targets**, not assignment-mandated guarantees. Design normal, invalid, equivalence-class, boundary and negative tests; inspect uncovered branches; add meaningful tests or explain infeasible paths. Coverage measures execution, not correctness. For the ten-file scope, calculate coverage using the selected-file denominator, not the whole library.

# 4. Tests and AI in testing

## 4.1 Unit-test design principles

Unit tests call each selected export in isolation using deterministic input data and an explicit expected result. Use equivalence partitioning (valid/invalid types), boundary-value analysis (empty, singleton, near page limits), decision/branch coverage, negative tests and regression cases. Tests must be independent, repeatable and named by behaviour. A confirmed contract—not a desired e-commerce feature—is the oracle. Avoid testing internal implementation details or duplicating `.internal` tests. Use stubs only for external or nondeterministic dependencies; pure collection helpers should not need them. Where actual source semantics differ from a proposed oracle, review the contract and update the case before calling the mismatch a defect. The detailed designs below are **planned test cases**, not executed results.

## 4.2 Detailed test designs for two selected functions

### Function A: `chunk.js` — pagination support (S1)

**Assumed interface for test design:** `chunk(array, size)` partitions an array into consecutive arrays of at most `size` elements, preserving order; the final chunk may be shorter. The precise default/invalid-size behaviour must be checked in the repository before implementation. **No stubs** are expected. Each case below states a concrete input and oracle for the normal positive-size contract.

| ID | Input | Expected output | Purpose |
|---|---|---|---|
| CH-01 | `[1,2,3,4,5]`, size `2` | `[[1,2],[3,4],[5]]` | Non-divisible page count |
| CH-02 | `[1,2,3,4]`, size `2` | `[[1,2],[3,4]]` | Exact division |
| CH-03 | `[]`, size `3` | `[]` | Empty catalogue |
| CH-04 | `[7]`, size `3` | `[[7]]` | Fewer items than page size |
| CH-05 | `[1,2,3]`, size `1` | `[[1],[2],[3]]` | Smallest positive size |
| CH-06 | `[1,2,3]`, size `5` | `[[1,2,3]]` | Oversized page |
| CH-07 | `['a','b','c']`, size `2` | `[['a','b'],['c']]` | Preserve values/order |
| CH-08 | `[0,null,false]`, size `2` | `[[0,null],[false]]` | Preserve falsy values |
| CH-09 | `[1,2,3]`, size `0` | **Confirm source contract before asserting** | Invalid/boundary size |
| CH-10 | `[1,2,3]`, size `-1` | **Confirm source contract before asserting** | Negative size handling |

For CH-09/10, record the observed contract (empty result, coercion, error or other documented behaviour) before assigning an expected value; do not invent an application-level validation rule for the helper. **Scenario link:** CH-01/02/04/05/06 address first/last-page behaviour; S1 additionally needs integration tests for preserving filters when navigating pages.

### Function B: `toNumber.js` — numeric conversion (S1, S2, S3)

**Assumed interface for test design:** `toNumber(value)` converts values according to the library's documented JavaScript number-coercion semantics; it does **not** itself validate a monetary amount or guarantee two-decimal currency precision. Compare `NaN` with `Number.isNaN`, not equality. **No stubs** are expected. Validate the exact source behaviour for whitespace, nondecimal syntax and objects before implementation.

| ID | Input | Expected result | Purpose |
|---|---|---|---|
| TN-01 | `'12.50'` | `12.5` | Decimal price string |
| TN-02 | `0` | `0` | Zero remains zero |
| TN-03 | `'-3'` | `-3` | Negative numeric conversion (not acceptance) |
| TN-04 | `'  42  '` | `42` | Surrounding whitespace |
| TN-05 | `'abc'` | `NaN` | Malformed numeric text |
| TN-06 | `''` | `0` | Empty-string conversion; app must separately reject blank price |
| TN-07 | `null` | `0` | Null conversion; app must separately reject missing price |
| TN-08 | `true` | `1` | Boolean coercion; not valid producer input |
| TN-09 | `'0x10'` | `16` | Nondecimal numeric notation |
| TN-10 | `'1.234'` | `1.234` | More than two decimal places remain unrounded |
| TN-11 | `'Infinity'` | `Infinity` | Non-finite values need separate validation |
| TN-12 | `['7']` | **Confirm source contract before asserting** | Array coercion / unexpected input type |

**Scenario link:** TN-01/04 support price-filter and product-entry parsing; TN-05/06/07/08/10/11 demonstrate why frontend/backend validation is needed *after* conversion. For S2, calculate a sample cart total separately (e.g. 2 × €3.25 + 1 × €1.40 = €7.90) and test formatting/rounding at the application layer, not as a promised `toNumber` feature.

### Coverage of the other eight selected files

For `filter`, test matching/nonmatching predicates, empty collections and multiple records. For `isEmpty`, compare empty/nonempty arrays, strings and objects. For `add`, test positive/negative/zero operands and floating-point behaviour without assuming currency rounding. For `countBy`, test repeated categories, unique values and empty arrays. For `reduce`, test an initial accumulator, empty input and noncommutative reducer order. For `every`, test all/one/none matching and the empty-array case. For `defaultTo`, distinguish null/undefined/NaN from meaningful falsy values such as `0` and `false` as dictated by the source. For `slice`, test negative indexes, out-of-range indexes, empty results and order preservation. Link test case IDs to source files and S1–S3 in the Part 2 report.

## 4.3 Additional testing for the complete e-commerce system

**Functional E2E (highest business priority):** execute S1 with combined filters, no results, optional missing fields and pagination; S2 with quantity changes, empty cart, price totals, failed checkout and duplicate-submission protection; S3 with valid/invalid records, partial import, duplicates and catalogue verification. Use realistic fixtures and verify backend persistence, not only frontend messages.

**API/integration:** test REST validation and response codes, schema/contract compatibility, catalogue/cart persistence, producer access permissions, checkout request mapping, provider timeout/retry and duplicate-payment prevention. Stub the external payment service for deterministic failure tests, then run a small sandbox contract test where permitted. Confirm whether bulk import is atomic or partial before asserting its behaviour.

**Security and data integrity:** test input sanitisation/XSS, authorization for producer operations, CSRF where session-based, injection resistance, rate limiting and protection of payment-related data; avoid real customer or card data. Verify required fields and two-decimal monetary policy server-side, since frontend checks can be bypassed.

**Nonfunctional and usability:** performance of large catalogues, filter combinations and bulk imports; pagination stability; accessibility of search/cart/errors; cross-browser/mobile layout; resilience to network interruption; and recovery from service failure. Prioritise price integrity, authorization, checkout reliability and product-data correctness before cosmetic concerns. These tests are **recommendations**, not executable Part 2 deliverables because the application is unavailable [1].

## 4.4 AI use, opportunities and safeguards

**Actual Phase 1 use:** ChatGPT was utilized help structure and edit this report, propose scenario-to-function mappings, refine the prioritisation explanation, draft unit-test cases and outline additional testing risks.  The team remained responsible for verifying all library contracts, testing tools, citations, diagrams, and the final content included in the report against the assignment requirements and source materials. No AI-generated test cases were considered "executed," and no test coverage or defect results were fabricated. The final submission must be reviewed and revised by both team members to account for any additional tools they may have employed.

**Potential Phase 2 use:** AI can suggest equivalence classes, edge cases and initial Jest test skeletons for the ten selected functions; assist in interpreting failing test output and identifying untested branches; and draft readable issue reports. This is especially useful with missing library documentation and conversion/empty-input edge cases. Risks include hallucinated function contracts, incorrectly asserted expected values, superficial tests that only repeat implementation behaviour, overlooked security cases and leakage of proprietary code/data. Mitigation: inspect source before adopting any suggestion, manually validate numeric and boundary oracles, keep reviewable tests in GitHub PRs, use synthetic data, and report AI assistance transparently. AI is an assistant, not an independent test oracle or substitute for human verification [1].

# 5. References

[1] Tampere University, *COMP.SE.200 Software Testing: Practical 1 — Part 1 Test Plan and submission instructions*, 2026 (course-provided assignment text).  
[2] Utility library repository: https://github.com/otula/COMP.SE.200-2024-2025-1  
[3] Jest documentation: https://jestjs.io/docs/getting-started  
[4] GitHub Actions documentation: https://docs.github.com/en/actions  
[5] Coveralls documentation: https://docs.coveralls.io/  
[6] Node.js documentation: https://nodejs.org/en/learn

# Appendix (excluded from ten-page main-content limit)

## Appendix A — S1 activity diagram

![S1 product search activity diagram](scenario1.png)

## Appendix B — S2 activity diagram

![S2 cart and checkout activity diagram](scenario2.png)

## Appendix C — S3 activity diagram

![S3 producer import activity diagram](scenario3.png)

## Appendix D — Candidate prioritisation register

| File | Scenarios | Coverage | Impact | Likelihood | Total | Decision |
|---|---|---:|---:|---:|---:|---|
| `toNumber.js` | 1,2,3 | 3 | 3 | 2 | 8 | Selected |
| `filter.js` | 1 | 1 | 3 | 3 | 7 | Selected |
| `chunk.js` | 1 | 1 | 3 | 3 | 7 | Selected |
| `isEmpty.js` | 1,2,3 | 3 | 2 | 2 | 7 | Selected |
| `add.js` | 2 | 1 | 3 | 2 | 6 | Selected |
| `countBy.js` | 3 | 1 | 2 | 3 | 6 | Selected |
| `reduce.js` | 2 | 1 | 3 | 1 | 5 | Selected |
| `every.js` | 1,3 | 2 | 2 | 1 | 5 | Selected |
| `defaultTo.js` | 1,3 | 2 | 2 | 1 | 5 | Selected |
| `slice.js` | 1 | 1 | 2 | 2 | 5 | Selected |
| `clamp.js` | 2 | 1 | 1 | 3 | 5 | Not selected |
| `toInteger.js` | 2 | 1 | 2 | 1 | 4 | Not selected |
| `toString.js` | 1 | 1 | 1 | 2 | 4 | Not selected |
| `upperFirst.js` | 3 | 1 | 1 | 1 | 3 | Not selected |

The score-5 tie is not uniquely resolved by impact alone: `slice` and `clamp` both score 5, but `slice` has greater direct relevance to the agreed pagination scenario. This documented judgement preserves the group's ten-file decision. Scores are planning estimates, not source-verified measurements.

## Appendix E — Planned Jest configuration sketch

```js
// Illustrative only: confirm module format and source paths before Part 2.
module.exports = {
  testEnvironment: 'node',
  collectCoverage: true,
  coverageReporters: ['text', 'lcov'],
  collectCoverageFrom: [
    'src/{toNumber,filter,chunk,isEmpty,add,countBy,reduce,every,defaultTo,slice}.js',
    '!src/.internal/**'
  ]
};
```

**Final submission check:** insert both student numbers; confirm exported function contracts and adjust provisional oracles; review AI declaration; check diagrams and page count; export the final PDF and name it `31_STUDENT1_STUDENT2_2026_part1.pdf` with the actual student numbers; submit to Moodle before the stated deadline [1].
