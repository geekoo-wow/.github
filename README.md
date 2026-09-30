# geekoo-wow/.github

Shared release tooling and org defaults for the geekoo-wow World of Warcraft
addons. Addon repositories call the workflows here instead of carrying their
own copies.

| File | What it is |
| --- | --- |
| [`.github/workflows/addon-release.yml`](.github/workflows/addon-release.yml) | Reusable workflow: builds the release notes, packages the addon and publishes it. |
| [`.github/workflows/addon-ci.yml`](.github/workflows/addon-ci.yml) | Reusable workflow (optional): luacheck, tests and a test package. |
| [`workflow-templates/`](workflow-templates) | "WoW addon release" template offered in the Actions tab of new repos. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Default contributing guide: the `Changelog:` commit convention and how to release. |

## Release workflow

On a version tag, `addon-release.yml`:

1. Collects the `Changelog: <sentence>` lines from the commit messages since
   the previous tag into an untracked `RELEASE_NOTES.md`, under the heading
   `# <addon-name> <tag>`. With no such lines the notes say
   "Maintenance release."
2. Runs [BigWigsMods/packager](https://github.com/BigWigsMods/packager), which
   publishes a GitHub release, uploads to CurseForge when the TOC has
   `X-Curse-Project-ID`, and to Wago when it has `X-Wago-ID`.

Add this as `.github/workflows/release.yml` in the addon repo (or pick the
"WoW addon release" template in the Actions tab):

```yaml
name: Release
on:
  push:
    tags: ["v*"]
jobs:
  release:
    uses: geekoo-wow/.github/.github/workflows/addon-release.yml@v1
    permissions:
      contents: write
    with:
      addon-name: AddonName   # heading of the release notes
    secrets: inherit
```

Requirements:

- The caller must grant `permissions: contents: write`. A called workflow
  cannot have more permissions than its caller, and creating the GitHub
  release needs write access.
- The caller must pass `secrets: inherit`, so the org secrets `CF_API_KEY`
  and `WAGO_API_TOKEN` reach the packager.
- The addon's `.pkgmeta` must point the packager at the generated notes:

  ```yaml
  manual-changelog:
    filename: RELEASE_NOTES.md
    markup-type: markdown
  ```

- `RELEASE_NOTES.md` is never committed. See [CONTRIBUTING.md](CONTRIBUTING.md)
  for how to write `Changelog:` lines and how to tag a release.

## CI workflow

`addon-ci.yml` is optional. It has two jobs:

- `check` runs `luacheck .` when the repo has a `.luacheckrc`, and the test
  command when one is given, both on Lua 5.1.
- `package` runs on pushes only: it builds the addon with the packager without
  uploading anything and keeps the result as an artifact named
  `<addon-name>-<sha>`. The download contains the addon folder, ready to drop
  into `Interface/AddOns`.

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  ci:
    uses: geekoo-wow/.github/.github/workflows/addon-ci.yml@v1
    with:
      addon-name: AddonName
      test-command: lua5.1 tests/run.lua
```

| Input | Default | |
| --- | --- | --- |
| `addon-name` | required | Artifact name prefix. |
| `test-command` | `""` | Command that runs the tests. Empty to skip. |
| `package` | `true` | Set to `false` to skip the test package. |

## Versioning

Callers pin to `@v1`, never to `@main`, so work in progress here cannot break
an addon release.

- **Backwards-compatible changes** (fixes, new optional inputs): commit to
  `main`, then move the `v1` tag. Every addon picks the change up on its next
  run.

  ```bash
  git tag -fa v1 -m v1 && git push -f origin v1
  ```

- **Breaking changes** (renamed or newly required inputs, changed
  conventions): tag `v2` and leave `v1` where it is. Addons move over by
  changing `@v1` to `@v2`.
