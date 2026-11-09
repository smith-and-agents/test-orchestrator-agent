# Test Automation Gates

## 1. Purpose

This document defines the quality and safety gates enforced by the
Test-Orchestrator during automated validation of an Azure DevOps story.

Gates determine whether the workflow may progress from one stage to
the next.

Gates do not perform specialist work.

A gate evaluates artifacts produced by specialist agents and returns
a deterministic decision to the Test-Orchestrator.


---

# 2. Gate Principles

Every gate must follow these principles.

## 2.1 Gates Evaluate

A gate may:

- inspect required artifacts
- verify required fields
- verify artifact relationships
- identify missing information
- identify inconsistencies
- validate prerequisites
- return findings
- recommend the next orchestration action


## 2.2 Gates Do Not Repair

A gate must not:

- modify source artifacts
- rewrite QA expectations
- generate automation
- repair generated tests
- rediscover browser elements
- execute tests
- change execution results
- manufacture missing evidence
- determine application behavior without evidence


## 2.3 Test-Orchestrator Controls Routing

A gate returns its result to the Test-Orchestrator.

The Test-Orchestrator decides whether to:

- continue
- retry
- return to an earlier specialist
- block the affected test case
- terminate the run


---

# 3. Gate Result Contract

Every gate must return a structured result.

Recommended contract:

{
  "gate": "",
  "runId": "",
  "testCaseId": null,
  "status": "PASS | BLOCK | ERROR",
  "classification": null,
  "checkedArtifacts": [],
  "findings": [],
  "warnings": [],
  "failedConditions": [],
  "recommendedAction": "CONTINUE | RETRY | RETURN | STOP"
}


## PASS

Use when all mandatory conditions required for progression are satisfied.


## BLOCK

Use when the artifacts are valid enough to inspect, but a required
business, QA, application, environment, test-data, discovery, or
execution prerequisite is unavailable.


## ERROR

Use when the gate itself cannot reliably evaluate its required input.

Examples:

- required artifact is corrupted
- artifact cannot be parsed
- run IDs conflict
- required artifact ownership is invalid
- artifact structure prevents reliable evaluation


---

# 4. Scope of Gate Evaluation

Gate evaluation may occur at two levels:

## Run-Level Gate

Applies to the entire orchestration run.

Example:

Story cannot be retrieved.

The run cannot continue.


## Test-Case-Level Gate

Applies independently to individual QA test cases.

Example:

TC-4812 is automation-ready.

TC-4813 is blocked because required test data is unavailable.

TC-4814 is automation-ready.

A blocked test case must not automatically prevent independent test
cases from progressing.


---

# 5. Gate Sequence

The workflow uses the following primary gates:

GATE 1
TEST_CASE_AVAILABILITY

GATE 2
AUTOMATION_READINESS

GATE 3
TEST_GENERATION

GATE 4
EXECUTION_COMPLETENESS

GATE 5
VALIDATION_INTEGRITY


The workflow sequence is:

Story Reader
    ↓
QA Test Case Reader
    ↓
TEST_CASE_AVAILABILITY
    ↓
React Source Analyzer
    ↓
Browser Discovery Agent
    ↓
AUTOMATION_READINESS
    ↓
Test Generator
    ↓
TEST_GENERATION
    ↓
Test Executor
    ↓
EXECUTION_COMPLETENESS
    ↓
Validation Agent
    ↓
VALIDATION_INTEGRITY
    ↓
Final Result


---

# 6. Gate 1: TEST_CASE_AVAILABILITY

## Purpose

Confirm that sufficient QA-authored test information exists before
application analysis and automation proceed.


## Evaluated Artifacts

Required:

story.json
qa-testcases.json


## Scope

Evaluate each QA test case independently where possible.


## Mandatory Conditions

The gate must verify:

1. story.json exists
2. qa-testcases.json exists
3. artifact run IDs correspond to the active run where run IDs are
   recorded
