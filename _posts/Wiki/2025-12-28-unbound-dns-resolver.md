---
title: "Using Unbound as a Local DNS Resolver (Beginner Guide)"
date: 2025-01-05
categories: [Networking, DNS]
tags: [dns, unbound, privacy, self-hosted]
---

## What is Unbound?

**Unbound** is a **validating, recursive DNS resolver**.
In simple terms, it:

- Recursively resolves domain names by querying root servers and authoritative name servers, rather than relying on third-party resolvers like Cloudflare or Google.
- Caches responses for faster subsequent lookups.
- Enhances privacy and security.

Instead of forwarding queries to services like Google DNS or Cloudflare, Unbound handles resolution independently, starting from the [DNS root servers](https://www.cloudflare.com/learning/dns/glossary/dns-root-server/).

## Why Use Unbound?

Here is why you could use Unbound DNS:

### Privacy
- Your DNS queries stay **local** on your server only
- No third-party logging your requests

### Performance
- DNS responses are **cached**
- Frequently visited sites load faster

### Security
- Supports **DNSSEC validation**
- Protects against DNS spoofing and poisoning

### Learning
- Great way to understand **how DNS really works** on a higher level than a simpler DNS server.

---

## What's a recursive DNS?

A **recursive DNS resolver** is a server that takes a DNS query from a client and does the full work of finding the answer. If it does not have the answer cached, it queries the DNS hierarchy on the client's behalf (root servers → TLD servers → authoritative name servers), follows referrals, and returns a single final response to the client. Recursive resolvers also cache results to speed up future lookups and can perform validation (for example, DNSSEC) for improved security.

**Resolution flow (high level):**

- Client → Recursive resolver
- Resolver → Root servers → TLD servers → Authoritative server (if needed)
- Authoritative → Resolver → Client (and the resolver caches the result)

Running Unbound as a recursive resolver means your machine performs these steps locally (improving privacy, control, and often performance) instead of forwarding queries to a third-party DNS service.


## Installing Unbound

### On Debian / Ubuntu

Update the local repositories and install with apt:

```bash
sudo apt update
sudo apt install unbound
sudo systemctl enable unbound
sudo systemctl start unbound
```

And check the service status:

```bash
systemctl status unbound
```

## Basic configuration concepts

Unbound works with sensible defaults, but it’s important to understand where it's configuration lives.

- Main config: `/etc/unbound/unbound.conf`
- Additional configs: `/etc/unbound/unbound.conf.d/`

The main config file has this line:

```conf
include-toplevel: "/etc/unbound/unbound.conf.d/*.conf"
```

This tells the unbound server to automatically add all the configurations inside the `unbound.conf.d` folder, which, by default has the files: 

- `remote-control.conf`

    With this content:

    ```conf
remote-control:
    remote-enable: yes
    control-interface: /run/unbound.ctl
    ```

<br>

- `root-auto-trust-anchor-file.conf`.

    With this content:

    ```conf
server:
    auto-trust-anchor-file: "/var/lib/unbound/root.key"
    ```

> 
> Including any of the above configurations in one of your config files will return configuration issues about duplicate entries.
{: .prompt-danger }

> If
> Unbound fails to start, check for port conflicts with sudo ss -tuln | grep :53. Disable conflicting services like systemd-resolved if necessary (e.g., sudo systemctl stop systemd-resolved and edit /etc/systemd/resolved.conf).
{: .prompt-info }


## Basic configuration file for recursive DNS

Below is a minimal `unbound` server configuration that uses a root hints file to bootstrap recursive resolution. Save it as `/etc/unbound/unbound.conf.d/01-root-hints.conf` (or similar).

```conf
server:
    # Listen on localhost only
    interface: 127.0.0.1/8
    interface: ::1
    port 53

    # Allow queries only from localhost range
    access-control: 127.0.0.0/8 allow
    access-control: 0.0.0.0/0 refuse
    
    # Root hints file path
    root-hints: "/etc/unbound/root.hints"

    # Log configuration
    verbosity: 1

    # Security & performance enhancements
    hide-identity: yes
    hide-version: yes
    prefetch: yes
    rrset-roundrobin: yes
```

Download the current root hints (named.root) and save it to `/etc/unbound/root.hints`:

```bash
sudo curl -o /etc/unbound/root.hints https://www.internic.net/domain/named.root
sudo chown root:root /etc/unbound/root.hints
sudo chmod 644 /etc/unbound/root.hints
```

> The root hints should be regularly updated, with a tool like crontab to regularly update the file
> Execute `crontab -e` and set a new trigger every sunday at 2am `0 2 * * 0 sudo curl -o /etc/unbound/root.hints https://www.internic.net/domain/named.root`


Explanation of the basic options:

- `interface`: The IP and range that the server will listen to (add another line with your local IP range)
- `root-hints`: path to a file containing the list of DNS root servers (used to bootstrap recursion).
- `auto-trust-anchor-file`: path to the DNSSEC root trust anchor (root key) used for validation.
- `interface` / `access-control`: define which addresses Unbound listens on and which clients are allowed.
- `prefetch`: helps keep popular records fresh by fetching them before they expire.
- `hide-identity` / `hide-version`: reduces fingerprinting by hiding server identity and version.

After adding the config and the root hints, validate and restart Unbound:

```bash
sudo unbound-checkconf
sudo systemctl restart unbound
sudo systemctl status unbound
```

### About `unbound-checkconf`

- Run `sudo unbound-checkconf` to validate your Unbound configuration **before** restarting the service. The command checks syntax and basic configuration problems and will return a **non-zero exit code** if it finds issues.
- On error it prints messages (usually including the file and line number) so you can locate and fix the problem. Example error output looks like:

```text
unbound-checkconf: error: can't parse 'access-control' at /etc/unbound/unbound.conf:45
```

- When the configuration is valid, many installations return exit code 0 and produce no output (some versions may print a short "no errors" message). If you want to be explicit, check the exit code with `echo $?` (0 means success).

- Fix any reported errors and re-run `unbound-checkconf` until it succeeds, then restart Unbound.

> Tip: Keep `/etc/unbound/root.hints` updated periodically (e.g., using a cron job or systemd timer) so the root server list stays current.

## Basic Testing & Troubleshooting

- Validate configuration: `unbound-checkconf`
- Query local resolver: `dig @127.0.0.1 example.com +dnssec`
- See the full resolution path: `dig +trace example.com`
- Check logs/status: `journalctl -u unbound -n 200 --no-pager` or `unbound-control status`
- If after installing packages `unbound` or `unbound-checkconf` aren't found maybe the path `/usr/sbin/` isn't set.

---

## Advanced configurations

### Privacy and hardening

Reducing fingerprinting and limiting data exposure.

```conf
server:
    qname-minimisation: yes
    minimal-responses: yes
    hide-identity: yes
    hide-version: yes
    harden-glue: yes
    harden-ddnssec-stripped: yes
    harden-short-bufsize: yes
    harden-large-queries: yes
    username: "unbound"
    chroot: "/var/lib/unbound"
```

- `harden-glue`: reduces risk from malicious [glue records](https://www.dondominio.com/en/help/204/glue-records-associated-dns/).
- `harden-dnssec-stripped`: detects and protects against servers that strip DNSSEC data.
- `harden-short-bufsize` and `harden-large-queries`: mitigates for some [UDP based amplification attacks](https://www.cisa.gov/news-events/alerts/2014/01/17/udp-based-amplification-attacks)
- `username` and `chroot`: run the service as an unprivileged user and chroot for extra isolation.   

> Some options depend on Unbound's version, always run `unbound-checkconf` before restarting the service to check for errors!



### Performance tuning: cache and RAM sizes

Tuning the cache and RAM usage is important for the amount of users, and queries per minute (queries/m or q/m).

```conf
server:
    rrset-cache-size: 32m
    msg-cache-size: 16m
```

- `rrset-cache-size`:
    - Is the cache for DNS resource record sets (RRsets) — the actual answers like A, AAAA, MX, TXT, NS, etc.
    - For example, it would store mappings like `example.com -> A 104.18.26.120` or `google.com -> AAAA 2a00:1450:4006:81a::200e`
    - To put it simply, it's the primary DNS cache, and has the most impact on performance.

- `msg-cache-size`:
    - Will store entire DNS messages exactly as received.
    - Full responses like `Question`, `Answer`, `Authority`, `Additional sections`.
    - It's a secondary cache as entries are less reusable as it must match exact queries.

A good rule of thumb is to keep the cache size of `rrset-cache-size` the double of `msg-cache-size`

| Total System RAM | `msg-cache-size` | `rrset-cache-size` | Estimated RAM usage |
|------------------|------------------|--------------------|---------------------|
| 512MB - 1GB      | 16MB             | 32MB               | ~ 80MB - 150MB      |
| 2G - 4G          | 128MB            | 256MB              | ~ 800MB - 1GB       |
| 8G               | 512MB            | 1024MB             | ~ 3GB - 4GB         |


Things you should keep in mind.

- `rrset-cache-size` matters more for performance than `msg-cache-size`
- Larger caches do not always result in faster DNS due to diminishing returns
- Over-allocating cache can hurt performance, especially on RAM-limited systems (e.g. VPSes)
- Unbound performs well even with modest cache sizes for small deployments


### Performance tuning: Threads

Unbound is multi-threaded and can take advantage of multiple CPU cores. Proper thread configuration can significantly improve performance on multi-core systems.

```conf
server:
    num-threads: 2
```

#### How does Unbound multi-threading work?

- Each thread runs an independent event loop
- Incoming queries are distributed across threads
- Each thread maintains its own internal structures but shares cache memory
- Threads are most useful under concurrent load, not single-query latency

Choosing the right number of threads

A common rule of thumb:

- Home system / single-user: 1 thread
- Small server / homelab: 2–4 threads
- Dedicated DNS server: number of physical CPU cores (or slightly less)

Example recommendations:

| CPU Cores | `num-threads` |
|-----------|---------------|
| 1         | 1             |
| 2         | 2             |
| 4         | 2-4           |
| 8         | 4-6           |

> You can find out how many cores the cpu has with `nproc` (usually installed in many linux distros).

