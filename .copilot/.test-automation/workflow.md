# Test Automation Workflow

## 1. Purpose

This document defines the end-to-end workflow used by the
Test-Orchestrator to validate an implemented Azure DevOps user story
against QA-authored test cases.

The workflow coordinates:

- Azure DevOps story retrieval
- QA test case retrieval
- React source-impact analysis
- Runtime browser discovery
- Playwright test generation
- Automated test execution
- Evidence collection
- Independent validation
- Final Azure DevOps reporting

The workflow must preserve QA-authored intent and produce traceable
evidence from actual execution.

The Test-Orchestrator coordinates this workflow.

It must not:

- implement or modify application code
- invent acceptance criteria
- invent QA test cases
- silently rewrite QA test steps
- manufacture selectors
- fabricate execution results
- approve its own generated test
- approve its own execution evidence
- report PASS when required evidence is unavailable


---

## 1.1 Pre-flight: Credential Environment Check

Before Stage 1 (run initialization), verify that `TEST_AUTOMATION_USERNAME` and
`TEST_AUTOMATION_PASSWORD` exist and are non-empty. Report only set/not-set and
never print, log, or store the values or their lengths in artifacts.

If either is missing or empty, mark the run BLOCKED (prerequisite unavailable)
and stop before any browser authentication.

A failed login must not be retried automatically; stop and report BLOCKED so
the credentials can be confirmed (avoids account lockout).

---

## 2. Workflow Principles

The workflow follows these principles.

### 2.1 QA owns test intent

QA-authored test cases define:

- the scenario to validate
- the required preconditions
- the actions to perform
- the expected result

Agents may normalize test case information into structured data, but they
must not change its functional meaning.

If a test case is unclear, incomplete, or contradictory, the workflow
must stop with an appropriate blocked status.


### 2.2 Source analysis does not prove runtime behavior

React source analysis may identify:

- routes
- pages
- components
- controls
- labels
- test identifiers
- API calls
- state-management logic
- existing test utilities

Source analysis provides implementation context only.

It does not prove that:

- the deployed environment contains the analyzed code
- a component is rendered
- a control is visible
- a control is enabled
- a user can access the functionality
- the runtime UI behaves as expected


### 2.3 Knowledge Lookup

## Purpose

Knowledge Lookup provides supporting project and application context
when a specialist agent requires additional information to complete
its assigned responsibility.

Knowledge Lookup is a supporting capability.

It is not:

- the source of QA expectations
- a replacement for the Azure DevOps user story
- a replacement for QA test cases
- proof of runtime application behavior
- execution evidence
- a mechanism for determining PASS or FAIL


---

## When Knowledge Lookup May Be Used

Knowledge Lookup may be performed when an agent requires additional
context that is not available from its primary input artifacts.

Examples include:

- understanding project conventions
- locating existing automation patterns
- identifying application architecture
- locating reusable Playwright utilities
- understanding route structure
- understanding authentication setup
- locating test-data guidance
- understanding project-specific terminology
- locating existing documentation
- identifying known environment configuration
- understanding repository structure


Knowledge Lookup must be driven by a specific information need.

Do not perform unrestricted knowledge retrieval without a clear purpose.


---

## Knowledge Sources

Knowledge may be retrieved from approved project sources defined in:

.copilot/.test-automation/tools.md


Depending on the configured project, approved sources may include:

- target project repository
- project documentation
- README files
- architecture documentation
- approved knowledge repositories
- Azure DevOps project documentation
- existing Playwright automation
- existing test utilities
- project-owned configuration documentation


Only configured and approved knowledge sources may be used.


---

## Source Priority

Use authoritative workflow artifacts before supplementary knowledge.

Recommended priority:

1. QA-authored test case

2. Azure DevOps user story and acceptance criteria

3. Current execution context

4. Current target repository

5. Runtime browser discovery

6. Approved project documentation and knowledge sources

7. Existing automation and reusable project utilities


The priority depends on the question being answered.

For example:

"What should happen?"

Primary authority:

QA test case and approved requirements.


"What component implements this functionality?"

Primary authority:

current target repository.


"What is actually rendered in the deployed application?"

Primary authority:

runtime browser discovery.


"Did the expected behavior occur?"

Primary authority:

current execution result and evidence.


Knowledge Lookup must not override the authoritative source for the
question being answered.


---

## Knowledge Lookup Workflow

When additional knowledge is required:

Agent
    ↓
Identify Knowledge Gap
    ↓
Define Lookup Purpose
    ↓
Search Approved Knowledge Sources
    ↓
Evaluate Relevance
    ↓
Record Useful References
    ↓
Continue Agent Responsibility


If reliable knowledge cannot be located:

Agent
    ↓
Record Knowledge Gap
    ↓
Determine Whether Gap Is Blocking
    ↓
Return Result to Test-Orchestrator


The agent must not guess missing information.


---

## Knowledge Lookup Contract

Every material lookup should record:

{
  "purpose": "",
  "query": "",
  "sourcesSearched": [],
  "referencesUsed": [],
  "result": "FOUND | NOT_FOUND | AMBIGUOUS",
  "summary": "",
  "usedFor": ""
}


Knowledge results should be associated with the specialist artifact that
required them.

Do not treat the knowledge lookup result itself as execution evidence.


### 2.4 Runtime discovery is required for UI automation

Runtime browser discovery must be used to understand the deployed
application before generating new UI automation.

Runtime discovery should identify:

- application navigation
- rendered elements
- accessible roles
- accessible names
- stable test identifiers
- frame or iframe boundaries
- dialogs
- forms
- dynamic content
- API failures visible from the browser
- authentication or authorization barriers

Selectors must not be guessed when runtime discovery can verify them.


### 2.5 Generation and validation remain independent

The agent that generates automation must not be the final authority that
decides whether the QA expectation passed.

The Validation Agent evaluates:

- original QA intent
- expected result
- generated automation summary
- execution result
- evidence

The Validation Agent must not modify the test under validation.


### 2.6 Evidence comes from execution

Evidence must originate from actual execution or an approved source
system.

Acceptable evidence may include:

- Playwright test result
- Playwright trace
- screenshot
- video
- browser console output
- relevant network request or response details
- application-visible confirmation
- execution metadata

Evidence must never be synthesized or inferred.


---

## 3. Workflow States

Every orchestrator run must have one current workflow state.

Supported states:

```text
INITIALIZED
CONTEXT_RESOLVED
STORY_RETRIEVED
TEST_CASES_RETRIEVED
SOURCE_ANALYZED
BROWSER_DISCOVERED
AUTOMATION_READY
TEST_GENERATED
EXECUTION_COMPLETED
VALIDATION_COMPLETED
REPORTING_COMPLETED
PASSED
FAILED
BLOCKED
ERROR
```