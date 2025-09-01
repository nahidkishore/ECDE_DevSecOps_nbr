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