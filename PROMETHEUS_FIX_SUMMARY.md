# Prometheus Railway Deployment Fix Summary

## Issues Found & Fixed

### ✅ Issue 1: Dockerfile Naming

**Problem**: Railway's Railpack couldn't detect `dockerfile` (lowercase)
**Fix**: Renamed `dockerfile` → `Dockerfile` (capital D)
**Status**: ✅ Fixed

### ✅ Issue 2: railway.json Configuration

**Problem**: railway.json referenced lowercase `dockerfile`
**Fix**: Updated `dockerfilePath` to `Dockerfile`
**Status**: ✅ Fixed

### ✅ Issue 3: Start Command Conflict

**Problem**: startCommand in railway.json conflicted with Dockerfile CMD
**Fix**: Removed startCommand (Dockerfile CMD handles it)
**Status**: ✅ Fixed

## Files Changed

1. **prometheus/dockerfile** → **prometheus/Dockerfile** (renamed)
2. **prometheus/railway.json** (updated dockerfilePath)

## Current Configuration

### prometheus/Dockerfile

```dockerfile
ARG VERSION=v3.2.1
FROM prom/prometheus:${VERSION}
COPY prom.yml /etc/prometheus/prom.yml
CMD ["--config.file=/etc/prometheus/prom.yml", "--storage.tsdb.path=/prometheus"]
```

### prometheus/railway.json

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### prometheus/prom.yml

✅ Already configured with production scrape targets:

- `api.routeroyal.com/metrics` (RouteRoyal API)
- Prometheus self-monitoring

## Next Steps

### 1. Commit Changes to GitHub Fork

```bash
cd railway-grafana-stack-main
git add prometheus/Dockerfile prometheus/railway.json
git rm prometheus/dockerfile  # Remove old lowercase file
git commit -m "Fix: Rename dockerfile to Dockerfile for Railway detection"
git push origin main
```

### 2. Configure Railway Service

In Railway Dashboard → Prometheus service:

**Settings → Source:**

- Repository: `Elementary-Digital/railway-grafana-stack`
- Root Directory: `prometheus` ✅ (already set)
- Branch: `main`

**Settings → Variables:**

- `VERSION=v3.2.1` (optional, to pin version)

### 3. Verify Deployment

After Railway redeploys, check logs for:

```
Step 1/3 : ARG VERSION=v3.2.1
Step 2/3 : FROM prom/prometheus:${VERSION}
...
Server is ready to receive web requests.
```

### 4. Test Metrics Scraping

In Grafana → Explore → Prometheus:

```promql
up{job="routeroyal-api"}
```

Should return `1` if scraping is working.

## Why This Works

1. **Capital D Dockerfile**: Railway's Docker detection prefers `Dockerfile` over `dockerfile`
2. **railway.json**: Explicitly tells Railway to use Docker builder
3. **Root Directory**: Points Railway to the `prometheus/` directory where Dockerfile lives
4. **No startCommand conflict**: Dockerfile CMD handles startup, no need for Railway override

## Troubleshooting

If Railway still uses Railpack:

1. Check Root Directory is set to `prometheus`
2. Verify `Dockerfile` (capital D) exists in `prometheus/` directory
3. Ensure `railway.json` exists and references `Dockerfile`
4. Try disconnecting and reconnecting the repo in Railway