4. at least one test case is associated with the story
5. test case ID is available
6. test case title is available
7. required test steps are available
8. required expected results are available
9. required shared steps have been resolved
10. required parameters are identifiable
11. story-to-test-case association is recorded


## PASS

PASS when the QA test case contains sufficient authoritative information
to continue toward automation.


## BLOCK Conditions

BLOCK when:

- no QA test case is associated with the story
- required steps are missing
- required expected results are missing
- required shared steps cannot be resolved
- required parameters cannot be determined
- QA expectations contradict each other
- the test case is intentionally inactive according to retrieved QA data
- a mandatory test requires unsupported non-automatable verification


## Classification

Use as appropriate:

QA_TEST_CASE_FAILURE

QA_EXPECTATION_AMBIGUOUS


## Example Result

{
  "gate": "TEST_CASE_AVAILABILITY",
  "runId": "12345-...",
  "testCaseId": 4812,
  "status": "PASS",
  "classification": null,
  "checkedArtifacts": [
    "story.json",
    "qa-testcases.json"
  ],
  "findings": [],
  "warnings": [],
  "failedConditions": [],
  "recommendedAction": "CONTINUE"
}


## Important Rule

Acceptance criteria must not be substituted for missing QA expected
results merely to make this gate pass.


---

# 7. Gate 2: AUTOMATION_READINESS

## Purpose

Determine whether the application and QA test case are sufficiently
understood to generate reliable automation.


## Evaluated Artifacts

Required:

context.json
story.json
qa-testcases.json
browser-discovery.json

When available:

source-analysis.json


## Scope

Test-case level.


## Mandatory Conditions

For each applicable test case verify:

1. QA test availability gate passed
2. required preconditions can be established
3. required application entry point is known
4. required navigation is known
5. required runtime controls have been discovered
6. required interactions can be mapped to runtime elements
7. required application states can be reached
8. required test data is available
9. authentication requirements can be satisfied
10. required runtime selectors are verified or an approved alternative
    exists
11. expected results can be observed through an approved validation
    mechanism
12. unresolved browser discovery items do not prevent execution


## Runtime Verification Rule

Source analysis alone must not satisfy runtime discovery requirements.

Example:

source-analysis.json:

{
  "testId": "add-connectivity",
  "runtimeVerified": false
}

This alone is insufficient when runtime verification is required.

Browser discovery should verify the actual deployed element.


## PASS

PASS when sufficient runtime knowledge exists to generate automation
without guessing required behavior or selectors.


## BLOCK Conditions

BLOCK when:

- required UI control cannot be discovered
- required page cannot be reached
- navigation is unresolved
- authentication cannot be completed
- required test data is unavailable
- required runtime state cannot be established
- required external dependency is unavailable
- expected result cannot be observed reliably
- human-only interaction prevents automated execution
- discovery remains incomplete after permitted recovery


## Classification

Use as appropriate:

DISCOVERY_FAILURE

ENVIRONMENT_FAILURE

ENVIRONMENT_CONFIGURATION_FAILURE

AUTHENTICATION_FAILURE

AUTHORIZATION_FAILURE

TEST_DATA_FAILURE


## Example Block Result

{
  "gate": "AUTOMATION_READINESS",
  "runId": "12345-...",
  "testCaseId": 4812,
  "status": "BLOCK",
  "classification": "DISCOVERY_FAILURE",
  "checkedArtifacts": [
    "qa-testcases.json",
    "source-analysis.json",
    "browser-discovery.json"
  ],
  "findings": [
    "Required runtime control could not be verified."
  ],
  "warnings": [],
  "failedConditions": [
    "Required interaction could not be mapped to a runtime element."
  ],
  "recommendedAction": "STOP"
}


---

# 8. Gate 3: TEST_GENERATION

## Purpose

Verify that generated Playwright automation faithfully represents the
QA test case before execution.


## Evaluated Artifacts

Required:

qa-testcases.json
browser-discovery.json
generation.json
generated Playwright test

Supporting:

source-analysis.json


## Scope

Test-case level.


## Mandatory Conditions

Verify:

