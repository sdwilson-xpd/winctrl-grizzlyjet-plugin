# GrizzlyJet-X WinCTRL fork  Codex reconstruction handoff

**Status:** Baseline committed by project owner, November 11, 2026. Work on this phase is **complete**, subject to bug fixes. This is a *reconstruction specification*, not a request to change the committed baseline.

**Purpose:** Given the **original version of the forked WinCTRL project**, reproduce the current GrizzlyJet-X CDU-focused, globally installed X-Plane plugin baseline as closely as possible. Work incrementally and report differences instead of making undocumented changes. The files shipped in this archive are **reference snapshots**, not a complete original or complete modified repository.

## 0. Instructions for Codex (paste this section as the task)

> Starting from the original WinCTRL fork, implement only the **Completed baseline** in this document, consulting the bundled source snapshots. First inspect the actual repository (including `CMakeLists.txt`, `src/include/products/fmc/product-fmc.cpp`, `src/include/products/fmc/profiles/`, `usbdevice.cpp`, and project entry points). Verify path and target names before editing; do not infer them from the snippets alone. Preserve the existing filenames (especially `product-fmc.cpp`) and preserve platform-specific Mac/Linux functionality. Do not remove the E-Jets profile yet. Do not implement any **Future work**. If a snapshot conflicts with the target repository or cannot be integrated without changes beyond scope, report the discrepancy and make the minimum compatible patch. Build and test as far as the available SDK and hardware permit; report what was actually verified. Never state that a build or runtime test succeeded unless run.

## 1. Scope / architectural decisions

- Project: a fork of WinCTRL; X-Plane global plugin (not installed solely inside an individual aircraft).
- Baseline is **CDU-focused** (ProductFMC / MCDU / PFP family), with E-Jets and GrizzlyJet-X aircraft profiles. This initial reduction of compiled hardware implementations is **temporary**; retain hardware definitions/interfaces and do not destroy other hardware source trees.
- GrizzlyJet-X uses an **E-Jets-derived FMS**, *not* a Citation X FMS.
- For now the GrizzlyJet profile deliberately uses `XCrafts/FMS/...` datarefs and `XCrafts/ERJ/...` commands. These will change **later**, independently of the E-Jets profile.
- Retain the original E-Jets profile as a reference and fallback until GrizzlyJet testing succeeds. Aircraft-specific profile selection should prioritize GrizzlyJet.
- Keep the CMake project cross-platform: Windows/MSVC, macOS/Clang, Linux/GCC. Windows build success alone is not evidence of cross-platform success.
- Preserve CMake as the source of truth. Changes made exclusively to generated Visual Studio `.vcxproj` files can be overwritten by regeneration.
- Hardware auto-detection/configuration, unique plugin identity, and coordinating the global fork with upstream WinCTRL are **future work**, not part of this baseline.

## 2. Completed baseline — chronological recap

