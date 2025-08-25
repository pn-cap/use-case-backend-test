# 🧪 GitHub Actions Pipeline Test

## 📄 Description

This document outlines the steps taken to test the GitHub Actions pipeline by modifying the `evaluation/evaluation.py`. The goal was to verify that the pipeline triggers correctly on pull request events and successfully runs the sample evaluation script.


Pipeline structure

* Checkout repository 
* Set up Python 
* Install dependencies 
* Run evaluation.py 
* Comment on PR with results

see .github/workflows/evaluation.yml for more details

## 🚀 Steps Taken

### 1. Create a Feature Branch
```bash
git checkout -b feature/test-pipeline
```

### 2. Update Code
Modify `evaluation.py`:

(example)
```python
# evaluation/evaluation.py
with open("results.md", "w") as f:
    f.write("## All checks passed!\n  ")
    f.write("test 1")
```

### 3. Commit and Push Changes
```bash
git add evaluation/evaluation.py
git commit -m "Update evaluation evaluation.py"
git push -u origin feature/test-pipeline
```

### 4. Create Pull Request via GitHub CLI
```bash
gh pr create --base main --head feature/test-pipeline \
  --title "Test pipeline with evaluation.py" \
  --body "This PR updates the workflow to run evaluation.py directly for testing purposes."
```

### 5. Monitor Workflow Execution
- Verify that the workflow was triggered on PR creation.
- Checked the **Actions** tab to confirm successful execution of `evaluation.py`.

```bash
gh run list
```

- ✅ Green check = success
- ❌ Red X = failure
- 🟡 Yellow dot = in progress or queue


Check the run logs 
```bash
gh run view <run-id>
```

More details 
```
gh run view --job=48480668091
gh run view --log --job=48480668091
```


---
  
***<TO be checked: updating the PR>    
Essencially add + commit to PRed branch - evaluation should be rerun***

--- 

### 6. Merge the Pull Request
```bash
gh pr merge <PR id> --squash --delete-branch
```

### 7. Pull Latest Changes to Local `main` (optional)
```bash
git checkout main
git pull origin main
```

### 8. Clean Up Feature Branch (optional)
```bash
git branch -d feature/test-pipeline
git push origin --delete feature/test-pipeline
```


## Evaluation Pipeline (parametrized)

This workflow (`.github/workflows/evaluation_parametrized.yml`) provides a flexible evaluation pipeline that can be triggered manually or called from other workflows. It supports parameterization for Python version, working directory, requirements file, and evaluation script.

### Features

- **Manual Trigger:** Run the workflow from the GitHub Actions tab or via GitHub CLI (`gh workflow run`) with custom inputs.
- **Reusable Workflow:** Can be called from other workflows using `workflow_call` and pass inputs.
- **Parameterization:** Easily change Python version, working directory, requirements file, and evaluation script path.
- **PR Commenting:** Automatically posts evaluation results as a comment on the pull request when triggered by a PR event.

### Workflow Inputs

| Input                | Description                       | Default                    |
|----------------------|-----------------------------------|----------------------------|
| python_version       | Python version to use             | `3.11`                     |
| working_directory    | Working directory for the job     | `./`                       |
| requirements_filepath| Path to requirements.txt          | `./requirements.txt`       |
| evaluation_filepath  | Path to evaluation script         | `./evaluation/evaluation.py`|

### Example Usage

#### Manual Trigger (GitHub CLI)
```bash
gh workflow run evaluation_parametrized.yml \
  --ref main \
  --field python_version="3.11" \
  --field working_directory="./" \
  --field requirements_filepath="./requirements.txt" \
  --field evaluation_filepath="./evaluation/evaluation.py"
```

#### Workflow Call (from another workflow)
```yaml
jobs:
  call-evaluation:
    uses: ./.github/workflows/evaluation_parametrized.yml
    with:
      python_version: "3.11"
      working_directory: "./"
      requirements_filepath: "./requirements.txt"
      evaluation_filepath: "./evaluation/evaluation.py"
    secrets: inherit
```

### Workflow Steps

1. **Checkout repository**
2. **Set up Python** (parameterized)
3. **Install dependencies** from the specified requirements file
4. **Run evaluation script** at the specified path
5. **Comment on PR** with results (only for PR events)

#### PR Comment Example

The workflow reads `results.md` and posts its content as a comment on the PR, including the actor and a link to the workflow run.

---

For the full workflow YAML, see `.github/workflows/evaluation_parametrized.yml`.


## ✅ Expected Outcome

The pipeline was successfully triggered and executed `evaluation.py` as expected.  
This confirms that the workflow is correctly configured to respond to pull request events and run the evaluation logic.
