# C/C++ Toolchain Guide

## 1) Repository-first detection
Use the repository's existing build system:
- If `CMakeLists.txt` or `CMakePresets.json` exists: use CMake.
- If Meson/Bazel/Makefile is the established system, follow it.
- If CI defines build/test commands, treat those as the source of truth.

## 2) Default CMake workflow (when CMake is present)
Prefer out-of-source builds:
- Configure: `cmake -S . -B build`
- Build: `cmake --build build --parallel`
- Test: `ctest --test-dir build --output-on-failure`
- For multi-config generators (Visual Studio, Xcode, Ninja Multi-Config), add `--config <Debug|Release>` to `cmake --build`.

If `CMakePresets.json` exists, prefer presets:
- Configure: `cmake --preset <name>`
- Build: `cmake --build --preset <name> --parallel`
- Test: `ctest --preset <name>`

## 3) Safety rules
- Do not add a new build system when one already exists.
- Keep changes minimal and buildable; run a quick build/test after changes.
