# How to Add a Fine-Grained GitHub Personal Access Token

This guide explains how to add a fine-grained personal access token (PAT) to your Upptime repository for enhanced security and granular permission control.

## Why Use a Fine-Grained Token?

Fine-grained tokens offer several advantages:
- **Enhanced Security**: Restrict access to specific repositories only
- **Minimal Permissions**: Grant only the permissions needed for Upptime operations
- **Better Control**: Expiration dates and revocable access
- **Audit Trail**: Better tracking of token usage

## When Do You Need a Custom Token?

The Upptime workflows in this repository currently use the default `github.token` which works for most use cases. You may want to add a custom fine-grained token if:

- You need to trigger other workflows that require additional permissions
- You want to use a token with a longer expiration than workflow tokens
- Your organization requires specific token policies
- You need to access resources across multiple repositories

## Step-by-Step Setup Guide

### 1. Create the Fine-Grained Token

1. Go to your GitHub settings: [https://github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta)
   - Or navigate: Profile Icon → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens

2. Click **"Generate new token"**

3. Configure the token:
   - **Token name**: Give it a descriptive name (e.g., "Upptime Monitor Token")
   - **Expiration**: Choose an appropriate expiration period (recommended: 90 days)
   - **Resource owner**: Select the owner of this repository (stephenmott)

4. **Repository access**:
   - Select "Only select repositories"
   - Choose the `webmonitor` repository

5. **Permissions** - Grant these repository permissions:
   - **Contents**: Read and write access (required for updating status files)
   - **Issues**: Read and write access (required for creating incident reports)
   - **Pull requests**: Read and write access (required for workflow updates)
   - **Workflows**: Read and write access (required for workflow dispatching)
   - **Pages**: Read and write access (required for deploying status page)

6. Click **"Generate token"**

7. **Important**: Copy the token immediately - you won't be able to see it again!

### 2. Add Token to Repository Secrets

1. Go to your repository settings: `https://github.com/stephenmott/webmonitor/settings/secrets/actions`

2. Click **"New repository secret"**

3. Configure the secret:
   - **Name**: `GH_PAT` (this is the name expected by Upptime workflows)
   - **Secret**: Paste your fine-grained token

4. Click **"Add secret"**

### 3. Update Workflows (if needed)

The current workflows in this repository are already configured to use a custom PAT if available. They look for the `GH_PAT` environment variable and fall back to `github.token` if not set.

Example from `.github/workflows/uptime.yml`:
```yaml
env:
  GH_PAT: ${{ secrets.GH_PAT || github.token }}
```

If your workflows don't have this pattern, you can update them to use the custom token:

```yaml
- name: Check endpoint status
  uses: upptime/uptime-monitor@v1.41.0
  with:
    command: "update"
  env:
    GH_PAT: ${{ secrets.GH_PAT }}
```

### 4. Verify Token is Working

After adding the token, trigger a workflow run to verify it works:

1. Go to Actions tab in your repository
2. Select any workflow (e.g., "Uptime CI")
3. Click "Run workflow" → "Run workflow"
4. Check that the workflow completes successfully

## Required Permissions Summary

For Upptime to work correctly, your fine-grained token needs these permissions on the webmonitor repository:

| Permission | Access Level | Purpose |
|------------|-------------|---------|
| Contents | Read and write | Update history files and status data |
| Issues | Read and write | Create and update incident reports |
| Pull requests | Read and write | Manage automated updates |
| Workflows | Read and write | Trigger workflow runs |
| Pages | Read and write | Deploy status page to GitHub Pages |

## Troubleshooting

### Token Not Working

If workflows fail after adding the token:

1. **Check token expiration**: Verify the token hasn't expired at [https://github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta)

2. **Verify permissions**: Ensure all required permissions are granted to the token

3. **Check repository access**: Confirm the token has access to the webmonitor repository

4. **Secret name**: Verify you named the secret exactly `GH_PAT` (case-sensitive)

5. **Organization approval**: If this repo belongs to an organization, the token may need approval from an admin

### Workflows Still Use Default Token

The workflows will automatically use `secrets.GH_PAT` if available, otherwise fall back to `github.token`. Check workflow run logs to see which token is being used.

### Permission Denied Errors

If you see "403 Forbidden" or permission errors:

1. Review the token permissions - you may need to add additional permissions
2. Check if the token has expired
3. Verify the token is added as a repository secret (not an environment secret)

## Token Rotation and Maintenance

### When to Rotate Tokens

- Before the token expires
- If the token may have been compromised
- As part of regular security practices (every 90 days recommended)

### How to Rotate

1. Create a new fine-grained token following steps above
2. Update the `GH_PAT` repository secret with the new token
3. Delete the old token from [GitHub settings](https://github.com/settings/tokens?type=beta)

## Reverting to Default Token

If you want to stop using a custom token and revert to the default `github.token`:

1. Delete the `GH_PAT` secret from repository settings
2. The workflows will automatically fall back to using `github.token`

Note: The default `github.token` has sufficient permissions for standard Upptime operations but expires after each workflow run.

## Additional Resources

- [GitHub: Managing Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [GitHub: Permissions for Fine-Grained Tokens](https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens)
- [Upptime Documentation](https://upptime.js.org/docs)
- [Authentication Fix Details](./AUTHENTICATION_FIX.md)

## Questions or Issues?

If you encounter problems not covered here, please [open an issue](https://github.com/stephenmott/webmonitor/issues/new) with:
- Description of the problem
- Error messages from workflow logs
- Steps you've already tried
