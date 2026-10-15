# Initial Test Plan Proposal

## E-commerce Store — Part 1

**Course:** Software Testing
**Member 1:** George
**Member 2:** Thanh

---

## 1. Purpose of This Proposal

The purpose of this initial proposal is to help our group agree on the structure, testing approach, responsibilities, and priorities for Part 1 of the Test Plan assignment.

The application is an e-commerce store that sells food products from different small producers. Customers can search and filter products, add products to a shopping cart, and proceed to checkout through a third-party service. Producers can add products individually or import products in bulk.

The main focus of the assignment is to plan unit testing for an existing utility library. We must also discuss other testing activities that would be necessary for the e-commerce store.

This proposal is a starting point for discussion. The final decisions will be made by both group members after reviewing the assignment and studying the provided library.

## 2. Proposed Report Structure

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

**Agreement needed:** Both members should confirm the structure and decide whether any subsections need to be renamed or reorganised. The final report must follow all the assignment requirements and remain within the 10-page limit for the main content.

## 3. Proposed End-to-End Scenarios

We propose three scenarios that represent important workflows in the e-commerce store.

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

## 4. Proposed Approach to Selecting the Utility Library Files

The assignment limits the unit testing scope to a maximum of 10 source files. We should not select these files until we have examined the provided library.

### Proposed selection process

1. Examine the repository structure and identify the relevant files in the `src` directory.
2. Read the available source code and examples to understand the functions.
3. Identify which functions may support the three proposed scenarios.
4. Evaluate the relevant files using a prioritisation method.
5. Select up to 10 source files and document the reasons for each selection.
6. Explain why relevant files are excluded from the testing scope when appropriate.

### Proposed prioritisation criteria

We suggest evaluating each candidate using the following criteria:

| Criterion | Main question |
|---|---|
| Functional importance | How important is the function to the proposed e-commerce workflows? |
| Risk of failure | What could happen if the function produces an incorrect result? |
| Testing relevance | How directly does the function support the planned scenarios? |
| Testability | Can the function be tested reliably with the available information and environment? |

We can use a simple scoring system, such as 1–5 for each criterion, and agree on the weighting before ranking the files.

The final selection must be based on the actual source code, not just the filenames.

**Agreement needed:** Choose the prioritisation method and review the final list together. The `.internal` directory must be excluded from the test and coverage reports, as instructed in the assignment.

## 5. Proposed Testing Tools

We should select a toolset that is suitable for the provided JavaScript utility library and can support the requirements of Part 2.

| Tool | Proposed purpose | Decision |
|---|---|---|
| GitHub | Store the project, source code, and test files | To be agreed |
| Jest or Mocha | Execute unit tests | Choose one after examining the library |
| Chai | Provide assertions if we choose a compatible Mocha-based setup | Depends on the selected framework |
| c8 | Measure test coverage | To be evaluated |
| GitHub Actions | Automatically execute tests in a CI pipeline | Required for Part 2 |
| Coveralls | Publish and track test coverage reports | Required for Part 2 |
| Testdouble | Mock or stub dependencies when necessary | Use if justified by the selected functions |

We should avoid adding tools that do not provide a clear benefit. The final tool selection must be compatible with the library and supported by basic setup instructions.

**Agreement needed:** Decide on the testing framework, assertion library, coverage tool, and any additional dependencies after inspecting the repository.

## 6. Proposed Reporting and Quality Approach

We propose documenting test execution and defects in a consistent way.

### Test reporting

Test reports should include:

- Test execution date and software version or commit.
- Number of tests passed, failed, or skipped.
- Coverage results and important uncovered areas.
- Defects discovered and their current status.
- Any limitations or problems affecting the results.

### Defect tracking

Each defect report should include:

- A short, descriptive title.
- Steps to reproduce the problem.
- Expected and actual results.
- Relevant input data and environment information.
- Severity and priority.
- Current status and supporting evidence, when available.

### Metrics and quality gates

We should define:

- Conditions required before testing can start.
- Criteria for deciding whether a test run passes.
- Conditions for completing the planned testing.
- Coverage expectations and how uncovered code will be reviewed.
- How critical defects affect the final assessment.

The final criteria should be realistic for the selected functions and the available time. We should not claim that high code coverage alone guarantees software quality.

## 7. Proposed Additional Testing

Although Part 2 focuses on unit tests for the utility library, the test plan should discuss other testing needed for the complete e-commerce application.

We propose considering:

- **Integration testing:** Verify interactions between the frontend, REST backend, utility functions, and third-party services.
- **End-to-end testing:** Verify complete customer and producer workflows.
- **Input validation testing:** Check invalid, missing, and boundary-value inputs.
- **Security testing:** Consider authentication, authorisation, data validation, and payment-related risks.
- **Performance testing:** Evaluate product search, filtering, pagination, and large catalogues.
- **Usability and compatibility testing:** Check whether the main workflows are understandable and work across supported browsers and devices.
- **Bulk import testing:** Verify that imported products are handled consistently and invalid records are reported appropriately.

We should explain the importance and scope of these tests rather than promising to implement every type during Part 2.

## 8. Proposed Work Division

### Member 1 — Scenarios and Library Analysis

- Draft the end-to-end scenarios and UML diagrams.
- Study the utility library.
- Propose the source files for unit testing.
- Draft the testing scope, assumptions, and prioritisation.
- Describe the general principles of unit testing.
- Independently prepare the detailed test cases required for Section 5.2, without AI assistance.

### Member 2 — Tools and Quality Management

- Research and select the testing tools.
- Draft the testing environment and setup guidance.
- Describe reporting, defect tracking, metrics, and quality gates.
- Discuss additional testing for the e-commerce store.
- Draft the AI and testing discussion and document AI usage where required.

### Shared responsibilities

Both members will:

- Agree on the scenarios and prioritisation method.
- Review the final list of selected source files.
- Check the consistency of the report.
- Review references, formatting, and assignment requirements.
- Approve the final PDF before submission.

The division can be adjusted if one member has a stronger preference or more experience with a particular task.

## 9. AI Usage and Academic Integrity

The assignment explicitly prohibits AI use for designing the detailed test cases in Section 5.2. Those test cases must therefore be developed independently by the responsible student.

For other sections, we must follow the course's AI-use rules and accurately disclose any assistance used. We should verify technical claims against the actual source code and official documentation instead of relying on unverified generated content.

## 10. Decisions for Our Meeting

Before starting the full report, we should agree on the following points:

- Confirm the report structure.
- Approve or revise the three proposed scenarios.
- Decide which UML diagrams to use.
- Agree on the prioritisation method for selecting up to 10 files.
- Decide how to evaluate and select the testing tools.
- Confirm the division of work and internal deadlines.
- Agree on how to combine and review the report.
- Confirm the rules for AI usage and disclosure.
- Set a target date for the first complete draft.

### Expected outcome of the meeting

At the end of the meeting, we should have an agreed report outline, three approved scenarios, a clear method for selecting the utility library files, a preliminary tool-selection plan, and a division of responsibilities.

We can then work independently on our assigned sections and combine them into one consistent Test Plan document.

---

**Status:** Initial proposal for discussion. All proposed decisions remain subject to agreement by both group members and verification against the assignment instructions and the provided utility library.