# Workflows

Repository to have shared workflows to be used across organization.

## Re-usable Workflows

To use a workflow:

```yaml
Uses:
  chill-viking/workflows/{FOLDER}/{FILENAME}.yml@TAG_OR_BRANCH
```

> // TODO: need to figure out if can do in own folders, or has to be in .github/workflows folder specifically.

Typically would suggest using `main` branch when using a workflow. Not really going to bother with tagging this repo at this time, but that may change later... we'll see.

To create a re-usable workflow, it just needs to have `workflow_call` specified as it's trigger, which you can use to specify `inputs` and `secrets` for the workflow.

For more detail you can start your journey at [Creating a reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow) in GitHub Docs.

### Available Re-usable Workflows

- [nx/test-affected.yml](#test-affectedyml)

#### nx workflows

Workflows meant for use with an `nx` project.

##### test-affected.yml

Will call lint/test/build targets for all affected projects inside of the specified nx workspace.

Suggested workflow to call this workflow:

```yml

```

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
