Here's a comprehensive guide to sanitizing GitHub commit history based on government cybersecurity protocols and EFF recommendations:

### 1. **Complete History Purge**
```bash
# Install git-filter-repo
git clone https://github.com/newren/git-filter-repo
export PATH=$PATH:$(pwd)/git-filter-repo

# Remove sensitive files from ALL commits
git filter-repo --path references.json --invert-paths
git filter-repo --path platforms.md --invert-paths
git filter-repo --path raw-data/ --invert-paths
```

### 2. **Pattern-Based Sanitization**
```bash
# Remove API keys/credentials
git filter-repo --replace-text <(echo 'AKIA[0-9A-Z]{16}==>REDACTED')
git filter-repo --replace-text <(echo 'ghp_[a-zA-Z0-9]{36}==>REDACTED')
```

### 3. **Force Push Clean History**
```bash
git push origin --force --all
git push origin --force --tags
```

### 4. **GitHub-Specific Cleanup**
```bash
# Enable secret scanning (requires admin)
gh api -X PATCH /repos/{owner}/{repo} \
  -f 'security_and_analysis.secret_scanning.status="enabled"'
  
# Purge GitHub cache
curl -X PURGE https://github.com/Jeyso215/SocialNet/commits/main
```

### 5. **Collaborator Coordination**
```markdown
1. Notify all contributors via Issues:
   ```github
   @all contributors must:
   - Delete local copies
   - Reclone from sanitized repo
   - Rebase any active branches
   ```
2. Delete all existing forks through GitHub API
```

### 6. **Legal Safeguards**
```bash
# Add takedown notice template
echo "# DMCA Compliance\nContact security@jeyso215.example" > DMCA.md
git add DMCA.md && git commit -m "Add compliance notice"
```

**Critical Considerations:**
- GitHub retains commit SHA hashes for 90 days even after force pushes 
- Public forks may persist unless deleted through GitHub Support 
- Full sanitization requires 3+ force pushes across all branches 
