## Cursor Cloud specific instructions

Arsenal Image Mounter is a Windows virtual SCSI disk-image mounter (kernel driver plus .NET tools). This VM has .NET SDK 8 and 9 on `PATH` (`dotnet`, install root `/usr/share/dotnet`). Use them to build the managed tree and to run `aim_cli` image create, convert, and checksum. Driver install, virtual-disk mount, and the WinForms apps need Windows. Build overview: `How-to-build.md`. Target frameworks: `Managed Source/Directory.build.props`.

- Case-sensitive checkout: MSBuild imports `Directory.Build.props` only. The repo file is `Managed Source/Directory.build.props`. The startup update script creates the relative symlink `Managed Source/Directory.Build.props` → `Directory.build.props`. Leave that symlink untracked so it does not collide with the real file on Windows.
- Restore and build the SDK projects under `Managed Source/`. `dotnet restore` of `ArsenalImageMounter.sln` fails because `ArsenalImageMounterHelpBuilder.shfbproj` has an empty target framework (`Invalid framework identifier ''`).
- Windows TFMs (`net8.0-windows`, `net9.0-windows`, WinForms) compile when you pass `-p:EnableWindowsTargeting=true`. Running them still requires the Windows desktop runtime.
- Development build of the cross-platform projects:

```
dotnet build "Managed Source/ArsenalImageMounterService/ArsenalImageMounterService.csproj"
dotnet build "Managed Source/ArsenalImageMounterAnalyze/ArsenalImageMounterAnalyze.csproj"
dotnet build "Managed Source/Arsenal.ImageMounter.PowerShell/Arsenal.ImageMounter.PowerShell.csproj"
```

  Add `-p:EnableWindowsTargeting=true` for `Arsenal.ImageMounter.Forms`, `ArsenalImageMounterMountTool`, `ArsenalImageMounterGUISetup`, `ArsenalImageMounterCLISetup`, and `TestProject`.
- Build output is `Managed Source/bin/<tfm>/`. The Linux CLI binary is `Managed Source/bin/net8.0/aim_cli`.
- `aim_cli --filename=<image> --convert=<out>` and `aim_cli --filename=<image> --checksum` complete without the kernel driver (see `Managed Source/Command-line-usage-examples.md`). `--mount`, `--list`, `--dismount`, and `--rescan` exit with a Windows-only message.
- `aim_cli --create --filename=<image> --disksize=<size>` writes the image, then exits 3 with `Value cannot be null. (Parameter 'BaseProvider')`. The image file is still valid for `--convert` and `--checksum`.
- `dotnet test "Managed Source/TestProject/TestProject.csproj" -p:EnableWindowsTargeting=true` builds, then the host cannot start. The project targets `net48`, `net8.0-windows`, and `net9.0-windows`. There is no `Microsoft.WindowsDesktop.App` runtime on Linux, and several tests call Win32 APIs.
- There is no separate lint script. `RunAnalyzersDuringBuild` is false. `dotnet build <project> -p:RunAnalyzersDuringBuild=true` runs the SDK analyzers.
- `Unmanaged Source/` (WDK drivers, `aim_ll`, `aimapi`) is not built here.
