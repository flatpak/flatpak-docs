Debugging
=========

This section includes documentation on how to debug Flatpak apps.

Debug packages
--------------

Before debugging, it is essential to install the debug packages used by
the application. This can be done by::


  $ flatpak install --include-sdk --include-debug $FLATPAK_ID

This will install the SDK, the Debug SDK and the Debug extension for
the application.

If it's a crash in the graphics stack, the GL debug extension will also
be needed.

First, note down the runtime branch used by the application::

  $ flatpak info --show-runtime $FLATPAK_ID
  org.freedesktop.Platform/x86_64/24.08

In the above example it is the Freedesktop SDK and the branch is
``24.08``. Then install the ``GL.debug`` extension for the above branch::

  $ flatpak install org.freedesktop.Platform.{GL,GL32}.Debug.default//24.08

The same process should be followed for any extension or baseapp used
by the app.

Debug shell
-----------

A debugging environment can be created by starting a shell inside the
sandbox::

  $ flatpak run --command=sh --devel --filesystem=$(pwd) $FLATPAK_ID

This creates a sandbox for the application with the given ID and, instead
of running the application, runs a shell inside the sandbox. The
``--devel`` option tells Flatpak to use the SDK as the runtime, which
includes various debugging tools and it also adjusts the sandbox setup
to enable debugging.

It is also possible to get a shell inside an application sandbox without
having to install it. This is done using ``flatpak-builder``'s ``--run``
option::

 $ flatpak-builder --run <build-dir> <manifest> sh

This sets up a sandbox that is populated with the build results found in
the build directory, and runs a shell inside it.

Using GDB in the sandbox
------------------------

Note that :ref:`debugging:Debug packages` must be installed to get
meaningful traces from GDB. Once inside the :ref:`debugging:Debug shell`
to run the application with ``gdb`` ::

 $ gdb /app/bin/<application-binary>

To pass arguments to the application::

  $ gdb --args /app/bin/<application-binary> <arguments>
  $ (gdb) run

A breakpoint can also be set for example on the ``main`` function
and once it is reached the source code can be listed::

  $ (gdb) break main
    (gdb) run
    (gdb) list

Once the bug is reproduced, if it is a crash it will automatically
return to the gdb prompt. In case of a freeze pressing Ctrl+c will cause
it to return to the gdb prompt. Now enable logging to a file (this will
be saved in the working directory and the Flatpak needs filesystem
access to that ``--filesystem=$(pwd)``)::

  $ (gdb) set logging enabled on

Then to get the backtrace::

  $ (gdb) bt full

Or for all threads, in case of a multi-threaded program::

  $ (gdb) thread apply all backtrace

Note that ``gdb`` inside the sandbox cannot use debug symbols from
host's `debuginfod servers <https://sourceware.org/elfutils/Debuginfod.html>`_.

Please also see the `GDB user manual <https://sourceware.org/gdb/current/onlinedocs/gdb.html/>`_
for a more complete overview on how to use GDB.

Getting stacktraces from a crash
--------------------------------

If an application crashed and the system has coredumps and
`systemd-coredump <https://www.freedesktop.org/software/systemd/man/latest/systemd-coredump.html#>`_
enabled, a coredump will be logged. Get the ``PID`` from that coredump::

  $ coredumpctl list

Now run ``flatpak-coredumpctl`` (this requires :ref:`debugging:Debug packages`
to be installed)::

  $ flatpak-coredumpctl -m <PID> $FLATPAK_ID
  (gdb) bt full

Using other debugging tools
---------------------------


``org.freedesktop.Sdk`` also includes other debugging tools like
`Valgrind <https://valgrind.org/>`_ which is useful to find memory leaks.
Once inside the :ref:`debugging:Debug shell`, it can be run with::

  $ valgrind --leak-check=full --track-origins=yes --show-leak-kinds=all --log-file="valgrind.log" /app/bin/<application-binary>

`Strace <https://strace.io/>`_ can be useful to check what an application
is doing. For example, to trace ``openat(), read()`` calls::

  $ strace -e trace=openat,read -o strace.log -f /app/bin/<application-binary>

`Perf <https://perfwiki.github.io/main/>`_ requires
access to ``--filesystem=/sys`` to run::

  $ flatpak run --command=perf --filesystem=/sys --filesystem=$(pwd) --devel $FLATPAK_ID record -v -- <command>

Multiple Debug shells in one sandbox
------------------------------------

Sometimes it can be helpful to have multiple debugging shells at a
time in the same sandox. For example, some debugging commands such as
`dotnet-dump <https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump>`_
expect to be given an identifier for a running process that it will
connect to for debugging. This essentially requires multiple instances
of the :ref:`debugging:Debug shell`, one to run the app, and one for
running the debugging tool.

To accomplish this, locate either the instance or process ID of an
existing debug shell with::

  $ flatpak ps

Then you can enter this same sandbox from a new terminal window using::

  $ flatpak enter <instance id> /bin/bash

Note that this second shell likely will not be configured identically
to the original debug shell (for example it will likely have different
environment variables such as PATH).

You can verify whether both the shells are indeed in the same sandbox by
checking the namespaces visible in each debug shell and
verifying that they are the same. The easiest way to do this is to list
the visible namespaces using::

  $ lsns --type pid

Creating a Debug extension
---------------------------

Like many other packaging systems, Flatpak separates bulky debug information
from regular content and ships it separately, in a Debug  extension.

When an application is built, ``flatpak-builder`` automatically
creates a Debug extension. This can be disabled with the ``no-debuginfo``
option.

To install the Debug extension created locally, pass ``--install``
to ``flatpak-builder`` which will set up a new remote for the build. The
remotes available can be checked with::

  $ flatpak remotes --columns=name,url

Then install the Debug extension from that remote::

  $ flatpak install foo-origin $FLATPAK_ID.Debug

Overriding sandbox permissions
------------------------------

It is sometimes useful to have extra permissions in a sandbox when debugging.
This can be achieved using the various sandbox options that are accepted by
the run command. For example::

 $ flatpak run --devel --command=sh --system-talk-name=org.freedesktop.login1 <application-id>

This command runs a shell in the sandbox for the given application, granting it
system bus access to the bus name owned by logind.

Inspecting portal permissions
-----------------------------

Flatpak has a number of commands that allow to manage portal permissions
for applications.

To see all portal permissions of an application, use::

 $ flatpak permission-show <application-id>

To reset all portal permissions of an application, use::

 $ flatpak permission-reset <application-id>


Interacting with running sandboxes
----------------------------------

You can see all the apps that are currently running in Flatpak sandboxes
(since 1.2)::

 $ flatpak ps

And, if you need to, you can terminate one by force (since 1.2)::

 $ flatpak kill <application-id>

Audit session or system bus traffic
-----------------------------------

A ``--socket=session-bus`` or a ``--socket=system-bus`` permission must
not be present for the logging to work.

Session bus traffic can be audited by passing ``--log-session-bus`` to
``flatpak run``::

  flatpak run --log-session-bus <application-id>

This can be useful to figure out the bus names used by an application
and the corresponding ``--talk-name`` or ``--own-name`` permissions
required::

  flatpak --log-session-bus run <application-id>| grep '(required 1)'

Similarly, system bus traffic can be audited by passing ``--log-system-bus``
to ``flatpak run``. This also requires a system bus name to be present
in the permissions. If not a bogus bus name can be passed::

  flatpak run --log-system-bus --system-talk-name=org.example.foo <application-id>
