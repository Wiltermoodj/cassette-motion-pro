# Codebase Assessment Report: Cassette Motion Pro

## 1. What it is and its intended purpose
**Cassette Motion Pro** is a professional bike fitting software application. It is a modified fork of the **Kinovea** open-source video analysis engine. The primary purpose of this software is to assist professional bike fitters by providing video analysis tools, side-by-side (before/after) comparisons, body angle overlays, and automated PDF/HTML report generation for their clients. It acts as an all-in-one workspace for client management and video-based biomechanical analysis tailored specifically for bike fitting.

## 2. Tech Stack
- **Languages**: C#, C++/CLI (for FFmpeg native integration), PowerShell/Batch (for CI/CD).
- **Framework**: .NET Framework 4.8 (Windows Desktop).
- **UI Framework**: Windows Forms (WinForms).
- **Video Processing Engine**: Custom engine built on FFmpeg and OpenCV (inherited from Kinovea).
- **Installer**: NSIS (Nullsoft Scriptable Install System).
- **CI/CD**: GitHub Actions for automated building of portable ZIPs and Windows Installer binaries.

## 3. Structure
The codebase isolates Cassette Motion Pro specific features while retaining Kinovea's core engine logic. The directory structure is organized as follows:
- **`src/Kinovea/`**: The core C# application.
  - **`Clients/`**: Client management logic (Client Manager, database models, etc.).
  - **`Workspace/`**: Custom bike-fitting workspace features (Guided Capture, Image Measurement Assistant, Report Generation).
  - **`Kernel.cs` & `Program.cs`**: The main entry points and application state managers.
- **`src/Installer/`**: Contains the NSIS installation scripts (`kinovea.nsi`) used to build the Windows installer.
- **`.github/workflows/`**: The CI/CD pipelines to build the application and installer.
- **`branding/`**: Editable source artwork and python scripts to generate logos, icons, and splash screens.

## 4. Functionality Gaps (Intended vs Actual)
- **Fitter Settings / Contact Information**: The release notes (`docs/releases/0.12.3.md`) state that "Phone, Email, and Website remain placeholders for a future Settings screen." Currently, the Fitter's name ("Cesar Correa") is hardcoded directly into the report generator (`FitSessionReportGenerator.cs`), and no UI settings screen exists to dynamically update these studio contact details.
- **Language / UI Culture Refreshing**: The application seems to have hooks intended for refreshing UI culture and settings dynamically across all sub-modules, but the parameterless public methods expose incomplete implementations.

## 5. Priority Targets for Debugging
While analyzing the application's core functionality, a few critical priority targets for debugging have surfaced, primarily around application state updates that could lead to crashes if triggered:

1. **`NotImplementedException` in `Kernel.cs` (Line 293)**:
   ```csharp
   public void RefreshUICulture()
   {
       throw new NotImplementedException();
   }
   ```
   *Issue*: There is an overloaded private method `RefreshUICulture(bool subModules)` that handles the actual culture refresh. The public parameterless method is currently throwing an exception, which will crash the app if triggered by an external UI event or module expecting the standard signature. It should be wired to call `RefreshUICulture(true)`.

2. **`NotImplementedException` in `Kernel.cs` (Line 317)**:
   ```csharp
   public void PreferencesUpdated()
   {
       throw new NotImplementedException();
   }
   ```
   *Issue*: Similarly, `PreferencesUpdated(bool sendMessage)` exists to handle the logic. The public parameterless method needs to bridge to the private implementation (e.g., `PreferencesUpdated(true)`) to prevent runtime crashes during preference updates.

3. **Installer Un-installation Lifecycle (`src/Installer/kinovea.nsi:173`)**:
   - There is a `TODO: terminate app.` comment indicating the installer currently does not automatically terminate a running instance of Cassette Motion Pro before attempting uninstallation. This could lead to locked file errors during an upgrade or uninstallation.
