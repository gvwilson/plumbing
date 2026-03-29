# Glossary

## A

<span id="access_control">access control</span>
:   FIXME

<span id="ascii">ASCII character encoding</span>
:   A standard way to represent the characters commonly used in the Western
    European languages as 7-bit integers, now largely superceded by
    [Unicode](#unicode).

<span id="allocation_block">allocation block</span>
:   The minimum unit of disk space that a [filesystem](#filesystem) allocates to a file.
    Even a file that contains a single byte occupies one full allocation block,
    so a file's actual disk usage is always a multiple of the block size.

<span id="auth_challenge">authentication challenge</span>
:   A piece of data sent by a server to a client that the client must transform
    using a secret (such as a [private key](#private_key)) to prove its identity
    without transmitting the secret itself.

<span id="authentication">authentication</span>
:   The act of establishing one's identity.

<span id="authorization">authorization</span>
:   The act of establishing that one has a right to access certain information.

## B

<span id="background_process">background a process</span>
:   To disconnect a [process](#process) from the terminal but keep it
    running.

<span id="ball_and_stick">ball-and-stick model</span>
:   FIXME

<span id="base64">base64 encoding</span>
:   A representation of binary data that represents each group of 6 bits
    as one of 64 printable characters.

<span id="block_device">"block device"</span>
:   FIXME

<span id="block_filesystem">"block (in filesystem)"</span>
:   FIXME

<span id="buffer_noun">buffer (noun)</span>
:   An area of memory used to hold data temporarily.

<span id="buffer_verb">buffer (verb)</span>
:   To store something in memory temporarily,
    e.g., while waiting for there to be enough data to make an I/O operation worthwhile.

## C

<span id="cache">cache</span>
:   To store a copy of data locally in order to speed up access,
    or the data being stored.

<span id="callback_function">callback function</span>
:   A function A that is passed to another function B so that B can call it at
    some later point.

<span id="capability">capability</span>
:   FIXME

<span id="certificate">certificate</span>
:   A digitally-signed document that binds a public key to an identity (such as a domain name).
    Used in [TLS](#tls) to allow clients to verify that they are communicating
    with the intended server.

<span id="certificate_authority">certificate authority (CA)</span>
:   An organization trusted to sign [certificates](#certificate),
    vouching that the public key in the certificate belongs to the claimed identity.
    Browsers and operating systems ship with a list of trusted root CAs.

<span id="certificate_issuer">certificate issuer</span>
:   The [certificate authority](#certificate_authority) that signed and issued a [certificate](#certificate).

<span id="certificate_subject">certificate subject</span>
:   The identity (typically a domain name or organization) that a [certificate](#certificate) belongs to.

<span id="character_device">"character device"</span>
:   FIXME

<span id="character_encoding">character encoding</span>
:   A way to represent characters as bytes. Common examples include [ASCII](#ascii)
    and [UTF-8](#utf_8).

<span id="child_process">child process</span>
:   A [process](#process) created by another process,
    which is called its [parent process](#parent_process).

<span id="cleartext">cleartext</span>
:   Text that has not been [encrypted](#encryption).

<span id="client">client</span>
:   A program such as a browser that sends requests to a server and does something with the response.

<span id="command_interpolation">command interpolation</span>
:   FIXME

<span id="concurrency">concurrency</span>
:   The ability of different parts of a system to take action at the same time.

<span id="copy_on_write">copy-on-write</span>
:   FIXME

## D

<span id="daemon">daemon</span>
:   A long-lived process managed by an operating system
    that provides a service such as printer management to other processes.

<span id="dependency_conflict">dependency conflict</span>
:   The situation that arises when two packages require incompatible versions of a third package.

<span id="device">device</span>
:   FIXME

<span id="digital_certificate">digital certificate</span>
:   A digitally-signed document that binds a [public key](#public_key) to an identity.
    See also [certificate](#certificate).

<span id="digital_signature">digital signature</span>
:   A value computed from some data using a [private key](#private_key) that allows anyone
    with the corresponding [public key](#public_key) to verify that the data has not been
    tampered with and was signed by the holder of the private key.

<span id="dns">Domain Name System (DNS)</span>
:   The distributed database that translates human-readable hostnames such as
    `example.com` into numeric IP addresses.

<span id="docker">Docker</span>
:   A tool for creating and managing isolated computing environments.

<span id="docker_container">Docker container</span>
:   A particular running (or runnable) instance of a [Docker image](#docker_image).

<span id="docker_image">Docker image</span>
:   A package containing the software and supporting files [Docker](#docker) needs
    to run an application in isolation.

<span id="docker_layer">layer (of Docker image)</span>
:   FIXME

<span id="docker_tag">tag (a Docker image)</span>
:   FIXME

<span id="dockerfile">Dockerfile</span>
:   The name usually given to a file containing commands to build a [Docker image](#docker_image).

<span id="dynamic_content">dynamic content</span>
:   Web site content that is generated on the fly.
    Dynamic content is usually customized according to the requester's identity,
    [query parameter](#query_parameter),
    etc.

## E

<span id="encryption">encryption</span>
:   The process of converting data from a representation that anyone can read
    to one that can only be read by someone with the right algorithm and/or key.

<span id="env_var">environment variable</span>
:   A [shell variable](#shell_var) that is inherited by [child processes](#child_process)

<span id="exit_status">exit status</span>
:   FIXME

## F

<span id="filesystem">filesystem</span>
:   The set of files and directories making up a computer's permanent storage,
    or the software component used to manage them.

<span id="flush">flush</span>
:   To move data from a [buffer](#buffer_noun) to its intended destination immediately.

<span id="foreground_process">foreground a process</span>
:   To reconnect a [process](#process) to the terminal after it has
    been [backgrounded](#background_process) or [suspended](#suspend_process).

<span id="fork_process">fork (a process)</span>
:   To create a duplicate of an existing [process](#process),
    typically in order to run a new program.

## G

<span id="gid">group ID (GID)</span>
:   FIXME

## H

<span id="hash">hash</span>
:   FIXME

<span id="hmac">HMAC (Hash-based Message Authentication Code)</span>
:   A type of message authentication code that combines a cryptographic hash function
    with a secret key, used to verify both the integrity and authenticity of a message.

<span id="host_key">host key</span>
:   A [key pair](#key_pair) that uniquely identifies an SSH server.
    Clients store the server's public host key after the first connection
    and reject connections if it changes unexpectedly.

<span id="hostname">hostname</span>
:   A human-readable name for a computer on a network.

<span id="http">HTTP</span>
:   full: HyperText Transfer Protocol
    The protocol used to exchange information between browsers and websites,
    and more generally between other clients and servers.
    Communication consists of [requests](#http_request) and [responses](#http_response).

<span id="http_header">header (of HTTP request or response)</span>
:   A name-value pair at the start of an [HTTP request](#http_request) or [response](#http_response).
    Headers are used to specify what data formats the sender can handle,
    the date and time the message was sent,
    and so on.

<span id="http_method">HTTP method</span>
:   The verb in an [HTTP request](#http_request) that defines what the [client](#client) wants to do.
    Common methods are `GET` (to get data) and `POST` (to submit data).

<span id="http_request">HTTP request</span>
:   A precisely-formatted block of text sent from a [client](#client) such as a browser
    to a [server](#server)
    that specifies what resource is being requested,
    what data formats the client will accept, etc.

<span id="http_response">HTTP response</span>
:   A precisely-formatted block of text sent from a [server](#server)
    back to a [client](#client) in reply to a [request](#http_request).

<span id="http_status_code">HTTP status code</span>
:   A numerical code that indicates what happened when an [HTTP request](#http_request) was processed,
    such as 200 (OK),
    404 (not found),
    or 500 (internal server error).

<span id="https">HTTPS</span>
:   HTTP over [TLS](#tls). Encrypts all traffic between client and server
    and allows the client to verify the server's identity using a [certificate](#certificate).

## I

<span id="inode">inode</span>
:   FIXME

<span id="internal_fragmentation">internal fragmentation</span>
:   The disk space wasted when a file does not completely fill its last [allocation block](#allocation_block).
    A 1-byte file in a filesystem with 4 KiB blocks wastes 4,095 bytes.

<span id="ip_address">IP address</span>
:   A numerical label assigned to each device on a network that uses the Internet Protocol.
    IPv4 addresses are 32 bits (e.g., `192.168.1.1`); IPv6 addresses are 128 bits.

## J

<span id="journald">journald</span>
:   The logging daemon that is part of systemd on Linux.
    It collects log messages from the kernel, services, and applications
    and stores them in a structured binary format queryable with `journalctl`.

<span id="json">JavaScript Object Notation (JSON)</span>
:   A way to represent data by combining basic values like numbers
    and character strings in lists and key-value structures. Unlike
    other formats, it is unencumbered by a syntax for writing comments.

## K

<span id="key_pair">key pair</span>
:   A matched pair of cryptographic keys consisting of a [private key](#private_key),
    which is kept secret, and a [public key](#public_key), which can be shared freely.
    Data encrypted with one key can only be decrypted with the other.

## L

<span id="link_hard">hard link (in filesystem)</span>
:   FIXME

<span id="link_sym">symbolic link (in filesystem)</span>
:   FIXME

<span id="lint">lint</span>
:   FIXME

<span id="local_server">local server</span>
:   A [server](#server) running on the programmer's own computer,
    typically for development purposes.

<span id="localhost">localhost</span>
:   A special [host name](#hostname) that identifies
    the computer that the software is running on.

<span id="log_formatter">log formatter</span>
:   A component of a logging system that controls the text layout of each log message,
    including fields such as timestamp, level, and message text.

<span id="log_handler">log handler</span>
:   A component of a logging system that decides where log messages are sent,
    such as to the terminal, a file, or a remote service.

<span id="log_level">log level</span>
:   A label indicating the severity or importance of a log message.
    Common levels in order of increasing severity are
    DEBUG, INFO, WARNING, ERROR, and CRITICAL.

<span id="logger">logger</span>
:   A named channel through which log messages flow in a structured logging system.
    Loggers can be given different levels and [handlers](#log_handler).

## M

<span id="mime_type">MIME type</span>
:   A standard that defines types of file content,
    such as `text/plain` for plain text and `image/jpeg` for JPEG images.

<span id="mitm_attack">man-in-the-middle attack</span>
:   An attack in which an adversary secretly intercepts and possibly alters
    communications between two parties who believe they are talking directly to each other.

<span id="mount">mount</span>
:   FIXME

## N

<span id="name_collision">name collision</span>
:   The problem that occurs when two different applications use the same name
    for different things.

## O

<span id="octal">octal</span>
:   FIXME

<span id="operating_system">operating system (OS)</span>
:   A program whose job is to manage the hardware of a computer.
    Other programs interact with the OS through [system calls](#system_call).

## P

<span id="parent_process">parent process</span>
:   A [process](#process) which has created one or more other processes,
    which are called its [child processes](#child_process).

<span id="passphrase">passphrase</span>
:   A password used to encrypt a [private key](#private_key) when it is stored on disk.
    Using a passphrase means a stolen key file cannot be used without it.

<span id="path">path (in filesystem)</span>
:   An expression that refers to a file or directory in a filesystem.

<span id="port">port</span>
:   A logical endpoint for communication,
    like a phone number in an office building.

<span id="private_key">private key</span>
:   The secret half of a [key pair](#key_pair).
    The private key must never be shared; it is used to decrypt messages encrypted
    with the corresponding [public key](#public_key) or to create [digital signatures](#digital_signature).

<span id="process">process</span>
:   A running instance of a program.

<span id="process_id">process ID</span>
:   The unique numerical identifier of a running [process](#process).

<span id="process_tree">process tree</span>
:   The set of processes created directly or indirectly by one process
    and the [parent](#parent_process)-[child](#child_process) relationships between them.

<span id="public_key">public key</span>
:   The non-secret half of a [key pair](#key_pair).
    The public key can be shared freely; it is used to encrypt messages intended for
    the holder of the corresponding [private key](#private_key) or to verify [digital signatures](#digital_signature).

## Q

<span id="query_parameter">query parameter</span>
:   A key-value pair included in a URL that the server may use to modify or customize its response.

## R

<span id="refactor">refactor</span>
:   To reorganize code without changing its overall behavior.

<span id="resume_process">resume (a process)</span>
:   To continue the execution of a [suspended](#suspend_process) [process](#process).

<span id="resolve_path">resolve (a path)</span>
:   To translate a [path](#path) into the canonical name of the file or directory it refers to.

<span id="robustness">robustness</span>
:   The property of a program or system that continues to function correctly
    across a wide range of inputs, conditions, and execution orderings.

<span id="root_directory">root directory</span>
:   The top-most directory in the [filesystem](#filesystem)
    that contains all other directories and files.

<span id="root_user">root (user account)</span>
:   The usual ID of the [superuser](#superuser) account on a computer.

## S

<span id="salt">salt</span>
:   A random value added to a password before hashing it,
    so that two users with the same password will have different stored hashes
    and precomputed rainbow-table attacks are ineffective.

<span id="sandbox">sandbox</span>
:   An isolated computing environment in which operations can be executed safely.

<span id="server">server</span>
:   A program that waits for requests from [clients](#client)
    and sends them data in response.

<span id="session_key">session key</span>
:   A symmetric encryption key generated for a single [TLS](#tls) session
    and discarded afterward.
    Both client and server derive the same session key during the [TLS](#tls) handshake
    without transmitting it directly.

<span id="shell">shell</span>
:   A program that allows a user to interact with a computer's operating system
    and other programs through a textual user interface.

<span id="shell_script">shell_script</span>
:   A program that uses [shell](#shell) commands as its programming language.

<span id="shell_var">shell variable</span>
:   A variable set and used in the [shell](#shell).

<span id="signal">signal</span>
:   A message sent to a running [process](#process) separate from its
    normal execution, such as an interrupt or a timer notification.

<span id="signal_handler">signal handler</span>
:   A [callback function](#callback_function) that is called when
    a [process](#process) receives a [signal](#signal).

<span id="socket">socket</span>
:   An endpoint for two-way communication between processes,
    either on the same machine or across a network.
    Sockets make network I/O look similar to file I/O.

<span id="source_shell">source (in shell script)</span>
:   To run one [shell script](#shell_script) in the same process as another.

<span id="ssh">SSH (Secure Shell)</span>
:   A network protocol and tool for logging into remote machines and running commands on them.
    All traffic is encrypted, and the server's identity is verified using a [host key](#host_key).

<span id="ssh_tunnel">SSH tunnel</span>
:   A secure channel created by SSH that forwards network traffic from a local port
    to a port on (or reachable from) the remote machine.

<span id="static_file">static file</span>
:   Web site content that is stored as a file on disk that is served as-is.
    Serving static files is usually faster than generating [dynamic content](#dynamic_content),
    but can only be done if what's wanted is unchanging and known in advance.

<span id="stream">stream</span>
:   A sequence of data elements made available one at a time,
    such as the bytes read from a file or received over a network connection.

<span id="structured_logging">structured logging</span>
:   A logging approach in which each log record is written as a machine-readable
    data structure (typically JSON) rather than as free-form text,
    making logs easier to filter and analyze programmatically.

<span id="superuser">superuser</span>
:   An administrative account on a computer that has permission
    to see, change, and run everything.

<span id="suspend_process">suspend (a process)</span>
:   To pause the execution of a [process](#process) but leave it intact so that it
    can [resume](#resume_process) later.

<span id="system_call">system call</span>
:   A call to one of the functions provided by an [operating system](#operating_system).

## T

<span id="tls">TLS (Transport Layer Security)</span>
:   A cryptographic protocol that provides encrypted, authenticated communication
    over a network. Used by [HTTPS](#https) and [SSH](#ssh), among others.
    Formerly known as SSL.

<span id="tls_ssl">TLS/SSL</span>
:   See [TLS](#tls). SSL (Secure Sockets Layer) is the older name for the protocol
    now standardized as TLS; the two terms are often used interchangeably.

## U

<span id="uid">user ID (UID)</span>
:   FIXME

<span id="unicode">Unicode</span>
:   A standard that defines numeric codes for many thousands of characters and
    symbols. Unicode does not define how those numbers are stored; that is
    done by standards like [UTF-8](#utf_8).

<span id="user_group">user group</span>
:   FIXME

<span id="utf_8">UTF-8</span>
:   A way to store the numeric codes representing [Unicode](#unicode)
    characters that is backward-compatible with the older [ASCII](#ascii) standard.

<span id="uuid">Universally Unique Identifier (UUID)</span>
:   FIXME

## V

<span id="virtual_env">virtual environment</span>
:   A set of libraries, applications, and other resources that are isolated from the main system
    and other virtual environments.

## W

<span id="web_scraping">web scraping</span>
:   The act of extracting data from HTML pages on the web.

<span id="wrap_object">wrap (an object)</span>
:   To create a new object that delegates most operations to an existing object
    while adding or modifying specific behavior,
    such as wrapping a plain socket with a TLS layer.

## X

## Y

## Z