1. generated test maps to exactly the intended QA test case
2. test case ID is preserved
3. required QA steps are represented
4. QA step order is preserved where order is functionally significant
5. required expected results have assertions
6. assertions are not weaker than the QA expectation
7. required runtime selectors come from verified discovery or approved
   selector policy
8. unresolved selectors are not represented as verified
9. no required QA step was silently removed
10. no assertion was intentionally suppressed
11. generated code contains no unresolved placeholder required for
    execution
12. generated automation contains no embedded credentials or secrets
13. generated test is syntactically valid
14. required project fixtures and helpers referenced by the test are
    available
15. generated test can be associated with the configured Playwright
    project


## Assertion Integrity

Example QA expectation:

Connectivity Editor is displayed.

Acceptable assertion:

expect(
  page.getByRole('heading', {
    name: 'Connectivity Editor'
  })
).toBeVisible();


A weaker assertion such as:

expect(page).toBeTruthy();

must not satisfy the QA expectation.


## Selector Integrity

A generated selector must not be accepted solely because it appears
reasonable.

Example:

button:nth-child(3)

must not be introduced simply to make generation succeed if a stable
runtime selector has not been established.


## PASS

PASS when generated automation faithfully represents the QA test case
and is technically ready for execution.


## BLOCK Conditions

BLOCK when:

- QA step is omitted
- expected result lacks an assertion
- expected behavior was weakened
- selector was fabricated
- unresolved placeholders remain
- credentials are embedded
- required fixture is unavailable
- generated automation cannot represent a required QA interaction


## ERROR Conditions

ERROR when:

- generated test cannot be parsed
- generation metadata cannot be correlated with the generated file
- artifact integrity cannot be established


## Classification

AUTOMATION_GENERATION_FAILURE


## Recovery

The Test-Orchestrator may route findings back to:

Test Generator

for one controlled correction according to workflow retry policy.

The gate itself must not repair the test.


---

# 9. Gate 4: EXECUTION_COMPLETENESS

## Purpose

Determine whether execution produced sufficient trustworthy output for
independent validation.


## Evaluated Artifacts

Required:

generation.json
execution.json

According to evidence policy:

evidence/


## Scope

Test-case level.


## Mandatory Conditions

Verify:

1. execution corresponds to the active run
2. execution corresponds to a generated test
3. generated test corresponds to the QA test case
4. execution attempt is identified
5. execution status is recorded
6. execution start information is recorded
7. execution completion information is recorded
8. failure information is retained when execution failed
9. failed step is retained when available
10. configured mandatory evidence is present
11. retry attempts remain distinguishable
12. execution information has not been silently overwritten by a later
    retry


## Important Rule

Execution completeness does not mean execution success.

Examples:

Playwright assertion failure + valid trace + required screenshot

may pass EXECUTION_COMPLETENESS.

The execution itself may still represent a functional failure.


## PASS

PASS when execution information is sufficiently complete and trustworthy
for the Validation Agent to inspect.


## BLOCK Conditions

BLOCK when execution cannot complete due to:

- application environment becoming unavailable
- required test data becoming unavailable
- authentication becoming unavailable
- required external dependency becoming unavailable


## ERROR Conditions

ERROR when:

- execution result is corrupted
- execution output cannot be associated with the generated test
- mandatory evidence expected from completed execution is missing
- artifact integrity cannot be established


## Classification

Use as appropriate:

AUTOMATION_INFRASTRUCTURE_FAILURE

ENVIRONMENT_FAILURE

TEST_DATA_FAILURE

AUTHENTICATION_FAILURE

EVIDENCE_FAILURE

UNKNOWN_FAILURE


## Critical Rule

An unsuccessful test execution must not automatically cause this gate
to fail.

The purpose of the gate is:

"Do we have enough reliable execution information for validation?"

not:

"Did the application pass the test?"


---

# 10. Gate 5: VALIDATION_INTEGRITY

## Purpose

Verify that the independent validation result is supported by the QA
expectation and execution evidence.


## Evaluated Artifacts

