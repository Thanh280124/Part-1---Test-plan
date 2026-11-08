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

**Prerequisites:** Catalog has multiple producers, optional missing attributes, and many more matches than can be displayed per page. **Main workflow:** (1) user opens catalog; (2) user types in the search query; (3) frontend validates input and makes a REST call; (4) backend responds with matches; (5) user uses filters by category, price, content, or producer; (6) system aggregates filters; (7) user flips through pages; (8) appropriate matches are shown on the page with applied filters. **Variations:** blank/white space search, no matches, missing optional attributes, last/invalid page, or invalid price range. **Expected result:** only valid products are shown; paging works fine; blank results are justified. **Candidate utilities:** `filter`, `every`, `toNumber`, `isEmpty`, `defaultTo`, `chunk`, `slice`. **Risk:** wrong filters or page numbers hide/misrepresent stock. Activity diagram: see Appendix A.

### S2 — Cart, quantities and checkout

**Preconditions:** Prices for products are legitimate; access to the cart and checkout. **Main flow:** (1) selection of products by customer; (2) addition/removal of product from the cart; (3) change of quantity; (4) calculation of sub-totals and total by the cart for all actions; (5) review of the cart; (6) sending of the checkout information from backend to external provider; (7) display of the result on UI. **Variations:** invalid, null, negative or non-integer quantity; empty cart; unavailable provider; duplicate submission. **Expected results:** accurate cart content and proper formatting of money values without fake message about success of payment. **Candidate utilities:** `add`, `reduce`, `toNumber` (plus `toInteger`, `clamp` as unselected variants). **Risk:** wrong totals or duplicate purchase. Activity Diagram: Appendix B. General `add` utility cannot be considered reliable for dealing with decimal money; specific money operations require separate testing.

### S3 — Producer creation and bulk import

**Prerequisites:** Access by the producer of the entry form or import portal; mandatory field requirements. **Main flow:** (1) choice of single entry or bulk import by the producer; (2) providing product name, price, producer, and optional additional information; (3) validation of input by the frontend/importer; (4) invalid entries reported per the import policy; (5) valid entries stored using REST; (6) producer verifies entries in the catalogue. **Variants:** missing mandatory field, optional fields null values, incorrectly formatted prices, mixed invalid/valid entries, duplicates, and bulk imports. **Expected results:** No silent addition of invalid products, successful import and correct catalogue. **Candidate utilities:** `toNumber`, `isEmpty`, `every`, `defaultTo`, `countBy`; `upperFirst` is an unused candidate for formatting purposes. **Risk:** Corrupted data in catalogue affects search and purchase. Activity diagram: Appendix C.

## 2.2 Scope and risk-based selection

Only ten `src` files at the highest level will be subjected to unit test. The hidden `.internal` folder is assumed to be error-free and hence will neither be tested nor covered, according to the instructions [1]. The user interface for the application, REST back-end service, and the interaction with the payment provider are beyond the scope of the provided implementation and their suggested tests are listed in §4.3.

The team chose the ten files above based on three criteria between 1-3 in total: **scenarios coverage** (number of S1-S3 scenarios that can be used), **impact of failure** (consequences of a possible error) and **estimated defect likelihood** (complexity and corner cases). **Total = coverage + impact + likelihood.** It is not about actual defect probability but rather an estimation for planning purposes. In case of ties, the priority goes first by the business impact, then by the coverage, and finally by the relevance to implementation chosen by the group. Scores and the unselected candidates are recorded in Appendix D.

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

The files `clamp.js`, `toInteger.js`, `toString.js`, and `upperFirst.js` were considered but not included due to the file limit of ten. `map.js` and `toFinite.js` are other possible choices but not included in the final selection of the team. This is a **selection rationale** that the team has to verify the actual exports of the functions prior to Part 2. The library helper does not perform the search, check out, or producer validation.

## 2.3 Assumptions and limitations

`.internal` functions within the library are deemed safe solely based on the fact that the assignment allows for such an assumption to be made, while no other `src` functions may be considered so. Expected results need to be based on the function semantics known for certain rather than on theoretical applications. If any of the exported helpers forcibly manipulates the input or outputs IEEE-754 floating point numbers, then such actions need to be mentioned along with the potential application-based issues. There will be no application code, backend interface, sandboxed payments, nor example products provided [1].

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

