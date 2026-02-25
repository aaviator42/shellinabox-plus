# shellinabox-plus  
*Mobile UI and dark mode for [Shell In A Box](https://github.com/shellinabox/shellinabox)*  

`2026-02-25` : `v0.1`  | License: `AGPLv3`  

Two drop-in additions to a stock shellinabox install. No patches, no recompile — loaded entirely via daemon flags.

-------

## Files

| File | Purpose |
|---|---|
| `dark.css` | Dark terminal theme: white text on black, corrected ANSI colours |
| `mobile.html` | Mobile-friendly wrapper UI served by shellinaboxd at `/mobile` |

---

## dark.css

Overrides the default shellinabox stylesheet. Sets the scrollable terminal area to `#000` background / `#fff` foreground and corrects the ANSI default-colour classes so colour output renders correctly on the dark background.

---

## mobile.html

Served directly by the shellinaboxd HTTP server as a static file — no web server proxy needed. Opens at `http://<host>:<port>/mobile`.

Embeds the shellinabox terminal in a same-origin `<iframe>` (same port), giving direct access to the `shellinabox.keysPressed()` JS API for sending input without any cross-origin restrictions.

### Layout

```
┌─────────────────────────────────────────────────┐
│                                                 │
│            shellinabox iframe                   │
│          (fills available height)               │
│                                                 │
├─────────────────────────────────────────────────┤
│ [Esc|^C|▲|▼]  [▲|▼]       [copy|download|pause] │
│ [________ command input ______________] [Enter] │
└─────────────────────────────────────────────────┘
```

### Button groups

| Group | Buttons | Action |
|---|---|---|
| Terminal controls | `Esc` `^C` `▲` `▼` | Send Esc (`\x1b`), Ctrl+C (`\x03`), terminal up/down arrow sequences (`\x1b[A` / `\x1b[B`) |
| Input history | `▲` `▼` (amber) | Recall previous commands into the text box; cursor placed at end |
| Actions | `📋` `⬇` `⏸` | Copy terminal output to clipboard; download as timestamped `.txt`; pause/freeze the display (snapshot) |

### Input row

Text field + `Enter` button. Every button press returns focus to the input with cursor at end. Enter key on keyboard also submits.

### Keyboard-aware layout

Uses the `visualViewport` API. When the on-screen keyboard opens, the layout height is updated in real time (`visualViewport.height` + `offsetTop` for iOS Safari scroll offset), the terminal iframe compresses, and the controls stay pinned directly above the keyboard. When the keyboard closes the terminal fills all available space.

### Output capture

- **Primary:** reads `iframe.contentDocument#scrollable.innerText` directly (same-origin DOM).
- **Fallback:** postMessage output relay — requires `--messages-origin=*` on the daemon; accumulates raw terminal data (ANSI stripped) in a buffer used when DOM access is unavailable.

---

## Configuration

Edit `/etc/default/shellinabox`, append flags to `SHELLINABOX_ARGS`, then restart:

```bash
sudo systemctl restart shellinabox
```

### (1) Dark mode only

```
SHELLINABOX_ARGS="--no-beep --disable-ssl --css /etc/shellinabox/dark.css"
```

### (2) Mobile UI only

```
SHELLINABOX_ARGS="--no-beep --disable-ssl --messages-origin=* --static-file=/mobile:/etc/shellinabox/mobile.html"
```

### (3) Both

```
SHELLINABOX_ARGS="--no-beep --disable-ssl --css /etc/shellinabox/dark.css --messages-origin=* --static-file=/mobile:/etc/shellinabox/mobile.html"
```

> Replace `/etc/shellinabox/` with the actual path to the directory containing these files.

Access the mobile UI at `http://<host>:<port>/mobile`.
