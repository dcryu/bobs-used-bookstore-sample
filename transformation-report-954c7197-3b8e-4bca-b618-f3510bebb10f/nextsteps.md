# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all five projects after transformation:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the migration to cross-platform .NET was completed without introducing any compilation errors. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the solution root to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Verify that no warnings or errors appear in the output. Pay attention to any deprecation warnings, as these may indicate APIs that could cause issues at runtime even if they compile successfully.

---

## 2. Run the Unit Tests

Execute the test project to confirm that all existing tests pass under the new runtime:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any tests that were previously passing but are now failing.
- Any tests that were skipped or ignored that may need to be re-enabled.
- Any runtime exceptions that do not surface as compile-time errors.

---

## 3. Validate the Data Layer

The `Bookstore.Data` project likely contains database access logic (e.g., Entity Framework Core migrations and a `DbContext`). Perform the following checks:

- Confirm that the correct EF Core provider package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer` or `Npgsql.EntityFrameworkCore.PostgreSQL`) and is compatible with the target .NET version.
- Run any pending migrations against a development database:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

- Verify that the schema is created correctly and that seed data, if any, is applied as expected.

---

## 4. Validate the Web Application

Run the web application locally to confirm it starts and functions correctly:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- All routes respond as expected.
- Static files, middleware, and authentication (if applicable) are functioning correctly.
- Any configuration values in `appsettings.json` or environment variables are being read correctly under the new hosting model.

---

## 5. Validate the CDK Project

The `Bookstore.Cdk` project defines infrastructure. Confirm it synthesizes correctly:

```bash
cd app/Bookstore.Cdk
dotnet build --configuration Release
cdk synth
```

Review the synthesized CloudFormation template to ensure all resources are defined as expected and that no properties were lost or altered during the transformation.

---

## 6. Check for Runtime Behavioral Differences

Even with a clean build, cross-platform migration can introduce subtle behavioral differences. Review the following areas manually:

- **File paths**: Ensure no hardcoded Windows-style paths (e.g., `C:\` or backslash separators) exist in the code. Use `Path.Combine` or `Path.DirectorySeparatorChar` where applicable.
- **Case sensitivity**: Linux file systems are case-sensitive. Verify that file references, view names, and static asset paths use consistent casing.
- **Configuration**: Confirm that `appsettings.json`, environment variable names, and secrets are correctly configured for the target environment.
- **Encoding and line endings**: Verify that any file I/O operations handle encoding explicitly rather than relying on platform defaults.

---

## 7. Review Target Framework and Package Versions

Open each `.csproj` file and confirm:

- The `<TargetFramework>` element targets the intended .NET version (e.g., `net8.0`).
- All NuGet package versions are compatible with that target framework.
- No packages reference `netstandard` or `net4x` only targets that may have limited functionality on newer runtimes.

---

## 8. Deploy to the Target Environment

Once local validation is complete:

1. Publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

2. Deploy the published output to your target hosting environment (e.g., AWS Elastic Beanstalk, EC2, or Lambda, depending on what the CDK stack defines).

3. Deploy the infrastructure using the CDK project if it has not already been deployed:

```bash
cd app/Bookstore.Cdk
cdk deploy
```

4. After deployment, perform smoke testing against the live environment to confirm the application behaves correctly end-to-end.