Creating a .Debug extension
===========================

Like many other packaging systems, Flatpak separates bulky debug information from
the regular content in runtimes, and ships it separately, in what is called a
.Debug extension. 

When you build an application or runtime, flatpak-builder will automatically
create a .Debug extension for you, unless you have disabled this with the no-debuginfo
option.
