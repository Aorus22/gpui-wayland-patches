# Local GPUI authoring patch

This is `gpui-pre` 0.3.8, a packaged Zed GPUI snapshot. The local addition
exposes `Window::set_background_blur_regions` and a default no-op method on
`PlatformWindow`, so views can request compositor blur for selected
window-local rectangles without coupling widgets to Wayland APIs.

The Linux backend implementation lives in `vendor/gpui-pre-linux`.
