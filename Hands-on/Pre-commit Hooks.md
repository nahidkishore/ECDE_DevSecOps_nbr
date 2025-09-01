# 1 Install Pre-commit
```bash
sudo apt update
sudo apt install pre-commit  # Uses Ubuntu's packaged version
pre-commit --version  # Verify (e.g., 2.21.0)
```

# 2. Install Gitleaks

```bash
# Download binary (Linux)
curl -sSL https://github.com/gitleaks/gitleaks/releases/download/v8.18.2/gitleaks_8.18.2_linux_x64.tar.gz -o gitleaks.tar.gz
tar -xvf gitleaks.tar.gz
sudo mv gitleaks /usr/local/bin/gitleaks
sudo chmod +x /usr/local/bin/gitleaks
gitleaks version
```


# Configure Pre-commit with Gitleaks

### Step 1: Initialize in Your Repo


```bash
cd your-repo/
git init  # You missed this step - pre-commit requires a git repo
ls -la .git  # Verify .git directory exists
# run Pre-commit Installation
pre-commit install  # Creates .git/hooks/pre-commit
# Expected output: "pre-commit installed at .git/hooks/pre-commit"

```

# Step 2: Create .pre-commit-config.yaml


```yml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.2
    hooks:
      - id: gitleaks
        args: [--verbose, --redact]

```