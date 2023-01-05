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
- [nx-sonar-cloud-scan.yml](#nx-sonar-cloud-scanyml)

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

#### nx-sonar-cloud-scan.yml

Will test and build nx project and publish results to sonar cloud.

Usage:

```yml
jobs:
  use-workflow:
    name: lint, test, and publish
    uses: chill-viking/workflows/.github/workflows/nx-sonar-cloud-scan.yml@main
    with:
      sonar-project: sonar-cloud-project-key
      working-directory: ./nx-workspace-folder/
      project-location: apps/project-name
      project-name: project-name
      config-location: tsconfig.app.json
      package-location: ./nx-workspace-folder/
    secrets:
      sonar-token: ${{ secrets.SONAR_TOKEN }}
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

To get code coverage included, update project `jest.config.ts` with the following:

```json
{
  coverageDirectory: 'coverage',
  coverageReporters: [['lcov', { projectRoot: 'apps/project-name' }]],
}
```

This will update coverage reporting to create the report at `coverage/lcov.info` in the project folder and have the generated coverage work for the project root.

Parameters:

- `sonar-org`: organization key to use for sonarcloud, defaults to `chill-viking-org`
- `sonar-project`: project key to use for sonarcloud, required input
- `working-directory`: location of nx workspace, will default to `'./'`
- `project-location`: location of project root in workspace, required input
- `coverage-location`: location to find `lcov.info` report, defaults to `coverage/lcov.info`
- `project-name`: name of project to test and build, required input
- `config-location`: TS config to use for report, relative to project location. Required input
- `project-version`: optional override for project version to be passed to sonarcloud
- `package-location`: folder containing `package.json` version will be retrieved from that file. Only used if `project-version` is not supplied

Secrets:

- `sonar-token`: sonar token to be used for publishing to sonarcloud
- `github-token`: GitHub token to use for publishing

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