1. **VS2017 → VS2022 conversion.** Generated a Visual Studio 2022 x64 CMake solution. Earlier sessions reported a working configuration using `cmake -S . -B build\\vs2022 -G "Visual Studio 17 2022" -A x64 -DSDK_VERSION=420`. MSVC compile fixes captured in final CMake include `_CRT_SECURE_NO_WARNINGS`, `NOMINMAX`, `/bigobj`, `/utf-8`, `/Zc:__cplusplus`, `/std:c++latest` and `/MP` (only after `winctrl` exists). Non-MSVC `-O2` and platform options were retained. An early `-g1` warning was encountered with MSVC; avoid applying GCC/Clang-only flags globally. **Other source-level VS2022 repairs are not fully present in this chat's attached snapshots.** Earlier work reportedly involved `usbdevice.cpp` and temporarily excluded AGP and other non-CDU implementation files; inspect the original fork, compiler diagnostics, and current source before reproducing those exact fixes. Do not fabricate patches.
2. **CDU-focused build.** CMake's `FIND_SOURCE_FILES` collects sources recursively while omitting `desktop` and `windows-stresstest`. `add_xplane_plugin` explicitly excludes other FMC aircraft profile `.cpp` files and re-adds the two required profiles; it filters out non-CDU hardware implementation `.cpp` files in `/products/(agp|ecam|fcu-efis|joystick|nws|orion-throttle|pap3-mcp|pdc|rmp|tcas|ursa-minor-throttle)/`. Existing hardware enums, interfaces and source files remain in the repository. Earlier simplification of USB device factory handling to ProductFMC cases was reported but no reference `usbdevice.cpp` is bundled here, so verify rather than guessing.
3. **Separate GrizzlyJet profile.** `grizzlyjet-fmc-profile.h/.cpp` were derived from `xcrafts-ejets-fmc-profile.h/.cpp`. New class `GrizzlyJetFMCProfile` retains the E-Jets CDU drawing, button maps, text color map, font selection, display datarefs, radio line behavior, brightness, and annunciator monitoring; all initial datarefs and commands remain in the X-Crafts namespaces.
4. **Airframe detection and profile priority.** `GrizzlyJetFMCProfile::IsAircraftLoaded()` uses `XPLMGetNthAircraftModel(0, aircraftName, aircraftPath)` and checks the displayed aircraft name and `.acf` basename. Alphanumeric normalization makes matches insensitive to case, whitespace and hyphens; it looks for `grizzlyjet`. `IsEligible()` requires both this aircraft identity and existence of `XCrafts/FMS/CDU_1_01`. In `ProductFMC::setProfileForCurrentAircraft()`: choose GrizzlyJet if eligible; otherwise choose E-Jets only if a GrizzlyJet aircraft **is not loaded** and E-Jets `IsEligible()` is true; otherwise choose no profile. Thus a GrizzlyJet without the expected CDU dataref cannot silently fall back to E-Jets.
5. **Source filename correction.** The generated changed file was originally delivered as `product-fmc-grizzlyjet.cpp` for review, but in the project its correct, unchanged filename is **`product-fmc.cpp`**. Replace/edit the existing source; do **not** add the renamed file as a second translation unit.
6. **CMake cleanup.** Removed redundant old E-Jets-only filtering inside `FIND_SOURCE_FILES`, leaving the single profile-selection filter inside `add_xplane_plugin`. Moved MSVC `/MP` targeting `winctrl` after `add_xplane_plugin(winctrl ...)`, so the target exists first.
7. **Output/deployment.** A Windows-only `POST_BUILD` custom command on **`winctrl`**, not `ALL_BUILD` or the solution, creates three destination directories and copies the actual built target (`$<TARGET_FILE:winctrl>`) to `winctrl.xpl` in each. CMake forward-slash paths and `VERBATIM` are used. The build target for SDK >=300 sets MSVC `OUTPUT_NAME` to target name, `PREFIX ""`, `SUFFIX ".xpl"` and output directory to `${CMAKE_BINARY_DIR}/${DIRECTORY_PREFIX}`. The post-build commands run on real builds/rebuilds and **not** on an up-to-date, skipped Build invocation.
8. **IDE-only preferences.** Custom VS2017 editor Fonts and Colors were exported as `.vssettings` and imported into VS2022. This is a user/editor preference, **not** part of the repo's CMake/source migration. Proposed CDU color-tint revisions were discussed but **not implemented** in this task.
9. **Acceptance.** User reported successful VS2022 builds and a working global CDU-focused fork earlier, then committed this new baseline. This handoff does **not** independently verify a full build or X-Plane runtime test of the final snapshots.

## 3. Exact CMake reference and insertion anchors

Treat the bundled **root `CMakeLists.txt`** as the authoritative snapshot of the final edited build configuration. Its important locations are:

- Before the target: `FUNCTION(FIND_SOURCE_FILES return_list base_dir)` excludes only desktop/testing trees; no special E-Jets-only filter remains here.
- Inside `FUNCTION(add_xplane_plugin library_name library_version src_dir)`, **immediately after** `FIND_SOURCE_FILES(FILES ${src_dir})`, the following two-profile and hardware filters are present **before** the platform-specific `ADD_LIBRARY`/`ADD_EXECUTABLE` call:

