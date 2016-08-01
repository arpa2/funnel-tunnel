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

### Setup of GUI applications

Note that OpenDesktop defines how [application menu
items](https://developer.gnome.org/integration-guide/stable/desktop-files.html.en)
are to be stored.  It should be possible to change, say,

`Exec=twinkle`

into

`Exec=funtun profile=6bed4 twinkle`

to apply a profile named `6bed4` and thereby extend the `twinkle` application
with the ability to run over IPv6 anywhere — without requirement of Twinkle to
support it in any way.

This sort of settings could be setup with a manual GUI component that edits
these files, or with a central provisioning mechanism such as
[SteamWorks](http://steamworks.arpa2.net).

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
