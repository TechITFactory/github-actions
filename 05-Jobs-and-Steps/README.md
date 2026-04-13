# Module 5: Jobs & Steps (Execution Flow)

## Overview
A workflow run is essentially a container for jobs, and jobs contain steps. Understanding how these elements execute relative to one another physically and logically is crucial for building efficient CI/CD pipelines.

**Official Reference:** [About Workflows](https://docs.github.com/en/actions/using-workflows/about-workflows)

---

## Core Rules of Execution

### Jobs run in PARALLEL. Steps run SEQUENTIALLY.
1. **Jobs:** By default, if you specify multiple jobs in a workflow, they will all start at the exact same time on completely separate runner machines. They share *no state* and cannot see each other's files.
2. **Steps:** Inside a specific job, every step runs sequentially, one after the other. Because they run on the *same runner machine*, steps can easily share data via the local filesystem (e.g., Step 1 builds a binary file, Step 2 uploads that file).

---

## Managing Job Execution Order (Dependencies)
Since jobs run in parallel, you must explicitly tell GitHub if one job relies on another. You do this using the `needs` keyword.

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
  build:
    needs: setup
    runs-on: ubuntu-latest
```
In this scenario, GitHub visually creates a dependency graph where `build` waits for `setup` to succeed before starting.

---

## Conditional Execution (`if`)
You don't always want a job or a step to run. You can use the `if` conditional to skip items based on context logic.

### Conditionals on Steps
```yaml
steps:
  - name: Run only on Main branch
    if: github.ref == 'refs/heads/main'
    run: echo "This only runs on main."
```

### Contextual Status Functions
GitHub Actions provides built in status check functions to control execution flow during failures:
- `success()`: The default behavior. Runs only if previous steps/jobs succeeded.
- `failure()`: Runs ONLY if a previous step/job failed. (Great for alert notifications).
- `always()`: Forces the step/job to run regardless of success or failure.
- `cancelled()`: Runs if the workflow was manually cancelled.

---

## Sharing Data Between Jobs
Because jobs run on different virtual machines, they cannot just read a local file created by a previous job. To pass data between jobs, you must use **Job Outputs** (for strings) or **Artifacts** (for files - covered in Module 8).

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs: # Map the job output to the step output
      my_value: ${{ steps.step1.outputs.data }}
    steps:
      - id: step1 # Add an ID to reference this step
        # Write to the special GITHUB_OUTPUT environment variable
        run: echo "data=hello" >> $GITHUB_OUTPUT 
        
  job2:
    needs: job1
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 1 said: ${{ needs.job1.outputs.my_value }}"
```

---

## Example Workflow: Execution Flow Deep Dive

You can find the standalone code in `execution-flow-demo.yml`.

```yaml
# .github/workflows/execution-flow-demo.yml

name: Execution Flow (Jobs and Steps)

on:
  workflow_dispatch:
    inputs:
      simulate_failure:
        description: 'Simulate a failure in Job 1?'
        type: boolean
        default: false

jobs:
  # Job 1: Generates data
  job1-generate-data:
    runs-on: ubuntu-latest
    
    # 1. Define an output that Job 2 can read
    outputs:
      secret_code: ${{ steps.create-code.outputs.code }}
      
    steps:
      - name: Sequential Step 1
        run: echo "Starting the generation process..."
        
      - name: Setup Output (Step 2)
        id: create-code
        run: |
          # Writing output using the required format
          echo "code=XYZ-890" >> $GITHUB_OUTPUT
          
      - name: Simulate Failure
        # 2. Step level conditional
        if: inputs.simulate_failure == true
        run: exit 1 # The exit 1 command forces the runner to log a failure

  # Job 2: Depends on Job 1
  job2-consume-data:
    # 3. Forces this job to wait for Job 1 to succeed
    needs: job1-generate-data
    runs-on: ubuntu-latest
    steps:
      - name: Read the dependency output
        run: echo "The code from Job 1 was ${{ needs.job1-generate-data.outputs.secret_code }}"

  # Job 3: Disaster Recovery Job
  job3-failure-handler:
    # 4. We want this job to run ONLY if Job 1 fails
    if: failure()
    needs: job1-generate-data
    runs-on: ubuntu-latest
    steps:
      - name: Alert Team
        run: |
          echo "CRITICAL: Job 1 failed!"
          echo "Executing alert procedures..."
```