```cmake
# Keep the E-Jets and GrizzlyJet X CDU profiles
list(FILTER FILES EXCLUDE REGEX "/profiles/.*\\.cpp$")
list(APPEND FILES
    "${src_dir}/include/products/fmc/profiles/xcrafts-ejets-fmc-profile.cpp"
    "${src_dir}/include/products/fmc/profiles/grizzlyjet-fmc-profile.cpp"
)
# Exclude all non-CDU hardware implementations
list(FILTER FILES EXCLUDE REGEX
    "/products/(agp|ecam|fcu-efis|joystick|nws|orion-throttle|pap3-mcp|pdc|rmp|tcas|ursa-minor-throttle)/.*\\.cpp$"
)
```

- `add_xplane_plugin` defines the actual `winctrl` target and configures MSVC options for its library build.
- **Near end of file, after `ENDFUNCTION(add_xplane_plugin)` and `IF(NOT DEFINED SDK_VERSION)` guard:**

```cmake
add_xplane_plugin(winctrl ${SDK_VERSION} "${CMAKE_CURRENT_SOURCE_DIR}/src")
if(MSVC)
    target_compile_options(winctrl PRIVATE /MP)
endif()
if(WIN32)
    set(PLUGIN_DIRS
        "E:/X-Plane Hub/X-Plane 12/Aircraft/! GrizzlyJet X/plugins/winctrl/win_x64"
        "E:/X-Plane Hub/X-Plane 12/Aircraft/! X-Crafts Aircraft/EJets Family WIP/plugins/winctrl/win_x64"
        "E:/X-Plane Hub/X-Plane 12/Aircraft/! X-Crafts Aircraft/Embraer E-Jets Family v2.1.0/plugins/winctrl/win_x64"
    )
    foreach(PLUGIN_DIR IN LISTS PLUGIN_DIRS)
        add_custom_command(TARGET winctrl POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E make_directory "${PLUGIN_DIR}"
            COMMAND ${CMAKE_COMMAND} -E copy
                "$<TARGET_FILE:winctrl>"
                "${PLUGIN_DIR}/winctrl.xpl"
            VERBATIM
        )
    endforeach()
endif()
```

Those three E: paths are **user-machine-specific destinations**, not universal project defaults. On another machine or in a CI environment, do not deploy to absent local drives; guard or parameterize destinations without changing baseline behavior on the user's current system. Do not change the target name from `winctrl` without verifying all references.

## 4. Source reference files / minimum implementation diff

All paths below are relative to repository root; confirm they exist in the checked-out original fork first.

- `src/include/products/fmc/profiles/grizzlyjet-fmc-profile.h` — add new class declaration and `IsAircraftLoaded`, `IsEligible` (bundled exact file).
- `src/include/products/fmc/profiles/grizzlyjet-fmc-profile.cpp` — add new class implementation (bundled exact file).
- `src/include/products/fmc/product-fmc.cpp` — **edit in place**, add `#include "profiles/grizzlyjet-fmc-profile.h"` and modify `setProfileForCurrentAircraft()` as in bundled file. The rest of the supplied original `product-fmc.cpp` is unchanged by this profile-selection edit.
- `src/include/products/fmc/profiles/xcrafts-ejets-fmc-profile.h/.cpp` — retain, not replace; copies supplied as reference.
- `CMakeLists.txt` — modify to match bundled snapshot as closely as the original repository structure permits.

The exact changed `ProductFMC::setProfileForCurrentAircraft()` is:

```cpp
void ProductFMC::setProfileForCurrentAircraft() {
    if (GrizzlyJetFMCProfile::IsEligible()) {
        clearDisplay();
        profile = new GrizzlyJetFMCProfile(this);
        profileReady = true;
    } else if (!GrizzlyJetFMCProfile::IsAircraftLoaded() && XCraftsEjetsFMCProfile::IsEligible()) {
        clearDisplay();
        profile = new XCraftsEjetsFMCProfile(this);
        profileReady = true;
    } else {
        profile = nullptr;
        profileReady = false;
    }
}
```

**Important nuance:** The current GrizzlyJet `IsEligible()` **does** depend on an X-Crafts dataref existing, despite the identity check being independent of the namespace. Preserve this for baseline compatibility. It must be revisited when the future dataref namespace changes.

## 5. Verification / acceptance criteria for Codex

