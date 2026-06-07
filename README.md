# portscanner

A lightweight TCP port scanner written in Go, built on the red-team tool framework: **Target → Communicate → Evade → Results**.

No dependencies. Single binary. Drop it anywhere.

---

## Features

- TCP connect scan with optional service banner grabbing
- Genuine low-and-slow stealth mode (single worker + wide randomised jitter)
- Port shuffle to break sequential scan signatures
- Full TLS handshake on TLS ports (443, 8443, 993, 995) to avoid plaintext-on-TLS alerts
- Flexible port spec — top 20, comma list, range, or a mix
- Concurrent workers for fast sweeps
- Clean sorted output with elapsed time

---

## Build

```bash
git clone https://github.com/mug1sh4/portscanner
cd portscanner
go build -o portscanner portscanner.go
```

Requires Go 1.20+. No external libraries.

Cross-compile for a target machine (useful when you can't install tools on the box):

```bash
# Linux target from any OS
GOOS=linux GOARCH=amd64 go build -o portscanner portscanner.go

# Windows target
GOOS=windows GOARCH=amd64 go build -o portscanner.exe portscanner.go
```

---

## Usage

```
./portscanner [flags] <target>

Flags:
  -t string       target host or IP (or pass as trailing argument)
  -p string       ports: 'top', list (22,80,443), range (1-1024), or mix (default "top")
  -c int          concurrent workers (default 50)
  -timeout        per-port connection timeout (default 2s)
  -stealth        low-and-slow: forces workers=1 + wide random jitter
  -jitter         max random pause before each dial, e.g. 5s
  -banner         grab service banners after connecting (LOUD — off by default)
```

---

## Examples

```bash
# Quick scan of the top 20 ports
./portscanner 10.0.0.5

# Fast sweep of the first 1024 ports
./portscanner -p 1-1024 -c 200 10.0.0.5

# Specific ports with banner grabbing
./portscanner -p 22,80,443,3389 -banner 10.0.0.5

# Stealth mode — one port at a time, 0–8s random gap between each
./portscanner -stealth 10.0.0.5

# Stealth with a custom jitter window
./portscanner -stealth -jitter 15s -p 22,80,443 10.0.0.5

# Full port range, medium concurrency
./portscanner -p 1-65535 -c 100 -timeout 1s 10.0.0.5
```

---

## Sample Output

```
[*] 10.0.0.5  ports=20  workers=50  jitter<=0s  banners=false  mode=normal

[+]    22/tcp  OPEN
[+]    80/tcp  OPEN
[+]   443/tcp  OPEN
[+]  3389/tcp  OPEN

[*] done: 4 open of 20 scanned in 312ms
```

With `-banner`:

```
[+]    22/tcp  OPEN   SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6
[+]    80/tcp  OPEN   HTTP/1.1 200 OK        nginx/1.24.0
[+]   443/tcp  OPEN   HTTP/1.1 200 OK        nginx/1.24.0
```

---

This tool is structured around a 4-step build pattern for offensive security tools:

| Step | What it does | Where in the code |
|------|-------------|-------------------|
| **Target** | Accept any host, any port spec — fully reusable | `parseFlags()`, `parsePorts()` |
| **Communicate** | Ask the OS to open a socket; read the service response | `scanPort()`, `grabBanner()` |
| **Evade** | Shuffle ports, jitter timing, TLS on TLS ports, quiet by default | `scan()`, `-stealth` flag |
| **Results** | Sorted, clean output with counts and elapsed time | `main()` output block |

---

## Limitations

This is a **TCP connect scanner** — it completes the full three-way handshake. That means:

- The remote service can log the connection (unlike a SYN/half-open scan)
- Stateful IDS (Snort, Suricata, Zeek) can still correlate hits over longer windows
- Cannot distinguish **closed** (RST) from **filtered** (no response) — both look the same
- No UDP scanning, no OS detection, no scripting engine

For full stealth, a SYN scan (raw sockets + root) is the next step.

---

## Legal

Only scan hosts you own or have **explicit written authorisation** to test. Unauthorised port scanning may be illegal in your jurisdiction.

---

## Author

[@mug1sh4](https://github.com/mug1sh4) — built as part of a tooling series.
