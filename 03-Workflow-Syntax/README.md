# Module 3: Workflow Syntax Deep Dive

## Overview
YAML (YAML Ain't Markup Language) is the data serialization language we use to tell GitHub Actions what to do. Understanding the structure, syntax blocks, and strict indentation of YAML is vital to creating successful CI/CD pipelines.

**Official Reference:** [Workflow Syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## Core Syntax Components

### 1. `name:`
*(Optional but highly recommended)* 
The name of your workflow. GitHub displays the names of your workflows on your repository's actions page.

### 2. `on:`
*(Required)* 
Defines the **event** or **events** that trigger the workflow.
- **Single event:** `on: push`
- **Multiple events:** `on: [push, pull_request]`
- **Events with activity types or deep filters:**
  ```yaml
  on:
    push:
      branches:
        - main
        - 'releases/**'
      paths:
        - 'src/**' # Only trigger if files inside src/ were changed
  ```

### 3. `env:`
*(Optional)*
A map of environment variables that are available to all jobs and steps in the workflow. You can also override or define `env` at the job level or step level.
```yaml
env:
  GLOBAL_VAR: "I am available everywhere"
```

### 4. `jobs:`
*(Required)*
A workflow run is made up of one or more jobs. **By default, jobs run in parallel.**
- **`<job_id>:`**: A unique string identifier for the job (e.g., `build`, `test`, `deploy`).
- **`runs-on:`** *(Required)* The type of machine to run the job on (e.g., `ubuntu-latest`, `windows-latest`).
- **`needs:`** Identifies any jobs that must complete successfully before this job will run (This allows you to create **sequential execution** pipelines).

### 5. `steps:`
*(Required within a job)*
A job contains a sequence of tasks called `steps`.
- **`name:`**: A friendly name for the step in the UI logs.
- **`uses:`**: Calls a pre-built Action (like checking out code or setting up PHP/Node).
- **`run:`**: Runs command-line programs using the operating system's shell. Use the pipe character `|` to create a multi-line shell script block.

---

## Example Workflow: Syntax Structure

You can find the standalone code in `syntax-demo.yml`. This example perfectly illustrates how to use `needs:` to control parallel vs. sequential execution.

```yaml
# .github/workflows/syntax-demo.yml

name: Workflow Syntax Deep Dive

# 1. Trigger section with complex filters
on:
  push:
    branches:
      - main
  workflow_dispatch:

# 2. Global Environment variables
env:
  APP_NAME: "My Awesome App"

# 3. Jobs definition
jobs:

  # Job A: The Build Phase
  build-job:
    runs-on: ubuntu-latest
    
    # Job-level environment variables
    env:
      BUILD_MODE: "production"
      
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Compile Application
        run: |
          echo "Compiling $APP_NAME in $BUILD_MODE mode..."
          sleep 5
          echo "Build successful!"

  # Job B: The Test Phase
  test-job:
    runs-on: ubuntu-latest
    # Use 'needs' to make this job wait for the 'build-job' to complete successfully
    needs: build-job
    
    steps:
      - name: Run Unit Tests
        run: echo "Running tests for $APP_NAME... All tests passed!"

  # Job C: An Independent Job
  linting-job:
    runs-on: ubuntu-latest
    # This job has no 'needs', so it will run in PARALLEL with 'build-job' as soon as the workflow starts.
    steps:
      - name: Run Linters
        run: echo "Linting codebase... No formatting errors found."
```

## How to Execute It

1. **Relocate the File:**
   Copy `syntax-demo.yml` into your repository's `.github/workflows/` path.
2. **Commit and Push:**
   ```bash
   git add .github/workflows/syntax-demo.yml
   git commit -m "Add syntax deep dive workflow"
   git push origin main
   ```
3. **Observe the Execution Layout:**
   - Go to the "Actions" tab in your repository and open the run.
   - You will see a visual dependency graph!
   - You will notice that `build-job` and `linting-job` start executing at exactly the same time (parallel). 
   - The `test-job` is visually linked to `build-job` and will wait idly until `build-job` succeeds before it spins up a runner.
