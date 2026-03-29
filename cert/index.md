# Certificates

## Securing the Server

-   Two components needed to serve content over HTTPS:
    -   Give the server a [digital certificate](g:digital_certificate) so that it can establish its identity
    -   Have the client communicate with it via [HTTPS](g:https) (the encrypted version of HTTP)
-   Certificate has private and public components
    -   Answer questions with `.` to skip optional fields
    -   `-nodes` is short for "no DES" so the certificate has no interactive password

```{data-file="create_pem.sh"}
openssl req -x509 -newkey rsa:4096 -sha256 -days 1000 -nodes \
	-keyout server_first_key.pem -out server_first_cert.pem
```

```{data-file="site/server_first_cert.pem"}
-----BEGIN CERTIFICATE-----
MIIE+zCCAuOgAwIBAgIUfqV4WLyo+hCSjqfLxt8gxP2SqWMwDQYJKoZIhvcNAQEL
BQAwDTELMAkGA1UEBhMCQ0EwHhcNMjQwMjI1MjI1OTMyWhcNMjYxMTIxMjI1OTMy
WjANMQswCQYDVQQGEwJDQTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIB
AL3+7HzDMcRLBUONmN65OSrk23ZitOXenyYaOmbkKH//lM6qMVDRJUA5FRiVuTV8
lx8uw2QKwCfgpcf3y6jk1L3p+eOY333BE38m7GCjysTMc7//aDxdEYu8rkzCeG/G
…
-----END CERTIFICATE-----
```

-   Modify the file server to listen for secure connections on port 1443
    -   [Wrap](g:wrap_object) the server's [socket](g:socket) with a layer of security
    -   Everything else stays the same

```{data-file="file_server_secure.py:main"}
if __name__ == "__main__":
    server_address = ("", 1443)
    sandbox = sys.argv[1]
    certfile = sys.argv[2]
    keyfile = sys.argv[3]

    os.chdir(sandbox)

    # If check_hostname is True, only the hostname that matches the certificate
    # will be accepted
    ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
    ssl_context.check_hostname = False
    ssl_context.load_cert_chain(certfile=certfile, keyfile=keyfile)

    server = HTTPServer(server_address, RequestHandler)
    server.socket = ssl_context.wrap_socket(server.socket, server_side=True)

    print(f"serving at {server_address} in {os.getcwd()}...")
    server.serve_forever()
```

-   Run it like this

```{data-file="file_server_secure.sh"}
python src/file_server_secure.py site server_first_cert.pem server_first_key.pem
```

-   Point browser at `https://localhost:1443/motto.txt`
    -   Must have *both* `https` *and* the port `1443`

## Securing the Client

-   Try `requests.get("https://localhost:1443/test.txt")`

```{data-file="requests_secure_no_verify.out"}
ERROR: HTTPSConnectionPool(host='localhost', port=1443):
Max retries exceeded with url: /test.txt
(Caused by SSLError(SSLCertVerificationError(1,
'[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate (_ssl.c:1000)')))
```

-   Fair enough: `requests` should not trust self-signed certificates by default
-   Can tell it not to check at all by passing `verify=False` but turning off security is a bad idea
-   Try `requests.get("https://localhost:1443/test.txt", verify="cert.pem")`
    -   I.e., pass it the server's certificate

```{data-file="requests_secure_server_cert.out"}
ERROR: HTTPSConnectionPool(host='localhost', port=1443):
Max retries exceeded with url: /test.txt
(Caused by SSLError(SSLCertVerificationError(1,
"[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed:
Hostname mismatch, certificate is not valid for 'localhost'. (_ssl.c:1000)")))
```

-   What we actually need is to sign the server's certificate with a certificate from some authority,
    then tell `requests` it can use the authority's certificate to check the server's certificate
    -   Seems roundabout, but it allows us to use a few hundred trusted certificates to check everyone else's
-   `import certifi` and `certifi.where()` to see the PEM file that `requests` uses by default

## Starting Over

-   Pretend to be our own [certificate authority](g:certificate_authority) (CA)
-   Use our CA certificate to [sign](g:digital_signature) the server's certificate
    -   We can use the CA to sign any number of server certificates

