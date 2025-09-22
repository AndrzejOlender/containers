# 🚀 Auto Build System

This repository includes an automated system for building and publishing container images to GitHub Container Registry.

## 📋 How it works

### 1. **Upstream Sync** (`sync-upstream.yml`)
- Runs daily at 3 AM UTC
- Automatically syncs with the main Bitnami containers repository
- Detects changes and triggers auto-build if needed
- Can be run manually with dry-run option

### 2. **Auto Build** (`auto-build-images.yml`)  
- Triggers when Dockerfiles change (push events)
- Can be run manually via GitHub Actions UI
- Builds only changed images from `build-preferences.txt` (selective builds)
- **🛡️ Security scanning with Trivy** - blocks vulnerable images
- Publishes to GitHub Container Registry (`ghcr.io`) only if security scan passes
- Supports multi-architecture builds (amd64/arm64)

> **Note**: Scheduled builds removed to save resources - builds only when needed!
> **Security**: Images with CRITICAL vulnerabilities are NOT published!

## 🛡️ Security Scanning

All images are automatically scanned for security vulnerabilities using **Trivy** before publication:

- **🔍 Scan Level**: CRITICAL severity vulnerabilities
- **🚫 Block Policy**: Images with critical vulnerabilities are NOT published
- **📊 Reports**: Security scan results are uploaded to GitHub Security tab
- **🔄 Process**: Build → Scan → Publish (only if secure)

### Security Scan Results

You can view detailed security reports in:
- **GitHub Actions logs** - Real-time scan output
- **Security tab** - SARIF reports with vulnerability details
- **Build summaries** - Quick pass/fail status

## 🎯 Configuration

### Adding/Removing Images

Edit `.github/build-preferences.txt`:

```bash
# Add applications you want to auto-build
multus-cni
redis
mastodon
minio
thanos
# Comment out or remove unwanted images
# mongodb
```

### Manual Triggers

#### Sync with upstream:
```bash
# Dry run to see what would change
gh workflow run sync-upstream.yml -f dry_run=true

# Force sync even if no changes detected  
gh workflow run sync-upstream.yml -f force_sync=true
```

#### Build images:
```bash
# Build all configured images
gh workflow run auto-build-images.yml

# Force rebuild all images
gh workflow run auto-build-images.yml -f force_rebuild=true
```

## 📦 Using Built Images

Your images will be available at:
```bash
# Pull latest version
docker pull ghcr.io/YOUR_USERNAME/containers/IMAGE_NAME:latest

# Pull specific version
docker pull ghcr.io/YOUR_USERNAME/containers/IMAGE_NAME:VERSION

# Examples:
docker pull ghcr.io/andrzejolender/containers/redis:latest
docker pull ghcr.io/andrzejolender/containers/multus-cni:4.2.2
docker pull ghcr.io/andrzejolender/containers/mastodon:latest
```

## 🔧 Image Discovery Logic

The system automatically finds the **latest version** of each application:

1. Reads application names from `build-preferences.txt`
2. Scans `bitnami/APP_NAME/` for available versions
3. Selects the highest version number that contains a `Dockerfile`
4. Builds from `bitnami/APP_NAME/VERSION/OS/Dockerfile`

## 📊 Monitoring

### GitHub Actions
- Check the **Actions** tab for build status
- Each workflow provides detailed summaries
- Failed builds show specific error information

### Container Registry
- Visit `ghcr.io/YOUR_USERNAME/containers` to see published images
- Each image includes metadata and build information

### Notifications
- Workflow summaries show what was built/synced
- Failed builds are clearly marked with error details

## 🛠️ Troubleshooting

### Common Issues

**Build fails for specific image:**
- Check if the Dockerfile exists and is valid
- Verify the application is available in the Bitnami repository
- Look at the build logs for specific error messages

**Sync conflicts:**
- Manual intervention required when upstream changes conflict
- Follow the instructions in the workflow summary
- Resolve conflicts locally and push

**No images found:**
- Verify `build-preferences.txt` contains valid application names
- Check that the applications exist in `bitnami/` directory
- Ensure Dockerfiles are present in the expected locations

### Manual Override

If you need to build a specific image manually:

```bash
# Navigate to the image directory
cd bitnami/APP_NAME/VERSION/OS/

# Build locally
docker build -t ghcr.io/YOUR_USERNAME/containers/APP_NAME:VERSION .

# Push to registry
docker push ghcr.io/YOUR_USERNAME/containers/APP_NAME:VERSION
```

## 🔒 Security

- Uses GitHub's built-in `GITHUB_TOKEN` for authentication
- Images are built in isolated GitHub Actions runners
- Multi-architecture builds ensure compatibility
- All images include security metadata and labels

## 📈 Scaling

To handle more images:
1. Add them to `build-preferences.txt`
2. Adjust `max-parallel` in workflows if needed
3. Monitor build times and resource usage
4. Consider splitting into multiple workflows for very large sets

---

**Need help?** Check the workflow logs in the Actions tab or create an issue.
