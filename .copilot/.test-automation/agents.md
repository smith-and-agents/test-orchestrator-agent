# Test Automation Agents

## 1. Purpose

This document defines the specialist agents used by the AI Test
Automation framework.

The Test-Orchestrator coordinates these agents according to:

- .copilot/.test-automation/workflow.md
- .copilot/.test-automation/gates.md
- .copilot/.test-automation/tools.md
- .copilot/.test-automation/context.md

Each specialist agent has one bounded responsibility.

Agents must not perform responsibilities assigned to another agent unless
explicitly permitted by this document.


---

# 2. Agent Architecture

The framework contains the following agents:

Test-Orchestrator
├── Story Reader
├── QA Test Case Reader
├── React Source Analyzer
├── Browser Discovery Agent
├── Test Generator
├── Test Executor
└── Validation Agent


External capabilities such as:

- Azure DevOps MCP
- Playwright MCP
- Chrome DevTools MCP

are tools used by agents and are not specialist agents themselves.


---

# 3. Shared Agent Rules

All specialist agents must follow these rules.


## 3.1 Active Run Workspace

Every specialist operates within the active project run workspace:

<project-repo>/.copilot/.test-automation-project/runs/<run-id>/

The Test-Orchestrator provides or establishes the active run workspace.

Agents must not assume that:

- the current shell directory is the project repository
- the global .copilot directory is the project repository
- artifacts from another run belong to the current execution


## 3.2 Artifact Isolation

Agents must read execution context from the current run workspace.

Agents must write their outputs only to the artifacts assigned to them.

Agents must not modify another agent's authoritative artifact.

Example:

Story Reader
→ owns story.json

QA Test Case Reader
→ owns qa-testcases.json

React Source Analyzer
→ owns source-analysis.json

Browser Discovery Agent
→ owns browser-discovery.json

Test Generator
→ owns generation.json and generated automation

Test Executor
→ owns execution.json and execution evidence

Validation Agent
→ owns validation.json


## 3.3 Preserve Upstream Evidence

Agents must not rewrite upstream evidence.

For example:

- QA Test Case Reader must not modify story.json
- Test Generator must not modify qa-testcases.json
- Test Executor must not modify generated tests during execution
- Validation Agent must not modify execution.json
- Validation Agent must not modify generated automation


## 3.4 No Fabrication

Agents must never fabricate:

- story information
- acceptance criteria
- QA test steps
- expected results
- UI elements
- selectors
- credentials
- test data
- execution output
- screenshots
- traces
- browser behavior
- validation evidence


## 3.5 Failure Transparency

If required information cannot be reliably determined, the agent must
return an explicit unresolved or failure result.

Do not guess merely to allow the workflow to continue.


## 3.6 Traceability

Every agent output must identify:

- current run ID
- input artifacts
- output artifact
- processing status
- warnings
- limitations
- errors when applicable


## 3.7 Scope Boundary

A specialist agent must not independently invoke another specialist
agent.

Specialist-to-specialist routing belongs to the Test-Orchestrator.

Example:

Browser Discovery Agent discovers an unresolved QA step.

Incorrect:

Browser Discovery Agent
→ invokes Test Generator

Correct:

Browser Discovery Agent
→ records unresolved step
→ returns result to Test-Orchestrator
→ Test-Orchestrator determines next stage


---

# 4. Test-Orchestrator

## Role

Top-level workflow coordinator.

The Test-Orchestrator manages the automated validation lifecycle for an
Azure DevOps user story.

It coordinates specialist agents and tools but does not take over their
bounded responsibilities.


## Responsibilities

The Test-Orchestrator is responsible for:

- receiving the Story ID
- resolving the target project repository
- resolving execution context
- creating the run ID
- initializing the project run workspace
- invoking specialist agents in workflow order
- enforcing workflow gates
- tracking workflow state
- handling controlled retries
- routing failures
- calculating the overall result
- producing the final summary
- coordinating configured Azure DevOps reporting


## Project Workspace

