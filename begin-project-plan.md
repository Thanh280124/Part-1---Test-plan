# Initial Test Plan Proposal

## E-commerce Store — Part 1

**Course:** Software Testing
**Member 1:** George
**Member 2:** Thanh

---

## 1. Proposed Report Structure

We propose using the following structure for the final report.

1. **Definitions, Acronyms and Abbreviations**
2. **Introduction**
   - Purpose of the document
   - Scope and overview of the test plan
3. **Context of Testing**
   - 3.1 End-to-End Scenarios
   - 3.2 Testing Scope and Prioritisation
   - 3.3 Testing Assumptions
4. **Tools and Reporting**
   - 4.1 Testing Environment and Tools
   - 4.2 Test Reporting
   - 4.3 Defect Tracking and Management
   - 4.4 Metrics and Quality Gates
5. **Tests**
   - 5.1 General Principles of Unit Testing
   - 5.2 Detailed Test Cases for Two Utility Functions
   - 5.3 Other Testing for the E-commerce Store
   - 5.4 AI and Testing
6. **References**
7. **Appendix**

The cover page and table of contents will be placed before the main sections.


## 2. Proposed End-to-End Scenarios

Three scenarios that represent important workflows in the e-commerce store.

### Scenario 1: Searching and Filtering Products

**Goal:** Verify that customers can find products using the available search and filtering features.

Proposed workflow:

1. The customer opens the product catalogue.
2. The customer searches for a product by name.
3. The customer applies filters such as category, price, contents, or producer.
4. The system displays matching products.
5. The customer navigates through multiple result pages when necessary.

**Main areas to consider:**

- Search and filtering correctness
- Empty search results
- Products with missing optional attributes
- Pagination and large result sets
- Input validation

**Proposed priority:** High, because searching and filtering are core customer-facing functions.

### Scenario 2: Shopping Cart and Checkout

**Goal:** Verify that customers can add products to their shopping cart and see the correct total price.

Proposed workflow:

1. The customer searches for a product.
2. The customer selects a product and adds it to the cart.
3. The customer adds another product or changes the quantity.
4. The shopping cart updates automatically.
5. The customer checks the total price.
6. The customer proceeds to the third-party checkout service.

**Main areas to consider:**

- Correct price calculations
- Quantity and cart updates
- Decimal precision and rounding
- Invalid quantities or prices
- Communication with the third-party checkout service

**Proposed priority:** High, because incorrect prices or cart behaviour can directly affect purchases.

### Scenario 3: Adding Products as a Producer

**Goal:** Verify that producers can add products with valid mandatory information while leaving optional fields empty when permitted.

Proposed workflow:

1. The producer opens the product management interface.
2. The producer enters the product name, price, and producer information.
3. The producer optionally enters the category, contents, and other attributes.
4. The system validates the submitted information.
5. The product is saved if the input is valid.
6. The product becomes available in the store's catalogue.

**Main areas to consider:**

- Mandatory and optional fields
- Invalid or missing information
- Product description validation
- Price format and accuracy
- Consistency between product creation and product search

Bulk product import should also be considered when discussing additional testing.

**Proposed priority:** High, because invalid product information can affect the catalogue and subsequent customer transactions.

**Agreement needed:** Both members should confirm these scenarios, identify any missing steps, and decide which UML diagram type is most suitable for each scenario.

# 4. Tests

## 4.1 General principles for unit testing the utility library

Unit tests for the selected utility library functions will follow these principles:

- **Isolation**: Each function is tested in isolation. External dependencies (if any) will be stubbed using Jest’s built-in mocking capabilities.
- **Behaviour-focused**: Tests verify observable behaviour (return values, thrown errors, side-effect-free nature) rather than internal implementation details.
- **Comprehensive input classes**: Every function will be tested with:
  - Normal / happy-path inputs
  - Boundary values
  - Invalid / unexpected types
  - Empty, null, and undefined inputs where relevant
  - Extreme values (very large numbers, empty arrays/strings, etc.)
- **Deterministic and independent**: Tests must not depend on execution order or shared mutable state.
- **Clear naming and documentation**: Test names clearly describe the scenario and expected outcome. Each test case states its purpose.
- **No modification of source code**: Defects are only reported, never fixed.
- **Coverage-driven but not coverage-only**: High statement and branch coverage is targeted, but meaningful assertions are prioritised over simply executing lines.

These principles ensure that the tests remain maintainable, readable, and useful for assessing the library’s suitability for the e-commerce application.

## 4.2 Detailed unit tests for two selected functions

Two functions were selected for detailed test design because of their high relevance to the end-to-end scenarios and the richness of their edge cases:

1. `filter.js` – core to product search and filtering (Scenario 1)
2. `add.js` – directly related to shopping-cart total calculation (Scenario 2)

**No stubs are required** for either function; both are pure and have no external dependencies.

### 4.2.1 Unit tests for `filter(array, predicate)`

