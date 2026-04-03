---
name: scenario-test-case
description: Manages Gherkin test case assets. Trigger scenarios: adding new test cases, updating existing cases, listing/filtering cases, reading case content, deduplication checks, querying Gherkin writing conventions. All cases are stored as .feature files; AI reads and writes the file system directly.
user-invocable: false
---

# Skill: scenario-test-case

Manages test case assets as Gherkin `.feature` files. All operations work directly on the file system with no code dependencies.

## Directory Conventions

**Test directory (`testDir`) inference rules** (by priority):
1. Check if the user's project root `scenario-run-config.md` has a `testDir` field
2. Detect `tests/`, `e2e/`, `test/` directories under the project root (in this order, use the first one found)
3. If none of the above exist, fall back to `tests/`

**Case storage path**: `{testDir}/features/` (fallback: `tests/features/`)

**Recommended directory structure**:
```
{testDir}/features/
└── <domain>/
    ├── ui/          # UI scenarios (executed by Playwright MCP)
    ├── api/         # API scenarios (executed by Bash curl)
    └── *.feature    # Mixed or general scenarios
```

## Capabilities

### add_case — Add Test Case

**Trigger**: User provides Gherkin text and requests it be added to the case library.

**Execution steps**:
1. Determine target path:
   - If the user specified a path (`path` parameter), use that path (relative to `{testDir}/features/`)
   - If not specified, infer from case content: UI-related goes to `ui/`, API-related goes to `api/`, mixed goes to root
   - File names use kebab-case, e.g., `search-product.feature`
2. **Check and generate `scenario-run-config.md`** (triggered on first case addition):
   If `{testDir}/features/scenario-run-config.md` doesn't exist:
   1. Copy from `{SKILL_DIR:scenario-test}/run-config-template.md` to `{testDir}/features/scenario-run-config.md` (create directory if it doesn't exist)
   2. **Auto-infer executor configuration**, checking in this priority order:
      - First check if the project already has a test framework (e.g., `package.json` contains vitest/jest/playwright/cypress dependencies, corresponding config files exist); if so, prefer reusing it, filling the corresponding CLI command into `executor.command`
      - If the project has no existing test framework, determine from case content:
        - Cases contain UI operations (page navigation, clicks, inputs, etc.) → `command: playwright-cli`
        - Cases contain API requests (endpoints, status codes, response bodies, etc.) → `command: bash-curl`
      - If a target address can be inferred from the project (e.g., dev script in `package.json`, URL in README), auto-fill `executor.config`
   3. **Only ask the user when uncertain** (e.g., cannot determine executor type, cannot find target address); when asking, explain the executor concept:
      > scenario-test uses "executors" to actually run test steps. Different executors correspond to different testing methods:
      > - `playwright-cli`: Browser automation for Web UI testing (clicking, filling, screenshots, etc.)
      > - `bash-curl`: HTTP requests for backend API testing
      > - You can also use a custom CLI command (e.g., `vitest run`, `jest --ci`)
      >
      > Please provide: 1) Test target address; 2) Authentication method (if any)
   4. Fill the inferred or user-provided information into the config file, inform the user that `scenario-run-config.md` has been generated with current settings, and can be modified at any time
3. **Deduplication check**: Scan all `.feature` files under `{testDir}/features/`, check if a case with the same path and Scenario name already exists (strong dedup)
   - If exists: Prompt the user, ask whether to overwrite; if yes, execute update_case
   - If not exists: Create the file directly
3. Write the `.feature` file, ensuring:
   - File starts with `Feature:`
   - Each Scenario has clear Given/When/Then steps
   - Appropriately add tags (at least `@P0` or `@P1` priority tag)
4. Inform the user of the file write path

**Example (T706)**:
```gherkin
Feature: Product Search

  @P0 @web
  Scenario: Search for existing product
    Given the user is logged into the product platform
    When the user types "headphones" in the search box
    Then the search results list shows at least 1 product entry

  @P0 @api
  Scenario: Search API returns product list
    Given the product search API GET /api/products/search is accessible
    When the request parameters include keyword="headphones"
    Then the response status code is 200
    And the response body's data.list array length is greater than 0
```

---

### update_case — Update Test Case

**Trigger**: User requests modification of an existing case.

**Execution steps**:
1. Locate the target file:
   - If the user provided a path, locate directly
   - If a Scenario name was provided, recursively search under `{testDir}/features/` for the matching Scenario
2. Read the original file content
3. Find the target Scenario in the file (match by name), replace that Scenario block with new content
   - If the target is the entire Feature file, replace the whole file
   - If only updating a specific Scenario, preserve other Scenarios in the file
4. Write back to file
5. Report the changes (in diff format)

