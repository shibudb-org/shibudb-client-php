# Releasing ShibuDb PHP Client

This document explains how to create a new release that will be recognized by Packagist as a stable version (instead of `dev-main`).

## Quick Release (Using GitHub Actions)

### Option 1: Manual Workflow Dispatch

1. Go to **Actions** → **Release** → **Run workflow**
2. Enter the version (e.g., `1.0.0`)
3. Click **Run workflow**
4. The workflow will:
   - Update `composer.json` version
   - Commit the change
   - Create a git tag (e.g., `v1.0.0`)
   - Create a GitHub Release
   - Push everything to the repository

### Option 2: Create Tag Manually

If you prefer to create the tag manually:

```bash
# Ensure composer.json has the correct version
# Edit composer.json and set "version": "1.0.0"

# Commit the version change
git add composer.json
git commit -m "chore: release version 1.0.0"
git push

# Create and push the tag
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

The GitHub Actions workflow will automatically:
- Detect the tag push
- Verify the version in composer.json
- Create a GitHub Release

## Why Packagist Shows "dev-main"

Packagist shows `dev-main` (or `dev-master`) when:
- It's reading from the default branch (main/master) instead of tags
- No stable version tags exist yet

## Fixing "dev-main" Issue

To make Packagist show version `1.0.0`:

1. **Ensure composer.json has the correct version** in the tag:
   ```json
   {
     "version": "1.0.0"
   }
   ```

2. **Create a git tag**:
   ```bash
   git tag -a v1.0.0 -m "Release 1.0.0"
   git push origin v1.0.0
   ```

3. **Update Packagist**:
   - If auto-update is enabled, Packagist will detect the tag automatically
   - Otherwise, visit https://packagist.org/packages/shibudb-org/shibudb-client-php and click "Update"
   - Or configure a webhook: `https://packagist.org/api/update-package?username=YOUR_USERNAME&apiToken=YOUR_TOKEN`

## Version Format

- Use semantic versioning: `MAJOR.MINOR.PATCH` (e.g., `1.0.0`, `1.0.1`, `1.1.0`, `2.0.0`)
- Tag format: `v1.0.0` (with 'v' prefix) or `1.0.0` (without prefix)
- The workflow accepts both formats

## Verifying the Release

After creating a release:

1. **Check GitHub Releases**: https://github.com/shibudb-org/shibudb-clients/releases
2. **Check Packagist**: https://packagist.org/packages/shibudb-org/shibudb-client-php
3. **Test installation**:
   ```bash
   composer require shibudb-org/shibudb-client-php:^1.0.0
   ```

## Important Notes

- **Always update composer.json version before creating a tag**
- **Tags are immutable** - if you need to fix a tag, create a new patch version
- **Packagist caches** - it may take a few minutes for the new version to appear
- **Default branch** will always show as `dev-{branch-name}` in Packagist - this is normal
