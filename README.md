# Atya.Build.Sdk

`Atya.Build.Sdk` centralizes Atya's build, packaging, SourceLink/MinVer, naming, and
NuGet metadata defaults.

## Usage

Pin the SDK version in the repository's `global.json`:

```json
{
  "msbuild-sdks": {
    "Atya.Build.Sdk": "1.3.2"
  }
}
```

Use the SDK from each project:

```xml
<Project Sdk="Atya.Build.Sdk">
  <PropertyGroup>
    <PackageId>Atya.Foundation.Example</PackageId>
    <AssemblyName>Atya.Foundation.Example</AssemblyName>
    <RootNamespace>Atya.Foundation.Example</RootNamespace>
    <Description>Example Atya package.</Description>
    <Authors>AtyaLibraries</Authors>
    <RepositoryUrl>https://github.com/AtyaLibraries/Example</RepositoryUrl>
  </PropertyGroup>
</Project>
```

## Defaults

All defaults are conditional and can be overridden in the consuming project.

- Supply chain: `NuGetAudit`, `NuGetAuditMode`, `NuGetAuditLevel`,
  `UseLockFiles`, `RestorePackagesWithLockFile`
- Framework and language: `TargetFramework`, `LangVersion`, `Nullable`,
  `ImplicitUsings`
- Build quality: `AnalysisLevel`, `EnforceCodeStyleInBuild`, `Deterministic`,
  `ContinuousIntegrationBuild`, `TreatWarningsAsErrors`
- Repository-derived: `RepositoryHasGitDirectory`,
  `RepositoryUrlIsConfigured`
- Packaging: `Company`, `RepositoryType`, `PackageLicenseExpression`,
  `PackageReadmeFile`, `GenerateDocumentationFile`, `IncludeSymbols`,
  `SymbolPackageFormat`, `DebugType`, `EmbedUntrackedSources`,
  `PublishRepositoryUrl`, `MinVerTagPrefix`
- Test projects: centralized versions and injected references/global usings for
  xUnit, FluentAssertions 7.2.0, NSubstitute, coverlet, and
  Atya.Governance.Testing when `IsTestProject=true`

Packable projects must use an `Atya.{Area}.{Name}` package ID, align
`AssemblyName` and `RootNamespace` with `PackageId`, provide `Description`,
`Authors`, and `RepositoryUrl`, and contain no unresolved `__` placeholders in
package metadata.

Set `SkipPackageNamingValidation=true` for an explicit naming exception. Set
`AtyaDisableBuildSdkGuards=true` only when all naming and metadata guards must be
disabled. Set `AtyaDisableInjectedReferences=true` to opt out of injected package
references.
