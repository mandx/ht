# vt / hvt / vterm / virterm / termviewer / termproxy / ?

TODO description

## Building

Building from source requires the [Rust](https://www.rust-lang.org/) compiler
(1.74 or later), and the [Cargo package
manager](https://doc.rust-lang.org/cargo/). If they are not available via your
system package manager then use [rustup](https://rustup.rs/).

To download the source code, build the binary, and install it in
`$HOME/.cargo/bin` run:

```sh
cargo install --git https://github.com/andyk/termproxy
```

Then, ensure `$HOME/.cargo/bin` is in your shell's `$PATH`.

Alternatively, you can manually download the source code and build the binary
with:

```sh
git clone https://github.com/andyk/termproxy
cd termproxy
cargo build --release
```

This produces the binary in _release mode_ (`--release`) at
`target/release/vt`. There are no other build artifacts so you can just
copy the binary to a directory in your `$PATH`.

## Usage

Run `vt` to start interactive bash shell running in a PTY (pseudo-terminal).

To launch a different program (a different shell, another program) run `vt
<command> <args...>`. For example:

- `vt fish` - starts fish shell
- `vt nano` - starts nano editor
- `vt nano /etc/fstab` - starts nano editor with /etc/fstab opened

Another way to run a specific program, e.g. `nano`, is to launch `vt` without a
command, i.e. use bash by default, and start nano from bash by sending `nano\r`
("nano" followed by "return" control character) to the process input. See [input
command](#input) below.

Default size of the virtual terminal window is 120x40 (cols by rows), which can
be changed with `--size` argument. For example: `vt --size 80x24`. The window
size can also be dynamically changed - see [resize command](#resize) below.

Run `vt -h` or `vt --help` to see all available options.

## API

Communication with vt is performed via stdin, stdout and stderr.

vt uses simple JSON-based protocol for sending commands to its stdin. Each 
command must be sent on a separate line and be a JSON object having `"type"`
field set to one the supported commands (below).

vt sends responses (where applicable) to its stdout, as JSON-encoded objects.

Diagnostic messages (notices, errors) are printed to stderr.

### input

`input` command allows sending arbitrary input to a process running in the
virtual terminal as if the input was typed on a keyboard.

```json
{ "type": "input", "payload": "ls\r" }
```

This command doesn't produce any output on stdout.

### getView

`getView` command allows obtaining a textual view of a terminal window.

```json
{ "type": "getView" }
```

This command responds with the current view on stdout. The view is a multi-line
string, where each line represents a terminal row.

```json
{ "view": "[user@host dir]$                 \n                       \n..." }
```

### resize

`resize` command allows resizing the virtual terminal window dynamically by
specifying new width (`cols`) and height (`rows`).

```json
{ "type": "resize", "cols": 80, "rows": 24 }
```

This command doesn't produce any output on stdout.

## Testing on command line

vt is aimed at programmatic use given its JSON-based API, however one can play
with it by just launching it in a normal desktop terminal emulator and typing in
JSON-encoded commands from keyboard and observing the output on stdout.

[rlwrap](https://github.com/hanslub42/rlwrap) can be used to wrap stdin in a
readline based editable prompt, which also provides history (up/down arrows).

To use `rlwrap` with `vt`:

```sh
rlwrap vt [vt-args...]
```

## License

All code is licensed under the Apache License, Version 2.0. See LICENSE file for
details.
