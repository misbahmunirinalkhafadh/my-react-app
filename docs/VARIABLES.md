# Daftar Variabel & Secrets

## GitHub Context Variables

Variabel yang tersedia secara otomatis dalam setiap workflow run:

| Variabel | Contoh Nilai | Keterangan |
|----------|-------------|-----------|
| `github.actor` | `john-doe` | Username yang trigger workflow |
| `github.event_name` | `push`, `pull_request`, `workflow_dispatch` | Jenis event yang trigger workflow |
| `github.ref` | `refs/heads/main` | Branch/tag reference lengkap |
| `github.ref_name` | `main` | Nama branch/tag tanpa prefix |
| `github.sha` | `abcdef1234567890` | Commit SHA lengkap |
| `github.repository` | `owner/repo-name` | Repository dalam format owner/name |
| `github.repository_owner` | `owner` | Owner repository |
| `github.run_number` | `123` | Nomor workflow run |
| `github.run_id` | `456789` | ID unik workflow run |
| `github.server_url` | `https://github.com` | URL GitHub server |
| `github.api_url` | `https://api.github.com` | URL GitHub API |
| `github.workspace` | `/home/runner/work/repo-name` | Working directory |
| `runner.os` | `Linux`, `Windows`, `macOS` | OS runner |
| `runner.temp` | `/home/runner/work/_temp` | Temporary directory |

## Custom Environment Variables

Definisikan custom variables di section `env`:

```yaml
env:
  NODE_ENV: production
  BUILD_TIMEOUT: 600
  REGISTRY: ghcr.io
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo ${{ env.NODE_ENV }}  # Output: production
```

## Secrets

Secrets digunakan untuk informasi sensitif (passwords, API keys, tokens).

### Setup Secrets

1. Repository Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Nama: `DOCKER_PASSWORD` (uppercase)
4. Value: `your_secret_value`

### Menggunakan Secrets

```yaml
- name: Login to Docker
  run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
```

### Secrets Umum

| Secret | Contoh | Kegunaan |
|--------|--------|----------|
| `GITHUB_TOKEN` | Built-in | GitHub API access |
| `DOCKER_USERNAME` | `username` | Docker Hub login |
| `DOCKER_PASSWORD` | `token` | Docker Hub password |
| `DOCKER_REGISTRY_URL` | `ghcr.io` | Container registry |
| `SSH_KEY` | `-----BEGIN PRIVATE KEY-----` | SSH deployment |
| `SSH_HOST` | `example.com` | Server hostname |
| `SSH_USER` | `deploy` | SSH username |
| `DATABASE_URL` | `postgresql://...` | Database connection |
| `API_TOKEN` | `sk_live_...` | API keys |
| `SLACK_WEBHOOK` | `https://hooks.slack.com/...` | Slack notifications |
| `SONAR_TOKEN` | `sonar_token_...` | SonarCloud token |
| `CODECOV_TOKEN` | `codecov_token_...` | Codecov token |
| `DEPLOY_KEY` | `-----BEGIN OPENSSH PRIVATE KEY-----` | Deployment key |

## Organization/Repository Level Variables

Variables yang dapat diakses dari seluruh workflow:

```yaml
# Mengakses repository variables
- name: Use variable
  run: echo ${{ vars.MY_VARIABLE }}

# Mengakses secrets
- name: Use secret
  run: echo ${{ secrets.MY_SECRET }}
```

## Output Variables (Step Outputs)

```yaml
- name: Get version
  id: version
  run: echo "version=1.0.0" >> $GITHUB_OUTPUT

- name: Use output
  run: echo ${{ steps.version.outputs.version }}
```

## Job Outputs

```yaml
jobs:
  build:
    outputs:
      build-version: ${{ steps.version.outputs.version }}
    steps:
      - name: Get version
        id: version
        run: echo "version=1.0.0" >> $GITHUB_OUTPUT
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Use build version
        run: echo ${{ needs.build.outputs.build-version }}
```

## Matrix Variables

Gunakan dalam strategy matrix:

```yaml
strategy:
  matrix:
    node-version: [14, 16, 18]
    os: [ubuntu-latest, windows-latest]

steps:
  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node-version }}
      
  - run: echo "Running on ${{ matrix.os }}"
```

## Expressions & Functions

```yaml
# String interpolation
${{ format('Hello {0}', github.actor) }}

# Conditionals
if: ${{ contains(github.ref, 'main') && success() }}

# Functions
${{ hashFiles('package-lock.json') }}
${{ startsWith(github.ref, 'refs/tags') }}
${{ endsWith(github.ref, '/develop') }}

# Matrix combinations
${{ matrix.node-version }}-${{ matrix.os }}
```

## Special Considerations

### Token Permissions

`GITHUB_TOKEN` memiliki izin terbatas. Untuk permission tambahan:

```yaml
permissions:
  contents: read
  packages: write
  pull-requests: write
  issues: write
```

### Masking Secrets

Secrets otomatis di-mask di logs. Untuk manual masking:

```yaml
- run: |
  echo "::add-mask::mysecretvalue"
  echo "mysecretvalue"  # Will show as ***
```

### Available in

- All jobs
- All steps dalam job
- Actions yang digunakan
- Environment variables

## Best Practices

1. **Gunakan Secrets untuk data sensitif**
   ```yaml
   # ❌ DON'T
   password: "my-secret-password"
   
   # ✅ DO
   password: ${{ secrets.PASSWORD }}
   ```

2. **Use descriptive names**
   ```yaml
   # ✅ GOOD
   - env: BUILD_TIMEOUT_MINUTES: 60
   
   # ❌ BAD
   - env: TIMEOUT: 60
   ```

3. **Define at appropriate scope**
   ```yaml
   # Global env (available to all jobs)
   env:
     REGISTRY: ghcr.io
   
   jobs:
     build:
       # Job-level env
       env:
         BUILD_ENV: development
   ```
