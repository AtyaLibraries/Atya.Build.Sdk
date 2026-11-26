# Atya.Build.Sdk Release Procedure

Release the SDK only through the standard protected flow. This repository has **no
publish workflow of its own**: a tag push publishes nothing. Releases are made by the
central `AtyaLibraries/publisher` workflow, which owns the NuGet.org trusted-publishing
policy, the `production` approval, SBOM and provenance attestation.

1. Open a pull request to `development` with the SDK change.
2. Wait for CI to pass on the pull request and merge it through the required PR flow.
3. Verify `development` is current locally, without discarding local work:

   ```powershell
   git fetch origin
   git switch development
   git merge --ff-only origin/development
   ```

4. Pack and verify the release candidate locally. At minimum, use the existing integration-consumer pattern to confirm:
   - `dotnet msbuild -getProperty:TreatWarningsAsErrors` returns `true` for a packable project.
   - A packable project with a deliberate compiler warning fails the build.
   - A stable package with a prerelease dependency fails pack with `NU5104`.
5. Promote `development` to `master` with a pull request and a **merge commit** (not
   squash or rebase), then create and push the release tag on the resulting `master`
   commit. Never move or delete an existing release tag.

   ```powershell
   git tag vX.Y.Z
   git push origin vX.Y.Z
   ```

6. Start the release from the publisher. This is the only step that publishes. The
   SDK is a content-only package (`symbols: none`), so the explicit paths are required:

   ```bash
   gh workflow run publish.yml --repo AtyaLibraries/publisher \
     -f repository=AtyaLibraries/Atya.Build.Sdk -f ref=vX.Y.Z \
     -f solution=src/Atya.Build.Sdk.csproj \
     -f package_project=src/Atya.Build.Sdk.csproj \
     -f global_json=tests/IntegrationConsumer/global.json
   ```

   Follow the run with `gh run list --repo AtyaLibraries/publisher --workflow publish.yml`.
7. Wait for the `publish` job to request the `production` deployment approval in the
   publisher run, review the repository, tag and commit, and request human approval.
   Never approve the production deployment yourself.
8. After publish completes, verify the new stable version appears on nuget.org and that
   the run produced the sealed release bundle and attestation.
9. Trigger the SDK canary rollout in `Atya.Foundation.Guards` only.
10. Wait for the Guards rollout PR to pass CI and merge.
11. Roll the remaining package repositories only after the Guards canary is green.

If the rollout exposes new XML documentation warnings, add the missing documentation in the rollout PR. Do not suppress `CS1591`.

## Failure and rollback

A failed or rejected publisher run publishes nothing before credentials are requested;
fix the cause and start a new run. Published versions, tags and attestations are never
overwritten or deleted: roll forward with a new, higher version. See the publisher
`docs/RUNBOOK.md` and `docs/RELEASE-ARTIFACT-CONTRACT.md` for partial-publication
recovery.
