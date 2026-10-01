# window-layout

`window-layout` is a Perl command-line tool designed to save, list, and restore
the geometry, position, and command of your Linux desktop windows.

- On X11 sessions, it uses `wmctrl` and `xdotool`.
- On KDE Wayland sessions, it uses `kdotool`.

This allows managing window states across sessions on both X11 and KDE Wayland.

## Dependencies

Before using `window-layout`, ensure you have the following system dependencies installed:

- `perl`
- `wmctrl`, `xdotool`, and `xprop` (for X11 sessions)
- `kdotool` (for KDE Wayland sessions)

## Usage

```bash
window-layout [options] <list|save|restore>
```

### Commands

- **`list`**: Prints out the current windows to standard output in JSON format.
- **`save`**: Saves the current window layout to the configuration file.
- **`restore`**: Reads the configuration file and restores the saved window layout. It will attempt to match existing windows or spawn missing ones as needed.

### Options

| Option            | Description                                                                 |
| ----------------- | --------------------------------------------------------------------------- |
| `--append`        | Append to the layout when saving instead of overwriting.                    |
| `-n`, `--dry-run` | Simulate the restore process without actually moving windows.               |
| `--screen ID`     | Filter operations by screen ID. (for multiple monitors)                     |
| `--desktop ID`    | Filter operations by desktop number (starting at 1).                        |
| `--filter CMD`    | Filter operations by command name (e.g., `konsole`).                        |
| `--config FILE`   | Specify an alternative JSON config file. (Default: `~/.window-layout.json`) |
| `-v`, `--verbose` | Print debug output during execution.                                        |
| `-h`, `--help`    | Print the help message.                                                     |

## Configuration Format

The tool uses a JSON array configuration file located by default at `~/.window-layout.json`.

Example configuration:

```json
[
  {
    "screen":    0,
    "desktop":   1,
    "command":  "konsole",
    "x":         2,
    "y":         28,
    "width":     1232,
    "height":    425
  },
  {
    "screen":    0,
    "desktop":   2,
    "command":  "firefox",
    "x":         0,
    "y":         0,
    "width":     1920,
    "height":    1043,
    "autostart": 0
  },
  {
    "screen":    0,
    "desktop":  "any",
    "command":  "remmina",
    "x":         0,
    "y":         0,
    "width":     800,
    "height":    600
  }
]
```

### JSON Attributes

| Attribute   | Description |
| ----------- | ----------- |
| `command`   | The command or executable name associated with the window. |
| `search`    | (Optional) A regular expression to match against the full command line of the window's PID. If set, this is used instead of `command` to find the existing window. |
| `desktop`   | Desktop number (starts at 1), `"all"` (sticky), or `"any"` (leave the desktop unchanged). |
| `screen`    | The monitor/screen ID the window is located on. |
| `width`     | (Optional) The width of the window in pixels. If omitted, the current width is kept. |
| `height`    | (Optional) The height of the window in pixels. If omitted, the current height is kept. |
| `x`         | The horizontal X coordinate of the window position. |
| `y`         | The vertical Y coordinate of the window position. |
| `autostart` | (Optional) Boolean indicating whether to start the command if the window is missing. Defaults to `true`. Set to `false` to skip starting the process automatically. |

## How It Works

When restoring windows, `window-layout` matches saved commands against existing
windows. If a window doesn't exist, it intelligently forks the new process and
actively traces the process tree to identify the newly created window
(even if the application forks to the background, like `konsole` or `gnome-terminal`),
correctly placing it according to your saved geometry.
