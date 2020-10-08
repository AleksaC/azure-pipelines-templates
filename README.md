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
      ref: refs/tags/v0.0.1

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

Parameters:
- `username` - git username
- `commitMessage`
- `displayName` - name to be displayed for the pushing task
- `condition` - [condition](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/conditions)
on which it depends whether the push task will be executed (default: `suceed()`)
- `skipCI` - determines whether to skip CI on pushed commit (only skips azure pipelines) (default: `true`)
- `sshKeySecureFile` - name of the secure file containing the private key corresponding
to github deploy key that was set up (default: `id_rsa`)

For sample usage take a look at [`pre-commit.yml`](jobs/pre-commit.yml) job.

### Jobs 💼
#### [`pre-commit.yml`](jobs/pre-commit.yml)
Runs [`pre-commit`](https://pre-commit.com) on all files and optionally pushes
autofixes back to the branch that triggered the build. For more information look
at the [official documentation](https://pre-commit.com/#azure-pipelines-example).

Parameters:
- `python` - python version to be used by pre-commit (default: `"3.8"`)
- `pushAutofixes` - whether to push autofixes (default: `true`)

#### [`python-tests.yml`](jobs/python-tests.yml)
Runs python tests using [`tox`](https://tox.readthedocs.io). Tests can be ran on
multiple versions of python interpreter (`python 3.5-3.8`, `pypy3` and `python 3.9` on `linux`)
as well as multiple operating systems (you need to define separate jobs for each
operating system).

Parameters:
- `toxenvs` - names of the tox environments to run
- `os` - `linux`, `windows` or `macos` (default: `linux`)
- `coverage` - whether to [publish coverage artifact](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/test/publish-code-coverage-results)
(default: `true`)
- `additionalVariables` - additional job [variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables)
- `preTest` - list of tasks to run before running tests
- `namePostflix` - string to be appended to the name of the job

#### [`sphinx-docs.yml`](jobs/sphinx-docs.yml)
Uses `sphinx-apidoc` to generate documentation for new modules and push them back
to the branch that triggered the build. Uses tox by default (see [sample config](tox.ini)).

Parameters:
- `python` - python version (default: `"3.8"`)
- `preGenerate` - list of steps to run before generating documentation
- `generateCommand` - task for generating the docs (default: `script: tox -e docs`)
- `postGenerate` - list of tasks to run after generating the docs

## Contact 🙋‍♂️
- [Personal website](https://aleksac.me)
- <a target="_blank" href="http://twitter.com/aleksa_c_"><img alt='Twitter followers' src="https://img.shields.io/twitter/follow/aleksa_c_.svg?style=social"></a>
- aleksacukovic1@gmail.com
