---
name: read-user-story
description: "Retrieve an Azure DevOps user story for the active Test-Orchestrator run, preserve its authoritative requirement information, and write the normalized story context to story.json."
user-invocable: false
---

# Read User Story

## Purpose

Retrieve the Azure DevOps user story associated with the active
Test-Orchestrator run and create the normalized story artifact used
by downstream test-automation agents.

This skill establishes the authoritative requirement context for the
run.

The skill retrieves and structures story information.

It does not:

- interpret QA test cases
- analyze application source code
- discover runtime UI behavior
- generate Playwright tests
- execute tests
- validate application behavior
- modify the Azure DevOps story


---

## Global References

Always follow:

GLOBAL:
- .copilot/.test-automation/workflow.md
- .copilot/.test-automation/agents.md
- .copilot/.test-automation/tools.md
- .copilot/.test-automation/gates.md
- .copilot/.test-automation/context.md


---

## Responsible Agent

This skill is executed for:

Story Reader


---

## Invocation

This skill is invoked by:

Test-Orchestrator

during the Story Reader stage.

It must not independently invoke another specialist agent or advance
the workflow.


---

## Required Input

Required:

- Azure DevOps Story ID
- active run workspace
- resolved Azure DevOps context

The active run workspace must be located under:

.test-automation-project/runs/<run-id>/


Example logical input:

{
  "runId": "12345-...",
  "storyId": 12345,
  "runWorkspace": ".test-automation-project/runs/12345-...",
  "ado": {
    "organization": "<resolved>",
    "project": "<resolved>"
  }
}


Do not guess missing required identifiers.


---

## Tool Usage

Use only the approved Azure DevOps integration defined in:

.test-automation/tools.md

Prefer the configured Azure DevOps MCP/tool for work-item retrieval.

The skill must retrieve the authoritative source work item before
creating story.json.

Do not reconstruct story content from:

- source code
- commit messages
- pull request descriptions
- test cases
- browser behavior
- conversation history

These may be related information but are not substitutes for the
authoritative Azure DevOps story.


---

# Processing Steps

## Step 1: Validate Execution Context

Verify that:

- run ID is available
- Story ID is available
- active run workspace is available
- Azure DevOps project context is available
- run workspace belongs to the active project repository

If required execution context is unavailable:

return:

BLOCKED

Do not create an authoritative story.json using assumptions.


---

## Step 2: Retrieve Story

Retrieve the work item identified by the Story ID from Azure DevOps.

Retrieve available authoritative fields including:

- work item ID
- work item type
- title
- description
- acceptance criteria
- state
- assigned owner
- area path
- iteration path
- tags
- relations
- linked work items
- linked test cases
- repository references
- pull request references

Only record fields actually retrieved from approved sources.


---

## Step 3: Verify Story Identity

Confirm that the retrieved work item corresponds to the requested
Story ID.

Record its actual work item type.

Do not silently replace the requested story with:

- parent work item
- child work item
- related story
- feature
- epic
- bug
- linked test case


If the requested work item exists but is not a Story-type work item,
preserve its actual type and report the condition to the
Test-Orchestrator.

Do not reinterpret the work item type.


---

## Step 4: Normalize Story Content

Normalize retrieved content into the story artifact.

Permitted normalization includes:

- converting supported markup into readable structured text
- converting tags into arrays
- separating acceptance criteria into structured entries when the
  source representation supports doing so without changing meaning
- normalizing relations into structured objects
- preserving source identifiers

Normalization must not change requirement meaning.


---

## Step 5: Preserve Acceptance Criteria

Acceptance criteria are authoritative requirement information.

Preserve them as retrieved.

Do not:

- add acceptance criteria
- remove acceptance criteria
- rewrite them to make automation easier
- convert implementation behavior into acceptance criteria
- infer missing acceptance criteria
- substitute QA expectations for acceptance criteria


If acceptance criteria are unavailable, record that they were not
available.

Do not create them.


---

## Step 6: Capture Relationships

Capture relevant retrieved work-item relationships.

Examples may include:

- linked test cases
- parent work item
- child work items
- related work items
- pull request references
- repository references

Record only relationships returned by the approved source.

Do not infer relationships from similar names or IDs.


---

## Step 7: Write Story Artifact

Write the normalized result to:

<active-run-workspace>/story.json

Recommended contract:

{
  "runId": "",
  "storyId": null,
  "workItemType": "",
  "title": "",
  "description": "",
  "acceptanceCriteria": [],
  "state": "",
  "assignedTo": null,
  "areaPath": "",
  "iterationPath": "",
  "tags": [],
  "linkedWorkItems": [],
  "linkedTestCases": [],
  "repositoryReferences": [],
  "pullRequestReferences": [],
  "source": {
    "system": "Azure DevOps",
    "workItemId": null,
    "retrievedAt": ""
  },
  "warnings": [],
  "limitations": []
}


