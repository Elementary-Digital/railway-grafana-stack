# Complete Railway Grafana Stack Fix Summary

## Overview

This template deploys **4 separate Railway services** when you click "Deploy on Railway":
1. **Grafana** - Visualization dashboard
2. **Prometheus** - Metrics collection
3. **Loki** - Log aggregation  
4. **Tempo** - Distributed tracing

Each service is deployed independently from its own directory.

## Issues Found & Fixed

### ✅ Issue 1: Dockerfile Naming (All Services)
**Problem**: Railway's Railpack couldn't detect `dockerfile` (lowercase)  
**Fix**: Renamed all `dockerfile` → `Dockerfile` (capital D)  
**Services Fixed**: Grafana, Loki, Tempo, Prometheus  
**Status**: ✅ Fixed

### ✅ Issue 2: Missing railway.json Configurations
**Problem**: Only Prometheus had `railway.json`, others relied on auto-detection  
**Fix**: Added `railway.json` to all service directories  
**Services Fixed**: Grafana, Loki, Tempo  
**Status**: ✅ Fixed

### ✅ Issue 3: Loki Configuration Typo
**Problem**: Loki Dockerfile had typo `loki-confi.yaml` instead of `loki-config.yaml`  
**Fix**: Corrected filename in Dockerfile  
**Status**: ✅ Fixed

### ✅ Issue 4: docker-compose.yml References
**Problem**: docker-compose.yml referenced lowercase `dockerfile`  
**Fix**: Updated all references to `Dockerfile` (capital D)  
**Status**: ✅ Fixed

### ✅ Issue 5: Prometheus Scrape Configuration
**Problem**: Prometheus wasn't configured to scrape production API  
**Fix**: Updated `prom.yml` with production scrape targets  
**Status**: ✅ Fixed

## Files Changed

### Service Directories
```
grafana/
├── Dockerfile          ✅ (renamed from dockerfile)
└── railway.json       ✅ (new)

loki/
├── Dockerfile          ✅ (renamed from dockerfile, fixed typo)
└── railway.json       ✅ (new)

tempo/
├── Dockerfile          ✅ (renamed from dockerfile)
└── railway.json       ✅ (new)

prometheus/
├── Dockerfile          ✅ (renamed from dockerfile)
├── railway.json        ✅ (updated dockerfilePath)
└── prom.yml            ✅ (production scrape config)
```

### Root Files
```
docker-compose.yml      ✅ (updated all dockerfile references)
```

## How Railway Templates Work

When you deploy this template on Railway:

1. **Railway creates 4 separate services** from the 4 directories
2. **Each service** gets its own:
   - Build process (from Dockerfile)
   - Environment variables
   - Persistent volumes
   - Internal networking URL

3. **Grafana service** automatically exposes:
   - `LOKI_INTERNAL_URL`
   - `PROMETHEUS_INTERNAL_URL`
   - `TEMPO_INTERNAL_URL`

4. **Services communicate** via Railway's internal network:
   - `http://grafana.railway.internal:3000`
   - `http://prometheus.railway.internal:9090`
   - `http://loki.railway.internal:3100`
   - `http://tempo.railway.internal:3200`

## Configuration Details

### Prometheus (prometheus/prom.yml)
```yaml
scrape_configs:
  - job_name: 'routeroyal-api'
    scheme: https
    static_configs:
      - targets: ['api.routeroyal.com']
```

### Grafana (grafana/datasources/datasources.yml)
Pre-configured to connect to:
- Prometheus: `http://prometheus:9090`
- Loki: `http://loki:3100`
- Tempo: `http://tempo:3200`

### All Services (railway.json)
```json
{
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  }
}
```

## Next Steps

### 1. Commit All Changes to GitHub Fork
```bash
cd railway-grafana-stack-main

# Stage all changes
git add grafana/Dockerfile grafana/railway.json
git add loki/Dockerfile loki/railway.json
git add tempo/Dockerfile tempo/railway.json
git add prometheus/Dockerfile prometheus/railway.json prometheus/prom.yml
git add docker-compose.yml

# Remove old lowercase files (if they still exist)
git rm grafana/dockerfile loki/dockerfile tempo/dockerfile prometheus/dockerfile 2>/dev/null || true

# Commit
git commit -m "Fix: Rename all dockerfile to Dockerfile and add railway.json configs

- Renamed all dockerfile files to Dockerfile for Railway detection
- Added railway.json to all service directories
- Fixed Loki config typo (loki-confi.yaml -> loki-config.yaml)
- Updated docker-compose.yml references
- Configured Prometheus to scrape production API"

# Push
git push origin main
```

### 2. Deploy on Railway

**Option A: Use Template Button (Recommended)**
1. Go to your GitHub fork
2. Click "Deploy on Railway" button
3. Railway will create all 4 services automatically
4. Set `GF_SECURITY_ADMIN_USER` variable
5. Wait for deployment (3-5 minutes)

**Option B: Manual Service Creation**
If services already exist, Railway should auto-redeploy from your fork.

### 3. Configure Each Service in Railway

For each service (Grafana, Prometheus, Loki, Tempo):

**Settings → Source:**
- Repository: `Elementary-Digital/railway-grafana-stack`
- Root Directory: `grafana` (or `prometheus`, `loki`, `tempo`)
- Branch: `main`

**Settings → Variables (Optional):**
- `VERSION=v3.2.1` (Prometheus)
- `VERSION=11.5.2` (Grafana)
- `VERSION=3.4` (Loki)
- `VERSION=latest` (Tempo)

### 4. Verify Deployment

**Check Prometheus:**
- In Grafana → Explore → Prometheus
- Query: `up{job="routeroyal-api"}`
- Should return `1` if scraping works

**Check Loki:**
- In Grafana → Explore → Loki
- Query: `{job="routeroyal-api"}`
- Should show logs if configured

**Check Grafana:**
- Access Grafana URL from Railway
- Login with admin credentials
- Check Data Sources (should show Prometheus, Loki, Tempo)

## Why These Fixes Work

1. **Capital D Dockerfile**: Railway's Docker detection prefers `Dockerfile`
2. **railway.json**: Explicitly tells Railway to use Docker builder for each service
3. **Root Directory**: Points Railway to the correct service directory
4. **Production Config**: Prometheus now scrapes your actual API

## Troubleshooting

### If Railway Still Uses Railpack:
1. Verify Root Directory is set correctly for each service
2. Check that `Dockerfile` (capital D) exists in each service directory
3. Ensure `railway.json` exists in each service directory
4. Try disconnecting and reconnecting each service repo

### If Services Can't Communicate:
1. Check Railway internal networking is enabled
2. Verify service names match internal URLs
3. Check Grafana datasources use internal URLs (not localhost)

### If Prometheus Isn't Scraping:
1. Verify `prom.yml` has correct scrape targets
2. Check API `/metrics` endpoint is accessible
3. Check Prometheus logs for scrape errors

## Summary

✅ All services now have proper Dockerfile detection  
✅ All services have railway.json configurations  
✅ Prometheus configured for production scraping  
✅ Loki typo fixed  
✅ docker-compose.yml updated for consistency  

The template is now ready to deploy all 4 services correctly on Railway!