Required:

qa-testcases.json
generation.json
execution.json
validation.json

Evidence:

evidence/


## Scope

Test-case level followed by run-level aggregation.


## Mandatory Conditions

Verify:

1. every executed applicable test case has a validation result
2. validation references the correct QA test case
3. original expected result is preserved
4. observed result is identified
5. validation result uses a supported status
6. PASS contains sufficient execution evidence
7. FAIL identifies the unmet QA expectation
8. BLOCKED identifies the unresolved prerequisite
9. INCONCLUSIVE identifies the evidence gap
10. evidence references belong to the active run
11. validation did not modify execution facts
12. validation did not modify QA expectations
13. validation did not modify generated automation
14. automation failure was not mislabeled as an application defect
15. environment failure was not mislabeled as an application defect


## Supported Validation Results

PASS

FAIL

BLOCKED

INCONCLUSIVE


## PASS Integrity

For a validation result of PASS:

The gate must establish that:

QA Expected Result
        +
Executed QA-Mapped Action
        +
Observed Result
        +
Execution Evidence

support the PASS result.

Absence of an error is not sufficient by itself.


## FAIL Integrity

For FAIL:

The result must identify:

- expected behavior
- observed behavior
- relevant execution evidence


## BLOCKED Integrity

For BLOCKED:

The result must identify the unresolved prerequisite.


## INCONCLUSIVE Integrity

For INCONCLUSIVE:

The result must identify why existing evidence cannot reliably support
PASS or FAIL.


## Gate PASS

The gate passes when the validation record is internally consistent,
traceable, and evidence-supported.

This does not mean that the application passed.

Example:

validation.json:

{
  "testCaseId": 4812,
  "result": "FAIL"
}

may pass VALIDATION_INTEGRITY when the FAIL verdict is properly
supported by evidence.


## Gate ERROR

Use when validation cannot be trusted.

Classification:

VALIDATION_INTEGRITY_FAILURE


## Critical Rule

A validation integrity failure must never be transformed into a
functional PASS.


---

# 11. Partial Gate Results

Test cases should be evaluated independently whenever possible.

Example:

TC-4812
AUTOMATION_READINESS = PASS

TC-4813
AUTOMATION_READINESS = BLOCK

TC-4814
AUTOMATION_READINESS = PASS


The Test-Orchestrator may continue with:

TC-4812
TC-4814

while preserving TC-4813 as blocked.


The final run result must account for every required applicable test
case.


---

# 12. Warning Versus Failure

Not every issue should block progression.


## Warning

Use a warning when:

- information is incomplete but does not compromise test correctness
- a non-critical source mapping could not be established
- optional evidence could not be captured
- an approved fallback selector was used
- an existing reusable helper could not be found


Warnings must remain visible in downstream reporting.


## Failed Condition

Use a failed condition when continuing could:

- change QA intent
- require guessing application behavior
- require guessing a runtime selector
- produce unreliable execution
- produce insufficient validation evidence
- expose secret information
- create an unreliable PASS/FAIL result


---

# 13. Retry Interaction

Gates do not automatically retry.

A failed gate returns findings to the Test-Orchestrator.

The Test-Orchestrator consults workflow.md to determine allowed
recovery.


## Example: Discovery problem

Browser Discovery Agent
    ↓
AUTOMATION_READINESS
    ↓
BLOCK: DISCOVERY_FAILURE
    ↓
Test-Orchestrator
    ↓
Approved rediscovery if applicable


## Example: Generation problem

Test Generator
    ↓
TEST_GENERATION
    ↓
BLOCK: AUTOMATION_GENERATION_FAILURE
    ↓
Test-Orchestrator
    ↓
Test Generator correction
    ↓
TEST_GENERATION


## Example: Environment problem

Test Executor
    ↓
EXECUTION_COMPLETENESS
    ↓
BLOCK: ENVIRONMENT_FAILURE
    ↓
Test-Orchestrator
    ↓
STOP


Recovery must obey retry limits defined by workflow.md.


---

