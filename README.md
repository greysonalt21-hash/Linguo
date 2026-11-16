# Linguo

Linguo is a small, interpreted programming language with a Windows command-line
runtime. Programs use `.linguo`, `.lg`, or `.ling` files. The runtime is
implemented in JavaScript and does not use Python.

Current command-line toolchain version: `0.3.1` (early development release).
The VS Code extension is versioned independently.

Linguo uses an **All Rights Reserved** license, not an open-source license.
The copyright holder retains ownership. You may use unmodified official
releases, but may not redistribute or publish modified versions without
written permission. See [`LICENSE`](LICENSE) for the terms.

## Install on Windows

Download and extract a Linguo Windows installer package. Open PowerShell in
the extracted folder and run:

```powershell
.\install-linguo.cmd
```

The setup wizard installs Linguo under your user profile, adds `linguo` to your
user PATH, and installs the VS Code extension if VS Code is present. If a
compatible Node.js runtime is not already installed, setup downloads the
Windows runtime and verifies its SHA-256 checksum. Python is not required.
Choose **Find releases** to list published installer ZIPs from GitHub Releases,
or install the bundled version without accessing GitHub. Downloaded release
ZIPs must have a GitHub-provided SHA-256 digest. If the repository or release
is unavailable, setup reports the error and the bundled version remains usable.

Close and reopen the terminal after installation.

## Run a program

```powershell
linguo run path\to\program.linguo
```

For example:

```powershell
linguo run "C:\Users\you\Documents\hello.lg"
```

Use `linguo --help` to display command usage. The previous direct form,
`linguo path\to\program.linguo`, still works. In VS Code, the optional
extension adds Run and Debug commands. Other editors can run Linguo through
their integrated terminal; editor-specific syntax highlighting and debugging
require an extension for that editor.

## Build the Windows installer package

First package the VS Code extension from `linguo-vscode`:

```powershell
Set-Location .\linguo-vscode
npx --yes '@vscode/vsce' package
Set-Location ..
.\linguo-installer\build-installer.ps1
```

The generated archive is written to `dist`. It contains the setup scripts,
CLI/runtime, and VS Code extension VSIX.

## Publish a version

The version picker checks `greysonalt21-hash/Linguo` on GitHub; versions do not
appear until published there. To publish an installer:

1. Create a public GitHub repository named `Linguo` under `greysonalt21-hash`.
2. Set `linguo-installer\version.json` to the intended version and build its
   installer as described above.
3. On the repository's **Releases** page, choose **Draft a new release**.
4. Create a tag matching the version, such as `v0.3.1`, and choose a release title.
5. Upload `dist\Linguo-Installer-0.3.1.zip` (using the matching version number)
   as a release asset, then publish the release.
6. Open setup, select **Find releases**, and select the published version. Only
   matching published assets are listed; absent versions are not invented.

GitHub's asset SHA-256 digest checks download integrity; it is not a publisher
signature. Older assets without a digest must be re-uploaded before setup can
install them. Publishing requires the copyright holder's authorization.

## Language features

The Python runner (`linguo.py`) and JavaScript runtime (`linguo-vscode\runtime.js`)
are separate implementations. Both support the documented Linguo syntax for
input, variables, arithmetic, printing, conditionals, loops, random choices,
named text lists, functions, imports, and program exit. Conditional branches
can contain consecutive statements and end at a blank line or the next
`elif`/`else`. Inputs can be limited to quoted choices with
`choice: allow "bl", "gh"` before the input statement.

On Windows, Linguo can open a native desktop window and place buttons using
pixel coordinates:

```linguo
import UIMaker
UIMaker:

make.window(W1)
W1.size(500pxX500px)
add.W1.button(B1)
B1.Cords(x=24 y=96)
functionality.B1 == click.Event(open.window(W2))

make.window(W2)
add.W2.button(B2)
B2.Cords(x=92 y=34)
```

`make.window(W1)` creates the window; `W1.size(500pxX500px)` sets its client
size in pixels. `add.W1.button(B1)` adds a named button, and
`B1.Cords(x=24 y=96)` sets its position relative to the window's top-left.
Windows default to 800 by 600 pixels and buttons to 100 by 32 pixels.
Buttons use their names as labels and report clicks to the Linguo output.
`functionality.B1 == click.Event(open.window(W2))` closes the current window
and opens W2; a `print "text"` click action writes to the Linguo output.

The built-in UI also supports labels and text inputs. A save click action can
write a note as a UTF-8 `.txt` file next to the Linguo program:
`click.Event(save {{NoteText}} as {{FileName}} with(.txt).end)`.
The entered name is suffixed with `.txt` when needed, and saving an existing
name replaces that file.

To add an image, type `image.create(I1)` in the Linguo editor or a Linguo file
in VS Code and select a PNG, JPEG, GIF, or BMP (up to 10 MB). The editor adds
an `image.data(I1, "...")` line containing the image; keep that line in the
saved program. The program can display the image without the original file:

```linguo
image.create(I1)
# The editor inserts image.data(I1, "...") here.
import UIMaker
make.window(W1)
add.W1.image(I1)
I1.Size(200pxX200px)
I1.Cords(x=20 y=20)
```

Canceling the picker leaves the declaration without image data; complete the
statement again to select a file. Image controls default to 200 by 200 pixels.
Editors without the Linguo image picker can run saved programs with embedded
data but do not automatically open an image selection dialog.

`W1.size(fullscreenWithadjust)` makes a window resizable and maximized; press
F11 to toggle borderless fullscreen. Add
`add.W1.cordCursor(cordCursor)`, `add.W1.display(display)`, and
`display.show(cordCursor).topMiddle` to display live x/y mouse coordinates at
the top center.

Linguo also includes an original one-button flying game with simple geometric
graphics. Create it with `make.game(SkyHopper)` inside the `UIMaker:` block;
press Space or click to fly, and press Space after a collision to restart.

The syntax guide is in [`linguo_syntax/README.md`](linguo_syntax/README.md).

## Development tests

```powershell
python -m unittest discover -s .\tests -v
.\tests\test_installer.ps1
Push-Location .\linguo-vscode
npm test
Pop-Location
```

The distributable package is built locally. GitHub Releases must be set up and
populated separately; this project does not publish releases automatically.
