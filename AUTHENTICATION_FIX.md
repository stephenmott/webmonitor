# GitHub Actions Authentication Fix

## Problem
The Upptime workflows were failing with authentication errors:
```
fatal: could not read Username for 'https://github.com': terminal prompts disabled
```

This error occurred during the checkout step in all Upptime workflows, starting around September 6, 2024.

## Root Cause
The workflows were configured to use `${{ secrets.GH_PAT || github.token }}` for authentication. The `GH_PAT` secret was either:
- Missing
- Expired
- Corrupted

And the fallback to `github.token` was not working properly due to the expression syntax.

## Solution
Updated all 8 Upptime workflow files to use only the default `github.token` instead of the potentially problematic `GH_PAT` secret:

### Files Fixed
1. `.github/workflows/uptime.yml`
2. `.github/workflows/setup.yml`
3. `.github/workflows/site.yml`
4. `.github/workflows/response-time.yml`
5. `.github/workflows/graphs.yml`
6. `.github/workflows/updates.yml`
7. `.github/workflows/update-template.yml`
8. `.github/workflows/summary.yml`

### Changes Made
- Changed `token: ${{ secrets.GH_PAT || github.token }}` to `token: ${{ github.token }}`
- Changed `GH_PAT: ${{ secrets.GH_PAT || github.token }}` to `GH_PAT: ${{ github.token }}`
- Changed `github_token: ${{ secrets.GH_PAT || github.token }}` to `github_token: ${{ github.token }}`

## Benefits
1. **Reliability**: Uses the always-available default `github.token`
2. **Simplicity**: Removes dependency on external secrets
3. **Security**: Default token has appropriate permissions set via the `permissions` block
4. **Maintenance**: No need to manage PAT expiration

## Permissions
All workflows already have the correct permissions configured:
```yaml
permissions:
  contents: write
  pages: write
  issues: write
  pull-requests: write
```

These permissions are sufficient for the `github.token` to perform all required Upptime operations.

## Testing
Once these changes are merged to the master branch, the scheduled workflows should run successfully without authentication errors.