# 14. Artifact Freshness and Consistency

A gate must reject or block progression when its required input
artifacts do not represent the same effective execution context.

Examples of material changes:

- QA test case revision changed
- repository commit changed
- target deployment changed
- browser discovery belongs to another run
- generated test belongs to an earlier discovery
- execution belongs to an earlier generated test


The Test-Orchestrator determines invalidation according to workflow.md.

A gate must not combine incompatible artifacts merely because all
required files exist.


---

# 15. Security Gate Conditions

Every gate that evaluates generated content or evidence must detect
known violations relevant to its stage.

Block progression when execution requires:

- embedded plaintext passwords
- embedded API keys
- embedded bearer tokens
- unauthorized credential storage


Do not publish evidence that exposes:

- passwords
- API keys
- access tokens
- session cookies
- restricted user information


If evidence contains sensitive information, route according to the
approved evidence-handling policy before publishing or attaching it.


---

# 16. Gate Ownership

The Test-Orchestrator invokes and enforces gates.

Specialist agents do not mark their own gates as passed.

Example:

Test Generator
    ↓
generation.json
    ↓
returns to Test-Orchestrator
    ↓
TEST_GENERATION gate
    ↓
PASS | BLOCK | ERROR


The specialist's own status:

SUCCESS

does not automatically mean:

GATE = PASS


---

# 17. Gate Decision Matrix

## TEST_CASE_AVAILABILITY

Purpose:
Can this QA test case safely proceed?

Required primary artifacts:

- story.json
- qa-testcases.json

PASS routes to:

React Source Analyzer


## AUTOMATION_READINESS

Purpose:
Do we understand enough of the runtime application to automate the QA
test safely?

Required primary artifacts:

- qa-testcases.json
- browser-discovery.json

Supporting:

- context.json
- story.json
- source-analysis.json

PASS routes to:

Test Generator


## TEST_GENERATION

Purpose:
Does the generated automation accurately represent QA intent and use
verified application knowledge?

Required primary artifacts:

- qa-testcases.json
- generation.json
- generated test
- browser-discovery.json

PASS routes to:

Test Executor


## EXECUTION_COMPLETENESS

Purpose:
Do we have sufficient trustworthy execution information for independent
validation?

Required primary artifacts:

- generation.json
- execution.json
- required evidence

PASS routes to:

Validation Agent


## VALIDATION_INTEGRITY

Purpose:
Is the final validation verdict traceable and supported by evidence?

Required primary artifacts:

- qa-testcases.json
- generation.json
- execution.json
- validation.json
- evidence

PASS routes to:

Final Result Calculation


---

# 18. Gate Evaluation Example

Consider:

QA Test Case:

TC-4812

Step:
Click Add Connectivity

Expected:
Connectivity Editor is displayed


## Before Browser Discovery

Source Analyzer finds:

data-testid="add-connectivity"

but:

runtimeVerified=false


AUTOMATION_READINESS:

BLOCK

Reason:

Required runtime interaction has not yet been verified.


## After Browser Discovery

Browser Discovery observes:

role=button
accessibleName=Add Connectivity
visible=true
enabled=true
runtimeVerified=true


AUTOMATION_READINESS:

PASS


## Generated Test

Generated automation:

await page
  .getByRole('button', {
    name: 'Add Connectivity'
  })
  .click();

await expect(
  page.getByRole('heading', {
    name: 'Connectivity Editor'
  })
).toBeVisible();


TEST_GENERATION:

PASS


## Execution

Playwright successfully clicks the button.

The expected heading does not appear.

Screenshot and trace are captured.


EXECUTION_COMPLETENESS:

PASS


This gate passes because execution evidence is complete.

It does not mean the test passed.


## Validation

Expected:

Connectivity Editor is displayed.

Observed:

Connectivity Editor heading did not appear.

Evidence:

Execution screenshot and trace.


Validation result:

FAIL


VALIDATION_INTEGRITY:

PASS


The gate passes because the FAIL verdict is properly supported.


Final functional result:

FAILED