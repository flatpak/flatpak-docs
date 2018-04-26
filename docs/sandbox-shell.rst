Getting a shell in your app’s sandbox
=====================================

If you need to investigate an installed Flatpak application, you can use the --command
option to get a shell inside the sandbox:

 $ flatpak run --command=sh --devel application.id

This is setting up the sandbox for the application with the given ID, but then runs
a shell inside it, instead of the application binary. From the shell prompt, you can
run the application binary, either directly, or under gdb:

 $ gdb /app/bin/application-binary

This works, because the --devel option tells flatpak to use the SDK as runtime,
which includes debugging tools like gdb. The option also makes flatpak tweak the
sandbox setup in some ways, to enable debugging.

gdb is much more useful when it has access to debug information for the application
and the runtime it is using. Flatpak is splitting this off into .Debug extensions,
which you should install before debugging an application:

 $ flatpak install RUNTIME_ID.Debug

When you use the --devel option, Flatpak will automatically use any matching .Debug
extensions that it finds.


When you've built the application using flatpak-builder, you can get a shell inside
the sandbox for the built app without installing it on your system, by using the --run
option of flatpak-builder:

 $ flatpak-builder --run build application.id.json sh

This is setting up a sandbox that is populated with the build results found in
the build/ directory, and runs a shell inside it.
