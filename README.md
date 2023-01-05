# Workflows

Repository to have shared workflows to be used across organization.

## Re-usable Workflows

To use a workflow:

```yaml
Uses:
  chill-viking/workflows/.github/workflows/{FOLDER}/{FILENAME}.yml@TAG_OR_BRANCH
```

Typically would suggest using `main` branch when using a workflow. Not really going to bother with tagging this repo at this time, but that may change later... we'll see.

To create a re-usable workflow, it just needs to have `workflow_call` specified as it's trigger, which you can use to specify `inputs` and `secrets` for the workflow.

For more detail you can start your journey at [Creating a reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow) in GitHub Docs.

### Available Re-usable Workflows

- [nx/test-affected.yml](#test-affectedyml)

#### nx workflows

Workflows meant for use with an `nx` project.

##### test-affected.yml

Will call lint/test/build targets for all affected projects inside of the specified nx workspace.

Usage:

```yml
- uses: chill-viking/workflows.github/workflows/nx/test-affected.yml@main
  name: 'Test Affected'
  with:
    working-directory: './npm-root-folder/'
    fetch-depth: 0
    agent-count: 4
```

Inputs:

- `working-directory`: location of nx workspace, will default to `'./'`
- `fetch-depth`: values passed to `actions/checkout` when checking out repository, defaults to `5`
- `agent-count`: number of parallel agents to use for `nx` commands, defaults to `3`

## Composite actions

Actions to be re-used in workflows.

To use an action:

```yml
uses: chill-viking/workflows/actions/{action-folder}@main
```

### Available composite actions

- [npm-ci](#npm-ci)

#### npm-ci

Install npm dependencies, using cache if already cached. Cache based on available `package-lock.json` files in source.

Usage:

```yml
- uses: chill-viking/workflows/actions/npm-ci@main
  name: Install dependencies
  with:
    working-directory: './npm-root-folder/'
```

Inputs:

- `working-directory`: location of root npm folder, defaults to `'./'` (should contain `/` as `node_modules` will be appended to specify location of node modules folder).
