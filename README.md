# GPUI Wayland Patches

This repository carries small local patches on top of Zed GPUI's `gpui-pre`
0.3.8 snapshot. It is shared by applications that need compositor-controlled
Wayland backdrop blur.

## Included changes

- `gpui-pre`: adds `Window::set_background_blur_regions` so a view can limit
  compositor blur to window-local rectangles.
- `gpui-pre-linux`: supports `ext-background-effect-v1` when the compositor
  advertises blur, with KDE's blur protocol as a fallback.

The source bases and local changes are recorded in each crate's
`LOCAL-PATCH.md`.

## Use from an application

Add the same pinned Git revision for both crates in the application's Cargo
workspace root:

```toml
[patch.crates-io]
gpui-pre = { git = "https://github.com/Aorus22/gpui-wayland-patches", rev = "<revision>" }
gpui-pre-linux = { git = "https://github.com/Aorus22/gpui-wayland-patches", rev = "<revision>" }
```

Keep `gpui-pre`, `gpui-pre-platform`, and related GPUI integrations on matching
versions. Applications such as WA-Bot must also use a compatible
`gpui-component` release.
