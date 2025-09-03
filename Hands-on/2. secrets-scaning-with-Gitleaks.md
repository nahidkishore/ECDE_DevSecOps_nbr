# Gitleaks Installation

```bash
curl -sSL https://github.com/gitleaks/gitleaks/releases/download/v8.18.2/gitleaks_8.18.2_linux_x64.tar.gz -o gitleaks.tar.gz
tar -xvf gitleaks.tar.gz
sudo mv gitleaks /usr/local/bin/gitleaks
sudo chmod +x /usr/local/bin/gitleaks
gitleaks version
```


# pipeline Script for Gitleaks
```groovy

stage('Secrets Scan with Gitleaks') {
    steps {
        echo " Scanning for secrets using Gitleaks..."
        sh '''
            gitleaks detect --source . --redact --no-git -v || echo "⚠️ Gitleaks found issues!"
        '''
    }
}

```




```markdown
# --source .: Scans the current directory.
# --redact: Hides the actual secret in the output for security.
# --no-git: Scans only the current files, not the entire Git history.
# -v: Provides verbose output.

```

---

# method:2 | If Gitleaks found secrets in the code! Failing pipeline.

```groovy

stage('🔐 Secrets Scan with Gitleaks') {
    steps {
        script {
            echo " Scanning for hardcoded secrets using Gitleaks..."

            // Run gitleaks and capture exit status
            def status = sh (
                script: 'gitleaks detect --source . --redact --no-git -v',
                returnStatus: true
            )

            if (status != 0) {
                error " Gitleaks found secrets in the code! Failing pipeline."
            } else {
                echo " No secrets found. Gitleaks scan passed."
            }
        }
    }
}

```

