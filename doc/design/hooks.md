Hooks into the Funnel Tunnel
============================

>   *It’s nice to imagine a rewriting framework for network traffic, but how to
>   actually pass traffic through it?*

Client Applications
-------------------

This section deals with client applications, usually run by users.

### A dæmon like SOCKS5

The [SOCKS5 protocol](https://tools.ietf.org/html/rfc1928) is defined to let
network clients leave a network, in spite of tight control over outgoing
traffic.  It will authenticate the client to an outgoing firewall, which will
selectively agree to setup an intermediate to which the client can connect
instead of directly.

Applications or operating systems may implement SOCKS5 natively.  Others can be
made to comply by setting `LD_PRELOAD` to overrule socket API calls with ones
that work through SOCKS5 instead of directly.  Commands like
[tsocks](http://tsocks.sourceforge.net/faq.php) or
[socksify](http://linux.die.net/man/1/socksify) are normally used to bypass to a
preconfigured SOCKS5 server.

This server may in fact be running the Funnel Tunnel, and not a SOCKS5 service.
The dæmon would be running as the user who started it, so it may be either a
system-level service or a user-level service.  Note that this may limit the
credentials available to the Funnel Tunnel.

### A library like SOCKS5

The trick to set `LD_PRELOAD` can also be used with another library, that
overrules the socket API with calls that integrate with the Funnel Tunnel.  In
this case, the Funnel Tunnel runs as the user who invoked the command.

The same library should be relatively easy to compile in with an application,
also replacing the usual socket API.  It may end up being a `./configure` option
in applications, which is a relatively simple way of embedding a potentially
unbounded plethora of tunnelling and other tricks within an application.

Additional options that may also exist to override header files, replace symbols
in linker prescriptions, and many other naughty tricks.

When dæmon code like SOCKS5 already exists, it may be possible to construct
a library whose initialisation code sets up such a dæmon over a UNIX domain
socket, and overrules á la SOCKS5 to forward the socket API to this
UNIX domain socket.  The dæmon may or may not run in a separate process;
but in any case it will run with the same privileges as the library that
started it, and so with the same privileges as the user starting the program.


Convincing Applications to be Confined by the Tunnel Tunnel
-----------------------------------------------------------

### Globally set Preload Environment Variable

The `LD_PRELOAD` variable can be set to always load `libfunnel.so`, which may
then find configuration files for the program being run under a user's setup.

### Setup of GUI applications

Note that OpenDesktop defines how [application menu
items](https://developer.gnome.org/integration-guide/stable/desktop-files.html.en)
are to be stored.  It should be possible to change, say,

`Exec=twinkle`

into

`Exec=funnel --profile=6bed4 -- twinkle`

to apply a profile named `6bed4` and thereby extend the `twinkle` application
with the ability to run over IPv6 anywhere — without requirement of Twinkle to
support it in any way.

This sort of settings could be setup with a manual GUI component that edits
these files, or with a central provisioning mechanism such as
[SteamWorks](http://steamworks.arpa2.net).

This requires either write access to the menu files (by administrator or
distribution) or a local meny option copy must be made into user controllable
space.

### Shell Wrapper Programs

This is a straightforward option; shell programs may be written to add local
modifications to one's commands.  A properly set `PATH` variable could prefer
per-user local variations over the widely installed ones.

This may not impact GUI menu entries when these have absolute paths.

### Patching ELF Files?

This is a bit of a stretch.

Just like the [patchelf](https://www.mankier.com/1/patchelf) utility can setup
a dedicated `LD_LIBRARY_PATH` for an ELF executable (which is used heavily in
Nix), there *may* be a way to use it, or an improved version of it, to set
the `LD_PRELOAD` variable for a specific binary.

This option either requires support from the administrator, the distribution
or to make a local copy of an executable.

It does seem to be possible within the [ELF](https://www.mankier.com/5/elf)
format to do these things; the `.dynamic` section holds `DT_NEEDED` entries
with libraries that are required.  A new entry might be prefixed before the
actual implementation of the socket API.  Note that this may be needed for
included libraries as well, if they are already linked to a library that
implements the socket API (such as glibc).  Such libraries can be detected
by looking for the symbols, so ample warnings might be given.  The Nix
approach of setting up a `LD_LIBRARY_PATH` through the `DT_RUNPATH`
setting in the same `.dynamic` section could be used to divert to the right
underlying libraries.

All this may turn out to be really simple with Nix.  Unfortunately, it would
not be wise to constrain ourselves to that platform, but instead we should
find a binding method that works everywhere -- even if it is inspired by Nix.

This binding cannot currently be made with `patchelf`, but it does seem to
be right up to its game, and so an extension of the program may be prudent.


Server Applications
-------------------

Servers are different types of program, because they tend to listen to
fixed/known resources.  And because they are usually run by a more advanced type
of user, perhaps even `root`.  This means that we have a few additional hooking
techniques that might work.

### A dæmon like inetd

Programs like `inetd` and `xinetd` listen on ports and start a program with an
incoming external connection assigned to `stdin` and `stdout`.  This may be used
with programs that (would be made to) expect TCP/IP interaction, rewriting their
streams as they pass through.

### A service like systemd

Several programs now offer more control over sockets outside the dæmon itself;
we will discuss `systemd` below, without intending to limit the options to just
that.

A program started by `systemd` is provided with a number of sockets already
opened.  These may in fact be handled inside the Funnel Tunnel, instead of
directly.  When the sockets are approached through `select()`, `poll()`,
`epoll()` or `kqueue()`, they may in fact end up waiting for Funnel Tunnel input
through a goroutine.  This would work when these special calls are taken over
with the `LD_PRELOAD` trick or another, mentioned above.

### A higher-level API

As an alternative to overruling existing APIs, it is also possible to consider a
higher-level API, where an application simply asks to open a socket of a given
template.  This is then dealt with in the Funnel Tunnel, which arranges whatever
is needed to connect the right Puzzle Pieces to that named template.

A higher-level API may also take over `epoll()` and alike.  Instead of passing
them through kernel calls, the wait for events would be achieved in user space,
through Funnel Tunnel constructs.  In the end, the Funnel Tunnel is likely to
wait for a similar event driver, of course.

### A dæmon like a SOCKS5 proxy

On the server end, SOCKS5 is not usually usable, because the protocol does not
deal with incoming connections.  What is possible however, is to create a dæmon
that talks the SOCKS5 protocol for connecting clients, and that will handle it
locally by connecting it to a local server-side.

### Firewall redirection rules

When `root` established the Funnel Tunnel support, which is not our primary
design goal but is also not something we wish to avoid, then it is possible to
create firewall rules that redirect traffic to a local socket for handling.
This too may end up in a Funnel Tunnel.
