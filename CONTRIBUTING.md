# Contributing

This is the default contributing guide for geekoo-wow addon repositories. A
repository with its own `CONTRIBUTING.md` follows that one instead.

## Changelog lines

Release notes are built from the commit messages, so every user-facing change
needs a `Changelog:` line in its commit.

- Add one `Changelog:` line per user-facing change at the end of the commit
  message, alongside trailers like `Co-Authored-By`.
- Write for players: describe the visible in-game effect, not the
  implementation.
- One complete sentence per line, with `Changelog:` starting at the beginning
  of the line.
- Use past tense or a noun phrase, capitalized, ending with a period.
- Internal changes (CI, refactors, docs, tests) get no `Changelog:` line.

Example:

```
Fix frame position after a UI scale change

The anchor offsets were cached at login and never recomputed, so changing
the UI scale left frames offset until the next reload.

Changelog: Fixed frames drifting out of place after changing the UI scale.
Changelog: Added a /reset command that restores the default positions.

Co-Authored-By: Jane Doe <jane@example.com>
```

This becomes:

```
- Fixed frames drifting out of place after changing the UI scale.
- Added a /reset command that restores the default positions.
```

## RELEASE_NOTES.md

`RELEASE_NOTES.md` is generated in CI at release time from the `Changelog:`
lines since the previous tag. Never create, edit or commit it.

## Releasing

1. Make sure the user-facing commits since the last tag have their
   `Changelog:` lines.
2. Create an annotated tag. It must be annotated: the packager takes the
   version from it.

   ```bash
   git tag -a vX.Y.Z -m "<Addon> vX.Y.Z"
   ```

3. Push the tag. This starts the release workflow, which packages the addon
   and publishes it.

   ```bash
   git push origin vX.Y.Z
   ```

If there are no `Changelog:` lines since the previous tag, the notes say
"Maintenance release."
