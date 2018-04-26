Overriding sandbox permissions
==============================

It is sometimes useful to have extra permissions in a sandbox that is used for debugging.
This can be achieved using the various options of the run command. For example,

 $ flatpak run --devel --command=sh --system-talk-name=org.freedesktop.login1 application.id

runs a shell in the sandbox for the given application, granting it system bus access
to the busname owned by logind.