```{data-file="create_signed_cert.sh"}
# create CA key and cert simultaneously
openssl req -x509 -newkey RSA -nodes -keyout CA.key -days 10 -out CA.pem -reqexts \
	v3_ca -subj "/C=CA/ST=ON/L=Toronto/O=Third Bit/OU=x509"

# create server key and cert
openssl req -new -newkey RSA -nodes -keyout server.key -out server.csr -batch \
	-reqexts v3_req -subj "/CN=localhost"

# sign server CSR with CA key
openssl x509 -req -days 10 -in server.csr -CAkey CA.key -CA CA.pem -CAcreateserial \
	-out server.pem -extfile extfile.txt
echo "certificates created"
```

-   Run the server with the newly-created certificate

```{data-file="file_server_signed.sh"}
python src/file_server_secure.py site server.pem server.key
```

-   Point `requests` at the CA certificate
    -   Which it then uses to check that the server's certificate is properly signed

```{data-file="requests_signed_cert.py"}
import requests

CA_FILE = "CA.pem"

print(f"trying request with verify={CA_FILE}")
try:
    r = requests.get("https://localhost:1443/motto.txt", verify=CA_FILE)
    print(f"SUCCESS: {r.status_code} {r.text}")
except Exception as exc:
    print(f"FAILURE: {exc}")
```

## This Is Hard {: .aside}

-   It took a full day and help from three other people to get this working
-   And the purpose of some of these options used when signing the server certificate is still unclear

```{data-file="extfile.txt"}
basicConstraints = CA:FALSE
subjectAltName = DNS:localhost
extendedKeyUsage = serverAuth
```

-   Some of this difficulty is intrinsic: security really is hard
-   Some is accidental complexity introduced by evolution over time

## Running Processes Together

-   Want to run a client and server side by side and capture their output
-   First attempt: start the server, wait one second, and run the client
-   When the client finishes, stop the script and its children
    -   Shuts down the server and anything else that may have started

```{data-file="run_2_sleep.sh"}
#!/usr/bin/env bash

# Save the process group ID of this script.
pgid=`ps -o pgid=$$`
echo "PGID $pgid"

# Trap a Ctrl-C SIGINT and kill everything running inside this script.
trap "pkill -TERM -g $pgid" INT

# Redirect server stderr to stdout and background the process.
$1 2>&1 &

# Wait one second.
sleep 1

# Redirect client stderr to stdout as well but run in foreground.
$2 2>&1

# Kill this script and its children (client and server) when client finishes.
pkill -TERM -g $pgid
```

-   Use this for the first script and something similar for the second

```{data-file="run_2_left.sh"}
for i in $(seq 1 1 3)
do
    echo "left $i"
    sleep 1
done
```

-   Run it

```{data-file="run_2_example.sh"}
$ bash src/run_2_sleep.sh "bash src/run_2_left.sh" "bash src/run_2_right.sh"
```

```{data-file="run_2_example.out"}
PGID  5299
left 1
left 2
right 1
left 3
right 2
right 3
```

## Partial Ordering {: .aside}

-   There are *choose(x+y, x)* ways to interleave two sequences of length *x* and *y*
-   Which is *(x+y)!/x!y!*
-   So two programs that open a file, write a line, and close the file
    can be interleaved in 20 different ways
-   A [robust](g:robustness) application works in all 20 ways
    -   Because sleeping for one second is no guarantee
        that another process has run far enough to open a socket

## Listing Open Files {: .aside}

-   Unix tries hard to make (almost) everything look like a file
    -   Read/write as a [stream](g:stream) or in [blocks](g:block_device)
-   In particular, a socket makes a network connection look like a file
-   Use `lsof` to list open files
    -   Operating system keeps track of what "files" a process is interacting with

```{data-file="run_lsof.sh"}
lsof -n -iTCP | head -n 10 | cut -c -72
```

