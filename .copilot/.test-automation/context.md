# Test Automation Context

## Purpose

This file defines the shared execution context used by the Test-Orchestrator
and all specialist agents.

The context is the common contract between agents.

Agents must exchange verified information through structured context rather
than relying on assumptions or conversational memory.

The context must preserve traceability between:

Azure DevOps Story
    ↓
Acceptance Criteria
    ↓
QA Test Cases
    ↓
Source Analysis
    ↓
Browser Discovery
    ↓
Generated Automation
    ↓
Execution Evidence
    ↓
Independent Validation


# Test Automation Project Structure

All generated Playwright test cases must use TypeScript.

Required language:

- TypeScript (`.ts`)

Do not generate Playwright test cases using:

- JavaScript
- Python
- Java
- C#


## Repository Structure

The `.test-automation-project` directory is reserved for generated
test-automation artifacts and test cases.

Node.js project dependencies and Playwright configuration must remain
in the parent repository folder.

Expected structure:

<repository-root>/
│
├── package.json
├── package-lock.json
├── node_modules/
├── playwright.config.ts
├── tsconfig.json
│
├── .test-automation-project/
│   ├── artifacts/
│   │   ├── story.md
│   │   ├── qa-testcases.json
│   │   ├── source-analysis.json
│   │   ├── browser-discovery.json
│   │   ├── execution.json
│   │   └── validation.json
│   │
│   ├── tests/
│   │   ├── story-<id>/
│   │   │   └── <test-case-id>.spec.ts
│   │   │
│   │   └── generated/
│   │
│   └── evidence/
│       ├── screenshots/
│       ├── traces/
│       └── logs/
│
└── application source...


## Node.js Project Ownership

The parent repository is the Node.js and Playwright project root.

The following must exist only at the parent project level:

- package.json
- package-lock.json
- node_modules/
- playwright.config.ts
- tsconfig.json
- other project-level TypeScript or Playwright configuration files

Agents must NOT create these files inside:

`.test-automation-project/`


## Dependency Management

Generated tests must use the dependencies defined by the parent
`package.json`.

Before creating additional dependencies:

1. Inspect the existing parent `package.json`.
2. Reuse existing dependencies whenever possible.
3. Do not create a separate Node.js project inside
   `.test-automation-project`.
4. Do not run `npm init` inside `.test-automation-project`.
5. Do not create another `node_modules` directory inside
   `.test-automation-project`.


## Playwright Configuration

Generated tests must use the parent project's:

`playwright.config.ts`

Do not generate another Playwright configuration inside
`.test-automation-project`.

Test execution must resolve Playwright configuration from the parent
project.


## TypeScript Configuration

Generated test cases must be compatible with the parent project's
TypeScript configuration.

Use:

`tsconfig.json`

from the parent project.

Do not create a separate `tsconfig.json` inside
`.test-automation-project` unless explicitly required by the user.


## Generated Test Location

Generated Playwright tests must be written under:

`.test-automation-project/tests/`

Recommended naming:

`.test-automation-project/tests/story-<story-id>/<test-case-id>.spec.ts`

Example:

`.test-automation-project/tests/story-12345/TC-67890.spec.ts`


## Rules

- All generated Playwright test cases must use TypeScript.
- Generated Playwright test files must use the `.spec.ts` extension.
- `.test-automation-project` is not a standalone Node.js project.
- Do not create `package.json` inside `.test-automation-project`.
- Do not create `node_modules` inside `.test-automation-project`.
- Do not create `playwright.config.ts` inside `.test-automation-project`.
- Do not create `tsconfig.json` inside `.test-automation-project`.
- Reuse the parent project's Node.js dependencies.
- Reuse the parent project's Playwright configuration.
- Reuse the parent project's TypeScript configuration.


# Core Principles

## 1. Source of Truth

Information must retain its original source.

Examples:

- Story requirements → Azure DevOps Story
- Acceptance criteria → Azure DevOps Story
- Test expectations → QA Test Case
- implementation details → source repository
- rendered UI behavior → browser discovery
- automation behavior → generated Playwright test
- execution result → Test Executor
- final assessment → Validation Agent

An agent must never replace authoritative information with an assumption.


## 2. Do Not Invent Missing Context

If required information is unavailable:

- mark the field as unavailable
- record the reason when known
- invoke the appropriate gate
- do not manufacture a value

Examples of information that must not be invented:

- acceptance criteria
- test cases
- expected results
- routes
- selectors
- element names
- test data
- credentials
- application URLs
- execution results
- screenshots
- traces


## 3. Preserve QA Intent

QA-authored test cases and expected results must remain unchanged.

Agents may normalize their representation for automation purposes, but they
must not change the functional meaning.

If a test case cannot be automated as written, record the problem rather than
silently rewriting the test case.


## 4. Separate Observed Facts from Derived Information

Context should distinguish between:

### Authoritative

Retrieved directly from a system of record.

Examples:

- Azure DevOps story
- QA test case
- repository source

### Observed

Collected from actual runtime behavior.

Examples:

- rendered button
- accessible name
- current URL
- API response
- browser state

### Derived

Produced by analysis.

Examples:

- impacted React components
- probable navigation path
- proposed automation strategy

Derived information must never be represented as authoritative or observed.


# Execution Context

Each validation run should have a unique execution context.

Recommended logical structure:

.test-automation/runs/<story-id>/

    run.json
    story.json
    qa-testcases.json
    source-analysis.json
    browser-discovery.json
    generation.json
    execution.json
    validation.json

    evidence/
        screenshots/
        traces/
        logs/


# Run Context

File:

run.json

Purpose:

Contains execution-level information shared across the workflow.

Recommended structure:

{
  "runId": "",
  "storyId": "",
  "project": "",
  "environment": "",
  "applicationUrl": "",
  "startedBy": "",
  "status": "",
  "currentStage": "",
  "target": {
    "environment": "",
    "applicationUrl": ""
  },
  "authentication": {
    "required": true,
    "usernameEnvironmentVariable": "TEST_AUTOMATION_USERNAME",
    "passwordEnvironmentVariable": "TEST_AUTOMATION_PASSWORD"
  },
}

Do not assume missing configuration.

Fields that cannot be resolved should remain unavailable until provided or
retrieved from an approved source.


# Story Context

File:

story.json

Owner:

Story Reader

Purpose:

Represents the Azure DevOps story being validated.

Recommended structure:

{
  "storyId": "",
  "title": "",
  "description": "",
  "acceptanceCriteria": [],
  "linkedWorkItems": [],
  "source": "Azure DevOps"
}

The Story Reader owns this context.

Other agents may read it but must not change the original story intent.


# QA Test Case Context

File:

qa-testcases.json

Owner:

QA Test Case Reader

Purpose:

Stores QA-authored test expectations.

Recommended structure:

{
  "storyId": "",
  "testCases": [
    {
      "testCaseId": "",
      "title": "",
      "preconditions": [],
      "steps": [
        {
          "stepNumber": 1,
          "action": "",
          "expectedResult": ""
        }
      ],
      "parameters": {},
      "configuration": {}
    }
  ],
  "source": "Azure DevOps Test Plans"
}

QA expectations are authoritative.

No downstream agent may rewrite expected results simply to make automation
generation or execution succeed.


# Source Analysis Context

File:

source-analysis.json

Owner:

React Source Analyzer

Purpose:

Provides implementation knowledge that may help automation generation and
browser discovery.

Recommended structure:

{
  "storyId": "",
  "repository": "",
  "branch": "",
  "commit": "",
  "impactedFiles": [],
  "routes": [],
  "components": [],
  "controls": [],
  "apiInteractions": [],
  "existingTestIds": [],
  "existingAutomation": [],
  "analysisNotes": []
}

