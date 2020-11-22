# azure-pipelines-templates 🚀

[![license](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/AleksaC/azure-pipelines-templates/blob/master/LICENSE)
[![Build Status](https://dev.azure.com/aleksac/aleksa-oss/_apis/build/status/AleksaC.azure-pipelines-templates?branchName=master)](https://dev.azure.com/aleksac/aleksa-oss/_build?definitionId=3&_a=summary)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/AleksaC/azure-pipelines-templates/blob/master/.pre-commit-config.yaml)

Collection of Azure Pipelines templates for jobs and tasks I commonly use in my projects.

## Getting started ⚙️
You can expand on the following sample `azure-pipelines.yml` configuration:
```yaml
resources:
  repositories:
    - repository: aleksac
      type: github
      endpoint: AleksaC
      name: AleksaC/azure-pipelines-templates
      ref: refs/tags/v0.0.2

jobs:
  - template: jobs/pre-commit.yml@aleksac
```

**Note**: You need to set the endpoint to the name of the github [service connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml)
you've set up.

### Tasks 📝
#### [`push-to-github.yml`](tasks/push-to-github.yml)
Pushes changes to `GitHub`. Uses `InstallSSHKey` task to provide authentication.
Take a look at its [documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/install-ssh-key)
to see how to set it up. Note that you need to set `KNOWN_HOST_ENTRY` and `SSH_PUBLIC_KEY`
[variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?tabs=classic%2Cbatch#set-variables-in-pipeline).
Make sure to add quotes when adding the variables so they are parsed correctly.
For security make sure that secrets aren't passed to builds for pull requests
from forks (Edit -> Triggers -> Pull request validation). Since the secrets
should not be passed to validation of PRs from forks the push task is not ran
for those builds either.

**Parameters**:
- `username` - git username
- `commitMessage`
- `displayName` - name to be displayed for the pushing task
- `condition` - [condition](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/conditions)
on which it depends whether the push task will be executed (default: `suceed()`)
- `skipCI` - determines whether to skip CI on pushed commit (only skips azure pipelines) (default: `true`)
- `sshKeySecureFile` - name of the secure file containing the private key corresponding
to github deploy key that was set up (default: `id_rsa`)

For sample usage take a look at [`pre-commit.yml`](jobs/pre-commit.yml) job.

#### [`deadsnakes-install.yml`](tasks/deadsnakes-install.yml)
Install python interpreters using [deadsnakes ppa](https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa).
Can only be used with Ubuntu images.

**Parameters**:
- `version` - python version to install (default: `"3.8"`)
- `condition` - [condition](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/conditions)
on which it depends whether the push task will be executed (default: `suceed()`)

For sample usage take a look at [`python-tests.yml`](jobs/python-tests.yml) job.

#### [`setup-go.yml`](tasks/setup-go.yml)
Set up go toolchain and environment.

**Parameters**:
- `goVersion` - version of go to use (default: `"1.14"`)
- `goPath` - path to set as `goPath` input to [`GoTool`](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/go-tool?view=azure-devops)
task (default: `$(Agent.HomeDirectory)/go`)

### Jobs 💼
#### [`pre-commit.yml`](jobs/pre-commit.yml)
Runs [`pre-commit`](https://pre-commit.com) on all files and optionally pushes
autofixes back to the branch that triggered the build. For more information look
at the [official documentation](https://pre-commit.com/#azure-pipelines-example).

**Parameters**:
- `python` - python version to be used by pre-commit (default: `"3.8"`)
- `pushAutofixes` - whether to push autofixes (default: `true`)

#### [`python-tests.yml`](jobs/python-tests.yml)
Runs python tests using [`tox`](https://tox.readthedocs.io). Tests can be ran on
multiple versions of python interpreter (`python 3.5-3.8` and `pypy3` on all
platforms as well as `python 3.9` and `3.10` on `linux`) as well as multiple
operating systems (you need to define separate jobs for each operating system).

**Parameters**:
- `toxenvs` - names of the tox environments to run
- `os` - `linux`, `windows` or `macos` (default: `linux`)
- `coverage` - whether to [publish coverage artifact](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/publish-code-coverage-results)
(default: `true`)
- `additionalVariables` - additional job [variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables)
- `preTest` - list of tasks to run before running tests
- `namePostfix` - string to be appended to the name of the job

#### [`sphinx-docs.yml`](jobs/sphinx-docs.yml)
Uses `sphinx-apidoc` to generate documentation for new modules and push them back
to the branch that triggered the build. Uses tox by default (see [sample config](tox.ini)).

**Parameters**:
- `python` - python version (default: `"3.8"`)
- `preGenerate` - list of steps to run before generating documentation
- `generateCommand` - task for generating the docs (default: `script: tox -e docs`)
- `postGenerate` - list of tasks to run after generating the docs

#### [`npm-build.yml`](jobs/npm-build.yml)
Runs a build for a node project (e.g. creating production webpack bundles) and
publishes the resulting artifact.

**Parameters**:
- `node` - node version to use (default: `"12.x"`)
- `artifactName`
- `artifactPath` - path to the artifact to be published
- `preInstall` - list of tasks to run before installing npm packages
- `installCommand` - command to run for installing npm packages - useful if you
want to use `yarn` instead of `npm` (default: `npm install`)
- `buildCommand` - command to run for building the artifact (default: `npm run build`)
- `additionalVariables` - additional job [variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables)
- `namePostfix` - string to be appended to the name of the job

#### [`sync-artifact-to-S3.yml`](jobs/sync-artifact-to-S3.yml)
Syncs an artifact to a directory within an S3 bucket, including deletion of the
files that are no longer present in the artifact.

**Parameters**:
- `artifactName`
- `bucketPath` - path to the directory where to upload the artifact starting with
the bucket name
- `dependsOn` - job or a list of jobs this job depends on
- `condition` - [condition](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/conditions)
on which it depends whether the artifact will be synced
(default: `and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master'))`)

**Example**:
```yaml
- template: jobs/sync-artifact-to-S3.yml@aleksac
    parameters:
      artifactName: artifact
      bucketPath: foo/bar
      dependsOn: build
      condition: succeeded()
```
In the example above the contents of the directory `bar` in the bucket `foo` would
be synced with the contents of the artifact `artifact` if a job named `build`
succeeded before it.

#### [`go-tests.yml`](jobs/go-tests.yml)
Runs golang tests with coverage. Tests can be ran for multiple versions of go compiler
as well as multiple operating systems (you need to define separate jobs for each operating system).
For coverage to work you need to run the job from the root of the go module.

**Parameters**:
- `goVersions` - versions of go compiler to use (default: `["1.14"]`)
- `packages` - argument to `go test` (default: `./...`)
- `os` - `linux`, `windows` or `macos` (default: `linux`)
- `coverage` - whether to [publish coverage artifact](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/publish-code-coverage-results)
(default: `true`)
- `additionalVariables` - additional job [variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables)
- `preTest` - list of tasks to run before running tests
- `namePostfix` - string to be appended to the name of the job

## Contact 🙋‍♂️
- [Personal website](https://aleksac.me)
- <a target="_blank" href="http://twitter.com/aleksa_c_"><img alt='Twitter followers' src="https://img.shields.io/twitter/follow/aleksa_c_.svg?style=social"></a>
- aleksacukovic1@gmail.com
