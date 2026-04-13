# Module 1: Foundations - CI/CD & GitHub Actions

## Transcript & Core Definitions

### What is CI/CD?
**Continuous Integration (CI)** and **Continuous Deployment/Delivery (CD)** are core DevOps practices that automate the building, testing, and deployment of code.
- **CI**: Automates the merging and testing of code changes to detect bugs early, ensuring the codebase is always in a working state.
- **CD**: Automates the release (Delivery) and actual deployment (Deployment) of validated code to staging or production environments.
*Goal:* Deliver software faster, with higher quality, reduced risk, and less manual intervention.

### What are GitHub Actions?
[GitHub Actions](https://docs.github.com/articles/getting-started-with-github-actions) is a native CI/CD and automation platform built directly into GitHub. It allows you to automate your software development workflows directly from your repositories. You can write individual tasks, called *Actions*, and combine them to create a customized *Workflow*. 

Because it's natively integrated, it has seamless access to repository events (like Pull Requests, Issues, and Pushes).

### Core Components (The Vocabulary)
To master GitHub Actions, you must understand these foundational concepts. They map directly to how you write the YAML files.

1. **Workflow**: A configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked into your repository under a specific directory.
2. **Event**: A specific activity in a repository that triggers a workflow run. For example: a `push` to the main branch, a `pull_request` being opened, or a manual trigger like `workflow_dispatch`.
3. **Jobs**: A set of steps in a workflow that execute on the same runner. Jobs run in *parallel* by default, but can be configured to run sequentially depending on each other.
4. **Steps**: An individual task inside a job. A step can either run a shell command (script) or run a pre-built Action. Steps within a given job run sequentially and share the same environment/runner workspace.
5. **Actions**: Reusable, standalone custom applications for the GitHub Actions platform that perform a complex but frequently repeated task (e.g., checking out your code, or authenticating with AWS/Azure).
6. **Runner**: A server that runs your workflows when they're triggered. GitHub provides hosted virtual machines (Ubuntu, Windows, macOS), or you can host your own custom machines (self-hosted).

---

## Example Workflow YAML

Here is a basic foundational workflow piece to demonstrate the concepts above in structural YAML format.

```yaml
# .github/workflows/foundations-demo.yml

# 1. The name of the workflow as it will appear in the GitHub Actions UI tab.
name: Foundations Demo Workflow

# 2. Events/Triggers: When should this workflow run?
on:
  push:
    branches:
      - main
  # 'workflow_dispatch' allows you to securely trigger the workflow manually from the GitHub UI
  workflow_dispatch: 

# 3. Jobs: What should happen when it runs?
jobs:
  # The ID of the job (e.g., "explore_actions"). A workflow can have multiple jobs.
  explore_actions:
    # 4. Runner: The type of machine OS to run the job on
    runs-on: ubuntu-latest
    
    # 5. Steps: The sequence of tasks to execute inside this job
    steps:
      # Step A: Using a pre-built Action to download (checkout) the repo code to the runner
      - name: Checkout Repository
        uses: actions/checkout@v4

      # Step B: Running a simple shell command
      - name: Print a greeting message
        run: echo "Hello! Welcome to GitHub Actions CI/CD Foundations."

      # Step C: Running a multi-line shell command
      - name: List files in workflow workspace
        run: |
          echo "Current working directory:"
          pwd
          echo "Files in directory:"
          ls -la
```

---

## How to Execute It

Unlike running a local bash script or a Docker container, GitHub Actions workflows are executed **natively by GitHub's infrastructure**. You don't execute them from your local terminal.

### Step-by-Step Execution Guide:

1. **Understand the Path Requirement:**
   GitHub Actions *strictly* looks for workflow YAML files inside a specific designated folder at the root of your repository: `.github/workflows/`. If the YAML file is anywhere else (like inside `01-Foundations`), GitHub will ignore it entirely.

2. **Add the YAML file to your Repository Setup:**
   - From your project's root folder, make sure the `.github/workflows` directory exists.
   - Copy the YAML block from above and save it into a file, for example: `.github/workflows/foundations-demo.yml`.

3. **Commit and Push:**
   To make GitHub recognize the file, you must push it to the server.
   ```bash
   git add .github/workflows/foundations-demo.yml
   git commit -m "Add foundations demo workflow"
   git push origin main
   ```

4. **Triggering the Execution:**
   - **Automatic Trigger:** Because we defined `on: push` for the `main` branch, pushing your commit in Step 3 automatically triggers the execution.
   - **Manual Trigger:** Because we defined `on: workflow_dispatch`, you can trigger it manually at any time:
     1. Go to your repository on GitHub.com.
     2. Click the **"Actions"** tab at the top.
     3. Select **"Foundations Demo Workflow"** from the left sidebar.
     4. Click the **"Run workflow"** drop-down button on the right side.
     5. Click the green **"Run workflow"** button.

5. **Viewing the Output / Logs:**
   Once triggered, an execution run will appear in the list. 
   - Click on the pipeline run.
   - Click on the `explore_actions` job box.
   - You will see a terminal-like view expanding in the browser, showing the real-time execution console logs for each of the *steps* we defined (checking out code, printing the greeting, and listing the files).