Example:

{
  "routes": [
    "/connectivity"
  ],
  "components": [
    "ConnectivityEditor"
  ],
  "controls": [
    {
      "component": "AddConnectivityButton",
      "testId": "add-connectivity"
    }
  ]
}

Source analysis describes implementation knowledge.

It does not prove that an element is rendered or available in the deployed
application.


# Browser Discovery Context

File:

browser-discovery.json

Owner:

Browser Discovery Agent

Purpose:

Records runtime-observed application behavior needed for automation.

Recommended structure:

{
  "applicationUrl": "",
  "environment": "",
  "pages": [],
  "navigation": [],
  "elements": [],
  "dialogs": [],
  "networkObservations": [],
  "discoveryIssues": []
}

Example element:

{
  "page": "Connectivity",
  "role": "button",
  "accessibleName": "Add Connectivity",
  "testId": "add-connectivity",
  "observed": true
}

Browser-discovered information must originate from actual runtime inspection.

Never invent an element or selector because source code suggests that it
should exist.


# Locator Context

The automation layer should prefer stable locator information discovered at
runtime or explicitly provided by the application.

Preferred locator order:

1. Explicit stable test identifier where appropriate
2. Accessible role + accessible name
3. Label
4. Placeholder
5. Text when sufficiently stable
6. Stable CSS locator when unavoidable

Avoid fragile selectors based primarily on:

- dynamic generated classes
- DOM position
- deep CSS hierarchy
- nth-child
- transient element IDs

A source-code identifier must not automatically be treated as a valid runtime
locator.


# Test Generation Context

File:

generation.json

Owner:

Test Generator

Purpose:

Records generated automation and its relationship to the original QA test.

Recommended structure:

{
  "testCaseId": "",
  "generatedTest": "",
  "framework": "Playwright",
  "mappedSteps": [],
  "requiredTestData": [],
  "generationIssues": []
}

Each generated action should remain traceable to the corresponding QA step.

Example:

{
  "qaStep": 2,
  "action": "Click Add Connectivity",
  "automationAction":
    "getByRole('button', { name: 'Add Connectivity' }).click()"
}

Generation must not change expected results.


# Execution Context

File:

execution.json

Owner:

Test Executor

Purpose:

Contains facts from actual test execution.

Recommended structure:

{
  "testCaseId": "",
  "status": "",
  "startedAt": "",
  "completedAt": "",
  "steps": [],
  "failure": null,
  "evidence": []
}

Example execution step:

{
  "qaStep": 2,
  "status": "PASSED",
  "action": "Click Add Connectivity",
  "evidence": []
}

Execution status must reflect actual execution.

Never report PASS when execution did not occur.


# Evidence Context

Evidence must originate from actual execution.

Supported evidence may include:

- screenshot
- Playwright trace
- execution log
- browser console output
- relevant network information

Example:

{
  "type": "screenshot",
  "path": "evidence/screenshots/test-4812-step-3.png",
  "testCaseId": "4812",
  "step": 3
}

Evidence should be linked to the test case and, where applicable, the
specific QA step.


# Validation Context

File:

validation.json

Owner:

Validation Agent

Purpose:

Records the independent assessment of QA expectation versus execution
evidence.

Recommended structure:

{
  "storyId": "",
  "testCases": [
    {
      "testCaseId": "",
      "result": "",
      "validatedSteps": [],
      "failureClassification": "",
      "reason": ""
    }
  ],
  "overallResult": ""
}

The Validation Agent evaluates:

QA Expectation
        +
Expected Result
        +
Actual Execution
        +
Evidence

The Validation Agent must not modify the generated automation or evidence
being evaluated.


# Failure Classification Context

Use consistent failure classifications across agents.

Allowed classifications:

TEST_FAILURE

    Application behavior did not meet the QA expectation.

