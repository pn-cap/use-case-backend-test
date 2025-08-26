# 🧪 GitHub Actions Pipeline Test

## 📄 Overview

This document outlines three approaches to testing and running the GitHub Actions pipeline for evaluating `evaluation/evaluation.py`. 

| Approach | Description |
|---------|-------------|
| 1️⃣ Standard | Basic pipeline triggered by PR events |
| 2️⃣ Extended | Parameterized pipeline with manual and reusable triggers |
| 3️⃣ Extended + Remote Trigger | Parameterized pipeline triggered via `workflow_call` using `evaluation_parametrized_call.yml` |

---

## 1️⃣ Standard Pipeline (Non-Parametrized)

### 🔧 Workflow: `.github/workflows/evaluation.yml`

This basic pipeline runs automatically on pull request events and executes a fixed evaluation script.

### 🔁 Pipeline Steps

- Checkout repository  
- Set up Python  
- Install dependencies  
- Run `evaluation.py`  
- Comment on PR with results  

### 🛠️ Setup & Execution

#### Create Feature Branch
```bash
git checkout -b feature/test-pipeline
```

#### Modify Evaluation Script
```python
# evaluation/evaluation.py
with open("results.md", "w") as f:
    f.write("## All checks passed!\n  ")
    f.write("test 1")
```

#### Commit & Push
```bash
git add evaluation/evaluation.py
git commit -m "Update evaluation evaluation.py"
git push -u origin feature/test-pipeline
```

#### Create Pull Request
```bash
gh pr create --base main --head feature/test-pipeline \
  --title "Test pipeline with evaluation.py" \
  --body "This PR updates the workflow to run evaluation.py directly for testing purposes."
```

#### Monitor Workflow
```bash
gh run list
gh run view <run-id>
gh run view --job=48480668091
gh run view --log --job=48480668091
```

#### Merge PR & Cleanup
```bash
gh pr merge <PR id> --squash --delete-branch
git checkout main
git pull origin main
git branch -d feature/test-pipeline
git push origin --delete feature/test-pipeline
```

---

## 2️⃣ Extended Pipeline (Parametrized)

### 🔧 Workflow: `.github/workflows/evaluation_parametrized.yml`

This version introduces flexibility via input parameters and supports manual or reusable triggers.

### ✨ Features

- Manual trigger via GitHub CLI or Actions tab  
- Reusable via `workflow_call`  
- Customizable inputs  
- PR commenting support  

### 🔣 Workflow Inputs

| Input                | Description                       | Default                    |
|----------------------|-----------------------------------|----------------------------|
| python_version       | Python version to use             | `3.11`                     |
| working_directory    | Working directory for the job     | `./`                       |
| requirements_filepath| Path to requirements.txt          | `./requirements.txt`       |
| evaluation_filepath  | Path to evaluation script         | `./evaluation/evaluation.py`|

### 🧪 Manual Trigger
```bash
gh workflow run evaluation_parametrized.yml \
  --ref main \
  --field python_version="3.11" \
  --field working_directory="./" \
  --field requirements_filepath="./requirements.txt" \
  --field evaluation_filepath="./evaluation/evaluation.py"
```

### 🧩 Workflow Steps

1. Checkout repository  
2. Set up Python (parameterized)  
3. Install dependencies  
4. Run evaluation script  
5. Comment on PR with results  

---

## 3️⃣ Extended Pipeline with Remote Trigger

### 🔧 Workflow: `.github/workflows/evaluation_parametrized_call.yml`
This workflow demonstrates how to orchestrate both local and remote evaluation pipelines using `workflow_call`. It can be triggered manually, on PR events, or on pushes to feature branches.

#### ✨ Features

- Manual trigger via GitHub Actions tab or CLI
- Automatic trigger on PRs and feature branch pushes
- Runs both a local evaluation workflow and a remote workflow from another repository
- Passes parameters and secrets to both workflows

#### 🧪 Manual Trigger

```bash
gh workflow run evaluation_parametrized_call.yml --ref main
```

#### 🧩 Workflow Chain

1. **Trigger:** The workflow can be started manually, by PR events, or by pushes to feature branches.
2. **Local Evaluation:**  
   - Runs the local parameterized evaluation workflow (`evaluation_parametrized.yml`) with specified inputs.
3. **Remote Evaluation:**  
   - Invokes a remote workflow (`evaluation_parametrized_remote.yml`) from the [`pn-cap/remote-workflows-test`](https://github.com/pn-cap/remote-workflows-test) repository, passing the same parameters and secrets.
4. **PR Commenting:**  
   - Both workflows can post results as comments on the PR if applicable.


---

## ✅ Expected Outcome

All three approaches successfully execute `evaluation.py` and validate the pipeline’s ability to respond to PR events, manual triggers, and remote workflow calls. The third approach ensures modularity and reusability across multiple workflows.

