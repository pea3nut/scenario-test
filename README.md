# scenario-test

[中文](README.zh.md) | [日本語](README.ja.md)

Write and execute test cases using **extremely generalized natural language**, zero code

```gherkin
Scenario: New user completes registration
    Given the user is on the registration page
    When they fill in all required fields and submit the registration form
    Then registration succeeds and redirects to the user center, with info matching what was entered
```

## Installation

```bash
npx -y agent-add \
  --skill https://github.com/pea3nut/scenario-test.git#skills/scenario-test \
  --command https://github.com/pea3nut/scenario-test.git#commands/scenario-exec.md \
  --sub-agent https://github.com/pea3nut/scenario-test.git#agents/scenario-case-runner.md
```

scenario-test depends on skill, command, and sub-agent support. Make sure your AI coding tool supports these.

## Generating Test Cases

After installation, the AI will automatically recognize the scenario-test skill. Just describe what you want to test in natural language.

Scenario 1: During new feature development

```md
/speckit.specify Develop xxx feature, use scenario-test for the testing part
```

Scenario 2: Generating cases for existing features

```md
Help me generate scenario-test cases for all login-related APIs, with full feature coverage
```

## Run Configuration

On the first case generation, the AI will automatically create `scenario-run-config.md` and infer the configuration based on project context. You can review and modify it before running.

| Config Item     | Default | Description                              |
| --------------- | ------- | ---------------------------------------- |
| `concurrency`   | 4       | Maximum number of concurrent sub-agents  |
| `maxFailures`   | 10      | Early termination after this many failures |
| `maxRetries`    | 1       | Smart retry count (0 = no retries)       |

## Running Test Cases

The most critical part is the **executor configuration** — scenario-test needs to know how to execute a test case, e.g., converting natural language cases into curl-based API tests or Playwright CLI-based E2E tests.

Generally, scenario-test will automatically determine and fill in the executor based on the project context.

All other config items have default values; adjust as needed:

```md
/scenario-exec {test case file or folder} {optional, run config file. Auto-selected by default}
```

Execution uses a **dual-model architecture** to maximize quality, cost efficiency, and execution speed:

- The main model (the one you're chatting with) is responsible for understanding cases, translating them into executor instructions, analyzing failure causes, and deciding whether to retry
- Sub-agent models (multiple sub-agent tasks spawned by the main model) are responsible for executing actual test steps (opening browsers, sending requests, clicking buttons...)

So you need to use a **smart model** (e.g., Claude Sonnet) to run this command; sub-agents will automatically use cheaper models.

## Use Cases

Scenario Test describes cases in natural language, making it naturally suited for **feature-level testing**. It supports very generalized descriptions, such as:

- Fill in all required fields
- Verify that the fields returned by the GET endpoint match what we submitted
- Find the record we just created on the list page

Once your tests get close to implementation-level details, such as a specific function's implementation, you have less room for natural language generalization, and Scenario Test's advantages diminish.
