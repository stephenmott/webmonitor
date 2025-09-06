# GitHub Actions Authentication Fix

## Problem
The Upptime workflows were failing with authentication errors:
```
fatal: could not read Username for 'https://github.com': terminal prompts disabled
```

This error occurred during the checkout step in all Upptime workflows, starting around September 6, 2024.

## Root Cause
The workflows were configured to pass `SECRETS_CONTEXT: ${{ toJson(secrets) }}` to Upptime actions. The repository had an invalid `GH_PAT` secret that was being prioritized over the valid `github.token`.

## Solution
Updated 3 Upptime workflow files to remove the problematic `SECRETS_CONTEXT` lines:

### Files Fixed
1. `.github/workflows/uptime.yml`
2. `.github/workflows/setup.yml` 
3. `.github/workflows/response-time.yml`

### Changes Made
- Removed `SECRETS_CONTEXT: ${{ toJson(secrets) }}` lines from affected workflows
- Kept `GH_PAT: ${{ github.token }}` for proper authentication
- Maintained all existing permissions blocks

## Benefits
1. **Reliability**: Uses the always-available default `github.token`
2. **Simplicity**: Removes dependency on problematic repository secrets
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