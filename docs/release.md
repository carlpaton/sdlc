# Releasing

The skills ship as a Claude Code plugin. The version lives in one place, `version` in [`.claude-plugin/plugin.json`](../.claude-plugin/plugin.json). Claude Code uses that field to decide whether users get an update, so a change only reaches users once the version is bumped. A GitHub release makes the version visible on GitHub and carries the notes.

`.claude-plugin/marketplace.json` has no version. Leave it that way so there is one place to bump.

## Choosing the version

Use [semantic versioning](https://semver.org). While the project is early stage, stay on `0.x`.

| Change | Bump | Example |
|--------|------|---------|
| Breaking: a skill is renamed or removed, or its output files change shape | minor while `0.x`, otherwise major | `0.1.0` to `0.2.0` |
| A new skill or feature | minor | `0.1.0` to `0.2.0` |
| A fix or wording change | patch | `0.1.0` to `0.1.1` |

## Steps

1. Bump `version` in `.claude-plugin/plugin.json`.
2. Check the plugin still validates:

   ```
   claude plugin validate .
   ```

3. Commit and push to `main`:

   ```
   git add .claude-plugin/plugin.json
   git commit -m "Release v0.1.0"
   git push origin main
   ```

4. Create the tag and release:

   ```
   gh release create v0.1.0 --title "v0.1.0" --generate-notes
   ```

   This tags the latest commit on `main`, so push first. `--generate-notes` builds the notes from the PRs and commits since the previous release. Add `--prerelease` while the project is early stage.

5. Check that users get it:

   ```
   claude plugin marketplace update carlpaton
   claude plugin update sdlc@carlpaton
   ```

The tag is `v` plus the version in `plugin.json`, so `0.1.0` is tagged `v0.1.0`.

## Fixing a mistake

- **Wrong notes:** `gh release edit v0.1.0 --notes "..."`.
- **Wrong version released:** do not move or reuse the tag, because users may already have it. Bump the patch version and release again.
- **Release made by accident:** `gh release delete v0.1.0 --cleanup-tag`, and only if nobody has installed it yet.

## Later

A GitHub Action could create the release whenever `version` changes on `main`, which would make step 4 automatic. It is not built.
