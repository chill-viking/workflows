# Workflows

Repository to have shared workflows to be used across organization.

## Re-usable Workflows

To use a workflow:

```yaml
jobs:
  job-name:
    uses: chill-viking/workflows/.github/workflows/{FILENAME}.yml@TAG_OR_BRANCH
```

Typically would suggest using `main` branch when using a workflow. Not really going to bother with tagging this repo at this time, but that may change later... we'll see.

To create a re-usable workflow, it just needs to have `workflow_call` specified as it's trigger, which you can use to specify `inputs` and `secrets` for the workflow.

For more detail you can start your journey at [Creating a reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow) in GitHub Docs.

### Available Re-usable Workflows

- [nx-test-affected.yml](#test-affectedyml)

#### test-affected.yml

Will call lint/test/build targets for all affected projects inside of the specified nx workspace.

Usage:

```yml
jobs:
  use-remote-workflow:
    uses: chill-viking/workflows/.github/workflows/nx-test-affected.yml@main
    name: 'Test Affected'
    with:
      working-directory: './npm-root-folder/'
      fetch-depth: 0
      agent-count: 4
```

Inputs:

- `working-directory`: location of nx workspace, will default to `'./'`
- `fetch-depth`: values passed to `actions/checkout` when checking out repository, defaults to `0`
- `agent-count`: number of parallel agents to use for `nx` commands, defaults to `3`
- `base`: specify the base for comparing affected, defaults to `origin/master` (include `origin/` as the local git repo will not have any other local branches)
- `head`: specify the head for comparing affected, defaults to `HEAD` (the latest commit)

## Composite actions

Actions to be re-used in workflows.

To use an action:

```yml
uses: chill-viking/workflows/actions/{action-folder}@main
```

### Available composite actions

- [npm-ci](#npm-ci)
- [nx-test-and-build](#nx-test-and-build)

#### npm-ci

Install npm dependencies, using cache if already cached. Cache based on available `package-lock.json` files in source.

Usage:

```yml
jobs:
  job-id:
    name: 'Name of job'
    runs-on: ubuntu-latest
    steps:
      - uses: chill-viking/workflows/actions/npm-ci@main
        name: Install dependencies
        with:
          working-directory: './npm-root-folder/'
```

Inputs:

- `working-directory`: location of root npm folder, defaults to `'./'` (should contain `/` as `node_modules` will be appended to specify location of node modules folder)

#### nx-test-and-build

Composite action to run test and build targets for the current nx workspace.

Usage:

```yml
jobs:
  job-id:
    name: 'Name of job'
    runs-on: ubuntu-latest
    steps:
      - uses: chill-viking/workflows/actions/nx-test-and-build@main
        name: Test and Build nx-project
        with:
          project-name: 'nx-project'
          working-directory: './npm-root-folder/'
```

Inputs:

- `project-name`: name of project in nx workspace to run `:test` and `:build:production` against
- `working-directory`: location of nx workspace, will default to `'./'`
