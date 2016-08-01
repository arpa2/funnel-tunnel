Funnel Tunnel Specification
===========================

>   *This specification describes the design of the Funnel Tunnel, a pluggable
>   architecture under user control that modifies user-level network traffic
>   without dependency on applications or administrators.*

Rationale
---------

Network protocol innovations are extremely difficult to get deployed. The
introduction of IPv6 and SCTP are good examples of very promosing technology,
which are slowed down by the conservative thinking “I cannot be sure that every
remote can handle it, so why should I bother?” Both can be solved with software
components in the right place, such as tunnels or other modifiers of network
frames.

The current model for these is a choice between either

1.  System-level support by administrators with tunnels, netfilters, routing,
    traffic shaping;

2.  Application-level support that builds network innovations into user code.

The first model is general, which may be good or bad, but it certainly is a
complicating matter in systems without qualified administrators. In practice,
this model is nice for hackers but not generally usable as a general solution.
Also, different innovations may interfere, such as an Android App that builds
into the VPNService; there can only be VPN, for good reasons, but this limits
the potential of stacking innovations that would want to sit in that slot.

The second model is much less invasive, because an application may automatically
negotiate the desired innovative technology. Though this is a perfect solution
for closed protocols with a limited number of clients, it is not very pleasant
with open protocols with a plethora of implementations; only those
implementations that add the innovation would be suitable communication peers.