| Test ID | Purpose | Input | Expected result / behaviour |
|---------|---------|-------|-----------------------------|
| FIL-01 | Normal filtering | `[{id:1, price:10}, {id:2, price:25}, {id:3, price:15}]`, predicate `p => p.price > 12` | `[{id:2, price:25}, {id:3, price:15}]` |
| FIL-02 | No elements match | Same array, predicate `p => p.price > 100` | `[]` |
| FIL-03 | All elements match | Same array, predicate `p => p.price > 0` | Original array (new instance) |
| FIL-04 | Empty array | `[]`, any predicate | `[]` |
| FIL-05 | Null / undefined array | `null` or `undefined`, any predicate | `[]` (or documented behaviour) |
| FIL-06 | Predicate always false | Any non-empty array, `() => false` | `[]` |
| FIL-07 | Predicate uses index | `[10, 20, 30, 40]`, `(v, i) => i % 2 === 0` | `[10, 30]` |
| FIL-08 | Sparse / missing elements | Array with holes, appropriate predicate | Correctly skips empty slots |
| FIL-09 | Non-array input | `"string"` or `42` | `[]` or TypeError (document actual behaviour) |
| FIL-10 | Predicate throws | Array + predicate that throws | Exception propagates |

### 4.2.2 Unit tests for `add(value, other)`

| Test ID | Purpose | Input | Expected result / behaviour |
|---------|---------|-------|-----------------------------|
| ADD-01 | Normal addition of integers | `5, 3` | `8` |
| ADD-02 | Addition of floats (price-related) | `19.99, 4.50` | `24.49` |
| ADD-03 | Adding zero | `10.5, 0` | `10.5` |
| ADD-04 | Negative numbers | `10, -3.5` | `6.5` |
| ADD-05 | Large numbers | `1e20, 1e20` | Correct sum (or Infinity handling) |
| ADD-06 | String numbers that can be coerced | `"19.99", "0.01"` | `20` (or documented coercion behaviour) |
| ADD-07 | Invalid types (object, array, null) | `null, 5` / `{}, 5` / `[1], 5` | `NaN` or documented behaviour |
| ADD-08 | Undefined arguments | `undefined, 5` / `5, undefined` | `NaN` |
| ADD-09 | Only one argument | `add(7)` | `NaN` or documented behaviour |
| ADD-10 | Precision-sensitive case | `0.1, 0.2` | Closest representable value (document floating-point reality) |

These test cases cover the most important behavioural aspects that would affect product filtering and cart total calculation in the e-commerce store.

## 4.3 Other testing recommended for the e-commerce store

Although Part 2 only implements unit tests for the selected library files, a complete quality strategy for the described e-commerce system should include the following levels:

**High priority**
- **Integration testing** between frontend and REST backend (search, filter, cart operations, product creation).
- **API contract testing** of the REST endpoints (status codes, response schemas, error formats).
- **Input validation testing** on the frontend (price format with two decimals, mandatory fields, description capitalisation rules, malformed quantities).
- **Pagination testing** with large result sets (empty page, last page, changing filters while paginated).
- **Shopping-cart calculation testing** (multiple items, quantity changes, removal of last item, floating-point precision).

**Medium priority**
- End-to-end tests of the three scenarios described in Section 2.1 (using tools such as Playwright or Cypress).
- Boundary and negative testing of bulk product import.
- Error-handling tests when the third-party payment service is unavailable or returns failures.
- Usability checks for empty states and error messages.

**Lower priority / later stages**
- Performance testing of search and pagination with thousands of products.
- Security testing (injection, authentication/authorisation for producer endpoints).
- Accessibility and cross-browser testing.

**Risky areas that deserve special attention**
- Price and total calculations (floating-point precision).
- Handling of optional product attributes (category, contents).
- Consistency between individual product creation and bulk import.
- Behaviour when the third-party checkout service fails.

Testers should prioritise risk-based testing: start with the money-related and data-integrity paths before moving to less critical features.

## 4.4 AI and testing

**Potential benefits of AI in this e-commerce context**
- Rapid generation of initial unit test skeletons for pure utility functions.
- Suggestion of edge cases that humans might overlook (especially for numeric and string functions).
- Automatic generation of realistic product test data for integration and E2E tests.
- Assistance in writing clear defect reports and test documentation.
- Helping new interns understand the purpose of library functions through explanations.

**Potential drawbacks and risks in this specific assignment**
- AI may generate tests that only cover the happy path or that assert incorrect expected behaviour when the library itself is buggy.
- Over-reliance can reduce the learning value of designing tests manually (especially important because Part 2 will compare human-written vs AI-generated tests).
- AI hallucinations about function behaviour are possible when source code comments are missing or misleading.
- Generated E2E scenarios may not match the limited documentation provided in the assignment.
- Using AI for the detailed unit-test design in Section 4.2 is explicitly forbidden by the assignment.

**Declaration of AI usage in Phase 1**
- AI tools (Grok) were used to:
  - Discuss structure and completeness of the test plan.
  - Review drafts of Sections 1–3.
  - Suggest improvements to wording, prioritisation rationale, and quality-gate criteria.
  - Help formulate the content of Section 4 (except the detailed test cases in 4.2, which were designed manually).
- No AI-generated test code or detailed test cases from Section 4.2 were copied.
- For Phase 2, the group plans to use AI only for:
  - Explaining difficult library functions if needed,
  - Reviewing test code style,
  - Drafting the final test report text.  
  AI will **not** be used to generate the actual unit test implementations that will be submitted.