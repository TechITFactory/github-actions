# Module 6: Runners (Execution Environments)

## Overview
A runner is the server infrastructure that executes the separate jobs in your GitHub Actions workflow. Every job must define exactly *where* it runs using the `runs-on` keyword. There are two primary categories of runners: **GitHub-hosted** and **Self-hosted**.

**Official Reference:** [About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)

---

## 1. GitHub-Hosted Runners
GitHub provides fully managed, fresh virtual machines for every job execution. Once the job completes, the VM is instantly destroyed.

### Common Operating Systems Available:
- **Linux:** `ubuntu-latest`, `ubuntu-24.04`, `ubuntu-22.04`
- **Windows:** `windows-latest`, `windows-2022`, `windows-2019`
- **macOS:** `macos-latest`, `macos-14`, `macos-13`

### Why use GitHub-hosted?
- **Zero Maintenance:** No infrastructure to manage, patch, or scale.
- **Clean Slate:** Every run starts with a completely fresh OS instance, removing "it works on my machine" phantom bugs.
- **Pre-installed Software:** They come pre-loaded with hundreds of common SDKs and tools (Docker, Node, Python, AWS CLI, Kubectl, etc.).

*Cost Note:* Public repositories get unlimited free runner minutes. Private repositories share a pool of free minutes depending on your GitHub tier. Be warned: macOS runners consume your minute pool at a 10x multiplier compared to Linux runners!

---

## 2. Self-Hosted Runners
You can host your own runners on your own physical machines, cloud VMs (like AWS EC2 or Azure VM), or running constantly inside Kubernetes clusters using tools like ARC (Actions Runner Controller).

### Why use Self-hosted?
- **Internal Network Boundaries:** If you need to deploy code to private servers, modify databases, or hit APIs that are NOT exposed to the public internet, a self-hosted runner placed securely inside your VPC resolves this instantly.
- **Custom Hardware:** Need massive GPU power for training ML models? Or 256GB of RAM for an enterprise C++ compilation? You provide the hardware.
- **Persistent Storage:** Because the machine is not automatically destroyed after every run, you can persist heavy local caches natively.

### How to Target Them
You target specific self-hosted runners by passing an array of custom labels to `runs-on`:
```yaml
jobs:
  my-heavy-job:
    runs-on: [self-hosted, linux, x64, gpu-enabled]
```

---

## Example Workflow: Multi-OS Build Matrix

To demonstrate runners in an advanced way, let's look at `runners-demo.yml`.

We will use a `matrix` strategy to execute the exact same job instructions natively across Linux, Windows, and macOS simultaneously.

```yaml
# .github/workflows/runners-demo.yml

name: Runner Matrix Demonstration

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  # A single job definition that will "fan-out" into 3 different jobs in the UI!
  multi-os-build:
    # 1. Define the strategy matrix
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        
    # 2. Assign the 'runs-on' value dynamically to the matrix variable
    runs-on: ${{ matrix.os }}
    
    steps:
      - name: Print OS Specific Details
        # PowerShell is the default shell on windows, bash on linux/mac
        # But echo works on all defaults
        run: |
          echo "Running natively on OS: ${{ runner.os }}"
          echo "Hardware Architecture: ${{ runner.arch }}"
          
      # 3. Running platform-specific commands gracefully using 'if' statements
      - name: Linux specific command
        if: runner.os == 'Linux'
        run: uname -a
        
      - name: Windows specific command
        if: runner.os == 'Windows'
        run: systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
        
      - name: macOS specific command
        if: runner.os == 'macOS'
        run: sw_vers
```