**Deduplication rules (during update)**:
- Same path, same Scenario name: Overwrite update (don't create a copy)
- Different path, same Scenario name: Treat as independent cases, don't auto-merge (prompt user about potential duplication)

**Weak deduplication strategy (T905)**:

When strong dedup (exact path + name match) doesn't hit, perform weak dedup detection to avoid semantically duplicate cases:

1. **Canonical Hash Calculation**: Calculate a normalized hash for each Scenario
   - Steps: Remove excess whitespace → Convert step keywords (Given/When/Then/And/But) to lowercase → Trim Scenario title and each step text → Concatenate and take first 8 characters of hash
   - Example: `"Search for existing product"` + `"given the user is logged into the product platform"` + `"when the user types in the search box"` + `"then search results show"` → hash `a3f9d2b1`

2. **Weak duplicate detection**: If canonical hash matches an existing case but path or name differs:
   - Prompt user about suspected duplicate case, show differences between the two
   - Ask user: Overwrite existing case / Keep as new case / Generate conflict file
   - If user chooses to generate conflict file: Save new case as `<original-filename>.conflict.feature`, don't modify the original file

---

### list_cases — List Test Cases

**Trigger**: User requests to view, search, or filter cases.

**Execution steps**:
1. Recursively scan all `.feature` files under `{testDir}/features/`
2. **Identify and extract Scenario tags (T704)**:
   - In each `.feature` file, the tags line immediately precedes the `Scenario:` keyword, starts with `@`, multiple tags separated by spaces
   - Example: `  @P0 @web @smoke` → tags are `["@P0", "@web", "@smoke"]`
   - Feature-level tags (before `Feature:`) are inherited by all Scenarios
   - Note indentation when extracting; tags lines typically have 2-space indentation
3. Filter by conditions:
   - **By directory**: Return only cases under a specified subdirectory
   - **By tags**: Supports `@P0`, `@P0 and @web`, `@P0 or @api`, `not @skip`, etc.
   - **By keyword**: Search in Feature names and Scenario names
4. Output results in table or list format:
   - File path, Feature name, Scenario name, tags, last modified time

**Tags filter syntax**:
- `@P0`: Contains only this tag
- `@P0 and @web`: Contains both tags (AND)
- `@P0 or @api`: Contains either tag (OR)
- `not @skip`: Does not contain this tag (NOT)
- Combination: `@P0 and (@web or @api)`

---

### get_case — Read Single Case

**Trigger**: User requests to view the full content of a case.

**Execution steps**:
1. Locate the file:
   - If a path is provided, read directly
   - If a Scenario name is provided, recursively search under `{testDir}/features/`
2. Display `.feature` file content in a code block
3. Also display: file path, tags, step count

---

### explain_how_to_write — Case Writing Guidelines

**Trigger**: User is unfamiliar with Gherkin, asks about writing conventions, or needs examples.

**Output the following content**:

#### Gherkin Basic Syntax

Follows the [Cucumber Gherkin official specification](https://cucumber.io/docs/gherkin/).

**File structure**:
```gherkin
Feature: Module name (brief description)

  Background:                     # Optional: Shared preconditions for each Scenario
    Given the user is logged in

  @P0 @web                        # Tags: priority + category
  Scenario: Scenario description (describe behavior, not implementation details)
    Given precondition (current system state)
    When user action (single action)
    Then expected observable result (assertion)
    And additional result (optional, continues Then logic)
    But exception case (optional, means NOT)
```

**Writing principles**:
- **One Scenario tests one thing**, recommended 3-7 steps
- **Given**: Describes the initial system state, not user actions
- **When**: Describes the single behavior triggered by the user
- **Then**: Describes the observable assertion result (UI change, API response, etc.)
- **Step descriptions should be specific**, e.g., use `"headphones"` not `valid keyword`

**Tags conventions**:
- Priority: `@P0` (must test), `@P1` (high frequency), `@P2` (extended)
- Type: `@web` (UI), `@api` (endpoint), `@smoke` (smoke test)
- Status: `@skip` (temporarily skipped), `@flaky` (unstable)

**Complete example**:
```gherkin
Feature: User Login

  @P0 @web @smoke
  Scenario: Successful login with correct credentials
    Given the user is on the login page
    When the user enters correct email "test@example.com" and password "password123"
    And the user clicks the login button
    Then the page redirects to the homepage
    And the top navigation bar displays the user avatar

  @P0 @web
  Scenario: Failed login shows error message
    Given the user is on the login page
    When the user enters email "test@example.com" and wrong password "wrong"
    And the user clicks the login button
    Then the page displays error message "Incorrect password"
    And the page remains on the login page

  @P1 @api
  Scenario: Login API returns correct token
    Given the login API POST /api/auth/login is accessible
    When the request body contains correct username and password
    Then the response status code is 200
    And the response body contains a token field
```

**Official documentation**: https://cucumber.io/docs/gherkin/