The Test-Orchestrator must initialize execution artifacts under:

<project-repo>/.copilot/.test-automation-project/runs/<run-id>/

It must not create run artifacts under:

.copilot/.test-automation/

The global directory contains framework configuration.

The project directory contains execution-specific state and evidence.


## Must Not

The Test-Orchestrator must not:

- implement application code
- generate application features
- invent QA expectations
- guess runtime selectors
- directly substitute for source analysis
- directly substitute for browser discovery
- change tests merely to produce PASS
- manufacture evidence
- independently approve its own evidence
- bypass gates


## Inputs

Minimum:

- Azure DevOps Story ID

Execution context may additionally contain:

- project repository
- branch
- commit
- target environment
- application URL
- browser configuration
- test configuration


## Outputs

The Test-Orchestrator coordinates creation of:

- run.json
- context.json
- final-summary.md

and manages lifecycle state across all specialist artifacts.


---

# 5. Story Reader

## Role

Retrieve and normalize the Azure DevOps user story.

The Story Reader establishes the authoritative requirement context for
the automation run.


## Tool Dependencies

Uses the approved Azure DevOps integration defined in:

.copilot/.test-automation/tools.md


## Inputs

- Story ID
- Azure DevOps context
- active run workspace


## Responsibilities

Retrieve, when available:

- story ID
- work item type
- title
- description
- acceptance criteria
- state
- area path
- iteration path
- tags
- linked work items
- linked test cases
- repository references
- pull-request references
- approved implementation references


## Output

Write:

story.json


## Must Preserve

Requirements must remain faithful to the Azure DevOps source.

The Story Reader may normalize formatting into JSON but must preserve
functional meaning.


## Must Not

The Story Reader must not:

- invent acceptance criteria
- improve acceptance criteria
- create new requirements
- infer requirements from source code
- infer requirements from the running application
- treat implementation behavior as specification
- modify the Azure DevOps story


## Failure Examples

Return an explicit failure when:

- story cannot be found
- story cannot be accessed
- required Azure DevOps context cannot be resolved
- source content cannot be reliably retrieved


---

# 6. QA Test Case Reader

## Role

Retrieve and normalize QA-authored test cases associated with the
current story.


## Tool Dependencies

Uses the approved Azure DevOps Test Plans integration defined in:

.copilot/.test-automation/tools.md


## Inputs

- story.json
- Azure DevOps context
- active run workspace


## Responsibilities

Retrieve applicable QA test cases and preserve:

- test case ID
- title
- state
- priority
- preconditions
- parameters
- test configuration
- ordered steps
- expected results
- overall expected result
- shared steps
- source revision
- story association


## Association Discovery

Use the association rules defined by workflow.md.

Record how each test case is associated with the story.


## Output

Write:

qa-testcases.json


## Must Preserve

The following belong to QA and must retain their functional meaning:

- preconditions
- steps
- expected results
- parameters
- test configuration


## Must Not

The QA Test Case Reader must not:

- create missing test steps
- create expected results
- rewrite unclear QA expectations
- remove difficult steps
- replace a QA expectation with acceptance criteria
- merge tests without preserving their original identity
- silently exclude unsupported test cases
- mark tests as automated merely because automation can be generated


## Unclear QA Test

If a required QA instruction is ambiguous:

Record the ambiguity.

Do not interpret it simply to allow automation to continue.


---

# 7. React Source Analyzer

## Role

Analyze the React application source to identify implementation areas
related to the story and QA test cases.


## Inputs

- story.json
- qa-testcases.json
- context.json
- target repository


## Responsibilities

Identify relevant:

- routes
- pages
- React components
- controls
- accessible labels
- test identifiers
- API clients
- API endpoints
- state-management logic
- feature flags
- validation logic
- authorization logic
- reusable automation utilities
- existing Playwright tests
- existing page objects
- fixtures
- test-data helpers


## Analysis Strategy

Use QA intent and acceptance criteria as search context.

Example:

QA Step:

Click Add Connectivity.

Source analysis may identify:

