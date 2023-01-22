# Workflows

Repository to have shared workflows to be used across organization.

## Re-usable Workflows

To use a workflow:

```yaml
jobs:
  job-name:
    uses: chill-viking/workflows/.github/workflows/{FILENAME}.yml@TAG_OR_BRANCH
```

Typically, would suggest using `main` branch when using a workflow. Not really going to bother with tagging this repo at this time, but that may change later... we'll see.

To create a re-usable workflow, it just needs to have `workflow_call` specified as it's trigger, which you can use to specify `inputs` and `secrets` for the workflow.

For more detail you can start your journey at [Creating a reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow) in GitHub Docs.

### Available Re-usable Workflows

- [nx-test-affected.yml](#test-affectedyml)
- [nx-sonar-cloud-scan.yml](#nx-sonar-cloud-scanyml)

#### test-affected.yml

Will call lint/test/build targets for all affected projects inside the specified nx workspace.

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

| Name                | Description                                      | Required | Default       |
|---------------------|--------------------------------------------------|----------|---------------|
| `working-directory` | The directory to run the workflow from.          | No       | `./`          |
| `fetch-depth`       | The number of commits to fetch.                  | No       | `0`           |
| `agent-count`       | The number of agents to use for parallelization. | No       | `3`           |
| `base`              | The base branch to compare against.              | No       | `origin/main` |
| `head`              | The head branch to compare against.              | No       | `HEAD`        |

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

```typescript
export default {
  // ...
  coverageDirectory: 'coverage',
  coverageReporters: [['lcov', { projectRoot: 'apps/project-name' }]],
  // ...
}
```

This will update coverage reporting to create the report at `coverage/lcov.info` in the project folder and have the generated coverage work for the project root.

Parameters:

| Name                | Description                             | Required | Default                                    |
|---------------------|-----------------------------------------|----------|--------------------------------------------|
| `sonar-org`         | Sonar Cloud organization key            | No       | `chill-viking-org`                         |
| `sonar-project`     | Sonar Cloud project key                 | Yes      |                                            |
| `working-directory` | Location of nx workspace                | No       | `./`                                       |
| `project-location`  | Location of project within nx workspace | Yes      |                                            |
| `coverage-location` | Location of coverage report             | No       | `coverage/lcov.info`                       |
| `project-name`      | Name of project                         | Yes      |                                            |
| `config-location`   | Location of project TS config file      | Yes      |                                            |
| `project-version`   | Version of project                      | No       | `''` - will be retrieved from package.json |
| `package-location`  | Location of project package.json file   | No       | `{working-directory}{project-location}`    |

Secrets:

| Name           | Description                  |
|----------------|------------------------------|
| `sonar-token`  | Sonar Cloud token            |
| `github-token` | GitHub token for PR comments |

## Composite actions

Actions to be re-used in workflows.

To use an action:

```yml
uses: chill-viking/workflows/actions/{action-folder}@main
```

### Available composite actions

- [get-json-version](#get-json-version)
- [get-version](#get-version)
- [npm-ci](#npm-ci)
- [nx-test](#nx-test)
- [nx-test-and-build](#nx-test-and-build)

#### get-json-version

Get version from `package.json` or any json file with a `version` property.

Usage:

```yml
jobs:
  job-id:
    name: 'Name of job'
    runs-on: ubuntu-latest
    steps:
      - uses: chill-viking/workflows/actions/get-json-version@main
        name: Get version
        id: step-id
        with:
          path: 'npm-root-folder'

      # Use version read from ./npm-root-folder/package.json
      - run: echo ${{ steps.step-id.outputs.json-version }} # 1.0.0
```

Inputs:

| Name        | Description                                          | Required | Default        |
|-------------|------------------------------------------------------|----------|----------------|
| `path`      | The path of the folder containing the json file. 1️  | No       | `.`            |
| `json-file` | The json file containing `version` property to read. | No       | `package.json` |

Notes:

1️ Should only be the folder path, not the file name.

Outputs:

| Name           | Description                                                       |
|----------------|-------------------------------------------------------------------|
| `json-version` | The value of `version` property from `json-file` in `path` folder |

#### get-version

Get version from `package.json` and prepare as an alpha version, if `is-release` not provided or set to `'false'`.

The alpha version generated will be `{adjusted-package-version}-alpha.{github.run_number}`.
Using `{github.run_number}` to ensure that the alpha version is unique.

`{adjusted-package-version}` will be a bumped version of the version from `package.json`.
Choose which part of the version to bump by providing `version-to-bump` input, which can be `major`, `minor`, or `patch`.
This will default to `patch`.

Usage:

```yml
jobs:
  job-id:
    name: 'Name of job'
    runs-on: ubuntu-latest
    steps:
      - uses: chill-viking/workflows/actions/get-version@main
        name: Get version
        id: step-id
        with:
          package-path: './npm-root-folder'

      # Use version read from ./npm-root-folder/package.json
      - run: echo ${{ steps.step-id.outputs.version }} # 1.0.0-alpha.123
```

Inputs:

| Name              | Description                                                | Required | Default   |
|-------------------|------------------------------------------------------------|----------|-----------|
| `package-path`    | The path of the folder containing the `package.json` file. | No       | `./`      |
| `is-release`      | Whether the output `version` is meant for a release.       | No       | `'false'` |
| `version-to-bump` | Which part of the version to bump.                         | No       | `'patch'` |

Outputs:

| Name      | Description                                                                    |
|-----------|--------------------------------------------------------------------------------|
| `version` | The resolved version to be used, either an alpha version or the actual version |

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

| Name                | Description                                 | Required | Default |
|---------------------|---------------------------------------------|----------|---------|
| `working-directory` | The directory to run the workflow from. 1️⃣ | No       | `./`    |

Notes:
1️⃣ Make sure to suffix `working-directory` path with `/`, as `node_modules` will be appended to specify folder to be cached

#### nx-test

Composite action to run test target for the current nx workspace.

Usage:

```yml
jobs:
  job-id:
    name: 'Name of job'
    runs-on: ubuntu-latest
    steps:
      - uses: chill-viking/workflows/actions/nx-test@main
        name: Test nx-project
        with:
          project-name: 'nx-project'
          working-directory: './npm-root-folder/'
```

Inputs:

| Name                | Description                                            | Required | Default |
|---------------------|--------------------------------------------------------|----------|---------|
| `project-name`      | name of project in nx workspace to run `:test` against | Yes      |         |
| `working-directory` | location of nx workspace                               | No       | `./`    |

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

| Name                | Description                                            | Required | Default |
|---------------------|--------------------------------------------------------|----------|---------|
| `project-name`      | name of project in nx workspace to run targets against | Yes      |         |
| `working-directory` | location of nx workspace                               | No       | `./`    |
