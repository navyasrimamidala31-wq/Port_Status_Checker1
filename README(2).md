# Port Status Checker

A Python command-line network utility that checks the TCP status of ports on a target host by attempting socket connections with a configurable timeout.

> **Use only on systems and networks you own or are explicitly authorized to test.** The included execution example scans `127.0.0.1` (localhost).

## Objective

Program network socket connections to audit the status of host ports and display whether each tested TCP port is **Open, Closed, or Filtered**.

## Step-by-Step Implementation

1. Import Python's `socket` library for TCP connections.
2. Accept a target host/IP address and port range from the command line.
3. Attempt TCP connections with a configurable timeout.
4. Classify the result:
   - **Open** — TCP connection succeeded.
   - **Closed** — TCP connection was refused.
   - **Filtered** — connection timed out or could not be completed.
5. Display a table and summary of the results.

## Source Code

Save the following as `port_status_checker.py`:

```python
import socket
import argparse
from concurrent.futures import ThreadPoolExecutor, as_completed

def check_port(host, port, timeout):
    """Return the port status: Open, Closed, or Filtered."""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(timeout)
    try:
        result = sock.connect_ex((host, port))
        if result == 0:
            return "Open"
        return "Closed"
    except socket.timeout:
        return "Filtered"
    except OSError:
        return "Filtered"
    finally:
        sock.close()

def scan_ports(host, start_port, end_port, timeout):
    results = {}
    ports = range(start_port, end_port + 1)

    with ThreadPoolExecutor(max_workers=20) as executor:
        jobs = {
            executor.submit(check_port, host, port, timeout): port
            for port in ports
        }
        for job in as_completed(jobs):
            port = jobs[job]
            results[port] = job.result()

    return results

def main():
    parser = argparse.ArgumentParser(
        description="Authorized TCP port status checker"
    )
    parser.add_argument("host", help="Target hostname or IP address")
    parser.add_argument("start_port", type=int, help="First TCP port")
    parser.add_argument("end_port", type=int, help="Last TCP port")
    parser.add_argument("--timeout", type=float, default=0.5,
                        help="Connection timeout in seconds")
    args = parser.parse_args()

    if not (1 <= args.start_port <= args.end_port <= 65535):
        parser.error("Port range must be between 1 and 65535.")

    print(f"Port Status Checker")
    print(f"Target : {args.host}")
    print(f"Range  : {args.start_port}-{args.end_port}")
    print("-" * 42)

    results = scan_ports(
        args.host, args.start_port, args.end_port, args.timeout
    )

    print(f"{'Port':<8}{'Status':<12}")
    print("-" * 20)
    for port in sorted(results):
        print(f"{port:<8}{results[port]:<12}")

    counts = {
        "Open": sum(v == "Open" for v in results.values()),
        "Closed": sum(v == "Closed" for v in results.values()),
        "Filtered": sum(v == "Filtered" for v in results.values()),
    }

    print("-" * 20)
    print(
        f"Summary: Open={counts['Open']}, "
        f"Closed={counts['Closed']}, Filtered={counts['Filtered']}"
    )

if __name__ == "__main__":
    main()
```

## How to Run

### Basic command

```bash
python port_status_checker.py 127.0.0.1 1 100
```

### With a custom timeout

```bash
python port_status_checker.py 127.0.0.1 1 100 --timeout 0.5
```

### Command format

```text
python port_status_checker.py <host> <start_port> <end_port> [--timeout SECONDS]
```

## Execution Log

The following execution was performed against **localhost (`127.0.0.1`)** using a temporary local TCP service on port `5050`.

![Port Status Checker execution log](execution_log.png)

### Execution command

```bash
python port_status_checker.py 127.0.0.1 5048 5052 --timeout 0.3
```

### Sample output

```text
Port Status Checker
Target : 127.0.0.1
Range  : 5048-5052
------------------------------------------
Port    Status      
--------------------
5048    Closed      
5049    Closed      
5050    Open        
5051    Closed      
5052    Closed      
--------------------
Summary: Open=1, Closed=4, Filtered=0
```

## Output Meaning

| Status | Meaning |
|---|---|
| Open | A TCP connection to the port succeeded. |
| Closed | The host responded, but the TCP connection was refused. |
| Filtered | The connection timed out or could not be completed within the configured timeout. |

## Technologies Used

- Python 3
- `socket`
- `argparse`
- `concurrent.futures`

## Project Files

```text
port_status_checker/
├── port_status_checker.py
├── execution_log.png
└── README.md
```

## Security / Responsible Use

This tool is intended for **authorized network auditing, coursework, and local testing**. Do not scan third-party systems without permission. The demonstration uses localhost so it does not require probing an external host.

## Author

**Sai Nikitha**
