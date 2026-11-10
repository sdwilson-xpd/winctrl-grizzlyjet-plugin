# WINCTRL GrizzlyJet fork — Visual Studio 2022 build

This patch changes **only** the root `CMakeLists.txt`; no plugin source code is modified.
It removes GNU-only flags from MSVC builds and configures a `.xpl` output filename.

## Apply

1. In the local Git clone, make a safety branch (`git switch -c vs2022-build`).
2. Copy the supplied `CMakeLists.txt` over the file at the repository root.
3. Ensure the SDK directory exists with `SDK/CHeaders` and `SDK/Libraries/Win/XPLM_64.lib` and `XPWidgets_64.lib`. This requires X-Plane SDK 4.2 or newer, with the corresponding headers.
4. Open **Developer Command Prompt for VS 2022** from the Windows Start menu and change to your repository root.
5. Run:

```bat
cmake -S . -B build\vs2022 -G "Visual Studio 17 2022" -A x64 -DSDK_VERSION=420
cmake --build build\vs2022 --config Debug
cmake --build build\vs2022 --config Release
```

The generated `.sln` is located under `build\vs2022` and can be opened in Visual Studio 2022. The output is expected below `build\vs2022\win_x64\Debug\winctrl.xpl` or `...\Release\winctrl.xpl` (Visual Studio multi-config layout). Check actual output in the Visual Studio Build Output log.

## Notes

* This is a **configuration patch**, not a verified Windows binary. MSVC has not been run in this environment; additional source-level compiler diagnostics may still arise. If the configuration or build fails, share the **first error and its surrounding lines**.
* The upstream CMake sets C++23. VS2022 selects its newest supported mode (`/std:c++latest`), which is not a guarantee that every C++23 feature has been implemented. Update Visual Studio 2022 to its latest 17.x servicing release.
* `SDK_VERSION=420` enables XPLM420 definitions; a more recent SDK can still be placed in `SDK/`.
* Do not copy the DLL/import libraries from the SDK to your aircraft. `winctrl.xpl` is the plugin binary.
* Commit the CMake change after the Windows build passes; generated `build/` files should remain ignored by Git.