```{data-file="run_lsof.out"}
COMMAND     PID  USER   FD   TYPE             DEVICE SIZE/OFF NODE
corespeec   585 tut    3u  IPv6 0x5f5fb0f69522963b      0t0  TCP
jetbrains  1609 tut   59u  IPv6 0x5f5fb0f69523863b      0t0  TCP
PowerChim 46752 tut    3u  IPv6 0x5f5fb0f69524ae3b      0t0  TCP
PowerChim 46752 tut    6u  IPv6 0x5f5fb0f695228e3b      0t0  TCP
Google    53670 tut   22u  IPv6 0x5f5fb0f695246e3b      0t0  TCP
Google    53670 tut   24u  IPv6 0x5f5fb0f69523d63b      0t0  TCP
Google    53670 tut   29u  IPv4 0x5f5fb0fb588afdd3      0t0  TCP
Google    53670 tut   33u  IPv6 0x5f5fb0f69524663b      0t0  TCP
Google    53670 tut   34u  IPv6 0x5f5fb0f69523de3b      0t0  TCP
```

-   Which means we can start a process and wait until it opens a particular port

## A Better Runner

```{data-file="run_and_wait.sh"}
# assume first arg is a set of TCP ports
# followed by one command per server listening to the port
# followed by client commands
# bash run_and_wait.sh "8000" "server 8000" "client"

PORTS="$1"
shift

CHILDREN=

await_port_free() {
    PORTNUM=$1
    while lsof -n -iTCP:${PORTNUM} ; do
        sleep 0.5
        printf "*"
    done
    printf "\nport $PORTNUM free\n"
}

await_port_listen() {
    PORTNUM=$1
    while ! lsof -n -iTCP:${PORTNUM}|grep -qw LISTEN ; do
        sleep 0.5
        printf "*"
    done
    printf "\nport $PORTNUM in LISTEN state\n"
}

on_exit(){
    # disable trap
    trap - exit int
    # gently kill every child
    kill -INT $CHILDREN &>/dev/null
    sleep 1
    # thorough cleanup
    pkill -TERM -g 0
}

# exiting or ^C runs on_exit
trap on_exit exit int

# server commands

for PORT in $PORTS; do
    await_port_free $PORT

    CMD="$1"
    shift
    $CMD &
    CHILDREN="$CHILDREN $!"

    await_port_listen $PORT
done

# client commands

for CMD in "$@"; do
    $CMD &
    CHILDREN="$CHILDREN $!"
done

# wait until any child process exits
while true; do
    for CHILD in $CHILDREN; do
        if ! kill -0 $CHILD &>/dev/null; then
            exit # to on_exit
        fi
    done
    sleep 0.5
done
```

-   There's a lot going on here
    -   Shell functions
    -   Using `shift` and `$!` to handle arguments
    -   Using `trap` to handle interrupts
    -   Using `printf` instead of `echo`
-   The most important part is the `while` loop
    -   `await_port_listen` function waits for someone to listen to a port
-   We can either learn more shell scripting
    or figure out how to do this in Python

## Introducing FastAPI

```{data-file="bird_server_fastapi.py"}
from fastapi import FastAPI
import os
import pandas as pd
import uvicorn
import sys

sandbox, filename = sys.argv[1], sys.argv[2]
os.chdir(sandbox)
df = pd.read_csv(filename)

app = FastAPI()

@app.get("/")
async def get_birds(year: int = None, species: str = None):
    result = df.copy()
    if species is not None:
        result = result[result["species_id"] == species]
    if year is not None:
        result = result[result["year"] == year]
    result["num"].fillna(0, inplace=True)
    return result.to_dict(orient="records")

uvicorn.run(app)
```

## At a Lower Level

```{data-file="socket_server.py"}
import socketserver

CHUNK_SIZE = 4096
HOST = "localhost"
PORT = 8000

RESPONSE = """HTTP/1.1 200 OK
Content-Length: 0
"""

class Handler(socketserver.BaseRequestHandler):

    def handle(self):
        print("server awaiting message")
        self.data = str(self.request.recv(CHUNK_SIZE), "utf-8")
        print(f"From {self.client_address[0]}: {len(self.data)} bytes\n{self.data}")
        self.request.sendall(bytes(RESPONSE, "utf-8"))


if __name__ == "__main__":
    server = socketserver.TCPServer((HOST, PORT), Handler)
    print("server about to handle request")
    server.handle_request()
    server.server_close()
    print("server done")
```

