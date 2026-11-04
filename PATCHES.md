# gpui-pre-linux 0.3.3 + `ext_background_effect_v1` blur

This branch is a copy of the published `gpui-pre-linux` **0.3.3** crate with one
change: the Wayland backend also implements the standard
[`ext_background_effect_v1`](https://wayland.app/protocols/ext-background-effect-v1)
protocol, instead of only the KDE one (`org_kde_kwin_blur`).

## Why this branch exists instead of moving `gpui-pre` forward

`gpui-pre` 0.3.8 - which the `main` branch of this repository forks - cannot be
used by applications built on `gpui-component` 0.6.x. Those releases are
source-incompatible with 0.3.8 (`gpui-base` 0.6.x fails to compile against it:
`LineWrapper::wrap_line` gained an `IndentAdjustment` argument), and the only
`gpui-component` releases that pin `gpui-pre == 0.3.8` are 0.7.1/0.7.2. Those in
turn draw their own window border and shadow unconditionally - `Root::bordered(false)`
is gone and the decoration cannot be switched off - which collides with an
application that draws its own CSD frame (`_GTK_FRAME_EXTENTS`, its own shadow
margin and resize bands).

Backdrop blur is a Wayland-backend feature only. Forking just this crate on the
0.3.3 base gives those applications real blur on GNOME 51 without moving the rest
of their dependency stack - in particular without touching `gpui-pre`,
`gpui-pre-platform` or `gpui-component`.

No dependency changed: 0.3.3 already requires `wayland-protocols` 0.32.9, the
same version the `main` branch uses, and that release ships the
`ext::background_effect` module.

## The change

`src/linux/wayland/client.rs`

- bind `ext_background_effect_manager_v1` into `Globals`;
- track its `Capabilities` event in `background_effect_blur_supported`
  (`Rc<Cell<bool>>`) and, when the answer changes, call
  `background_effect_capabilities_changed()` on every open window - the capability
  arrives *after* the surface exists, so the first
  `set_background_appearance` may have run before it was known;
- add `dispatch`/`delegate_noop!` wiring for both new objects;
- add `WaylandClientStatePtr::try_get_client()`, so the event handler can bail out
  when the client was dropped (headless mode).

`src/linux/wayland/window.rs`

- keep an `ext_background_effect_surface_v1` per window and destroy it with the
  other per-window objects;
- `update_window`: when the window wants `Blurred` **and** the compositor
  advertised `Capability::Blur`, drive the standard protocol (dropping any KDE
  blur first, since the two would otherwise stack); otherwise keep the previous
  KDE path unchanged;
- the blur rectangle is the **whole surface**, built from
  `window_bounds.size` at origin `(0, 0)`. `set_blur_region` takes
  surface-local coordinates, whereas `window_bounds` carries the window's
  position on screen — reusing the opaque-area region there offsets the
  rectangle by the window's screen position and leaves part of the window
  unblurred.

Deliberately **not** included: `Window::set_background_blur_regions()` and its
`blur_regions` field. That API was added by the `main` branch's patch together
with a matching trait method in its own `gpui-pre`; the applications this branch
serves blur the whole client area, and they run against the stock `gpui-pre`
0.3.3. The `blur_regions == None` branch below is exactly what `main` falls back
to.

## Use

```toml
[patch.crates-io]
gpui-pre-linux = { git = "https://github.com/Aorus22/gpui-wayland-patches", rev = "<this branch's commit>" }
```

Keep `gpui-pre`, `gpui-pre-platform` and `gpui-component` at their 0.3.3 / 0.6.x
versions - this branch does not require any of them to move.
