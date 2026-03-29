# SSH

## What Is SSH?

-   [SSH](g:ssh) (Secure Shell) is a protocol for logging into remote machines and running commands on them
-   Runs over TCP port 22 by default
-   Replaced older tools like `telnet` and `rsh` that sent everything as plain text
-   Uses TLS-style encryption: all traffic between client and server is encrypted
    -   See the [Certificates](@/cert/) chapter for background on public-key cryptography

## Connecting with a Password

-   Simplest way to connect:

```{data-file="ssh_password.text"}
$ ssh tut@remote.example.com
tut@remote.example.com's password:
Last login: Mon Mar  3 09:14:22 2026
tut@remote:~$
```

-   The shell prompt changes to show we are now on the remote machine
-   Type `exit` or press Ctrl-D to close the connection
-   Password authentication is convenient but has risks:
    -   Weak passwords can be guessed by automated scanners
    -   Passwords can be captured if the user is tricked into connecting to the wrong host
    -   Most administrators disable it in favor of key-pair authentication

## Key-Pair Authentication

-   More secure alternative: generate a [key pair](g:key_pair)
    -   A [private key](g:private_key) that stays on your machine and is never shared
    -   A [public key](g:public_key) that you copy to remote machines
-   When connecting, the server issues a [challenge](g:auth_challenge) that only the holder of the private key can answer
    -   The private key never travels over the network

## Generating a Key Pair

```{data-file="ssh_keygen.text"}
$ ssh-keygen -t ed25519 -C "tut@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/tut/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/tut/.ssh/id_ed25519
Your public key has been saved in /Users/tut/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:abc123xyz456... tut@example.com
```

-   `ed25519` is a modern elliptic-curve algorithm; use it instead of the older `rsa`
-   `-C` adds a comment to identify the key (conventionally your email address)
-   The [passphrase](g:passphrase) encrypts the private key on disk
    -   If someone steals the file, they still cannot use the key without the passphrase
    -   An empty passphrase is convenient but less secure

## Copying the Public Key to a Server

```{data-file="ssh_copy_id.text"}
$ ssh-copy-id tut@remote.example.com
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
tut@remote.example.com's password:
Number of key(s) added: 1
```

-   `ssh-copy-id` appends the public key to `~/.ssh/authorized_keys` on the remote machine
-   After this, `ssh tut@remote.example.com` no longer asks for a password

## Host Key Verification

-   The first time you connect to a new server, SSH shows its [host key](g:host_key) fingerprint:

```{data-file="ssh_first_connect.text"}
$ ssh tut@remote.example.com
The authenticity of host 'remote.example.com (203.0.113.5)' can't be established.
ED25519 key fingerprint is SHA256:abc123xyz456...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'remote.example.com' to the list of known hosts.
```

-   SSH stores the fingerprint in `~/.ssh/known_hosts`
-   If the fingerprint changes on a later connection, SSH refuses to connect
    -   This protects against [man-in-the-middle attacks](g:mitm_attack)
    -   Could also mean the server was reinstalled — verify with the administrator before overriding

## `ssh-agent`

-   Typing the private key passphrase every time is tedious
-   `ssh-agent` holds decrypted keys in memory for the duration of a session

```{data-file="ssh_agent.text"}
$ eval "$(ssh-agent -s)"
Agent pid 12345

$ ssh-add ~/.ssh/id_ed25519
Enter passphrase for /Users/tut/.ssh/id_ed25519:
Identity added: /Users/tut/.ssh/id_ed25519 (tut@example.com)
```

-   `ssh-add` without arguments adds the default key (`~/.ssh/id_ed25519`)
-   On macOS, keys are stored in the system Keychain automatically

## The SSH Config File

-   `~/.ssh/config` lets you define shortcuts for frequently used hosts:

```{data-file="ssh_config.text"}
Host myserver
    HostName remote.example.com
    User tut
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes
```

-   `ssh myserver` now expands to `ssh -i ~/.ssh/id_ed25519 tut@remote.example.com`
-   `ForwardAgent yes` passes your local `ssh-agent` through to the remote machine
    -   Useful when the remote machine itself needs to connect to a third machine using your keys

## Copying Files

-   `scp` copies files over an SSH connection — same syntax as `cp` but with a host prefix:

```{data-file="scp_example.text"}
$ scp local_data.csv tut@remote.example.com:~/data/

$ scp tut@remote.example.com:~/results/output.csv .
```

-   `rsync` is more efficient for large or frequently updated files:
    -   Only transfers the parts that changed
    -   Can resume interrupted transfers

```{data-file="rsync_example.text"}
$ rsync -avz data/ tut@remote.example.com:~/data/
```

## SSH Tunnels

-   Sometimes a service is running on a remote machine but is not directly reachable
    -   E.g., a database that accepts connections only from `localhost`
-   Use local port forwarding to create a [tunnel](g:ssh_tunnel):

```{data-file="ssh_tunnel.text"}
$ ssh -L 5432:localhost:5432 tut@remote.example.com
```

-   This binds port 5432 on your local machine to port 5432 on the remote machine
-   Connections to `localhost:5432` on your laptop go through the SSH connection to the remote machine
-   Useful for database administration, Jupyter notebooks running on a cluster, etc.

## Programmatic SSH with `paramiko`

-   `paramiko` is a Python library for SSH connections:

```{data-file="paramiko_example.py"}
import paramiko

client = paramiko.SSHClient()
client.load_system_host_keys()
client.connect("remote.example.com", username="tut")

stdin, stdout, stderr = client.exec_command("ls ~/data")
for line in stdout:
    print(line.strip())

client.close()
```

-   `load_system_host_keys()` reads `~/.ssh/known_hosts` to verify the server
-   Use `paramiko.AutoAddPolicy()` only in trusted environments where you control the server

<section class="exercise" markdown="1">

## Exercise: Keys and Config

1.  Generate an `ed25519` key pair.
    What are the permissions on `~/.ssh/id_ed25519`?
    Why are those permissions necessary?

1.  Add an entry to `~/.ssh/config` for a machine you have access to
    and verify that `ssh <alias>` connects without specifying a username or hostname.

1.  Use `scp` to copy a small file to a remote machine and back.
    Then do the same with `rsync`.
    What differences do you notice?

</section>
