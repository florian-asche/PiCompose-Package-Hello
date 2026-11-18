# AI Working Guide for PiCompose-Package-Software-Hello

This package exists to test the pipeline of the [PiCompose-Repository](https://github.com/florian-asche/PiCompose-Repository) end to end. The script is irrelevant, change it only to get a commit to test with. Use this repository for every pipeline test, never a real package.

## Rules

- Nothing is pushed, merged or released without the explicit consent of the developer for that test. A consent for one test does not cover the next one.
- Documentation and commit messages are in English.
- Never edit `debian/changelog` by hand, the workflow "Version" writes it.
- A pull request needs exactly one box under "Version bump" and one under "Types of changes" (copy the body from `.github/PULL_REQUEST_TEMPLATE.md`, tick `patch` and for example `maintenance`).
- Do not change the other package repositories or PiCompose or linux-voice-assistant to test something.

## Full test of the pipeline

Run the steps in this order. Replace `R=florian-asche/PiCompose-Package-Software-Hello`. Wait for runs with a loop over `gh run list -R $R --limit 3 --json status` until none is `in_progress` or `queued`, a single `sleep` over 2 minutes is not allowed by the tool.

1. **Pull request.** Branch from `main`, change a line of `picompose-hello`, push, `gh pr create` with the template body. Expected checks: `Apply` (label from the type box), `Check / Version label` (label `patch`), `Package / Prepare build`, `Package / Build (amd64)` and `(arm64)` pass, `Package / Publish` is skipped. The push of the branch builds a dev package (artifact for 14 days, nothing published). The log of `Prepare build` has the step "Find the shared workflows".
2. **Merge** with `gh pr merge <n> -R $R --squash --delete-branch` (only with consent). Expected: workflow `Version` runs on `main`, adds the commit `Version x.y.z` to `debian/changelog` (patch raises the last number) and creates the tag `vx.y.z` as **pre-release**. It starts `Build and publish a version` on the tag (event `workflow_dispatch`), which starts `Publish to APT repository` in the PiCompose-Repository (`gh run list -R florian-asche/PiCompose-Repository --workflow publish.yml`).
3. **Nightly check.** After about a minute GitHub Pages serves the new version: `curl -s https://florian-asche.github.io/PiCompose-Repository/dists/trixie-nightly/main/binary-amd64/Packages | grep -A1 '^Package: picompose-software-hello'`. The suite `trixie` still has the old version.
4. **Release.** Turn the pre-release into a release. `gh release edit` has no flag for this, use the API:
   ```bash
   id=$(gh api repos/$R/releases/tags/vX.Y.Z -q .id)
   gh api -X PATCH repos/$R/releases/$id -F prerelease=false -F make_latest=true
   ```
   Expected: `Build and publish a version` with event `release`, the `.deb` files attached to the release, release notes generated again, then a publish run in the hub. Check `dists/trixie/...` as in step 3.
5. **Report** what was checked and what not. Installing the package on a device (`apt install picompose-software-hello`) is not part of this test unless it was done.

## Known pitfalls

- A pull request that changes `debian/changelog` (for example a rename of the package) gives no version, because the workflow counts the commits since the last change of that file. `Version` then succeeds with "nothing to do". Start it by hand: `gh workflow run Version -R $R -f level=patch`.
- Direct pushes to `main` without a pull request make `Version` fail (every commit needs a merged pull request). Start it with a level to get past that.
- A tag build fails in the last step "Tell the APT repository to publish" when the secret `HUB_DISPATCH_TOKEN` is missing. The build and the release assets are fine then, start the build again after the secret is set.
- The publish workflow keeps only the newest waiting run. Start publishes one after the other.
- Dependabot pull requests get the label `patch` and raise the version like any other merge.
