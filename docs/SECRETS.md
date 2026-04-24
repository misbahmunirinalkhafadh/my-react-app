# Setup & Manage GitHub Secrets

Secrets digunakan untuk menyimpan informasi sensitif seperti passwords, API keys, dan tokens.

## How to Add a Secret

### 1. Repository Settings

1. Buka GitHub repository
2. Settings → Secrets and variables → Actions
3. Click "New repository secret"

### 2. Create Secret

- **Name**: `DOCKER_PASSWORD` (gunakan UPPERCASE)
- **Value**: Masukkan nilai secret
- Click "Add secret"

### 3. Use in Workflow

```yaml
- name: Login to Docker
  run: docker login -u myuser -p ${{ secrets.DOCKER_PASSWORD }}
```

## Types of Secrets

### 1. Repository Secrets

Hanya tersedia untuk satu repository.

**How to access:**
```
Settings → Secrets and variables → Actions → Repository secrets
```

### 2. Organization Secrets

Tersedia untuk semua repositories dalam organization.

**How to access:**
```
Organization Settings → Secrets and variables → Actions
```

Dengan access control per repository.

### 3. Environment Secrets

Tersedia untuk specific deployment environment.

**How to access:**
```
Settings → Environments → Environment name → Secrets
```

```yaml
jobs:
  deploy:
    environment: production
    steps:
      - run: echo ${{ secrets.DEPLOY_TOKEN }}
```

## Common Secrets & Setup

### Docker Hub

```yaml
DOCKER_USERNAME: your_docker_username
DOCKER_PASSWORD: your_docker_password_or_token
```

**Setup:**
1. Docker Hub → Account Settings → Security → Access Tokens
2. Create token (read & write)
3. Copy token value

**Usage:**
```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### GitHub Container Registry

```yaml
GHCR_USERNAME: ${{ github.actor }}
GHCR_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

**Usage:**
```yaml
- name: Login to GHCR
  uses: docker/login-action@v2
  with:
    registry: ghcr.io
    username: ${{ secrets.GHCR_USERNAME }}
    password: ${{ secrets.GHCR_PASSWORD }}
```

### SSH Deployment

```yaml
SSH_PRIVATE_KEY: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  MIIEpAIBAAKCAQEA...
  -----END OPENSSH PRIVATE KEY-----
SSH_HOST: example.com
SSH_USER: deploy
```

**Setup Private Key:**
1. Generate SSH key: `ssh-keygen -t ed25519 -f deploy_key`
2. Copy content of `deploy_key`
3. Add as secret
4. Copy `deploy_key.pub` ke `~/.ssh/authorized_keys` di server

**Usage:**
```yaml
- name: Deploy via SSH
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SSH_HOST }}
    username: ${{ secrets.SSH_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /var/www/app
      git pull origin main
      npm install
      npm run build
```

### Database Connection

```yaml
DATABASE_URL: postgresql://user:password@host:5432/dbname
```

**Security Tips:**
- Gunakan read-only credentials jika possible
- Rotate credentials secara berkala
- Use separate credentials per environment

### API Keys & Tokens

```yaml
SLACK_WEBHOOK: https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX
SONAR_TOKEN: squ_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CODECOV_TOKEN: xxxxxxxxxxxxxxxxxxxxxxxxxxxx
OPENAI_API_KEY: sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxx
DEPLOY_TOKEN: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Security Best Practices

### 1. Minimal Permissions

```yaml
# ❌ DON'T: Use personal token dengan full access
GITHUB_TOKEN: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ✅ DO: Use specific scoped token
# Buat token hanya untuk yang dibutuhkan (repo, packages, etc)
```

### 2. Rotate Secrets Regularly

- Setup reminders untuk rotate quarterly
- Update semua places yang menggunakan secret
- Revoke old credentials

### 3. Never Log Secrets

GitHub automatically masks secrets di logs, tapi:

```yaml
# ❌ DON'T
- run: echo "Password is ${{ secrets.PASSWORD }}"

# ✅ DO
- run: mycommand --password=${{ secrets.PASSWORD }}
```

### 4. Use Environment Secrets

```yaml
jobs:
  deploy:
    environment:
      name: production
      # Require approval sebelum deploy
      deployment-branch-policy:
        protected-branches: true
    steps:
      - run: deploy --token=${{ secrets.DEPLOY_TOKEN }}
```

### 5. Limit Secret Usage

```yaml
# Use in specific jobs saja
deploy:
  runs-on: ubuntu-latest
  environment: production
  steps:
    - run: deploy --token=${{ secrets.DEPLOY_TOKEN }}
```

## Access Control

### Organization Secrets Access

```yaml
# Accessible to: selected repositories
name: CI
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Use org secret
        run: echo ${{ secrets.ORG_SECRET }}
```

### Required Reviewers

Untuk sensitive deployments:

1. Settings → Environments → Production
2. Deployment branches → Require branches to be up to date
3. Required reviewers → Add reviewers

## Debugging Secrets

### View Secret Names (Not Values)

```bash
# In your workflow, you can see which secrets are available
- name: Check available secrets
  run: env | grep -E "^(DOCKER|SSH|DATABASE)" | cut -d= -f1
  # Shows: DOCKER_USERNAME, SSH_HOST, DATABASE_URL
  # But NOT the values!
```

### Troubleshooting

**Secret not found:**
```yaml
# ❌ WRONG: Case sensitive!
run: echo ${{ secrets.docker_password }}

# ✅ CORRECT
run: echo ${{ secrets.DOCKER_PASSWORD }}
```

**Secret in pull_request event:**
```yaml
# Secrets NOT available di pull_request dari fork
# Use: if: github.event_name == 'push'
```

## Masking Output

Manually mask output:

```yaml
- run: |
    SECRET_VALUE="my-secret-value"
    echo "::add-mask::${SECRET_VALUE}"
    echo "The secret is: ${SECRET_VALUE}"  # Output: ***
```

## Viewing Secret Metadata

```bash
# List all secrets (without values)
curl -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
  https://api.github.com/repos/${{ github.repository }}/actions/secrets
```

## Template: Complete Setup

```yaml
name: Build & Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build & Push Image
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          script: |
            docker login -u ${{ secrets.REGISTRY_USER }} \
              -p ${{ secrets.REGISTRY_PASSWORD }} ghcr.io
            docker pull ghcr.io/${{ github.repository }}:latest
            docker-compose up -d
```
