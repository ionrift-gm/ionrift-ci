# ionrift-ci

Shared reusable GitHub Actions workflows for the `ionrift-gm` organisation.

All Ionrift module CI pipelines call these workflows instead of duplicating the implementations. Changes here propagate to every consumer on the next push.

---

## Workflows

| File | Purpose | Used by |
|------|---------|---------|
| `lint.yml` | ESLint, inclusion scan, import integrity, case-sensitivity, signal check | All source modules |
| `hygiene.yml` | Dev doc / tooling / LevelDB sweep | All modules |
| `vitest.yml` | Headless Vitest unit tests | library, respite, arbiter, workshop |
| `module-manifest.yml` | Validates `module.json` fields | ionrift-library |

---

## Calling a Reusable Workflow

```yaml
jobs:
  lint:
    uses: ionrift-gm/ionrift-ci/.github/workflows/lint.yml@main
    with:
      eslint_max_warnings: 200

  hygiene:
    uses: ionrift-gm/ionrift-ci/.github/workflows/hygiene.yml@main

  vitest:
    uses: ionrift-gm/ionrift-ci/.github/workflows/vitest.yml@main
```

---

## lint.yml Inputs Reference

| Input | Type | Default | Notes |
|-------|------|---------|-------|
| `skip_eslint` | boolean | `false` | Set `true` for compiled-script modules (Resonance) |
| `eslint_version` | string | `'8'` | `'8'` or `'9'` |
| `eslint_pattern` | string | `'scripts/'` | Quote globs: `'"scripts/**/*.mjs"'` |
| `eslint_ext_args` | string | `'--ext .js'` | Empty when using a glob pattern |
| `eslint_max_warnings` | number | `50` | Ratchet down as warnings are resolved |
| `eslint_extra_args` | string | `''` | Any extra ESLint flags |
| `script_ext` | string | `'.js'` | `.js` or `.mjs`. Drives inclusion + import checks |
| `include_html` | boolean | `true` | Scan `*.html` in inclusion check |
| `include_styles` | boolean | `false` | Scan `*.css` (set `true` for ionrift-library) |
| `inclusion_scan_dirs` | string | `'scripts/ lang/'` | Space-separated dirs |
| `skip_import_integrity` | boolean | `false` | Set `true` for compiled modules |
| `skip_case_sensitivity` | boolean | `false` | Set `true` for compiled modules |
| `signal_check_required` | boolean | `true` | Fail if `tools/signal_check.js` absent |

## vitest.yml Inputs Reference

| Input | Type | Default | Notes |
|-------|------|---------|-------|
| `vitest_command` | string | `'npm test'` | Override for modules with a custom vitest config |

---

## Adding a New Module

1. Create `.github/workflows/ci.yml` in the new module repo.
2. Set the correct `branches:` trigger (`main` or `master`).
3. Add `repository_dispatch: types: [lib-updated]` if the module depends on `ionrift-library`.
4. Call `hygiene.yml` and `lint.yml` at minimum.
5. Call `vitest.yml` if the module has a Vitest suite.
6. Add the module to the `fan-out` job in `ionrift-library/ci.yml`.

---

## Updating Signal Check Patterns

`tools/signal_check.js` is maintained in `ionrift-devtools`. When the pattern list changes:

1. Copy the updated file into each module's `tools/` directory.
2. Commit and push. CI will pick up the new patterns on the next run.

The reusable `lint.yml` calls `node tools/signal_check.js` from the **consumer repo's** checkout, so each module controls its own copy.
