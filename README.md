# rboz
Fast multi-threaded Python HTTP/S stresser using raw sockets (SOCKS4/SOCKS5/HTTP/HTTPS proxies).

> **Disclaimer:** For educational and authorised testing purposes only.
> The author is not responsible for any illegal use.

---

## Requirements

- Python **3.12+**
- [PySocks](https://pypi.org/project/PySocks/) – `pip install PySocks`

---

## Features

- Raw-socket HTTP/1.1 keep-alive (no `requests`, no `urllib3`)
- Multi-threaded via `ThreadPoolExecutor`
- Proxy support: `direct`, `http`, `https`, `socks4`, `socks5`
- GET and POST methods
- Round-robin proxy rotation (`itertools.cycle`)
- Live stats every N seconds; final summary on exit
- Graceful shutdown on `SIGINT` / `SIGTERM` / Ctrl-C

---

## Usage

### Direct connection
```bash
python rboz.py <url> <threads> <method>
```
```bash
python rboz.py http://example.com 50 get
```

### With proxies
```bash
python rboz.py <url> <threads> <method> --proxy-type <type> --proxy-file <file>
```
```bash
python rboz.py http://example.com 50 get --proxy-type socks5 --proxy-file proxies.txt
```

---

## Arguments

| Argument | Default | Description |
|---|---|---|
| `url` | — | Target URL |
| `workers` | — | Number of concurrent threads |
| `method` | — | `get` or `post` |
| `--proxy-type` | `direct` | `direct` / `http` / `https` / `socks4` / `socks5` |
| `--proxy-file` | — | Path to proxy list (required unless `direct`) |
| `--user-agents` | `default/useragents.txt` | File of User-Agent strings |
| `--connect-timeout` | `10.0` | TCP connect timeout (seconds) |
| `--rw-timeout` | `15.0` | Socket read/write timeout (seconds) |
| `--reqs-per-conn` | `100` | Requests per keep-alive connection |
| `--stats-interval` | `5.0` | Live stats print interval (seconds) |
| `--inter-request-sleep` | `0.0` | Sleep between requests on same connection |
| `--fail-sleep` | `0.5` | Sleep after a failed connection |

---

## Proxy file format

One proxy per line, `host:port`:
```
1.2.3.4:1080
5.6.7.8:8080
# lines starting with # are ignored
```

---

## Performance notes

- Direct mode with `--inter-request-sleep 0` is the fastest path – each worker loops at full socket speed.
- Proxy mode is limited by proxy latency; use more workers to compensate.
- Each connection is reused for `--reqs-per-conn` requests before being recycled.
- Failed connections back off for `--fail-sleep` seconds to avoid busy-looping.
