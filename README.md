# ⚠️ You probably don't need this! [DataDog has a first-party integration now](https://www.datadoghq.com/blog/datadog-github-actions-ci-visibility/). ⚠️

When I made this, there wasn't an easy way to report these metrics automatically. Now there is. I'm archiving this since I am not using it anymore (because the first-party integration is easier and nicer). I recommend using that if you can.

# Datadog Metrics Reporter Action

[![CI](https://github.com/drewwyatt/datadog-metrics-reporter/actions/workflows/ci.yml/badge.svg)](https://github.com/drewwyatt/datadog-metrics-reporter/actions/workflows/ci.yml) ![Package Version](https://img.shields.io/github/package-json/v/drewwyatt/datadog-metrics-reporter)

Report CI Metrics to Datadog.

This action captures conclusions and durations for jobs and steps from [context](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#job-context) and the actions toolkit ([@actions/github](https://github.com/actions/toolkit/tree/main/packages/github)) and reports them to Datadog using autogenerated metrics keys.

## Usage

For this action to report accurate metrics, you need to (at least) ensure two things:

1. `datadog-metrics-reporter` is the last thing to run in your workflow (since it is only possible to report the status/duration of a job/step that has already completed).
2. `datadog-metrics-reporter` `always()` runs (so that failures are also reported).

You can achieve this by adding this to your workflow file as a separate job that [`needs`](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idneeds) every other job in the workflow _and_ set the [`if` conditional](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idif) to `always()`.

e.g.

```yml
# ...

jobs:
  build:
    # ...
  test:
    # ...
  report:
    runs-on: ubuntu-latest
    needs: [build, test] # run these first
    if: always() # still run even if other jobs fail
    steps:
      - uses: drewwyatt/datadog-metrics-reporter@v0.5.0
        with:
          datadog-api-key: ${{secrets.DATADOG_API_KEY}}
          github-token: ${{secrets.GITHUB_TOKEN}}
```

### Inputs

| Name              | Description                                                                                                                         |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `datadog-api-key` | An [API Key or Client Token](https://docs.datadoghq.com/account_management/api-app-keys/#client-tokens), **not** an Application Key |
| `github-token`    | A token capable of calling `listJobsForWorkflowRun` (probably `${{secrets.GITHUB_TOKEN}}`)                                          |
