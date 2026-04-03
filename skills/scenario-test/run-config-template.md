# scenario-run-config.md
# Test Execution Configuration File
# Auto-generated from template on first case addition or first /scenario-exec run. Please fill in based on your actual project.

---

## Run Control

# Maximum concurrent Runner count (default: 4)
# Recommendation: 2-4 for local development, 4-8 for CI environments
concurrency: 4

# Maximum failure tolerance count (triggers early stop when exceeded, default: 10)
maxFailures: 10

# Maximum smart retry count (default: 1; 0 means smart retry is disabled)
# Each Case will be attempted at most (1 + maxRetries) times; beyond that, it's recorded as final status
maxRetries: 1

# Test directory path (optional, leave empty for auto-detection of tests/ / e2e/ / test/)
# Examples: tests | e2e | src/__tests__
testDir:

# Tags filter (optional, leave empty for no filtering)
# Syntax: @P0 | @P0 and @web | @P0 or @api | not @skip | @P0 and (@web or @api)
tags:

---

## Executor Configuration

# Executor used for this run (single executor model: only one executor per run)
executor:

  # Executor command (required)
  # Built-in options:
  #   playwright-cli         — playwright-cli (browser UI automation, default; concurrent isolation via -s=<caseId> sessions)
  #   bash-curl              — Bash curl (HTTP API requests)
  # Custom examples (unit test framework CLI):
  #   command: vitest run --reporter=json
  #   command: jest --ci --json --outputFile=result.json
  command: playwright-cli

  # Executor configuration (free-text description, AI will understand and extract as needed)
  # Different executors require different configurations; just describe clearly in natural language.
  #
  # playwright-cli example:
  #   Target address: https://your-platform.example.com
  #   Authentication: Bearer Token eyJhbGciOiJSUzI1NiJ9...
  #   (or) Authentication: Use SSO login state saved in playwright-storage-state.json
  #   Installation: npm install -g @playwright/cli@latest && playwright-cli install --skills
  #
  # bash-curl example:
  #   API address: https://api.example.com
  #   Authentication: Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
  #
  # vitest example:
  #   Config file: vitest.config.ts, tests run in staging environment
  #
  config: |
    (Describe your executor configuration in natural language here. Delete the examples above when done.)

---

## Runtime Notes (Auto-maintained by /scenario-exec)

# This section records universal MixedCase generation improvements discovered during past executions, applicable to all Scenarios.
# /scenario-exec must reference this section when synthesizing each MixedCase and apply its content to every instruction packet.
# ⚠️ Do not manually delete AI-appended entries (unless the entry is confirmed to be outdated), otherwise subsequent runs will lose important context.
# ⚠️ This section is auto-appended by the command; users should only manually delete entries when confirmed outdated.

notes: |
  (This section is populated by /scenario-exec during execution with universal improvement entries. Initially empty.)
