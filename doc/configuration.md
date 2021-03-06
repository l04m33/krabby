# Configuration

## Files

- [`~/.config/krabby`](/share/krabby)
  - [`manifest.json`](/share/krabby/manifest.json): extension’s entry point.
  - [`config.js`](/share/krabby/config.js): contains the user configuration.
  - [`fetch`](/share/krabby/fetch): shell script to fetch plugins.
  - [`Makefile`](/share/krabby/Makefile): contains commands to build and update Krabby.
  - `packages`: contains files used by Krabby: [Modal], [Mouse Selection], [Prompt], [Hint], [Mark], [Selection], [Mouse], [Clipboard], [Scroll], [Player], [icons][Krabby icon] and [`krabby`](/src/krabby).
  - `extensions`: contains extensions used by Krabby: [Commands], [Shell], [Editor] and [dmenu].  [Krabby] repository can be found here, to update the extension when you run `make update`.
- [`~/.local/bin`](/bin)
  - [`plumb`](/bin/plumb)

Krabby’s default configuration is located in [`~/.config/krabby/packages/krabby`](/src/krabby).

## Mapping

Creating and removing mappings boils down to the following commands:

``` javascript
krabby.modes.modal.map(context, keys, command, description, label)
```

``` javascript
krabby.modes.modal.unmap(context, keys)
```

The **context** dictates in what context the mapping will be available:
**Command**, **Text**, **Link**, **Image**, **Video**, **Document** or **Page**.

The **keys** represent a chord – a key sequence in which the keys are pressed at
the same time.  They are composed of a single [key code][KeyboardEvent.code] and
optional [modifiers].  For special keys, the list of key values can be found
[here][Key Values].

The **command** is the function to evaluate.  The function takes exactly one
argument, the [`keydown`] event that triggered the command.  For most commands,
this argument can be ignored.  The command can also be an instance of **Modal**
to facilitate command chaining.

**Example** – Example where `event` parameter can be useful for smooth scrolling:

`~/.config/krabby/config.js`

``` javascript
const { modes, scroll } = krabby
const { modal } = modes

modal.map('Command', ['KeyJ'], ({ repeat }) => scroll.down(repeat), 'Scroll down', 'Scroll')
modal.map('Command', ['KeyK'], ({ repeat }) => scroll.up(repeat), 'Scroll up', 'Scroll')
```

The **description** is the description of the command.

The **label** is the label of the command.

## Hint appearance

`~/.config/krabby/config.js`

``` javascript
const { env } = krabby

env.HINT_STYLE = {
  textColor: 'royalblue',
  activeCharacterTextColor: 'lightsteelblue',
  backgroundColorStart: 'white',
  backgroundColorEnd: 'ghostwhite',
  borderColor: 'ghostwhite'
}
```

See [Hint – Appearance] for more information.

## External editor

### [Alacritty]

Override individual configuration options with a temporary config file.

`~/.config/krabby/config.js`

``` javascript
const { env } = krabby

env.EDITOR = `
  # Environment variables
  XDG_CONFIG_HOME=\${XDG_CONFIG_HOME:-~/.config}
  CONFIG=$XDG_CONFIG_HOME/alacritty/alacritty.yml
  # Create a temporary config file
  config=$(mktemp)
  trap 'rm -f "$config"' EXIT
  # Populate configuration if available
  if test -f "$CONFIG"; then
    cp "$CONFIG" "$config"
  fi
  # Additional settings
  cat <<'EOF' >> "$config"
background_opacity: 0.75
EOF
  alacritty --config-file "$config" --class alacritty-float --command kak "$1" -e "select $2.$3,$4.$5"
`
```

### [kitty]

`~/.config/krabby/config.js`

``` javascript
const { env } = krabby

env.EDITOR = 'kitty --class kitty-float --override background_opacity=0.75 kak "$1" -e "select $2.$3,$4.$5"'
```

## mpv

`~/.config/krabby/config.js`

``` javascript
const { env } = krabby

env.MPV_CONFIG = ['-no-config', '-no-terminal']
```

## HTML filter

`~/.config/krabby/config.js`

``` javascript
const { env } = krabby

env.HTML_FILTER = ['pandoc', '--from', 'html', '--to', 'asciidoc']
```

## Examples

### [Read Berserk] with [mpv]

`~/.config/krabby/config.js`

``` javascript
const { extensions, modes } = krabby
const { shell } = extensions
const { modal } = modes

modal.filter('Read Berserk', () => location.hostname === 'readberserk.com', 'Command')
modal.filter('Read Berserk · Chapter', () => location.pathname.startsWith('/chapter'), 'Read Berserk')
modal.enable('Read Berserk · Chapter', 'Read Berserk', ...modal.context.filters)

modal.map('Read Berserk · Chapter', ['KeyM'], () => shell.send('mpv', ...Array.from(document.querySelectorAll('.pages__img'), (image) => image.src)), 'Read Berserk with mpv', 'Read Berserk · Chapter')
```

[Krabby]: https://github.com/alexherbo2/krabby
[Krabby icon]: https://iconfinder.com/icons/877852/kanto_krabby_pokemon_water_icon

[Modal]: https://github.com/alexherbo2/modal.js
[Mouse Selection]: https://simonwep.github.io/selection/
[Prompt]: https://github.com/alexherbo2/prompt.js
[Hint]: https://github.com/alexherbo2/hint.js
[Hint – Appearance]: https://github.com/alexherbo2/hint.js#appearance
[Mark]: https://github.com/alexherbo2/mark.js
[Selection]: https://github.com/alexherbo2/selection.js
[Mouse]: https://github.com/alexherbo2/mouse.js
[Clipboard]: https://github.com/alexherbo2/clipboard.js
[Scroll]: https://github.com/alexherbo2/scroll.js
[Player]: https://github.com/alexherbo2/player.js

[Commands]: https://github.com/alexherbo2/chrome-commands
[Shell]: https://github.com/alexherbo2/chrome-shell
[Editor]: https://github.com/alexherbo2/chrome-editor
[dmenu]: https://github.com/alexherbo2/chrome-dmenu

[mpv]: https://mpv.io

[Read Berserk]: https://readberserk.com

[`keydown`]: https://developer.mozilla.org/en-US/docs/Web/API/Document/keydown_event
[KeyboardEvent.code]: https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code
[Key Values]: https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values
[Modifiers]: https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values#Modifier_keys

[Alacritty]: https://github.com/jwilm/alacritty
[kitty]: https://sw.kovidgoyal.net/kitty/
