---
name: application-knowledge
description: >
  Read and maintain reusable application knowledge discovered during
  browser-based test automation.
---

# Application Knowledge

Maintain reusable application knowledge discovered from actual
runtime application observation.

Knowledge is used to reduce unnecessary browser rediscovery across
test cases.

## Knowledge Location

Use:

.test-automation-project/knowledge/

## Knowledge May Contain

- application routes
- pages
- navigation paths
- stable UI controls
- accessible roles
- accessible names
- test identifiers
- dialogs
- forms
- reusable application interactions
- relevant API observations

## Knowledge Lookup

Before performing browser discovery:

1. Identify the application information required by the QA test case.
2. Search existing application knowledge.
3. Reuse relevant runtime knowledge when appropriate.
4. Identify missing, conflicting, or stale information.
5. Request browser discovery only for information that requires runtime
   observation.

## Knowledge Update

After successful runtime discovery:

- add newly observed information
- update stale information
- preserve observation source
- preserve environment
- preserve relevant repository or application version context

Do not replace valid knowledge using assumptions from source analysis.

## Knowledge Trust

Application knowledge represents previously observed runtime behavior.

It must not be treated as permanent application truth.

Browser discovery should be performed when:

- required knowledge is missing
- existing knowledge conflicts with the current application
- relevant UI implementation has changed
- stored locator fails during execution
- environment behavior differs
- existing knowledge cannot reliably satisfy the QA test

## Knowledge Safety

Never store:

- passwords
- access tokens
- PATs
- session cookies
- authentication headers
- secrets
- sensitive test data

## Rules

- Prefer reuse over rediscovery.
- Prefer runtime observations over inferred UI behavior.
- Never invent missing knowledge.
- Never change QA expectations based on stored knowledge.
- Never report stored knowledge as current execution evidence.
- Execution evidence must still come from the current test run.