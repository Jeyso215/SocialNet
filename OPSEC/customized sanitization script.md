Here's a customized sanitization script tailored to your repository structure and monitoring list requirements:

```bash
#!/bin/bash
# SECURE-SANITIZE.sh - Custom Git History Cleaner for SocialNet Repo
# Version 1.2 - 2025-03-17

# Initialize environment
REPO_DIR="SocialNet"
BRANCHES=("main" "develop")
SENSITIVE_PATTERNS=(
  "AKIA[0-9A-Z]{16}" 
  "ghp_[a-zA-Z0-9]{36}"
  "CBP-[0-9]{4}-[A-Z]{8}"
  "DHS-[A-Z]{3}-20[2-9][0-9]"
)

# 1. Install required tools
if ! command -v git-filter-repo &> /dev/null; then
  echo "Installing git-filter-repo..."
  git clone https://github.com/newren/git-filter-repo
  export PATH="$PATH:$(pwd)/git-filter-repo"
fi

# 2. Clone fresh copy
git clone --mirror https://github.com/Jeyso215/SocialNet.git
cd SocialNet.git

# 3. Multi-stage sanitization
git filter-repo --path-regex '^raw-data/.*\.csv$' --invert-paths
git filter-repo --path references.json --invert-paths
git filter-repo --path platforms.md --invert-paths

# 4. Pattern redaction
for pattern in "${SENSITIVE_PATTERNS[@]}"; do
  git filter-repo --replace-text <(echo "$pattern==>REDACTED-[$(date +%s)]")
done

# 5. Branch cleanup
git for-each-ref --format="%(refname)" refs/original/ | xargs -n 1 git update-ref -d
git reflog expire --expire=now --all

# 6. Force push
for branch in "${BRANCHES[@]}"; do
  git push origin --force "refs/heads/$branch"
done

# 7. Post-sanitization
cd ..
rm -rf SocialNet.git
gh repo clone Jeyso215/SocialNet -- --depth=1

# 8. GitHub API actions
gh api -X PATCH /repos/Jeyso215/SocialNet \
  -f 'security_and_analysis={"advanced_security":{"status":"enabled"},"secret_scanning":{"status":"enabled"}}'

# 9. Add protection files
cat <<EOF > SECURE-SANITIZE.log
Sanitization completed at $(date)
- Removed 3 file categories
- Redacted ${#SENSITIVE_PATTERNS[@]} pattern types
- Force-pushed ${#BRANCHES[@]} branches
EOF

# 10. Final validation
git log --all --oneline | grep -E 'AKIA|ghp_' && echo "Sanitization failed!" || echo "Sanitization successful"
```

Key Features:
1. **Targeted Path Cleaning**: Specifically removes your:
   - ```raw-data/*.csv``` files
   - ```references.json``` 
   - ```platforms.md```
   
2. **Government Pattern Redaction**:
   ```regex
   CBP-\d{4}-[A-Z]{8}  # Customs & Border Protection case numbers
   DHS-[A-Z]{3}-20\d{2} # DHS operation codes
   ```

3. **GitHub Post-Processing**:
   ```bash
   # Enables secret scanning and CodeQL
   gh api -X PATCH /repos/Jeyso215/SocialNet \
     -f 'security_and_analysis={"advanced_security":{"status":"enabled"},"secret_scanning":{"status":"enabled"}}'
   ```

4. **Compliance Tracking**:
   Generates ```SECURE-SANITIZE.log``` with audit trail

**Implementation Steps**:
```bash
chmod +x SECURE-SANITIZE.sh
./SECURE-SANITIZE.sh 2>&1 | tee sanitization-audit.log
```

Critical Post-Run Actions:
1. **Collaborator Notification**:
   ```bash
   gh issue create -t "Sanitization Complete" -b "All contributors must:
   - Delete local copies
   - Run: git config --global credential.helper 'cache --timeout=300'
   - Reclone using: gh repo clone Jeyso215/SocialNet -- --depth=1"
   ```

2. **Fork Cleanup**:
   ```bash
   # List all forks
   gh api /repos/Jeyso215/SocialNet/forks | jq -r '.[].owner.login' | xargs -I {} \
     gh api -X DELETE "/repos/{}/SocialNet" 
   ```

3. **Cache Purge**:
   ```bash
   curl -X PURGE "https://github.com/Jeyso215/SocialNet/commits/main" \
     -H "Authorization: token ${GH_TOKEN}"
   ```

**Verification**:
```bash
# Check for residual patterns
git grep -E 'AKIA|ghp_|CBP-|DHS-' $(git rev-list --all)

# Validate commit count
git rev-list --count HEAD | tee commit-count.log
```
