# Anyrun (fuzzel-style fork)

A wayland native krunner-like runner, made with customizability in mind.

This is a **personal fork** of [anyrun-org/anyrun](https://github.com/anyrun-org/anyrun)
that tweaks it to behave more like [fuzzel](https://codeberg.org/dnkl/fuzzel):
the application menu expands by default on empty input and is fully navigable
with the keyboard.

<img width="950" height="702" alt="image" src="https://github.com/user-attachments/assets/0a53b435-58f5-4a7c-90f7-b3f39266f2f4" />

## What this fork changes

- **fuzzel-style empty input**: when the input is empty, all applications are
  listed immediately (sorted by name) instead of showing nothing.
- **Scrollable results list**: a new `list_height` config option caps the result
  list height and lets it scroll internally, instead of the window growing
  unbounded.
- **Reliable keyboard navigation**: arrow keys / Tab move the selection and the
  list scrolls the active row into view. Key repeat now works when holding an
  arrow key (previously it only moved once).
- **Toggle via the same keybinding**: launching `anyrun` while it is already
  visible now closes it, so a single key (e.g. `Mod+D`) both opens and closes
  the launcher. Requires running `anyrun daemon`.
- **Click-outside-to-close**: `close_on_click` keeps its meaning, but the window
  stays visually small (no full-screen overlay) thanks to a transparent capture
  layer.
- **Selection styling**: removed the fade-in animation on the selected row so it
  doesn't flicker during fast keyboard navigation.

## Environment

Anyrun runs on any Wayland compositor that implements the
[wlr-layer-shell](https://gitlab.freedesktop.org/wlroots/wlr-protocols/-/blob/master/unstable/wlr-layer-shell-unstable-v1.xml)
protocol (niri, Hyprland, Sway, river, etc.). The changes in this fork are
pure GTK4 / layer-shell and have **no compositor-specific dependency** — they
behave identically everywhere.

Development and testing were done on:

- OS: Debian testing
- Compositor: niri (Wayland)
- GTK4 + gtk4-layer-shell
- Rust (via [rustup](https://rustup.rs))
- Depends on [anyrun-provider](https://github.com/anyrun-org/anyrun-provider)
  for search results (must be in `$PATH`, or set via the `provider` config
  option).

> [!NOTE]
> If you use Nvidia and Anyrun refuses to close for you, you need to set `GSK_RENDERER=ngl` for Anyrun.
> As in, running it with `GSK_RENDERER=ngl anyrun`. This is a [known issue](https://forums.developer.nvidia.com/t/580-65-06-gtk-4-apps-hang-when-attempting-to-exit-close/341308/6)
> and is quite driver version dependent.

## Features

- Style customizability with GTK4 CSS
  - More info in [Styling](#styling)
- Can do basically anything
  - As long as it can work with input and selection
  - Hence the name Anyrun
- Easy to make plugins
  - You only need 4 functions!
  - See [Rink](plugins/rink) for a simple example. More info in the
    documentation of the [anyrun-plugin](anyrun-plugin) crate.
- Responsive
  - Asynchronous running of plugin functions
- Wayland native
  - GTK4 layer shell for overlaying the window
  - data-control for managing the clipboard

## Usage

### Dependencies

Anyrun mainly depends various GTK4 libraries, and rust of course for building the
project. Rust you can get with [rustup](https://rustup.rs). The rest are
statically linked in the binary. Here are the libraries you need to have to
build & run it:

- `gtk4-layer-shell (libgtk4-layer-shell)`
- `gtk4 (libgtk-4 libgdk-4)`
- `pango (libpango-1.0)`
- `cairo (libcairo libcairo-gobject)`
- `gdk-pixbuf2 (libgdk_pixbuf-2.0)`
- `glib2 (libgobject-2.0 libgio-2.0 libglib-2.0)`

> [!NOTE]
> Since 25.12.0, Anyrun also depends on [anyrun-provider](https://github.com/anyrun-org/anyrun-provider)
> to provide search results. Make sure it is installed as well for Anyrun to function. If you don't want to install
> it into your `$PATH`, you can set the path to it via the `provider` config option.

## Installation

This fork is not published to any distribution's package repositories or
nixpkgs. Build it from source.

### Manual installation

Make sure all of the dependencies from [Dependencies](#dependencies) are
installed, and then run the following commands in order:

```bash
# Clone this fork and move to the cloned location
git clone https://github.com/Kzbon/anyrun anyrun && cd anyrun

# Build all packages, and install the Anyrun binary
cargo build --release
cargo install --path anyrun/

# Create the config directory and the plugins subdirectory
mkdir -p ~/.config/anyrun/plugins

# Copy all of the built plugins to the correct directory
cp target/release/*.so ~/.config/anyrun/plugins

# Copy the default config file
cp examples/config.ron ~/.config/anyrun/config.ron
```

### Running as a daemon (recommended)

The toggle / click-outside-to-close behaviour depends on the daemon being
running. Start it alongside your compositor:

**niri** (`~/.config/niri/config.kdl`):
```kdl
spawn-at-startup "anyrun" "daemon"
```

**Hyprland** (`~/.config/hypr/hyprland.conf`):
```
exec-once = anyrun daemon
```

**Sway** (`~/.config/sway/config`):
```
exec anyrun daemon
```

Then bind a single key (e.g. `Mod+D`) to plain `anyrun` — it will both open
and close the launcher.

## Plugins

Anyrun requires plugins to function, as they provide the results for input. The
library name after the plugin name is what you use for including the plugin
inside the configuration. The list of plugins in this repository is as follows:

- [Applications](plugins/applications/README.md) `libapplications.so`
  - Search and run system & user specific desktop entries.
- [Symbols](plugins/symbols/README.md) `libsymbols.so`
  - Search unicode symbols.
- [Rink](plugins/rink/README.md) `librink.so`
  - Calculator & unit conversion.
- [Shell](plugins/shell/README.md) `libshell.so`
  - Run shell commands.
- [Translate](plugins/translate/README.md) `libtranslate.so`
  - Quickly translate text.
- [Kidex](plugins/kidex/README.md) `libkidex.so`
  - File search provided by [Kidex](https://github.com/Kirottu/kidex).
- [Randr](plugins/randr/README.md) `librandr.so`
  - Rotate and resize; quickly change monitor configurations on the fly.
  - TODO: Only supports Hyprland, needs support for other compositors.
- [Stdin](plugins/stdin/README.md) `libstdin.so`
  - Turn Anyrun into a dmenu-like fuzzy selector.
  - Should generally be used exclusively with the `--plugins` argument.
- [Dictionary](plugins/dictionary/README.md) `libdictionary.so`
  - Look up definitions for words
- [Websearch](plugins/websearch/README.md) `libwebsearch.so`
  - Search the web with configurable engines: Google, Ecosia, Bing, DuckDuckGo.
- [Nix-run](plugins/nix-run/README.md) `libnix_run.so`
  - `nix run` graphical applications straight from Anyrun.
- [niri-focus](plugins/niri-focus/README.md) `libniri_focus.so`
  - Focus active windows with fuzzy search on niri.
- [Actions](plugins/actions/README.md) `libactions.so`
  - Run power management actions or any custom commands.

## Configuration

The default configuration directory is `$HOME/.config/anyrun` the structure of
the config directory is as follows and should be respected by plugins:

```
- anyrun/
  - plugins/
    - <plugin dynamic libraries>
  - config.ron
  - style.css
  - <any plugin specific config files>
```

The [default config file](examples/config.ron) contains the default values, and
annotates all configuration options with comments on what they are and how to
use them.

## Styling

Anyrun supports [GTK4 CSS](https://docs.gtk.org/gtk4/css-properties.html) styling.
The style classes and widgets that use them are as follows:

- No class, unique widget:
  - `GtkText`: The main entry box
  - `GtkWindow`: The main window
- `.main`:
  - `GtkBox`: The box that contains everything else
- `.matches`:
  - `GtkBox`: The box that contains all the results & info boxes
- `.plugin`:
  - `GtkBox`: The main plugin box
  - `.info`:
    - `GtkBox`: Box containing the plugin info
    - `GtkImage`: Icon of the plugin
    - `GtkLabel`: Name of the plugin
- `.match`:
  - `GtkBox`: The box containing all contents of a match
  - `GtkImage`: The icon (if present)
  - `.title`:
    - `GtkLabel`: The title
  - `.description`
    - `GtkLabel`: The description (if present)

Refer to the [default style](anyrun/res/style.css) for an example, and use `GTK_DEBUG=interactive anyrun`
to edit styles live.

## Arguments

The custom arguments for anyrun are as follows:

- `--config-dir`, `-c`: Override the configuration directory

The rest of the arguments are automatically generated based on the config, and
can be used to override configuration parameters. For example if you want to
temporarily only run the Applications and Symbols plugins on the top side of the
screen, you would run
`anyrun --plugins libapplications.so --plugins libsymbols.so --position top`.

## Plugin development

The plugin API is intentionally very simple to use. This is all you need for a
plugin:

`Cargo.toml`:

```toml
#[package] omitted
[lib]
crate-type = ["cdylib"] # Required to build a dynamic library that can be loaded by anyrun

[dependencies]
anyrun-plugin = { git = "https://github.com/Kzbon/anyrun" }
abi_stable = "0.11.1"
# Any other dependencies you may have
```

`lib.rs`:

```rs
use abi_stable::std_types::{RString, RVec, ROption};
use anyrun_plugin::*;

#[init]
fn init(config_dir: RString) {
  // Your initialization code. This is run in another thread.
  // The return type is the data you want to share between functions
}

#[info]
fn info() -> PluginInfo {
  PluginInfo {
    name: "Demo".into(),
    icon: "help-about".into(), // Icon from the icon theme
  }
}

#[get_matches]
fn get_matches(input: RString) -> RVec<Match> {
  // The logic to get matches from the input text in the `input` argument.
  // The `data` is a mutable reference to the shared data type later specified.
  vec![Match {
    title: "Test match".into(),
    icon: ROption::RSome("help-about".into()),
    use_pango: false,
    description: ROption::RSome("Test match for the plugin API demo".into()),
    id: ROption::RNone, // The ID can be used for identifying the match later, is not required
  }].into()
}

#[handler]
fn handler(selection: Match) -> HandleResult {
  // Handle the selected match and return how anyrun should proceed
  HandleResult::Close
}
```

And that's it! That's all of the API needed to make runners. Refer to the
plugins in the [plugins](plugins) folder for more examples.

## Acknowledgements

This fork is built on top of [anyrun-org/anyrun](https://github.com/anyrun-org/anyrun).
All credit for the original runner, plugin system, and the vast majority of the
codebase goes to the upstream authors and contributors. This fork only adds a
small set of personal UX tweaks on top of their work.
