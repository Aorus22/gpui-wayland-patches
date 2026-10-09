# Local GPUI Linux patch

This is `gpui-pre-linux` 0.3.8, a packaged snapshot of Zed GPUI at commit
`279fe070bb389b79652e52065b2f001edcc0b11b`.

The local change adds Wayland `ext-background-effect-v1` support for
`WindowBackgroundAppearance::Blurred`. It listens for the compositor's blur
capability, applies a blur region to each requested surface, and keeps the
existing KDE blur protocol as a fallback. Views can override the default
full-surface blur region through `Window::set_background_blur_regions`.

When updating GPUI, update `gpui-pre`, `gpui-pre-platform`, and this backend
snapshot together, then carry this protocol change forward.
