# Dashboard

Central dashboard for project reports — coverage (ReportGenerator) and mutation (Stryker).

## Adding a New Project

### 1. Update `index.html`

Add an entry to the `projects` array:

```js
const projects = [
  { id: 'scrum-poker',     label: 'Scrum Poker',     url: 'https://github.com/spicycoder/scrum-poker' },
  { id: 'resume-analyzer', label: 'Resume Analyzer', url: 'https://github.com/spicycoder/resume-analyzer' },
  { id: 'my-project',      label: 'My Project',      url: 'https://github.com/spicycoder/my-project' },
];
```

- `id` — folder name under which reports will be stored
- `label` — display name in the sidebar
- `url` — optional, adds an external link icon

### 2. Add CI Steps to Your Project

A GitHub PAT with `contents:write` on this repo must be set as `PAT` secret in your project's repository.

#### Coverage (on merge)

```yaml
- uses: danielpalme/ReportGenerator-GitHub-Action@5
  with:
    reports: .coverage/coverage.cobertura.xml
    targetdir: .coverage
    reporttypes: Html

- run: |
    git clone --depth 1 https://x-access-token:${{ secrets.PAT }}@github.com/spicycoder/dashboard.git
    rm -rf dashboard/my-project/coverage/
    mkdir -p dashboard/my-project/coverage/
    cp -r .coverage/* dashboard/my-project/coverage/
    cd dashboard
    git config user.name "ci-bot"
    git config user.email "ci-bot@users.noreply.github.com"
    git add my-project/coverage/
    git commit -m "my-project: update coverage"
    git push
```

#### Mutation (nightly)

Same pattern, just replace `coverage` with `stryker`:

```yaml
- run: |
    git clone --depth 1 https://x-access-token:${{ secrets.PAT }}@github.com/spicycoder/dashboard.git
    rm -rf dashboard/my-project/stryker/
    mkdir -p dashboard/my-project/stryker/
    cp -r StrykerOutput/* dashboard/my-project/stryker/
    cd dashboard
    git config user.name "ci-bot"
    git config user.email "ci-bot@users.noreply.github.com"
    git add my-project/stryker/
    git commit -m "my-project: update stryker"
    git push
```

### Stub Placeholder Files

Each `project/report/` directory needs a stub `index.html` that shows a friendly message until the first CI run overwrites it. Create one per report type:

**Coverage stub** (`my-project/coverage/index.html`):
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="h-screen flex flex-col items-center justify-center bg-gray-900 text-gray-400">
  <p class="text-lg font-medium">Report not yet available</p>
  <p class="text-sm mt-1">Coverage reports are generated on each merge.</p>
</body>
</html>
```

**Mutation stub** (`my-project/stryker/index.html`):
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="h-screen flex flex-col items-center justify-center bg-gray-900 text-gray-400">
  <p class="text-lg font-medium">Report not yet available</p>
  <p class="text-sm mt-1">Mutation reports are generated nightly.</p>
</body>
</html>
```

### Notes

- Coverage reports are generated on merge. Mutation reports are generated nightly (separate scheduled workflow).
- Real report files overwrite the stubs automatically — no manual cleanup needed.
