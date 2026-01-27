[]()# Contributing to Gutenberg Workflows

## Writing re-usable workflows

Create a workflow file in `.github/workflows` with the logic for your workflow. The workflow must be a callable /
reusable workflow, indicated by:

```
on:
  workflow_call:
```

## Testing

Every workflow should be tested. This is achieved by running each callable workflow within CI for this repository.
Create a corresponding test workflow for your workflow. For example `python_checks.yaml` is tested by
`test_python_checks.yaml`.
Create a sample test project in the `test-projects` folder - this is the project that your test workflow will run on.
This can be simple but should be representative of the project structure used by the trading engineers.

The test workflow should call your workflow using
a [matrix strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs), passing in a range of
different inputs that could be passed. This should cover the inputs we expect from users.
See [test_python_checks.yaml](.github/workflows/test_python_checks.yaml) for an example.

:warning: Gotcha: :warning: If your workflow calls
the [coverage_report_comment.yaml](.github/workflows/coverage_report_comment.yaml) workflow, do not turn this on for
every test case because you might hit the secondary Github rate limits. Instead follow the pattern followed in the fix
in #15.

## Documenting

All of your workflow inputs should have
a [description](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/workflow-syntax-for-github-actions#example-of-onworkflow_callinputs).

Write a quickstart section in [README.md](README.md) (follow an example from there).

Write more detailed documentation in [the docs/ folder](docs). This should include a table of workflow inputs which will
be auto-updated by the [documentation_updates.yaml](.github/workflows/documentation_updates.yaml) workflow:

Include the following in your file in the docs folder:

```
<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

<!-- AUTO-DOC-INPUT:END -->
```

Then update the [documentation_updates.yaml](.github/workflows/documentation_updates.yaml) workflow to auto-update your
workflow inputs. Add the following step to the documentation_updates.yaml:

```
      - name: "Run auto-doc on {my new workflow name}"
        uses: tj-actions/auto-doc@v2
        with:
          filename: .github/workflows/my_new_workflow.yaml
          output: docs/my_new_workflow.md
          reusable: true
```

## Releasing

We generate a Github release with automatically generated release notes from every commit to main. The new version is
automatically generated from a required label on every pull request.

Every pull request needs to labelled with one of `release/major` / `release/minor` / `release/patch`, and
the [validate_release_label.yaml](./.github/workflows/validate_release_label.yaml) check will fail if your PR does not
have exactly one of these labels.

Make sure you apply the correct label for your changes:

### Major

Any breaking change for the workflow users. Examples include:

- A breaking change on the inputs to any workflow (introducing a new required input, previously optional input becomes
  required, remove a required input).
- A functional change for the workflow that will break for the users **without a code change**. e.g:
    - Changing the linter settings, so users would need to change their codebase to pass the new settings.
    - Enforcing a new test library where users will have to install it as a dependency in their project.

### Minor

A new feature for a workflow(s) without breaking changes. Examples include:

- Adding a new optional input to a workflow, and no functional breaking changes.
- Update a workflow to comment on pull requests.
- Update a workflow to notify to slack.

### Patch

- Update a dependent library in a workflow job where the update is also a patch.
- Bug fixes in a workflow without breaking changes in the inputs.