```{data-file="socket_server.out"}
S:  server about to handle request
S:  server awaiting message
S:  From 127.0.0.1: 154 bytes
S:  GET /motto.txt HTTP/1.1
S:  Host: localhost:8000
S:  User-Agent: python-requests/2.31.0
S:  Accept-Encoding: gzip, deflate
c:  client received: <Response [200]>
S:  Accept: */*
S:  Connection: keep-alive
S:
S:
S:  server done
```

-   A [socket](g:socket) is a channel between two computers
    -   Makes network I/O look (sort of) like file I/O
-   Our server handles a single request by:
    -   Waiting for a connection
    -   Creating a `Handler` object
    -   Calling its `handle` method
-   And then it closes
-   It responds to requests with an HTTP "OK"

## Sending an HTTP Request

```{data-file="socket_client.py"}
import socket

CHUNK_SIZE = 4096
HOST = "localhost"
PORT = 8000
PATH = "/motto.txt"
MESSAGE = f"GET {PATH} HTTP/1.1\r\nHost: {HOST}\r\n\r\n"
SERVER_ADDRESS = (HOST, 8000)

socket = socket.socket()
socket.connect(SERVER_ADDRESS)
socket.sendall(bytes(MESSAGE, "utf-8"))
print(f"client sent:\n{MESSAGE}")

first = socket.recv(CHUNK_SIZE)
first_str = str(first, "utf-8")
print(f"client received {len(first)} bytes:\n{first_str}")

second = socket.recv(CHUNK_SIZE)
second_str = str(second, "utf-8")
print(f"client received {len(second)} bytes:\n{second_str}")
```

```{data-file="socket_client.out"}
S:  ::ffff:127.0.0.1 - - [16/Feb/2024 17:53:40] "GET /motto.txt HTTP/1.1" 200 -
c:  client sent:
c:  GET /motto.txt HTTP/1.1
c:  Host: localhost
c:
c:
c:  client received 244 bytes:
c:  HTTP/1.0 200 OK
c:  Server: SimpleHTTP/0.6 Python/3.12.1
c:  Date: Fri, 16 Feb 2024 22:53:40 GMT
c:  Content-type: text/plain
c:  Content-Length: 58
c:  Last-Modified: Thu, 15 Feb 2024 12:58:01 GMT
c:
c:  Start where you are, use what you have, help who you can.
c:
c:  client received 0 bytes:
c:
```

-   Connect to a local server
-   Send an HTTP request with verb, path, HTTP version, and a host header
-   Read the response back in chunks
-   You can see why we use `requests`, right?

## Secure Sockets

```{data-file="https_client.py"}
import socket
import ssl
from headers import headers

CHUNK_SIZE = 4096
HOST = "gvwilson.github.io"
PORT = 443
PATH = "/web-tutorial/site/motto.txt"
MESSAGE = f"GET {PATH} HTTP/1.1\r\nHost: {HOST}\r\n\r\n"
SERVER_ADDRESS = (HOST, PORT)

socket = socket.socket()
context = ssl.create_default_context()
connection = context.wrap_socket(socket, server_hostname=HOST)

connection.connect(SERVER_ADDRESS)
connection.sendall(bytes(MESSAGE, "utf-8"))
print(f"client sent:\n{MESSAGE}")

first = connection.recv(CHUNK_SIZE)
first_str = headers(str(first, "utf-8"), "HTTP", "Content-Length", "Content-Type")
print(f"client received {len(first)} bytes:\n{first_str}\n")

second = connection.recv(CHUNK_SIZE)
second_str = str(second, "utf-8")
print(f"client received {len(second)} bytes:\n{second_str}")
```

```{data-file="https_client.out"}
client sent:
GET /web-tutorial/site/motto.txt HTTP/1.1
Host: gvwilson.github.io


client received 682 bytes:
HTTP/1.1 200 OK
Content-Length: 58
Content-Type: text/plain; charset=utf-8
…plus 22 more lines…

client received 58 bytes:
Start where you are, use what you have, help who you can.

```

-   There's a lot going on here (which is why we use `requests`)
-   Create a socket and wrap it with [TLS/SSL](g:tls_ssl) security
    -   Which GitHub requires
-   Like we said, this is why we use `requests`
