# Logging

## Why Not Just Print?

-   `print()` goes to standard output, which is often redirected or discarded
-   Logs need to go to the right destination (file, terminal, network) with the right level of detail
-   Structured logs can be filtered, searched, and fed into monitoring tools
-   A proper logging library lets you control verbosity without changing application code

## Log Levels

-   Log messages have a [log level](g:log_level) indicating their importance:

| Level      | When to use                                              |
| ---------- | -------------------------------------------------------- |
| `DEBUG`    | Detailed diagnostic information useful during development |
| `INFO`     | Confirmation that things are working as expected          |
| `WARNING`  | Something unexpected happened, but the program continues  |
| `ERROR`    | A serious problem; the program could not do something     |
| `CRITICAL` | The program cannot continue                               |

-   Setting the minimum level filters out less important messages
    -   Running with `INFO` in production suppresses `DEBUG` noise
    -   Temporarily switching to `DEBUG` is safer than adding print statements

## Python's `logging` Module

-   The standard library's `logging` module follows the Unix syslog tradition

```{data-file="basic_logging.py"}
import logging

logging.basicConfig(level=logging.DEBUG)

logging.debug("starting up")
logging.info("processing file: birds.csv")
logging.warning("species column has 3 missing values")
logging.error("cannot connect to database: connection refused")
```
```{data-file="basic_logging.out"}
DEBUG:root:starting up
INFO:root:processing file: birds.csv
WARNING:root:species column has 3 missing values
ERROR:root:cannot connect to database: connection refused
```

-   `basicConfig` is fine for scripts; real applications configure handlers explicitly

## Loggers, Handlers, and Formatters

-   `logging` has three main concepts:
    -   A [logger](g:logger) is a named channel for messages (e.g., `logging.getLogger("birdcount")`)
    -   A [handler](g:log_handler) decides where messages go (terminal, file, network)
    -   A [formatter](g:log_formatter) controls the text layout of each message

```{data-file="configured_logging.py"}
import logging

logger = logging.getLogger("birdcount")
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter(
    "%(asctime)s %(name)s %(levelname)s %(message)s"
))
logger.addHandler(handler)

logger.info("loaded %d records", 4821)
logger.warning("skipped %d records with missing species", 3)
```
```{data-file="configured_logging.out"}
2026-03-29 14:02:11,304 birdcount INFO loaded 4821 records
2026-03-29 14:02:11,305 birdcount WARNING skipped 3 records with missing species
```

-   Use `%s`-style formatting in log messages — the string is only built if the message will actually be emitted

## Rotating Log Files

-   Long-running services accumulate large log files
-   `RotatingFileHandler` caps file size and keeps a fixed number of old files:

```{data-file="rotating_log.py"}
import logging
from logging.handlers import RotatingFileHandler

# MAX_BYTES: rotate after 1 MiB; keep 3 old files
MAX_BYTES = 1_048_576
BACKUP_COUNT = 3

handler = RotatingFileHandler("app.log", maxBytes=MAX_BYTES, backupCount=BACKUP_COUNT)
handler.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(message)s"))

logger = logging.getLogger("birdcount")
logger.setLevel(logging.INFO)
logger.addHandler(handler)
```

-   When `app.log` reaches 1 MiB, it is renamed to `app.log.1`, and a new `app.log` is started
-   Older files become `app.log.2`, `app.log.3`; the oldest is deleted

## Structured (JSON) Logging

-   Plain text logs are easy to read but hard to search programmatically
-   [Structured logging](g:structured_logging) writes each message as a machine-readable record (usually JSON):

```{data-file="json_logging.py"}
import logging
from pythonjsonlogger import jsonlogger

handler = logging.StreamHandler()
handler.setFormatter(jsonlogger.JsonFormatter(
    "%(asctime)s %(name)s %(levelname)s %(message)s"
))

logger = logging.getLogger("birdcount")
logger.setLevel(logging.INFO)
logger.addHandler(handler)

logger.info("loaded records", extra={"count": 4821, "filename": "birds.csv"})
```
```{data-file="json_logging.out"}
{"asctime": "2026-03-29 14:02:11,304", "name": "birdcount", "levelname": "INFO", "message": "loaded records", "count": 4821, "filename": "birds.csv"}
```

-   JSON logs can be filtered with `jq`:

```{data-file="jq_filter.sh"}
cat app.log | jq 'select(.levelname == "ERROR")'
```

## Where Logs Come From

-   A production system produces logs from many sources simultaneously:
    -   Your application code
    -   The web server (e.g., nginx, gunicorn)
    -   The operating system (kernel messages, auth failures)
    -   Containers and orchestrators (Docker, Kubernetes)
-   On modern Linux systems, [journald](g:journald) collects all of these centrally
    -   Use `journalctl` to query them:

```{data-file="journalctl_example.sh"}
journalctl -u birdcount.service --since "1 hour ago"
```

-   On macOS, `log show` queries the unified logging system

## What Not to Log

-   Never log passwords, API tokens, or private keys — even at DEBUG level
-   Be careful with personally identifiable information (PII): names, email addresses, IP addresses
    -   Check your organization's data retention policies before logging user data
-   Do not log the full body of HTTP requests or responses unless you have scrubbed secrets first

```{data-file="safe_logging.py"}
import logging

logger = logging.getLogger("birdcount")

def authenticate(username, password):
    # safe: log the username but never the password
    logger.info("authentication attempt for user: %s", username)
    # ... authentication logic ...
```

<section class="exercise" markdown="1">

## Exercise: Adding Logging

1.  Take the bird server from the [HTTP](@/http/) chapter
    and replace all `print` statements with appropriate `logging` calls.
    Use `INFO` for normal requests and `ERROR` for failed ones.

1.  Add a `RotatingFileHandler` so that the server logs to `birdserver.log`
    and rotates after 512 KiB with two backup files.

1.  Use `jq` to extract all requests that returned a 404 status code from the log.

</section>
