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

This workflow remotely triggers the parameterized evaluation pipeline using `workflow_call`.


### 🧪 Manual Trigger
```bash
gh workflow run evaluation_parametrized_call.yml --ref main
```

This command triggers the `evaluation_parametrized_call.yml` workflow, which in turn calls the reusable evaluation workflow with the specified parameters. 

### 🧩 Workflow Chain

1. `evaluation_parametrized_call.yml` is triggered manually, on PR and branch push 
2. It invokes `evaluation_parametrized.yml` via `workflow_call`  
3. Evaluation logic runs with passed parameters  
4. PR comment is posted if applicable  

---

## ✅ Expected Outcome

All three approaches successfully execute `evaluation.py` and validate the pipeline’s ability to respond to PR events, manual triggers, and remote workflow calls. The third approach ensures modularity and reusability across multiple workflows.