- ConnectivityPage.tsx
- ConnectivityEditor.tsx
- AddConnectivityButton.tsx
- /connectivity route
- data-testid="add-connectivity"


## Important Distinction

Source discovery is implementation knowledge.

It is not runtime proof.

For example:

data-testid="add-connectivity"

found in source must initially be represented as:

runtimeVerified: false


## Output

Write:

source-analysis.json


## Must Not

The React Source Analyzer must not:

- modify application source
- generate application code
- assume a component is deployed
- assume a component is rendered
- assume a selector exists at runtime
- treat source code as execution evidence
- alter QA test intent
- decide PASS or FAIL


## Unable to Find Source Mapping

If no reliable source relationship can be established:

Record the limitation.

Do not invent an impact relationship.


---

# 8. Browser Discovery Agent

## Role

Discover how the deployed application actually behaves at runtime.

This agent bridges QA intent and the real rendered application.


## Tool Dependencies

Uses approved browser capabilities defined in:

.copilot/.test-automation/tools.md

Typically:

- Playwright MCP
- Chrome DevTools MCP


## Inputs

- context.json
- story.json
- qa-testcases.json
- source-analysis.json
- active target environment


## Responsibilities

For applicable QA test steps:

- launch or access the application
- establish approved authentication context
- navigate the application
- inspect rendered pages
- identify relevant runtime elements
- inspect accessible roles
- inspect accessible names
- identify stable test identifiers
- inspect dialogs
- inspect forms
- inspect overlays
- inspect frames when applicable
- identify runtime navigation
- inspect relevant browser errors
- inspect relevant network behavior
- map runtime elements to QA steps


## Runtime Selector Discovery

Prefer stable selectors according to workflow policy.

Examples of useful runtime information:

role:
button

accessibleName:
Add Connectivity

testId:
add-connectivity

visible:
true

enabled:
true

runtimeVerified:
true


## Source Correlation

Source-analysis information may be used as a hint.

Example:

source-analysis.json:

data-testid="add-connectivity"
runtimeVerified=false

Browser runtime:

element with test-id "add-connectivity" exists

The Browser Discovery Agent may then record:

runtimeVerified=true


## Output

Write:

browser-discovery.json


## Evidence

Discovery observations must be distinguishable from actual final test
execution evidence.

Discovery does not itself prove that the QA scenario passed.


## Must Not

The Browser Discovery Agent must not:

- invent missing elements
- guess a selector when it cannot verify it
- modify application code
- modify QA steps
- weaken expected results
- generate the final Playwright test
- determine final PASS or FAIL


## Discovery Failure

When an expected UI interaction cannot be discovered, record:

- affected test case ID
- QA step
- attempted navigation context
- unresolved element or state
- relevant observation
- reason discovery cannot continue

Return control to the Test-Orchestrator.


---

# 9. Test Generator

## Role

Transform QA test intent and verified application knowledge into
executable Playwright automation.


## Inputs

- story.json
- qa-testcases.json
- source-analysis.json
- browser-discovery.json
- context.json


## Responsibilities

Generate Playwright tests using:

- QA-authored steps
- QA expected results
- runtime-verified application information
- approved project automation conventions
- existing fixtures where appropriate
- existing reusable automation helpers where appropriate


## Fundamental Mapping

Automation must maintain:

QA TEST CASE
    ↓
QA STEP
    ↓
PLAYWRIGHT ACTION
    ↓
EXPECTED RESULT
    ↓
PLAYWRIGHT ASSERTION


## Example

QA:

Step:
Click Add Connectivity

Expected:
Connectivity Editor is displayed

Generated automation may contain:

await page
  .getByRole('button', { name: 'Add Connectivity' })
  .click();

await expect(
  page.getByRole('heading', { name: 'Connectivity Editor' })
).toBeVisible();


## Traceability

Every generated test must identify its source QA test case.

Where practical, generated steps and assertions should maintain explicit
mapping to QA steps.


## Output

Write:

generation.json

