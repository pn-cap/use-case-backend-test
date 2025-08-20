# 🧪 GitHub Actions Pipeline Test

## 📄 Description

This document outlines the steps taken to test the GitHub Actions pipeline by modifying the `evaluation/evaluation.py`. The goal was to verify that the pipeline triggers correctly on pull request events and successfully runs the sample evaluation script.

---

## Pipeline structure

```yaml
# .github/workflows/evaluation.yml
name: Use Remote Evaluation

on:
  pull_request:
    types: [opened, synchronize, reopened]
    paths:
      - 'src/*.py'
      - 'evaluation/evaluation.py'
      - 'src/evaluatio/evaluation.py'

jobs:
  run-evaluation:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install numpy

      - name: Run evaluation.py
        run: python evaluation/evaluation.py
```

## 🚀 Steps Taken

### 1. Create a Feature Branch
```bash
git checkout -b feature/test-pipeline
```

### 2. Update Code
Modify `evaluation.py`:

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


### 6. Merge the Pull Request
```bash
gh pr merge <PR id> --squash
```

### 7. Pull Latest Changes to Local `main`
```bash
git checkout main
git pull origin main
```

### 8. Clean Up Feature Branch
```bash
git branch -d feature/test-pipeline
git push origin --delete feature/test-pipeline
```


## ✅ Expected Outcome

The pipeline was successfully triggered and executed `evaluation.py` as expected. This confirms that the workflow is correctly configured to respond to pull request events and run the evaluation logic.
