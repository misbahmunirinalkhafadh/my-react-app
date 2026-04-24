# Workflow Triggers

Berbagai cara untuk memicu workflow execution.

## 1. Push Event

Trigger ketika ada push ke repository.

```yaml
on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'
      - 'package.json'
    tags:
      - 'v*'
    tags-ignore:
      - 'v0.*'
```

**Konfigurasi:**
- `branches`: Branch mana yang trigger workflow
- `paths`: File/folder tertentu yang trigger workflow
- `tags`: Tag patterns yang trigger workflow

## 2. Pull Request Event

Trigger saat PR dibuat atau update.

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened
    branches:
      - main
    paths:
      - 'src/**'
```

**Types:**
- `opened`: PR baru dibuat
- `synchronize`: Commit baru push ke PR
- `reopened`: PR di-reopen
- `ready_for_review`: PR di-mark ready

## 3. Scheduled Workflow

Trigger berdasarkan schedule (cron).

```yaml
on:
  schedule:
    - cron: '0 2 * * *'          # Setiap hari jam 2 pagi UTC
    - cron: '0 10 * * 1'         # Setiap hari Senin jam 10 pagi
    - cron: '*/15 * * * *'       # Setiap 15 menit
```

**Cron Format:** `minute hour day month day-of-week`

Contoh:
- `0 0 * * *` - Tengah malam setiap hari
- `0 */6 * * *` - Setiap 6 jam
- `0 0 * * 0` - Tengah malam Minggu
- `30 2 1 * *` - Jam 2:30 setiap awal bulan

## 4. Manual Trigger (workflow_dispatch)

Trigger workflow secara manual dari GitHub UI.

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - development
          - staging
          - production
      version:
        description: 'Version to deploy'
        required: false
        type: string
      debug:
        description: 'Enable debug mode'
        required: false
        type: boolean
```

**Input Types:**
- `string`: Text input
- `choice`: Dropdown selection
- `boolean`: Checkbox
- `environment`: Environment selector

**Akses input:**
```yaml
steps:
  - run: echo ${{ inputs.environment }}
```

## 5. Release Event

Trigger saat release dibuat.

```yaml
on:
  release:
    types:
      - published
      - created
      - edited
      - deleted
      - prereleased
      - released
```

## 6. Webhook Events

### Issues

```yaml
on:
  issues:
    types:
      - opened
      - closed
      - reopened
      - edited
```

### Issue Comments

```yaml
on:
  issue_comment:
    types:
      - created
      - edited
```

### Discussions

```yaml
on:
  discussions:
    types:
      - created
      - edited
```

## 7. Repository Events

```yaml
on:
  # Repository ditonton
  watch:
    types:
      - started
  
  # Workflow repository trigger workflow lain
  workflow_run:
    workflows:
      - 'Tests'
    types:
      - completed
    branches:
      - main
```

## 8. Page Build

Trigger saat GitHub Pages build.

```yaml
on:
  page_build
```

## Combining Triggers

Trigger workflow saat salah satu event terjadi:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'
  workflow_dispatch:
```

## Filtering Events

```yaml
on:
  push:
    # Branch filters
    branches:
      - main
      - 'feature/**'           # Wildcard
      - '!bugfix/*'            # Exclude
    
    # Path filters
    paths:
      - 'src/**'
      - '*.md'
      - '!docs/**'             # Exclude
    
    # Tag filters
    tags:
      - 'v*'
      - '!v0.*'
```

## Skip Workflows

### Dalam commit message

Tambahkan `[skip ci]` atau `[ci skip]`:

```bash
git commit -m "Update README [skip ci]"
```

### Menggunakan path filters

```yaml
on:
  push:
    paths:
      - 'src/**'
      - '.github/workflows/**'
    paths-ignore:
      - 'docs/**'
      - '*.md'
```

## Trigger dari Workflow Lain

```yaml
on:
  workflow_run:
    workflows: ['CI']
    types:
      - completed
    branches: [main]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: echo "Deploying after successful CI"
```

## Best Practices

1. **Gunakan branch filters**
   ```yaml
   on:
     push:
       branches: [main, develop]
   ```

2. **Gunakan path filters untuk efficiency**
   ```yaml
   on:
     push:
       paths:
         - 'src/**'
         - '.github/workflows/**'
   ```

3. **Hindari terlalu banyak triggers manual**
   ```yaml
   # Gunakan untuk deployment/hotfix saja
   workflow_dispatch:
     inputs:
       environment:
         description: 'Deploy environment'
   ```

4. **Dokumentasikan trigger behavior**
   ```yaml
   # Comment di awal workflow
   # Triggers:
   # - Push ke main/develop branch
   # - Manual trigger untuk deployment
   ```
