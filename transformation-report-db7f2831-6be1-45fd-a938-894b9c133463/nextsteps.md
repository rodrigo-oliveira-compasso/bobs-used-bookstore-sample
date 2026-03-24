# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run the following command from the solution root to ensure all NuGet packages are restored correctly:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages targeting older frameworks.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compile-time issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas of the code that may behave differently on cross-platform .NET compared to .NET Framework.

---

## 3. Run the Test Project

Execute the test suite in `Bookstore.Domain.Tests` to verify that domain logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated to determine whether the failure is due to a behavioral difference in cross-platform .NET or a pre-existing issue.

---

## 4. Validate the Data Layer

The `Bookstore.Data` project likely contains database access logic. Verify the following:

- **Connection strings** have been updated in configuration files (e.g., `appsettings.json`) to reflect the target environment.
- If **Entity Framework** is used, run the following to verify the model is consistent with the database schema:

```bash
dotnet ef migrations list --project app/Bookstore.Data
```

If there are pending migrations or model mismatches, resolve them before proceeding.

---

## 5. Run the Web Application Locally

Start the `Bookstore.Web` project locally to perform manual validation:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- Key pages and API endpoints load and return expected results.
- Authentication and authorization flows work correctly if applicable.
- Static assets (CSS, JavaScript, images) are served correctly.

---

## 6. Review Platform-Specific Code

Search the solution for any code that may have platform-specific behavior, including:

- Use of `System.Drawing` (not fully supported cross-platform without additional packages).
- Windows registry access (`Microsoft.Win32.Registry`).
- File path separators — ensure `Path.Combine` is used rather than hardcoded backslashes.
- Any use of `AppDomain` or reflection-based APIs that behave differently on modern .NET.

---

## 7. Validate the CDK Project

The `Bookstore.Cdk` project appears to contain infrastructure-as-code definitions. Verify the following:

- The CDK project targets the correct .NET version consistent with the rest of the solution.
- Run a CDK synthesis to confirm the infrastructure definitions compile and produce valid output:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Review the synthesized output for correctness before applying any infrastructure changes.

---

## 8. Check Configuration Files

Review the following configuration concerns across the solution:

- `appsettings.json` and `appsettings.Production.json` contain correct and complete settings.
- Any configuration previously stored in `Web.config` or `App.config` has been migrated to the appropriate `appsettings.json` structure.
- Environment-specific settings are handled using the `IConfiguration` / `IOptions` pattern.

---

## 9. Publish the Application

Once local validation is complete, publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Review the contents of the `./publish` directory to confirm all required files are present before deploying to the target environment.