1. Audit target names, source paths, and generated solution before editing. Ensure no duplicate `product-fmc` translation units or duplicated E-Jets profile source lists.
2. CMake configure for VS2022 x64 (with the SDK version actually supported by the checked-out repository; `420` was used previously):
   `cmake -S . -B build/vs2022 -G "Visual Studio 17 2022" -A x64 -DSDK_VERSION=420`.
3. Build the `winctrl` target in Debug and Release. Check all source files compile and link to `.xpl` using the SDK and environment actually available. Do not assume the referenced third-party SDK/libraries exist on a different machine.
4. On an actual link, confirm each destination directory is created and receives `winctrl.xpl`; on an up-to-date Build, expect CMake/MSBuild to skip `POST_BUILD` (this is not a defect).
5. Runtime tests with CDU hardware where available: GrizzlyJet `.acf` name/path selects `GrizzlyJetFMCProfile` **if** the X-Crafts FMS dataref exists; a non-GrizzlyJet E-Jets aircraft selects `XCraftsEjetsFMCProfile`; a GrizzlyJet without that dataref does **not** fall through to E-Jets. Verify CDU lines, LSK/key presses, EXEC/CALL/FAIL/MSG/OFST annunciators, power, and brightness.
6. Check that aircraft changes/unload still properly deregister dataref callbacks and refresh the device profile. Check that changing the CDU side still works where supported.
7. Preserve Mac/Linux CMake paths and compiler flags; at minimum run CMake configure or dry-run on available toolchains, and record what was not tested.
8. Record actual validation evidence (commands, results, test environment), not just intended behavior.

## 6. Known risks / items deliberately not solved

- The complete original fork and all fixes made during VS2022 migration were **not uploaded** here. The current attachments cover final CMake, two profiles and `ProductFMC`. Codex must derive any additional compiler compatibility changes from the real repository/errors; no exact diff for `usbdevice.cpp` is available here.
- The `.acf` detection looks for normalized substring `grizzlyjet` in the aircraft model name or `.acf` basename. Confirm the real aircraft uses this identity. The routine currently ignores folder names (except `.acf` basename).
- Potential collision with another global WinCTRL plugin and separation of identity/resource ownership is a future task, not resolved in this snapshot.
- Hardware-only/CDU-limited build must eventually be expanded to other supported products, ideally with Auto/Enabled/Disabled per-device configuration to avoid needless USB polling. Do not implement in this reconstruction.
- The post-build destination strings target three **aircraft-local plugin folders**, even though the fork has been described as a global plugin. The source record supports the copy destinations and the description but does not establish why those are mixed. Preserve paths faithfully; ask the project owner before changing the deployment model.
- Existing code comments that refer to X-Crafts E-Jets 3.0 are inherited from the copied profile; cosmetic cleanup is optional *later*, not a functional change now.
- MSVC and GCC/Clang flag differences matter; maintain conditional flags rather than deleting cross-platform support.
- CMake's source-globbing behavior can require manual regeneration to discover newly added `.cpp` files. Explicitly listed profile sources here mitigate that for the two profiles.
- Visual Studio `.vssettings` palette preferences and proposed custom CDU color tints are not supplied as part of the baseline.

## 7. Deliverable expected from Codex

A reviewable patch applied to the **original** fork, preserving pre-existing filenames; a concise file-by-file change log; a validation report with both successful and untested checks; and a list of any discrepancies between this handoff snapshot and the working baseline. Do not claim bit-for-bit equivalence without the committed baseline commit SHA and a full original-to-final diff.

## 8. Provenance and confidence

**Verified against attached files:** final `CMakeLists(2).txt`; original E-Jets profile `.h/.cpp`; newly generated GrizzlyJet profile `.h/.cpp`; before/after `product-fmc.cpp`; archive contents. The included snapshots have not been compiled in this environment.

**Reported earlier in conversation but not available as full patch:** VS2022 compiler migration work, USB factory changes, successful build/load, project owner's baseline Git commit.

**Not completed:** future namespace replacement, E-Jets profile removal, global plugin coexistence, restoration/configuration of other hardware, cosmetic CDU color revisions.