AUTOMATION_FAILURE

    Automation failed independently of demonstrated application behavior.

DISCOVERY_FAILURE

    Required runtime UI/application information could not be discovered.

ENVIRONMENT_FAILURE

    The target environment prevented reliable execution.

TEST_DATA_FAILURE

    Required test data was unavailable or invalid.

BLOCKED

    A required prerequisite prevented validation.

Failure classifications must be supported by available execution context.


# Context Ownership

Each specialist owns only its output context.

Story Reader
    → story.json

QA Test Case Reader
    → qa-testcases.json

React Source Analyzer
    → source-analysis.json

Browser Discovery Agent
    → browser-discovery.json

Test Generator
    → generation.json

Test Executor
    → execution.json
    → evidence/

Validation Agent
    → validation.json

Test-Orchestrator
    → run.json
    → workflow state


# Context Mutation Rules

Agents must not overwrite another specialist's authoritative output.

For example:

Test Generator MUST NOT modify:

- story.json
- qa-testcases.json
- browser-discovery.json

Test Executor MUST NOT modify:

- generated QA expectations
- original test steps
- source analysis

Validation Agent MUST NOT modify:

- generated tests
- execution results
- evidence

If conflicting information is discovered, record the conflict rather than
rewriting the original context.


# Minimum Context Required by Agent

## Story Reader

Input:

- Story ID
- Azure DevOps configuration


## QA Test Case Reader

Input:

- Story ID
- Story context


## React Source Analyzer

Input:

- Story context
- repository
- branch or commit


## Browser Discovery Agent

Input:

- QA test cases
- application URL
- environment

Optional supporting input:

- source analysis


## Test Generator

Required:

- QA test cases
- browser discovery

Supporting:

- story context
- source analysis


## Test Executor

Required:

- generated automation
- target environment
- application URL
- required test data


## Validation Agent

Required:

- original QA test case
- expected results
- execution result
- execution evidence


# Conflict Resolution

When information conflicts, do not silently choose whichever value makes
automation easiest.

Use the following authority model according to the type of information:

Requirement question
    → Story / Acceptance Criteria

Expected behavior question
    → QA Test Case expected result

Implementation question
    → Repository source

Runtime UI question
    → Browser Discovery

Execution outcome question
    → Test Executor evidence

Final QA expectation assessment
    → Validation Agent


# Security Context

Never persist secrets into shared context files.

Do not store:

- passwords
- API keys
- tokens
- session cookies
- connection secrets
- authentication headers

Store only an approved reference or identifier to a secret when required.

Sensitive browser state should not be copied into execution artifacts unless
explicitly required and permitted.

# Credential Handling

Authentication secrets must never become part of shared test-automation
context.

Credentials must be resolved at runtime from approved environment variables.

Default credential variables:

- TEST_AUTOMATION_USERNAME
- TEST_AUTOMATION_PASSWORD

Agents may know the environment-variable names but must not persist their
resolved values.

Credentials must not be written to:

- run.json
- story.json
- qa-testcases.json
- source-analysis.json
- browser-discovery.json
- generation.json
- execution.json
- validation.json
- Playwright test source
- logs
- screenshots
- traces
- console output

The Browser Discovery Agent and Test Executor may use credentials only when
required for authentication.

If required credentials cannot be resolved, authentication-dependent
execution must stop and return:

BLOCKED

Reason:

Required authentication credentials are unavailable.


# Context Cleanup

Context should contain only information needed to reproduce, explain, or
validate the execution.

Do not persist:

- unnecessary conversational history
- speculative reasoning
- duplicate source material
- secrets
- temporary browser state unrelated to evidence

The final context should make it possible to answer:

1. What story was validated?
2. Which QA test case was used?
3. What did QA expect?
4. What implementation and runtime information was discovered?
5. What automation was generated?
6. What actually executed?
7. What evidence was produced?
8. Why did validation pass, fail, or become blocked?