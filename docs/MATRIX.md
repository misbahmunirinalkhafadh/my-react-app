# Matrix Strategy untuk Multi-Version Testing

Matrix strategy memungkinkan menjalankan job dengan kombinasi variable yang berbeda.

## Basic Matrix

Menjalankan job dengan multiple versions:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [14, 16, 18, 20]
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

Ini akan membuat **4 job** (satu untuk setiap node version).

## Multi-Dimension Matrix

Kombinasi multiple variables:

```yaml
strategy:
  matrix:
    node-version: [14, 16, 18]
    os: [ubuntu-latest, windows-latest, macos-latest]
    include:
      - node-version: 20
        os: ubuntu-latest
    exclude:
      - node-version: 14
        os: windows-latest
```

Ini akan create:
- 14 x ubuntu ✓
- 14 x windows ✗ (excluded)
- 14 x macos ✓
- 16 x ubuntu ✓
- 16 x windows ✓
- 16 x macos ✓
- 18 x ubuntu ✓
- 18 x windows ✓
- 18 x macos ✓
- 20 x ubuntu ✓ (included)

Total: **10 job**

## Include & Exclude

### Include: Tambah kombinasi khusus

```yaml
strategy:
  matrix:
    node-version: [16, 18]
    include:
      - node-version: 20
        experimental: true
      - node-version: 20
        experimental: false

steps:
  - name: Run tests
    run: npm test
    if: ${{ matrix.experimental != true }}
  
  - name: Run experimental tests
    run: npm test --experimental
    if: ${{ matrix.experimental == true }}
```

### Exclude: Hilangkan kombinasi tertentu

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [14, 16, 18, 20]
    exclude:
      - os: windows-latest
        node-version: 14
      - os: macos-latest
        node-version: 14
```

## Advanced Matrix Patterns

### Conditional Matrix

```yaml
strategy:
  matrix:
    include:
      - node-version: 16
        os: ubuntu-latest
        test-suite: unit
      - node-version: 16
        os: ubuntu-latest
        test-suite: integration
      - node-version: 18
        os: ubuntu-latest
        test-suite: unit

steps:
  - name: Run tests
    run: npm run test:${{ matrix.test-suite }}
```

### Using Matrix in Job Name

```yaml
name: Test Node ${{ matrix.node-version }} on ${{ matrix.os }}
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    node-version: [16, 18]
    os: [ubuntu-latest, windows-latest]
```

Output:
- `Test Node 16 on ubuntu-latest`
- `Test Node 16 on windows-latest`
- `Test Node 18 on ubuntu-latest`
- `Test Node 18 on windows-latest`

## Job Outputs with Matrix

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18]
    outputs:
      versions: ${{ steps.versions.outputs.result }}
    
    steps:
      - id: versions
        run: echo "result=[${{ matrix.node-version }}]" >> $GITHUB_OUTPUT

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo ${{ needs.build.outputs.versions }}
```

## Fail Fast & Continue on Error

### Fail Fast (Default)

```yaml
strategy:
  fail-fast: true  # Stop semua job jika ada yang gagal
  matrix:
    node-version: [14, 16, 18]
```

### Continue on Error

```yaml
strategy:
  fail-fast: false  # Lanjutkan semua job meskipun ada yang gagal
  matrix:
    node-version: [14, 16, 18]
```

## Max Parallel Jobs

Batasi jumlah job yang berjalan bersamaan:

```yaml
strategy:
  max-parallel: 2
  matrix:
    node-version: [14, 16, 18, 20, 21]
```

Hanya 2 job yang berjalan pada saat yang sama (useful untuk resource intensive tasks).

## Matrix Context

Akses matrix values dalam workflow:

```yaml
strategy:
  matrix:
    node-version: [16, 18]
    os: [ubuntu-latest, windows-latest]

steps:
  - run: |
      echo "Node: ${{ matrix.node-version }}"
      echo "OS: ${{ matrix.os }}"
```

## Practical Examples

### 1. Node.js Multi-Version Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x, 20.x]
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

### 2. Python Multi-Version Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10', '3.11']
        include:
          - python-version: '3.11'
            experimental: true
    
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -r requirements.txt
      - run: pytest
        continue-on-error: ${{ matrix.experimental == true }}
```

### 3. Java Multi-Version Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        java-version: ['11', '17', '21']
        maven-version: ['3.8.1', '3.9.0']
      exclude:
        - java-version: '11'
          maven-version: '3.9.0'
    
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
      - run: mvn clean test
```

### 4. .NET Multi-Version Testing

```yaml
jobs:
  test:
    strategy:
      matrix:
        dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
        os: [ubuntu-latest, windows-latest]
      exclude:
        - os: macos-latest
          dotnet-version: '6.0.x'
    
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v3
        with:
          dotnet-version: ${{ matrix.dotnet-version }}
      - run: dotnet test
```

## Best Practices

1. **Jangan terlalu banyak kombinasi**
   ```yaml
   # ❌ Menghasilkan 36 jobs (6x6)
   strategy:
     matrix:
       os: [ubuntu, windows, macos]
       version: [14, 16, 18, 20, 21, 22]
   
   # ✅ Lebih efisien dengan exclude
   exclude:
     - os: windows
       version: 14
   ```

2. **Gunakan include untuk special cases**
   ```yaml
   strategy:
     matrix:
       version: [16, 18]
       include:
         - version: 20
           experimental: true
   ```

3. **Timeout per matrix job**
   ```yaml
   timeout-minutes: 30
   strategy:
     matrix:
       node-version: [14, 16, 18]
   ```

4. **Upload artifacts per matrix**
   ```yaml
   - uses: actions/upload-artifact@v3
     with:
       name: coverage-node-${{ matrix.node-version }}
       path: coverage/
   ```
