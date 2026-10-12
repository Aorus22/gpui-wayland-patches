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

Layer-shell surfaces also report `Decorations::Server`: they cannot have window
decorations, so client-side border/shadow insets from UI toolkits would only
waste space around a popover.

A 0x0 window (layer surfaces that let the compositor pick the size) skips
`wp_viewport.set_destination` until it has a real size: a fractional-scale event
that arrives before the first configure otherwise sends a zero destination and
the compositor disconnects the client (`negative or zero values in width or height`).