The unit tests invoke each chosen export in isolation against deterministic input data along with an expected result. Employ equivalence partitioning (valid/invalid type classes), boundary value analysis (empty, singleton, around page limits), decision branch coverage, negative testing and regression test cases. Test cases should be independent, repeatable, and behave-named. The verified contract and not any particular e-commerce feature will be the oracle. Do not include testing of implementation detail, nor replicate tests from `.internal`. Use stubs only where necessary for external and nondeterministic dependencies, otherwise not for pure collection helper functions. Where actual source semantics differ from a suggested oracle, check the contract and adjust the test case before treating it as a bug. The detailed designs below are **planned test cases**, not executed results.

## 4.2 Detailed test designs for two selected functions

### Function A: `chunk.js` — pagination support (S1)

**Assumed interface for test design:** `chunk(array, size)` splits an array into subarrays that are consecutive and contain no more than `size` items each; the last chunk can be smaller. The exact behavior in the case of the default or invalid value of size should be checked in the repo. **No stubs** are needed.

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

CH-09/10 should be documented first with the observed contract (no result, coercion, error, etc.) before choosing an expected value; do not make up a validation rule for the helper in your application. **Scenarios linked:** CH-01/02/04/05/06 cover first/last page handling; S1 also requires integration tests for saving filters between pages.

### Function B: `toNumber.js` — numeric conversion (S1, S2, S3)

**Assumed interface for test design:** `toNumber(value)` handles conversions based on the specified number coercion semantics in the library; it does **not** itself validate a monetary amount or two-decimal precision in currency. Check for `NaN` using `Number.isNaN`, not equality. **No stubs** will be needed. Validate the actual source behavior for white spaces, non-decimal syntax, and object types.

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

For `filter`, check for match/mismatch predicates, empty collections and multiple entries. For `isEmpty`, contrast empty/nonnull arrays, strings and objects. For `add`, check positive/negative/zero operands and float arithmetic without using rounding conventions for currency. For `countBy`, check repeated categories, unique entries and empty arrays. For `reduce`, check for an initial accumulator, empty input, and order of reducer. For `every`, check all/one/no matches and the empty array condition. For `defaultTo`, check null/undefined/NaN against true false values, such as `0` and `false` per source. For `slice`, check negative indexes, index out of bounds, empty output and order preservation. Link test case IDs to source files and S1–S3 in the Part 2 report.

## 4.3 Additional testing for the complete e-commerce system

**Functional E2E (top business priority):** run S1 with filters, no results, optional missing fields and pagination; S2 with quantity changes, empty cart, totals, checkout failure and duplicate submission protection; S3 with valid and invalid records, partial import, duplicates and catalogue verification. Use realistic fixtures and backend persistence verification, not only frontend notifications.

**API/integration:** validate REST API request/response schema and error codes, catalogue/cart persistence, producer API access permissions, checkout request mapping, payment provider retry mechanism and duplicate payment prevention. Stub external payment provider for deterministic failures and run a small contract sandbox test if possible. Determine whether bulk import API call is atomic or partial first to properly validate it.

**Security/data integrity:** check input validation/XSS, authorization of producer API calls, CSRF if session based, protection from injections, rate limiting and protection of payment data; do not use real customer/card details. Check required fields and two decimal places monetary policy on the server side as the frontend validation can be easily bypassed.

**Nonfunctional and usability testing:** testing performance with regard to big catalogues, filtering, and batch uploads; test pagination consistency; test search/cart/errors accessibility; test compatibility with browsers/mobility; test fault tolerance for network problems and failure recovery. The tests related to price accuracy, authentication, check out functionality, and data accuracy have top priority over cosmetic tests. They are **recommendations** as the program is not accessible [1].

## 4.4 AI use, opportunities and safeguards

**Actual Phase 1 use:** ChatGPT was utilized help structure and edit this report, propose scenario-to-function mappings, refine the prioritisation explanation, draft unit-test cases and outline additional testing risks.  The team remained responsible for verifying all library contracts, testing tools, citations, diagrams, and the final content included in the report against the assignment requirements and source materials. No AI-generated test cases were considered "executed," and no test coverage or defect results were fabricated. The final submission must be reviewed and revised by both team members to account for any additional tools they may have employed.

**Possible Phase 2 application:** The AI could propose equivalence classes, edge cases, and preliminary Jest test skeletons for the ten chosen functions; aid in understanding failing test results and finding uncovered branches; and generate clear issue reports. This is particularly helpful when there is no documentation for the used library and for the conversion/empty-input edge cases. Possible risks include generating imaginary function contracts, erroneous assertion of expected results, shallow tests that merely mirror the behavior of the implementation, neglecting security tests, and leaking proprietary information/code. Mitigations: Review source code before accepting any recommendation, double-check numerical oracles by hand, keep reviewable tests in GitHub PRs, synthesize data, and disclose AI assistance openly. AI is an assistant, not an independent test oracle [1].

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
