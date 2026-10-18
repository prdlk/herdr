# herdr

My config for [herdr](https://herdr.dev) ([herdrdev/herdr](https://github.com/herdrdev/herdr)), a terminal multiplexer built around coding agents. It lives at `~/.config/herdr`.

The setup in short: tmux-style `ctrl+b` keys, no pane borders on a translucent [cyberdream](https://github.com/scottmckendry/cyberdream.nvim) theme, and `ctrl+h/j/k/l` moves across herdr and nvim splits alike. Plugins add an nvim sidebar, a command palette, a browser pane, and a diff reviewer.

## Install

You need herdr itself first ([install docs](https://herdr.dev)). Then:

```sh
git clone https://github.com/prdlk/herdr ~/.config/herdr
```

This is a plain config repo. In my own setup [chezmoi](https://www.chezmoi.io) clones it as a `git-repo` external and fast-forwards it once a week.

Plugin checkouts aren't committed (`plugins/github/` is gitignored), so a fresh machine has to install them again. These are the revisions recorded in `plugins.json`:

```sh
herdr plugin install ChmaraX/herdr-nvim --ref 40aadeab3cef3702ef5e05069181c7168084794f
herdr plugin install qu8n/herdr-automatic-rename
herdr plugin install ogulcancelik/herdr-browser --ref be6888b71cf4eb5939ee79a746bd1a1c22ade046
herdr plugin install alvinunreal/oh-my-opencode-slim --ref 220eb2be677cb739f795b97a62909110e04ebfc5
herdr plugin install persiyanov/herdr-reviewr --ref 792b4b31475ddbf263ee4d421a984e7418031506
herdr plugin install rjyo/herdr-window-title-sync --ref b07f1140b7308d66487b2f4be546c0c7db065569
herdr plugin install crafts69guy/herdr-switchboard --ref b68d053b6fe54b8e0bcb46a90ed36135da57b4d3
herdr plugin install paulbkim-dev/vim-herdr-navigation
```

Plugins run as your user with no sandbox. `herdr plugin install` previews what it will run, so read that before you confirm. `herdr plugin update` moves the GitHub plugins to their latest versions.

`plugins.json` is herdr's own plugin registry; it records each install's source, pinned commit, and absolute checkout path (under `/home/prad`). Leave it to herdr rather than editing by hand.

## Running it in kitty

herdr runs in any terminal, but I recommend [kitty](https://sw.kovidgoyal.net/kitty/) for this config:

- **Images in panes.** herdr renders the [kitty graphics protocol](https://sw.kovidgoyal.net/kitty/graphics-protocol/) in compatible outer terminals (on here via `kitty_graphics = true`), so image previews from tools like yazi work inside panes.
- **Translucency.** `panel_bg = "reset"` lets kitty's `background_opacity` and your compositor's blur show through herdr.
- **Chords as keys.** kitty's `send_text` maps one shortcut to the `ctrl+b` prefix byte plus a key, so herdr actions get single-chord shortcuts.

### Install kitty and the font

```sh
# Arch
pacman -S kitty ttf-victor-mono-nerd
# macOS
brew install --cask kitty font-victor-mono-nerd-font
```

The config uses VictorMono Nerd Font Mono. The plugins and the sidebar use Nerd Font icons, so pick some Nerd Font if you swap it out.

### A kitty config that hosts herdr

herdr gets its own kitty config instead of sharing your everyday one. kitty becomes a plain window: no tab bar, and every window opens straight into herdr. Mine is [`herdr.conf` in prdlk/kitty](https://github.com/prdlk/kitty/blob/main/herdr.conf). Copy it to `~/.config/kitty/herdr.conf`, or start from these essentials:

```conf
# ~/.config/kitty/herdr.conf
font_family      family="VictorMono Nerd Font Mono"
background_opacity 0.9
hide_window_decorations yes
window_padding_width 8
allow_remote_control socket-only
confirm_os_window_close 0

# herdr draws its own sidebar and tab strip
tab_bar_style hidden

# Launch or attach the persistent herdr session; closing the window only detaches
shell herdr

# Leave ctrl+h/j/k/l unmapped: vim-herdr-navigation handles them inside herdr
```

Run it with:

```sh
kitty --class=kitty-herdr --config ~/.config/kitty/herdr.conf
```

`--class` sets the window class / Wayland app-id (`kitty-herdr`), so compositor rules can target herdr windows. It has to be on the command line: kitty ignores `class` in a config file.

Bind that command in your window manager or desktop launcher. It has to start a separate kitty process, because kitty's `launch` action can't take `--config`. From inside another kitty, map it as a background launch. My [tmux host config](https://github.com/prdlk/kitty/blob/main/tmux.conf) uses `ctrl+shift+a`:

```conf
map ctrl+shift+a launch --type=background --cwd=current kitty --class=kitty-herdr --config ~/.config/kitty/herdr.conf
```

The last line of my `herdr.conf` includes the active [Omarchy](https://omarchy.org) theme (`include ~/.local/state/omarchy/current/theme/kitty.conf`). On other systems, replace it with your own color scheme, such as kitty's cyberdream theme.

### Chord shortcuts

`herdr.conf` turns kitty chords into herdr prefix sequences with `send_text all \x02<key>`. `\x02` is `ctrl+b`, and an uppercase letter is the shift variant (`\x02N` is `prefix shift+n`).

| kitty key | Sends | herdr action |
| --- | --- | --- |
| `super+t`, `ctrl+shift+t` | `prefix c` | New tab |
| `ctrl+shift+h` / `ctrl+shift+l` | `prefix p` / `prefix n` | Previous / next tab |
| `ctrl+shift+j` / `ctrl+shift+k` | `prefix j` / `prefix k` | Next / previous workspace |
| `ctrl+shift+n` | `prefix shift+n` | New workspace |
| `super+n` | `prefix shift+g` | New git worktree |
| `ctrl+shift+w` | `prefix shift+w` | Rename workspace |
| `ctrl+shift+r` | `prefix shift+t` | Rename tab |
| `ctrl+shift+q` | `prefix x` | Close pane |
| `super+z` | `prefix z` | Zoom pane |
| `ctrl+tab` / `ctrl+shift+tab` | `prefix .` / `prefix ,` | Next / previous agent |
| `ctrl+shift+e` | `prefix e` | Toggle nvim sidebar |
| `ctrl+shift+o` | `prefix o` | Open file from agent output |
| `ctrl+space` | `prefix t` | Switchboard menu |
| `ctrl+shift+f` | `prefix shift+k` | Switchboard projects |
| `ctrl+shift+g` | `prefix shift+l` | Switchboard git |
| `ctrl+shift+p` | `prefix shift+p` | Switchboard commands |
| `ctrl+shift+,` | `prefix shift+s` | herdr settings |

It also maps `super+c` / `super+v` to copy / paste, and `ctrl+plus` / `ctrl+minus` / `ctrl+0` to font size. `ctrl+shift+alt+,` reloads the kitty config, since herdr settings took `ctrl+shift+,`.

## Files

| Path | What it is |
| --- | --- |
| `config.toml` | Main config: UI, theme, keys, plugin keybindings |
| `plugins.json` | herdr's registry of installed plugins (maintained by herdr) |
| `.plugins.lock` | Plugin lock file, committed empty |
| `plugins/config/<plugin>/` | Per-plugin config directories (only `.keep` placeholders right now) |
| `kitty.conf` | Old kitty config from before herdr. Not used; the live kitty host config is `herdr.conf` in [prdlk/kitty](https://github.com/prdlk/kitty) |
| `config.toml.bak` | Older snapshot of `config.toml` |

## Look

- **No borders.** `pane_borders` and `pane_outer_borders` are off. Gaps and the terminal background tell panes apart, and the compositor draws the window frame.
- **Translucent.** `theme.custom.panel_bg = "reset"` keeps the terminal's own background, so kitty's opacity and the compositor blur show through.
- **cyberdream colors.** The sidebar uses `#16181a`, the active row `#1e2124`, and selection `#3c4048`. The active tab is a `#243348` plate with a `#5ea1ff` label, matching kitty's tab bar.
- **Agent states.** Green `#5eff6c`, yellow `#f1ff5e`, red `#ff6e5e`, peach `#ffbd5e`, mauve `#bd5eff`. `status_indicators = "symbols"`.
- **Other settings.** Toasts are sent as system notifications. Kitty graphics are on, onboarding is off, and new tabs don't ask for a name.

## Keys

The prefix is herdr's default, `ctrl+b`.

| Key | Action |
| --- | --- |
| `prefix %` / `prefix "` | Split vertical / horizontal |
| `prefix d` | Detach |
| `prefix s` | Workspace picker (`j`/`k` move inside it) |
| `prefix shift+s` | Settings |
| `prefix j` / `prefix k` | Next / previous workspace |
| `prefix .` / `prefix ,` | Next / previous agent |
| `prefix ←↓↑→` | Focus pane |
| `ctrl+h/j/k/l` | Move between panes; passes through to nvim splits ([vim-herdr-navigation](https://github.com/paulbkim-dev/vim-herdr-navigation)) |
| `prefix e` | Toggle the nvim sidebar |
| `prefix o` | Open a file mentioned in agent output |
| `prefix t` | Switchboard menu |
| `prefix shift+k` | Switchboard projects |
| `prefix shift+p` | Switchboard commands |
| `prefix shift+l` | Switchboard git |
| `prefix y` | Browser in a right split |
| `prefix shift+o` | Browser overlay |

The `ctrl+b` prefix is the same one tmux uses, so terminal chords that send the `\x02` prefix byte work with either multiplexer. See [Chord shortcuts](#chord-shortcuts) for the kitty mappings.

## Plugins

| Plugin | Repo | What it does | Needs |
| --- | --- | --- | --- |
| `chmarax.herdr-nvim` | [ChmaraX/herdr-nvim](https://github.com/ChmaraX/herdr-nvim) | Full-height nvim sidebar with agent annotations; opens files clicked or picked from agent output | `nvim` |
| `herdr-automatic-rename` | [qu8n/herdr-automatic-rename](https://github.com/qu8n/herdr-automatic-rename) | Names tabs after the foreground program and puts the 1–9 jump number in front of workspaces, tabs, and agents | `bash` |
| `official.browser` | [ogulcancelik/herdr-browser](https://github.com/ogulcancelik/herdr-browser) | Opens real browser pages in a pane; localhost links open there | `bun` |
| `oh-my-opencode-slim` | [alvinunreal/oh-my-opencode-slim](https://github.com/alvinunreal/oh-my-opencode-slim) | OpenCode agent orchestration with a herdr adapter | OpenCode |
| `persiyanov.reviewr` | [persiyanov/herdr-reviewr](https://github.com/persiyanov/herdr-reviewr) | Shows agent-written diffs next to the chat and adds your line comments to the agent's input | `bash` |
| `rjyo.window-title-sync` | [rjyo/herdr-window-title-sync](https://github.com/rjyo/herdr-window-title-sync) | Keeps the outer terminal title in sync with the focused workspace, tab, and agent | `bun` |
| `switchboard` | [crafts69guy/herdr-switchboard](https://github.com/crafts69guy/herdr-switchboard) | Command palette: projects, commands, ports, git, clone, settings | `bash`; `ghq`, `lazygit`, `tuicr` for some pickers |
| `vim-herdr-navigation` | [paulbkim-dev/vim-herdr-navigation](https://github.com/paulbkim-dev/vim-herdr-navigation) | Vim-aware `ctrl+h/j/k/l` across herdr panes and nvim splits | `bash` |
| `io.github.salemsayed.omaherd` | linked locally | Tells the [Omarchy](https://omarchy.org) bar when an agent changes state | Omarchy |

`omaherd` isn't installed from GitHub. It's linked from `~/.config/omarchy/plugins/` with `herdr plugin link`, so skip it if you don't run Omarchy.

### Plugin config outside this repo

- **herdr-nvim** reads `~/.config/herdr-nvim/config.toml`. I set it to open the sidebar on the left:

  ```toml
  [sidebar]
  position = "left"
  ```

- **herdr-automatic-rename** ships a shell hook for renaming tabs. In fish:

  ```fish
  for hook in ~/.config/herdr/plugins/github/herdr-automatic-rename-*/shell/hook.fish
      test -r $hook; and source $hook; and break
  end
  ```