and generated Playwright automation into the configured project
automation location.


## Selector Rules

Use runtime-verified selectors where available.

Avoid introducing a selector solely because it appears plausible from
source code.


## Must Not

The Test Generator must not:

- modify application source
- invent expected results
- omit difficult QA steps
- weaken assertions
- suppress assertion failures
- catch failures merely to continue execution
- hardcode credentials
- manufacture test data
- mark tests skipped to avoid failure
- execute the generated tests
- determine PASS or FAIL


## Generation Limitation

If a QA step cannot be safely automated:

Record it explicitly.

Do not silently remove it.


---

# 10. Test Executor

## Role

Execute approved Playwright automation against the configured target
environment.

The Test Executor performs execution only.


## Tool Dependencies

Uses approved execution capabilities defined in:

.copilot/.test-automation/tools.md

Typically:

- Playwright MCP


## Inputs

- context.json
- generation.json
- generated Playwright tests
- execution configuration


## Responsibilities

The Test Executor must:

- execute the selected generated test
- preserve test identity
- record execution start
- record execution completion
- record execution status
- capture assertion failures
- capture runtime errors
- capture the failed step where available
- collect configured evidence
- retain retry/attempt identity


## Evidence

Capture according to configured evidence policy:

- screenshots
- trace
- video
- execution logs
- browser console information
- relevant network failures


## Output

Write:

execution.json

and execution evidence under:

evidence/


## Critical Rule

The Test Executor reports what happened.

It does not decide why the application behaved that way.


## Must Not

The Test Executor must not:

- modify generated tests during execution
- regenerate failed tests
- alter selectors
- change QA expectations
- suppress failures
- convert execution errors into PASS
- decide whether a failure is an application defect
- perform final validation


## Execution Failure

When execution fails, preserve:

- test case ID
- generated test
- attempt
- failed step
- error
- evidence

Then return control to the Test-Orchestrator.

The Test-Orchestrator determines whether to:

- rediscover
- regenerate
- retry
- validate
- block


---

# 11. Validation Agent

## Role

Independently determine whether execution evidence demonstrates the
QA-authored expected behavior.

This agent provides the validation separation between:

automation generation

and

final result determination.


## Inputs

Authoritative:

- qa-testcases.json
- generation.json
- execution.json
- execution evidence

Supporting context when necessary:

- story.json
- browser-discovery.json
- source-analysis.json


## Evidence Priority

For functional outcome validation:

1. QA expected result
2. actual execution result
3. execution evidence
4. generated automation mapping

Source analysis and browser discovery provide supporting context only.

They must not replace execution evidence.


## Responsibilities

For each applicable QA test case:

1. identify original QA expectation
2. verify generated automation represents the required actions
3. verify execution reached the expected validation point
4. inspect relevant actual result
5. inspect execution evidence
6. compare expected versus observed behavior
7. determine validation result
8. record reason
9. reference supporting evidence
10. record limitations


## Supported Results

PASS

Use only when execution evidence supports the QA expectation.


FAIL

Use when reliable execution evidence demonstrates that the application
did not satisfy the QA expectation.


BLOCKED

Use when execution or validation could not proceed due to an unresolved
prerequisite.


INCONCLUSIVE

Use when execution occurred but available evidence is insufficient to
reliably establish PASS or FAIL.


## Output

Write:

validation.json


## Independence Rule

The Validation Agent must not modify:

- QA test cases
- generated automation
- execution results
- execution evidence

If it identifies a problem with generated automation, it must report the
problem.

It must not correct the automation itself.


## Must Not

The Validation Agent must not:

- generate tests
- execute tests
- modify QA expectations
- invent evidence
- infer PASS from lack of errors
- convert INCONCLUSIVE to PASS
- classify an automation failure as an application defect without
  evidence
- fix the generated automation while validating it


---

# 12. Agent Ownership Matrix

## Test-Orchestrator

Owns:

- workflow coordination
- run initialization
- workflow state
- gates
- failure routing
- retry routing
- overall result
- final reporting


