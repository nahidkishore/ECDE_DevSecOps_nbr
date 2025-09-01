
# Pre-commit Hooks with Gitleaks: Enterprise Implementation

## ✅ Proper Installation on Ubuntu (With Solutions)

### 1. Install Pre-commit Correctly

#### Option A: Recommended (Using apt)
```bash
sudo apt update
sudo apt install pre-commit  # Uses Ubuntu's packaged version
pre-commit --version  # Verify (e.g., 2.21.0)
```

## 2. Install Gitleaks
```bash
# Download binary (Linux)
curl -sSL https://github.com/gitleaks/gitleaks/releases/download/v8.18.2/gitleaks_8.18.2_linux_x64.tar.gz -o gitleaks.tar.gz
tar -xvf gitleaks.tar.gz
sudo mv gitleaks /usr/local/bin/gitleaks
sudo chmod +x /usr/local/bin/gitleaks
gitleaks version
```

## 3. Configure Pre-commit with Gitleaks

### Step 1: Initialize in Your Repo
```bash
cd your-repo/
git init  # You missed this step - pre-commit requires a git repo
ls -la .git  # Verify .git directory exists
# run Pre-commit Installation
pre-commit install  # Creates .git/hooks/pre-commit
# Expected output: "pre-commit installed at .git/hooks/pre-commit"
```

### Step 2: Create `.pre-commit-config.yaml`
```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.2
    hooks:
      - id: gitleaks
        args: [--verbose, --redact]
```

---

## 4. Demo Lab: Production-Grade Testing

```bash
echo 'DB_PASSWORD="supersecret123"' > pass.txt
git add pass.txt
git commit -m "add pass file"
```

### Test 1: Commit a Secret (Should Fail)
```bash
ubuntu@ip-172-31-30-62:~/test-repo$ ls
ubuntu@ip-172-31-30-62:~/test-repo$ echo 'DB_PASSWORD="supersecret123"' > pass.txt
ubuntu@ip-172-31-30-62:~/test-repo$ ls
pass.txt
ubuntu@ip-172-31-30-62:~/test-repo$ git add pass.txt
ubuntu@ip-172-31-30-62:~/test-repo$ git commit -m "add pass file"
```

### Expected output
```
[INFO] Initializing environment for https://github.com/gitleaks/gitleaks.
[INFO] Installing environment for https://github.com/gitleaks/gitleaks.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
Detect hardcoded secrets.................................................Failed
- hook id: gitleaks
- exit code: 1

○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     DB_PASSWORD=REDACTED
Secret:      REDACTED
RuleID:      hashicorp-tf-password
Entropy:     3.327820
File:        pass.txt
Line:        1
Fingerprint: pass.txt:hashicorp-tf-password:1

10:39AM INF 1 commits scanned.
10:39AM INF scan completed in 5.87ms
10:39AM WRN leaks found: 1
```

---