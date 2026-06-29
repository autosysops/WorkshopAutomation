# WorkshopAutomation

Welcome to the workshop about automation for Azure DevOps and GitHub. This repository will help you through the workshop.

## Exercise 1: Create a Azure DevOps organization and project (12 min)

Create an Azure DevOps organization via [this link](https://go.microsoft.com/fwlink/?LinkId=307137). If you don't have an Azure Subscription you can create a free trail for Azure. No billing will be applied. If you already have an Azure DevOps organizatin you can skip this step. If you can't create a trail Azure subscription check with other participants in the workshop. Every organization comes with 5 free licenses. Ask if you can share the organization.

Once you created the organization create a project inside this organization. (if you share the organization make sure both of you have a different project)

## Exercise 2: Create your first pipeline (3 min)

In Azure DevOps first go to your Repositories and initialize the repository.
Now go to pipelines and create a new pipeline. Select the Azure Repo's version and select the repo you just initialized. Select the starter pipeline. You will see something like this appear.

```YAML
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'
```

Click save and run. Give it a proper commit message and see the pipeline run.

Note that if the pipeline fails due to an agents not being found this is because the project is still provisioning. Wait a couple of minutes for this to finish.

If the job is queing very long it can help to start the pipeline again and then cancel the previous run.

## Exercise 3: Create a self hosted agent (8 min)

In your project settings go to the Agent Pools and create a new pool of the type self hosted.
Once it's created go in there and create an agent.

Follow the instructions, keep in mind that the script used require permission directly on the C drive and assumes you downloaded the installer to your downloads folder. You can change the script to have it work in a different folder.

The server URL is something like this: `https://dev.azure.com/WorkshopDWX` Where the last part is the name of your organization.
Use the PAT authentication (so press enter in the next step).
In your project you can go to the top right to your settings and select Personal Access Tokens there. You can create a PAT there. For the workshop purposes I recommend creating one with Full Access. If you do this for production workloads make sure you scope the permissions right. Make sure you copy the PAT token after creation and enter this in the config.
For the pool enter the name of the pool you created. For the name you can decide a name.
Keep the work folder default and don't run the worker as a service. Also don't run the agent on startup.
Once the config is done start the agent.

Go back to your pipeline and change the Pool in your pipeline to the pool you just created. Like this:

```YAML
pool: Self Hosted
```

Run the pipeline (it should start by itself already due to a change) and check the permissions request. Allow this.
Keep an eye on the terminal window were your agent is running to see if the job is found.

## Exercise 4: Jobs (6 min)

Make the following changes to the pipeline:

- Change the trigger to "none"
- Remove any steps that exist
- Create two jobs (job1 and job2)
- In both jobs run a PowerShell 7 step with the following code:

```PowerShell
1..10 | Foreach-Object { Write-Host "Count $_ and wait 5 seconds"; Start-Sleep -Seconds 5}
```

After you made the changes run the pipeline. You will see the jobs run.

## Exercise 5: Jobs on different pools (5 min)

Make the following changes to the pipeline:

- run job1 on the self hosted pool
- run job2 on the ubuntu-latest azure pool

When running the pipeline notice that the jobs run in parallel

## Exercise 6: Depends on (5 min)

Make the following chages to the pipeline:

- Add a job3 with the same PowerShell script as step
- Make sure job3 runs when both job1 and job2 are done
- Have job3 run on the self-hosted runner
- Change the powershell script in job1 to:

```PowerShell
Write-Host "Hello World"
```

You should see that job1 is done very quickly while job2 takes a while to finish. The pool for Self hosted is available already but it waits untill job2 is done before it starts executing job3.

## Exercise 7: Depends on Stages (8 min)

Make the following changes to the pipeline:

- make every job (1,2 and 3) a seperate stage with 1 job in them.
- make sure stage3 is depending on stage1 and stage 2

Run the pipeline and notice it now shows a visual representation of the dependencies.

## Exercise 8: Variables (7 min)

Make the following changes to the pipeline:

- in every stage define a variable named "countTo"
- make sure every stage runs the PowerShell script which counts numbers again.
- adept the script so that the `1..10` is now 1 to the number defined in the variable.
- set a different number for every stage and see if it counts to this number.
- add a displayname to the step which shows to what number needs to be counted

Check that you see the display name reflect the nummer you set for the variable in the stage and check that the powershell script counts up correctly.

## Exercise 9: Variable Groups (10 min)

Make the following changes to the pipeline:

- Create 3 variable groups named group1, group2 and group3
- In every variable group create a variable named countTo and give it a number
- Change the stages in the pipeline to use the variable groups (so stage1 uses group1, stage 2 uses group2 etc)
- Remove the manually assigned variables

When running the pipeline keep in mind that you will need to permit the use of the variable groups the first time.

Check that you see the display name reflect the nummer you set for the variable in the stage and check that the powershell script counts up correctly.

## Exercise 10: Templates (15 min)

Make the following changes to the pipeline:

- Create a template that has a job
- The job will run the PowerShell script
- The job will run on the ubuntu-latest vmimage
- The job will use variable group 1

## Exercise 11: Parameters in Templates (5 min)

Make the following changes to the pipeline:

- Create a parameter named groupname for the variable group name.
- Make sure each stage has the right variable group name again by setting it as a parameter.

## Exercise 12: Expressions (10 min)

Make the following changes to the pipeline:

- Create another parameters called pool
- Create an if statement where if the pool is empty you use the ubuntu-latest else you use the pool named in the parameter.
- Add the parameter in stage1 and stage3 and set it to the name of your self hosted host pool

## Exercise 13: Pass variables (10)

Make a new pipeline like this and fill in the fields marked with ***. Eventually it should say Hello DWX2026 in stage2 job1

```YAML
trigger:
- none

pool:
  vmImage: ubuntu-latest

stages:
- stage: stage1
  jobs:
  - job: job1
    steps:
    - task: PowerShell@2
      name: setvar
      inputs:
        targetType: 'inline'
        script: 'Write-Host "##vso[task.*** variable=var1;isOutput=***]DWX2026"'
        pwsh: true
  - job: job2
    dependsOn: job1
    variables:
      job1var1: $[ dependencies.***.outputs['***.***'] ]
    steps:
      - task: PowerShell@2
        name: setvar2
        inputs:
          targetType: 'inline'
          script: |
            Write-Host "##[debug] var1 = $(***)"

            Write-Host "##vso[task.*** variable=var2;isOutput=***]Hello $(***)"
- stage: stage2
  dependsOn: stage1
  jobs:
  - job: job1
    variables:
      stage1job2var2: $[ stageDependencies.***.***.outputs['***.***'] ]
    steps:
    - task: PowerShell@2
      inputs:
        targetType: 'inline'
        script: |
          Write-Host "##[debug] var2 = $(***)"

          Write-Host "##[warning] var2 = $(****)":
```

## Exercise 14: Create GitHub Action (5 min)

Create a GitHub account if you don't have any yet and create a private repository.

Go to the actions tab and create a simple workflow. This should create a file like this:

```YAML
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the "main" branch
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v4

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Runs a set of commands using the runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.
```

The action will trigger instantly. Check how it runs.

## Exercise 15: Create GitHub Self Hosted Runner (8 min)

Go to the Actions tab and on the left select runners and click the button to create a new runner. Then click the button to create a new self-hosted runner and follow the steps.
You can use the default settings.

Start the runner and change the action so that it runs on the self-hosted runner. Keep in mind that changes will trigger the action to run instantly again.

Check that you see the job running in your agent on the device.

## Exercise 16: Writing to github (10 min)

Create a new file in your repo called data.json with the content

```JSON
[]
```

Make the following changes to the workflow:

- remove the triggers except the manual dispatch
- Add these line on the main level of the YAML

```YAML
permissions:
  contents: write
```

- Add this PowerShell step:

```YAML
- name: Get data
  shell: pwsh
  run: |
    $usage = (Get-Counter -Counter "\Processor(_Total)\% Processor Time").Readings.split(":")[1].trim()
    Write-Host "usage = $usage"
    [string[]] $data = (Get-Content ./data.json -Raw) | ConvertFrom-Json
    Write-Host "amount of datapoints = $($data.count)"
    $data += $usage
    Write-Host "new amount of datapoints = $($data.count)"
    $data | ConvertTo-Json -AsArray | Out-File ./data.json -Force -Encoding utf8
```

- Add the following step to the action

```YAML

- name: Commit changes
  run: |
    git config user.name "github-actions"
    git config user.email "github-actions@users.noreply.github.com"
    git add data.json
    git commit -m "Update CPU usage data" || echo "No changes"
    git push
```

Now run the action manually and see if it will add a value to the array. Try it a couple of times.

## Bonus Exercise: Cron

Change the pipeline trigger so that it runs every 5 minutes.
