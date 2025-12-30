# GitHub Actions Ubuntu Runner Disk Space Analysis

This repository demonstrates how to analyze and optimize disk space on GitHub Actions `ubuntu-latest` runners.

## Overview

GitHub Actions `ubuntu-latest` runners come with many pre-installed tools and SDKs. While convenient, these can consume significant disk space (typically starting with ~25-30GB free out of ~84GB total). For builds that generate large artifacts or require significant disk space, this can be insufficient.

## What This Workflow Does

The workflow in `.github/workflows/blank.yml` performs:

1. **Initial disk space analysis** - Shows current disk usage
2. **Directory analysis** - Identifies large directories in `/usr`, `/opt`, `/var`
3. **Package inventory** - Lists the largest installed packages
4. **Docker image listing** - Shows any pre-existing Docker images
5. **Space optimization demonstration** - Removes unnecessary tools and shows space gained

## Common Space Consumers on ubuntu-latest

Based on typical ubuntu-latest runner images, here are the main space consumers:

- **Android SDK** (`/usr/local/lib/android`): ~10-12 GB
- **.NET SDKs** (`/usr/share/dotnet`): ~2-3 GB
- **Swift** (`/usr/share/swift`): ~1-2 GB
- **Hostedtoolcache** (`/opt/hostedtoolcache`): Varies (Python, Node.js, Ruby versions)
- **Docker images**: Varies depending on pre-cached images
- **APT cache** (`/var/cache/apt`): ~100-500 MB

## Space Optimization Strategies

### 1. Remove Unnecessary SDKs and Tools

```yaml
- name: Free up disk space
  run: |
    # Remove Android SDK (if you don't need Android builds)
    sudo rm -rf /usr/local/lib/android
    
    # Remove .NET (if you don't need .NET builds)
    sudo rm -rf /usr/share/dotnet
    
    # Remove Swift (if you don't need Swift builds)
    sudo rm -rf /usr/share/swift
```

### 2. Clean Package Manager Cache

```yaml
- name: Clean package cache
  run: |
    sudo apt-get clean
    sudo apt-get autoremove -y
```

### 3. Remove Unused Docker Images

```yaml
- name: Clean Docker
  run: |
    docker system prune -af
```

### 4. Use Specific Hosted Tool Versions

Instead of keeping all cached versions in `/opt/hostedtoolcache`, you can remove versions you don't need:

```yaml
- name: Clean old Python versions
  run: |
    # Keep only Python 3.11, remove others
    sudo rm -rf /opt/hostedtoolcache/Python/3.9.*
    sudo rm -rf /opt/hostedtoolcache/Python/3.10.*
```

### 5. Use a Cleanup Action

For convenience, you can use community actions like `jlumbroso/free-disk-space`:

```yaml
- name: Free Disk Space
  uses: jlumbroso/free-disk-space@v1.3.1
  with:
    android: true
    dotnet: true
    haskell: true
    large-packages: true
    docker-images: true
    swap-storage: true
```

Note: Always use a specific version tag instead of `@main` for production workflows to ensure stability.

## Typical Space Gains

By removing unnecessary components, you can typically gain:

- **Minimal cleanup** (APT cache only): ~500 MB
- **Moderate cleanup** (Android, .NET, Swift): ~15-18 GB
- **Aggressive cleanup** (above + Docker + tool cache): ~20-25 GB

## Running the Analysis

The workflow runs automatically on:
- Push to `main` branch
- Pull requests to `main` branch
- Manual trigger via GitHub Actions UI

View the results in the Actions tab to see:
- Current disk space usage
- What's consuming the most space
- How much space can be freed by removing specific components

## Best Practices

1. **Only remove what you don't need** - Don't blindly remove all tools
2. **Do cleanup early** - Remove unnecessary tools at the start of your workflow
3. **Monitor disk usage** - Add `df -h` commands before and after major build steps
4. **Consider storage needs** - If you need >30GB, plan for cleanup from the start
5. **Use caching wisely** - GitHub Actions cache can consume disk space too

## References

- [GitHub Actions Runner Images](https://github.com/actions/runner-images)
- [Ubuntu 22.04 Image Specifications](https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2204-Readme.md)
- [Disk Space Management Issue](https://github.com/actions/runner-images/issues/2840)