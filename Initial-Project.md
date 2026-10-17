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

