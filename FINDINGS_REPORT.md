# Research Findings and Fixes

## 1. `Kernel.cs` Method Signatures
- **Issue**: The parameterless `RefreshUICulture()` and `PreferencesUpdated()` in `src/Kinovea/Kernel.cs` were throwing `NotImplementedException`, which could cause unexpected application crashes.
- **Fix**: Updated both methods to pass `true` to their respective private parameterized counterparts:
  ```csharp
  public void RefreshUICulture()
  {
      RefreshUICulture(true);
  }

  public void PreferencesUpdated()
  {
      PreferencesUpdated(true);
  }
  ```

## 2. Fitter Settings (Hardcoded String)
- **Issue**: The fitter's name "Cesar Correa" is currently hardcoded in `src/Kinovea/Workspace/FitSessionReportGenerator.cs` at line 23 (`private const string FitterName = "Cesar Correa";`).
- **Context for UI Fixes**: General application preferences are handled by `FormPreferences2.cs`. A dynamic "Fitter Settings" tab should ideally be added to this form, persisting values via the `PreferencesManager`, and used by `FitSessionReportGenerator` instead of the hardcoded string.

## 3. NSIS Process Termination
- **Issue**: The NSIS script (`src/Installer/kinovea.nsi`) lacks application termination prior to uninstallation. Line 173 has a `TODO: terminate app.` comment.
- **Proposed Solution**: Use the NSIS `nsProcess` plugin or the generic Win32 `FindWindow` / `SendMessage` to gracefully close the application, or `KillProc` / `ExecWait "taskkill /F /IM CassetteMotionPro.exe"` to forcefully terminate the process prior to the `RMDir` operations to prevent file-locking issues.

## 4. Workflows and Test Coverage
- **Issue**: Reviewing `.github/workflows/build.yml` reveals that the current CI pipeline builds both the portable ZIP and NSIS installer using `msbuild` for standard `.NET` and `C++/CLI` targets. However, there are no steps executing automated tests (`vstest.console.exe` or `dotnet test`) indicating a lack of integrated CI testing coverage. Adding test coverage targets would improve release reliability.
