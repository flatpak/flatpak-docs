# Build and Debug

Related public docs: [Building your first Flatpak](https://docs.flatpak.org/en/latest/first-build.html), [Building Introduction](https://docs.flatpak.org/en/latest/building-introduction.html), [Flatpak Builder](https://docs.flatpak.org/en/latest/flatpak-builder.html), [Debugging](https://docs.flatpak.org/en/latest/debugging.html), [Tips and Tricks](https://docs.flatpak.org/en/latest/tips-and-tricks.html), [Using Flatpak](https://docs.flatpak.org/en/latest/using-flatpak.html).

## Setup and build

Install `flatpak` and `flatpak-builder` from the distribution or from Flathub. For development, add Flathub user-wide:

```bash
flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Build and install a manifest locally:

```bash
flatpak-builder --force-clean --user --install-deps-from=flathub --repo=repo --install builddir org.example.App.yml
flatpak run org.example.App
```

What `flatpak-builder` does:

- Creates the build directory.
- Downloads and verifies module sources.
- Builds and installs each module into the app prefix.
- Applies finish/sandbox metadata.
- Exports the result to a repository when `--repo` is used.

Useful variants:

```bash
# Export to a local repo without installing.
flatpak-builder --force-clean --user --install-deps-from=flathub --repo=repo builddir org.example.App.yml

# Install directly without a separate repo step.
flatpak-builder --user --install builddir org.example.App.yml

# Sign exported commits.
flatpak-builder --gpg-sign=<key-id> --repo=repo builddir org.example.App.yml
```

Use `--gpg-homedir` if the signing key lives outside the default GPG home.

## Run commands inside the sandbox

Use an installed app sandbox:

```bash
flatpak run --command=sh --devel --filesystem="$(pwd)" org.example.App
```

Use an uninstalled build tree:

```bash
flatpak-builder --run builddir org.example.App.yml sh
flatpak-builder --run builddir org.example.App.yml /app/bin/app-command
```

`--devel` uses the SDK as runtime and adjusts sandbox setup for debugging.

## Debug symbols and tools

Install debug materials first:

```bash
flatpak install --include-sdk --include-debug org.example.App
```

For graphics stack crashes, identify the runtime branch and install GL debug extensions:

```bash
flatpak info --show-runtime org.example.App
flatpak install org.freedesktop.Platform.{GL,GL32}.Debug.default//24.08
```

Inside a debug shell:

```bash
gdb /app/bin/app-command
gdb --args /app/bin/app-command --some-arg
valgrind --leak-check=full --track-origins=yes --show-leak-kinds=all --log-file=valgrind.log /app/bin/app-command
strace -e trace=openat,read -o strace.log -f /app/bin/app-command
```

For `perf`, grant `/sys` and a writable output path:

```bash
flatpak run --command=perf --filesystem=/sys --filesystem="$(pwd)" --devel org.example.App record -v -- <command>
```

## Coredumps

If systemd coredumps are enabled:

```bash
coredumpctl list
flatpak-coredumpctl -m <PID> org.example.App
(gdb) bt full
```

## Multiple shells in one sandbox

List running sandbox instances and enter one:

```bash
flatpak ps
flatpak enter <instance-id> /bin/bash
```

Use this when one shell runs the app and another shell attaches a debugger.

## Permissions while debugging

Temporarily add runtime permissions to a debug run rather than broadening the manifest:

```bash
flatpak run --devel --command=sh --system-talk-name=org.freedesktop.login1 org.example.App
```

Inspect and reset portal permissions:

```bash
flatpak permission-show org.example.App
flatpak permission-reset org.example.App
```

See running apps and stop one:

```bash
flatpak ps
flatpak kill org.example.App
```

## D-Bus audit

Do not grant `--socket=session-bus` or `--socket=system-bus` when auditing; those disable the useful filtering.

```bash
flatpak run --log-session-bus org.example.App
flatpak run --log-session-bus org.example.App | grep '(required 1)'
flatpak run --log-system-bus --system-talk-name=org.example.Placeholder org.example.App
```

Translate observed names into narrow `--talk-name=` or `--system-talk-name=` grants only when a portal is not appropriate.

## Test against runtimes

For testing only:

```bash
flatpak run --runtime-version=master org.example.App
flatpak run --runtime=org.gnome.Sdk org.example.App
flatpak run -d org.example.App
```

Do not treat success on an ABI-incompatible runtime as supported behavior.
