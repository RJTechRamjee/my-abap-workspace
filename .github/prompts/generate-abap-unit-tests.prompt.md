---
description: "Generate an ABAP Unit test class (ltcl_*) with test doubles for the selected class or method, following AAA pattern."
agent: "agent"
argument-hint: "Class/method to test, e.g. 'ZRK_CL_ORDER_MANAGER->CHECK_STOCK'"
---
Generate ABAP Unit tests for: ${input:testTarget:Class and method(s) to test}

Follow [clean-abap.instructions.md](../instructions/clean-abap.instructions.md) testing conventions.

Pick the framework from the code under test:
- RAP behavior → `cl_abap_behv_test_environment`; EML `MODIFY`/`READ` in tests; assert on
  `reported`/`failed` for validations, on a follow-up `READ` for determinations, on the result +
  state for actions.
- CDS view → `cl_cds_test_environment=>create( )`; insert test doubles for every data source;
  test each DCL branch.
- Class with released-CDS/OSQL reads → `cl_osql_test_environment=>create( )`.
- Class with interface collaborators → `cl_abap_testdouble=>create( )`, injected via constructor.

Rules:
1. Create (or extend) a local test class `ltcl_<name>` (`FOR TESTING`, `DURATION SHORT`,
   `RISK LEVEL HARMLESS` unless the code under test needs otherwise).
2. One test method per business rule (if numbered, R1/R2…) plus boundary and negative paths;
   method names read as sentences (`<method>_<scenario>_<expectation>`).
3. Given/When/Then delimited in each method; `cl_abap_unit_assert` with a `msg` naming the rule.
4. Set up doubles in `setup`/`class_setup`, tear down cleanly; no shared mutable state;
   deterministic (freeze or inject date/time/random) — no real DB, no `WAIT`, no RFC.
5. Show the generated test code, end with a rule → test-method coverage table, and mark any
   uncovered rule `[GAP]` — do not leave an empty assertion in a method that claims to test a rule.
6. Flag any production code that needs refactoring (e.g., to inject dependencies) before it can
   be tested in isolation — don't refactor silently.

---
**Chain**: `create-rap-bo` → **generate-abap-unit-tests** → `atc-fix` → `pre-transport-check`
