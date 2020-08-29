
# Azure Pipeline Skeleton

This skeleton repo contains sample Azure pipeline templates for CI and CD.

## Pipeline templates

The CI pipeline is defined in ci-pipeline.yml. Read more about Azure Pipelines YAML [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema).
The repo's pipeline extends the pipeline templates.
Pipeline Templates let you define reusable content, logic, and parameters. Reaad more about Azure Pipeline Templates [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops).

One line summary of pipeline structure hierarchy (top to bottom):
Pipeline > Stage > Job > Step/Task

To strengthen your fundamentals on Azure DevOps Pipelines, please refer to my [crash course blog post on Azure pipelines](http://abstraction.blog/2020/08/25/azure-pipelines-crash-course).

**This repo contains templates for stages, jobs, tasks and variables.**

### Using templates in a repo

3 options to use the given pipelines templates :

#### Use an existing stage template

- When to use this approach : Your requirements match the pipeline template and don't require any customization.
- View the list of stage templates in the stages folder of ci-pipeline-templates or cd-pipeline-templates.

Example ci-pipeline.yml in your application code repository :

```yml
trigger:
  - master

resources:
  repositories:
    - repository: common
      type: git
      name: ado-org-name/azure-pipeline-templates-skeleton
      ref: refs/heads/master

variables:
  - template: ci-pipeline-templates/variables/vars.yml@common # Template reference

stages:
  - template: ci-pipeline-templates/stages/sample.yml@common # Template reference

```

#### Create an additional stage/job/task template in this repo and use it in your application code repository
  
- When to use this approach : Re-use by other repositories.

Example stage template in this repo :

```yml
stages:
  - stage: Test
    pool:
      vmImage: "ubuntu-latest"
    jobs:
      - job: Test
        steps:
          - template: ../tasks/unit-test.yml
          - template: ../tasks/integration-test.yml
  - stage: Publish
    dependsOn: Test
    condition: and(succeeded(), in(variables['Build.SourceBranch'], 'refs/heads/master'))
    pool:
      vmImage: "ubuntu-latest"
    jobs:
      - template: ../jobs/publish.yml
```

#### Create your own pipeline re-using the smaller pipeline templates of this repo

- When to use this approach : You have a task which is specific to your application code respository and won't be reused by others.
- It is the most flexible way to use templates.

Example ci-pipeline.yml of your application code respository using various task templates :

```yml
trigger:
  - master

resources:
  repositories:
    - repository: common
      type: git
      name: ado-org-name/azure-pipeline-templates-skeleton
      ref: refs/heads/master
      # OR USE ref: refs/tags/v1.0

variables:
  - template: ci-pipeline-templates/variables/vars.yml@common # Template reference

stages:
  - stage: Test
    pool:
      vmImage: "ubuntu-latest"
    jobs:
      - job: Test
        steps:
          - template: pipeline-templates/tasks/create-dependencies.yml@common
          - task: Bash@3
            displayName: "Replace variable words for testing"
            inputs:
              targetType: "inline"
              script: |
                # Add code to run integration tests here
          - template: pipeline-templates/tasks/unit-test.yml@common
          - template: pipeline-templates/tasks/integration-test.yml@common
  - stage: Publish
    dependsOn: Test
    condition: and(succeeded(), in(variables['Build.SourceBranch'], 'refs/heads/master'))
    pool:
      vmImage: "ubuntu-latest"
    jobs:
      - template: pipeline-templates/jobs/publish.yml@common
```

### Pipeline Dependencies

The CI-CD pipeline dependencies :

- Existing pipeline mapped to this file in Azure pipelines.
- .azuredevops folder : This contains the Pull Request template which has placeholders for :
  - Pull request change impact (major/minor/patch) to be used for Semantic Versioning using Git tags.
  - Common content to be automatically used for Git tag description and CHANGELOG.md.
