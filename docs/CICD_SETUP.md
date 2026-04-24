# CI/CD Setup Guide

Setup CI/CD pipeline Anda untuk React Vite app ini sudah siap! File workflow sudah di-create di `.github/workflows/ci.yml`.

## 📋 Overview

Pipeline ini meliputi:
- ✅ **Build**: Compile app dengan Vite → output ke `dist/`
- ✅ **Lint**: Check code quality dengan ESLint
- ✅ **Security**: Scan vulnerabilities dengan npm audit & Trivy
- 📦 **Staging Deploy** (template): Untuk deploy ke server staging
- 🚀 **Production Deploy** (template): Untuk deploy ke production
- 📊 **Summary**: Ringkasan hasil pipeline

## 🚀 Quick Start

### 1. Push ke GitHub
```bash
git add .github/workflows/ci.yml
git commit -m "Setup: Add CI/CD pipeline"
git push origin main
```

### 2. Lihat workflow berjalan
- Buka GitHub repository Anda
- Tab "Actions" 
- Akan melihat job "CI/CD Pipeline" berjalan

### 3. Cek hasil
Workflow akan:
1. Setup dependencies
2. Build project dengan `npm run build`
3. Check linting dengan `npm run lint`
4. Run security scan

## 🔐 Konfigurasi Deployment (Optional)

Jika ingin aktifkan auto-deploy ke staging/production, ikuti langkah berikut:

### A. Setup GitHub Secrets

1. Buka GitHub repository settings → **Secrets and variables** → **Actions**
2. Tambah secrets berikut:

**Untuk Staging:**
```
Name: STAGING_HOST
Value: staging.example.com

Name: STAGING_USER  
Value: ubuntu

Name: STAGING_KEY
Value: [paste private SSH key dari file pem]
```

**Untuk Production:**
```
Name: PROD_HOST
Value: example.com

Name: PROD_USER
Value: ubuntu

Name: PROD_KEY
Value: [paste private SSH key dari file pem]
```

> **⚠️ Penting**: Private key harus dalam format yang benar dengan newline:
> ```
> -----BEGIN OPENSSH PRIVATE KEY-----
> [content]
> -----END OPENSSH PRIVATE KEY-----
> ```

### B. Uncomment Deployment Commands

Edit `.github/workflows/ci.yml` di section deployment:

**Untuk Staging (line ~173):**
```yaml
- name: Deploy to Staging
  run: |
    scp -i /tmp/staging_key -r dist/* ${{ secrets.STAGING_USER }}@${{ secrets.STAGING_HOST }}:/var/www/app/
    # ssh -i /tmp/staging_key ${{ secrets.STAGING_USER }}@${{ secrets.STAGING_HOST }} 'cd /var/www && systemctl restart myapp'
```

**Untuk Production (line ~220):**
```yaml
- name: Deploy to Production
  run: |
    scp -i /tmp/prod_key -r dist/* ${{ secrets.PROD_USER }}@${{ secrets.PROD_HOST }}:/var/www/app/
    # ssh -i /tmp/prod_key ${{ secrets.PROD_USER }}@${{ secrets.PROD_HOST }} 'cd /var/www && systemctl restart myapp'
```

### C. Setup GitHub Environments

1. Settings → **Environments**
2. Create environment "staging":
   - Deployment branches: `develop`
3. Create environment "production":
   - Deployment branches: `main`
   - Add required reviewers (optional, untuk approval)

## 📊 Branch Strategy

Recommended flow:

```
feature branch → develop → main
     ↓              ↓         ↓
  (testing)    (staging)  (production)
```

- **Push to `develop`**: Auto deploy to staging
- **Push to `main` with workflow_dispatch**: Manual deploy to production

## 🎯 Workflow Behavior

### Saat push ke `develop`:
```
setup → build + quality + security → deploy-staging → summary
```

### Saat push ke `main`:
```
setup → build + quality + security → summary
(no auto deploy - manual trigger needed)
```

### Saat manual trigger (workflow_dispatch):
```
setup → build + quality + security → deploy-production → summary
```

## 🔍 Monitoring & Troubleshooting

### Lihat logs
1. GitHub → Actions tab
2. Click workflow run
3. Click job untuk lihat detail logs

### Common issues:

**Build gagal:**
- Check `npm run build` berjalan locally: `npm run build`
- Cek browser console untuk errors

**Linting error:**
- Run: `npm run lint`
- Fix errors yang ditunjukkan

**Deploy gagal:**
- Verify SSH credentials di secrets
- Test SSH manually: `ssh -i key user@host`
- Check disk space di server

## 📝 Customization

### Add test step
Jika nanti ada test suite:

```yaml
- name: Run tests
  run: npm test -- --coverage --watchAll=false
```

### Add Slack notifications
Uncomment section notification, add `SLACK_WEBHOOK` secret

### Change Node version
Edit di `env`:
```yaml
NODE_VERSION: '20'  # Change from '18'
```

### Add caching for faster builds
Sudah included dengan `cache: 'npm'`

## ✅ Verification Checklist

- [x] File `.github/workflows/ci.yml` exist
- [ ] Pushed to GitHub
- [ ] Workflow runs successfully (check Actions tab)
- [ ] Build output di `dist/` folder
- [ ] ESLint passing atau fixed
- [ ] (Optional) Secrets configured untuk deployment
- [ ] (Optional) Environments setup

## 📚 Resources

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Vite Deployment Guide](https://vitejs.dev/guide/static-deploy.html)
- [ESLint Docs](https://eslint.org/docs/latest/)

## 🆘 Next Steps

1. **Immediately**: Push workflow ke GitHub dan verify berjalan
2. **Soon**: Setup linting rules yang sesuai project
3. **Later**: Configure deployment secrets untuk staging/production

---

Questions? Check GitHub Actions logs untuk debugging.
