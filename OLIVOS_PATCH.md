# OlivOS patch set

This fork is maintained by [OlivOS-Team](https://github.com/OlivOS-Team) for
[OlivOS Companion](https://github.com/OlivOS-Team/OlivOS-Companion). It carries
only the fixes required by the companion app; upstream code is otherwise left
unchanged so the branch can be rebased on newer releases.

## Baseline

- Upstream repository: https://github.com/pichillilorenzo/flutter_inappwebview
- Base commit: `17527cae5a3371c1e87f9c1df4281a7710debf28` (upstream `master`)
- Umbrella package: `flutter_inappwebview 6.2.0-beta.3`
- Windows package: `flutter_inappwebview_windows 0.7.0-beta.3`
- Maintenance branch: `olivos/main`

## Patches

### 1. Windows focus

- File: `flutter_inappwebview_windows/windows/in_app_webview/in_app_webview_manager.cpp`
- Change: create the inline WebView host with `WS_CHILD | WS_CLIPSIBLINGS`
  instead of style `0`.

Upstream creates the host as an owned top-level window. Clicking the WebView
activates that window, so the Flutter runner loses foreground state; the native
window title bar changes color and focus-related input behavior is affected.
A child window stays inside the Flutter view and no longer competes for
activation.

### 2. Texture bridge frame copying

- Files:
  - `flutter_inappwebview_windows/windows/custom_platform_view/texture_bridge.h`
  - `flutter_inappwebview_windows/windows/custom_platform_view/texture_bridge.cc`
  - `flutter_inappwebview_windows/windows/custom_platform_view/texture_bridge_gpu.h`
  - `flutter_inappwebview_windows/windows/custom_platform_view/texture_bridge_gpu.cc`
  - `flutter_inappwebview_windows/windows/custom_platform_view/texture_bridge_fallback.h`
  - `flutter_inappwebview_windows/windows/custom_platform_view/texture_bridge_fallback.cc`
- Change: track a serial for each accepted captured frame, and only copy the
  WebView texture into the Flutter surface when that serial changes or the
  surface size changes.

`TextureBridgeGpu::GetSurfaceDescriptor` is called for every Flutter frame,
while WebView2 may produce frames at a different cadence. The upstream code
copies the last WebView frame and calls `ID3D11DeviceContext::Flush` even when
no new frame arrived. On high-refresh displays this repeats expensive GPU work
up to the display refresh rate. Skipping the redundant copy and flush keeps
the Flutter UI responsive without changing the displayed frame.

## Upstream status (checked 2026-11-26)

- The upstream repository has had no code commit since 2026-02-10.
- The latest hosted Windows release is still `0.7.0-beta.3` (2026-02-04) and
  does not contain this fix.
- Related pull requests are closed without being merged:
  - https://github.com/pichillilorenzo/flutter_inappwebview/pull/2803
  - https://github.com/pichillilorenzo/flutter_inappwebview/pull/2802
  - https://github.com/pichillilorenzo/flutter_inappwebview/pull/2845
- Related issue is still open:
  - https://github.com/pichillilorenzo/flutter_inappwebview/issues/2736

## Removal condition

When upstream merges equivalent fixes and publishes a release containing them:

1. Switch `OlivOS-Companion` back to the hosted package.
2. Remove the `dependency_overrides` entry for `flutter_inappwebview_windows`.
3. Remove this file and the maintenance branch, or reset it to upstream.

## Maintenance notes

- Keep the patches limited to the files above. Do not reformat or upgrade
  upstream code in this branch.
- When rebasing, use an upstream tag or commit as the base and re-apply only the
  changes described under "Patches".
- Consumers should pin the branch commit SHA, not a moving branch name.