The [Funnel Tunnel](http://funnel.arpa2.net) sits in between the application and
the operating system, modifying the traffic flow between them. The manner in
which this is done can be configured by the user, and it is assumed that clever
operational tools are introduced to make this happen. The Funnel Tunnel is then
the implementation platform for these operational tools to work on.

Within the Funnel Tunnel, components can be used to implement various
modifications, and some may be very simple yet powerful. For example, a tunnel
maker has the last vote on carrying a protocol over UDP and not TLS. But a
second module may take a UDP-carried protocol and pass it over TLS. This is the
sort of end-user control that is currently missing, and it leads to feature
bloat in tunnels and/or applications. This is why the Funnel Tunnel is built to
accommodate pluggable network modification components.

### Case study: SIP over IPv6

One reason that SIP technology is still locked in by telecoms and network
operators is that it is incredibly difficult to setup point-to-point
connections. There are situations where NAT traversal makes it impossible to be
sure that direct media connections will actually work. Using IPv6 evades this
problem, because it evades NAT; all that remains is a firewall, which is simple
enough to pass through. Without NAT, no external party is needed to correct SDP
attachments, so that these can be encrypted — and make them negotiate keys used
to encrypt RTP traffic: Encrypted telephony at last!

But not all phones will run over IPv6; tools like
[SIPproxy64](http://devel.0cpm.org/sipproxy64/) could be plugged in between to
resolve that.

And not all phones implement encryption; that could be yet another plugin. Using
the scheme described above, and falling back to ZRTP if need be.  In fact, ZRTP
is a good example of a “hack in the network stack” that innovated the network
stack with a fair degree of success.

The main point however, is that control over user-level protocols lies with the
user, not the administrator and also not constrained by applications.

An important requirement that comes from this case study is that traffic must be
dealt with in realtime; this interferes with any dependency on a garbage
collector for the normal handling of message flows (though not necessarily with
such a dependency for structural administration and configuration).

### Case study: Tunnels

There are many reasons to setup tunnels, but mostly it is to fulfil a missing
network facility, such as SCTP, ESP or IPv6. There are many choices. Looking
back at the SIP example, it is quite attractive if not the phone application
limits what tunnel it supports, but on the other hand it is also not always
pleasant to be constrained to what the administrator has setup. Note that we are
not talking about attacks of any kind — it is merely a matter of getting access
to more advanced network facilities.

As an example tunnel, [6bed4](http://devel.0cpm.org/6bed4/) is designed as a
plug-and-play IPv6 solution. It may fallback on a tunnel server for pathetic
cases, but will be able to connect directly to the remote’s IPv4 address in any
normal case. The flexibility is all at the IPv4 level, and IPv6 addressing
reflects transparent addressing as it ought to.

It is very useful for users to be able to insert 6bed4 whenever needed, and not
have to rely on either all administrators to support it, or all applications.
Since tunnels are often run over UDP, it is not a security breach to make them
controllable by users.

Another angle might be that 6bed4, which runs over UDP, may need to be run over
another carrier in certain internal contexts; for instance, over an SCTP stream.

### Case study: SCTP concentrators

Finally, the [original reason](http://funnel.arpa2.net) for wanting a Funnel
Tunnel was the ability to pass multiple administrative protocols between network
sites. Protocols such as RADIUS and SNMP are designed to run locally. They use
UDP and resends, which may be counter-productive when they are passed between
remote sites.

Given a single SCTP connection between sites, it is possible to pass UDP
connections over a stream reliably. Any acknowledgements can be made locally,
and timeouts and such may be avoided. Similar things can be said for multicast
protocols that repeat information; for those, only changes need to be passed
reliably.

SCTP streams can be encrypted too — and one style may not be for everyone, so
once more it is a good idea to have user control over what to apply. And once
more, it is the sort of thing where it is neither necessary nor desirable to
depend on all administrators or all applications to accommodate what a series of
Funnel Tunnel plugins can achieve if and when the user wants it.

Network Puzzle Pieces
---------------------

The active pieces that are connected by the Funnel Tunnel are like pieces of a
puzzle; certain connections are possible and others are not, depending

### Requirements to Puzzle Pieces

-   Puzzle Pieces must support being joined on Edges with a matching Shape

-   Puzzle Pieces form self-contained modifiers for traffic passing through

-   Puzzle Pieces declare their structure so they can be used by the Funnel
    Tunnel

-   Puzzle Pieces communicate directly when the Funnel Tunnel has joined them

-   Puzzle Pieces exist to entrance and exit nodes for network traffic

### Descriptors of Protocols and Stacks

To be able to **recognise frames** in the Funnel Tunnel, we will match a part of
the protocol stack.  For example, IP, containing TCP, using local port 80 could
represent a local web service.  Usually, we match identifiers for a protocol in
a frame of the right type.  The frame’s type may be determined by an identifier
in a lower-layer protocol.  Protocol identifiers are often numeric (and
registered by name space authorities such as IANA) but they may also be textual
(such as for DNS-SD) or have any other form.

The generalisation of this idea is that **Name Spaces define a scope for
Protocol Identifiers**.  Each Protocol may define a Name Space for the/each
contained payload, which means that a contained Protocol may be matched after
the containing Protocol has been matched, and so on.  A **Protocol Stack is a
matching description** consisting of an initial Name Space, followed by zero or
more Protocols, each next one contained in the preceding one.  The given example
would start at the Name Space of an
[ethertype](http://www.iana.org/assignments/ieee-802-numbers/ieee-802-numbers.xhtml#ieee-802-numbers-1),
hold Protocol 0x0800 for IPv4 which brings us to the [protocol
number](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)
Name Space, where Protocol 6 selects TCP, which brings us to the Name Space for
[port
numbers](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml),
from which 80 selects HTTP.

This description could match an Ethernet frame.  An alternative Protocol Stack
might have started at the protocol number Name Space, and support other
transports as well, such as PPP.  The Funnel Tunnel would then interpret the
incoming protocol, be it PPP or Ethernet, to detect a contained IP packet and
start matching from there on.  This is possible because network data is normally
explicit about the frames carried; network stacks need that kind of information,
and the Funnel Tunnel can use it just as well.

It may be possible for the Funnel Tunnel to be agnostic to the distinction
between IPv4 and IPv6 when referred to as IP; but in some places (such as
IPv6-over-IPv4 tunnelling) there may be a use for mentioning the distinction
explicitly.  In the example where we treated PPP and Ethernet, usually from
different inputs, as **Equivalent Carriers** towards any Puzzle Piece that
starts caring at the IP level, independently of how it is carried.  The case for
treating IPv4 and IPv6 together as IP is a special case, because it may actually
match an IP header of which a recipient is aware.  We might describe IP as a
**Generic Carrier** if we wish to distinguish it.

### Passing around Network Frames

Upon entry, the Funnel Tunnel will **parse headers and possibly trailers**,
using that to split a network frame into a format that should be fairly
straightforward to process in Puzzle Pieces.  These are supplied with the
complete frame as a sequence of headers, a sequence of body parts and a sequence
of trailers.  Since not all headers and trailers are meaningful to all Puzzle
Pieces, this passed information will be annotated with “you need not concern
yourself with the first *N* headers, nor with the last *M* trailers”.  The
Puzzle Piece is likely to consider only a few of the first headers after *N* and
possibly a few trailers before the last *M*, and consider the rest payload.

The Funnel Tunnel forms a description of the longest Protocol Stacks that may be
needed to avoid further parsing in the Puzzle Pieces.  To this end, it needs a
view of the manipulations that may be going on inside each Puzzle Piece.  This
ensures that a network frame is parsed once, and then passed around and
modified.  Puzzle Pieces can alter the composition of a network frame, but will
always deliver the same preparsed format, so any follow-up Puzzle Pieces can
continue to work on them without much parsing.  In fact, even the parser may
have a need to prefix headers, such as a UDP header when data comes in over a
UDP socket (and is thus stripped from this particular header).

The Funnel Tunnel connects Puzzle Pieces, and each Puzzle Piece takes care of
passing the network frame around to connected Puzzle Pieces, perhaps subject to
certain conditions that it holds dear.  Part of the conditions may be protocol
properties that are exchanged as metadata, such as whether delivery is reliable,
in-order, duplicate-free, framed or simply concatenated, allowable sizes, and so
on.

Naturally, when a network frame is sent out, the headers, payloads and trailers
that matter will be **reconstituted into a frame for sending**.

### Descriptors of Puzzle Pieces

We now turn to the description of the Puzzle Pieces, which are the dynamically
pluggable components that encompass the Funnel Tunnel’s traffic handling.

The most important aspect to Puzzle Pieces is the **definition of Edges** of
each Puzzle Piece:

-   Puzzle Pieces have any number of connections to the outside network, named
    Edges

-   Connections are defined as a number of protocols layered on top of each
    other

-   Protocols are generally defined within the nameset of the lower layer
    protocol

-   Protocols in multiple namespaces may be equivalent, e.g. Ethernet,IP ==
    PPP/IP

-   Protocols have opposing directions, usually “up” and “down”; alternatively,
    “peer”

-   Protocol sets are equivalent when a bidirectional equivalence between
    elements exists

-   A namespace of the lowest protocol may be made explicit, without enforcing
    that layer

-   An Edge may be optional; and it can have alternative specifications, named
    Shapes

To “match” the Shape of an Edge to a protocol stack, first find an equivalent
piece of the stack. Then compare layers by looking for overlap in the sets of
protocols. When the end of an Shape is reached, then a match has been found. An
Edge is said to match a Protocol Stack when at least one of its Shapes matches.

There is an operation called **joining of Puzzle Pieces**. This is when two or
more Edges of Puzzle Pieces are brought together, and a Protocol Stack can be
constructed that matches all these Edges together. The place where Edges come
together is called a Joint.  For practical purposes, a Joint can have (Edges of)
Puzzle Pieces added and removed, to which it will adapt.

Another part of the description of a Puzzle Piece are the **possible traffic
paths, named Curves** within each:

-   A traffic path is named a Curve; it’s like a (possibly bent) line over a
    puzzle piece.

-   Curves run from one Shape to the same or another Shape

-   Curves from an Edge back to itself are called Hairpin Curves

-   Curves are bidirectional; this saves a lot of work during specifications and
    matching.

-   Curves describe optional, alternative, splitting or even delayed
    communication paths.

-   Curves are applied without transitive closure, except for Hairpin Curves

Based on the specified Curves, it is possible to derive what protocol layers are
additionally assumed when a network packet passes through a Puzzle Piece. This
is useful because it helps to compute the possible relations between inputs and
outputs of a Puzzle Piece.  This means that changes to a Joint behind a Puzzle
Piece can be applied to other Joints as well, so as to know what the
header/trailer parser must prepare for the Puzzle Piece and everything the might
come after it.

Hairpin Curves are implemented as a Hairpin that turns back to the same Edge
within the Puzzle Piece.  The Funnel Tunnel never causes traffic to pass back to
the sending component.

In addition, there is an **API** instantiation for each Shape, possibly shared
within the Edge:

-   Settings for parameters specific to an Edge instance

-   Interaction about required/desired/rejected and available/enabled protocol
    properties

-   Functions for registering and deregistering listening Shapes of other Puzzle
    Pieces

-   …more…?

### Implementation: Memory Management and Header Handling

Garbage collection can be costly, and for this reason it *may* be desirable for
efficiency reasons to recycle memory blocks. Specifically for Go, it is an
implementation-depenendent factor.  In practice, 10 ms stop-the-world is
considered practical, which is however not tolerable to realtime protocols such
as RTP.

Memory can be explicitly discarded by submitting it to a memory manager, which
cares for recycling the memory. In Go this can be done with `sync.Pool` or a
wrapper around it that may add reference counting.  Even if this adds overhead,
it is a constant amount of overhead per frame, and that will give more
consistent network behaviour than garbage collection mechanisms which
stop-the-world.

One implication of this style is that Puzzle Pieces maintain a reference count.
When a network packet passes through one-to-one, it need not do anything; but
when a packet is swallowed or split to multiple targets, then the usecount must
be accordingly decremented or increased.

Saving on dynamic allocation is a constant concern, so buffers are mostly static
and memory is reused whenever possible. A binary packet will be split into a
sequence of headers with a payload, but referring to the same underlying byte
array.  This array may be extended by Puzzle Pieces that wish to add headers or
trailers.  Similarly, the headers/payload/trailers slices should come from (an)
array(s) that support extension.  This will lead to some dynamic allocation when
the protocol is started, but the pooled and reused frame memory will quickly
learn, and be large enough.  (It may be optimal to have multiple pools, though,
but that need not concern us in a first version.)

There is no strict reason why headers and trailers must be binary. Textual
parsers may be just as useful. To facilitate multiple (sub)headers, parsed
independently, we mark each header with the protocol that recognises it. That
way, even flexible header schemes such as SMTP (and, for that matter, IPv6) can
be properly parsed. Addition of the role (header, payload, trailer) offers even
more specialisation facilities, even supporting multipart payloads if need be.

### Implementation: Coroutines and Channels

The requirements of static memory allocation for constant network traffic
patterns, and of sending traffic in both upstream and dowstream direction (or,
for that matter, between peering network nodes) combine well in a model with
coroutines and channels. Although we started work on
[coconut](https://github.com/vanrein/coconut/blob/master/manual.md) to
facilitate these functions in C, the computation model is directly built into
Go, where it works much more comfortably. Also, with the possible exception of
garbage collection which *may* be implemented to clash with networking software
requirements, the communication model of Go is perfectly suited for the Funnel
Tunnel design.

Every Puzzle Piece can be a goroutine; every Shape can be a Channel. Various
Shapes from the same Edge may share one Channel, but different Edges are always
different Channels. Registration comes down to connecting Channels between
Puzzle Pieces; the Funnel Tunnel would supplie a sending-side Shape of a Puzzle
Piece with the receiving-side Shape of a Puzzle Piece; usually this would be
applied symmetrically. There is no intermediate function performed by the Funnel
Tunnel. Even networks that broadcast traffic to all connected peers will be
implemented as a Puzzle Piece… because they can. And even connections to the
context, such as to an operating system network interface or to a locally run
application or to an underlying C library, are made through a Puzzle Piece
(which may be specific to an operating system, or which may have a dedicated
implementation on each operating system).

Upon entry into the system, header/trailer parsers are run to accommodate the
possible Shapes on the outgoing Joint. This leads to a network frame in the form
of a sequence of array slices (with parsed header/payload/trailer typing) taken
from the array of bytes that represents the incoming frame. The storage space
used for this array of bytes comes from the recycling allocator (such as a
`sync.Pool` derivative), and a reference to be used for reference counting is
included with the parsed frame. When passed to a peer over a Channel, an index
to start from is passed along with it. Note how even recursive parsing (such as
may occur for tunnels) can be properly handled when this is done. Also note that
the needed index must be communicated during the registration of Channels in the
various components.

Upon exit from the system the reverse from parsing is done — the various parts
and pieces are combined and used to dispatch the network frame over
infrastructure out of sight of the Funnel Tunnel. It will be very, very common
for entrance and exit to occur on the same nodes, but in the opposite direction
of network traffic.

Funnel Tunnel in Deployments
----------------------------

### Automated default policy

TODO

-   Silently hook up matching protocols

-   Allow configurations to provide a network name for the underlying layer for
    segmentation

### Explicit policies

TODO

-   Instantiate Puzzle Pieces, and give them a name

-   Connect the Edges (or Edge Shapes) to named networks

### Provisioned policies

TODO

-   Provide configurations over LDAP (using SteamWorks Pulley)

-   Insert the configuration into the Funnel Tunnel configuration

-   Every configuration can have a (file) name that can be bound in by
    applications

### System-level Funnel Tunnels

TODO

-   Started by an operating system daemon rather than an application

-   Perhaps with per-user / per-application forking (independent name spaces)

-   Perhaps with shared parts between users / between applications / between
    both

### Atomic configuration changes

-   Install configuration changes, detect if it is acceptable

-   When all accept it, confirm; otherwise, best retract

-   May use global locking; then new changes replace previous attempts

-   Only when all agree, make the actual change (including adding/removing
    Puzzle Pieces)

-   Permit as few network disruptions as possible

-   Network frames in transit may have been parsed with an older model!

Funnel Tunnel in its Networking Context
---------------------------------------

TODO

### Network sockets

TODO

### Library-style sockets

TODO

### UNIX-domain sockets and NamedPipes

TODO

### Acting alongside inetd, systemd et al

TODO

### Event-reporting socket

TODO

A few Possible Puzzle Pieces
----------------------------

These are a few Puzzle Pieces that we could build into a Funnel Tunnel. The may
be implemented in Go or taken from external programs, linked into Go and
connected as a Puzzle Piece.

The design should be suitable for doing this kind of things, because they are
part of the aforementioned use cases. In all cases, the idea is that a Puzzle
Piece need only focus on one network mapping aspect, and the Funnel Tunnel takes
care of linking it up in as many ways as imaginable.

Some pieces may declare a “protocol” specifically meant to define the
pluggability, such as “RoHC”, which can in turn be sent over various carriers.

### Sockets

For the various kinds of links to the environment, ranging from tunnel devices
to TCP sockets supplied through a systemd (or comparable) interface, there are
one-sided connectors that internally take care of connections to the local
network infrastructure.

### SCTP Concentrator

This Puzzle Piece links in UDP and TCP sources, and even (streams from) SCTP
sources. A basic function is packing and unpacking things, and more advanced
(protocol specific) Puzzle Pieces will be possible for applications such as
mDNS. General support for resend detection is available.

### GSS-API protection

A stream (such as a TCP or SCTP stream) is wrapped into GSS-API for
authentication and (possibly) encryption. The are sent as a sequence of frames.

### Frame Sequence Submission

We might want to transport frames up to 4 GB. For transports that support frames
with a lower maximum size, frames are sent in a way that supports later
reconnection.

For transports that do not support frames, they are prefixed with 32 bits of
frame size and the frame is then appended.

#### TLS Pool

This Puzzle Piece encapsulates UDP, TCP or SCTP into TLS using libtlspool and
the TLS Pool daemon. The local/remote identities are available for the level
just as local/remote addresses and ports with other network protocol layers.

### 6bed4

This Puzzle Piece offers IPv6 connectivity by sends it over UDP and IPv4. The
plugin attempts direct connections with the remote peer’s IPv4 address when
possible, and falls back to a tunnel server when it has to.

Special versions exist for a number of protocols that run over IPv6; these
define protocol-specific ways of trying direct contact, such as trying to send a
first SYN in a TCP flow to the remote peer, and only pass resends of that SYN
through the backend server.

### SIPproxy64

This Puzzle Piece connects the stacks SIP/UDP/IPv4 with RTP/UDP/IPv4 to the
stacks SIP/UDP/IPv6 with RTP/UDP/IPv6

### RTP encryption

This (not yet specified) Puzzle Piece connects a SIP+RTP stack without
encryption to a SIP+RTP stack with encryption. In several cases, the SDP
extension will be modified to hold a key under some protection, and the RTP
stream is accordingly encrypted/decrypted.

### Robust Header Compression

This is a series of Puzzle Pieces, each implementing a specific RoHC profile.
For example, there is a compressor for RTP/UDP/IP and another for ESP/IP.

### Codec Freedom

Various components, to be determined, for sending other protocols over a phone
connection, but within the standard codec available (G.711, G.729 or GSM).
