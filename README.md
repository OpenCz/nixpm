# nixpm 

apt-like CLI to manage NixOS packages from the terminal — no more manual editing of `config.nix`.

```bash
nixpm install neovim ripgrep   # like apt install
nixpm remove  htop             # like apt remove
nixpm search  spotify          # search nixpkgs
nixpm list                     # list installed packages
```

---

## Why

NixOS is declarative, which is great — but editing `config.nix` by hand every time you want a new package gets old fast. `nixpm` wraps that workflow into a single command, handles the rebuild, and keeps a backup in case something goes wrong.

---

## Install

```bash
git clone https://github.com/<you>/nixpm
cd nixpm
sudo cp nix-pkg.py /usr/local/bin/nixpm
sudo chmod +x /usr/local/bin/nixpm
```

> Requires Python 3.10+ (already on your system if you're on NixOS).

---

## Usage

### Install a package
```bash
nixpm install <package>
nixpm install neovim ripgrep bat   # multiple at once
```
Adds the package(s) to `environment.systemPackages` in `/etc/nixos/config.nix` and runs `nixos-rebuild switch`.

### Remove a package
```bash
nixpm remove <package>
nixpm rm neovim ripgrep
```

### List installed packages
```bash
nixpm list
nixpm ls
```

### Search nixpkgs
```bash
nixpm search <term>
nixpm s firefox
```

### Skip the rebuild
```bash
nixpm install neovim --no-rebuild
```
Edits the config without triggering a rebuild. Useful when batching multiple changes.

---

## How it works

1. Parses the `environment.systemPackages` block in your `config.nix`
2. Creates a backup at `config.nix.bak` before any change
3. Adds / removes the requested packages
4. Asks for confirmation
5. Runs `sudo nixos-rebuild switch`

---

## Aliases

| Short | Full command |
|-------|-------------|
| `i`   | `install`   |
| `r`, `rm` | `remove` |
| `ls`, `l` | `list`  |
| `s`   | `search`    |

---

## Requirements

- NixOS with a `config.nix` at `/etc/nixos/config.nix`
- Python 3.10+
- `sudo` access
