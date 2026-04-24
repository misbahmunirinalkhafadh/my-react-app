# Environment Variables Setup

Project ini menggunakan Vite, yang membutuhkan environment variables yang diawali dengan `VITE_` untuk expose ke client-side.

## 📁 Files

- **`.env`** - Local development variables (tidak di-commit ke Git)
- **`.env.example`** - Template/reference untuk setup (di-commit ke Git)
- **`.env.staging`** - (Optional) Untuk staging deployment
- **`.env.production`** - (Optional) Untuk production deployment

## 🔧 Setup Local Development

### 1. Copy template
```bash
cp .env.example .env
```

### 2. Edit `.env` sesuai local setup Anda
```env
VITE_APP_NAME=my-react-app
VITE_API_URL=http://localhost:3000/api
VITE_ENV=development
```

### 3. Start dev server
```bash
npm run dev
```

Variabel dari `.env` akan otomatis di-load oleh Vite.

## 📋 Available Variables

### Application Settings
| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `VITE_APP_NAME` | my-react-app | Nama aplikasi |
| `VITE_APP_VERSION` | 0.0.0 | Version aplikasi (dari package.json) |
| `VITE_ENV` | development | Environment: development, staging, production |

### API Configuration
| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `VITE_API_URL` | http://localhost:3000/api | Base URL untuk API calls |
| `VITE_API_TIMEOUT` | 30000 | Request timeout dalam milliseconds |

### Feature Flags
| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `VITE_ENABLE_ANALYTICS` | false | Enable Google Analytics / tracking |
| `VITE_ENABLE_SENTRY` | false | Enable Sentry error tracking |

## 🔐 Accessing Variables di Code

### React Component
```jsx
import { useEffect } from 'react'

function App() {
  useEffect(() => {
    console.log('API URL:', import.meta.env.VITE_API_URL)
    console.log('App Name:', import.meta.env.VITE_APP_NAME)
    console.log('Environment:', import.meta.env.VITE_ENV)
  }, [])

  return <div>My React App</div>
}
```

### Service/Utility
```js
// services/api.js
const API_URL = import.meta.env.VITE_API_URL
const API_TIMEOUT = import.meta.env.VITE_API_TIMEOUT

export const fetchData = async () => {
  const response = await fetch(`${API_URL}/data`, {
    timeout: API_TIMEOUT
  })
  return response.json()
}
```

### Conditional Logic
```js
if (import.meta.env.VITE_ENV === 'production') {
  // Production-only code
}

if (import.meta.env.VITE_ENABLE_ANALYTICS) {
  // Initialize analytics
}
```

## 🚀 Deployment Environments

### Staging Deployment
Create `.env.staging`:
```env
VITE_APP_NAME=my-react-app-staging
VITE_API_URL=https://staging-api.example.com/api
VITE_ENV=staging
VITE_ENABLE_ANALYTICS=true
```

Build untuk staging:
```bash
# Tidak bisa langsung pake .env.staging di npm run build
# Harus di-pass via command line atau set variabel
VITE_ENV=staging npm run build
```

Atau lebih baik, setup di CI/CD pipeline:
```yaml
- name: Build for Staging
  env:
    VITE_API_URL: ${{ secrets.STAGING_API_URL }}
    VITE_ENV: staging
  run: npm run build
```

### Production Deployment
```yaml
- name: Build for Production
  env:
    VITE_API_URL: ${{ secrets.PROD_API_URL }}
    VITE_ENV: production
    VITE_ENABLE_ANALYTICS: true
    VITE_ENABLE_SENTRY: true
  run: npm run build
```

## 📝 GitHub Secrets (untuk CI/CD)

Untuk build otomatis di CI/CD dengan credentials:

Settings → Secrets and variables → Actions:

```
Name: STAGING_API_URL
Value: https://staging-api.example.com/api

Name: PROD_API_URL
Value: https://api.example.com/api

Name: SENTRY_DSN
Value: https://xxxxx@sentry.io/xxxxx
```

Kemudian gunakan di workflow:
```yaml
env:
  VITE_API_URL: ${{ secrets.STAGING_API_URL }}
```

## ⚠️ Important Notes

1. **Jangan commit `.env`** - Sudah ada di `.gitignore`
2. **Commit `.env.example`** - Untuk reference team
3. **Variabel VITE_** - Hanya ini yang di-expose ke client
4. **Sensitive data** - Gunakan GitHub Secrets untuk deployment
5. **Build-time variables** - Env vars di-embed saat build, bukan runtime

## 🔍 Debug

### Check variabel yang ter-load
```js
console.log(import.meta.env)
```

### Check di DevTools
1. F12 → Console
2. Ketik: `import.meta.env`

### Common Issues

**"undefined" value di console:**
- ✓ Pastikan variabel di `.env` ada
- ✓ Pastikan nama dimulai dengan `VITE_`
- ✓ Restart dev server setelah edit `.env`

**Variabel tidak ter-update:**
- ✓ Stop dev server: `Ctrl + C`
- ✓ Start lagi: `npm run dev`

---

Sudah siap! Edit `.env` sesuai local setup, atau beri tahu jika butuh custom variables.
