Funnel Tunnel in Go
===================

>   *Is Go a suitable language for the Funnel Tunnel? Yes, in many respects. No,
>   in other. Let’s give it a try.*

Pros
----

-   Systems programming language, integrates properly with the OS environment

-   Excellent message passing semantics; CSP works better than the C calling
    convention

-   Fine-grained concurrency through coroutines scheduled in (invisible) threads

-   No excessive use of Garbage Collection (simply reuse []byte buffers)

-   Statically typed, object-based with some flexibility through interfaces

-   Data types for byte-level control, ranging to user-friendly maps and slices

Cons
----

-   No exception handling?

-   No generic types (yet)

-   No message broadcast semantics (CCS?)

-   Library: No [support for
    SCTP](<http://www.cyberroadie.org/AsiaBSD2013_Paper.pdf>) is provided today

-   Library: No support for ancillary data on UNIX domain sockets (needed for
    TLS Pool)

    -   Note that `syscall/sockmsg_linux.go` does have hints in that direction

    -   Indeed, `UnixRights` / `ParseUnixRights` map between `fd[]` and `byte[]`

    -   Plus, `Sendmsg()` and `Recvmsg()` handle `[]byte oob` data

    -   Making a TLS Pool wrapper should not be difficult to do at all

Funnel Tunnel modular API
-------------------------

-   Configurations apply functions to one TCP connection or all traffic to a
    host, or...

-   Configuration can be written in Go as a simple top-level program

-   Funnel Tunnel then acts like a powerful traffic manipulation library

-   Various tunnels and network stacking facilities are mere plugin modules to
    this library

-   Modules in the Funnel Tunnel will translate network-stack-left into
    network-stack-right

-   Examples are insertion of tunnels and security layers

-   Other examples are more clever mDNS, SNMP, RADIUS through reliable delivery
    of SCTP

-   API will define “sockets” based on light-weight CSP messaging

-   These “sockets” send/receive their network-stack-selection as slices

-   The Funnel Tunnel connects these network components to the general
    mechanisms

-   Implementations may use user-level coding, tunnel devices, kernels, daemons,
    ...

-   I’d say we go for it!
