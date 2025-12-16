# The Foundry Guardian

External monitoring system for [the-foundry](https://github.com/nickarrow/the-foundry) repository.

## Purpose

This repository contains the Guardian workflow that monitors `the-foundry` for unauthorized modifications to the enforcement infrastructure. Unlike the original Guardian design (which ran from a protected branch), this external Guardian:

- ✅ **Cannot be disabled by attackers** - They don't have access to this repo
- ✅ **Runs independently** - Scheduled workflow runs every 15 minutes
- ✅ **True external monitoring** - Monitors from outside the target repository
- ✅ **Can restore even if all workflows deleted** - Has canonical copies stored here

## How It Works

### Every 15 Minutes:

1. **Clone `the-foundry`** - Gets latest state
2. **Check workflow integrity** - Compares against canonical versions stored here
3. **If compromised**:
   - Find last known good commit in history
   - Restore workflow files from canonical versions
   - Restore all content from last good commit
   - Push restoration to `the-foundry`
   - Create GitHub Issue alert in `the-foundry`

### Attack Scenario Protection:

```
2:00 PM - Attacker deletes enforcement workflows from the-foundry
2:01 PM - Attacker deletes player content
2:05 PM - Attacker makes more malicious commits

2:15 PM - Guardian runs (from THIS repo):
  - Detects workflows missing
  - Finds last good state (1:59 PM)
  - Restores everything
  - Pushes fix to the-foundry
  - Creates alert issue

2:16 PM - the-foundry is fully restored
```

## Repository Structure

```
the-foundry-guardian/
├── .github/workflows/
│   └── monitor.yml              # Guardian monitoring workflow
├── canonical-workflows/
│   ├── enforce-ownership.yml    # Canonical enforcement workflow
│   └── enforce_ownership.py     # Canonical enforcement script
└── README.md                    # This file
```

## Updating Canonical Workflows

When you legitimately update the enforcement workflows in `the-foundry`:

```bash
# 1. Update the-foundry first
cd the-foundry
# Make your changes to .github/workflows/ or .github/scripts/
git add .github/
git commit -m "Update: [description]"
git push

# 2. Update canonical versions in guardian repo
cd ../the-foundry-guardian
cp ../the-foundry/.github/workflows/enforce-ownership.yml canonical-workflows/
cp ../the-foundry/.github/scripts/enforce_ownership.py canonical-workflows/
git add canonical-workflows/
git commit -m "Update canonical workflows"
git push
```

## Monitoring

- **GitHub Actions**: Check this repo's Actions tab for Guardian runs
- **Attack Alerts**: Check `the-foundry` Issues with label `guardian-alert`
- **Logs**: Each Guardian run shows what it checked and any actions taken

## Security

- **This repository is private** - Only you have access
- **Attackers cannot disable Guardian** - They can't access this repo
- **Token has minimal permissions** - Only what's needed for monitoring
- **Audit trail** - All Guardian actions logged in GitHub Actions and Issues

## Maintenance

- **Token rotation**: Set reminder to rotate PAT annually
- **Update canonical files**: When you update enforcement workflows
- **Monitor runs**: Check Actions tab periodically to ensure Guardian is running

## Status

✅ **Active** - Guardian monitors `the-foundry` every 15 minutes
