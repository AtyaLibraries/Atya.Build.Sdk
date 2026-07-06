# Atya.Build.Sdk Release Procedure

Release the SDK only through the standard protected flow.

1. Open a pull request to `development` with the SDK change.
2. Wait for CI to pass on the pull request and merge it through the required PR flow.
3. Verify `development` is current locally:

   ```powershell
   git fetch origin
   git switch development
   git reset --hard origin/development
   ```

4. Pack and verify the release candidate locally. At minimum, use the existing integration-consumer pattern to confirm:
   - `dotnet msbuild -getProperty:TreatWarningsAsErrors` returns `true` for a packable project.
   - A packable project with a deliberate compiler warning fails the build.
   - A stable package with a prerelease dependency fails pack with `NU5104`.
5. Create and push the release tag:

   ```powershell
   git tag vX.Y.Z
   git push origin vX.Y.Z
   ```

6. Wait for the `publish` workflow to request the `production` deployment approval.
7. Request human approval and wait. Never approve the production deployment yourself.
8. After publish completes, verify the new stable version appears on nuget.org.
9. Trigger the SDK canary rollout in `Atya.Foundation.Guards` only.
10. Wait for the Guards rollout PR to pass CI and merge.
11. Roll the remaining package repositories only after the Guards canary is green.

If the rollout exposes new XML documentation warnings, add the missing documentation in the rollout PR. Do not suppress `CS1591`.