Use null or an empty collection where appropriate when a field is
unavailable.

Do not invent values merely to complete the structure.


---

# Artifact Ownership

This skill owns:

story.json


It may create or replace story.json only for the active run.

It must not modify:

- context.json
- qa-testcases.json
- source-analysis.json
- browser-discovery.json
- generation.json
- execution.json
- validation.json


---

# Output Contract

After writing story.json, return a structured result to the
Test-Orchestrator.

Recommended result:

{
  "agent": "Story Reader",
  "skill": "read-user-story",
  "runId": "",
  "status": "SUCCESS",
  "classification": null,
  "inputArtifacts": [
    "context.json"
  ],
  "outputArtifacts": [
    "story.json"
  ],
  "warnings": [],
  "limitations": [],
  "errors": [],
  "recommendedAction": "CONTINUE"
}


SUCCESS means the Story Reader completed its responsibility.

It does not mean the next workflow gate automatically passes.


---

# Failure Contract

## Story Not Found

Return:

{
  "agent": "Story Reader",
  "skill": "read-user-story",
  "status": "BLOCKED",
  "classification": "STORY_NOT_FOUND",
  "recommendedAction": "STOP"
}


Do not create fabricated story content.


---

## Access Failure

When the work item exists but cannot be accessed:

classification:

ACCESS_FAILURE


Return control to the Test-Orchestrator.


---

## Azure DevOps Tool Failure

If the approved Azure DevOps integration fails:

classification:

TOOL_FAILURE


Preserve available diagnostic information.

Do not substitute information from another source and represent it as
the authoritative story.


---

## Invalid Execution Context

If required run or Azure DevOps context is unavailable:

classification:

ORCHESTRATION_FAILURE


Return:

BLOCKED or FAILED according to the workflow contract.


---

# Story Reader Rules

The following rules are mandatory.

1. Always retrieve the authoritative work item.

2. Always preserve the requested Story ID.

3. Preserve the actual work item type returned by Azure DevOps.

4. Preserve acceptance criteria without changing their functional
   meaning.

5. Do not generate missing requirements.

6. Do not generate QA test cases.

7. Do not infer expected behavior from application source.

8. Do not infer expected behavior from browser behavior.

9. Do not interpret current application behavior as the requirement.

10. Do not modify the Azure DevOps work item.

11. Do not perform source analysis.

12. Do not perform browser discovery.

13. Do not generate automation.

14. Do not execute automation.

15. Do not determine PASS or FAIL.

16. Write output only to the active run workspace.

17. Never read another run's story.json as a substitute for the current
    story retrieval.

18. Return control to the Test-Orchestrator after completion.


---

# Example

Input:

{
  "runId": "7421-20261026-ab12",
  "storyId": 7421,
  "runWorkspace": ".test-automation-project/runs/7421-20261026-ab12/"
}


Azure DevOps provides:

Story ID:
7421

Title:
Add Connectivity

Description:
Allow users to create a new connectivity configuration.

Acceptance Criteria:

- User can open the connectivity editor.
- Required connectivity fields are displayed.


Normalized story.json:

{
  "runId": "7421-20261026-ab12",
  "storyId": 7421,
  "workItemType": "User Story",
  "title": "Add Connectivity",
  "description": "Allow users to create a new connectivity configuration.",
  "acceptanceCriteria": [
    "User can open the connectivity editor.",
    "Required connectivity fields are displayed."
  ],
  "state": "",
  "assignedTo": null,
  "areaPath": "",
  "iterationPath": "",
  "tags": [],
  "linkedWorkItems": [],
  "linkedTestCases": [],
  "repositoryReferences": [],
  "pullRequestReferences": [],
  "source": {
    "system": "Azure DevOps",
    "workItemId": 7421,
    "retrievedAt": ""
  },
  "warnings": [],
  "limitations": []
}


The skill then returns:

{
  "agent": "Story Reader",
  "skill": "read-user-story",
  "runId": "7421-20261026-ab12",
  "status": "SUCCESS",
  "classification": null,
  "inputArtifacts": [
    "context.json"
  ],
  "outputArtifacts": [
    "story.json"
  ],
  "warnings": [],
  "limitations": [],
  "errors": [],
  "recommendedAction": "CONTINUE"
}


Control returns to:

Test-Orchestrator


Next workflow capability:

QA Test Case Reader