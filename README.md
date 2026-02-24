# isim

A CLI tool for quickly launching and managing iOS Simulators.

## Features

- Launch simulators by name, OS version, or UDID
- Set and launch a default simulator with a single command
- List all available simulators in a formatted table
- Filter simulators by device type, OS version, etc.

## Requirements

- macOS with Xcode installed
- [uv](https://docs.astral.sh/uv/)

## Install

```sh
# Clone the repo
git clone <repo-url> ~/projects/isim

# Symlink into your PATH
ln -s ~/projects/isim/isim ~/.local/bin/isim
```

## Usage

```
isim                      Launch the default simulator
isim list [filter]        List available simulators
isim launch <query>       Launch by name, OS version, or UDID
isim default              Show current default
isim default <udid>       Set default simulator
isim help                 Show this help
```

## Examples

```sh
# List all available simulators
isim list

# Filter to iPhones only
isim list iphone

# Filter by OS version
isim list 'iOS 17'

# Launch a specific device
isim launch 'iPhone 16 Pro'

# Set a default and launch it
isim default <udid>
isim
```

The default simulator UDID is stored in `~/.config/isim/default`.