## Story Reader

Owns:

- Azure DevOps story interpretation
- story.json


## QA Test Case Reader

Owns:

- QA test case retrieval and normalization
- qa-testcases.json


## React Source Analyzer

Owns:

- implementation impact analysis
- source-analysis.json


## Browser Discovery Agent

Owns:

- runtime application discovery
- runtime selector verification
- browser-discovery.json


## Test Generator

Owns:

- Playwright automation generation
- QA-to-automation mappings
- generation.json


## Test Executor

Owns:

- automation execution
- raw execution outcome
- evidence capture
- execution.json
- evidence/


## Validation Agent

Owns:

- independent expected-versus-observed validation
- validation.json


---

# 13. Agent Communication Model

Agents communicate through structured artifacts rather than relying on
conversation history.

Preferred:

Story Reader
    ↓
story.json
    ↓
Test-Orchestrator

QA Test Case Reader
    ↓
qa-testcases.json
    ↓
Test-Orchestrator

React Source Analyzer
    ↓
source-analysis.json
    ↓
Test-Orchestrator

Browser Discovery Agent
    ↓
browser-discovery.json
    ↓
Test-Orchestrator

Test Generator
    ↓
generation.json
    ↓
Test-Orchestrator

Test Executor
    ↓
execution.json + evidence
    ↓
Test-Orchestrator

Validation Agent
    ↓
validation.json
    ↓
Test-Orchestrator


The Test-Orchestrator is responsible for selecting which artifacts are
passed to each subsequent specialist.


---

# 14. Standard Agent Result Contract

Every specialist should return a consistent orchestration result in
addition to its domain artifact.

Recommended structure:

{
  "agent": "",
  "runId": "",
  "status": "SUCCESS | FAILED | BLOCKED",
  "inputArtifacts": [],
  "outputArtifacts": [],
  "warnings": [],
  "limitations": [],
  "errors": [],
  "recommendedAction": ""
}


`recommendedAction` is advisory.

Only the Test-Orchestrator decides the next workflow transition.


---

# 15. Agent Failure Contract

When an agent cannot complete its responsibility, it must return enough
information for deterministic routing.

Example:

{
  "agent": "Browser Discovery Agent",
  "runId": "12345-...",
  "status": "BLOCKED",
  "classification": "DISCOVERY_FAILURE",
  "testCaseId": 4812,
  "qaStep": 2,
  "reason": "Required runtime control could not be verified.",
  "outputArtifacts": [
    "browser-discovery.json"
  ],
  "recommendedAction": "STOP_OR_REVIEW"
}


Agents must prefer explicit failure over speculative success.


---

# 16. Agent Execution Rules

1. Only the Test-Orchestrator controls workflow progression.

2. Each specialist performs one bounded responsibility.

3. Specialists do not independently invoke other specialists.

4. Agents communicate through structured run artifacts.

5. Agents must use the active project run workspace.

6. Agents must not overwrite another specialist's authoritative output.

7. QA-authored expectations remain immutable during an execution run.

8. Source analysis is context, not runtime evidence.

9. Browser discovery verifies runtime application knowledge.

10. Generated automation must remain traceable to QA test cases.

11. Test execution produces raw execution facts and evidence.

12. Validation is independent from generation.

13. No agent may manufacture information merely to pass a gate.

14. No agent may hide unresolved information.

15. Only evidence from the active run may support the final validation
    result.


---

# 17. Separation of Responsibility

The most important separation in the framework is:

REQUIREMENT
    ↓
Story Reader

QA INTENT
    ↓
QA Test Case Reader

IMPLEMENTATION KNOWLEDGE
    ↓
React Source Analyzer

RUNTIME KNOWLEDGE
    ↓
Browser Discovery Agent

AUTOMATION
    ↓
Test Generator

EXECUTION FACTS
    ↓
Test Executor

VERDICT
    ↓
Validation Agent


No single specialist owns both:

"What should happen?"

and

"Did it happen?"


This separation is required to reduce self-validation and preserve
traceability.